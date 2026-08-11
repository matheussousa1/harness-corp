---
description: Fatia o plano de uma SPEC em tasks verificáveis
argument-hint: SPEC-XXX
---

Fatie o plano da SPEC `$ARGUMENTS` em tasks.

Regras:
- Cada task entregável em **≤ 1 dia**.
- Cada task tem um **comando de verificação** próprio.
- Declare dependências (`depende de: T-nn`).
- Marque `[paralelizável]` só quando a task **não compartilha arquivo** com outra
  paralela. Se compartilha, ela não é paralela — corrija o fatiamento.
- Toda task precisa se ligar a pelo menos um critério de aceite (CA-nn) da SPEC.
  Task que não serve a nenhum CA está fora de escopo: remova.
- Se resultarem mais de 10 tasks, provavelmente são duas SPECs. Diga isso.

Preencha a seção **Tasks** e inicialize a tabela **Execução** com todas em `pendente`.
