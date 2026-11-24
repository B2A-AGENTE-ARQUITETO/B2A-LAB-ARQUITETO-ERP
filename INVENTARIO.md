# INVENTARIO.md — Monorepo B2A-LAB-ARQUITETO-ERP

**Versão:** 1.0.0 · **Data:** 2026-11-24

## Estrutura completa

```text
B2A-LAB-ARQUITETO-ERP/
├── AGENTS.md                              # Instruções raiz lidas pelo OpenBot
├── README.md                              # Visão geral e instalação
├── PROMPT-INICIALIZACAO-OPENBOT.md        # Prompt de bootstrap do agente
├── CONFIGURACAO-CREDENCIAIS.md            # Guia de credenciais
├── RELATORIO-AUDITORIA-ERP.md             # Auditoria de segurança do ERP
├── RELATORIO-VALIDACAO.md                 # Validações executadas
├── PENDENCIAS-INTEGRACAO.md               # Pendências e próximos passos
├── INVENTARIO.md                          # Este arquivo
├── .gitignore                             # Ignora .env, node_modules, etc.
├── .env.example                           # Template de variáveis
│
├── .agents/skills/                        # Skills para Codex (symlinks pós-install)
│
├── architect/                             # Tudo do Agente Arquiteto (41 arquivos)
│   ├── AGENTS.md                          # Instruções específicas do Arquiteto (v1.1.0)
│   ├── knowledge/
│   │   ├── erp/                           # 11 arquivos sobre o ERP
│   │   │   ├── INDEX.md
│   │   │   ├── 01-visao-geral.md
│   │   │   ├── 02-modulos.md
│   │   │   ├── 03-processos.md
│   │   │   ├── 04-regras-negocio.md
│   │   │   ├── 05-modelo-dados.md
│   │   │   ├── 06-apis-integracoes.md
│   │   │   ├── 07-fluxos-entre-modulos.md
│   │   │   ├── 08-estado-implementacao.md
│   │   │   ├── 09-limitacoes-problemas.md
│   │   │   └── 10-historico-decisoes.md
│   │   └── ecosystem/                     # 7 arquivos sobre o ecossistema
│   │       ├── INDEX.md
│   │       ├── openbot.md
│   │       ├── papeis-agentes.md
│   │       ├── padrao-criacao-funcionarios.md
│   │       ├── estrategia-integracao-ferramentas.md
│   │       ├── principios-seguranca.md
│   │       └── estrategia-evolucao.md
│   ├── skills/                            # 6 skills operacionais
│   │   ├── erp-knowledge/SKILL.md
│   │   ├── agent-design/SKILL.md
│   │   ├── skill-engineering/SKILL.md
│   │   ├── tool-integration/SKILL.md
│   │   ├── software-engineering/SKILL.md
│   │   └── agent-evaluation/SKILL.md
│   ├── templates/
│   │   └── funcionario-digital/           # Template reutilizável (7 arquivos)
│   │       ├── README.md
│   │       ├── agent.yaml
│   │       ├── AGENTS.md
│   │       ├── tools.yaml
│   │       ├── tests/cases.yaml
│   │       ├── tests/fixtures/example.json
│   │       └── knowledge/INDEX.md
│   ├── permissions/                       # 4 arquivos de controle de acesso
│   │   ├── tools-catalog.yaml
│   │   ├── pending.md
│   │   ├── granted.md
│   │   └── erp-changes-proposed.md
│   ├── memory/                            # 10 arquivos de memória persistente
│   │   ├── decisions.md
│   │   ├── agents-registry.md
│   │   ├── tools-registry.md
│   │   ├── research-log.md
│   │   ├── knowledge-gaps.md
│   │   ├── divergences.md
│   │   ├── incidents.md
│   │   ├── evolution-log.md
│   │   ├── pending-tasks.md
│   │   └── conversation-summary.md
│   └── tests/
│       └── aceitacao-arquiteto.md         # 10 testes de aceitação
│
├── erp/                                   # Cópia do ERP B2A
│   ├── README.md                          # Como popular
│   └── INTEGRACAO-LAB.md                  # Adaptações do laboratório
│   # (código do ERP a ser copiado pelo proprietário - ver P2)
│
├── integrations/                          # Scripts e configs de integração
│   └── README.md
│
├── docs/                                  # Documentação técnica complementar
│   └── README.md
│
└── assets/                                # Recursos visuais (vazio)
```

