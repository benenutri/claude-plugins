# Simulador local do Mitra

Leia ao criar o simulador de um projeto novo, ao adicionar uma spec (novas SFs,
tabelas, dados) ou quando algo roda no simulador e não na plataforma (ou o
contrário).

## Por que existe

Nada do `mitra-sdk` roda da máquina local (não gera migration e bate em
produção). Sem simulador não há como saber se uma Server Function roda antes de
publicar no sandbox. Por isso o simulador **é a suíte de testes do backend** e
vai no git; só `simulator/data/` (o banco gerado) fica de fora.

## Princípio: nenhum handler reimplementado

O simulador carrega as `definitions` dos `add-*.mjs` — as mesmas que a
publicação envia ao Mitra — e executa o SQL literal das `type: 'SQL'` e o corpo
literal das `type: 'JAVASCRIPT'`. O schema é lido dos `CREATE TABLE` do
`setup-backend.mjs`. Um bug que existiria em produção aparece aqui; a correção
testada aqui é a que vai publicada. Handler próprio testaria a si mesmo.

O preço é a tradução MySQL → SQLite, concentrada em `mysql-dialect.mjs`. Regra
de ouro dessa tradução: **preservar o significado**, nunca só evitar o erro de
sintaxe — um shim que devolve valor plausível e errado transforma o simulador
em fonte de falso positivo.

## O kit (`assets/backend/simulator/`)

| Arquivo | Papel | Muda por projeto? |
|---|---|---|
| `config.mjs` | porta, semente, `PUBLISH_ORDER`, `DDL_SOURCES`, mapa do frontend, `CHECK_INPUT` | **sim — o único** |
| `seed.mjs` | usuários (`INT_USER`) e dados do domínio, determinísticos | sim (dado do projeto) |
| `runtime.mjs` | carrega definitions, `bindParams` (`{{x}}`, `:VAR_USER`), executa SQL/JS com `mitra-sdk` falso | não |
| `server.mjs` | HTTP no protocolo do SDK; resolve ids pelo mapa do frontend; login com JWT não assinado | não |
| `schema.mjs` | lê DDL dos arquivos em `DDL_SOURCES` e traduz para SQLite | não |
| `mysql-dialect.mjs` | funções MySQL registradas no SQLite + reescritas sintáticas | só quando o SQL usa algo novo |
| `check.mjs` | roda todas as SFs SQL de leitura com `CHECK_INPUT` | não |
| `seed-cli.mjs` | regera o banco | não |
| `README.md` | logins, portas, endpoints | sim (texto) |

Requer Node 22.5+ (`node:sqlite`). Sem dependência npm além do que o backend já tem.

## Instalar num projeto

1. Copie `assets/backend/simulator/` para `backend/simulator/` e os scripts
   `sim`, `sim:seed`, `sim:check` para `backend/package.json`.
2. Edite `config.mjs`: `PROJECT_LABEL`, `SEED` (= NNNNN), `PORT` (próxima
   livre — ver `estrutura-projeto.md`), `PUBLISH_ORDER` (todos os `add-*.mjs`
   na ordem de publicação), `DDL_SOURCES` (setup + qualquer `add-*` que crie
   tabela), `CHECK_INPUT` (nomes dos parâmetros que as SFs de leitura usam).
3. Edite `seed.mjs`: usuários com os perfis que o app distingue; **um registro
   por cenário da spec** (C-001…), datas relativas a `hoje`.
4. Garanta que cada `add-*.mjs` exporta `definitions` e só publica no
   `isEntrypoint()`; que cada tabela do setup está num template literal próprio
   começando pela frase `CREATE TABLE IF NOT EXISTS NOME (`.
5. `frontend/src/lib/server-functions.ts` com os dois blocos (`SERVER_FUNCTION_NAMES`
   / `SERVER_FUNCTIONS`) no formato rígido — o simulador e o sync fazem parse
   por texto (uma linha `chave: 'valor',` por entrada, sem comentário dentro).
6. `frontend/src/lib/mitra-auth.ts` com `SIMULADOR_URL` na porta escolhida e
   uma `LoginPage` que, em `MODO_LOCAL`, mostra e-mail/senha e chama
   `loginLocally()`.
7. `cd backend && npm run sim:check` → deve terminar sem `FALHOU`.
8. Registre no `mitra-hub/server.mjs`, adicione `.claude/launch.json` e a
   entrada no `.gitignore` (`backend/simulator/data/`).

## Ao adicionar uma spec

