# Rodando o pipeline com várias IAs

O projeto é usado com **Claude Code, Codex, Gemini CLI e opencode**. O fluxo
spec-driven é o mesmo em todos. O que muda é só como cada ferramenta é acionada.

## Princípio: uma fonte, vários atalhos

```
AGENTS.md                    regras do projeto  ← fonte única
prompts/workflows/*.md       instruções de cada etapa  ← fonte única
prompts/roles/*.md           definição dos papéis  ← fonte única
        ▲            ▲            ▲            ▲
   .claude/     .gemini/    .opencode/     Codex
   commands/    commands/   commands/    (via AGENTS.md)
```

Os arquivos em `.claude/`, `.gemini/` e `.opencode/` têm 5 linhas cada: descrição,
argumentos e um ponteiro. **Toda a lógica está em `prompts/`.**

Motivo: seis fluxos × quatro ferramentas = 24 cópias para manter em sincronia. Cópias
divergem em silêncio, e aí cada IA passa a seguir um processo diferente — o oposto do
que um harness deve fazer. Ferramenta nova = 6 ponteiros, zero regra reescrita.

## Por ferramenta

### Claude Code
- Regras: `CLAUDE.md` → importa `AGENTS.md`
- Comandos: `.claude/commands/*.md` → `/spec`, `/plan`, `/tasks`, `/implement`, `/verify`, `/measure`
- Subagentes: `.claude/agents/*.md`
- **Vantagem:** o `verifier` roda como subagente sem ferramenta de escrita. O gate é
  independente por construção.

```
/implement SPEC-000 T-01
```

### Codex
- Regras: lê `AGENTS.md` nativamente. Nada a configurar.
- Comandos: prompts customizados do Codex são globais (`~/.codex/prompts/`), não
  versionáveis no repo. Use a invocação direta:

```
Leia prompts/workflows/implement.md e siga por completo. {ARGS} = SPEC-000 T-01
```

Opcional, para ganhar o `/implement`: `just sync-codex-prompts` copia
`prompts/workflows/*.md` para `~/.codex/prompts/`. Vira cópia local, fora do Git —
rode de novo quando os prompts mudarem.

### Gemini CLI
- Regras: `GEMINI.md` → importa `AGENTS.md`
- Comandos: `.gemini/commands/*.toml`

```
/implement SPEC-000 T-01
```

### opencode
- Regras: lê `AGENTS.md` nativamente.
- Comandos: `.opencode/commands/*.md`, que usam `@prompts/workflows/<etapa>.md` para
  incluir o conteúdo real.

```
/implement SPEC-000 T-01
```

## O que NÃO é portável — e como compensar

| Recurso | Claude Code | Codex | Gemini CLI | opencode |
|---|---|---|---|---|
| Regras de projeto em arquivo | sim | sim | sim | sim |
| Slash commands versionados no repo | sim | **não** (global) | sim | sim |
| Subagente com ferramentas restritas | sim | parcial | **não** | parcial |

**A restrição que importa é a terceira.** O `verifier` só é um gate de verdade se não
puder editar arquivo — senão ele "conserta e aprova", e a independência morre.

Onde não dá para restringir ferramenta:

> Rode o `verify` em **sessão nova**, sem o contexto de quem implementou, e cole a
> saída dos comandos no relatório.

Sessão separada não é tão forte quanto ferramenta restrita, mas remove o viés de
"eu escrevi isso, então está certo", que é a maior parte do problema.

## Qual IA em qual etapa

Hipótese inicial. `metrics/` corrige ao longo do tempo (ver `docs/04-metricas/framework.md`).

| Etapa | Sugestão | Por quê |
|---|---|---|
| spec / plan | modelo mais capaz disponível | maior alavancagem; errar aqui custa a SPEC inteira |
| tasks | mesmo da spec | é continuação do raciocínio |
| implement | modelo médio, ferramenta com boa edição de arquivo | tática; a SPEC já removeu a ambiguidade |
| verify | modelo capaz, **ferramenta diferente da que implementou** | ceticismo + independência |
| measure | nenhum (script) | não gaste LLM onde `jq` resolve |

Usar ferramentas diferentes para implement e verify é um bônus real do setup
multi-IA: o verificador não herda os pontos cegos de quem escreveu.

Registre em `metrics/SPEC-XXX.json`, no campo `model_by_stage`, **qual ferramenta e
qual modelo** rodaram cada etapa. Sem isso, comparar custo entre SPECs não significa nada.

## Regra

Regra nova vai para `AGENTS.md`. Instrução de fluxo vai para `prompts/`.
Nada de regra dentro dos arquivos de `.claude/`, `.gemini/` ou `.opencode/` —
eles são ponteiros, e ponteiro com lógica vira a cópia divergente que este desenho
existe para evitar.
