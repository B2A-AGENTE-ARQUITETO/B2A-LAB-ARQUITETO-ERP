# 02 — Módulos e Funcionalidades

**Última verificação:** 2026-11-24 · **Fonte:** `Catalogo-Capacidades-ERP-B2A-Engenharia.md` seção 2.3

## Sumário

O ERP possui 9 módulos de negócio implementados, organizados como rotas Next.js App Router. Esta página lista cada módulo, sua função e referência ao código.

## Rotas ativas `[VERIFICADO]`

| Rota | Função | Arquivo principal |
|---|---|---|
| `/` | Landing page com 4 cards (Administrativo, Obras, Financeiro, Vendas — os 2 últimos "Em breve") | `src/app/page.tsx` |
| `/dashboard` | Painel principal com obras | `src/app/dashboard/page.tsx` |
| `/obras` | CRUD de obras | `src/app/obras/` |
| `/obras/[id]` | Detalhe de obra | `src/app/obras/[id]/` |
| `/cronograma` | Etapas e marcos | `src/app/cronograma/` |
| `/contratos` | Contratos de empreiteiros + itens | `src/app/contratos/` |
| `/medicoes` | Medições por contrato + itens + acumulado | `src/app/medicoes/` |
| `/pendencias` | Pendências técnicas com checklist e criticidade | `src/app/pendencias/` |
| `/compras` | Solicitações → Cotações → Mapa → Pedidos | `src/app/compras/` |
| `/empreiteiros` | CRUD de empreiteiros + PIX | `src/app/empreiteiros/` |
| `/financeiro` | Contas a pagar com PDF + dashboard consolidado | `src/app/financeiro/` |
| `/financeiro/conciliacao` | Fila de conciliação (approve/reject) | `src/app/financeiro/conciliacao/` |
| `/admin/classificacoes` | Categorias financeiras | `src/app/admin/classificacoes/` |
| `/admin/configuracoes` | Configurações gerais | `src/app/admin/configuracoes/` |
| `/admin/tipos-servico` | Tipos de serviço + atividades padrão | `src/app/admin/tipos-servico/` |
| `/login` | Autenticação | `src/app/login/` |
| `/reset-password` | Recuperação de senha | `src/app/reset-password/` |
| `/debug` | Diagnóstico | `src/app/debug/` |

## Features isoladas (feature folders) `[VERIFICADO]`

Definidas em `ESTRUTURA_PADRAO_DESENVOLVIMENTO.md`: novas funcionalidades devem ser encapsuladas em `src/features/<nome>/` com subpastas `components/`, `hooks/`, `api/`, `lib/`, `types/`.

| Feature | Caminho | Função |
|---|---|---|
| `compras-workflow` | `src/features/compras-workflow/` | Fluxo completo de compras (solicitação → cotação → mapa → pedido) |
| `contas-a-pagar` | `src/features/contas-a-pagar/` | Contas a pagar com PDF, dashboard, lista de lançamentos |
| `financeiro-conciliacao` | `src/features/financeiro-conciliacao/` | Fila de conciliação financeira |

## Módulo: Obras `[VERIFICADO]`

**Função:** gestão de obras de construção civil.

**Tipo principal:** `Obra` em `src/lib/types.ts`:
- `id_obra: string` (UUID)
- `nome_obra: string`
- `data_inicio_prevista: string` (YYYY-MM-DD)
- `prazo_dias: number`
- `data_fim_prevista: string`
- `status_obra: StatusObra` — enum: `Planejada | Em Execução | Concluída | Encerrada | Cancelada`
- `nome_proprietario: string`
- `gerente_obra?: string`
- `risco_obra: RiscoObra` — enum: `Normal | Atenção | Crítico`
- `percentual_fisico_atual: number` (0–100)
- `custo_planejado: number`
- `endereco?: string`
- `email_vinculado?: string` (desde migration `20260325`)
- `email_obra?: string`
- `pasta_drive_id?: string` (Google Drive)

**Storage methods:** `getObras()`, `saveObra()`.

**Validação:** `ObraSchema` em `src/lib/schemas.ts` com Zod (nome_obra obrigatório, email_vinculado valida formato, percentual_fisico 0–100, custo_planejado ≥ 0).

**Migrations relevantes:**
- `20260325_obra_email_drive.sql` — adiciona email_vinculado, email_obra, pasta_drive_id.
- `20260920_obra_orcamento_etapas.sql` — cria `obra_orcamento_etapas` (14 etapas padrão B2A) + `obra_curva_s`.

## Módulo: Cronograma `[VERIFICADO]`

