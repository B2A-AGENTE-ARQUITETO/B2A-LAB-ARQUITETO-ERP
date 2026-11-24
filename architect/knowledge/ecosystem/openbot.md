# OpenBot — Ambiente de Execução

**Última verificação:** 2026-11-24 · **Fonte:** `https://github.com/nightly-labs/openbot` (repo público, v0.72.0)

## O que é

OpenBot é um **workspace desktop local-first para agentes de IA persistentes**. É um aplicativo Electron (TypeScript) que roda em macOS 13+, Windows 10+ e Linux (AppImage x64/arm64). Licença PolyForm Noncommercial 1.0.0.

Não é um framework de agentes nem um runtime de LLM. É um **workspace** que dá aos runtimes dos providers um lugar para trabalhar com persistência, conversas, arquivos, ferramentas e mensagens entre agentes.

## Capacidades nativas verificadas

### Providers suportados
- Codex App Server (OpenAI)
- Claude Code (Anthropic)
- Grok CLI (xAI)
- OpenCode
- Gemini (Google Antigravity CLI)
- Cursor CLI
- Cline CLI
- Kimi Code CLI
- Devin CLI
- GitHub Copilot CLI
- Mistral Vibe
- Hermes Agent (via ACP)

### Estrutura de workspace
- Cada agente tem workspace próprio em `~/OpenBot/Agents/<agent-id>/`.
- Pasta compartilhada entre agentes: `~/OpenBot/Shared/`.
- Banco de dados local: `openbot.db` (SQLite).
- Conversas salvas por agente, persistem entre reinicializações.

### Skills (Agent Skills specification)
- **Formato:** pasta com `SKILL.md` (YAML frontmatter + markdown).
- **Frontmatter obrigatório:** `name` (string, deve bater com nome da pasta) e `description` (1–1024 chars).
- **Campos opcionais:** `example-prompt`, scripts, references, assets.
- **Ícone:** `assets/icon.png` (preview local).
- **Localizações de skills:**
  - `/.agents/skills/` — lido por Codex, Grok, OpenCode, Gemini, Cursor, Cline, Pi, custom agents.
  - `/.claude/skills/` — lido por Claude Code, OpenCode, Cursor.
  - `~/.agents/skills/` — user-level, lido por Codex, Grok, OpenCode, Cursor, Cline, Pi.
  - `~/.claude/skills/` — user-level, lido por Claude Code, OpenCode, Grok, Cursor.
- OpenBot escreve skills em **ambos** `.agents/skills/` e `.claude/skills/` porque Claude Code não lê `.agents/skills`.
- `.openbot/skills-lock.json` registra hashes dos arquivos de skill.
- `.openbot/skills-disabled/` guarda skills desativadas.

### Instruções de agente
- **Formato:** `AGENTS.md` (formato aberto da Linux Foundation).
- **Hierarquia:** para monorepos, `AGENTS.md` aninhados em subprojetos. O arquivo mais próximo do editado tem precedência.
- **Sem campos obrigatórios** — é markdown padrão.
- Seções populares: overview, build commands, code style, testing, security, commit guidelines.
- **Hermes** lê `AGENTS.md` apenas em workspaces sem `.hermes.md`.

### Ferramentas nativas do OpenBot (confirmadas em `docs/architecture/providers.md`)
- `ask_user` — perguntar ao usuário.
- `attach_files_to_response` — anexar arquivos à resposta.
- `list_agents` — listar outros agentes no workspace.
- `send_message` — enviar mensagem a outro agente.
- `search_memories` — buscar em memórias salvas.
- `remember` — salvar fato em memória.
- `forget_memory` — remover fato da memória.
- `react_to_user_message` — reagir a mensagem do usuário.
- `request_api_key` — solicitar API key ao proprietário.
- `routine_no_update` — reportar ausência de atualização em rotina.
- `routine_report_failure` — reportar falha de rotina.
- `history_search` — buscar em histórico de conversa (default 5 resultados, max 10).
- `history_read` — ler mensagens do histórico (recentes, nomeadas, ou paginadas).

