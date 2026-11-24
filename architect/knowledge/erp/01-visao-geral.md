# 01 — Visão Geral e Arquitetura Técnica

**Última verificação:** 2026-11-24 · **Fonte:** `Catalogo-Capacidades-ERP-B2A-Engenharia.md` seção 2

## Sumário

O APPSITE ERP B2A Engenharia é um ERP web vertical para gestão de obras de construção civil, ativo em produção na Vercel. Stack moderna baseada em Next.js 16 + React 19 + Supabase + n8n + Gemini 2.0 Flash. Possui 9 módulos de negócio implementados, 3 endpoints de API server-side e 2 workflows n8n versionados.

## Stack técnica `[VERIFICADO]`

| Camada | Tecnologia | Versão | Arquivo de referência |
|---|---|---|---|
| Framework | Next.js (App Router) | 16.1.6 | `package.json` |
| UI | React | 19.2.4 | `package.json` |
| Estilo | Tailwind CSS | 4.2.1 | `package.json`, `src/app/globals.css` |
| Linguagem | TypeScript (strict) | 5.9.3 | `tsconfig.json` |
| Banco/Auth | Supabase (PostgreSQL + Auth) | — | `src/lib/supabase.ts`, `src/lib/supabase-admin.ts` |
| IA | Google Gemini via `@google/generative-ai` | 2.0 Flash ^0.24.1 | `src/app/api/agent/route.ts` |
| Validação | Zod | 4.3.6 | `src/lib/schemas.ts` |
| PDF | jsPDF + jspdf-autotable | 4.2.0 + 5.0.7 | `src/features/contas-a-pagar/lib/pdfGenerator.ts` |
| Notificações UI | sonner | 2.0.7 | hook de compras |
| Automação externa | n8n | — | `instrucoes_n8n.ps1`, URL `https://n8nb2aengenharia.site` |
| Hosting | Vercel | — | `vercel.json`, `ENVIRONMENT.md` |
| Node.js | ≥ 22.0.0 (validado em 24.19.0) | — | `package.json`, `OPERATIONS.md` |

## Componentes e fluxo de dados `[VERIFICADO]`

```
┌────────────────────────────────────────────────────────────────────┐
│ Browser (React 19 client)                                          │
│  ├─ AuthContext (sessão Supabase)                                  │
│  ├─ ObraContext (obra ativa — RN-01 isolamento)                    │
│  └─ storage.ts (41 KB, 30+ métodos, cache 5 min)                   │
│        └─ supabase.ts (browser client, chave publishable)          │
└────────────────┬───────────────────────────────────────────────────┘
                 │ HTTPS / cookies de sessão
┌────────────────▼───────────────────────────────────────────────────┐
│ Next.js 16 (Vercel)                                                │
│  ├─ middleware.ts (falha fechado se Auth indisponível)              │
│  ├─ Route Handlers:                                                 │
│  │    • POST /api/agent            → Gemini 2.0 Flash               │
│  │    • POST /api/financeiro/importar → n8n webhook (bearer)       │
│  │    • POST /api/financeiro/lancamentos → updateRateio/delete     │
│  └─ supabase-admin.ts (service_role — somente server)             │
└────────────────┬───────────────────────────────────────────────────┘
                 │
        ┌────────┴────────┬────────────────┐
        ▼                 ▼                ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  Supabase    │  │   Gemini     │  │     n8n      │
│  PostgreSQL  │  │   2.0 Flash  │  │  (externo)   │
│  + Auth      │  │              │  │              │
│  + RLS       │  │              │  │  Gmail → OCR │
│  admin_only  │  │              │  │  → POST ERP  │
└──────────────┘  └──────────────┘  └──────────────┘
```

## URL de produção `[VERIFICADO]`

`https://appsite-erp-b2-a-engenharia.vercel.app`

## Estrutura de diretórios do repositório `[VERIFICADO]`

```
APPSITE-ERP-B2A-Engenharia/
├── .github/                        # copilot-instructions.md
├── .kilocode/rules/memory-bank/    # template inicial (legacy)
├── AGENTS.md                       # Instruções para IA e humanos
├── ARCHITECTURE.md                 # Arquitetura confirmada e limites
├── CLAUDE.md
├── ENVIRONMENT.md                  # Variáveis de ambiente
├── ESTRUTURA_PADRAO_DESENVOLVIMENTO.md  # Padrão de feature development
├── GEMINI.md
├── OPERATIONS.md                   # Comandos e procedimentos
├── SECURITY.md                     # Regras de segurança
├── bun.lock + package-lock.json    # Lockfiles DUPLICADOS (ambiguidade)
├── docs/
│   ├── architecture-map.html
│   ├── architecture-map.json
│   └── relatorios/
├── financial_agent_prompt.md       # Prompt para agente de conciliação (descritivo)
├── instrucoes_n8n.ps1              # Script de configuração n8n
├── next.config.ts
├── package.json
├── plano_integracao_cronograma_financeiro.md  # PLANO (não implementado)
├── postcss.config.mjs
├── skills-lock.json
├── src/
│   ├── app/                        # App Router (páginas + APIs)
│   ├── components/{layout,ui}/
│   ├── contexts/{AuthContext,ObraContext}.tsx
│   ├── features/{compras-workflow,contas-a-pagar,financeiro-conciliacao}/
│   ├── lib/                        # storage.ts, types.ts, schemas.ts, etc.
│   ├── middleware.ts
│   └── utils/supabase/
├── supabase/migrations/            # 12 migrations SQL
├── tsconfig.json
├── vercel.json
├── wf_healthy_template.json        # Workflow n8n (legacy)
├── wf_healthy_template_v2.json     # Workflow n8n (criação de obra via Sheets)
└── workflow_n8n_financeiro.json    # Workflow n8n (OCR financeiro)
```

## Princípios operacionais do próprio ERP `[VERIFICADO]`

O `AGENTS.md` do ERP define o princípio operacional para qualquer IA ou humano trabalhando nele:

> COMPREENDER → DIAGNOSTICAR → PLANEJAR → VERSIONAR → ALTERAR → TESTAR → DOCUMENTAR

## Estado geral de maturidade `[VERIFICADO]`

- **Em produção:** sim, ativo na Vercel.
- **Cobertura de testes:** **nenhuma** — `package.json` não declara framework de testes; `OPERATIONS.md` confirma ausência.
- **Lint:** bloqueado por incompatibilidade ESLint 10 vs `eslint-config-next` (suporta até ESLint 9).
- **TypeCheck:** `tsc --noEmit` passa (validado em 2026-08-18 com Node 24.19.0).
- **Build:** passa em Node 24.19.0.

## Lacunas de conhecimento sobre este domínio `[DESCONHECIDO]`

- Variáveis de ambiente efetivamente configuradas na Vercel.
- Domínio customizado (se houver) além do subdomínio Vercel.
- Estado real do schema-base do Supabase (migrations locais não criam todas as tabelas centrais).
- Buckets Supabase Storage configurados (não há referências no código).

## Dependências

- Este documento depende de: `02-modulos.md`, `06-apis-integracoes.md`, `09-limitacoes-problemas.md`.
- Atualizações neste documento podem afetar: `08-estado-implementacao.md`.
