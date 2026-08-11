# Runbooks

Procedimentos operacionais. Escritos **quando o componente existir**, não antes —
runbook de sistema inexistente é ficção.

## Planejados

| Runbook | Nasce com | Pergunta que responde |
|---|---|---|
| `provider-fora-do-ar.md` | SPEC-001 | Primário caiu — o failover cobriu? Como forçar a troca? |
| `budget-estourado.md` | SPEC-003 | Workspace bloqueado no meio do expediente. Como liberar com autorização? |
| `politica-indisponivel.md` | SPEC-002 | Fail-closed disparou e ninguém consegue usar. Diagnóstico e restauração. |
| `custo-anomalo.md` | SPEC-009 | Gasto saltou. Como achar a conversa/usuário responsável. |
| `worker-travado.md` | SPEC-005 | Turnos presos na fila. Como inspecionar checkpoint e retomar. |
| `rotacao-de-chaves.md` | SPEC-001 | Rotacionar chave de provider sem derrubar conversas em andamento. |
| `atualizacao-do-electron.md` | SPEC-005 | Rotina trimestral de segurança (custo aceito no ADR-0004). |

## Formato

Todo runbook segue: **Sintoma → Diagnóstico (comandos) → Ação → Verificação →
Prevenção**. Sem prosa. Quem lê está com o sistema quebrado.