**Função:** etapas e marcos do cronograma físico.

**Tipo principal:** `CronogramaEtapa`:
- `id_etapa`, `id_obra`, `id_obra_tipo_servico`
- `id_equipe_responsavel?`
- `descricao_etapa`, `data_inicio_planejada`, `prazo_dias_uteis`, `data_fim_planejada`
- `data_inicio_real?`, `data_fim_real?`
- `percentual_execucao: number`
- `status: StatusEtapa` — enum: `Planejado | Em Execução | Atrasado | Concluído`
- `peso_na_obra?`
- `eh_marco?: boolean` (desde migration `20261104`)
- `nome_marco?: string`
- `ordem_marco?: integer`

**Migrations relevantes:**
- `20261104_cronograma_marcos.sql` — adiciona `eh_marco`, `nome_marco`, `ordem_marco` com index `(id_obra, eh_marco, ordem_marco)`.

## Módulo: Contratos `[VERIFICADO]`

**Função:** contratos de empreiteiros e seus itens.

**Tipos principais:**
- `ContratoEmpreiteiro`: `id_contrato`, `numero_contrato`, `id_obra`, `id_empreiteiro`, `id_tipo_servico`, `data_inicio`, `data_fim`, `tipo_contrato` (`Unitário | Global`), `valor_total_contratado`, `valor_total_medido`, `status_contrato` (`Aberto | Em Medição | 100% Medido | Encerrado`).
- `ContratoItem`: `id_item_contrato`, `id_contrato`, `id_atividade_padrao?`, `descricao_item`, `unidade_medida`, `quantidade_contratada`, `valor_unitario?`, `valor_global_item?`, `valor_total_contratado_item`.

**Regras:**
- `numero_contrato` é uppercase + trim antes de salvar.
- Bloqueio de exclusão se há medições vinculadas.

## Módulo: Medições `[VERIFICADO]`

**Função:** medições por contrato com itens, acumulado e saldo.

**Tipos principais:**
- `Medicao`: `id_medicao`, `numero_medicao`, `id_contrato`, `periodo_inicio`, `periodo_fim`, `data_medicao`, `status_medicao` (`Rascunho | Aguardando Pagamento | Paga`), `valor_total`.
- `MedicaoItem`: `id_item_medicao`, `id_medicao`, `id_item_contrato`, `percentual_executado_periodo`, `quantidade_executada_periodo`, `valor_medido_periodo`, `percentual_acumulado`, `quantidade_acumulada`, `valor_acumulado`, `saldo_percentual`, `saldo_valor`.

**Storage:** `getMedicoes()`, `saveMedicao()`, `deleteMedicao()` (cascata para itens), `getMedicaoItens()`, `saveMedicaoItem()`, `getMedicaoItensByContrato()` (filtra `status != "Rascunho"`).

## Módulo: Pendências `[VERIFICADO]`

**Função:** pendências técnicas com checklist e criticidade.

**Tipo principal:** `Pendencia`:
- `id_pendencia`, `id_obra`, `id_contrato?`, `id_equipe?`
- `tipo_pendencia: TipoPendencia` — enum: `Técnica | Tarefa combinada | Ajuste | Material | Problema`
- `descricao`, `responsavel`, `data_criacao`, `data_limite`
- `status: StatusPendencia` — enum: `Aberto | Em Análise | Resolvido | Encerrado | Concluída`
- `criticidade: CriticidadePendencia` — enum: `Baixa | Média | Alta`
- `checklists?: Array<{ id, text, done }>`

**Tipo definido mas sem CRUD:** `PendenciaComentario` existe em `types.ts` mas **nenhum método em `storage.ts`** lê ou grava comentários `[VERIFICADO]` — lacuna funcional confirmada.

## Módulo: Compras `[VERIFICADO]`

**Função:** fluxo completo de solicitação → cotação → mapa → pedido.

