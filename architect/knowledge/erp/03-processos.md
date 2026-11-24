# 03 — Processos Operacionais

**Última verificação:** 2026-11-24 · **Fonte:** `Catalogo-Capacidades-ERP-B2A-Engenharia.md` seção 4

## Sumário

10 processos operacionais identificados no ERP, cada um com fluxo, regras, intervenção humana, lacunas e oportunidades de automação.

---

## Processo 1: Gestão de Obras `[VERIFICADO]`

### Implementação
- Rotas `/obras` e `/obras/[id]`.
- `storage.getObras()`, `storage.saveObra()`.
- Validação Zod em `ObraSchema`.
- `storage.inicializarPlanejamentoObra(obra_id, dataInicio, dataFim, valorTotalOrcado)`: gera automaticamente 14 etapas orçamentárias com percentuais fixos B2A + curva S mensal linear.

### Percentuais das 14 etapas padrão B2A `[VERIFICADO]`
| # | Etapa | Percentual |
|---|---|---|
| 1 | Serviços Preliminares | 3.59% |
| 2 | Muro de Arrimo e Divisa | 4.63% |
| 3 | Fundações | 4.00% |
| 4 | Superestrutura | 9.03% |
| 5 | Alvenaria / Fechamento | 4.05% |
| 6 | Esquadrias Alumínio / Vidro / Madeira | 8.14% |
| 7 | Cobertura / Estrutura Metálica | 6.95% |
| 8 | Instalações Hidráulicas | 6.69% |
| 9 | Instalações Elétricas/Lógica | 4.17% |
| 10 | Impermeabilização | 0.85% |
| 11 | Revestimento de Piso / Parede / Teto | 11.31% |
| 12 | Pintura | 1.90% |
| 13 | Serviços Complementares | 5.60% |
| 14 | Mão de Obra | 29.08% |

### Intervenção humana obrigatória
- Criação de obra (define orçamento total, datas, responsável).
- Mudança de status para `Concluída` ou `Encerrada`.
- Atribuição de `risco_obra = Crítico`.

---

## Processo 2: Cronograma Físico e Marcos `[VERIFICADO]`

### Implementação
- Rota `/cronograma`.
- `storage.getCronogramaEtapas()`, `storage.saveCronogramaEtapa()`.
- Migration `20261104_cronograma_marcos.sql` adiciona `eh_marco`, `nome_marco`, `ordem_marco`.

### Regras
- `status` deriva de `percentual_execucao` + datas reais (cálculo no frontend, não no banco).

### Lacunas
- Cálculo automático de `status` no banco (trigger PostgreSQL) — **não implementado** `[VERIFICADO]`.
- Alertas de atraso (sem notificação) `[VERIFICADO]`.
- Integração com financeiro (EVM) prevista em `plano_integracao_cronograma_financeiro.md` mas **não implementada** `[VERIFICADO]`.

---

## Processo 3: Contratos de Empreiteiros `[VERIFICADO]`

### Implementação
- `storage.getContratos()`, `saveContrato()`, `deleteContrato()` (impede exclusão se há medições), `getContratoItens()`, `saveContratoItem()`.

### Regras
- `numero_contrato` é uppercase + trim.
- Cálculo de `valor_total_medido` (acumulado de medições).
- Bloqueio de exclusão se há medições vinculadas.

---

## Processo 4: Medições de Empreiteiros `[VERIFICADO]`

### Implementação
- `storage.getMedicoes()`, `saveMedicao()`, `deleteMedicao()` (cascata), `getMedicaoItens()`, `saveMedicaoItem()`, `getMedicaoItensByContrato()` (filtra `status != "Rascunho"`).

### Regras
- Cálculo de acumulado (soma de medições anteriores) e saldo (`100% - acumulado`).
- Transição de status: `Rascunho → Aguardando Pagamento → Paga`.
- `marcarLancamentoPago(origem_id, 'medicao')` ao confirmar pagamento.

### Lacunas
- Cálculo automático de `valor_total` (soma dos itens) — hoje é informado `[VERIFICADO]`.
- Validação de `percentual_acumulado <= 100` no banco `[VERIFICADO]`.
- Workflow de aprovação multi-nível `[VERIFICADO]`.

