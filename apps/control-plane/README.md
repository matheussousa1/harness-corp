# control-plane

Backend. **O produto está aqui.** Python 3.12 + FastAPI + LangGraph.

Único componente que conhece as API keys dos providers e que decide política e budget.

## Estrutura alvo

```
src/
  api/           rotas FastAPI. Finas: validam, enfileiram, fazem streaming.
                 NUNCA chamam LLM diretamente.
  policy/        resolução em cascata. Única fonte de verdade sobre permissão.
  budget/        reserve() / settle(). Contadores atômicos + ledger.
  orchestrator/  grafos LangGraph. Rodam em worker, fora do request HTTP.
  router/        drivers de provider, failover, pricing. Única camada que toca a chave.
  knowledge/     KB corporativa. Filtro ACL aplicado NA CONSULTA.
  audit/         log append-only.
config/
  models.yaml    registro de modelos lógicos → cadeia de providers + preços
tests/
```

## Regras deste app

- `router/` não importa nada de `api/`. A dependência é sempre para dentro.
- Nenhum módulo além de `router/` lê credencial de provider.
- Nó de grafo é função pura sobre `State`. Efeito colateral só em tool.
- Checkpointer **sempre** Postgres, inclusive em dev. `MemorySaver` só em teste unitário.
- Todo `except` em `policy/`, `budget/` ou credencial **nega ou re-lança**. Fail-open
  é bug de segurança, não estilo.
- Toda função pública tem tipo. `mypy --strict` é gate.

## Comandos

```bash
make check
```

Estado: não iniciado. Nasce em `specs/SPEC-000-fundacao-e-gates.md`.
