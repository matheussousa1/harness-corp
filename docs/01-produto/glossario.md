# Glossário

Vocabulário único do projeto. Se um termo não está aqui, não use em SPEC.

| Termo | Definição |
|---|---|
| **Harness** | A camada em volta do modelo: system prompt, rules, memória, skills, subagentes, tools e processos. É o produto — o modelo é commodity. |
| **Control Plane** | Serviço central da empresa (`apps/control-plane`). Único componente que conhece as API keys de provider e resolve políticas. |
| **Desktop** | App Electron do colaborador (`apps/desktop`). Cliente burro em termos de política: só renderiza o que o Control Plane autoriza. |
| **Provider** | Fornecedor de LLM (Anthropic, OpenAI, Google, Z.ai/GLM…). |
| **Workspace** | Unidade de agrupamento e de política. Um colaborador pode pertencer a vários. |
| **Política (Policy)** | Conjunto de permissões: modelos habilitados, tools/MCP habilitados, limites, modo de aprovação. |
| **Política em cascata** | Resolução por precedência: **Workspace > default pessoal > nenhuma política**. "Nenhuma" resolve para negar. |
| **Budget** | Teto de consumo (em tokens ou R$) por período, atribuível a org, workspace, time ou usuário. |
| **Reserva de budget** | Débito otimista feito **antes** do turno, com estimativa; ajustado no `settle()` após o uso real. |
| **Turno (turn)** | Um ciclo completo usuário→agente→resposta final, possivelmente com N chamadas de LLM e tools dentro. |
| **Fail-closed** | Diante de política ausente, corrompida ou host offline, o sistema **nega**. Nunca assume permissão. |
| **Aprovação por conversa** | Configuração por conversa/workspace que define se ações de tool são auto-aprovadas ou exigem confirmação humana. |
| **Memória semântica local** | Vetores do trabalho do usuário, em SQLite + `sqlite-vec`, na máquina dele. Nunca sobe. |
| **KB corporativa / RAG** | Base de conhecimento da empresa, server-side, com resultado filtrado pela permissão do usuário antes de entrar no contexto. |
| **MCP** | Model Context Protocol. Como o agente descobre e chama ferramentas externas. |
| **Skill** | Instrução empacotada e reutilizável que o agente carrega sob demanda. |
| **Subagente** | Agente com escopo e ferramentas próprias, invocado por um agente coordenador. |
| **SPEC** | Contrato executável de uma feature (`specs/SPEC-XXX-*.md`): objetivo, restrições, critérios de aceite. |
| **Gate** | Verificação automatizada que bloqueia progresso (lint, types, testes, contrato, invariantes de segurança). |
| **AX (Agent Experience)** | Qualidade do repositório do ponto de vista de um agente: previsibilidade, contratos explícitos, verificação de um comando. |
| **Camada 1/2/3/4** | Cérebro (LLM) / Harness proprietário / Processos / Métricas. Ver `README.md`. |
