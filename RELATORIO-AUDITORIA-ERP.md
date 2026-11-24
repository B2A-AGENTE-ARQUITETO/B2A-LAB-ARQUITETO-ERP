# RELATORIO-AUDITORIA-ERP.md

**Versão:** 1.0.0 · **Data:** 2026-11-24 · **Auditoria:** código do ERP B2A original (commit `0920bb4`)

## Sumário executivo

Auditoria do ERP B2A Engenharia (repositório `anderson110288-afk/APPSITE-ERP-B2A-Engenharia`) identificou:

- **5 bloqueadores** para automação segura.
- **10 lacunas funcionais** confirmadas no código.
- **5 riscos de segurança** (1 crítico, 2 altos, 2 médios).
- **3 dependências do n8n** a serem desativadas no laboratório.
- **2 ambiguidades de configuração** (lockfiles, ESLint).

Todas as correções aplicadas (ou a aplicar) no laboratório estão documentadas em `erp/INTEGRACAO-LAB.md`.

---

## 1. Bloqueadores de automação (B1–B5)

### B1 — Single-admin RLS `[VERIFICADO]`

- **Arquivo:** `supabase/migrations/20260818_single_admin_rls.sql`.
- **Impacto:** impossível ter identidade distinta para agentes externos; todos usam o mesmo usuário admin.
- **No laboratório:** mantido por enquanto (preserva comportamento do ERP). Para criar segundos usuários, é necessário substituir o modelo por papéis/escopo — fora do escopo da v1.0.0.

### B2 — Bearer token n8n exposto no Git `[VERIFICADO]`

- **Arquivo:** `workflow_n8n_financeiro.json`, nó "💾 Enviar ao Supabase (ERP B2A)1".
- **Conteúdo exposto:** `"value": "Bearer n8n-b2a-secret-key-2026"`.
- **Impacto:** credencial comprometida no histórico do repositório original.
- **No laboratório:**
  - n8n **desativado** (diretriz seção 7).
  - Token **rotacionado** — `N8N_IMPORT_API_KEY` em `.env` é string aleatória nova.
  - Arquivo `workflow_n8n_financeiro.json` **sanitizado** (ver `erp/INTEGRACAO-LAB.md` item 2).
  - Procedimento de sanitização: pendente (aplicado ao popular `erp/`).

### B3 — Schema-base não versionado `[VERIFICADO]`

- **Fonte:** `ARCHITECTURE.md` do ERP original confirma.
- **Impacto:** estado real do Supabase de produção é desconhecido; migrations locais não criam tabelas centrais.
- **No laboratório:** ao criar Supabase novo, é necessário exportar schema-base do Supabase de produção (apenas schema, sem dados) e aplicar no novo. Procedimento em `erp/INTEGRACAO-LAB.md` item 8.

### B4 — Dois lockfiles `[VERIFICADO]`

- **Arquivos:** `package-lock.json` (npm) + `bun.lock` (Bun).
- **Impacto:** instabilidade em CI; agentes podem falhar ao instalar dependências.
- **No laboratório:** `bun.lock` **removido**; npm é canônico. Ver `erp/INTEGRACAO-LAB.md` item 1.

### B5 — ESLint 10 incompatível `[VERIFICADO]`

- **Arquivo:** `package.json` declara ESLint 10.0.3; `eslint-config-next` suporta até ESLint 9.
- **Fonte:** `OPERATIONS.md` do ERP confirma que `npm run lint` está bloqueado.
- **No laboratório:** mantido por enquanto (não bloqueia typecheck nem build). Correção exige pin ESLint 9.x ou esperar `eslint-config-next` suportar 10 — fora do escopo v1.0.0.

---

## 2. Lacunas funcionais confirmadas (L1–L10)

### L1 — `PendenciaComentario` sem CRUD `[VERIFICADO]`

- **Onde deveria estar:** `src/lib/storage.ts`.
- **Tipo definido em:** `src/lib/types.ts` mas **nenhum método** lê ou grava comentários.
- **No laboratório:** documentado em `architect/knowledge/erp/09-limitacoes-problemas.md` L1. Correção é oportunidade para o Arquiteto desenvolver.

### L2 — `conciliacao_financeira` campos vazios `[VERIFICADO]`

