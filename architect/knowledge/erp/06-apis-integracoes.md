# 06 — APIs e Integrações

**Última verificação:** 2026-11-24 · **Fonte:** `Catalogo-Capacidades-ERP-B2A-Engenharia.md` seção 3

## Sumário

3 endpoints de API server-side + 2 workflows n8n + 1 integração Gemini + 1 cliente Supabase. Nenhum endpoint está exposto como REST estruturado para agentes externos — todos são internos ao Next.js.

---

## APIs Server-Side `[VERIFICADO]`

### API 1: POST `/api/agent` — Agente IA Conversacional

**Arquivo:** `src/app/api/agent/route.ts`

**Autenticação:** sessão Supabase Auth via cookie (`getAuthenticatedRequest()` em `src/lib/api-auth.ts`). Rejeita 401 se não houver `userId`.

**Schema de entrada (Zod estrito):**
```typescript
{
  message: string (1–4000 chars),
  obraId?: string (1–128 chars),
  history?: Array<{ role: 'user'|'model', content: string (1–4000) }> (max 20)
}
```

**Contexto construído para o Gemini (5 entidades por tipo):**
- 5 obras mais recentes (`id_obra, nome_obra, status_obra, custo_planejado, percentual_fisico_atual`)
- 5 lançamentos financeiros mais recentes da obra ativa
- 5 lançamentos pendentes de conciliação
- 5 pedidos de compra mais recentes da obra ativa

**System prompt fixo:**
```
Você é o Assistente B2A, um agente especializado no ERP de gestão de obras da B2A Engenharia.
Responda em português brasileiro, de forma objetiva, usando somente o contexto fornecido.
Não invente dados.
```

**Output:** `{ reply: string }`.

**Limitações confirmadas:**
- Contexto limitado a 5 entidades por tipo.
- Sem function calling — apenas geração de texto.
- `obraId` opcional; se ausente, queries de lançamentos retornam vazio.

### API 2: POST `/api/financeiro/importar` — Webhook n8n

**Arquivo:** `src/app/api/financeiro/importar/route.ts`

**Autenticação:** `Authorization: Bearer <N8N_IMPORT_API_KEY>` — bearer token comparado com `===` (não constant-time) `[VERIFICADO]`. **Exceção deliberada** ao middleware de Auth.

**Schema (Zod passthrough):**
```typescript
{
  DATA_PAGAMENTO?: string,
  NOME_FORNECEDOR: string (1–200, obrigatório),
  VALOR: number (finite, ≥ 0, coercível),
  NUMERO_NOTA?: string,
  NOME_ARQUIVO?: string,
  OBRA_ID?: string,
  EMAIL_DESTINATARIO?: string,
  EMAIL_REMETENTE?: string,
  EMAIL_CONTA?: string,
  LINK_ARQUIVO?: string (URL, max 2048),
  EXTERNAL_ID?: string (1–255, idempotência)
}
```

**Lógica em `financialService.processImport()` (`src/lib/financial-service.ts`):**
1. Resolve `obra_id`: se `OBRA_ID` informado, valida existência; senão, busca por `email_conta`/`email_destinatario`/`email_remetente` em `obras.email_vinculado` ou `obras.email_obra` ou `obra_emails.email`.
2. Idempotência: se `EXTERNAL_ID` já existe em `transacoes_reais.external_id`, retorna `{ action: "ignored_duplicate", id, obra_id }`.
3. Resolve `fornecedor_id` por `ilike` em `fornecedores.nome_fantasia` (primeiro match).
4. INSERT em `transacoes_reais` com `obra_id`, `email_conta`, `descricao`, `fornecedor_id`, `valor`, `data_pagamento` (ISO), `numero_nota`, `link_documento`, `external_id`.
5. Retorna `{ action: "created", id, obra_id }`.

**Status codes:**
- 200 (created/ignored_duplicate)
- 400 (payload inválido)
- 401 (bearer inválido)
- 422 (obra não localizada)
- 500 (erro interno)
- 503 (N8N_IMPORT_API_KEY não configurada)

### API 3: POST `/api/financeiro/lancamentos` — Rateio / Exclusão

**Arquivo:** `src/app/api/financeiro/lancamentos/route.ts`

