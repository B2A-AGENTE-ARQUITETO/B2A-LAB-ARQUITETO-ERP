# ERP Changes Proposed — Propostas de Alteração no ERP B2A

**Formato:** append-only. Novas propostas no final. Quando proposta é decidida, atualizar a entrada correspondente com decisão.

## Status possíveis
- `pending_approval` — proposta enviada, aguardando decisão do proprietário.
- `approved` — aprovada, aguardando implementação.
- `rejected` — rejeitada pelo proprietário.
- `needs_revision` — proprietário pediu ajustes.
- `implementing` — implementação em andamento (em fork isolado).
- `pr_open` — PR aberto, aguardando revisão.
- `merged` — merge feito pelo proprietário.
- `cancelled` — proposta cancelada pelo Arquiteto.

---

_Nenhuma proposta registrada ainda. O Arquiteto só propõe alterações no ERP após identificar oportunidade concreta (via Modo EE ou pedido do proprietário)._

---

## Princípios

Estas propostas seguem rigorosamente:

- **P1 (Independência do ERP por padrão):** o Arquiteto não acessa o ERP sem autorização específica para a tarefa.
- **P4 (Sem alteração direta em produção):** o Arquiteto nunca faz merge direto em `main`.
- **P10 (Sem auto-permissão):** o Arquiteto não concede a si próprio acesso ao ERP.

## Fluxo

1. Identificar oportunidade/melhoria (via Modo EE ou pedido do proprietário).
2. Análise técnica (consultar `knowledge/erp/`).
3. Escrever proposta formal (template abaixo).
4. Apresentar ao proprietário via `ask_user`.
5. Aguardar decisão.
6. Se aprovada: receber credenciais temporárias do fork, implementar em branch isolada, abrir PR.
7. Proprietário revisa e faz merge (ou rejeita).
8. Pós-merge: atualizar knowledge base, registrar em `memory/decisions.md`.
9. Acesso expira, Arquiteto retorna ao estado default.
