# User Stories

Formato: `Como <persona>, quero <ação>, para <resultado>.`
Toda story tem critérios de aceite em Gherkin. Story sem critério verificável não
entra em SPEC.

Personas: **Colaborador**, **Admin**, **Segurança**, **Dev do harness**.

---

## E0 — Fundação e gates

### US-001 · Gate impede chave no cliente
**Como** Segurança, **quero** que o build quebre se uma chave de provider aparecer no
app desktop, **para** que a invariante não dependa de disciplina humana.

```gherkin
Dado um arquivo em apps/desktop/src contendo "sk-ant-api03-xxxx"
Quando o CI executar o job security-invariants
Então o job falha
E a saída aponta arquivo e linha
```

### US-002 · Verificação de um comando
**Como** Dev do harness, **quero** um único `just check`, **para** que humano e agente
verifiquem o trabalho do mesmo jeito.

```gherkin
Dado o repositório limpo
Quando eu executar "just check"
Então o exit code é 0
E a execução leva menos de 90 segundos
```

---

## E1 — Gateway de inferência

### US-010 · Conversar sem chave local
**Como** Colaborador, **quero** conversar com o modelo sem ter nenhuma chave na minha
máquina, **para** que eu não seja responsável por um segredo da empresa.

```gherkin
Dado que estou autenticado com meu token de sessão
Quando eu enviar uma mensagem para /v1/chat
Então recebo a resposta em streaming
E nenhuma credencial de provider existe na minha máquina
```

### US-011 · Trocar de provider sem deploy
**Como** Admin, **quero** apontar o modelo lógico "fast-general" para outro provider,
**para** reagir a preço, incidente ou qualidade sem tocar no cliente.

```gherkin
Dado que "fast-general" aponta para o Provider A
Quando eu alterar o registro de modelos para o Provider B
Então a próxima conversa usa o Provider B
E nenhum app de colaborador precisa ser atualizado
```

### US-012 · Continuar funcionando com provider fora do ar
**Como** Colaborador, **quero** que minha conversa continue se o provider primário
falhar, **para** não perder trabalho por incidente de terceiro.

```gherkin
Dado que o provider primário retorna 429
Quando eu enviar uma mensagem
Então o gateway usa o próximo provider da cadeia
E o failover fica registrado na auditoria
```

---

## E2 — Política e budget

### US-020 · Ver só o que posso usar
**Como** Colaborador, **quero** ver apenas os modelos e ferramentas que o meu
workspace liberou, **para** não perder tempo com opção que vai ser negada.

```gherkin
Dado que meu workspace habilita apenas "fast-general"
Quando eu abrir o seletor de modelos
Então vejo somente "fast-general"
E uma requisição forjada para outro modelo retorna 403 policy_denied
```

### US-021 · Política pessoal não amplia a do workspace
**Como** Segurança, **quero** que o default pessoal nunca conceda além do workspace,
**para** que a permissão fique centralizada.

```gherkin
Dado um workspace que habilita apenas o modelo X
E um default pessoal que habilita X e Y
Quando a política efetiva for resolvida
Então o resultado contém apenas X
```

### US-022 · Bloqueio quando a política não carrega
**Como** Segurança, **quero** que falha de política bloqueie o uso, **para** que
indisponibilidade nunca vire permissão.

```gherkin
Dado que o cache de política está corrompido
Quando um colaborador enviar uma mensagem
Então a requisição é negada com policy_unavailable
E o evento é registrado na auditoria
```

### US-023 · Teto de gasto respeitado
**Como** Admin, **quero** um teto mensal por workspace, **para** que o custo de IA
seja previsível.

```gherkin
Dado um workspace com budget mensal de R$ 100
E R$ 100 já consumidos
Quando um colaborador enviar uma mensagem
Então recebe 402 budget_exceeded
E nenhuma chamada a provider é feita
```

### US-024 · Aviso antes do corte
**Como** Admin, **quero** ser avisado em 80% do budget, **para** decidir antes de
interromper o time.

```gherkin
Dado um workspace que ultrapassa 80% do budget
Quando o consumo for registrado
Então um alerta é emitido uma única vez no período
```

---

## E3 — Painel admin

### US-030 · Configurar workspace
**Como** Admin, **quero** definir modelos, tools, budget e modo de aprovação de um
workspace, **para** aplicar a política da empresa sem envolver engenharia.

```gherkin
Dado que estou autenticado como admin
Quando eu salvar a política do workspace "acme"
Então a política vale para a próxima requisição, sem reinício de serviço
E a alteração fica na auditoria com autor e horário
```

### US-031 · Ver onde o dinheiro foi
**Como** Admin, **quero** custo por workspace, usuário e conversa, **para** justificar
o investimento e cortar desperdício.