**Autenticação:** sessão Supabase Auth (via `getAuthenticatedRequest`).

**Schema (Zod discriminatedUnion):**
```typescript
| { action: "updateRateio", id: string, classificacao_financeira_id: string|null }
| { action: "delete", id: string }
```

**Observação:** o comentário no código confirma que a ação `create` foi **removida** — lançamentos só existem como transações reais vindas do n8n.

---

## Camada `storage.ts` — Operações NÃO expostas como API `[VERIFICADO]`

**Arquivo:** `src/lib/storage.ts` (41 KB, ~30 métodos). Esta é a **fonte canônica** de operações de negócio. Cada método é candidato direto a virar uma *tool* para agentes.

**Categorias de operações disponíveis:**

| Domínio | Métodos públicos |
|---|---|
| Sequências | `getNextSequence(prefix)` |
| Obras | `getObras`, `saveObra` |
| Obras Email | `getObrasEmails`, `saveObraEmail`, `deleteObraEmail`, `getObraIdByEmail` |
| Pendências | `getPendencias`, `savePendencia`, `deletePendencia` |
| Contratos | `getContratos`, `saveContrato`, `deleteContrato`, `getContratoItens`, `saveContratoItem` |
| Medições | `getMedicoes`, `saveMedicao`, `deleteMedicao`, `getMedicaoItens`, `saveMedicaoItem`, `getMedicaoItensByContrato` |
| Solicitações | `getSolicitacoes`, `saveSolicitacao`, `deleteSolicitacao`, `getSolicitacaoItens`, `saveSolicitacaoItem` |
| Equipes | `getEquipes`, `saveEquipe`, `deleteEquipe` |
| Materiais | `getMateriais`, `saveMaterial`, `deleteMaterial` |
| Tipos Serviço | `getTiposServico`, `saveTipoServico`, `deleteTipoServico` |
| Cronograma | `getCronogramaEtapas`, `saveCronogramaEtapa` |
| Empreiteiros | `getEmpreiteiros`, `saveEmpreiteiro`, `deleteEmpreiteiro` |
| Fornecedores | `getFornecedores`, `saveFornecedor`, `deleteFornecedor` |
| Cotações | `getCotacoes`, `saveCotacao`, `getCotacoesItensFornecedor`, `saveCotacaoItemFornecedor` |
| Pedidos Compra | `getPedidosCompra`, `savePedidoCompra`, `getPedidoCompraItens`, `savePedidoCompraItem`, `deletePedidoCompra`, `updatePedidoStatus` |
| Rateios | `saveRateioFinanceiro`, `getRateiosByReferencia`, `deleteRateiosByReferencia` |
| Lançamentos Financeiros | `getLancamentosFinanceiros`, `getLancamentosFinanceirosByObra`, `saveLancamentoFinanceiro` (com idempotência por `origem_id`), `marcarLancamentoPago`, `deleteLancamentoFinanceiro` |
| Conciliação | `getConciliacaoFinanceira`, `approveConciliacao`, `rejectConciliacao` |
| Transações Reais | `getTransacoesReais`, `getTransacoesReaisAll`, `saveTransacaoReal`, `deleteTransacaoReal` |
| Orçamento + Curva S | `getOrcamentoEtapasByObra`, `saveOrcamentoEtapa`, `deleteOrcamentoEtapa`, `getCurvaSByObra`, `saveCurvaSMensal`, `inicializarPlanejamentoObra` |

---

## Integrações externas `[VERIFICADO]`

### Supabase

- **Auth:** Supabase Auth com cookie de sessão.
- **Banco:** PostgreSQL com RLS habilitada em todas as tabelas públicas.
- **Client browser:** `src/lib/supabase.ts` — `createBrowserClient` com `NEXT_PUBLIC_SUPABASE_URL` + `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` (ou fallback `NEXT_PUBLIC_SUPABASE_ANON_KEY`).
- **Client admin (server-only):** `src/lib/supabase-admin.ts` — usa `SUPABASE_SERVICE_ROLE_KEY` (com fallback perigoso para publishable se service_role vazio `[VERIFICADO]`).
- **Auth helper:** `src/lib/api-auth.ts` — `getAuthenticatedRequest()` para Route Handlers.
- **Utils:** `src/utils/supabase/server.ts` — server client via cookies.

