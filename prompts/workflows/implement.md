> **{ARGS}** = os argumentos passados na invocação (ex.: `SPEC-000 T-01`).
> Fonte única deste fluxo. Os arquivos de comando de cada ferramenta só apontam para cá.

Execute a task **{ARGS}**.

Antes de começar, confirme:
- [ ] A SPEC está com status `Aprovada` ou `Em execução`
- [ ] As dependências da task estão `done`
- [ ] O plano lista os arquivos que a task pode tocar

Se algum item falhar, **pare** e diga qual.

Use o papel `implementor` (prompts/roles/implementor.md). Ao final:
1. Atualize a tabela **Execução** da SPEC (status, arquivos, notas).
2. Rode `just check` e cole o exit code.
3. Reporte no formato de saída definido em `prompts/roles/implementor.md`.

Não toque em arquivo fora do plano da task. Se precisar, pare e reporte
`needs-spec-amendment`.
