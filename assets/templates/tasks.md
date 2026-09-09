# Tasks: [NOME DA FEATURE]

**Plan:** `plan.md` · **Data:** AAAA-MM-DD

> Ordem de execução. Nenhuma decisão de design se resolve aqui — se apareceu
> uma, volta pro `plan.md`.
>
> `[P]` = paralelizável (arquivos distintos, sem dependência entre si).

## Backend

- [ ] **T-001** — [ação concreta] → `backend/[arquivo]`
- [ ] **T-002** `[P]` — …

## Frontend

- [ ] **T-010** — [ação concreta] → `frontend/src/[arquivo]`
- [ ] **T-011** `[P]` — …

## Verificação

- [ ] **T-090** — `cd frontend && npm run build` passa limpo
- [ ] **T-091** — Cenário **C-001** da spec reproduzido manualmente: [resultado esperado]
- [ ] **T-092** — Cenário de erro **C-002** reproduzido: [resultado esperado]
- [ ] **T-093** — Tema claro (`.theme-light`) sem cor quebrada

## Dependências

```
T-001 → T-002 → T-010
T-011 [P] independente
```

## Rastreabilidade

Toda task existe por causa de um requisito. Requisito sem task é escopo perdido.

| Requisito | Tasks |
|---|---|
| RF-001 | T-001, T-010 |
