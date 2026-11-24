# Padrão de Criação de Funcionários Digitais

**Última verificação:** 2026-11-24

## Sumário

Todo funcionário digital criado pelo Arquiteto segue o mesmo padrão: estrutura de arquivos idêntica, fluxo de criação E→P→X→R, versionamento por commit SHA, testes de aceitação obrigatórios.

---

## Estrutura padrão de um funcionário

```
agents/<agent-id>/
├── agent.yaml              # Manifesto principal (identidade, missão, versão)
├── AGENTS.md               # Instruções principais (system prompt)
├── tools.yaml              # Ferramentas autorizadas + escopo
├── tests/
│   ├── cases.yaml          # Casos de teste com fixtures
│   ├── fixtures/           # Dados fictícios
│   └── results/            # Última execução por caso
├── knowledge/              # Conhecimento específico do funcionário (opcional)
│   └── INDEX.md
├── memory/                 # Memória operacional do funcionário
│   └── session.jsonl       # Append-only JSONL
└── README.md               # Documentação operacional (gerada)
```

**Localização no workspace do Arquiteto:** `~/OpenBot/Agents/agente-arquiteto/agents/<agent-id>/`.

Para funcionários fictícios (testes): mesma estrutura, prefixo `ficticio-` no `agent-id`.

---

## Schema do `agent.yaml`

```yaml
# Manifesto do funcionário digital
id: <string>                    # identificador único (kebab-case)
display_name: <string>          # nome exibido no OpenBot
role: <string>                  # cargo/role curto
version: <semver>               # 1.0.0 inicial
created_at: <ISO-8601>
created_by: agente-arquiteto
parent_commit: <git-sha>        # commit do Arquiteto que criou

mission: <string>               # missão em 1 frase
responsibilities:               # 3-7 responsabilidades
  - <string>
limits:                         # limites explícitos
  - <string>

provider:
  primary: <provider>           # codex | claude | gemini | ...
  fallback: [<provider>, ...]
  model: <string>               # ex.: 'gpt-5', 'claude-sonnet-4'

permissions_profile: workspace_only | ask_first | full_access

tools:                          # referência a tools.yaml
  file: tools.yaml

knowledge:                      # referência a knowledge/ (opcional)
  folder: knowledge

tests:                          # referência a tests/
  folder: tests
  threshold: 0.85               # score mínimo para aprovação
```

---

## Schema do `tools.yaml`

```yaml
# Ferramentas autorizadas para este funcionário
tools:
  - id: <tool_id>                        # ex.: mcp.github
    kind: internal | mcp | http | cli | architect_developed
    permissions_used: [read] | [write] | [read, write]
    scope:                                # escopo específico
      repositories: ["org/repo-x"]
      operations: ["list_issues", "read_file"]
    reason: <string>                      # por que precisa desta ferramenta
```

**Regras:**
- `permissions_used` deve ser subset das `permissions` declaradas no catálogo.
- `scope` deve ser subset do `approved_scope` em `permissions/granted.md`.
- O interceptor valida antes de cada chamada.

---

## Schema do `tests/cases.yaml`

```yaml
# Casos de teste de aceitação
cases:
  - id: <string>                          # ex.: case-001
    description: <string>
    input:
      type: <string>                      # document | message | file | ...
      content: <path-para-fixture>
    expected:
      kind: <string>
      confidence: ">= 0.85"
    criteria:
      must_extract: [<string>, ...]       # campos obrigatórios
      max_latency_ms: <int>
      max_tokens: <int>
    tags: [functional, regression]
```

---

## Fluxo E→P→X→R

### Fase E — Especificação (autônoma)

```
[E1A] Receber missão do proprietário
[E2A] Analisar competências necessárias
      - consultar knowledge/erp/INDEX.md
      - consultar knowledge/ecosystem/papeis-agentes.md
      - identificar capacidades existentes (memory/agents-registry.md)
[E3A] Selecionar template base
      - usar templates/funcionario-digital/ como ponto de partida
      - customizar identidade, missão, responsabilidades
[E4A] Gerar arquivos:
      - agent.yaml (manifesto)
      - AGENTS.md (instruções executáveis, não aspiracionais)
      - tools.yaml (ferramentas mínimas necessárias)
      - tests/cases.yaml (pelo menos 3 casos: normal, borda, erro)
      - tests/fixtures/ (dados fictícios)
[E5A] spec.validate(agent_id) → validar contra schema
[E6A] Commit em branch isolada no workspace do Arquiteto
```

