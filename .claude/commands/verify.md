---
description: Verifica uma task ou SPEC contra os critérios de aceite
argument-hint: SPEC-XXX [T-nn]
---

Leia `prompts/workflows/verify.md` e siga por completo.

**Execute pelo subagente `verifier`.** Ele não tem ferramenta de escrita — é isso que
torna o gate independente. Executar inline na sessão principal anula o gate.

**{ARGS}** = $ARGUMENTS

Este arquivo é só um atalho. A instrução real, compartilhada por todas as ferramentas
de IA do projeto, está em `prompts/workflows/verify.md`. Não duplique regra aqui —
edite lá.