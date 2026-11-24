# INTEGRACAO-LAB.md — Adaptações do Laboratório

**Versão:** 1.0.0 · **Data:** 2026-11-24

## Sumário

Adaptações aplicadas (ou a aplicar) à cópia do ERP B2A neste monorepo para torná-la segura e independente do ambiente de produção.

## Status das adaptações

| # | Adaptação | Status | Aplicada por |
|---|---|---|---|
| 1 | Remover `bun.lock` | Pendente | Proprietário ao popular `erp/` |
| 2 | Sanitizar `workflow_n8n_financeiro.json` (bearer token) | Pendente | Proprietário ao popular `erp/` |
| 3 | Sanitizar `instrucoes_n8n.ps1` (URL/key) | Pendente | Proprietário ao popular `erp/` |
| 4 | Atualizar `.gitignore` do ERP | Pendente | Proprietário ao popular `erp/` |
| 5 | Remover `.kilocode/` (legacy) | Pendente | Proprietário ao popular `erp/` |
| 6 | Verificar histórico Git por segredos | Pendente | Proprietário ao popular `erp/` |
| 7 | Configurar `.env` com credenciais novas | Pendente | Proprietário (ver `../CONFIGURACAO-CREDENCIAIS.md`) |
| 8 | Aplicar migrations no Supabase novo | Pendente | Proprietário (ver `../CONFIGURACAO-CREDENCIAIS.md`) |

**Notar:** as adaptações 1–6 só podem ser aplicadas após a cópia do código original para `erp/`. Ver `README.md` desta pasta para o procedimento.

## Detalhamento das adaptações

### 1. Remover `bun.lock`

**Por quê:** o ERP original tem dois lockfiles (`package-lock.json` para npm e `bun.lock` para Bun). Esta ambiguidade foi identificada como débito técnico no `RELATORIO-AUDITORIA-ERP.md`. Para o laboratório, padronizamos em **npm** (compatibilidade Next.js garantida).

**Como aplicar:**
```bash
cd erp
git rm bun.lock
git commit -m "chore(erp): remove bun.lock — npm é canônico no laboratório"
```

### 2. Sanitizar `workflow_n8n_financeiro.json`

**Por quê:** o arquivo original contém o bearer token `n8n-b2a-secret-key-2026` em texto plano no JSON versionado (lacuna de segurança crítica documentada no Catálogo de Capacidades do ERP B2A, seção 7.1, item B2).

**Como aplicar:**

Substituir no arquivo `workflow_n8n_financeiro.json` o trecho:
```json
{
  "name": "Authorization",
  "value": "Bearer n8n-b2a-secret-key-2026"
}
```

Por:
```json
{
  "name": "Authorization",
  "value": "Bearer <ROTATED-N8N-IMPORT-API-KEY-SEE-ENV>"
}
```

E substituir:
```json
"url": "https://appsite-erp-b2-a-engenharia.vercel.app/api/financeiro/importar"
```

Por:
```json
"url": "<LAB-URL-SEE-ENV>/api/financeiro/importar"
```

**Notar:** o workflow n8n está **desativado** no laboratório (ver item 7 abaixo). Esta sanitização é para segurança do código versionado, não para reativar o workflow.

### 3. Sanitizar `instrucoes_n8n.ps1`

**Por quê:** o script PowerShell original contém a URL do ERP de produção e o bearer token em texto plano:
```powershell
$n8n_url = "https://n8nb2aengenharia.site"
$erp_url = "https://appsite-erp-b2-a-engenharia.vercel.app"
$api_key = "n8n-b2a-secret-key-2026"
```

**Como aplicar:**

Substituir o conteúdo de `instrucoes_n8n.ps1` por:
```powershell
# Script DEPRECEADO — n8n desativado no laboratório B2A-LAB-ARQUITETO-ERP
# Mantido apenas como referência histórica da configuração original.
#
# Para integracao financeira no laboratorio, desenvolver novo mecanismo
# usando ferramentas nativas do OpenBot (ver ../architect/skills/tool-integration/SKILL.md).

Write-Host "Este script esta desativado no laboratorio." -ForegroundColor Yellow
Write-Host "n8n nao e mais utilizado. Ver INTEGRACAO-LAB.md para detalhes." -ForegroundColor Yellow
```

### 4. Atualizar `.gitignore` do ERP

**Por quê:** o `.gitignore` original do ERP cobre dependências e artefatos, mas não cobre `.env` explicitamente (dependia do `.gitignore` global). No laboratório, garantir que `.env` nunca seja commitado.

**Como aplicar:**

Adicionar ao final de `erp/.gitignore` (se já não existir):
```gitignore
# Environment variables (lab)
.env
.env.local
.env.*.local
!.env.example

# Vercel (lab)
.vercel/

# OpenBot state (lab)
state/
instances/
```

OU substituir o `.gitignore` do ERP pelo do monorepo (raiz). Recomendado o segundo para consistência.

### 5. Remover `.kilocode/`

**Por quê:** o diretório `.kilocode/rules/memory-bank/` contém template inicial (legacy) que não reflete o ERP atual. Documentação do ERP confirma: "`.kilocode/rules/memory-bank/{brief,architecture,tech}.md` descreve um template inicial e não o ERP atual" (`ARCHITECTURE.md` do ERP original).

**Como aplicar:**
```bash
cd erp
git rm -rf .kilocode
git commit -m "chore(erp): remove .kilocode legacy template"
```

### 6. Verificar histórico Git por segredos

**Por quê:** mesmo após sanitizar os arquivos atuais, o histórico Git pode conter credenciais antigas (em commits anteriores).

