# 09 — Limitações e Problemas Conhecidos

**Última verificação:** 2026-11-24 · **Fonte:** `Catalogo-Capacidades-ERP-B2A-Engenharia.md` seções 2.5 e 7

## Sumário

Débitos técnicos, lacunas funcionais e riscos de segurança identificados no código, com remediação sugerida.

---

## Bloqueadores de automação `[VERIFICADO]`

Estes 5 itens bloqueiam a exposição segura de operações do ERP a agentes externos:

### B1 — Single-admin RLS

- **Arquivo:** `supabase/migrations/20260818_single_admin_rls.sql`.
- **Impacto:** impossível ter identidade distinta para agentes externos; todos os agentes usariam o mesmo usuário admin.
- **Remediação mínima:** migration de papéis (`erp_admin`, `erp_agent_read`, `erp_agent_write`) com scoped RLS por `obra_id`.

### B2 — Bearer token n8n exposto no Git

- **Arquivo:** `workflow_n8n_financeiro.json`, nó "💾 Enviar ao Supabase (ERP B2A)1", linha de `headerParameters`.
- **Conteúdo exposto:** `"value": "Bearer n8n-b2a-secret-key-2026"`.
- **Impacto:** credencial comprometida no histórico do Git; qualquer com acesso ao repo tem acesso de escrita ao ERP.
- **Remediação:** rotacionar token imediatamente; usar n8n credentials manager; remover do JSON versionado.

### B3 — Schema-base não versionado

- **Fonte:** `ARCHITECTURE.md` confirma.
- **Impacto:** estado real do Supabase é desconhecido; agentes podem assumir schema que não existe.
- **Remediação:** export do schema ativo (`supabase db pull`) + reconciliar com migrations locais.

### B4 — Dois lockfiles

- **Arquivos:** `package-lock.json` (npm) + `bun.lock` (Bun).
- **Impacto:** instabilidade em CI; agentes podem falhar ao instalar dependências.
- **Remediação:** decidir gerenciador canônico (recomenda-se `npm` pela compatibilidade Next.js); remover o outro lockfile.

### B5 — ESLint 10 incompatível

- **Arquivo:** `package.json` declara ESLint 10.0.3; `eslint-config-next` suporta até ESLint 9.
- **Fonte:** `OPERATIONS.md` confirma que `npm run lint` está bloqueado.
- **Impacto:** sem lint no CI; débitos entram em produção silenciosamente.
- **Remediação:** pin ESLint 9.x ou esperar `eslint-config-next` suportar 10.

---

## Lacunas funcionais confirmadas `[VERIFICADO]`

### L1 — `PendenciaComentario` sem CRUD

- **Onde deveria estar:** `src/lib/storage.ts`.
- **Tipo definido em:** `src/lib/types.ts` mas **nenhum método** lê ou grava comentários.
- **Impacto:** funcionalidade de comentários em pendências não funciona.

### L2 — `conciliacao_financeira` campos vazios

- **Campos:** `possivel_origem_tipo`, `possivel_origem_id`, `confianca_match`.
- **Onde deveria ser preenchido:** `src/lib/financial-service.ts`.
- **Impacto:** match automático não acontece — toda conciliação é manual.

### L3 — Workflow n8n só processa RECIBOs

- **Arquivo:** `workflow_n8n_financeiro.json`, filter node "🎯 Filtrar Somente Recibos1".
- **Impacto:** NFs e Boletos são ignorados.

### L4 — `forma_pagamento` não enviado ao ERP

- **Arquivo:** `workflow_n8n_financeiro.json`, body do HTTP Request.
- **Impacto:** ERP não sabe se foi PIX, Boleto, etc.

### L5 — Match de fornecedor por `ilike %nome%`

- **Arquivo:** `src/lib/financial-service.ts:69` (aproximado).
- **Impacto:** falsos positivos em homônimos (ex.: "Concreto RP Locações" vs. "Concreto RP Matéria").
- **Evidência:** relatório `docs/relatorios/OBR-003-custos-2026-09-14.md` mostra 65 transações onde 60+ estão marcadas como "manual" indicando que o match falha frequentemente.

### L6 — Cache sem invalidação seletiva

