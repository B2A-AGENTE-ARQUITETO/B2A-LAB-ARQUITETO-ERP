# 04 — Regras de Negócio

**Última verificação:** 2026-11-24 · **Fonte:** `Catalogo-Capacidades-ERP-B2A-Engenharia.md` seção 4

## Sumário

Regras de negócio formais (RN) identificadas no código, com referência aos arquivos que as implementam.

---

## RN-01 — Isolamento de obra em queries financeiras `[VERIFICADO]`

**Enunciado:** lançamentos financeiros e transações reais devem ser sempre filtrados por `obra_id`, jamais retornando dados de outras obras.

**Implementação:**
- `storage.getLancamentosFinanceirosByObra(obra_id, status?)` em `src/lib/storage.ts` — query sempre com `.eq("obra_id", obra_id)`.
- `storage.getTransacoesReais(obra_id, email_obra?)` em `src/lib/storage.ts` — query com `OR`: `obra_id.eq.X OR email_conta.ilike.Y`.
- `useLancamentosByObra` hook em `src/features/contas-a-pagar/hooks/useLancamentosByObra.ts` — sempre busca pelo `obraSelecionada.id_obra` do `ObraContext`.

---

## RN-02 — Gatilho financeiro ao emitir pedido de compra `[VERIFICADO]`

**Enunciado:** ao emitir um pedido de compra a partir de uma cotação aprovada, o sistema deve gerar automaticamente lançamento(s) financeiro(s) com status `previsto`.

**Implementação:**
- Função `pedidoToFinanceiro()` em `src/features/compras-workflow/lib/pedidoToFinanceiro.ts`.
- Chamada por `useComprasWorkflow.confirmarPedidos()` em `src/features/compras-workflow/hooks/useComprasWorkflow.ts`.

**Regras do lançamento gerado:**
- `tipo_lancamento: 'compra'`
- `origem: 'pedido_compra'`
- `origem_id: id_pedido`
- `status: 'previsto'`
- Datas no formato YYYY-MM-DD (usando `.split("T")[0]`)
- Parcelas: `numero_parcela` de 1 a `numParcelas`, vencimento calculado como `dataPrevista + (i-1) meses`.

**Idempotência:**
- `storage.saveLancamentoFinanceiro()` com idempotência por `(origem, origem_id)` — se já existe lançamento com `origem='pedido_compra'` e `origem_id=id_pedido`, faz update; caso contrário, insert.

---

## RN-03 — Cabeçalho obrigatório em PDF de relatório financeiro `[VERIFICADO]`

**Enunciado:** a primeira página do PDF de relatório financeiro deve conter cabeçalho com identificação da obra antes de qualquer dado.

**Implementação:** `gerarRelatorioFinanceiroPDF()` em `src/features/contas-a-pagar/lib/pdfGenerator.ts`.

**Layout obrigatório:**
1. Linha decorativa superior laranja (`#f97316`).
2. `nome_obra` — destaque máximo (16pt bold).
3. `endereco` (10pt, normal).
4. `id_obra` (8pt, cinza).
5. Linha separadora.
6. Título do relatório (14pt bold).
7. Data de emissão (8pt, cinza).
8. KPIs (Total Previsto, Total Pago, Pendente Conciliação).
9. Tabela de lançamentos.

**Rodapé em cada página:**
`${obra.nome_obra} | Página X de Y | Gerado pelo APPSITE ERP B2A`

---

## RN-04 — Sequência de IDs alfanuméricos `[VERIFICADO]`

**Enunciado:** IDs de entidades operacionais seguem padrão `<PREFIX>-<NNNN>` com 4 dígitos zero-padded, gerados por RPC.

**Implementação:**
- RPC `get_next_sequence(p_prefix text, p_length integer default 4)` em `supabase/migrations/20260314_fix_sequence_rpc.sql`.
- `SECURITY DEFINER`.
- Revogada execução anônima em `20260819_revoke_anon_sequence_rpc.sql` (apenas `authenticated` e `service_role` podem executar).

**Prefixos em uso `[VERIFICADO]`:**
- `SOL` — Solicitação de material (ex.: `SOL-0005`)
- `COT` — Cotação (ex.: `COT-0012`)
- `PED` — Pedido de compra (ex.: `PED-2026-0042`)
- `MED` — Medição (ex.: `MED-0008`)
- `ITM-PED` — Item de pedido de compra

**Storage method:** `storage.getNextSequence(prefix)`.

---

## RN-05 — Validação Zod em entidades críticas `[VERIFICADO]`

**Enunciado:** entidades críticas têm validação de schema via Zod antes de upsert no banco.

**Schemas definidos em `src/lib/schemas.ts`:**

| Schema | Campos validados |
|------|
| `ObraSchema` | `nome_obra` obrigatório, `email_vinculado` valida formato, `percentual_fisico_atual` 0–100, `custo_planejado` ≥ 0 |
| `PendenciaSchema` | `descricao` obrigatória, `criticidade` enum, `checklists` array tipado |
| `ContratoEmpreiteiroSchema` | `numero_contrato`, `valor_total_contratado` ≥ 0, enums de tipo e status |
| `FornecedorSchema` | `nome_fantasia` obrigatório, `email` valida formato se presente |
| `NovoContratoSchema` | `id_obra`, `id_empreiteiro`, `id_tipo_servico`, datas obrigatórios |

**Entidades sem validação Zod (upsert direto):** medições, itens de medição, cotações, itens de cotação, pedidos, itens de pedido, etapas de cronograma, equipes, materiais, empreiteiros, tipos de serviço, classificações financeiras, rateios, transações reais, conciliações.

---

## RN-06 — Cache em memória de 5 minutos para leituras `[VERIFICADO]`

