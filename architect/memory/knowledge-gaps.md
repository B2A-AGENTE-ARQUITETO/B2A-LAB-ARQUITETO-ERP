# Knowledge Gaps — Lacunas Identificadas

**Formato:** append-only. Novas lacunas no final.

## Status possíveis
- `pendente` — aguardando verificação ou consulta autorizada.
- `em_consulta` — consulta em andamento (após autorização).
- `preenchido` — lacuna resolvida (manter entrada para histórico).

---

## Lacunas sobre o ERP B2A `[DESCONHECIDO]`

Estas lacunas foram herdadas do Catálogo de Capacidades do ERP B2A (entrega anterior) e não podem ser preenchidas sem consulta autorizada ao repositório ou ambiente real:

1. **Schema-base do Supabase** — migrations locais não criam tabelas centrais. Estado real do banco é desconhecido.
2. **Estado real das policies RLS em produção** — não inferível do Git.
3. **Buckets Supabase Storage configurados** — não há referências no código.
4. **Variáveis de ambiente configuradas na Vercel** — não versionadas no Git.
5. **Configuração OAuth do n8n** — credenciais vivem no próprio n8n.
6. **Lockfile canônico** — ambiguidade npm vs Bun não resolvida no repositório.
7. **Conteúdo de `/admin/configuracoes`** — não analisado em detalhe.
8. **Conteúdo de `/debug`** — não analisado.
9. **Funcionalidades administrativas em produção** que podem não estar no código.
10. **Lista completa de webhooks n8n ativos** além do workflow financeiro.
11. **Status da configuração CSP** em `next.config.ts`.
12. **Domínio customizado** além do subdomínio Vercel.
13. **Outras integrações n8n** além do workflow financeiro (Gemini mencionado em `ARCHITECTURE.md` mas não visível).
14. **Migrations remotas do Supabase** não versionadas no Git.
