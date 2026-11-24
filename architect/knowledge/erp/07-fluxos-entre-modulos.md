# 07 — Fluxos entre Módulos

**Última verificação:** 2026-11-24 · **Fonte:** `Catalogo-Capacidades-ERP-B2A-Engenharia.md` seções 3 e 4

## Sumário

Fluxos ponta-a-ponta que cruzam múltiplos módulos do ERP. Cada fluxo tem diagrama textual, regras e pontos de intervenção.

---

## Fluxo 1: Criação de Obra → Planejamento Automático `[VERIFICADO]`

```
[Usuário cria obra em /obras]
  ↓
storage.saveObra(validated) [validação Zod ObraSchema]
  ↓
storage.inicializarPlanejamentoObra(obra_id, dataInicio, dataFim, valorTotalOrcado)
  ├─ Insere 14 etapas em obra_orcamento_etapas (percentuais fixos B2A)
  └─ Gera N meses em obra_curva_s (distribuição linear)
  ↓
[Obra pronta para uso em /obras/[id]]
```

**Pontos de intervenção humana:**
- Definição de orçamento total, datas e responsável.
- Atribuição de `risco_obra = Crítico`.

**Lacunas:**
- Soma dos 14 percentuais fixos não é validada (deve bater 100%) — **não verificado** `[DESCONHECIDO]`.

---

## Fluxo 2: Solicitação → Cotação → Mapa → Pedido → Lançamento Financeiro `[VERIFICADO]`

Este é o **fluxo mais rico do ERP**.

```
[1] Solicitação criada em /compras
    storage.saveSolicitacao() → status="Aberta"
    ↓
[2] iniciarCotacao(sol, cotacoesExistentes)
    storage.getNextSequence("COT") → "COT-xxxx"
    storage.saveCotacao(novaCotacao) → status_cotacao="Aberta"
    storage.saveSolicitacao(solUpdate) → status_solicitacao="Em Cotação"
    ↓
[3] Registro de cotações dos fornecedores
    storage.saveCotacaoItemFornecedor() para cada fornecedor × item
    ↓
[4] Geração do mapa de cotação
    buildMapaCotacao({ itensSolicitacao, itensCotacao, fornecedores })
    → normaliza descrições
    → detecta duplicatas idênticas (consolida) e divergentes (bloqueia)
    → calcula propostas completas e economia
    → retorna melhorProposta
    ↓
[5] Envio do mapa para aprovação (WhatsApp template A)
    templateMapaCotacao({ obra, idCotacao, idSolicitacao, urgencia, mapa })
    abrirWhatsApp(message, telefone)
    ↓
[6] Aprovação humana (APROVAR N via WhatsApp)
    [HOJE: captura manual — sem callback automatizado]
    ↓
[7] confirmarPedidos({ cotacao, solicitacao, itensParaPedido, itensCotacao, fornecedores, dataPrevistaPagamento? })
    Para cada fornecedor selecionado:
      storage.getNextSequence("PED") → "PED-xxxx"
      storage.savePedidoCompra(novoPedido) → status_pedido="Emitido"
      Para cada item:
        storage.getNextSequence("ITM-PED")
        storage.savePedidoCompraItem()
      ── RN-02: Gatilho Financeiro ──
      pedidoToFinanceiro({ pedido, nomeFornecedor, classificacaoFinanceiraId, dataPrevista, numParcelas })
        → gera 1+ lançamentos com:
          - tipo_lancamento="compra"
          - origem="pedido_compra"
          - origem_id=id_pedido
          - status="previsto"
        storage.saveLancamentoFinanceiro(lanc)
    storage.saveSolicitacao(solUpdate) → status_solicitacao="Pedido Emitido"
    storage.saveCotacao(cotUpdate) → status_cotacao="Concluída"
    ↓
[8] Marcação de pedido pago/entregue
    marcarPedidoPago(pedido) → status_pedido="Entregue Total"
    [HOJE: não chama marcarLancamentoPago — lacuna confirmada]
```

