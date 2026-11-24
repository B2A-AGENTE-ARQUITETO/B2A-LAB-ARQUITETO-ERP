# CONFIGURACAO-CREDENCIAIS.md

**Versão:** 1.0.0 · **Data:** 2026-11-24

## Princípios

1. **NUNCA** commite `.env` — está no `.gitignore`.
2. **NUNCA** reutilize credenciais do ERP de produção.
3. **NUNCA** exponha tokens em relatórios, logs, prompts ou documentação.
4. **NUNCA** solicite tokens no chat — use mecanismos oficiais de autenticação.
5. **SEMPRE** crie projetos/contas NOVOS para o laboratório.

## Credenciais necessárias

| # | Credencial | Onde obter | Quem cria |
|---|---|---|---|
| 1 | GitHub account (nova) | github.com | Proprietário |
| 2 | GitHub repo privado `B2A-LAB-ARQUITETO-ERP` | GitHub | Proprietário (ou Arquiteto após auth) |
| 3 | Supabase project (NOVO) | supabase.com | Proprietário |
| 4 | Supabase URL + publishable key + anon key + service_role key | Painel Supabase | Proprietário |
| 5 | Google Gemini API key | aistudio.google.com/apikey | Proprietário |
| 6 | N8N_IMPORT_API_KEY (nova string aleatória) | Gerar localmente | Proprietário (no .env) |
| 7 | OpenBot provider (Codex / Claude Code) | OpenBot Settings | Proprietário |

## Passo 1 — GitHub (nova conta + repositório privado)

### 1.1 Criar nova conta GitHub

Se o proprietário já tem uma conta GitHub separada para laboratório, pular para 1.2.

Caso contrário:
1. Acesse `https://github.com/signup`.
2. Crie conta com email dedicado ao laboratório (não o mesmo de produção).
3. Ative 2FA (Settings → Password and authentication → Two-factor authentication).
4. Anote a senha em cofre seguro (1Password, Bitwarden, etc.).

### 1.2 Criar repositório privado

**Opção A — Pelo proprietário (recomendado):**

1. Acesse `https://github.com/new`.
2. **Owner:** nova conta de laboratório.
3. **Repository name:** `B2A-LAB-ARQUITETO-ERP`.
4. **Description:** `Monorepo de desenvolvimento e testes do Agente Arquiteto + ERP B2A.`
5. **Visibility:** **Private** (obrigatório).
6. **Initialize:** NÃO marcar "Add a README" (o repo já tem um).
7. Criar.

**Opção B — Pelo Arquiteto (após autorização):**

Após o proprietário autenticar o OpenBot com a nova conta GitHub (via device flow ou `gh auth login`), o Arquiteto pode criar o repositório:

```bash
gh repo create B2A-LAB-ARQUITETO-ERP --private --description "Monorepo de desenvolvimento e testes do Agente Arquiteto + ERP B2A"
```

### 1.3 Autenticar Git local

```bash
# Opção A: GitHub CLI (recomendado)
gh auth login
# Escolher: GitHub.com → HTTPS → Login with a web browser
# Seguir instruções na tela

# Opção B: SSH (avançado)
ssh-keygen -t ed25519 -C "lab-b2a@<email>"
# Adicionar chave pública em github.com/settings/keys
```

### 1.4 Push inicial

```bash
cd B2A-LAB-ARQUITETO-ERP
git init
git add .
git commit -m "feat: monorepo inicial com Arquiteto + ERP B2A (cópia lab)"
git branch -M main
git remote add origin git@github.com:<nova-conta>/B2A-LAB-ARQUITETO-ERP.git
git push -u origin main
```

## Passo 2 — Supabase (projeto NOVO)

### 2.1 Criar projeto

1. Acesse `https://supabase.com/dashboard` com conta NOVA (não a de produção B2A).
2. Clique em **New project**.
3. **Name:** `B2A-LAB-ARQUITETO-ERP`
4. **Database Password:** gerar senha forte (anotar em cofre).
5. **Region:** a mais próxima do proprietário.
6. **Plan:** Free tier (suficiente para testes).
7. Criar e aguardar provisionamento (~2 min).