- Novo `add-NNN-*.mjs` → entra em `PUBLISH_ORDER` (e em `DDL_SOURCES` se criar
  tabela). Nomes novos → `SERVER_FUNCTION_NAMES` + um id local qualquer em
  `SERVER_FUNCTIONS`.
- Parâmetro novo de leitura → `CHECK_INPUT`.
- Cenário novo → registro no `seed.mjs`; `npm run sim:seed` para regerar.
- Função MySQL nova no SQL (`DATE_FORMAT`, `TIMESTAMPDIFF`, `FIELD`,
  `GROUP_CONCAT SEPARATOR`, `CAST AS UNSIGNED`…) → confira se o dialeto cobre;
  se não, adicione shim **com o mesmo significado**.
- Coluna nova via `ALTER TABLE` num `add-*`: o simulador não lê ALTER; adicione
  a coluna também no `CREATE` do setup (o setup é idempotente na plataforma
  porque é `IF NOT EXISTS`, e o ALTER no `add-*` cobre o banco que já existe).

## smoke.mjs (fluxo de escrita ponta a ponta)

`check.mjs` responde "o SQL roda?". Para "o fluxo faz a coisa certa?" crie
`simulator/smoke.mjs` por projeto: banco descartável em `tmpdir()`, executa as
SFs JAVASCRIPT reais e verifica com `assert` o que a spec exige. Esqueleto:

```js
import assert from 'node:assert/strict';
import { DatabaseSync } from 'node:sqlite';
import { rmSync } from 'node:fs';
import { tmpdir } from 'node:os';
import { join } from 'node:path';
import { seedDatabase } from './seed.mjs';
import { registerMysqlFunctions } from './mysql-dialect.mjs';
import { loadDefinitions, createRunner } from './runtime.mjs';

const DB = join(tmpdir(), 'p-NNNNN-smoke.db');
rmSync(DB, { force: true });
seedDatabase(DB, { reset: true });
const db = new DatabaseSync(DB);
registerMysqlFunctions(db);
const { byName } = await loadDefinitions();
const execute = createRunner(db);
const USUARIO = 1; // INT_USER.ID → :VAR_USER
const chamar = (nome, input, userId = USUARIO) => execute(byName.get(nome), input, userId);

let feitos = 0;
const teste = async (titulo, fn) => { await fn(); feitos += 1; console.log(`  ok  ${titulo}`); };

await teste('C-006: cadastro sem campo obrigatório é recusado com o motivo', async () => {
  const saida = await chamar('appExemploValidarTitulo', { titulo: '' });
  assert.equal(saida.ok, false);
  assert.match(saida.error, /título/i);
});

console.log(`\n${feitos} verificações passaram.`);
db.close();
```

Cada `teste` cita o cenário/requisito da spec no título — é a rastreabilidade
que faz o smoke valer como evidência.

## Quando a SF fala com banco externo (JDBC ≠ 1) ou integração

O simulador roda tudo no mesmo SQLite. Duas saídas, em ordem de preferência:

1. **Semear as tabelas externas** com o mesmo nome e colunas que o SQL usa
   (p-45547 semeia `PCPRODUT`, `PCEST`, `PCMOV`… do WinThor). Fica testável e
   determinístico.
2. Devolver lista vazia sem quebrar a tela (`/interactions/integrations/call`
   já faz isso em SQL que o SQLite não entende) — só para SQL Oracle que não
   compensa traduzir.

SF `INTEGRATION` é tratada como SQL no SQLite: se a tabela existe no seed, roda.

## Diagnóstico

- `GET /health`: `semDefinicao` lista ids que a tela pediu e não existem (mapa
  do frontend desatualizado ou SF sem definition); `erros` traz a mensagem
  real do SQL que falhou; `redefinidas` acusa dois scripts com o mesmo nome.
- Tela vazia sem erro: veja o log `[simulador] Server Function N sem definicao`.
- `executionStatus: 'FAILED'` no frontend: a mensagem em `result.error` é o
  erro do SQLite — geralmente sintaxe MySQL sem tradução ou coluna que o seed
  não criou (setup desatualizado?).
- Divergência plataforma × simulador sem erro em nenhum dos dois: suspeite do
  dialeto (`toSqlite`) antes do SQL.

## Voltar ao Mitra real

`VITE_MITRA_AUTH_URL` + `VITE_MITRA_PROJECT_ID` preenchidos no `frontend/.env`.
Sem alteração de código. Lembre: com eles preenchidos, **toda chamada bate em
produção**.
