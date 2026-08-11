# Visão e Escopo

## Problema

Empresas adotaram IA por pessoa, não por organização. O resultado hoje:

- Cada colaborador usa a conta/chave que quiser → **dado corporativo vaza** para
  provider não aprovado.
- Ninguém sabe **quanto se gasta**, nem por quem, nem em quê.
- Não existe **política**: qualquer um pode plugar qualquer ferramenta em qualquer modelo.
- O conhecimento da empresa não está no contexto do modelo → respostas genéricas.
- Trocar de provider é um projeto, não uma configuração → **lock-in**.

## Visão

> Um sistema operacional de IA para a empresa: o colaborador ganha um agente que
> conhece o contexto do negócio; a empresa mantém controle total sobre modelos,
> ferramentas, dados e custo — e pode trocar de provider a qualquer momento.

## Proposta de valor

| Para | Valor |
|---|---|
| Colaborador | Agente com memória do seu trabalho + base de conhecimento da empresa, integrado às ferramentas que ele já usa |
| Gestor / Admin | Painel com controle de modelos, tools, budget e políticas por workspace; visibilidade de custo |
| Segurança / Jurídico | Chave nunca na máquina do usuário, trilha de auditoria completa, fail-closed |
| Engenharia | Multiprovider real: troca de modelo é config, não refactor |

## Escopo — MVP (V1)

### Dentro

| # | Capacidade | Épico |
|---|---|---|
| 1 | Gateway de inferência multiprovider com streaming | E1 |
| 2 | Política em cascata (workspace > pessoal > nenhuma) | E2 |
| 3 | Budget hierárquico com reserva antes do turno | E2 |
| 4 | Painel admin: workspaces, modelos, tools, budgets, políticas | E3 |
| 5 | App desktop Electron com chat + seleção de modelo permitido | E4 |
| 6 | Memória semântica local (SQLite + `sqlite-vec`) | E5 |
| 7 | Base de conhecimento corporativa (RAG) filtrada por permissão | E6 |
| 8 | Tools via MCP, habilitadas por política | E7 |
| 9 | Aprovação por conversa (auto-aprovado vs. confirmação humana) | E7 |
| 10 | Telemetria de custo/tokens/tempo por execução | E8 |

### Fora do MVP (backlog explícito)

- Transcrição de reuniões
- Crons / automações agendadas
- Kanban interno de tarefas
- Integrações prontas (ClickUp, Google Drive, Jira) — o MVP entrega **MCP genérico**;
  conectores específicos vêm depois
- Mobile / web app (desktop primeiro)
- SSO/SAML corporativo (V1 usa OIDC simples)
- Fine-tuning de modelo

### Não-objetivos (nunca)

- Não vamos hospedar modelo próprio.
- Não vamos ser um IDE.
- Não vamos permitir que o desktop fale direto com provider — nem "só em dev".

## Métricas de sucesso do MVP

| Métrica | Alvo |
|---|---|
| Chave de provider exposta no cliente | **0** (validado por teste automatizado) |
| Requisições de inferência sem checagem de política | **0** |
| Latência p95 até o primeiro token | < 1,2 s |
| Troca de provider padrão de um workspace | < 1 min, sem deploy |
| Precisão do custo reportado vs. fatura do provider | ±3% |
| Overhead do harness sobre a chamada crua ao provider | < 150 ms p50 |

## Restrições

- Time de 1 pessoa + agentes. Escopo é fatiado para caber em incrementos de 1–3 dias.
- Rodar em máquina Windows de desenvolvimento; deploy alvo em container Linux.
- Custo de LLM no desenvolvimento é orçamento real → o pipeline mede e otimiza
  (ver `docs/04-metricas/`).

## Riscos principais

| Risco | Mitigação |
|---|---|
| Escopo explodir (produto grande demais para 1 dev) | Fatiar por SPEC; V1 corta 7 features listadas acima |
| Divergência entre política declarada e aplicada | Política é um único módulo, com teste de propriedade; nenhuma rota decide sozinha |
| Custo de LLM no dev | Fase 4 mede custo/entrega e rebaixa modelo onde não degrada resultado |
| Diferença de plataforma no `sqlite-vec` | Fallback brute-force obrigatório + teste nos 3 SOs |
