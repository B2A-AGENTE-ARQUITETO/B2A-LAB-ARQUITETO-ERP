# Registry — Ferramentas Disponíveis, Integradas e Desenvolvidas

**Formato:** append-only. Novas entradas no final.

## Categorias

- `native_openbot` — ferramentas nativas do OpenBot.
- `internal` — ferramentas internas do runtime do Arquiteto.
- `mcp_verified` — MCPs externos com `status: verified`.
- `mcp_pending` — MCPs com solicitação pendente.
- `architect_developed` — ferramentas próprias desenvolvidas pelo Arquiteto.
- `deprecated` — ferramentas removidas.

---

## Native OpenBot `[VERIFICADO]`

| Tool | Função | Disponibilidade |
|---|---|---|
| `ask_user` | Perguntar ao proprietário | Sempre |
| `attach_files_to_response` | Anexar arquivos à resposta | Sempre |
| `list_agents` | Listar outros agentes no workspace | Sempre |
| `send_message` | Enviar mensagem a outro agente | Sempre |
| `search_memories` | Buscar em memórias salvas | Sempre |
| `remember` | Salvar fato em memória | Sempre |
| `forget_memory` | Remover fato da memória | Sempre |
| `react_to_user_message` | Reagir a mensagem | Sempre |
| `request_api_key` | Solicitar API key ao proprietário | Sempre |
| `routine_no_update` | Reportar ausência de atualização | Sempre |
| `routine_report_failure` | Reportar falha de rotina | Sempre |
| `history_search` | Buscar em histórico de conversa | Sempre |
| `history_read` | Ler mensagens do histórico | Sempre |

## Internal (runtime do Arquiteto)

| Tool | Status |
|---|---|
| `spec.create` | disponível (após Fase 2 do plano) |
| `spec.read` | disponível (após Fase 2) |
| `spec.write` | disponível (após Fase 2) |
| `spec.validate` | disponível (após Fase 2) |
| `tool.catalog.list` | disponível (após Fase 2) |
| `tool.request` | disponível (após Fase 2) |
| `provision.create` | disponível (após Fase 3) |
| `execute.run` | disponível (após Fase 4) |
| `test.run` | disponível (após Fase 5) |
| `test.evaluate` | disponível (após Fase 5) |
| `memory.read` | disponível (após Fase 2) |
| `memory.write` | disponível (após Fase 2) |
| `knowledge.read` | disponível (após Fase 10) |
| `knowledge.search` | disponível (após Fase 10) |
| `knowledge.consult` | disponível (após Fase 10) |
| `conversation.open` | disponível (após Fase 11) |
| `conversation.post` | disponível (após Fase 11) |
| `develop.research` | disponível (após Fase 12) |
| `develop.design` | disponível (após Fase 12) |
| `develop.implement` | disponível (após Fase 12) |
| `develop.test` | disponível (após Fase 12) |
| `develop.register` | disponível (após Fase 12) |

## MCPs Externos

_Nenhum MCP verificado ainda. Solicitações pendentes aparecem em `permissions/pending.md`._

## Ferramentas Próprias (`architect_developed`)

_Nenhuma ferramenta própria desenvolvida ainda._
