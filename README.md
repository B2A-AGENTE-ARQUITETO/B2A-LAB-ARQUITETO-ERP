# B2A-LAB-ARQUITETO-ERP

**Monorepositório de desenvolvimento e testes do Agente Arquiteto + ERP B2A Engenharia**
**Versão:** 1.0.0 · **Data:** 2026-11-24 · **Status:** laboratório (não-produção)

---

## 1. O que é

Repositório privado que integra, em um único workspace:

1. **Cópia independente do ERP B2A Engenharia** em `erp/` — para análise, evolução e testes locais sem tocar no ambiente de produção.
2. **Pacote completo do Agente Arquiteto** em `architect/` — instruções, skills, knowledge base, templates, memória e permissões.
3. **Skills carregáveis pelo OpenBot** em `.agents/skills/` — symlink/disponibilização para o provedor (Codex por padrão).
4. **Documentação de instalação e operação** na raiz.

Este repositório **não substitui** o ERP em produção. É o ambiente oficial onde o **Agente Arquiteto** trabalha com código real, compreende a estrutura do ERP, propõe correções, desenvolve funcionalidades, executa testes e mantém conhecimento técnico atualizado.

## 2. Estrutura

```text
B2A-LAB-ARQUITETO-ERP/
├── AGENTS.md                              # Instruções raiz lidas pelo OpenBot
├── README.md                              # Este arquivo
├── PROMPT-INICIALIZACAO-OPENBOT.md        # Prompt de bootstrap do agente
├── CONFIGURACAO-CREDENCIAIS.md            # Guia de configuração de credenciais
├── RELATORIO-AUDITORIA-ERP.md             # Auditoria de segurança e n8n
├── RELATORIO-VALIDACAO.md                 # Validação do monorepo
├── PENDENCIAS-INTEGRACAO.md               # Pendências e próximos passos
├── INVENTARIO.md                          # Finalidade de cada arquivo
│
├── .gitignore
├── .env.example                           # Template de variáveis (preencher em .env)
│
├── .agents/skills/                        # Skills carregadas pelo Codex/OpenBot
│   └── (symlinks ou cópias das skills em architect/skills/)
│
├── architect/                             # Tudo do Agente Arquiteto
│   ├── AGENTS.md                          # Instruções específicas do Arquiteto
│   ├── knowledge/
│   │   ├── erp/                           # 10 domínios + INDEX
│   │   └── ecosystem/                     # 6 documentos + INDEX
│   ├── skills/                            # 6 skills operacionais
│   │   ├── erp-knowledge/SKILL.md
│   │   ├── agent-design/SKILL.md
│   │   ├── skill-engineering/SKILL.md
│   │   ├── tool-integration/SKILL.md
│   │   ├── software-engineering/SKILL.md
│   │   └── agent-evaluation/SKILL.md
│   ├── templates/
│   │   └── funcionario-digital/           # Template reutilizável
│   ├── permissions/                       # Catálogo de tools, denylist, approvals
│   ├── memory/                            # Memória persistente do Arquiteto
│   └── tests/
│       └── aceitacao-arquiteto.md         # 10 testes de aceitação
│
├── erp/                                   # Cópia do ERP B2A (código real)
│   ├── README.md                          # Como esta cópia foi preparada
│   ├── (arquivos do ERP original)
│   └── INTEGRACAO-LAB.md                  # Notas de adaptações feitas no lab
│
├── integrations/                          # Scripts e configs de integração
│   └── README.md
│
├── docs/                                  # Documentação técnica complementar
│   └── README.md
│
└── assets/                                # Recursos visuais (ícones, imagens)
```

## 3. Pré-requisitos

- **Node.js 22+** (validado em 24.19.0)
- **npm** (gerenciador canônico — `bun.lock` do ERP original foi removido)
- **Git** (com acesso configurado ao repositório privado)
- **OpenBot v0.72.0+** (para rodar o Agente Arquiteto)
- **Conta GitHub** com acesso ao repositório `B2A-LAB-ARQUITETO-ERP`
- **Projeto Supabase NOVO** (não o de produção — ver `CONFIGURACAO-CREDENCIAIS.md`)
- **Google Gemini API key** (para o endpoint `/api/agent` do ERP)

## 4. Instalação rápida

### 4.1 Clonar o monorepositório

```bash
git clone <url-do-repo-privado> B2A-LAB-ARQUITETO-ERP
cd B2A-LAB-ARQUITETO-ERP
```

### 4.2 Configurar variáveis de ambiente

```bash
cp .env.example .env
# Editar .env e preencher com credenciais NOVAS (ver CONFIGURACAO-CREDENCIAIS.md)
```

### 4.3 Instalar dependências do ERP

```bash
cd erp
npm install --no-package-lock  # usa package-lock.json do repo
cd ..
```

### 4.4 Verificações estáticas

```bash
cd erp
npm run typecheck
npm run build
cd ..
```

### 4.5 Rodar o ERP localmente

```bash
cd erp
npm run dev
# Acessar http://localhost:3000
```

### 4.6 Configurar o OpenBot

Ver `PROMPT-INICIALIZACAO-OPENBOT.md` para instruções detalhadas de como criar o agente **Arquiteto B2A** no OpenBot apontando para este workspace.

## 5. O que mudou em relação ao ERP original