---

## Processo 5: Pendências e Checklists `[VERIFICADO]`

### Implementação
- `storage.getPendencias()` (ordenado por `data_criacao` desc), `savePendencia()`, `deletePendencia()`.
- Validação Zod em `PendenciaSchema`.

### Lacuna crítica
- `PendenciaComentario` definido em `types.ts` mas **sem CRUD em `storage.ts`** `[VERIFICADO]`.
- Notificação de pendências vencidas (sem scheduler) `[VERIFICADO]`.
- Anexo de fotos/evidências (sem tabela de mídia) `[VERIFICADO]`.

---

## Processo 6: Compras (Solicitação → Cotação → Mapa → Pedido) `[VERIFICADO]`

Este é o **processo mais rico do ERP** e o melhor candidato a automação por agentes.

### Mapa de Cotação (`src/features/compras-workflow/lib/cotacaoMap.ts`) `[VERIFICADO]`

Função pura `buildMapaCotacao({ itensSolicitacao, itensCotacao, fornecedores })`.

**Regras críticas:**
- Normaliza descrições (remove acentos, lowercase) para agrupar ofertas por item.
- Detecta duplicatas idênticas (mesmo fornecedor + item + valores) e consolida em memória.
- Detecta duplicatas divergentes (mesmo fornecedor + item mas valores diferentes) e **bloqueia aprovação** — expõe como `conflitos[]`.
- Calcula propostas completas (fornecedor que cotou todos os itens).
- Calcula economia: `economiaValor = segundaMelhor - melhor` e `economiaPercentual`.
- Retorna `melhorProposta` (menor valor entre as completas).

### Hook central (`src/features/compras-workflow/hooks/useComprasWorkflow.ts`) `[VERIFICADO]`

- `iniciarCotacao(sol, cotacoesExistentes)`: cria `COT-xxxx` via `getNextSequence("COT")`, muda status para `Em Cotação`.
- `confirmarPedidos({ cotacao, solicitacao, itensParaPedido, itensCotacao, fornecedores, dataPrevistaPagamento? })`:
  1. Agrupa itens selecionados por fornecedor (permite pedido multi-fornecedor).
  2. Para cada fornecedor: gera `PED-xxxx` via `getNextSequence("PED")`, salva `PedidoCompra`, salva itens (`ITM-PED-xxxx`).
  3. **RN-02**: chama `pedidoToFinanceiro()` que cria lançamento financeiro `previsto` com `origem: 'pedido_compra'` e `origem_id: id_pedido`.
  4. Atualiza `solicitacao.status_solicitacao = "Pedido Emitido"` e `cotacao.status_cotacao = "Concluída"`.
- `marcarPedidoPago(pedido)`: muda status para `Entregue Total`.

### Geração de ID sequencial `[VERIFICADO]`
- RPC `get_next_sequence(text, integer default 4)` — `SECURITY DEFINER` (`supabase/migrations/20260314_fix_sequence_rpc.sql`).
- Revogada execução anônima em `20260819_revoke_anon_sequence_rpc.sql`.

### Templates WhatsApp (`src/features/compras-workflow/lib/whatsappTemplates.ts`) `[VERIFICADO]`

4 templates padronizados:
- **Template A:** `templateMapaCotacao` — envia mapa de cotação para aprovação (com opções `APROVAR N` ou `REVISAR`).
- **Template B:** `templateProgramacaoPagamento` — aviso de vencimento/boleto.
- **Template C:** `templateSolicitacaoCotacao` — solicitação de cotação para fornecedor.
- **Template D:** `templateOrdemPagamento` — ordem de pagamento com PIX.

Função `abrirWhatsApp(message, telefone?)` — abre `https://wa.me/...` (apenas `window.open`, sem callback).

### Intervenção humana obrigatória
- Aprovação do mapa de cotação pelo cliente (template WhatsApp com opções explícitas).
- Emissão do pedido (somente após confirmação).
- Confirmação de entrega do pedido.

