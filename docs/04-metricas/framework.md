# Fase 4 — Métricas e otimização

> Sem isso, o harness é opinião. Com isso, é engenharia.

## Duas famílias de métrica

| Família | Pergunta que responde | Onde |
|---|---|---|
| **Métricas do harness** | O nosso processo de construir com IA está valendo a pena? | `metrics/` |
| **Métricas do produto** | O produto que entregamos está saudável? | telemetria do Control Plane |

---

## A. Métricas do harness (por SPEC e por task)

Registradas em `metrics/SPEC-XXX.json` ao fim de cada entrega.

| Métrica | Unidade | Por que importa |
|---|---|---|
| `cost_brl` | R$ | O número que decide se vale a pena |
| `tokens.input` / `output` / `cache_read` / `cache_write` | tokens | Onde o custo está; cache alto = harness eficiente |
| `wall_time_s` | s | Tempo de parede da entrega |
| `turns` | nº | Turnos até o gate verde. Alto = SPEC ambígua |
| `verify_rejections` | nº | Quantas vezes o Verifier reprovou |
| `files_read` / `files_changed` | nº | Sinal de AX; ler muito e mudar pouco = navegação ruim |
| `scope_violations` | nº | Agente mexeu fora do plano |
| `model_by_stage` | mapa | Qual modelo rodou em cada estágio |
| `human_minutes` | min | Tempo humano gasto. É o custo mais caro |

### Derivadas

```
custo_por_task        = cost_brl / tasks_entregues
custo_por_linha_util  = cost_brl / linhas_que_sobreviveram_30d
razao_retrabalho      = turns_de_retentativa / turns_totais
alavancagem           = human_minutes_estimado_sem_ia / human_minutes_real
```

`custo_por_linha_util` é a métrica honesta: código gerado que é deletado em 30 dias
foi dinheiro queimado, não produtividade.

### Formato

```json
{
  "spec": "SPEC-001",
  "date": "2026-08-14",
  "cost_brl": 18.40,
  "tokens": { "input": 412000, "output": 38000, "cache_read": 980000, "cache_write": 120000 },
  "wall_time_s": 5400,
  "human_minutes": 55,
  "turns": 14,
  "verify_rejections": 2,
  "files_read": 22,
  "files_changed": 9,
  "scope_violations": 0,
  "model_by_stage": { "spec": "opus", "implement": "sonnet", "verify": "opus" },
  "retro": "Task T-03 mal fatiada: misturou schema e handler. Fatiar por camada."
}
```

### Ciclo de otimização (a cada 5 SPECs)

1. O custo valeu a entrega? Compare `cost_brl` com `human_minutes` economizados.
2. Onde está o gasto? Se `input` domina → problema de contexto (AX/instrução inchada).
   Se `output` domina → tasks grandes demais.
3. `cache_read` baixo → estrutura de prompt instável; estabilize a ordem do contexto.
4. `turns` alto → SPEC ambígua, não modelo fraco. Corrija a SPEC antes de subir de modelo.
5. **Só então** mexa no modelo: rebaixe um estágio de cada vez e compare
   `verify_rejections`. Se não piorou, ficou mais barato de graça.

Regra: **muda uma variável por ciclo.** Trocar modelo e rules juntos não ensina nada.

---

## B. Métricas do produto

### Delivery (DORA, base)
lead time, frequência de deploy, taxa de falha de mudança, tempo de recuperação.

### DORA estendido para código gerado por IA
DORA puro não basta quando 30–70% do código é gerado. Adicionamos:

| Métrica | Definição |
|---|---|
| **Atribuição de IA** | % do diff originado de agente |
| **Durabilidade do código** | % das linhas geradas ainda vivas após 30 dias |
| **Throughput ajustado por complexidade** | entregas ponderadas por tamanho/risco, não LOC |
| **Estabilidade** | mudança na taxa de falha desde a adoção de IA — o sinal que o DORA aponta como risco |

### Operação do harness corporativo (runtime)

| Métrica | Alerta |
|---|---|
| Custo por workspace / por usuário / por conversa | 80% do budget |
| Tokens por turno (p50 / p95) | p95 > 2× p50 sustentado |
| Latência até o primeiro token (p95) | > 1,2 s |
| Overhead do Control Plane sobre a chamada crua (p50) | > 150 ms |
| Taxa de negação por política | pico súbito = política mal configurada |
| Taxa de falha de provider / acionamentos de failover | > 1% |
| Cache hit de contexto | queda súbita |
| Turnos que pararam esperando aprovação humana | fila crescendo = fricção |

Instrumentação: **OpenTelemetry em cada nó do grafo e cada tool**, exportado para
Langfuse (custo/traço de LLM) + Prometheus/Grafana (infra).

---

## Regra final

Toda otimização precisa de um **antes e depois numérico** no mesmo formato.
Sem número, é achismo — e achismo com IA sai caro em escala.
