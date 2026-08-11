# ADR-0005 — Backend-first e contract-first

- **Status:** Aceito
- **Data:** 2026-08-10
- **Responde à pergunta:** "começa pelo frontend ou pelo backend?"

## Contexto

Uma pergunta aberta do projeto: por onde começar a construir. As opções reais eram
UI-first (protótipo de chat bonito e depois o servidor) ou backend-first.

## Decisão

**Backend-first, contract-first.** Nesta ordem, sempre:

```
1. Contrato   packages/contracts/openapi.yaml   (+ JSON Schemas)
2. Servidor   apps/control-plane                (implementa o contrato)
3. Cliente    apps/desktop                      (consome o contrato)
```

O contrato é escrito e revisado **antes** do servidor. Cliente e servidor geram
tipos a partir dele — nenhum lado escreve o tipo do outro à mão.

## Justificativa

1. **O produto é o backend.** O valor (política, budget, multiprovider, auditoria)
   está inteiro no Control Plane. Uma UI de chat sem isso é um demo, não um produto.
2. **As invariantes de segurança são do servidor.** Fazer UI primeiro empurra decisão
   de política para o cliente "temporariamente" — e "temporário" vira permanente.
3. **Risco maior primeiro.** Streaming + reserva de budget + human-in-the-loop com
   estado durável é a parte que pode não funcionar. Descobrir isso na semana 1, não
   na semana 8.
4. **Contrato destrava paralelismo.** Com o OpenAPI fechado, o desktop pode ser
   construído contra um mock enquanto o servidor evolui — inclusive por agentes
   diferentes em paralelo.
5. **AX.** Um agente que lê o contrato não precisa adivinhar formato de payload;
   é a diferença entre gerar código certo de primeira e três rodadas de correção.

## Ressalva aceita

UI-first tem uma vantagem real: feedback visual cedo, o que ajuda motivação e
descoberta de requisito. Mitigação: a **fatia vertical V0** (`SPEC-001` + `SPEC-004`)
entrega uma janela de chat mínima já falando com o gateway real. Feio, mas ponta a
ponta, na primeira entrega. Polimento de UI vem depois, com o esqueleto seguro.

## Consequências

**Positivas**
- Nenhuma regra de política nasce no cliente.
- Tipos gerados dos dois lados ⇒ quebra de contrato aparece em compile time.
- Mock server sai de graça do OpenAPI.

**Negativas / custos aceitos**
- Primeiras semanas sem entregável visual bonito.
- Mudança de contrato exige regenerar os dois lados → disciplina de versionamento
  (`/v1`, mudança quebra-contrato só em major).

## Regra operacional

Mudou `packages/contracts/openapi.yaml`? O CI regenera os tipos e falha se houver
diferença não commitada. Contrato e código nunca divergem em silêncio.
