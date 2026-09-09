# Spec: [NOME DA FEATURE]

**ID:** NNN-slug · **Status:** rascunho | em revisão | aprovada · **Data:** AAAA-MM-DD

> Escrito para quem usa e para quem paga. **Proibido** citar stack, tabela,
> endpoint, componente ou biblioteca. Isso é assunto do `plan.md`.

## Problema

[Que dor existe hoje, para quem, e o que custa deixar como está. 3-5 linhas.]

## Resultado esperado

[Como fica depois. Uma frase que o usuário final assinaria embaixo.]

## Usuários

| Perfil | O que precisa fazer | Frequência |
|---|---|---|
| [ex: gestor financeiro] | [ex: acompanhar saldo por centro de custo] | [diária] |

## Cenários

Formato Given/When/Then — cada cenário é verificável por uma pessoa.

**C-001 — [título]**
- **Dado que** [estado inicial]
- **Quando** [ação do usuário]
- **Então** [resultado observável]

**C-002 — [caso de erro / borda]**
- **Dado que** …
- **Quando** …
- **Então** …

## Requisitos funcionais

| ID | Requisito | Critério de aceite | Prioridade |
|---|---|---|---|
| RF-001 | O sistema DEVE … | [verificável, com número quando couber] | must |
| RF-002 | O sistema DEVE … | … | should |

> `must` = sem isso a feature não entrega valor. `should` = entra se couber.
> `could` = registrado, não prometido.

## Requisitos não-funcionais

| ID | Requisito | Como se mede |
|---|---|---|
| RNF-001 | [ex: listagem responde em até 2s com 10k registros] | [cronômetro, volume de teste] |

## Entidades do domínio

Conceitos do negócio e como se relacionam — **sem** schema, tipo ou tabela.

- **[Entidade]** — [o que representa]. Relaciona-se com [outra] por [regra].

## Regras de negócio

- **RN-001** — [ex: lançamento com data futura entra como "pendente"]

## Fora de escopo

Obrigatório. O que não estiver listado como incluído está excluído.

- [ ] [o que explicitamente NÃO será feito, e por quê]

## Pendências

Cada item aqui **bloqueia** o `plan.md`.

- [ ] `[NEEDS CLARIFICATION: pergunta objetiva para o usuário]`

## Checklist de revisão

- [ ] Nenhuma menção a tecnologia, tabela ou componente
- [ ] Todo RF tem critério de aceite verificável
- [ ] Seção "Fora de escopo" preenchida
- [ ] Cenários cobrem pelo menos um caminho de erro
- [ ] Zero `[NEEDS CLARIFICATION]` em aberto
