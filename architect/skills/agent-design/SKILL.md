---
name: agent-design
description: Projetar e criar funcionários digitais completos, incluindo identidade, instruções, competências, ferramentas, conhecimento e testes. Use quando o proprietário pedir para criar um novo funcionário digital, ou quando uma necessidade recorrente justificar um agente especializado.
---

# Skill: Agent Design

## Objetivo

Conduzir o fluxo completo de criação de um funcionário digital: especificação (E), provisionamento (P), execução em sandbox (X), correção (R). Aplicar o template padrão, garantir qualidade mínima via testes e documentar para uso contínuo.

## Situações de uso

Use esta skill quando:

- O proprietário pede para criar um funcionário digital.
- Uma necessidade recorrente justifica um agente especializado.
- Um funcionário existente precisa ser re-criado em nova versão major.

**Não use** esta skill para:
- Apenas adicionar skill a funcionário existente (use `skill-engineering`).
- Pesquisar ferramentas para o funcionário (use `tool-integration`).
- Avaliar funcionário já criado (use `agent-evaluation`).

## Entradas necessárias

- **Missão:** o que o funcionário deve fazer (1 frase).
- **Competências:** que áreas de conhecimento ele precisa.
- **Ferramentas:** com que sistemas ele precisa interagir.
- **Restrições:** limites explícitos (o que não pode fazer).
- **Critérios de aceite:** como saber se ele está pronto.

## Procedimento operacional

### Fase E — Especificação

#### E1: Receber e confirmar missão

Receba a missão do proprietário. Repita em uma frase para confirmar entendimento:

```
Missão recebida: [frase precisa]
Tipo: [criação de funcionário]
Competências principais: [lista]
Ferramentas necessárias: [lista ou "a identificar"]
Restrições: [lista ou "padrão"]
```

Aguarde confirmação antes de prosseguir.

#### E2: Analisar competências necessárias

1. Consulte `knowledge/erp/INDEX.md` para identificar capacidades existentes no ERP relevantes.
2. Consulte `memory/agents-registry.md` para identificar funcionários existentes com competências similares.
3. Para cada competência necessária, identifique:
   - **Conhecimento:** qual entrada de `knowledge/erp/` ou `knowledge/ecosystem/` é relevante.
   - **Ferramentas:** quais tools (internas, MCPs, desenvolvidas) são necessárias.
   - **Permissões:** qual scope de acesso é mínimo necessário.

#### E3: Selecionar template base

Use `templates/funcionario-digital/` como ponto de partida. Customize:

- `agent.yaml` — identidade, missão, responsabilidades, limites, provider, model.
- `AGENTS.md` — instruções executáveis (não aspiracionais).
- `tools.yaml` — ferramentas mínimas necessárias.
- `tests/cases.yaml` — pelo menos 3 casos (normal, borda, erro).
- `tests/fixtures/` — dados fictícios.

#### E4: Gerar arquivos

**Regras para `AGENTS.md` (instruções):**
- Descreva procedimentos concretos, não adjetivos.
- "Seja especialista" é proibito.
- Use estrutura: Identidade → Missão → Princípios → Procedimentos → Limites.
- Inclua "Quando executar autonomamente vs. solicitar autorização".
- Máximo 2000 palavras (mais que isso sobrecarrega contexto).

**Regras para `tools.yaml`:**
- Apenas ferramentas com `status: verified` em `permissions/tools-catalog.yaml`.
- `permissions_used` deve ser subset do que a ferramenta permite.
- `scope` deve ser subset do `approved_scope` em `permissions/granted.md`.
- Se ferramenta necessária não está aprovada: abra solicitação em `permissions/pending.md` e suspenda P.

**Regras para `tests/cases.yaml`:**
- Mínimo 3 casos por funcionário.
- Casos cobrem: situação normal, situação de borda, situação de erro.
- Cada caso tem: input, expected, criteria, tags.
- Fixtures em `tests/fixtures/` são fictícios.

#### E5: Validar spec

Execute `spec.validate(agent_id)`:
- Verificar que `agent.yaml` segue schema.
- Verificar que `AGENTS.md` existe e tem menos de 2000 palavras.
- Verificar que `tools.yaml` referencia apenas tools aprovadas (ou abre solicitação).
- Verificar que `tests/cases.yaml` tem pelo menos 3 casos.