**Pontos de intervenção humana:**
- Aprovação do mapa de cotação (passo 6).
- Emissão do pedido (passo 7).
- Confirmação de entrega (passo 8).

**Lacunas:**
- Captura da resposta do WhatsApp (sem callback) `[VERIFICADO]`.
- `marcarPedidoPago` **não chama** `marcarLancamentoPago` — lançamento previsto não é atualizado para `pago` automaticamente `[VERIFICADO]`.
- Validação de saldo de obra antes de emitir pedido `[VERIFICADO]`.

---

## Fluxo 3: Contrato → Medição → Lançamento Financeiro `[VERIFICADO]`

```
[1] Contrato criado em /contratos
    storage.saveContrato() [validação Zod ContratoEmpreiteiroSchema]
    storage.saveContratoItem() para cada item
    ↓
[2] Medição criada em /medicoes
    storage.saveMedicao() → status_medicao="Rascunho"
    storage.saveMedicaoItem() para cada item
      → calcula percentual_executado_periodo
      → calcula acumulado via getMedicaoItensByContrato()
      → calcula saldo = 100% - acumulado
    ↓
[3] Aprovação da medição
    storage.saveMedicao(update) → status_medicao="Aguardando Pagamento"
    [HOJE: storage.saveLancamentoFinanceiro() com origem="medicao"?
     — INFERIDO mas não confirmado no código lido]
    ↓
[4] Confirmação de pagamento
    storage.saveMedicao(update) → status_medicao="Paga"
    storage.marcarLancamentoPago(origem_id=id_medicao, origem="medicao")
      → update lancamentos_financeiros set status="pago", data_pagamento=hoje
```

**Pontos de intervenção humana:**
- Lançamento manual do `% executado` por item.
- Aprovação de medição (`Rascunho → Aguardando Pagamento`).
- Confirmação de pagamento recebido.

**Lacunas:**
- Cálculo automático de `valor_total` (soma dos itens) — hoje é informado `[VERIFICADO]`.
- Validação de `percentual_acumulado <= 100` no banco `[VERIFICADO]`.
- Workflow de aprovação multi-nível `[VERIFICADO]`.
- Integração com comprovante de pagamento (`link_documento`) `[VERIFICADO]`.

---

## Fluxo 4: n8n → ERP → Conciliação `[VERIFICADO]`

```
[1] n8n monitora 3 caixas Gmail (não lidos com anexos)
    ↓
[2] Filtra anexos PDF/imagem (extensões suportadas: pdf, jpg, jpeg, png, gif, bmp, tiff, webp)
    ↓
[3] Branch: PDF → extractFromFile operation=pdf
          Imagem → OCR Tesseract (idioma "por")
    ↓
[4] Parser determinístico por regex (sem IA):
    - classifica tipo_documento: NF | BOLETO | RECIBO | FATURA | OUTRO
    - extrai: nome_fornecedor, valor, data_pagamento, data_vencimento,
              numero_nota, numero_fatura, forma_pagamento
    - gera id estável (hash base32)
    - deduplica por id
    - marca status="revisar" se faltar nome_fornecedor ou valor
    ↓
[5] Filter: somente tipo_documento="RECIBO" prossegue
    [NFs e Boletos são IGNORADOS — lacuna confirmada]
    ↓
[6] HTTP POST para /api/financeiro/importar
    Authorization: Bearer n8n-b2a-secret-key-2026
    Body JSON com campos extraídos + EMAIL_CONTA + EXTERNAL_ID
    ↓
[7] ERP processa importação (financialService.processImport)
    - resolve obra_id por OBRA_ID direto ou email_conta/destinatario/remetente
    - idempotência por external_id em transacoes_reais
    - resolve fornecedor_id por ilike %nome% em nome_fantasia
    - INSERT em transacoes_reais
    - retorna { action: "created" | "ignored_duplicate", id, obra_id }
    ↓
[8] Operador acessa /financeiro/conciliacao
    storage.getConciliacaoFinanceira(obra_id) → lista pendentes
    [HOJE: conciliacao_financeira NÃO é populada — lacuna crítica]
    [Transações reais aparecem na lista de contas a pagar]
    ↓
[9] Rateio manual (classificação financeira)
    POST /api/financeiro/lancamentos { action: "updateRateio", id, classificacao_financeira_id }
    ↓
[10] Aprovação de conciliação (se houver)
     storage.approveConciliacao(id, lancamento_id)
       → update lancamentos_financeiros set status="pago"
       → update conciliacao_financeira set status="aprovado"
```

