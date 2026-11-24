# 05 — Modelo de Dados

**Última verificação:** 2026-11-24 · **Fonte:** `Catalogo-Capacidades-ERP-B2A-Engenharia.md` seção 2.4 + `ESTRUTURA_PADRAO_DESENVOLVIMENTO.md`

## Sumário

Inventário das tabelas referenciadas no código, com migrations conhecidas. **Atenção:** o schema-base do Supabase não está integralmente versionado no Git — o estado real do banco pode conter tabelas, índices e policies adicionais.

---

## Tabelas referenciadas no código `[VERIFICADO]`

| Tabela | Migrations locais | Tipo |
|---|---|---|
| `obras` | `20260325_obra_email_drive.sql` (adiciona campos) | Central |
| `pendencias` | — | Central |
| `contratos_empreiteiros` | — | Central |
| `contrato_itens` | — | Central |
| `medicoes` | — | Central |
| `medicao_itens` | — | Central |
| `solicitacoes_material` | — | Central |
| `solicitacao_itens` | — | Central |
| `cotacoes` | — | Central |
| `cotacao_itens_fornecedor` | — | Central |
| `pedidos_compra` | — | Central |
| `pedido_compra_itens` | — | Central |
| `fornecedores` | — | Central |
| `materiais` | — | Central |
| `empreiteiros` | — | Central |
| `equipes` | — | Central |
| `tipos_servico` | — | Central |
| `obras_tipos_servico` | — | Central |
| `atividades_padrao` | — | Central |
| `cronograma_etapas` | `20261104_cronograma_marcos.sql` (adiciona marcos) | Central |
| `classificacoes_financeiras` | — | Central |
| `rateios_financeiros` | — | Central |
| `lancamentos_financeiros` | `20260323_financial_module.sql` (cria) | Financeiro |
| `conciliacao_financeira` | `20260323_financial_module.sql` (cria) | Financeiro |
| `transacoes_reais` | `20260920_transacoes_reais.sql` (cria), `20260921_transacoes_email_conta.sql` (adiciona email_conta) | Financeiro |
| `obra_orcamento_etapas` | `20260920_obra_orcamento_etapas.sql` (cria) | Planejamento |
| `obra_curva_s` | `20260920_obra_orcamento_etapas.sql` (cria) | Planejamento |
| `obra_emails` | — `[DESCONHECIDO]` | Referenciado em `financial-service.ts` mas sem migration local |
| `erp_admin_users` | `20260818_single_admin_rls.sql` (cria) | RLS |

---

## RPCs `[VERIFICADO]`

### `get_next_sequence(p_prefix text, p_length integer default 4)`

- **Migration:** `20260314_fix_sequence_rpc.sql`.
- **Tipo:** `SECURITY DEFINER`.
- **Retorna:** string no formato `<PREFIX>-<NNNN>` (ex.: `PED-0042`).
- **Permissões:**
  - Antes de `20260819`: executável por `public` (incluindo `anon`).
  - Após `20260819_revoke_anon_sequence_rpc.sql`: apenas `authenticated` e `service_role`.

### `private.is_erp_admin()`

- **Migration:** `20260818_single_admin_rls.sql`.
- **Tipo:** `SECURITY DEFINER`, `STABLE`, `LANGUAGE sql`.
- **Retorna:** `boolean` — `true` se `auth.uid()` está em `public.erp_admin_users`.
- **Usada por:** todas as policies `erp_admin_only` em tabelas públicas.

---

## Schema detalhado de tabelas com migration local `[VERIFICADO]`

### `lancamentos_financeiros` (migration `20260323_financial_module.sql`)

```sql
CREATE TABLE public.lancamentos_financeiros (
    id TEXT PRIMARY KEY DEFAULT gen_random_uuid()::text,
    obra_id TEXT NOT NULL REFERENCES public.obras(id_obra) ON DELETE CASCADE,
    tipo_lancamento TEXT NOT NULL CHECK (tipo_lancamento IN ('medicao', 'compra', 'manual', 'automatizado')),
    origem TEXT NOT NULL CHECK (origem IN ('medicao', 'pedido_compra', 'n8n', 'manual')),
    origem_id TEXT,
    fornecedor_id TEXT,
    classificacao_financeira_id TEXT REFERENCES public.classificacoes_financeiras(id_classificacao),
    descricao TEXT,
    valor_total NUMERIC NOT NULL DEFAULT 0,
    numero_parcela INTEGER NOT NULL DEFAULT 1,
    total_parcelas INTEGER NOT NULL DEFAULT 1,
    valor_parcela NUMERIC NOT NULL DEFAULT 0,
    data_prevista DATE,
    data_pagamento DATE,
    status TEXT NOT NULL DEFAULT 'previsto' CHECK (status IN ('previsto', 'pago', 'conciliacao_pendente')),
    numero_nota TEXT,
    link_documento TEXT,
    created_by TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);
```

