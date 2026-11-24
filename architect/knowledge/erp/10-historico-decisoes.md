# 10 — Histórico de Decisões Técnicas

**Última verificação:** 2026-11-24 · **Fonte:** `Catalogo-Capacidades-ERP-B2A-Engenharia.md` + `ARCHITECTURE.md` + `AGENTS.md` + `ESTRUTURA_PADRAO_DESENVOLVIMENTO.md` + `plano_integracao_cronograma_financeiro.md`

## Sumário

Decisões técnicas registradas no repositório do ERP, com data, contexto e rationale quando disponível.

---

## D1 — Adoção de Next.js 16 App Router `[VERIFICADO]`

- **Data:** anterior a 2026-03-14 (primeira migration).
- **Decisão:** usar Next.js 16 com App Router (não Pages Router).
- **Rationale:** `[INFERIDO]` stack moderna, suporte a React Server Components, integração nativa com Vercel.
- **Arquivo:** `package.json` declara `next: 16.1.6`.

---

## D2 — Supabase como banco + auth `[VERIFICADO]`

- **Data:** anterior a 2026-03-14.
- **Decisão:** PostgreSQL gerenciado via Supabase + Supabase Auth.
- **Rationale:** `[INFERIDO]` reduz overhead operacional, RLS nativa, auth integrada.
- **Arquivos:** `src/lib/supabase.ts`, `src/lib/supabase-admin.ts`, `src/utils/supabase/server.ts`.

---

## D3 — Padrão Feature Folder `[VERIFICADO]`

- **Data:** definido em `ESTRUTURA_PADRAO_DESENVOLVIMENTO.md`.
- **Decisão:** novas funcionalidades em `src/features/<nome>/` com subpastas `components/`, `hooks/`, `api/`, `lib/`, `types/`. Rota de teste em `src/app/[nome]-teste/page.tsx`. Mesclagem via "Antigravity" (assistente principal do projeto).
- **Rationale:** encapsulamento plug-and-play, evita interferência em regras globais.
- **Proibições explícitas no documento:**
  - Não alterar `package.json` raiz sem validação.
  - Não hardcode de credenciais.
  - Não usar `@ts-ignore` ou casting excessivo para `any`.
  - Não mutar schemas de produção sem migration.
  - Não remover ou contornar regras CSP.
  - Não usar datas com timezone nativo do JS (usar `.split("T")[0]`).

---

## D4 — Sequência de IDs via RPC SECURITY DEFINER `[VERIFICADO]`

- **Data:** 2026-03-14.
- **Decisão:** criar RPC `get_next_sequence(text, integer)` com `SECURITY DEFINER` em vez de sequence nativa do PostgreSQL.
- **Rationale:** `[INFERIDO]` permite controle fino de formato (`PREFIX-NNNN`), padronização B2A.
- **Atualização em 2026-08-19:** revogada execução anônima — apenas `authenticated` e `service_role`.

---

## D5 — Módulo financeiro com idempotência dupla `[VERIFICADO]`

- **Data:** 2026-03-23.
- **Decisão:** `lancamentos_financeiros` com idempotência por `(origem, origem_id)` para escritas internas e por `external_id` para escritas n8n.
- **Rationale:** evitar duplicatas quando n8n reenvia webhook ou quando medição/pedido é re-salvo.
- **Arquivo:** `src/lib/storage.ts`, função `saveLancamentoFinanceiro`.

---

## D6 — Modelo single-admin RLS `[VERIFICADO]`

- **Data:** 2026-08-18.
- **Decisão:** substituir policies permissivas ( migration `20260413_master_rls_fix.sql`) por modelo single-admin com allowlist `erp_admin_users` + function `private.is_erp_admin()`.
- **Rationale:** simplificar autorização enquanto há apenas 1 usuário; fail-closed.
- **Limitação explícita no documento:** "Antes de adicionar novos usuários, implemente papéis e escopo por obra em migration própria" (`SECURITY.md`).
- **Migration aborta se não houver exatamente 1 usuário em `auth.users`.**

---

## D7 — Parser determinístico por regex no n8n (sem IA) `[VERIFICADO]`

- **Data:** `[INFERIDO]` entre 2026-03 e 2026-09.
- **Decisão:** usar parser determinístico por regex no workflow n8n em vez de LLM para extrair dados de comprovantes.
- **Rationale:** `[INFERIDO]` reduz consumo de tokens e latência; regras explícitas para NF/Boleto/Recibo/Fatura.
- **Comentário no código do n8n:** "parser deterministico sem IA para reduzir consumo de tokens".
- **Limitação:** workflow só processa `tipo_documento == 'RECIBO'` — NFs e Boletos são ignorados.

---

## D8 — Remoção da ação `create` em `/api/financeiro/lancamentos` `[VERIFICADO]`

- **Data:** `[INFERIDO]` entre 2026-09 e 2026-11.
- **Decisão:** remover a ação `create` do endpoint; lançamentos só existem como transações reais vindas do n8n.
- **Rationale:** `[INFERIDO]` centralizar criação de lançamentos no pipeline n8n (fonte única de verdade para transações confirmadas).
- **Comentário no código:** "Ação removida: 'create' (criação prévia de lançamentos por medição/pedido) — Lançamentos só existem como transações reais provenientes do n8n."
- **Tensão com RN-02:** RN-02 ainda cria lançamentos previstos via `pedidoToFinanceiro()`. `[INFERIDO]` a coexistência é: lançamentos previstos são criados no ERP (origem pedido), mas **não** pela API externa — apenas pela UI/Storage.

