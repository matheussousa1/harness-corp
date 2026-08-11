# CLAUDE.md

As regras do projeto estão em **`AGENTS.md`** — fonte única, compartilhada com Codex,
Gemini CLI, opencode e demais ferramentas. Leia antes de agir.

@AGENTS.md

---

## Específico do Claude Code

- **Subagentes:** `.claude/agents/` (`spec-writer`, `implementor`, `verifier`). Cada um
  é um atalho; a definição real está em `prompts/roles/`.
- **Slash commands:** `.claude/commands/` (`/spec`, `/plan`, `/tasks`, `/implement`,
  `/verify`, `/measure`). Também são atalhos para `prompts/workflows/`.
- **Verify precisa ser um agente diferente do implementor.** Use o subagente
  `verifier`, que não tem ferramenta de escrita — é o que garante a independência do gate.

Não duplique regra aqui. Regra nova vai para `AGENTS.md`; instrução de fluxo vai para
`prompts/`.
