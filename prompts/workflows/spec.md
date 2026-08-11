> **{ARGS}** = os argumentos passados na invocação (ex.: `SPEC-000 T-01`).
> Fonte única deste fluxo. Os arquivos de comando de cada ferramenta só apontam para cá.

Crie uma nova SPEC para: **{ARGS}**

1. Descubra o próximo número livre em `specs/` (`SPEC-XXX`).
2. Use o papel `spec-writer` (prompts/roles/spec-writer.md).
3. A SPEC deve seguir `specs/_template/SPEC-TEMPLATE.md` na íntegra.
4. Só preencha as seções 1–8 (o *quê*). **Não** escreva o Plano nem as Tasks ainda —
   isso é `/plan` e `/tasks`, depois que eu revisar o escopo.
5. Ao final, liste as perguntas em aberto que precisam de decisão minha.

Não escreva código.
