# erp/ — Cópia do ERP B2A Engenharia

**Status:** laboratório (não-produção) · **Origem:** `anderson110288-afk/APPSITE-ERP-B2A-Engenharia` (branch `main`, commit `0920bb4`)

## O que é

Cópia independente do código do ERP B2A Engenharia para desenvolvimento, testes, correções e evolução técnica no laboratório.

Esta cópia **não se conecta** ao ambiente de produção. Nenhuma credencial antiga é reutilizada. Nenhum workflow n8n está ativo.

## Como esta cópia foi preparada

### Pendência de cópia do código

**Importante:** no momento da entrega v1.0.0 deste monorepo, o código-fonte completo do ERP **não foi copiado fisicamente** para esta pasta porque a conta autenticada no plugin GitHub (`B2A-AGENTE-ARQUITETO`) não tem acesso ao repo original (`anderson110288-afk/APPSITE-ERP-B2A-Engenharia`).

Para completar a cópia, execute o procedimento abaixo.

### Procedimento para popular `erp/`

```bash
# 1. Clonar o repositório original (exige acesso de leitura)
git clone https://github.com/anderson110288-afk/APPSITE-ERP-B2A-Engenharia.git /tmp/erp-original

# 2. Copiar todo o conteúdo para esta pasta (preservando .git existente do monorepo)
cp -r /tmp/erp-original/* B2A-LAB-ARQUITETO-ERP/erp/
cp -r /tmp/erp-original/.github B2A-LAB-ARQUITETO-ERP/erp/ 2>/dev/null || true
# NÃO copiar .git (preserva o .git do monorepo)
# NÃO copiar .kilocode (legacy)

# 3. Limpar
rm -rf /tmp/erp-original

# 4. Aplicar adaptações do laboratório (ver INTEGRACAO-LAB.md)
#    - Remover bun.lock
#    - Sanitizar workflow_n8n_financeiro.json
#    - Atualizar .gitignore
```

### Após popular

1. **Verificar integridade:**
   ```bash
   cd erp
   npm install  # usa package-lock.json
   npm run typecheck
   npm run build
   ```

2. **Confirmar adaptações do laboratório** em `INTEGRACAO-LAB.md`.

3. **Atualizar `RELATORIO-AUDITORIA-ERP.md`** com a confirmação de que a cópia foi populada.

## Estrutura esperada após cópia

```
erp/
├── .github/
├── AGENTS.md                          # Instruções do ERP original (preservadas)
├── ARCHITECTURE.md
├── CLAUDE.md
├── ENVIRONMENT.md
├── ESTRUTURA_PADRAO_DESENVOLVIMENTO.md
├── GEMINI.md
├── OPERATIONS.md
├── SECURITY.md
├── bun.lock                           # REMOVER (ver INTEGRACAO-LAB.md)
├── docs/
├── eslint.config.mjs
├── financial_agent_prompt.md
├── instrucoes_n8n.ps1                 # Sanitizar (ver INTEGRACAO-LAB.md)
├── next.config.ts
├── package-lock.json
├── package.json
├── plano_integracao_cronograma_financeiro.md
├── postcss.config.mjs
├── skills-lock.json
├── src/
│   ├── app/
│   ├── components/
│   ├── contexts/
│   ├── features/
│   ├── lib/
│   ├── middleware.ts
│   └── utils/
├── supabase/migrations/
├── tsconfig.json
├── vercel.json
├── wf_healthy_template.json           # n8n DESATIVADO (preservar como referência)
├── wf_healthy_template_v2.json        # n8n DESATIVADO (preservar como referência)
├── workflow_n8n_financeiro.json       # n8n DESATIVADO + SANITIZADO (ver INTEGRACAO-LAB.md)
└── workflow_n8n_supabase_financeiro.json  # n8n DESATIVADO (preservar como referência)
```

## Pré-requisitos

- **Node.js 22+** (validado em 24.19.0)
- **npm** como gerenciador de pacotes canônico (`bun.lock` removido nesta cópia)
- **Variáveis de ambiente** preenchidas no `.env` da raiz do monorepo (ver `../CONFIGURACAO-CREDENCIAIS.md`)

## Rodar localmente

```bash
cd erp
npm install
npm run dev
# Acessar http://localhost:3000
```

## Verificações estáticas

```bash
cd erp
npm run typecheck    # Verificação de tipos TypeScript
npm run build        # Build de produção (valida compilação)
# npm run lint       # Pode falhar por incompatibilidade ESLint 10 vs eslint-config-next
```

## Login

Após aplicar migrations no Supabase de testes (ver `../CONFIGURACAO-CREDENCIAIS.md` Passo 2):

1. Acesse `http://localhost:3000/login`.
2. Entre com email/senha do usuário admin criado no Supabase de testes.
3. Deve redirecionar para `/dashboard`.

## Modelo de segurança

Esta cópia mantém o modelo **single-admin RLS** do ERP original (migration `20260818_single_admin_rls.sql`). Para criar usuários adicionais, é necessário primeiro substituir o modelo por papéis/escopo — fora do escopo desta versão.

## Diferenças em relação ao ERP original

| Aspecto | Original | Esta cópia |
|---|---|---|
| n8n workflows | Ativos | **Desativados** (preservados como referência) |
| `bun.lock` | Presente | **Removido** (npm canônico) |
| Bearer token n8n | `n8n-b2a-secret-key-2026` exposto no JSON | **Sanitizado** — placeholder `<ROTATED-SEE-ENV>` |
| URL de webhook no workflow | `appsite-erp-b2-a-engenharia.vercel.app` | **Sanitizado** — placeholder `<LAB-URL-SEE-ENV>` |
| Variáveis de ambiente | Configuradas na Vercel | `.env` local (não commited) |

Ver `INTEGRACAO-LAB.md` para detalhes completos das adaptações.

## Não fazer nesta cópia

- ❌ Não fazer push para o repositório original.
- ❌ Não usar credenciais de produção.
- ❌ Não configurar workflows n8n ativos.
- ❌ Não conectar ao Supabase de produção B2A.
- ❌ Não fazer deploy Vercel usando domínio de produção.
- ❌ Não executar migrações destrutivas sem backup.

## Suporte

Para dúvidas sobre o ERP original (regras de negócio, módulos, etc.), consulte `../architect/knowledge/erp/INDEX.md`.

Para dúvidas sobre as adaptações do laboratório, consulte `INTEGRACAO-LAB.md`.

Para pendências de configuração, consulte `../PENDENCIAS-INTEGRACAO.md`.