### 2.2 Obter credenciais

No painel do projeto:
1. Settings → API
2. Anotar:
   - **Project URL** → `NEXT_PUBLIC_SUPABASE_URL`
   - **Project API keys → anon public** → `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY`
   - **Project API keys → service_role** → `SUPABASE_SERVICE_ROLE_KEY` (MANTEM SECRETO)

### 2.3 Aplicar migrations

O ERP tem migrations em `erp/supabase/migrations/`. Para aplicá-las:

**Opção A — Supabase CLI (recomendado):**

```bash
# Instalar Supabase CLI
npm install -g supabase

# Login
supabase login

# Link ao projeto
cd erp
supabase link --project-ref <project-id-do-painel>

# Aplicar migrations
supabase db push
```

**Opção B — SQL Editor manual:**

1. No painel Supabase → SQL Editor
2. Para cada arquivo em `erp/supabase/migrations/` (em ordem cronológica):
   - Abrir arquivo
   - Copiar SQL
   - Colar no SQL Editor
   - Executar
3. Repetir para todas as 12 migrations.

### 2.4 Criar usuário admin

Após aplicar migrations (especialmente `20260818_single_admin_rls.sql`):

1. No painel Supabase → Authentication → Users → Add user
2. Email: `lab-admin@<dominio-teste>`
3. Senha: gerar forte (anotar em cofre)
4. Auto Confirm User: SIM
5. Criar.

O modelo single-admin da migration exige **exatamente 1 usuário** em `auth.users`. Este usuário criado será o admin do laboratório.

### 2.5 Verificar RLS

No SQL Editor:
```sql
SELECT tablename, policyname, cmd, roles, qual
FROM pg_policies
WHERE schemaname = 'public'
ORDER BY tablename, policyname;
```

Deve retornar policies `erp_admin_only` em todas as tabelas públicas.

## Passo 3 — Google Gemini API key

### 3.1 Obter chave

1. Acesse `https://aistudio.google.com/apikey` com conta Google.
2. Clique em **Create API key**.
3. Selecione projeto Google Cloud (criar novo se necessário).
4. Copie a chave gerada.

### 3.2 Configurar quota (opcional)

Para evitar custos inesperados:
1. Google Cloud Console → APIs & Services → Generative Language API
2. Quotas → Set quota limits
3. Recomendado: 1000 requests/dia para testes.

## Passo 4 — Gerar N8N_IMPORT_API_KEY

**IMPORTANTE:** O workflow n8n foi **desativado** nesta cópia (ver `RELATORIO-AUDITORIA-ERP.md`). Esta variável existe apenas para compatibilidade de código.

Gerar string aleatória forte:

```bash
openssl rand -hex 32
```

OU (sem openssl):

```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

Copiar o resultado para `.env` em `N8N_IMPORT_API_KEY`.

## Passo 5 — Preencher `.env`

```bash
cd B2A-LAB-ARQUITETO-ERP
cp .env.example .env
# Editar .env com editor de texto (NÃO commitar)
```

Preencher:

```env
NEXT_PUBLIC_SUPABASE_URL=https://<project-id>.supabase.co
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=<anon-key-do-painel>
NEXT_PUBLIC_SUPABASE_ANON_KEY=<anon-key-do-painel>
SUPABASE_SERVICE_ROLE_KEY=<service-role-key-do-painel>
GEMINI_API_KEY=<chave-do-aistudio>
N8N_IMPORT_API_KEY=<string-aleatoria-gerada>
NODE_ENV=development
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

**Nunca** commite este arquivo. Ver `.gitignore` (`.env` já está ignorado).

## Passo 6 — OpenBot provider

### 6.1 Instalar OpenBot

Se ainda não tem OpenBot:
1. Baixe em `https://github.com/nightly-labs/openbot/releases`.
2. Instale conforme seu SO.

### 6.2 Conectar provider

