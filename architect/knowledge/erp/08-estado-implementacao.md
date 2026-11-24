# 08 — Estado de Implementação

**Última verificação:** 2026-11-24 · **Fonte:** `Catalogo-Capacidades-ERP-B2A-Engenharia.md` seções 2 e 4

## Sumário

Classificação do que está **concluído**, **parcial** e **pendente** no ERP, com base no código analisado (commit `0920bb4`).

---

## Concluído `[VERIFICADO]`

### Módulos de negócio
- ✅ Gestão de Obras (CRUD + validação Zod + inicialização automática de planejamento).
- ✅ Cronograma (etapas + marcos desde 2026-11-04).
- ✅ Contratos de Empreiteiros (CRUD + itens + bloqueio de exclusão com medições).
- ✅ Medições (CRUD + itens + acumulado + saldo).
- ✅ Pendências (CRUD + checklist + criticidade).
- ✅ Compras (fluxo completo: solicitação → cotação → mapa → pedido).
- ✅ Empreiteiros (CRUD + PIX).
- ✅ Fornecedores (CRUD + PIX).
- ✅ Materiais (CRUD).
- ✅ Equipes (CRUD).
- ✅ Tipos de Serviço (CRUD + atividades padrão).
- ✅ Financeiro — Contas a Pagar (lista + dashboard + PDF).
- ✅ Admin — Classificações Financeiras (CRUD).
- ✅ Admin — Configurações.
- ✅ Admin — Tipos de Serviço.
- ✅ Auth (login, reset-password, middleware falha-fechado).

### APIs server-side
- ✅ POST `/api/agent` (Gemini 2.0 Flash com contexto limitado).
- ✅ POST `/api/financeiro/importar` (webhook n8n com bearer + idempotência).
- ✅ POST `/api/financeiro/lancamentos` (updateRateio + delete).

### Integrações
- ✅ Supabase (Auth + PostgreSQL + RLS single-admin).
- ✅ Google Gemini 2.0 Flash (apenas para chat, sem function calling).
- ✅ n8n (workflow de OCR financeiro em produção).
- ✅ Vercel (deploy ativo).
- ✅ Templates WhatsApp (4 templates: mapa, programação, solicitação, ordem pagamento).

### Regras de negócio
- ✅ RN-01 (isolamento por obra em queries financeiras).
- ✅ RN-02 (gatilho financeiro ao emitir pedido).
- ✅ RN-03 (cabeçalho obrigatório em PDF).
- ✅ RN-04 (sequência de IDs alfanuméricos via RPC).
- ✅ RN-05 (validação Zod em 5 entidades críticas).
- ✅ RN-06 (cache em memória de 5 min).
- ✅ RN-07 (RLS single-admin).
- ✅ RN-08 (bearer token webhook n8n — com ressalva de segurança).
- ✅ RN-09 (capitalização forçada).
- ✅ RN-10 (middleware falha-fechado).
- ✅ RN-11 (rateio por referência).
- ✅ RN-12 (inicialização automática de planejamento).

### Workflows n8n
- ✅ OCR financeiro (Gmail → anexos → parser regex → POST ERP).
- ✅ Criação de obra via Google Form → Sheets (legacy).

---

## Parcial `[VERIFICADO]`

### Funcionalidades parcialmente implementadas

- ⚠️ **Conciliação financeira automática:** tabela `conciliacao_financeira` existe com campos `possivel_origem_tipo`, `possivel_origem_id`, `confianca_match`, mas **nenhum código os preenche**. Toda conciliação é manual.
- ⚠️ **Match de fornecedor por CNPJ:** usa `ilike %nome%` em `nome_fantasia` (frágil para homônimos). Não usa CNPJ.
- ⚠️ **Workflow n8n OCR:** só processa `tipo_documento == 'RECIBO'`. NFs e Boletos são ignorados.
- ⚠️ **Parser de documentos:** extrai `forma_pagamento` mas **não envia ao ERP**.
- ⚠️ **Memória do Agente IA:** histórico é passado manualmente em cada chamada (até 20 mensagens); sem persistência entre sessões.
- ⚠️ **Cache de storage.ts:** invalidação é global (`cacheInvalidate()` limpa tudo), não seletiva.
- ⚠️ **Validação Zod:** apenas 5 entidades têm schema; demais fazem upsert direto.
- ⚠️ **PDF de relatório:** geração é client-side (jsPDF); não há armazenamento server-side.
- ⚠️ **`created_by` em `lancamentos_financeiros`:** campo existe mas nunca é preenchido.

### Documentos que descrevem funcionalidades não implementadas
- ⚠️ `financial_agent_prompt.md`: descreve lógica de categorização NF/Boleto que deveria estar num agente IA, mas o n8n atual usa parser determinístico por regex.
- ⚠️ `plano_integracao_cronograma_financeiro.md`: descreve EVM (Earned Value Management) com EV/PV/AC, orçamento por etapa, curva S — **não implementado**. A migration `obra_orcamento_etapas` sugere que parte foi feita, mas EVM não está no código.

---

## Pendente `[VERIFICADO]`

### Funcionalidades previstas mas não implementadas

- ❌ **`PendenciaComentario`:** tipo definido em `types.ts` mas **sem CRUD em `storage.ts`**.
- ❌ **Match automático transação ↔ lançamento previsto** (conciliação automática).
- ❌ **Integração cronograma ↔ financeiro (EVM)**.
- ❌ **Cálculo automático de `status` do cronograma no banco** (trigger PostgreSQL).
- ❌ **Notificação de pendências vencidas** (sem scheduler/cron).
- ❌ **Notificação de expiração de cotação** (sem scheduler).
- ❌ **Captura de resposta WhatsApp** (hoje é só `window.open`).
- ❌ **Workflow de aprovação multi-nível** (mediação, pedido, financeiro).
- ❌ **Anexo de fotos/evidências em pendências** (sem tabela de mídia).
- ❌ **Geração server-side de PDFs**.
- ❌ **Armazenamento de PDFs no Supabase Storage**.
- ❌ **Templates customizáveis por cliente**.
- ❌ **Validação de saldo de obra antes de emitir pedido**.
- ❌ **Histórico de reajustes contratuais** (tabela inexistente).
- ❌ **Rate limiting nos Route Handlers**.
- ❌ **CSP estrito** (não verificado em detalhe).
- ❌ **Testes automatizados** (nenhum framework configurado).

### Módulos marcados "Em breve" na landing page `[VERIFICADO]`
- ❌ **Administrativo** (gestão corporativa, RH, processos internos).
- ❌ **Vendas** (leads, propostas, contratos de vendas, CRM).

---

## Lacunas de conhecimento sobre este domínio `[DESCONHECIDO]`

- Funcionalidades específicas de `/admin/configuracoes` (não analisado em detalhe).
- Conteúdo de `/debug` (não analisado).
- Estado real de migrations remotas no Supabase (não versionadas no Git).
- Funcionalidades administrativas em produção que podem não estar no código.

---

## Dependências

- Este documento depende de: `02-modulos.md`, `03-processos.md`, `09-limitacoes-problemas.md`.
- Atualizações neste documento podem afetar: `10-historico-decisoes.md`.