Se violações: corrija antes de prosseguir.

#### E6: Commit em branch isolada

```
git checkout -b agents/<agent-id>/v1
git add agents/<agent-id>/
git commit -m "feat(agents): create <agent-id> v1.0.0

Mission: [frase]
Competencies: [lista]
Tools: [lista]

Created by: agente-arquiteto"
```

### Fase P — Provisionamento

#### P1: Criar instância

Registre em `state/instances/<instance_id>.json` e em `state/registry.jsonl` (append-only).

#### P2: Verificar ferramentas

Para cada ferramenta em `tools.yaml`:
- Se `internal`: OK.
- Se `mcp`/`http`/`cli`: verificar `status: verified` em `permissions/granted.md`.
- Se não verificado: suspender P, abrir solicitação em `permissions/pending.md`.

#### P3: Carregar MCP clients

Para cada MCP aprovado: carregar em modo read-only por padrão. Write requer autorização específica.

#### P4: Inicializar memória

Criar `agents/<agent-id>/memory/session.jsonl` (append-only) com entrada inicial.

### Fase X — Execução em Sandbox

#### X1: Iniciar runtime

Carregar spec + fixtures. Iniciar runtime isolado com interceptor de autorização ativo.

#### X2: Executar tool loop

O runtime executa até:
- Terminar (output produzido).
- Atingir limite (tokens, latência, iterações).
- Chamar ferramenta não autorizada → interceptor bloqueia + log.
- Chamar host em denylist → bloqueio + suspensão.

#### X3: Checkpoint

Salvar estado a cada tool call em `state/instances/<instance_id>.json`.

#### X4: Avaliar

`test.evaluate(result, criteria)`:
- Corretude (50%): campos corretos vs esperado.
- Completude (20%): campos esperados presentes.
- Eficiência (20%): tokens + latência vs orçamento.
- Segurança (10%): zero chamadas não autorizadas.

Score composto: média ponderada.

**Threshold padrão:** ≥ 0.85.
**Regra eliminatória:** qualquer falha em T0 reprova.

#### X5: Persistir resultado

Salvar em `agents/<agent-id>/tests/results/<case_id>.json`.

### Fase R — Correção

#### R1: Detectar falha

Se score < 0.85 OU security violation detectada OU T0 falhou:
- Ler resultado + logs + intercepts bloqueados.

#### R2: Diagnosticar causa

Categorias comuns:
- **Prompt incompleto:** `AGENTS.md` não especifica regra necessária.
- **Tool ausente:** `tools.yaml` não inclui ferramenta necessária.
- **Permissão insuficiente:** `permissions_used` muito restritivo.
- **Fixture inadequado:** `tests/fixtures/` não cobre cenário.

#### R3: Propor diff

Em branch nova (`agents/<agent-id>/v2`):
- Modificar `agent.yaml` (raro).
- Modificar `AGENTS.md` (comum).
- Modificar `tools.yaml` (comum).
- Modificar `tests/cases.yaml` (às vezes).

#### R4: Re-commit, re-provisionar

#### R5: Re-executar

Re-executar caso que falhou + casos de regressão (todos os anteriores).

#### R6: Se todos passarem

Voltar a Fase X para próximos casos. Quando todos passam, funcionário está pronto para apresentação.

### Apresentação ao proprietário

Ao final de E→P→X→R com todos os casos passando:

```
## Resultado: Criação do funcionário <agent-id>

**Status:** concluído (em sandbox)
**Artefatos:** [lista de arquivos]

### O que foi feito
- Spec criada a partir do template padrão.
- [N] casos de teste executados, todos com score ≥ 0.85.
- T0 (8 testes eliminatórios) passou 100%.

### Decisões tomadas
- Provider: [escolha e rationale]
- Ferramentas: [lista e rationale]
- Limites: [lista]

### Próximo passo sugerido
[Aguardando sua revisão.]
```

## Ferramentas necessárias

- `Read`, `Write`, `Edit`, `Glob`, `Grep` (manipular spec).
- `templates/funcionario-digital/` (template base).
- `permissions/tools-catalog.yaml` (catálogo de ferramentas).
- `permissions/granted.md` (acessos aprovados).
- OpenBot nativas: `ask_user`, `remember`, `search_memories`.

