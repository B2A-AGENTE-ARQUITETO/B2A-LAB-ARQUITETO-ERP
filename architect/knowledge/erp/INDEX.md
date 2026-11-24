# Knowledge Base — ERP B2A Engenharia

**Versão:** 1.0.0 · **Última atualização:** 2026-11-24 · **Fonte primária:** `Catalogo-Capacidades-ERP-B2A-Engenharia.md`

## Convenção de verificação

Toda afirmação técnica é marcada com uma de três etiquetas:

| Etiqueta | Significado |
|---|---|
| `[VERIFICADO]` | Confirmado no código do ERP no commit `0920bb4` (branch `main`). Referência ao arquivo incluída. |
| `[INFERIDO]` | Inferência razoável a partir do código ou docs, mas não confirmada no schema ativo do Supabase. |
| `[DESCONHECIDO]` | Informação que exigirá consulta futura ao repositório ou ambiente real. |

## Índice de domínios

| # | Documento | Assunto | Estado |
|---|---|---|---|
| 01 | [visao-geral](01-visao-geral.md) | Stack, arquitetura, componentes | Verificado |
| 02 | [modulos](02-modulos.md) | Módulos de negócio e rotas | Verificado |
| 03 | [processos](03-processos.md) | Processos operacionais | Verificado |
| 04 | [regras-negocio](04-regras-negocio.md) | Regras de negócio (RN-01, RN-02, RN-03) | Verificado |
| 05 | [modelo-dados](05-modelo-dados.md) | Tabelas, schemas, migrations | Parcial |
| 06 | [apis-integracoes](06-apis-integracoes.md) | APIs server-side, n8n, Gemini | Verificado |
| 07 | [fluxos-entre-modulos](07-fluxos-entre-modulos.md) | Fluxos ponta-a-ponta | Verificado |
| 08 | [estado-implementacao](08-estado-implementacao.md) | O que está concluído, parcial, pendente | Verificado |
| 09 | [limitacoes-problemas](09-limitacoes-problemas.md) | Débitos técnicos, lacunas, riscos | Verificado |
| 10 | [historico-decisoes](10-historico-decisoes.md) | Decisões técnicas registradas | Verificado |

## Como consultar

1. **Para pergunta específica:** localize o domínio no índice acima, abra o documento, use o sumário interno do documento.
2. **Para visão geral:** comece por `01-visao-geral.md`.
3. **Para implementar integração:** consulte `06-apis-integracoes.md` e `07-fluxos-entre-modulos.md`.
4. **Para identificar lacunas:** consulte `09-limitacoes-problemas.md`.
5. **Para entender por que algo é como é:** consulte `10-historico-decisoes.md`.

## Como atualizar

Quando você obtiver nova informação verificada sobre o ERP:

1. Localize o documento relevante.
2. Edite a seção apropriada mantendo a etiqueta de verificação.
3. Atualize `last_verified_at` no topo do documento.
4. Registre a atualização em `../../memory/evolution-log.md`.
5. Se a informação contradiz documento existente, registre divergência em `../../memory/divergences.md` antes de atualizar.

## Lacunas conhecidas (não tente preencher sem consulta autorizada)

- Schema-base do Supabase (tabelas centrais não versionadas no Git) `[DESCONHECIDO]`
- Estado real das policies RLS em produção `[DESCONHECIDO]`
- Buckets Supabase Storage configurados `[DESCONHECIDO]`
- Configuração Vercel (variáveis de ambiente em produção) `[DESCONHECIDO]`
- Credenciais n8n/OAuth ativas `[DESCONHECIDO]`
- Status real do ESLint (incompatibilidade ESLint 10 vs eslint-config-next) `[VERIFICADO]` — bloqueado
- Lockfile canônico (npm vs Bun) `[DESCONHECIDO]` — ambiguidade não resolvida

## Fontes de referência

| Fonte | Localização | Tipo |
|---|---|---|
| Catálogo de Capacidades (primária) | `download/Catalogo-Capacidades-ERP-B2A-Engenharia.md` | Documento de análise |
| Repositório GitHub | `github.com/anderson110288-afk/APPSITE-ERP-B2A-Engenharia` (privado) | Código-fonte |
| Branch canônica | `main` | — |
| Commit de referência | `0920bb4` | — |

**Regra:** consulte o repositório GitHub **apenas** sob autorização explícita do proprietário (ver `AGENTS.md` seção 11).