| Aspecto | ERP original (produção) | Este monorepo (laboratório) |
|---|---|---|
| Repositório | `anderson110288-afk/APPSITE-ERP-B2A-Engenharia` | `<nova-conta>/B2A-LAB-ARQUITETO-ERP` |
| Branch principal | `main` | `main` |
| Hosting | Vercel (produção) | Local apenas (Vercel preview opcional) |
| Supabase | Projeto de produção B2A | Projeto NOVO independente |
| n8n | Workflow financeiro ativo | **Desativado** (ver `RELATORIO-AUDITORIA-ERP.md`) |
| Bearer token webhook | `n8n-b2a-secret-key-2026` (exposto no Git) | **Rotacionado** — gerar novo em `.env` |
| Variáveis de ambiente | Configuradas na Vercel | `.env` local (não commited) |
| `bun.lock` | Presente | **Removido** (npm é canônico) |
| Credenciais OAuth n8n | Ativas | Não aplicável (n8n desativado) |

Ver `RELATORIO-AUDITORIA-ERP.md` para detalhes completos.

## 6. Arquitetura ativa vs desativada

### Ativo no laboratório
- Next.js 16 + React 19 + TypeScript
- Supabase (projeto novo) — Auth + PostgreSQL + RLS
- Gemini 2.0 Flash (endpoint `/api/agent`)
- 9 módulos de negócio do ERP (Obras, Cronograma, Contratos, Medições, Pendências, Compras, Empreiteiros, Financeiro, Admin)
- 6 skills do Arquiteto
- Knowledge base do ERP (10 domínios) e do ecossistema (6 documentos)

### Desativado no laboratório
- **n8n** — workflows `workflow_n8n_financeiro.json`, `wf_healthy_template.json`, `wf_healthy_template_v2.json` preservados como referência mas não ativos. Ver `erp/INTEGRACAO-LAB.md`.
- **Vercel** — sem deploy automático. Deploy preview opcional mediante autorização.
- **OAuth do n8n** (Gmail, Google Drive, Google Sheets) — não configurado.

### Pendência de implementação futura
- **Orquestrador de funcionários digitais** — não implementado nesta versão.
- **MCPs externos** — nenhum configurado por padrão. Solicitar via `architect/permissions/pending.md`.

## 7. Regras de segurança

1. **NUNCA commite `.env`** — já está no `.gitignore`.
2. **NUNCA reutilize credenciais** do ERP de produção.
3. **NUNCA faça push para o repositório original** (`anderson110288-afk/APPSITE-ERP-B2A-Engenharia`).
4. **NUNCA execute migrações destrutivas** no Supabase de testes sem backup.
5. **NUNCA conecte o laboratório a serviços de produção.**

Ver `architect/AGENTS.md` seção 3 para princípios inegociáveis completos.

## 8. Como o Arquiteto usa este monorepo

O Arquiteto, quando instanciado no OpenBot com este workspace:

1. Lê `AGENTS.md` da raiz e `architect/AGENTS.md` para instruções específicas.
2. Tem acesso direto ao código em `erp/` para leitura, análise e edição.
3. Pode criar branches, modificar código, executar testes — autonomia de desenvolvimento.
4. Solicita autorização para: conectar contas externas, instalar dependências com risco, incorporar à `main`, publicar.
5. Carrega as 6 skills de `architect/skills/` (também disponíveis em `.agents/skills/` para o provider).
6. Consulta knowledge base em `architect/knowledge/` antes de propor alterações.
7. Registra decisões em `architect/memory/decisions.md` e evoluções em `architect/memory/evolution-log.md`.

## 9. Próximos passos (após instalação)

1. **Validar ambiente:** rodar `npm run typecheck` e `npm run build` em `erp/`.
2. **Inicializar Supabase:** criar projeto novo, aplicar migrations de `erp/supabase/migrations/`.
3. **Configurar OpenBot:** seguir `PROMPT-INICIALIZACAO-OPENBOT.md`.
4. **Diagnóstico inicial:** primeira conversa com o Arquiteto para auto-diagnóstico do ERP.
5. **Primeira missão:** definir com o proprietário a primeira melhoria concreta a desenvolver.

## 10. Documentação complementar

| Documento | Finalidade |
|---|---|
| `PROMPT-INICIALIZACAO-OPENBOT.md` | Prompt de bootstrap do agente no OpenBot |
| `CONFIGURACAO-CREDENCIAIS.md` | Guia passo-a-passo de credenciais |
| `RELATORIO-AUDITORIA-ERP.md` | Auditoria de segurança e dependências do ERP copiado |
| `RELATORIO-VALIDACAO.md` | Validações executadas neste monorepo |
| `PENDENCIAS-INTEGRACAO.md` | Pendências e o que falta para operação completa |
| `INVENTARIO.md` | Finalidade de cada arquivo |
| `architect/AGENTS.md` | Instruções operacionais do Arquiteto |
| `architect/knowledge/erp/INDEX.md` | Índice do conhecimento sobre o ERP |
| `architect/knowledge/ecosystem/INDEX.md` | Índice do conhecimento sobre o ecossistema |
| `architect/tests/aceitacao-arquiteto.md` | 10 testes de aceitação do Arquiteto |
| `erp/README.md` | Notas específicas da cópia do ERP |
| `erp/INTEGRACAO-LAB.md` | Adaptações feitas no laboratório |

## 11. Licença e propriedade

Propriedade da B2A Engenharia & Construção. Uso interno. Não distribuir.

## 12. Status

**v1.0.0 — 2026-11-24**

- Estrutura do monorepo criada.
- Pacote do Arquiteto integrado em `architect/`.
- ERP copiado para `erp/` (ver `RELATORIO-AUDITORIA-ERP.md` para status da cópia).
- n8n desativado da arquitetura ativa.
- Credenciais antigas identificadas e marcadas para rotação.
- Pendências de criação do repositório GitHub privado e configuração do Supabase — ver `PENDENCIAS-INTEGRACAO.md`.
