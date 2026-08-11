# Definition of Ready / Definition of Done

## Definition of Ready — uma SPEC pode virar código quando…

- [ ] Problema descrito sem mencionar solução
- [ ] Escopo **e não-escopo** explícitos
- [ ] Requisitos funcionais numerados (`RF-01`…)
- [ ] Requisitos não-funcionais com número (latência, limite, tamanho)
- [ ] Invariantes de segurança tocadas listadas (ou "nenhuma", justificado)
- [ ] Mudança de contrato (`openapi.yaml`) descrita, se houver
- [ ] Entidade nova ou alterada refletida em `docs/02-arquitetura/modelo-de-dominio.md`
- [ ] **Todo critério de aceite é verificável por comando ou teste**
- [ ] Plano com arquivos a criar/alterar
- [ ] Tasks fatiadas em ≤ 1 dia, com dependências marcadas
- [ ] Riscos e plano de rollback
- [ ] Aprovada pelo Coordinator (humano)

Falhou um item ⇒ volta para o Spec Writer. Não começa implementação.

## Definition of Done — uma task está pronta quando…

- [ ] Critérios de aceite da task verificados **com evidência colada** (saída real de comando)
- [ ] `just check` verde (lint + types + testes + validação de contrato)
- [ ] Testes novos cobrem o caminho feliz **e** o de falha
- [ ] Nenhuma invariante de segurança violada (gate de CI passou)
- [ ] `git diff --stat` só mostra arquivos previstos no plano
- [ ] Contrato e tipos gerados em sincronia (sem diff pendente)
- [ ] ADR criado se houve decisão arquitetural nova
- [ ] Métricas da execução registradas em `metrics/`
- [ ] Sem TODO órfão — todo TODO tem item em `backlog/`
- [ ] Verificada por um Verifier diferente de quem implementou

## Definition of Done — uma SPEC está pronta quando…

- [ ] Todas as tasks Done
- [ ] Critérios de aceite **da SPEC** (não só das tasks) verificados ponta a ponta
- [ ] Documentação afetada atualizada (README, arquitetura, runbook)
- [ ] Custo total e tempo da SPEC registrados em `metrics/`
- [ ] Retro de uma linha: o que ajustar nas rules/modelo na próxima
- [ ] Marcada como `Status: Entregue` no cabeçalho da SPEC

## Escala de severidade em review

| Nível | Significado | Ação |
|---|---|---|
| **Bloqueia** | Viola invariante de segurança, quebra contrato, sem teste | Não passa |
| **Corrige antes do merge** | Bug, caso de borda não tratado, teste frágil | Corrige |
| **Backlog** | Melhoria, dívida conhecida | Vira item, segue o merge |
| **Nit** | Estilo, preferência | Ignorável |