### Fase P — Provisionamento (autônomo após spec commitada)

```
[P1A] provision.create(agent_id, commit_sha)
      - cria instância em state/instances/<instance_id>.json
      - registra em state/registry.jsonl
      - prepara sandbox vazio
[P2A] Verificar tools.yaml:
      - cada ferramenta externa deve ter status=verified em permissions/granted.md
      - se não: abrir solicitação (ver estratégia-integracao-ferramentas.md) e suspender P
[P3A] Carregar MCP clients aprovados (read-only por padrão)
[P4A] Inicializar memória M2 scope=agent:<instance_id>
```

### Fase X — Execução em Sandbox (autônoma)

```
[X1A] execute.run(instance_id, case_id)
      - carrega spec + fixtures
      - inicia runtime isolado com interceptor
[X2A] Runtime executa tool loop até:
        - terminar (output produzido)
        - atingir limite (tokens, latência, iterações)
        - chamar ferramenta não autorizada → interceptor bloqueia + log
        - chamar host em denylist → bloqueio + suspensão
[X3A] Checkpoint de estado a cada tool call
[X4A] Ao término: test.evaluate(result, criteria) → score composto
[X5A] Persistir resultado em agents/<id>/tests/results/<case_id>.json
```

### Fase R — Correção (autônoma se falhar)

```
[R1A] Se score < threshold OU security violation detectada:
      - ler resultado + logs + intercepts bloqueados
[R2A] Diagnosticar causa:
      - prompt incompleto / tool ausente / permissão insuficiente / fixture inadequada
[R3A] Propor diff em branch:
      - agent.yaml | AGENTS.md | tools.yaml
[R4A] Aplicar diff, re-commit, re-provisionar (P1)
[R5A] Re-executar caso que falhou + casos de regressão
[R6A] Se todos passarem: voltar a Fase X para próximos casos
```

### Fase A — Ativação em Produção (exige humano; não implementada nesta fase)

```
[A1H] Humano revisa spec + resultados de testes + logs de auditoria
[A2H] Humano aprova promotion: status=production_candidate
[A3_] Reservado para o Orquestrador futuro (não implementado)
```

---

## Princípios de criação

1. **Um funcionário = uma competência principal.** Não crie "faz-tudo".
2. **Instruções executáveis, não aspiracionais.** "Seja especialista" é proibido; descreva procedimentos concretos.
3. **Ferramentas mínimas necessárias.** Conceda apenas o que o funcionário precisa (princípio least privilege).
4. **Testes antes de uso.** Funcionário sem testes de aceitação não está pronto.
5. **Documentação automática.** Gere README a partir da spec, não manualmente.
6. **Versionamento por commit SHA.** Cada spec tem um `parent_commit` que a identifica unicamente.

---

## Quando criar vs. reusar

Antes de criar um novo funcionário, verifique:

1. **Já existe um funcionário com competência similar?** Consulte `memory/agents-registry.md`.
2. **Pode-se estender um existente com nova skill?** Adicionar skill é mais barato que criar agente.
3. **A tarefa é recorrente o suficiente para justificar um funcionário dedicado?**

Se a resposta às três é "não", crie. Caso contrário, estenda o existente.

---

## Exemplo de `agent.yaml` (funcionário fictício)

```yaml
id: ficticio-classificador-fiscal
display_name: Classificador Fiscal (Fictício)
role: Especialista em classificação de documentos fiscais
version: 1.0.0
created_at: 2026-11-24T10:00:00Z
created_by: agente-arquiteto
parent_commit: abc123def456

mission: Classificar documentos fiscais fictícios em NF, Boleto, Recibo ou Outro, extraindo valor e data de pagamento.
responsibilities:
  - Classificar o tipo do documento
  - Extrair valor monetário
  - Extrair data de pagamento
  - Extrair CNPJ do emitente quando presente
limits:
  - Não acessa internet
  - Não armazena dados de documentos reais
  - Apenas dados fictícios em testes

provider:
  primary: codex
  fallback: [claude]
  model: gpt-5

permissions_profile: workspace_only

tools:
  file: tools.yaml

knowledge:
  folder: knowledge

tests:
  folder: tests
  threshold: 0.85
```

---

## Dependências

- Este documento depende de: `papeis-agentes.md`, `estrategia-integracao-ferramentas.md`, `principios-seguranca.md`.
- Skills relacionadas: `agent-design` (procedimento detalhado), `agent-evaluation` (testes).
