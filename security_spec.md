# Security Specification - ATLAS OS

## 1. Data Invariants
- **Identity Integrity**: All user-owned documents must have a `uid` field matching the `request.auth.uid`.
- **Relational Integrity**: Messages must belong to a conversation document that the user owns.
- **Type Safety**: Financial values must be numbers. Dates must be timestamps or numbers (as per app design).
- **Immutability**: `criado_em`, `uid`, and `email` fields must not be changed once set.
- **Terminal States**: Items marked as 'completed' or 'abandoned' cannot be updated by non-admins.

## 2. The Dirty Dozen Payloads (TDD)

| ID | Name | Target Path | Payload | Expected | Reason |
|---|---|---|---|---|---|
| P1 | Identity Spoofing | `/itens_focoflow/atk1` | `{"uid": "victim_id", "titulo": "Hacked", "categoria": "task"}` | DENIED | `uid` must match `auth.uid` |
| P2 | Privilege Escalation | `/Usuarios/my_id` | `{"role": "admin"}` | DENIED | Users cannot change their own roles |
| P3 | Shadow Injection | `/itens_focoflow/atk2` | `{"uid": "my_id", "titulo": "Task", "categoria": "task", "isPremium": true}` | DENIED | Schema validation blocks unknown fields |
| P4 | Value Poisoning | `/itens_focoflow/atk3` | `{"uid": "my_id", "titulo": "A" * 10000, "categoria": "task"}` | DENIED | String size limits enforced |
| P5 | Terminal Bypass | `/itens_focoflow/done1` | `{"status": "todo"}` | DENIED | Completed items are locked |
| P6 | Orphaned Write | `/conversas/victim_conv/mensagens/m1` | `{"text": "hi", "role": "user"}` | DENIED | Conversation not owned by attacker |
| P7 | System Tampering | `/transacoes_financeiras_focoflow/t1` | `{"criado_em": 9999999999999}` | DENIED | Immutable system fields |
| P8 | PII Leak | `/Usuarios/victim_id` | `GET` | DENIED | Can only read own profile |
| P9 | Admin Bypass | `/authorized_emails/hacker@evil.com` | `{"status": "active"}` | DENIED | Requires admin privileges |
| P10 | Balance Fraud | `/contas_focoflow/c1` | `{"balance": 1000000}` | DENIED | Direct balance updates restricted/validated |
| P11 | Query Scraping | `/Usuarios` | `LIST` | DENIED | Blanket user listing forbidden |
| P12 | Log Tampering | `/security_logs/log1` | `DELETE` | DENIED | Logs are append-only (no delete/update) |

## 3. Test Runner (Draft)
The `firestore.rules.test.ts` file will implement these checks.