**Como aplicar:**

Após popular `erp/` (e antes do primeiro push para o GitHub privado):

```bash
# Auditar histórico por padrões de secret
cd B2A-LAB-ARQUITETO-ERP

# Opção A: usar git-secrets (recomendado)
# Instalar: https://github.com/awslabs/git-secrets
git secrets --install
git secrets --scan-history

# Opção B: usar trufflehog
# Instalar: https://github.com/trufflesecurity/trufflehog
trufflehog git file://. --since-commit HEAD~100

# Opção C: grep manual (simplificado)
git log -p | grep -iE "(api[_-]?key|token|password|secret|bearer)" | head -50
```

Se encontrar segredos no histórico:

1. **NÃO** fazer push para o GitHub privado ainda.
2. Decidir entre:
   - **Squash histórico:** reescrever com `git filter-repo` ou BFG.
   - **Início limpo:** descartar histórico antigo e commitar apenas estado atual.
3. Após reescrever, fazer push.

**Recomendação:** como este é um repo novo de laboratório, recomenda-se **início limpo** — commitar apenas o estado atual sem histórico antigo. Isso elimina qualquer risco de segredos antigos.

```bash
# Início limpo (descartar histórico do ERP original)
cd B2A-LAB-ARQUITETO-ERP/erp
rm -rf .git
cd ..
git add erp/
git commit -m "feat(erp): copia inicial do ERP B2A para laboratório (sem histórico antigo)"
```

### 7. n8n desativado — detalhamento

**Decisão:** o n8n **não é utilizado** no laboratório. Ver diretriz executiva seção 7.

**Implementação:**

- Workflows JSON (`workflow_n8n_financeiro.json`, `wf_healthy_template.json`, `wf_healthy_template_v2.json`, `workflow_n8n_supabase_financeiro.json`) são **preservados como referência histórica** mas **não são ativos**.
- Nenhuma integração OAuth (Gmail, Google Drive, Google Sheets) é configurada.
- O endpoint `/api/financeiro/importar` permanece no código (não removido — seria mudança de código), mas **nenhum webhook ativo** o chama.
- A variável `N8N_IMPORT_API_KEY` em `.env` é mantida para compatibilidade de código, mas deve ser **string aleatória nova**, não o token antigo.

**Futuro:** se for necessário reativar automação financeira, desenvolver novo mecanismo usando:
- Ferramentas nativas do OpenBot (browser embutido, mensagens entre agentes).
- MCP server para OCR de documentos (ver `../architect/skills/tool-integration/SKILL.md`).
- Cron job local (em vez de n8n externo).

Esta decisão fica registrada em `../architect/memory/decisions.md`.

### 8. Supabase — migrações

**Procedimento completo:** ver `../CONFIGURACAO-CREDENCIAIS.md` Passo 2.

**Resumo:**
1. Criar projeto NOVO no Supabase (conta nova, não a de produção B2A).
2. Obter URL + anon key + service_role key.
3. Aplicar as 12 migration de `erp/supabase/migrations/` em ordem cronológica.
4. Criar 1 usuário admin em Authentication → Users.
5. Verificar RLS com query em `pg_policies`.

**Pendência conhecida:** o ERP original tem `schema-base` não versionado (ver `../architect/knowledge/erp/09-limitacoes-problemas.md` B3). Algumas tabelas centrais (obras, pendencias, contratos_empreiteiros, etc.) podem não ter migrations locais — suas estruturas estão apenas no Supabase remoto de produção.

**Solução:** após criar o Supabase novo, antes de aplicar migrations locais, exportar o schema completo do Supabase de produção (apenas schema, sem dados) e aplicar no novo. Procedimento:

```bash
# No Supabase de produção (apenas schema, sem dados)
pg_dump --schema-only --no-owner <prod-supabase-url> > schema-base.sql

# No Supabase novo de testes
psql <new-supabase-url> < schema-base.sql

# Depois aplicar migrations locais
cd erp
supabase db push
```

**Atenção:** este procedimento exige acesso ao Supabase de produção. Solicitar ao proprietário.

## Validação pós-aplicação

Após aplicar todas as adaptações:

```bash
cd B2A-LAB-ARQUITETO-ERP

# 1. Confirmar que bun.lock foi removido
test ! -f erp/bun.lock && echo "OK: bun.lock removido" || echo "ERRO: bun.lock ainda existe"

# 2. Confirmar que workflow_n8n_financeiro.json foi sanitizado
grep -q "n8n-b2a-secret-key-2026" erp/workflow_n8n_financeiro.json && echo "ERRO: token antigo ainda presente" || echo "OK: workflow sanitizado"

# 3. Confirmar que instrucoes_n8n.ps1 foi sanitizado
grep -q "n8n-b2a-secret-key-2026" erp/instrucoes_n8n.ps1 && echo "ERRO: token antigo presente" || echo "OK: script sanitizado"

# 4. Confirmar que .kilocode foi removido
test ! -d erp/.kilocode && echo "OK: .kilocode removido" || echo "ERRO: .kilocode ainda existe"

# 5. Confirmar que .env não está no Git
git status erp/.env 2>&1 | grep -q "nothing to commit" && echo "OK: .env não tracked" || echo "ALERTA: verificar .env"

# 6. Typecheck do ERP
cd erp && npm run typecheck && echo "OK: typecheck passou" || echo "ERRO: typecheck falhou"
```

## Histórico de versões

- **v1.0.0 (2026-11-24):** versão inicial. Adaptações documentadas, aplicação pendente (pendente de cópia do código original).