## Descrição por arquivo

### Arquivos raiz (10 arquivos)

| Arquivo | Finalidade | Quem lê |
|---|---|---|
| `AGENTS.md` | Instruções raiz do agente (lida pelo OpenBot/Codex a cada conversa) | Provider de IA |
| `README.md` | Visão geral do monorepo + instalação rápida | Proprietário |
| `PROMPT-INICIALIZACAO-OPENBOT.md` | Prompt de bootstrap do agente no OpenBot (14 itens da diretriz §11) | Proprietário |
| `CONFIGURACAO-CREDENCIAIS.md` | Guia passo-a-passo de credenciais (7 passos) | Proprietário |
| `RELATORIO-AUDITORIA-ERP.md` | Auditoria de segurança do ERP original (5 bloqueadores, 10 lacunas, 5 riscos) | Proprietário + Arquiteto |
| `RELATORIO-VALIDACAO.md` | 15 verificações executadas no monorepo | Proprietário |
| `PENDENCIAS-INTEGRACAO.md` | 14 pendências (P1–P14) com responsável e procedimento | Proprietário |
| `INVENTARIO.md` | Este arquivo — finalidade de cada arquivo | Referência |
| `.gitignore` | Ignora `.env`, `node_modules`, `.next/`, etc. | Git |
| `.env.example` | Template de variáveis (NÃO commiter `.env`) | Proprietário |

### architect/ (41 arquivos)

**`architect/AGENTS.md`** (v1.1.0) — instruções detalhadas do Arquiteto: identidade, missão, 12 princípios inegociáveis (P1–P12, incluindo P11 n8n desativado e P12 sem alterar produção), hierarquia de instruções, procedimentos operacionais, hierarquia de autonomia, fluxo EE, etc.

**`architect/knowledge/erp/`** (11 arquivos) — base de conhecimento sobre o ERP B2A:
- `INDEX.md` — índice navegável.
- `01-visao-geral.md` — stack, arquitetura, componentes.
- `02-modulos.md` — 9 módulos de negócio + rotas.
- `03-processos.md` — 10 processos operacionais.
- `04-regras-negocio.md` — 12 regras formais (RN-01 a RN-12).
- `05-modelo-dados.md` — 28 tabelas, DDL, migrations, tipos TypeScript.
- `06-apis-integracoes.md` — 3 APIs server-side + 5 integrações externas.
- `07-fluxos-entre-modulos.md` — 6 fluxos ponta-a-ponta.
- `08-estado-implementacao.md` — concluído/parcial/pendente.
- `09-limitacoes-problemas.md` — 19 itens (5 bloqueadores + 10 lacunas + 4 segurança).
- `10-historico-decisoes.md` — 17 decisões técnicas (D1–D17).

**`architect/knowledge/ecosystem/`** (7 arquivos) — base de conhecimento sobre o ecossistema:
- `INDEX.md` — índice navegável.
- `openbot.md` — capacidades verificadas do OpenBot (v0.72.0).
- `papeis-agentes.md` — 4 papéis (Proprietário, Arquiteto, Funcionários, Orquestrador).
- `padrao-criacao-funcionarios.md` — template + fluxo E→P→X→R.
- `estrategia-integracao-ferramentas.md` — MCP, plugins, denylist.
- `principios-seguranca.md` — SG1–SG7, T0 eliminatórios.
- `estrategia-evolucao.md` — Modo EE, atualização, manutenção.

**`architect/skills/`** (6 SKILL.md) — skills operacionais no formato Agent Skills:
- `erp-knowledge` — consultar/analisar/atualizar conhecimento ERP.
- `agent-design` — criar funcionários digitais.
- `skill-engineering` — desenvolver skills reutilizáveis.
- `tool-integration` — pesquisar/integrar ferramentas externas.
- `software-engineering` — analisar/implementar alterações no ERP.
- `agent-evaluation` — testar/avaliar/corrigir funcionários.