1. Abra OpenBot.
2. Settings → Providers.
3. Escolha **Codex** (recomendado).
4. Clique em **Download** para baixar o CLI do Codex.
5. Siga o fluxo de sign-in (via navegador, usando conta Google/ChatGPT).
6. Confirme que o provider está conectado.

Alternativa: **Claude Code** (também funciona com AGENTS.md e skills).

### 6.3 Verificar OpenBot skills

OpenBot lê skills de `.agents/skills/`. Criar symlinks:

```bash
cd B2A-LAB-ARQUITETO-ERP
mkdir -p .agents/skills
for skill in architect/skills/*/; do
  name=$(basename "$skill")
  ln -sf "../../architect/skills/$name" ".agents/skills/$name"
done
```

Para Claude Code, também criar em `.claude/skills/`:

```bash
mkdir -p .claude/skills
for skill in architect/skills/*/; do
  name=$(basename "$skill")
  ln -sf "../../architect/skills/$name" ".claude/skills/$name"
done
```

## Passo 7 — Verificação final

### 7.1 Verificar variáveis

```bash
cd B2A-LAB-ARQUITETO-ERP
# Confirmar que .env existe
test -f .env && echo "OK: .env existe" || echo "ERRO: .env não existe"

# Confirmar que .env NÃO está no Git
git status .env
# Deve retornar: nothing to commit
```

### 7.2 Testar ERP localmente

```bash
cd erp
npm install
npm run dev
# Acessar http://localhost:3000
# Login com email/senha do Passo 2.4
```

### 7.3 Testar endpoint do agente IA

Após ERP rodando:

```bash
curl -X POST http://localhost:3000/api/agent \
  -H "Content-Type: application/json" \
  -d '{"message":"Olá, confirme que você está ativo"}' \
  --cookie "<cookies-de-sessao-do-login>"
```

Deve retornar `{ "reply": "..." }` confirmando Gemini ativo.

### 7.4 Verificar denylist

O Arquiteto (após configurado) NÃO deve acessar:
- `appsite-erp-b2-a-engenharia.vercel.app`
- `n8nb2aengenharia.site`
- `supabase.co` do projeto de produção B2A

Ver `architect/permissions/tools-catalog.yaml` seção "Denylist".

## Segredo em cofre

Recomenda-se armazenar TODAS as credenciais em cofre seguro:
- 1Password
- Bitwarden
- Apple Keychain
- KeePassXC

Estrutura sugerida no cofre:

```
B2A-LAB-ARQUITETO-ERP/
├── GitHub/
│   ├── nova-conta@github.com
│   └── 2FA recovery codes
├── Supabase/
│   ├── project-id
│   ├── database-password
│   ├── anon-key
│   ├── service-role-key
│   └── admin-user@b2a-lab
├── Google/
│   ├── gemini-api-key
│   └── google-cloud-project-id
└── N8N_IMPORT_API_KEY
```

## Rotação de credenciais

Se uma credencial for comprometida (ex: commited ao Git por engano):

1. **Rotacionar imediatamente** no provedor (Supabase, Gemini, etc.).
2. **Remover** do histórico Git com `git filter-repo` ou BFG.
3. **Forçar push** (apenas em caso de comprometimento real — normalmente evitar).
4. **Registrar incidente** em `architect/memory/incidents.md`.

## Credenciais antigas (NÃO usar)

Estas credenciais do ERP original NÃO devem ser reutilizadas:

- ❌ Bearer token `n8n-b2a-secret-key-2026` (exposto no Git original — ver `RELATORIO-AUDITORIA-ERP.md`).
- ❌ Credenciais OAuth do n8n B2A (Gmail, Google Drive, Google Sheets).
- ❌ Supabase URL/keys do projeto de produção B2A.
- ❌ Vercel token do projeto de produção.
- ❌ Gemini API key usada em produção.

## Próximos passos

Após concluir toda configuração:

1. Ver `PROMPT-INICIALIZACAO-OPENBOT.md` para criar o agente no OpenBot.
2. Ver `RELATORIO-AUDITORIA-ERP.md` para entender as adaptações de segurança.
3. Ver `PENDENCIAS-INTEGRACAO.md` para o que ainda falta.