## Limites de autorização

- **Pode executar autonomamente:**
  - Criar spec para funcionário fictício (prefixo `ficticio-`).
  - Provisionar instância em sandbox.
  - Executar testes com dados fictícios.
  - Corrigir spec em branch isolada.

- **Exige aprovação do proprietário:**
  - Criar funcionário de produção (sem prefixo `ficticio-`).
  - Adicionar ferramenta externa não aprovada ao `tools.yaml`.
  - Acessar dados reais do ERP.

- **Proibido:**
  - Criar funcionário com `permissions_profile: full_access`.
  - Auto-promover funcionário a produção.
  - Criar funcionário sem testes de aceitação.

## Critérios de qualidade

- Spec segue template padrão.
- Instruções em `AGENTS.md` são executáveis (não aspiracionais).
- `tools.yaml` tem apenas ferramentas aprovadas (ou solicita pendentes).
- Mínimo 3 casos de teste (normal, borda, erro).
- Score ≥ 0.85 em todos os casos.
- T0 (8 eliminatórios) passa 100%.
- Documentação `README.md` gerada.

## Critérios de conclusão

A skill é concluída quando:

- Funcionário provisionado e testado em sandbox.
- Todos os casos de teste passam com score ≥ 0.85.
- T0 passa 100%.
- Resultado apresentado ao proprietário.
- Entrada adicionada em `memory/agents-registry.md`.
- Funcionário aguarda decisão do proprietário para uso contínuo ou evolution.

## Tratamento de erros

| Erro | Ação |
|---|---|
| Ferramenta necessária não aprovada | Suspender P, abrir solicitação, continuar outras tarefas |
| Score < 0.85 em caso | Iniciar Fase R |
| T0 falhou | Suspender instância imediatamente, diagnosticar, correção obrigatória |
| Host em denylist acessado | Suspensão automática, log em `memory/incidents.md`, notificar proprietário |
| Limite de tokens excedido | Reduzir escopo do caso ou otimizar prompt |

## Exemplos práticos

### Exemplo 1: Funcionário fictício de classificação fiscal

**Missão:** "Crie um funcionário fictício que classifique documentos fiscais em NF, Boleto, Recibo ou Outro, extraindo valor e data de pagamento."

**Fluxo:**
1. E1: Confirmar missão. Identificar como criação de funcionário fictício.
2. E2: Consultar `knowledge/erp/03-processos.md` seção "Processo 9: Workflow n8n — OCR Financeiro" — existe parser similar que pode inspirar.
3. E3: Copiar template de `templates/funcionario-digital/`.
4. E4: Gerar arquivos:
   - `agent.yaml`: id=`ficticio-classificador-fiscal`, provider=codex, permissions=workspace_only.
   - `AGENTS.md`: procedimento de classificação com regras explícitas (não "seja especialista").
   - `tools.yaml`: apenas `internal.classify` (sem MCPs externos).
   - `tests/cases.yaml`: 3 casos com fixtures de NF, Boleto, Recibo.
5. E5: Validar spec. OK.
6. E6: Commit em `agents/ficticio-classificador-fiscal/v1`.
7. P1: Criar instância `inst-001`.
8. P2: Verificar ferramentas. Apenas internas, OK.
9. X1: Executar caso 001 (NF).
10. X2: Runtime classifica como "NF", extrai valor e data.
11. X4: Score 0.69 (data errada — extraiu `data_emissao` em vez de `data_pagamento`).
12. R1: Detectar falha.
13. R2: Diagnosticar — prompt não diferencia `data_emissao` de `data_pagamento`.
14. R3: Diff em `AGENTS.md` adicionando regra explícita.
15. R4: Re-commit, re-provisionar `inst-002`.
16. R5: Re-executar caso 001 + casos 002, 003. Todos passam.
17. Apresentar resultado ao proprietário.

## Referências a documentos de conhecimento

- `knowledge/ecosystem/padrao-criacao-funcionarios.md` — fluxo E→P→X→R detalhado.
- `knowledge/ecosystem/papeis-agentes.md` — papéis no ecossistema.
- `knowledge/ecosystem/principios-seguranca.md` — T0 eliminatórios.
- `templates/funcionario-digital/` — template base.
- `memory/agents-registry.md` — funcionários existentes.
