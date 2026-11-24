# Granted Authorizations — Acessos Concedidos Ativos e Expirados

**Formato:** append-only. Novas concessões no final. Quando acesso expira ou é revogado, atualizar a entrada correspondente (não remover).

## Status possíveis
- `verified` — aprovado, verificado e ativo.
- `pending_verification` — aprovado mas ainda não verificado (config em andamento).
- `expired` — janela de acesso expirou (renovação necessária).
- `revoked` — revogado antecipadamente pelo proprietário.

---

_Nenhuma autorização concedida ainda. O Arquiteto começa sem nenhum acesso externo ativo._

---

## Regras de expiração

| Tipo de acesso | Duração padrão | Renovação |
|---|---|---|
| MCP read-only | 90 dias | Re-confirmação humana obrigatória |
| MCP write | 30 dias | Re-confirmação humana obrigatória |
| Acesso ao ERP B2A (fork/branch) | Janela específica da tarefa | Expira ao fim da janela |
| API key | 90 dias | Re-confirmação ou rotação |
| OAuth token | Conforme provider | Renovação automática (pelo provider) |

## Revogação antecipada

Proprietário pode revogar a qualquer momento:

1. Atualizar entrada correspondente neste arquivo: `Status: revoked`, `Revoked at: <ISO-8601>`, `Revocation reason: <motivo>`.
2. Atualizar `permissions/tools-catalog.yaml`: `status: denied` ou remover entrada.
3. Interceptor bloqueia imediatamente todas as chamadas futuras.
4. Arquiteto notifica funcionários afetados via `send_message`.
5. Funcionários que dependiam da ferramenta têm suas instâncias suspensas até substituição ou evolução.

## Regra inegociável

O Arquiteto **nunca** concede acesso a si próprio. Toda entrada neste arquivo tem `Approved by: <proprietário>` — nunca `Approved by: agente-arquiteto`.
