# Plan: [NOME DA FEATURE]

**Spec:** `../NNN-slug/spec.md` · **Data:** AAAA-MM-DD

> O COMO. Nenhum requisito novo nasce aqui — se faltou algo, volta pra spec.

## Checagem de princípios

Preencher **antes** de detalhar o desenho. Falha aqui é replanejamento.

| Princípio | Toca? | Como atende |
|---|---|---|
| Spec antes de código | sim | spec aprovada em `../NNN-slug/spec.md`, zero pendências |
| Backend escrito local, executado no sandbox | | [quais `add-*.mjs`/`setup-backend.mjs` mudam; nada roda do `mitra-sdk` daqui] |
| Migrations append-only | | [nada em `backend/migrations/` é tocado à mão] |
| Leitura e escrita por Server Function | | [nenhum `listRecords`/`patchRecord` em tela de usuário final] |
| Simulador cobre a feature | | [definitions carregadas, seed com os cenários da spec, `sim:check` verde] |
| Design tokens, sem cor literal | | [tokens usados; zero hex] |
| Simplicidade | | [nenhuma abstração com uma implementação só; dependência nova justificada abaixo] |

## Contexto técnico

- **Frontend:** React 19 · Vite 7 · Tailwind 4 · react-router 7 · `mitra-interactions-sdk`
- **Backend:** tabelas e Server Functions na plataforma Mitra, provisionadas por `backend/*.mjs` com `mitra-sdk`
- **Reuso identificado:** [componentes/hooks/libs existentes que já cobrem parte]
- **Dependência nova:** nenhuma | [nome + por que nada instalado resolve]

## Modelo de dados

Entidades da spec → estrutura real. Toda mudança de schema entra em
`setup-backend.mjs` (ou num `add-NNN-*.mjs`) e vira migration gerada pelo
sistema ao rodar no sandbox — nunca escrita à mão.

| Entidade (spec) | Tabela | Campos-chave | Observação |
|---|---|---|---|
| | `NOME_TABELA` | `ID`, ... | datas em `VARCHAR(10/19)`; booleanos `BOOLEAN` |

## Contratos de backend

Uma linha por Server Function. Nome é o contrato com `SERVER_FUNCTION_NAMES`.

| Nome (`add-NNN`) | Tipo | Entrada (`{{param}}`) | Saída | Atende |
|---|---|---|---|---|
| `appExemploListar` | SQL | `busca`, `status`, `limite`, `offset` | linhas `ID, TITULO, STATUS, CRIADO_EM` | RF-001 |
| `appExemploInserir` | SQL | `titulo`, `criadoEm` | — | RF-002 |
| `appExemploValidar` | JAVASCRIPT | `titulo` | `{ ok, error? }` | RN-001 |

Regras: filtro desligado é `''` no texto e `0` no numérico; toda listagem tem
`LIMIT/OFFSET`; `:VAR_USER` identifica quem chama; escrita de usuário business
só por SF SQL.

## Superfície de interação

| Tela / rota | Responsabilidade | Componentes reusados | Novos |
|---|---|---|---|

## Fluxo

[Do gatilho do usuário até a persistência. Texto ou lista numerada.
 Diagrama só se o texto realmente não der conta.]

## Simulador

- Seed: [quais registros entram em `simulator/seed.mjs` para cobrir C-001…C-00N]
- `CHECK_INPUT`: [parâmetros novos que o `sim:check` precisa conhecer]
- Dialeto: [função MySQL nova usada no SQL? precisa de shim em `mysql-dialect.mjs`?]

## Decisões

| # | Decisão | Alternativa descartada | Por quê |
|---|---|---|---|
| D-001 | | | |

## Riscos

| Risco | Impacto | Mitigação |
|---|---|---|

## Complexity Tracking

Só preencher se houver violação de princípio. Vazio é o estado saudável.

| Princípio violado | Por que é necessário | Por que a alternativa simples não serve |
|---|---|---|