### Lacunas
- Captura da resposta do WhatsApp (hoje é só `window.open` — sem callback) `[VERIFICADO]`.
- Notificação de expiração de cotação (sem scheduler) `[VERIFICADO]`.
- Validação de saldo de obra antes de emitir pedido `[VERIFICADO]`.
- Auditoria: quem aprovou, quando, com qual argumento `[VERIFICADO]`.

---

## Processo 7: Financeiro — Conciliação de Transações `[VERIFICADO]`

### Transações reais (origem n8n)
- Tabela `transacoes_reais` (migration `20260920_transacoes_reais.sql`).
- `storage.getTransacoesReais(obra_id, email_obra?)` com `OR` query: `obra_id.eq.X OR email_conta.ilike.Y`.
- `storage.saveTransacaoReal()` e `deleteTransacaoReal()` para rateio manual.

### Lançamentos previstos (origem pedidos/medições)
- Tabela `lancamentos_financeiros` (migration `20260323_financial_module.sql`).
- `storage.saveLancamentoFinanceiro()` com **idempotência dupla**: por `(origem, origem_id)` para escritas internas e por `external_id` para escritas n8n.
- `storage.marcarLancamentoPago(origem_id, origem)`.

### Conciliação
- Tabela `conciliacao_financeira` (migration `20260323_financial_module.sql`).
- `storage.getConciliacaoFinanceira(obra_id)` — retorna pendentes.
- `storage.approveConciliacao(id, lancamento_id)`: marca lançamento como `pago` + conciliação como `aprovado` + invalida cache.
- `storage.rejectConciliacao(id)`: apenas marca como `rejeitado`.

### Route handlers `[VERIFICADO]`
- `POST /api/financeiro/importar` — webhook n8n (ver `06-apis-integracoes.md`).
- `POST /api/financeiro/lancamentos` — updateRateio / delete.

### Lacuna crítica
- `conciliacao_financeira` tem campos `possivel_origem_tipo`, `possivel_origem_id`, `confianca_match` mas **nenhum código os preenche** `[VERIFICADO]`.
- Match automático transação ↔ lançamento previsto **não acontece** — toda conciliação é manual `[VERIFICADO]`.
- Match de fornecedor por `ilike %nome%` em `nome_fantasia` (frágil para homônimos) `[VERIFICADO]`.
- `created_by` em `lancamentos_financeiros` existe mas **nunca é preenchido** `[VERIFICADO]`.

### Intervenção humana obrigatória
- Aprovação de conciliação (`approveConciliacao`).
- Aplicação manual de rateio (classificação financeira).
- Rejeição com justificativa.

---

## Processo 8: Agente IA Conversacional (Gemini) `[VERIFICADO]`

### Implementação
- `POST /api/agent/route.ts`.
- System prompt fixo em português.
- Histórico de conversa até 20 mensagens.
- Sem function calling, sem tools, sem RAG além do contexto limitado.

### Contexto construído para o Gemini
- 5 obras mais recentes (`id_obra, nome_obra, status_obra, custo_planejado, percentual_fisico_atual`).
- 5 lançamentos financeiros mais recentes da obra ativa.
- 5 lançamentos pendentes de conciliação.
- 5 pedidos de compra mais recentes da obra ativa.

### Limitações
- Contexto limitado a 5 entidades por tipo.
- Sem ferramentas (function calling).
- `obraId` opcional; se ausente, queries de lançamentos retornam vazio.

### System prompt fixo `[VERIFICADO]`
```
Você é o Assistente B2A, um agente especializado no ERP de gestão de obras da B2A Engenharia.
Responda em português brasileiro, de forma objetiva, usando somente o contexto fornecido.
Não invente dados.
```

---

## Processo 9: Workflow n8n — OCR Financeiro `[VERIFICADO]`

### Arquivo
`workflow_n8n_financeiro.json` (23 KB).