**Índices:**
- `idx_lancamentos_obra ON lancamentos_financeiros(obra_id)`
- `idx_lancamentos_origem ON lancamentos_financeiros(origem_id)`

### `conciliacao_financeira` (mesma migration)

```sql
CREATE TABLE public.conciliacao_financeira (
    id TEXT PRIMARY KEY DEFAULT gen_random_uuid()::text,
    obra_id TEXT REFERENCES public.obras(id_obra) ON DELETE SET NULL,
    fornecedor_nome_extraido TEXT,
    valor NUMERIC,
    data_pagamento DATE,
    numero_nota TEXT,
    possivel_origem_tipo TEXT,
    possivel_origem_id TEXT,
    confianca_match NUMERIC,
    status TEXT NOT NULL DEFAULT 'pendente' CHECK (status IN ('pendente', 'aprovado', 'rejeitado')),
    json_original JSONB,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);
```

**Índice:** `idx_conciliacao_status ON conciliacao_financeira(status)`.

### `obra_orcamento_etapas` (migration `20260920_obra_orcamento_etapas.sql`)

```sql
CREATE TABLE public.obra_orcamento_etapas (
    id TEXT PRIMARY KEY DEFAULT gen_random_uuid()::text,
    obra_id TEXT NOT NULL REFERENCES public.obras(id_obra) ON DELETE CASCADE,
    etapa TEXT NOT NULL,
    orcado NUMERIC NOT NULL DEFAULT 0,
    seq_ordem INTEGER NOT NULL DEFAULT 0,
    UNIQUE(obra_id, etapa)
);
```

**Índice:** `idx_orcamento_obra ON obra_orcamento_etapas(obra_id, seq_ordem)`.

### `obra_curva_s` (mesma migration)

```sql
CREATE TABLE public.obra_curva_s (
    id TEXT PRIMARY KEY DEFAULT gen_random_uuid()::text,
    obra_id TEXT NOT NULL REFERENCES public.obras(id_obra) ON DELETE CASCADE,
    mes_iso TEXT NOT NULL,
    valor_previsto NUMERIC NOT NULL DEFAULT 0,
    UNIQUE(obra_id, mes_iso)
);
```

**Índice:** `idx_curva_s_obra ON obra_curva_s(obra_id, mes_iso)`.

### `transacoes_reais` `[VERIFICADO]`

- Migration criadora: `20260920_transacoes_reais.sql` (não foi lida em detalhe, conteúdo `[DESCONHECIDO]` em nível de DDL).
- Migration adicional: `20260921_transacoes_email_conta.sql` — adiciona coluna `email_conta`.

**Tipo TypeScript correspondente (`src/lib/types.ts`):**
```typescript
interface TransacaoReal {
  id: string;
  obra_id: string | null;
  email_conta?: string | null;
  descricao?: string;
  fornecedor_id?: string;
  valor: number;
  data_pagamento: string;
  numero_nota?: string;
  link_documento?: string;
  external_id?: string;
  classificacao_financeira_id?: string | null;
  created_at: string;
}
```

### `cronograma_etapas` (migration `20261104_cronograma_marcos.sql`)

Adiciona colunas:
```sql
ALTER TABLE public.cronograma_etapas
  ADD COLUMN IF NOT EXISTS eh_marco boolean NOT NULL DEFAULT false,
  ADD COLUMN IF NOT EXISTS nome_marco text,
  ADD COLUMN IF NOT EXISTS ordem_marco integer;

CREATE INDEX IF NOT EXISTS cronograma_etapas_marcos_idx
  ON public.cronograma_etapas (id_obra, eh_marco, ordem_marco);
```

### `erp_admin_users` (migration `20260818_single_admin_rls.sql`)

```sql
CREATE TABLE public.erp_admin_users (
  user_id uuid PRIMARY KEY REFERENCES auth.users(id) ON DELETE RESTRICT,
  created_at timestamptz NOT NULL DEFAULT now()
);
ALTER TABLE public.erp_admin_users ENABLE ROW LEVEL SECURITY;
REVOKE ALL ON TABLE public.erp_admin_users FROM anon, authenticated;
```

---

## Tipos TypeScript canônicos `[VERIFICADO]`

Todos os tipos estão em `src/lib/types.ts` (9.4 KB). Resumo:

- `StatusObra`, `RiscoObra`, `Obra`, `ObraEmail`
- `ClassificacaoFinanceira`, `RateioFinanceiro`
- `TipoServico`, `ObraTipoServico`, `Equipe`
- `StatusEtapa`, `CronogramaEtapa`
- `Empreiteiro`, `AtividadePadrao`
- `TipoContrato`, `StatusContrato`, `ContratoEmpreiteiro`, `ContratoItem`
- `StatusMedicao`, `Medicao`, `MedicaoItem`
- `TipoPendencia`, `StatusPendencia`, `CriticidadePendencia`, `Pendencia`, `PendenciaComentario`
- `Material`, `UrgenciaSolicitacao`, `StatusSolicitacao`, `SolicitacaoMaterial`, `SolicitacaoMaterialItem`
- `Fornecedor`, `StatusCotacao`, `Cotacao`, `CotacaoItemFornecedor`
- `StatusPedido`, `PedidoCompra`, `PedidoCompraItem`
- `TipoLancamento`, `OrigemLancamento`, `StatusLancamento`, `LancamentoFinanceiro`
- `StatusConciliacao`, `ConciliacaoFinanceira`
- `TransacaoReal`
- `ObraOrcamentoEtapa`, `ObraCurvaS`

---

## Migrations locais `[VERIFICADO]`

12 arquivos em `supabase/migrations/`:

| Migration | Data | Função |
|---|---|---|
| `20260314_fix_sequence_rpc.sql` | 2026-03-14 | RPC `get_next_sequence` SECURITY DEFINER |
| `20260323_financial_module.sql` | 2026-03-23 | Cria `lancamentos_financeiros` + `conciliacao_financeira` |
| `20260325_obra_email_drive.sql` | 2026-03-25 | Adiciona `email_vinculado`, `email_obra`, `pasta_drive_id` em `obras` |
| `20260413_master_rls_fix.sql` | 2026-04-13 | RLS/policy ampla para tabelas públicas (antes do single-admin) |
| `20260818_single_admin_rls.sql` | 2026-08-18 | Substitui policies permissivas por `erp_admin_only` + allowlist |
| `20260819_revoke_anon_sequence_rpc.sql` | 2026-08-19 | Revoga execução anônima da RPC de sequência |
| `20260917_seed_obr003_lancamentos.sql` | 2026-09-17 | Seed de lançamentos para obra OBR-003 |
| `20260920_obra_orcamento_etapas.sql` | 2026-09-20 | Cria `obra_orcamento_etapas` + `obra_curva_s` |
| `20260920_seed_obr003_orcamento_etapas.sql` | 2026-09-20 | Seed de etapas orçamentárias para OBR-003 |
| `20260920_seed_obr003_transacoes.sql` | 2026-09-20 | Seed de transações para OBR-003 |
| `20260920_transacoes_reais.sql` | 2026-09-20 | Cria `transacoes_reais` |
| `20260921_transacoes_email_conta.sql` | 2026-09-21 | Adiciona `email_conta` a `transacoes_reais` |
| `20261104_cronograma_marcos.sql` | 2026-11-04 | Adiciona marcos ao `cronograma_etapas` |

---

## Lacunas de conhecimento sobre este domínio `[DESCONHECIDO]`

- **Schema-base não versionado:** as migrations locais **não criam** as tabelas centrais (`obras`, `pendencias`, `contratos_empreiteiros`, `contrato_itens`, `medicoes`, `medicao_itens`, `solicitacoes_material`, `solicitacao_itens`, `cotacoes`, `cotacao_itens_fornecedor`, `pedidos_compra`, `pedido_compra_itens`, `fornecedores`, `materiais`, `empreiteiros`, `equipes`, `tipos_servico`, `obras_tipos_servico`, `atividades_padrao`, `cronograma_etapas`, `classificacoes_financeiras`, `rateios_financeiros`). O Supabase ativo contém migrations remotas anteriores não presentes no Git.
- DDL exato de `transacoes_reais` (não lida em detalhe).
- DDL exato de `obra_emails` (referenciada em código mas sem migration local visível).
- Estado real das policies RLS em produção.
- Buckets Supabase Storage configurados.
- Schema remoto `external_id` mencionado em `ARCHITECTURE.md` mas não no checkout.
- Índices adicionais criados diretamente no banco.

---

## Dependências

- Este documento depende de: `02-modulos.md`, `04-regras-negocio.md`.
- Atualizações neste documento podem afetar: `08-estado-implementacao.md`, `09-limitacoes-problemas.md`.
