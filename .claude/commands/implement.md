---
description: Executa uma task de uma SPEC com o agente implementor
argument-hint: SPEC-XXX T-nn
---

Execute a task **$ARGUMENTS**.

Antes de começar, confirme:
- [ ] A SPEC está com status `Aprovada` ou `Em execução`
- [ ] As dependências da task estão `done`
- [ ] O plano lista os arquivos que a task pode tocar

Se algum item falhar, **pare** e diga qual.

Use o agente `implementor`. Ao final:
1. Atualize a tabela **Execução** da SPEC (status, arquivos, notas).
2. Rode `make check` e cole o exit code.
3. Reporte no formato de saída definido em `.claude/agents/implementor.md`.

Não toque em arquivo fora do plano da task. Se precisar, pare e reporte
`needs-spec-amendment`.