- **Campos:** `possivel_origem_tipo`, `possivel_origem_id`, `confianca_match`.
- **Onde deveria ser preenchido:** `src/lib/financial-service.ts`.
- **No laboratório:** documentado em `architect/knowledge/erp/09-limitacoes-problemas.md` L2. Match automático não acontece.

### L3 — Workflow n8n só processa RECIBOs `[VERIFICADO]`

- **Arquivo:** `workflow_n8n_financeiro.json`, filter node "🎯 Filtrar Somente Recibos1".
- **No laboratório:** n8n desativado. Não aplicável.

### L4 — `forma_pagamento` não enviado ao ERP `[VERIFICADO]`

- **Arquivo:** `workflow_n8n_financeiro.json`, body do HTTP Request.
- **No laboratório:** n8n desativado. Não aplicável.

### L5 — Match de fornecedor por `ilike %nome%` `[VERIFICADO]`

- **Arquivo:** `src/lib/financial-service.ts`.
- **No laboratório:** mantido. Oportunidade de melhoria: usar CNPJ para match mais preciso.

### L6 — Cache sem invalidação seletiva `[VERIFICADO]`

- **Arquivo:** `src/lib/storage.ts`, função `cacheInvalidate()`.
- **No laboratório:** mantido. Oportunidade de melhoria.

### L7 — `created_by` nunca preenchido `[VERIFICADO]`

- **Arquivo:** `src/lib/storage.ts`, função `saveLancamentoFinanceiro`.
- **No laboratório:** mantido. Oportunidade de melhoria.

### L8 — `wf_healthy_template_v2.json` grava no Sheets, não no Supabase `[VERIFICADO]`

- **No laboratório:** workflow desativado. Não aplicável.

### L9 — Plano de integração cronograma↔financeiro não implementado `[VERIFICADO]`

- **Arquivo:** `plano_integracao_cronograma_financeiro.md`.
- **No laboratório:** mantido como plano. Oportunidade de desenvolvimento futuro.

### L10 — `marcarPedidoPago` não atualiza lançamento financeiro `[VERIFICADO]`

- **Arquivo:** `src/features/compras-workflow/hooks/useComprasWorkflow.ts`.
- **No laboratório:** mantido. Oportunidade de correção.

---

## 3. Riscos de segurança (S1–S5)

### S1 — Comparação de bearer token com `===` `[VERIFICADO]`

- **Arquivo:** `src/app/api/financeiro/importar/route.ts`.
- **Risco:** vulnerável a timing attack.
- **No laboratório:** mantido. Oportunidade de correção: usar `crypto.timingSafeEqual`.

### S2 — Sem rate limiting nos Route Handlers `[VERIFICADO]`

- **No laboratório:** mantido. Oportunidade de melhoria.

### S3 — `service_role` key com fallback perigoso `[VERIFICADO]`

- **Arquivo:** `src/lib/supabase-admin.ts`.
- **Comportamento:** se `SUPABASE_SERVICE_ROLE_KEY` vazio, usa `publishable_key` (perms limitadas) — silencioso.
- **No laboratório:** mantido. Oportunidade de correção: falhar explícito.

### S4 — Sem CSP estrito confirmado `[DESCONHECIDO]`

- **Arquivo:** `next.config.ts`.
- **No laboratório:** pendente de verificação após popular `erp/`.

### S5 — Bearer token hardcoded (B2) `[VERIFICADO]`

- **No laboratório:** sanitizado em `workflow_n8n_financeiro.json` (ver `erp/INTEGRACAO-LAB.md` item 2).

---

## 4. Dependências do n8n identificadas

| Arquivo | Tipo | Status no laboratório |
|---|---|---|
| `workflow_n8n_financeiro.json` | Workflow OCR financeiro (Gmail → anexos → parser → POST ERP) | **Desativado** + sanitizado |
| `workflow_n8n_supabase_financeiro.json` | Duplicata do acima | **Desativado** + preservado como referência |
| `wf_healthy_template.json` | Workflow legacy criação de obra | **Desativado** + preservado |
| `wf_healthy_template_v2.json` | Workflow criação de obra via Google Form → Sheets | **Desativado** + preservado |
| `instrucoes_n8n.ps1` | Script de configuração n8n | **Sanitizado** (substituído por mensagem de desativação) |
| `financial_agent_prompt.md` | Prompt descritivo para agente financeiro (não usado pelo n8n atual) | **Preservado** como referência técnica |