### Permissões
3 níveis por agente (em Settings):
- **Workspace only:** mudanças de arquivo limitadas ao workspace do agente, shared folder, temp e pastas explicitamente permitidas. **Não bloqueia leituras fora do workspace nem acesso à rede.**
- **Ask first:** mantém approval requests para OpenBot tools, site publishing e requests do provider.
- **Full access:** age sem approvals rotineiros; pode mudar qualquer arquivo que o OS account alcança. Computer Use e browser não são limitados ao workspace.

Computer Use tem switch próprio separado das permissões de arquivo.

### Persistência
- Banco SQLite local (`openbot.db`) é source of truth.
- Conversas, anexos, browser data e team data ficam no computador que roda OpenBot.
- Migrations são irreversíveis (sem backup automático antes de upgrade).
- Sessões de provider idle são descarregadas mas identidade é preservada (Claude fecha processo idle; Codex libera sessão 5 min após último turn; ACP tem handling especial).

### Browser embutido
- Browser persistente embutido que agentes podem controlar.
- Computer Use disponível em todas as plataformas (requer permissões adicionais no macOS).

### Mensagens entre agentes
- Agent-to-agent messaging, replies, reactions, images, file transfers.
- Delegação de tarefas entre agentes.

### Skills learning
- Background memory review após 20 turns completados ou 15 min sem turns.
- 3 modos: Automatic (default, salva sem perguntar), Ask first, Off.
- Quando em Automatic, agentes podem corrigir skills locais durante turns.

## Limitações confirmadas

- Agents rodam com `danger-full-access` e `approvalPolicy: never` no nível do provider — o usuário é avisado para rodar apenas agentes que confia.
- Sem per-action confirmations após consentimento inicial.
- Não há isolamento forte entre agentes no nível do OS (compartilham filesystem do usuário).
- Migrations de banco são irreversíveis.
- Sem built-in rate limiting por agente (limites vêm do provider).
- Sem suporte nativo a MCP servers (cada provider tem sua config: Claude `.mcp.json`, Codex `config.toml`, etc.).

## O que NÃO é

- Não é um SaaS — é desktop app local.
- Não é um runtime de LLM — usa runtimes dos providers.
- Não é um framework de agentes — não define como agents são estruturados.
- Não é um orquestrador — não distribui trabalho entre agentes automaticamente.
- Não é um marketplace — embora tenha plugin catalog em `openbot.run/plugins/`.

## Como o Agente Arquiteto usa o OpenBot

### Workspace do Arquiteto
`~/OpenBot/Agents/agente-arquiteto/` contém:
- `AGENTS.md` — instruções principais.
- `knowledge/` — base de conhecimento (erp, ecosystem).
- `skills/` — skills operacionais do Arquiteto.
- `templates/` — template de funcionário digital.
- `memory/` — arquivos de memória persistente.
- `permissions/` — catálogo de ferramentas, acessos pending/granted.
- `tests/` — testes de aceitação do Arquiteto.
- `assets/` — ícone, recursos visuais.

### Skills que o Arquiteto usa
- Suas próprias skills em `skills/` (6 skills operacionais).
- Skills nativas do OpenBot (`skill-creator`, `skill-installer`) se necessário.

### Ferramentas que o Arquiteto usa
- Ferramentas nativas do OpenBot (`ask_user`, `remember`, `search_memories`, `send_message`, `list_agents`, etc.).
- Skills carregadas como instruções especializadas.
- MCPs externos **apenas** quando autorizado via `permissions/pending.md` → `permissions/granted.md`.

### Persistência que o Arquiteto usa
- `memory/` (arquivos markdown) — para decisões, lacunas, divergências, incidentes, etc.
- `remember`/`search_memories` nativos do OpenBot — para fatos recuperáveis em conversa.
- `history_search`/`history_read` — para recuperar contexto de conversas anteriores.

## Lacunas de conhecimento sobre o OpenBot `[DESCONHECIDO]`

- Conteúdo de `docs/architecture/agents.md` (não lido em detalhe).
- Conteúdo de `docs/architecture/skills-and-plugins.md` (parcialmente lido).
- Mecanismo exato de `routine flows between agents`.
- API de plugins (além do catálogo).
- Mecanismo de `computer use` em detalhe.
- Comportamento exato de context compaction.

Estas lacunas podem ser preenchidas consultando o repositório oficial `https://github.com/nightly-labs/openbot` sob autorização do proprietário.