### Fluxo (14 nodes)
1. Manual trigger.
2. 3× Gmail nodes em paralelo (`email sandro`, `Obras01`, `email obras`) — busca e-mails não lidos com anexos.
3. Merge (3 inputs).
4. Filter `Tem Anexo?`.
5. SplitOut — separa cada anexo em item próprio.
6. Code `Classificar Arquivo` — valida extensão (PDF/JPG/PNG/GIF/BMP/TIFF/WEBP), preserva metadados de e-mail.
7. If `PDF ou Imagem?` — branch por tipo.
8. Extrair Texto PDF ou OCR Tesseract (idioma `por`).
9. Code `Parser sem IA + ID Unico` — **parser determinístico por regex** (sem chamada a LLM):
   - Classifica tipo: `NF | BOLETO | RECIBO | FATURA | OUTRO`.
   - Extrai: `nome_fornecedor`, `valor`, `data_pagamento`, `data_vencimento`, `numero_nota`, `numero_fatura`, `forma_pagamento`.
   - Gera `id` estável: hash base32 de campos-chave.
   - Deduplica por `id`.
   - Marca `status = 'revisar'` se faltar `nome_fornecedor` ou `valor`.
10. Filter `Somente Recibos` — só processa `tipo_documento == 'RECIBO'`.
11. HTTP Request `Enviar ao Supabase (ERP B2A)`:
    - URL: `https://appsite-erp-b2-a-engenharia.vercel.app/api/financeiro/importar`.
    - Headers: `Authorization: Bearer n8n-b2a-secret-key-2026` (exposto no JSON!).
    - Body JSON com campos extraídos + `EMAIL_CONTA`, `EMAIL_DESTINATARIO`, `EMAIL_REMETENTE`, `EXTERNAL_ID`.

### Workflow secundário `wf_healthy_template_v2.json` `[VERIFICADO]`
- Webhook + Google Form (`Form Obra` com 7 campos).
- Code `Validar Dados`: lança erro se `nome_obra` vazio.
- ExecuteWorkflow `WF_CORE_GerarID`.
- Google Sheets append em planilha `BD_GESTAO_OBRAS_V2 / 01_OBRAS`.
- **[INFERIDO]:** workflow legacy — ERP hoje cria obras direto no Supabase; Sheets pode ser mirror.

### Lacunas
- Bearer token hardcoded no JSON versionado (risco de segurança) `[VERIFICADO]`.
- Workflow só processa `tipo_documento == 'RECIBO'` — NFs e Boletos são ignorados `[VERIFICADO]`.
- `forma_pagamento` extraído mas **não enviado ao ERP** `[VERIFICADO]`.
- Match de fornecedor por `ilike %nome%` em `nome_fantasia` (frágil) `[VERIFICADO]`.

---

## Processo 10: Relatórios Financeiros em PDF `[VERIFICADO]`

### Implementação
- `src/features/contas-a-pagar/lib/pdfGenerator.ts` — `gerarRelatorioFinanceiroPDF()`.
- **RN-03**: cabeçalho obrigatório com `nome_obra` (16pt bold), `endereco` (10pt), `id_obra` (8pt), título do relatório, data de emissão.
- KPIs no topo: `Total Previsto`, `Total Pago`, `Pendente Conciliação`.
- Tabela `jspdf-autotable` com colunas: `Vencimento, Descrição, Fornecedor, Parcela, Valor, Status`.
- Rodapé em cada página: `${obra.nome_obra} | Página X de Y | Gerado pelo APPSITE ERP B2A`.
- Cor de marca laranja B2A (`#f97316` = orange-500).
- `src/app/dashboard/page.tsx` e `src/features/contas-a-pagar/components/FinanceiroRelatorioObra.tsx` (54 KB) — UI de geração.
- `src/features/contas-a-pagar/components/FinanceiroObrasDashboard.tsx` — dashboard consolidado.

### Regras
- Isolamento por obra é forçado por `useLancamentosByObra` (RN-01).

### Outputs
- Arquivo `financeiro_<id_obra>.pdf` no cliente (não armazenado no ERP).

### Lacunas
- Geração server-side do PDF (hoje é client-side via jsPDF).
- Armazenamento do PDF no Supabase Storage (sem bucket referenciado).
- Templates customizáveis por cliente.

---

## Dependências

- Este documento depende de: `02-modulos.md`, `04-regras-negocio.md`, `06-apis-integracoes.md`.
- Atualizações neste documento podem afetar: `08-estado-implementacao.md`, `09-limitacoes-problemas.md`.