**Tipos principais:**
- `SolicitacaoMaterial`: `id_solicitacao`, `id_obra`, `urgencia` (`Baixa | Média | Alta | Crítico`), `id_obra_tipo_servico?`, `status_solicitacao` (`Aberta | Em Cotação | Aprovada | Pedido Emitido | Entregue | Cancelada`), `data_criacao`.
- `SolicitacaoMaterialItem`: `id_item_solicitacao`, `id_solicitacao`, `id_material?`, `descricao_item`, `quantidade`, `unidade`.
- `Cotacao`: `id_cotacao`, `id_solicitacao`, `data_abertura`, `status_cotacao` (`Aberta | Em Análise | Concluída`).
- `CotacaoItemFornecedor`: `id_item_fornecedor`, `id_cotacao`, `id_fornecedor`, `descricao_item`, `quantidade`, `unidade`, `valor_unitario`, `valor_total_item`, `observacao?`.
- `PedidoCompra`: `id_pedido`, `numero_pedido`, `id_cotacao`, `id_obra`, `id_fornecedor_escolhido?`, `data_emissao`, `status_pedido` (`Emitido | Entregue Parcial | Entregue Total | Cancelado`), `valor_total`.
- `PedidoCompraItem`: `id_item_pedido`, `id_pedido`, `id_item_cotacao`, `quantidade_pedida`, `valor_unitario`, `created_at`.
- `Fornecedor`: `id_fornecedor`, `nome_fantasia`, `razao_social?`, `cnpj?`, `telefone?`, `email?`, `endereco?`, `pix_tipo?`, `pix_chave?`, `ativo`.

**Detalhes do fluxo:** ver `03-processos.md` seção Compras.

## Módulo: Empreiteiros `[VERIFICADO]`

**Função:** CRUD de empreiteiros.

**Tipo principal:** `Empreiteiro`: `id_empreiteiro`, `nome_fantasia`, `razao_social?`, `cnpj?`, `cpf_responsavel?`, `telefone?`, `pix_tipo?`, `pix_chave?`, `ativo`.

## Módulo: Financeiro `[VERIFICADO]`

**Função:** contas a pagar, conciliação, transações reais, relatórios PDF.

**Tipos principais:**
- `LancamentoFinanceiro`: `id`, `obra_id`, `tipo_lancamento` (`medicao | compra | manual | automatizado`), `origem` (`medicao | pedido_compra | n8n | manual`), `origem_id?`, `fornecedor_id?`, `classificacao_financeira_id?`, `descricao?`, `valor_total`, `numero_parcela`, `total_parcelas`, `valor_parcela`, `data_prevista?`, `data_pagamento?`, `status` (`previsto | pago | conciliacao_pendente`), `numero_nota?`, `link_documento?`, `external_id?`, `created_by?`, `created_at`.
- `TransacaoReal`: `id`, `obra_id`, `email_conta?`, `descricao?`, `fornecedor_id?`, `valor`, `data_pagamento`, `numero_nota?`, `link_documento?`, `external_id?`, `classificacao_financeira_id?`, `created_at`.
- `ConciliacaoFinanceira`: `id`, `obra_id?`, `fornecedor_nome_extraido?`, `valor?`, `data_pagamento?`, `numero_nota?`, `possivel_origem_tipo?` (`medicao | pedido_compra`), `possivel_origem_id?`, `confianca_match?` (0–1), `status` (`pendente | aprovado | rejeitado`), `json_original?`, `created_at`.
- `ClassificacaoFinanceira`: `id_classificacao`, `descricao`, `ativo`.
- `RateioFinanceiro`: `id_referencia`, `id_classificacao`, `percentual`, `valor`.

**Detalhes do fluxo de conciliação:** ver `03-processos.md` seção Conciliação.

## Módulo: Admin `[VERIFICADO]`

**Função:** classificações financeiras, configurações gerais, tipos de serviço.

**Tipos relevantes:**
- `TipoServico`: `id_tipo_servico`, `descricao`, `ativo`.
- `ObraTipoServico`: `id_obra_tipo_servico`, `id_obra`, `id_tipo_servico`, `peso_fisico`.
- `AtividadePadrao`: `id_atividade_padrao`, `id_tipo_servico`, `descricao_atividade`, `unidade_medida`, `ativo`.
- `Equipe`: `id_equipe`, `id_empreiteiro?`, `nome_exibicao`, `ativo`.
- `Material`: `id_material`, `descricao`, `unidade_padrao`, `ativo`.

## Módulos não implementados `[VERIFICADO]`

A landing page em `src/app/page.tsx` mostra 4 cards:
- **Administrativo** — marcado "Em breve" (cursor not-allowed).
- **Gestão de Obras** — ativo.
- **Financeiro** — ativo.
- **Vendas** — marcado "Em breve" (cursor not-allowed).

## Lacunas de conhecimento sobre este domínio `[DESCONHECIDO]`

- Funcionalidades específicas de `/admin/configuracoes` (não analisado em detalhe).
- Conteúdo de `/debug` (não analisado).
- Lista completa de `Material` e `AtividadePadrao` cadastrados em produção.

## Dependências

- Este documento depende de: `01-visao-geral.md`, `05-modelo-dados.md`, `03-processos.md`.
- Atualizações neste documento podem afetar: `08-estado-implementacao.md`.
