# Pending Authorizations — Solicitações de Acesso Aguardando Decisão

**Formato:** append-only. Novas solicitações no final. Mover para `granted.md` (se aprovado) ou manter com `status: denied` (se negado).

---

_Nenhuma solicitação pendente ainda. O Arquiteto começa sem nenhum acesso externo._

---

## Regras

- O Arquiteto **não** prossegue com a tarefa que depende do acesso até aprovação.
- Pode continuar outras tarefas independentes na mesma missão.
- Após 7 dias sem resposta, o Arquiteto pode relembrar o proprietário (uma vez) via `ask_user`.
- Após 30 dias sem resposta, a solicitação é considerada expirada e o Arquiteto notifica o proprietário.
- Acesso negado não impede o Arquiteto de propor alternativa.
