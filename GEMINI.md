# GEMINI.md

As regras do projeto estão em **`AGENTS.md`** — fonte única, compartilhada com Claude
Code, Codex, opencode e demais ferramentas. Leia antes de agir.

@AGENTS.md

---

## Específico do Gemini CLI

- **Custom commands:** `.gemini/commands/*.toml` (`/spec`, `/plan`, `/tasks`,
  `/implement`, `/verify`, `/measure`). São atalhos para `prompts/workflows/`.
- O Gemini CLI não tem subagentes com ferramentas restritas. Para o gate de
  verificação, **rode o `/verify` em uma sessão separada**, sem o contexto de quem
  implementou — a independência do Verifier é o que faz o gate valer
  (`docs/03-processo/metodologia.md`).

Não duplique regra aqui. Regra nova vai para `AGENTS.md`.
