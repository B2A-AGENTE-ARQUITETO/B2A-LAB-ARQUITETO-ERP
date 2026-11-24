# integrations/ — Scripts e Configurações de Integração

**Status:** vazio por design (v1.0.0).

## Finalidade

Pasta para scripts, configurações e adapters de integração entre o Arquiteto, o ERP e serviços externos autorizados.

## O que colocar aqui (futuro)

- Scripts de sync entre Supabase novo e ferramentas externas.
- Configurações de MCP servers (após autorização em `architect/permissions/pending.md`).
- Adapters para APIs externas (GitHub, Gemini, etc.).
- Webhooks locais para o ERP de testes.
- Scripts de migração de dados fictícios.

## Regras

1. **Nada de n8n** — diretriz §7. Substituir por ferramentas nativas do OpenBot ou MCP servers.
2. **Sem credenciais em código** — usar `process.env.VARIAVEL` ou cofre do OpenBot.
3. **Sem acesso a produção** — denylist em `architect/permissions/tools-catalog.yaml`.
4. **Cada integração deve ser autorizada** antes de ser adicionada aqui.

## Próximos passos

Pasta será populada conforme o Arquiteto identifica necessidades concretas de integração. Ver `architect/skills/tool-integration/SKILL.md` para procedimento.