### Google Gemini

- **SDK:** `@google/generative-ai` ^0.24.1.
- **Modelo:** `gemini-2.0-flash`.
- **Uso:** apenas em `POST /api/agent`.
- **Auth:** `GEMINI_API_KEY` env var.
- **Sem function calling** — apenas chat completion com system prompt + contexto.

### n8n

- **Instância:** `https://n8nb2aengenharia.site` `[VERIFICADO]`.
- **Workflows versionados no Git:**
  - `workflow_n8n_financeiro.json` (23 KB) — OCR financeiro.
  - `wf_healthy_template_v2.json` (25 KB) — criação de obra via Google Form → Sheets.
- **Conexão com ERP:** POST para `https://appsite-erp-b2-a-engenharia.vercel.app/api/financeiro/importar` com bearer `n8n-b2a-secret-key-2026`.
- **Integrações dentro do n8n:** Gmail (3 contas), Tesseract OCR (idioma português), Google Sheets, Google Drive, Gemini (mencionado em `ARCHITECTURE.md` mas não visível no workflow principal).

### Vercel

- **Hosting:** `vercel.json` declara framework Next.js.
- **URL de produção:** `https://appsite-erp-b2-a-engenharia.vercel.app`.
- **Configuração adicional:** `[DESCONHECIDO]` — não versionada no Git.

### Google Workspace

- **Gmail:** 3 contas monitoradas pelo n8n (`email sandro`, `Obras01`, `email obras`).
- **Google Sheets:** planilha `BD_GESTAO_OBRAS_V2` (workflow secundário de criação de obra).
- **Google Drive:** `pasta_drive_id` em `obras` (referenciado mas uso não detalhado no código lido).

---

## Variáveis de ambiente referenciadas `[VERIFICADO]`

| Variável | Uso | Onde |
|---|---|---|
| `NEXT_PUBLIC_SUPABASE_URL` | URL pública Supabase | clientes browser/server e middleware |
| `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` | chave pública Supabase preferida | clientes browser/server e middleware |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | fallback de chave pública | clientes browser/server e middleware |
| `SUPABASE_SERVICE_ROLE_KEY` | acesso administrativo somente servidor | agente IA e serviço financeiro |
| `GEMINI_API_KEY` | auth API Gemini | `/api/agent` |
| `N8N_IMPORT_API_KEY` | bearer token webhook n8n | `/api/financeiro/importar` |

---

## Webhook payload canônico (n8n → ERP) `[VERIFICADO]`

**Headers obrigatórios:**
```http
Authorization: Bearer <N8N_IMPORT_API_KEY>
Content-Type: application/json
```

**Request body:**
```json
{
  "DATA_PAGAMENTO": "2026-07-10",
  "NOME_FORNECEDOR": "Nome Fornecedor LTDA",
  "VALOR": 1250.50,
  "NUMERO_NOTA": "00054321",
  "NOME_ARQUIVO": "NF-54321.pdf",
  "OBRA_ID": "uuid-da-obra-relacionada",
  "EMAIL_CONTA": "email-da-obra@b2a.com.br",
  "LINK_ARQUIVO": "https://drive.google.com/open?id=...",
  "EXTERNAL_ID": "external_id_unico_para_idempotencia"
}
```

**Response 200 OK:**
```json
{
  "action": "created" | "ignored_duplicate",
  "id": "UUID-do-registro",
  "obra_id": "uuid-da-obra"
}
```

---

## Lacunas de conhecimento sobre este domínio `[DESCONHECIDO]`

- Endpoint `external_id` mencionado em `ARCHITECTURE.md` mas não no checkout.
- Buckets Supabase Storage (não há referências no código).
- Configuração OAuth do n8n (Gmail, Google Drive, Google Sheets) — credenciais vivem no próprio n8n.
- Lista completa de webhooks n8n ativos.
- Outras integrações n8n além do workflow financeiro (Gemini é mencionado em `ARCHITECTURE.md`).

---

## Dependências

- Este documento depende de: `02-modulos.md`, `05-modelo-dados.md`, `03-processos.md`.
- Atualizações neste documento podem afetar: `09-limitacoes-problemas.md`.
