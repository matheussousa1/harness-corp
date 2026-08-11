# Personas

## 1. Colaborador — "Ana", analista

**Contexto:** usa IA todo dia para escrever, resumir e buscar informação interna.
Hoje usa a conta pessoal dela em três ferramentas diferentes.

| | |
|---|---|
| **Quer** | Um agente que já conheça o contexto da empresa e do trabalho dela |
| **Odeia** | Reexplicar o mesmo contexto toda conversa; resposta genérica |
| **Não quer saber sobre** | Modelo, provider, token, prompt |
| **Sinal de sucesso** | Ela abre o app em vez do chat público, por vontade própria |
| **Risco** | Se o produto for mais lento ou mais burro que a ferramenta pública, ela volta — levando o dado da empresa junto |

**Implicação de produto:** latência importa tanto quanto controle. Por isso RNF de
150 ms de overhead e 1,2 s até o primeiro token.

---

## 2. Admin — "Rodrigo", gestor de operações/TI

| | |
|---|---|
| **Quer** | Saber quanto se gasta, por quem, e conseguir cortar sem abrir chamado para engenharia |
| **Odeia** | Descobrir gasto na fatura; depender de deploy para mudar uma permissão |
| **Sinal de sucesso** | Muda a política de um workspace em < 1 min, sem engenheiro |
| **Risco** | Painel complexo demais → ele não usa e a política fica default para sempre |

**Implicação:** política tem que ser aplicável sem reinício de serviço, e o painel
precisa mostrar o **efeito** da mudança, não só o formulário.

---

## 3. Segurança / Jurídico — "Carla"

| | |
|---|---|
| **Quer** | Garantia de que dado sensível não sai para provider não aprovado, e trilha para auditoria |
| **Odeia** | "Confia em mim, está seguro" sem evidência |
| **Sinal de sucesso** | Ela consegue responder sozinha "quem usou o quê, quando, com qual modelo" |
| **Risco** | Se ela não aprovar, o produto não entra na empresa — ela é o veto |

**Implicação:** as 5 invariantes não são features, são **condição de venda**. Por isso
são verificadas por CI e não por promessa.

---

## 4. Dev do harness — "você"

| | |
|---|---|
| **Quer** | Construir isso com agentes, aprendendo o método, sem estourar orçamento |
| **Odeia** | Agente em loop caro gerando código que precisa jogar fora |
| **Sinal de sucesso** | `custo_por_task` caindo ao longo das SPECs, com qualidade estável |
| **Risco** | Escopo de produto grande demais para uma pessoa → nada fica pronto |

**Implicação:** WIP de uma SPEC por vez, V1 com 7 features cortadas, e Fase 4 medindo
desde a primeira entrega.

---

## Jobs to be done

| Job | Persona | Hoje resolve com | Por que é ruim |
|---|---|---|---|
| Responder rápido usando conhecimento interno | Ana | Perguntar no chat da equipe / procurar no Drive | Lento, depende de outra pessoa |
| Não vazar dado da empresa | Carla | Política escrita, sem aplicação técnica | Não é verificável |
| Controlar gasto com IA | Rodrigo | Fatura no fim do mês | Reativo, sem atribuição |
| Trocar de provider quando o preço muda | Rodrigo | Projeto de engenharia | Lock-in na prática |