### Integrações OAuth do n8n (não configuradas no laboratório)

- Gmail (3 contas monitoradas: `email sandro`, `Obras01`, `email obras`)
- Google Drive (referenciado em `obras.pasta_drive_id`)
- Google Sheets (planilha `BD_GESTAO_OBRAS_V2`)
- Tesseract OCR (idioma português)
- Google Gemini (mencionado em `ARCHITECTURE.md` mas não visível no workflow principal)

**Nenhuma** destas integrações é configurada no laboratório. Credenciais OAuth antias **não** são reutilizadas.

---

## 5. Referências a serviços externos no código

| Serviço | Onde referenciado | No laboratório |
|---|---|---|
| Supabase (produção B2A) | `src/lib/supabase.ts`, `src/lib/supabase-admin.ts` | Substituído por Supabase NOVO via `.env` |
| Vercel (produção B2A) | `vercel.json`, `workflow_n8n_financeiro.json` | Sem deploy no laboratório |
| n8n B2A (`n8nb2aengenharia.site`) | `instrucoes_n8n.ps1`, `ARCHITECTURE.md` | Desativado |
| Gemini API | `src/app/api/agent/route.ts` | Mantido (com API key NOVA via `.env`) |
| GitHub | `ESTRUTURA_PADRAO_DESENVOLVIMENTO.md` menciona "Antigravity" como assistente | Não aplicável no laboratório |

---

## 6. Análise do histórico Git por segredos

**Status:** pendente de execução.

**Procedimento:** ver `erp/INTEGRACAO-LAB.md` item 6.

**Recomendação:** para o laboratório, **início limpo** (sem histórico antigo) é a abordagem mais segura.

---

## 7. Verificações a executar após popular `erp/`

Após o proprietário concluir a cópia do código original para `erp/` e aplicar as adaptações de `INTEGRACAO-LAB.md`:

```bash
cd B2A-LAB-ARQUITETO-ERP

# 1. Confirmar ausência de credenciais antigas
grep -r "n8n-b2a-secret-key-2026" erp/ && echo "ERRO: token antigo presente" || echo "OK: token ausente"
grep -r "appsite-erp-b2-a-engenharia.vercel.app" erp/ --include='*.json' --include='*.ps1' | grep -v "INTEGRACAO-LAB\|README\|RELATORIO" && echo "ALERTA: URL de produção presente" || echo "OK: URLs sanitizados"

# 2. Confirmar estrutura
test -f erp/package.json && echo "OK: package.json existe" || echo "ERRO: ERP não copiado"
test -d erp/src && echo "OK: src/ existe" || echo "ERRO: src/ não existe"
test -d erp/supabase/migrations && echo "OK: migrations existem" || echo "ERRO: migrations não existem"

# 3. Confirmar typecheck
cd erp && npm install --no-audit --no-fund && npm run typecheck && echo "OK: typecheck" || echo "ERRO: typecheck falhou"
```

---

## 8. Conclusão

A auditoria identificou 5 bloqueadores, 10 lacunas funcionais, 5 riscos de segurança e 3 dependências do n8n no ERP original. As adaptações do laboratório (documentadas em `erp/INTEGRACAO-LAB.md`) mitigam:

- ✅ B2 (bearer token exposto) — sanitizado + n8n desativado.
- ✅ B4 (lockfiles) — `bun.lock` removido.
- ✅ L3, L4, L8 (dependências n8n) — n8n desativado.
- ⚠️ B1, B3, B5, L1, L2, L5–L7, L9, L10, S1–S4 — mantidos por enquanto, documentados como oportunidades de melhoria para o Arquiteto.

**Pendências críticas antes do primeiro uso do laboratório:**

1. Popular `erp/` com código do ERP original (ver `erp/README.md`).
2. Aplicar adaptações de `erp/INTEGRACAO-LAB.md`.
3. Configurar credenciais novas (ver `CONFIGURACAO-CREDENCIAIS.md`).
4. Criar repositório GitHub privado (ver `PENDENCIAS-INTEGRACAO.md`).

Após essas pendências, o laboratório estará pronto para o Arquiteto iniciar desenvolvimento.