```gherkin
Quando eu abrir o relatório de custos do mês
Então vejo custo por workspace, usuário e conversa
E o total diverge menos de 3% da fatura dos providers
```

---

## E4 — App desktop

### US-040 · Chat nativo no desktop
**Como** Colaborador, **quero** um app no meu computador, **para** usar o agente sem
depender do navegador e com acesso às minhas ferramentas locais.

```gherkin
Dado o app instalado e eu autenticado
Quando eu enviar uma mensagem
Então a resposta aparece token a token
E o app nunca abre conexão com um host de provider
```

### US-041 · App endurecido
**Como** Segurança, **quero** o Electron com isolamento total, **para** reduzir a
superfície de ataque na máquina do colaborador.

```gherkin
Dado o app empacotado
Quando as configurações da janela forem inspecionadas
Então contextIsolation=true, nodeIntegration=false e sandbox=true
E o renderer não tem acesso direto a fs ou rede
```

---

## E5 — Memória local

### US-050 · Agente lembra do meu trabalho
**Como** Colaborador, **quero** que o agente recupere contexto do que venho fazendo,
**para** não repetir explicação a cada conversa.

```gherkin
Dado que conversei sobre o projeto "Fênix" na semana passada
Quando eu perguntar "como estava o Fênix?"
Então trechos relevantes da memória local entram no contexto
E nenhum conteúdo da memória é enviado a um servidor exceto para gerar embedding
```

### US-051 · Funciona mesmo sem a extensão vetorial
**Como** Colaborador, **quero** que a memória funcione mesmo se `sqlite-vec` não
carregar, **para** não ficar sem o recurso por causa da minha plataforma.

```gherkin
Dado que a extensão sqlite-vec não carrega
Quando eu fizer uma busca na memória
Então o resultado é retornado pelo modo brute-force
E o app registra um aviso de degradação de performance
```

---

## E6 — KB corporativa

### US-060 · Respostas com o conhecimento da empresa
**Como** Colaborador, **quero** que o agente responda usando a documentação interna,
**para** obter resposta certa em vez de genérica.

```gherkin
Dado que a KB tem a política de reembolso
Quando eu perguntar sobre reembolso
Então a resposta cita o documento com link
```

### US-061 · RAG não vaza entre setores
**Como** Segurança, **quero** que a busca só alcance o que o usuário pode ver, **para**
que o RAG não vire canal de vazamento.

```gherkin
Dado um documento restrito ao setor Financeiro
Quando um usuário de outro setor fizer uma pergunta relacionada
Então nenhum trecho desse documento entra no contexto do modelo
E o filtro é aplicado na consulta, não após a busca
```

---

## E7 — Tools via MCP e aprovação

### US-070 · Ferramentas sob política
**Como** Admin, **quero** habilitar servidores MCP por workspace, **para** controlar o
que o agente consegue fazer.

```gherkin
Dado que o workspace não habilita o servidor MCP "email"
Quando o agente montar a lista de ferramentas
Então "email" não é exposto ao modelo
```

### US-071 · Confirmação antes de agir
**Como** Colaborador, **quero** confirmar ações com efeito externo, **para** não ser
surpreendido pelo agente.

```gherkin
Dado o modo de aprovação "confirm"
Quando o agente decidir chamar uma ferramenta de escrita
Então a execução pausa e me pede confirmação
E o estado da conversa sobrevive a um reinício do servidor
E a decisão é gravada no servidor, não no cliente
```

---

## E8 — Observabilidade

### US-080 · Rastrear uma conversa cara
**Como** Dev do harness, **quero** o traço completo de uma conversa com custo por
passo, **para** saber onde otimizar.

```gherkin
Dado um request_id
Quando eu abrir o traço
Então vejo cada nó do grafo, cada chamada de tool, tokens e custo por passo
```

---

## Backlog não priorizado (pós-V1)

| ID | Story | Épico futuro |
|---|---|---|
| US-090 | Transcrição de reuniões | E9 |
| US-091 | Crons e automações agendadas | E10 |
| US-092 | Kanban de tarefas do agente | E11 |
| US-093 | Conector ClickUp dedicado | E12 |
| US-094 | Conector Google Drive | E12 |
| US-095 | Sincronização criptografada da memória entre máquinas | E5 |
| US-096 | SSO/SAML corporativo | E13 |
| US-097 | Cache semântico no gateway | E1 |
| US-098 | Roteamento por custo (modelo mais barato que atende) | E1 |
| US-099 | `docs/03-processo/como-escrever-testes.md` — convenções de teste dos dois lados, escrito **depois** de SPEC-000 T-02/T-03, quando houver testes reais para descrever. Referência única, nunca repetida nos tutoriais de `validacao/` | E0 |
