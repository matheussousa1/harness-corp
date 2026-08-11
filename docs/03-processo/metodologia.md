# Metodologia — Spec-Driven Development com agentes

> Camada 3 do harness: **processos**. O que impede o agente de virar um loop infinito
> caro e o que garante consistência de entrega.

## O loop

```
SPEC  →  PLAN  →  TASKS  →  IMPLEMENT  →  VERIFY  →  MEASURE
  ↑                                                      │
  └──────────────── ajusta rules / modelo ───────────────┘
```

Cada seta é um artefato versionado no Git, não uma conversa de chat. Conversa evapora;
artefato acumula.

## Os cinco passos

### 1. SPEC — *o quê e por quê*
Arquivo `specs/SPEC-XXX-<slug>.md` a partir de `specs/_template/SPEC-TEMPLATE.md`.

Contém: problema, escopo (**e não-escopo**), requisitos funcionais numerados
(`RF-01`…), requisitos não-funcionais, invariantes de segurança tocadas, contrato de
API afetado, **critérios de aceite verificáveis por máquina**, riscos.

**Não contém** solução, nome de arquivo, nome de classe.

Regra: se um critério de aceite não pode ser transformado em teste ou comando, ele
não é critério de aceite — é desejo. Reescreva.

### 2. PLAN — *como*
Seção "Plano" dentro da própria SPEC. Contém: abordagem escolhida, alternativas
descartadas em uma linha cada, arquivos a criar/alterar, mudanças de contrato e de
schema, plano de teste, plano de rollback.

Aqui é onde um humano gasta atenção. É o ponto de maior alavancagem do processo:
**o plano é a parte estratégica; a implementação é tática.** IA é ótima na tática.
Arquitetura e estratégia continuam sendo responsabilidade do desenvolvedor.

### 3. TASKS — *fatias*
Seção "Tasks" da SPEC. Cada task:
- é entregável em **≤ 1 dia**
- toca poucos arquivos
- tem um **comando de verificação** próprio
- declara dependências (`depende de: T-02`)

Tasks independentes são marcadas `[paralelizável]` — podem rodar em agentes
simultâneos.

### 4. IMPLEMENT — *tática*
Um agente Implementor por task. Regras:
- Lê a SPEC e **só** a SPEC + os arquivos da task. Sem "melhorar" o resto.
- Escreve teste junto, não depois.
- Se descobrir que a SPEC está errada, **para** e propõe emenda. Não improvisa.
- Roda `make check` antes de dizer que terminou.

### 5. VERIFY — *o gate*
Um agente Verifier, **diferente** de quem implementou. Confere:
- cada critério de aceite, um por um, com evidência (saída de comando)
- `make check` verde
- invariantes de segurança não violadas
- nada fora do escopo da task foi alterado (`git diff --stat`)

O Verifier existe porque quem escreveu o código (humano ou agente) é o pior juiz do
próprio trabalho. Ele também é quem detecta conflito entre Implementors paralelos
antes do merge.

### 6. MEASURE — *fecha o ciclo*
Registra em `metrics/` custo, tokens, tempo e nº de turnos da entrega.
Ver `docs/04-metricas/framework.md`.

## Papéis

| Papel | Quem | Responsabilidade |
|---|---|---|
| **Coordinator** | Humano (você) | Decide o que entra, aprova SPEC e PLAN, arbitra conflito |
| **Spec Writer** | Agente | Transforma ideia solta em SPEC completa e questionável |
| **Implementor** | Agente (1..N) | Executa uma task, com teste |
| **Verifier** | Agente | Prova que os critérios foram atendidos; nunca implementa |

Definições em `.claude/agents/`.

## Por que este processo e não "só ir pedindo pro agente"

Dados de 2026 (ver `docs/00-pesquisa/`): adoção de IA aumenta a efetividade individual
**e a instabilidade de entrega**, porque a geração de código cresce mais rápido que a
capacidade de review absorver. Sinais medidos: +67% de contextos de PR por dia,
+14% de retrabalho, +26% de tarefas travadas há 7+ dias.

O processo acima ataca exatamente isso: limita o WIP (uma SPEC por vez), fixa o escopo
antes de gerar código, e coloca um gate automatizado entre geração e merge.

## Definition of Ready / Done

`docs/03-processo/definition-of-ready-done.md`.

## Anti-padrões (rejeitar em review)

| Anti-padrão | Por quê |
|---|---|
| SPEC com critério de aceite subjetivo ("deve ser rápido") | Não verificável ⇒ não é gate |
| Implementor mexendo fora do escopo da task | Torna o diff irreviewável e quebra paralelismo |
| Agente rodando em loop até "dar certo" | Custo sem teto, resultado não reprodutível |
| Pular VERIFY porque "está óbvio que funciona" | É onde a instabilidade entra |
| SPEC escrita depois do código, para constar | Vira documentação morta |