---

## D9 — Inicialização automática de planejamento (14 etapas B2A) `[VERIFICADO]`

- **Data:** 2026-09-20.
- **Decisão:** ao criar obra, inicializar automaticamente 14 etapas orçamentárias com percentuais fixos B2A + curva S mensal linear.
- **Rationale:** `[INFERIDO]` padronização de planejamento entre obras, baseline consistente para comparação.
- **Arquivo:** `src/lib/storage.ts`, função `inicializarPlanejamentoObra`.

---

## D10 — Marcos no cronograma `[VERIFICADO]`

- **Data:** 2026-11-04.
- **Decisão:** adicionar `eh_marco`, `nome_marco`, `ordem_marco` a `cronograma_etapas`.
- **Rationale:** `[INFERIDO]` permitir apresentação de marcos ao cliente (não apenas etapas executivas).
- **Arquivo:** `supabase/migrations/20261104_cronograma_marcos.sql`.

---

## D11 — Aprovação de cotação via WhatsApp com opções explícitas `[VERIFICADO]`

- **Data:** `[INFERIDO]` entre 2026-03 e 2026-11.
- **Decisão:** template WhatsApp envia mapa de cotação com opções `APROVAR N` ou `REVISAR` explícitas.
- **Rationale:** `[INFERIDO]` permitir aprovação rápida pelo cliente sem acessar o ERP.
- **Limitação:** sem callback automatizado — resposta é capturada manualmente.
- **Arquivo:** `src/features/compras-workflow/lib/whatsappTemplates.ts`.

---

## D12 — Princípio operacional do ERP `[VERIFICADO]`

- **Data:** definido em `AGENTS.md`.
- **Decisão:** qualquer IA ou humano trabalhando no ERP deve seguir: `COMPREENDER → DIAGNOSTICAR → PLANEJAR → VERSIONAR → ALTERAR → TESTAR → DOCUMENTAR`.
- **Regra explícita:** "Não refatore por iniciativa própria durante uma auditoria."
- **Regra explícita:** "Não faça commit, push, PR ou deploy sem pedido explícito."

---

## D13 — Uso de Zod apenas em 5 entidades críticas `[VERIFICADO]`

- **Data:** `[INFERIDO]` incremental.
- **Decisão:** validar com Zod apenas `Obra`, `Pendencia`, `ContratoEmpreiteiro`, `Fornecedor`, `NovoContrato`. Demais entidades fazem upsert direto.
- **Rationale:** `[DESCONHECIDO]` — não documentado; possivelmente pragmatismo inicial.
- **Risco:** dados inválidos podem entrar em tabelas sem validação.

---

## D14 — Cache em memória de 5 minutos `[VERIFICADO]`

- **Data:** `[INFERIDO]` inicial.
- **Decisão:** cache em memória no browser com TTL de 5 minutos para leituras frequentes.
- **Rationale:** `[INFERIDO]` reduzir latência e custo de queries Supabase.
- **Limitação:** invalidação é global, não seletiva.

---

## D15 — Plano de integração cronograma↔financeiro (NÃO implementado) `[VERIFICADO]`

- **Data:** documento `plano_integracao_cronograma_financeiro.md`.
- **Decisão planejada:** implementar EVM (Earned Value Management) com:
  - `OrcamentoCronograma` (valor orçado por etapa).
  - `LancamentoFinanceiroPrevisto` (previsão baseada em cronograma).
  - Campos em `Obra`: `custo_orcado`, `custo_executo`, `variancia_orcamentaria`, `indice_desempenho_financeiro`, `indice_desempenho_cronograma`.
  - Cálculos: EV = % × orçamento; PV = previsto no período; AC = custo real.
- **Status:** **não implementado**. A migration `obra_orcamento_etapas` (D9) sugere que parte foi feita, mas EVM não está no código.
- **Rationale do plano:** "O módulo de cronograma atual foca apenas no aspecto físico das obras, sem integração com o aspecto financeiro. Isso dificulta o controle de custos, previsão de fluxo de caixa e análise de desempenho financeiro das obras."

---

## D16 — Uso de service_role apenas server-side `[VERIFICADO]`

- **Data:** definido em `SECURITY.md` e `src/lib/supabase-admin.ts`.
- **Decisão:** `SUPABASE_SERVICE_ROLE_KEY` pode ser usada somente em contexto servidor; arquivo `src/lib/supabase-admin.ts` criado com essa intenção.
- **Regra explícita:** "Não use `SUPABASE_SERVICE_ROLE_KEY` no frontend."
- **Regra explícita:** "Não desabilite RLS como atalho."
- **Lacuna:** o arquivo tem fallback para `publishable_key` se service_role vazio (S3 em `09-limitacoes-problemas.md`).

---

## D17 — Bearer token do webhook n8n `[VERIFICADO]`

- **Data:** `[INFERIDO]` inicial.
- **Decisão:** endpoint `/api/financeiro/importar` autentica via bearer token compartado com variável de ambiente `N8N_IMPORT_API_KEY`.
- **Rationale:** permitir que n8n (não-usuario) acesse o endpoint sem sessão Supabase Auth.
- **Risco crítico:** token hardcoded no JSON versionado do workflow n8n (`n8n-b2a-secret-key-2026`).
- **Lacuna:** comparação com `===` (não constant-time).

---

## Dependências

- Este documento depende de: `08-estado-implementacao.md`, `09-limitacoes-problemas.md`.
- Atualizações neste documento podem afetar: nenhum outro diretamente (documento de referência histórica).