**Enunciado:** leituras de tabelas frequentes são cacheadas em memória por 5 minutos para reduzir latência e custo de queries Supabase.

**Implementação:** `src/lib/storage.ts`:
```typescript
const CACHE_TTL = 5 * 60 * 1000; // 5 minutes
```

**Tabelas com cache:**
- `contratos_empreiteiros`
- `medicoes`
- `solicitacoes_material` (por obra)
- `tipos_servico`
- `atividades_padrao`
- `empreiteiros`
- `classificacoes_financeiras`
- `cotacoes` (por obra)
- `lancamentos_financeiros` (consolidado e por obra)
- `transacoes_reais` (por obra e consolidado)
- `obra_orcamento_etapas`
- `obra_curva_s`

**Invalidação:**
- `cacheInvalidate(...keys)` limpa **todo o cache** (não chaves específicas) `[VERIFICADO]` — potencial race condition em ambientes multi-agente.

---

## RN-07 — RLS Single-Admin `[VERIFICADO]`

**Enunciado:** acesso ao banco restrito a um único usuário Supabase Auth, com allowlist explícita.

**Implementação:**
- Migration `supabase/migrations/20260818_single_admin_rls.sql`:
  - Cria `public.erp_admin_users (user_id uuid primary key references auth.users(id))`.
  - `private.is_erp_admin()` function `SECURITY DEFINER` que verifica se `auth.uid()` está na allowlist.
  - Aplica policy `erp_admin_only` em todas as tabelas públicas: `USING ((select private.is_erp_admin())) WITH CHECK ((select private.is_erp_admin()))`.
  - Revoga `all` de `anon` e `authenticated`; concede `select, insert, update, delete` a `authenticated`.
  - **Aborta se não houver exatamente 1 usuário em `auth.users`.**

**Implicações:**
- Impossível ter identidade distinta para agentes externos sem migration de papéis.
- Criar um segundo usuário exige antes uma migration que substitua o modelo de admin único.

---

## RN-08 — Bearer token para webhook n8n `[VERIFICADO]`

**Enunciado:** o endpoint `/api/financeiro/importar` aceita apenas requisições com `Authorization: Bearer <N8N_IMPORT_API_KEY>` válido.

**Implementação:** `src/app/api/financeiro/importar/route.ts`:

**Lacuna de segurança:** comparação com `===` (não constant-time) — vulnerável a timing attack `[VERIFICADO]`.

**Lacuna de segurança crítica:** bearer token hardcoded no JSON versionado: `"Bearer n8n-b2a-secret-key-2026"` em `workflow_n8n_financeiro.json` `[VERIFICADO]`.

---

## RN-09 — Capitalização forçada `[VERIFICADO]`

**Enunciado:** campos textuais livres são capitalizados antes de salvar.

**Implementação:** `src/lib/storage.ts`:
- `nome_obra`: `capitalize(validated.nome_obra)`
- `descricao` (pendência): `capitalize(validated.descricao)`
- `numero_contrato`: `validated.numero_contrato.toUpperCase().trim()`
- `nome_exibicao` (equipe), `descricao` (material, tipo serviço, atividade padrão, empreiteiro, fornecedor, classificação financeira): `capitalize()`

---

## RN-10 — Middleware falha-fechado `[VERIFICADO]`

**Enunciado:** se Supabase Auth não está disponível, o middleware bloqueia acesso a todas as rotas não públicas.

**Implementação:** `src/middleware.ts`:
- `publicPaths = ['/login', '/auth/callback', '/reset-password']`
- Exceção deliberada: `/api/financeiro/importar` é tratado pelo próprio route handler (bearer token).
- Se `process.env.NEXT_PUBLIC_SUPABASE_URL` ou chave publishable não configurados: retorna 503.
- Se `supabase.auth.getClaims()` falha ou `userId` é null:
  - Em rotas `/api/*`: retorna 401.
  - Em páginas não públicas: redireciona para `/login`.

---

## RN-11 — Rateio financeiro por referência `[VERIFICADO]`

**Enunciado:** rateios financeiros são associados a uma referência (`id_referencia`) que pode ser `id_solicitacao`, `id_pedido`, `id_medicao`, etc.

**Implementação:**
- `storage.saveRateioFinanceiro(rateio)` em `src/lib/storage.ts`.
- `storage.getRateiosByReferencia(id_referencia)`.
- `storage.deleteRateiosByReferencia(id_referencia)` (usado ao excluir solicitação, por exemplo).

**Tipo:** `RateioFinanceiro { id_referencia, id_classificacao, percentual, valor }`.

**Uso confirmado:** quando um pedido é emitido a partir de uma solicitação com rateio, o `classificacao_financeira_id` é propagado para o lançamento financeiro gerado (RN-02).

---

## RN-12 — Inicialização automática de planejamento de obra `[VERIFICADO]`

**Enunciado:** ao criar uma obra, o sistema inicializa automaticamente 14 etapas orçamentárias padrão B2A + curva S mensal linear.

**Implementação:** `storage.inicializarPlanejamentoObra(obra_id, dataInicio, dataFim, valorTotalOrcado)`:
- Insere 14 etapas com percentuais fixos (ver `03-processos.md` Processo 1).
- Gera meses entre `dataInicio` e `dataFim`.
- Distribui `valorTotalOrcado` linearmente entre os meses (`valorMensal = valorTotalOrcado / months.length`).
- Round a 2 casas.

**Tabelas envolvidas:**
- `obra_orcamento_etapas` (migration `20260920_obra_orcamento_etapas.sql`)
- `obra_curva_s` (mesma migration)

---

## Dependências

- Este documento depende de: `02-modulos.md`, `05-modelo-dados.md`, `06-apis-integracoes.md`.
- Atualizações neste documento podem afetar: `09-limitacoes-problemas.md`.
