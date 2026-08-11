---
description: Escreve a seção Plano de uma SPEC já aprovada em escopo
argument-hint: SPEC-XXX
---

Escreva a seção **Plano** da SPEC `$ARGUMENTS`.

1. Leia a SPEC inteira e confirme que as seções 1–8 estão completas. Se faltar
   critério de aceite verificável, **pare** e aponte quais.
2. Explore o código existente antes de decidir (`Glob`/`Grep`). Não invente arquivo.
3. Use o agente `spec-writer`. Preencha:
   - Abordagem escolhida (3–8 linhas)
   - Alternativas descartadas (mínimo 2, uma linha cada)
   - Tabela de arquivos (criar/alterar + o que muda)
   - Mudanças de schema/contrato
   - Plano de teste (unit, integração, falha/borda)
   - Rollback
4. Se o plano exigir uma decisão arquitetural nova, proponha o ADR correspondente em
   `docs/02-arquitetura/adr/` — não a enterre no plano.

Ainda **não** implemente nada.