**Pontos de intervenção humana:**
- Revisão de documentos com `status = 'revisar'` (fornecedor ou valor não extraídos).
- Revisão de fornecedores homônimos.
- Aprovação de rateio manual.
- Aprovação de conciliação.

**Lacunas críticas:**
- Workflow n8n só processa `tipo_documento == 'RECIBO'` — NFs e Boletos ignorados `[VERIFICADO]`.
- `forma_pagamento` extraído mas não enviado ao ERP `[VERIFICADO]`.
- `conciliacao_financeira` não é populada pelo código atual (campos `possivel_origem_*` e `confianca_match` nunca preenchidos) `[VERIFICADO]`.
- Match de fornecedor por `ilike %nome%` (frágil para homônimos) `[VERIFICADO]`.

---

## Fluxo 5: Geração de Relatório Financeiro PDF `[VERIFICADO]`

```
[1] Operador seleciona obra ativa em /financeiro
    ObraContext atualiza obraSelecionada
    ↓
[2] useLancamentosByObra({ status }) busca lançamentos
    storage.getLancamentosFinanceirosByObra(obra_id, status)
    → query sempre .eq("obra_id", obra_id) [RN-01]
    ↓
[3] Operador clica em "Gerar PDF"
    gerarRelatorioFinanceiroPDF({ obra, lancamentos, fornecedores, empreiteiros })
    ↓
[4] jsPDF constrói documento client-side
    - Cabeçalho obrigatório [RN-03]:
      nome_obra (16pt bold), endereco (10pt), id_obra (8pt),
      título, data de emissão
    - KPIs: Total Previsto, Total Pago, Pendente Conciliação
    - Tabela: Vencimento, Descrição, Fornecedor, Parcela, Valor, Status
    - Rodapé em cada página
    ↓
[5] doc.save(`financeiro_${obra.id_obra}.pdf`)
    [Download no cliente — não armazenado no ERP]
```

**Lacunas:**
- Geração server-side do PDF (hoje é client-side) `[VERIFICADO]`.
- Armazenamento no Supabase Storage (sem bucket referenciado) `[VERIFICADO]`.
- Templates customizáveis por cliente `[VERIFICADO]`.

---

## Fluxo 6: Agente IA Conversacional `[VERIFICADO]`

```
[1] Usuário abre /dashboard (ou outra página)
    ↓
[2] Usuário envia mensagem ao Assistente B2A
    POST /api/agent { message, obraId?, history? }
    ↓
[3] getAuthenticatedRequest() valida sessão
    ↓
[4] getERPContext(supabase, obraId) constrói contexto:
    - 5 obras mais recentes
    - 5 lançamentos financeiros da obra ativa
    - 5 lançamentos pendentes de conciliação
    - 5 pedidos de compra da obra ativa
    ↓
[5] GoogleGenerativeAI(GEMINI_KEY).getGenerativeModel({ model: 'gemini-2.0-flash' })
    systemInstruction: SYSTEM_PROMPT + contextPrompt
    ↓
[6] model.startChat({ history }).sendMessage(message)
    ↓
[7] return { reply: text }
```

**Limitações:**
- Sem function calling — não pode executar ações no ERP.
- Memória manual (history[] passado a cada chamada).
- Contexto limitado a 5 entidades por tipo.
- Sem persistência de conversa entre sessões.

---

## Dependências

- Este documento depende de: `02-modulos.md`, `03-processos.md`, `04-regras-negocio.md`, `06-apis-integracoes.md`.
- Atualizações neste documento podem afetar: `08-estado-implementacao.md`.