- **Arquivo:** `src/lib/storage.ts`, função `cacheInvalidate()`.
- **Comportamento:** limpa **todo o cache** em vez de chaves específicas.
- **Impacto:** race conditions em ambientes multi-agente.

### L7 — `created_by` nunca preenchido

- **Arquivo:** `src/lib/storage.ts`, função `saveLancamentoFinanceiro`.
- **Impacto:** sem auditoria de quem criou cada lançamento.

### L8 — `wf_healthy_template_v2.json` grava no Sheets, não no Supabase

- **Impacto:** divergência entre ERP e Sheets.

### L9 — Plano de integração cronograma↔financeiro não implementado

- **Arquivo:** `plano_integracao_cronograma_financeiro.md`.
- **Impacto:** indicadores EV/PV/AC indisponíveis.

### L10 — `marcarPedidoPago` não atualiza lançamento financeiro

- **Arquivo:** `src/features/compras-workflow/hooks/useComprasWorkflow.ts`.
- **Impacto:** lançamento previsto não é atualizado para `pago` quando pedido é marcado como entregue.

---

## Lacunas de segurança `[VERIFICADO]`

### S1 — Comparação de bearer token com `===`

- **Arquivo:** `src/app/api/financeiro/importar/route.ts:32` (aproximado).
- **Risco:** vulnerável a timing attack.
- **Remediação:** usar `crypto.timingSafeEqual`.

### S2 — Sem rate limiting nos Route Handlers

- **Arquivo:** todos.
- **Risco:** agentes maliciosos podem DoS.
- **Remediação:** implementar com `@upstash/ratelimit` ou middleware custom.

### S3 — `service_role` key com fallback perigoso

- **Arquivo:** `src/lib/supabase-admin.ts:31`.
- **Comportamento:** se `SUPABASE_SERVICE_ROLE_KEY` vazio, usa `publishable_key` (perms limitadas) — silencioso.
- **Risco:** erro mascarado como permissão RLS.
- **Remediação:** falhar explícito if service_role key não configurada em server-side.

### S4 — Sem CSP estrito confirmado

- **Arquivo:** `next.config.ts` (não lido em detalhe).
- **Risco:** XSS em campos livres.
- **Remediação:** auditar CSP headers.

### S5 — Headers de Authorization em logs de erro

- **Status:** não encontrado nos logs do route handler, mas risco potencial.
- **Remediação:** sanitizar headers antes de logar.

---

## Limitações arquiteturais `[VERIFICADO]`

### A1 — Sem framework de testes

- **Fonte:** `package.json` não declara; `OPERATIONS.md` confirma.
- **Impacto:** nenhuma regressão é detectada automaticamente.

### A2 — Todas as operações de negócio acopladas ao browser

- **Arquivo:** `src/lib/storage.ts` (41 KB) é cliente browser.
- **Impacto:** não há endpoint REST estruturado para agentes externos.
- **Remediação:** criar camada de Tools API server-side que reutiliza `storage.ts`.

### A3 — Agente IA sem function calling

- **Arquivo:** `src/app/api/agent/route.ts`.
- **Impacto:** agente não pode executar ações no ERP, apenas responder com base em contexto limitado.

### A4 — Memória do agente IA não persistente

- **Impacto:** cada chamada é stateless com histórico manual.

### A5 — Contexto do agente IA limitado a 5 entidades

- **Impacto:** agente não vê histórico completo.

---

## Itens marcados `[DESCONHECIDO]`

Estes itens não puderam ser verificados a partir do código-fonte e exigem consulta autorizada ao ambiente real:

- Variáveis de ambiente efetivamente configuradas na Vercel.
- Domínio customizado além do subdomínio Vercel.
- Estado real do schema-base do Supabase (migrations remotas não versionadas).
- Buckets Supabase Storage configurados.
- Configuração OAuth do n8n ativa.
- Lista completa de webhooks n8n ativos.
- Políticas RLS efetivamente aplicadas em produção.
- Status da configuração CSP em `next.config.ts`.
- Conteúdo de `/admin/configuracoes` e `/debug`.

---

## Dependências

- Este documento depende de: `02-modulos.md`, `03-processos.md`, `05-modelo-dados.md`, `06-apis-integracoes.md`.
- Atualizações neste documento podem afetar: `08-estado-implementacao.md`, `10-historico-decisoes.md`.
