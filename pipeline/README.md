# pipeline/

O harness que **constrói** o harness-corp. Não confunda com o grafo de produto
(`apps/control-plane/orchestrator/`) — ver `docs/03-processo/pipeline.md`.

## Conteúdo

```
graphs/     definição dos grafos LangGraph do fluxo de desenvolvimento
prompts/    prompts estáveis e versionados usados pelos nós
```

## Por que os prompts ficam em arquivo versionado

Prompt em string dentro do código é impossível de fazer diff, de testar e de
estabilizar. Ordem de contexto instável destrói o cache de prompt — que aparece em
`metrics/` como `cache_read` baixo e custo alto. Prompt é artefato, com versão.

## Grafo do fluxo de desenvolvimento

`graphs/dev_flow.py` (a implementar) modela o que hoje é feito por slash commands:

```
spec → review(humano, interrupt) → plan → tasks → fila
fila → implement(N paralelos) → verify → {merge | fila(retry≤2) | blocked}
merge → measure
```

Regras que o grafo precisa impor:
- `recursion_limit` obrigatório
- máximo de 2 retentativas por task; depois vira `blocked`
- máximo de 3 implementors simultâneos
- teto de custo por task; estourou, para e reporta
- `interrupt()` nos pontos de decisão humana (aprovar SPEC, aprovar plano)

## Ordem de construção

Só vale automatizar o que já roda bem manualmente. Sequência:

1. **Agora:** rodar o fluxo pelos slash commands (`.claude/commands/`), à mão.
2. Depois de ~3 SPECs, com `metrics/` mostrando onde o tempo humano vai, automatizar
   o trecho mais repetitivo — provavelmente `implement → verify → retry`.
3. Só então o grafo completo.

Automatizar antes de medir é construir o harness no escuro — exatamente o erro que
`docs/03-processo/agent-experience.md` descreve no "princípio da subtração".