**`architect/templates/funcionario-digital/`** (7 arquivos) — template reutilizável:
- `README.md` — guia de uso (8 passos).
- `agent.yaml` — manifesto com placeholders.
- `AGENTS.md` — estrutura de instruções com placeholders.
- `tools.yaml` — ferramentas (apenas internas por padrão).
- `tests/cases.yaml` — 3 casos template (normal, borda, erro).
- `tests/fixtures/example.json` — fixture fictício.
- `knowledge/INDEX.md` — índice de conhecimento específico template.

**`architect/permissions/`** (4 arquivos):
- `tools-catalog.yaml` — catálogo de ferramentas + denylist.
- `pending.md` — solicitações aguardando aprovação.
- `granted.md` — acessos concedidos ativos.
- `erp-changes-proposed.md` — propostas de alteração no ERP.

**`architect/memory/`** (10 arquivos):
- `decisions.md` — decisões técnicas com rationale.
- `agents-registry.md` — funcionários criados.
- `tools-registry.md` — ferramentas disponíveis.
- `research-log.md` — pesquisas técnicas.
- `knowledge-gaps.md` — lacunas identificadas (14 herdadas).
- `divergences.md` — divergências entre documentos.
- `incidents.md` — falhas e incidentes.
- `evolution-log.md` — histórico de evoluções.
- `pending-tasks.md` — tarefas suspensas.
- `conversation-summary.md` — resumo da última conversa.

**`architect/tests/`** (1 arquivo):
- `aceitacao-arquiteto.md` — 10 testes de aceitação.

### erp/ (2 arquivos + código a popular)

| Arquivo | Finalidade |
|---|---|
| `erp/README.md` | Instruções para popular `erp/` com código do ERP original |
| `erp/INTEGRACAO-LAB.md` | 8 adaptações do laboratório (remover bun.lock, sanitizar n8n, etc.) |

**Código do ERP:** pendente de cópia pelo proprietário (ver `PENDENCIAS-INTEGRACAO.md` P2). Após populado, conterá ~28 arquivos entre código, configs, migrations e documentação do ERP original.

### integrations/ (1 arquivo)

| Arquivo | Finalidade |
|---|---|
| `integrations/README.md` | Placeholder para scripts e configs de integração futura |

### docs/ (1 arquivo)

| Arquivo | Finalidade |
|---|---|
| `docs/README.md` | Placeholder para documentação técnica complementar |

### assets/ (vazio)

Pasta para ícone futuro do agente no OpenBot (`assets/icon.png`, 256x256).

## Total de arquivos

| Categoria | Quantidade |
|---|---|
| Arquivos raiz | 10 |
| `architect/` | 41 |
| `erp/` (atual) | 2 |
| `integrations/` | 1 |
| `docs/` | 1 |
| `.agents/skills/` (vazio, symlinks pós-install) | 0 |
| `assets/` (vazio) | 0 |
| **Total** | **55 arquivos** |

Após popular `erp/` (P2), o total sobe para ~83 arquivos.

## Diferenças em relação ao pacote anterior (AGENTE_ARQUITETO_OPENBOT v1.0.0)

| Item | Pacote anterior | Monorepo atual |
|---|---|---|
| Estrutura | Pacote isolado | Monorepo com `architect/` + `erp/` |
| ERP | Não incluído (apenas knowledge base) | Incluído em `erp/` (pendente de cópia) |
| n8n | Documentado como ativo | **Desativado** (diretriz §7) |
| `.env.example` | Não tinha | Template com 7 variáveis |
| `.gitignore` | Não tinha | Cobertura completa |
| `PROMPT-INICIALIZACAO-OPENBOT.md` | Não tinha | Prompt completo de bootstrap |
| `CONFIGURACAO-CREDENCIAIS.md` | Não tinha | Guia de 7 passos |
| `RELATORIO-AUDITORIA-ERP.md` | Não tinha | Auditoria completa |
| `PENDENCIAS-INTEGRACAO.md` | Não tinha | 14 pendências |
| Skills em `architect/skills/` | Skills em `skills/` (raiz) | Movidas para `architect/skills/` |
| Symlinks `.agents/skills/` | Não aplicável | A criar pós-instalação |
| Princípios | P1–P10 | P1–P12 (acrescentados P11 n8n desativado, P12 sem alterar produção) |
