---
name: software-engineering
description: Analisar código, propor arquitetura, implementar alterações autorizadas em ambientes controlados, testar e documentar. Use quando precisar analisar repositório (com autorização), propor mudança no ERP B2A, implementar feature em fork isolado, ou revisar PR.
---

# Skill: Software Engineering

## Objetivo

Conduzir atividades de engenharia de software em ambientes controlados: análise de código, proposta arquitetural, implementação em fork isolado, testes, documentação e revisão de PR. Respeita rigorosamente P1 (independência do ERP por padrão), P4 (sem alteração direta em produção) e P10 (sem auto-permissão).

## Situações de uso

Use esta skill quando:

- O proprietário pede para analisar ou propor mudança no ERP B2A.
- Você precisa consultar repositório GitHub privado (com autorização).
- Uma proposta de alteração foi aprovada e precisa ser implementada.
- Um PR precisa ser revisado.
- Código de ferramenta própria precisa ser implementado ou corrigido.

**Não use** esta skill para:
- Pesquisar ferramentas open-source (use `tool-integration`).
- Criar skill de procedimento (use `skill-engineering`).
- Acessar ERP sem autorização (proibido).

## Entradas necessárias

- **Alvo:** repositório/arquivo/migration a analisar ou modificar.
- **Tipo de operação:** análise | proposta | implementação | revisão.
- **Autorização:** confirmação do proprietário (para qualquer acesso externo).

## Procedimento operacional

### Fluxo A — Análise de código (read-only)

#### A1: Confirmar autorização

Antes de qualquer acesso a repositório externo:

1. Verificar `permissions/granted.md` para acesso ativo ao repositório alvo.
2. Se não há acesso: abrir solicitação em `permissions/pending.md`.
3. Aguardar aprovação.
4. **Não** acessar sem aprovação.

#### A2: Clonar para workspace isolado

Após aprovação:

```bash
cd ~/OpenBot/Agents/agente-arquiteto/workspaces/external/
git clone --branch main --depth 1 https://github.com/<org>/<repo>.git
cd <repo>
```

**Regras:**
- Sempre `--depth 1` (shallow clone) para análise.
- Diretório isolado do workspace principal.
- Não criar branches neste clone durante análise.

#### A3: Análise estruturada

Para cada arquivo analisado, registrar em `memory/evolution-log.md`:

```markdown
## [DATA] — Análise: [arquivo]

**Arquivo:** [caminho]
**Commit:** [sha]
**Tipo:** [schema | route | lib | feature | migration | doc]

**Resumo técnico:**
[1-2 parágrafos objetivos]

**Capacidades identificadas:**
- [capacidade 1]

**Dependências:**
- [dep 1]

**Limitações observadas:**
- [limitação 1]

**Estado de verificação:** [VERIFICADO]
```

#### A4: Atualizar knowledge base

Se a análise revela informação nova sobre o ERP:

1. Propor atualização em `knowledge/erp/` (ver skill `erp-knowledge` Passo 5).
2. Atualizar `last_verified_at` no documento.
3. Commit da atualização em branch isolada do workspace do Arquiteto.

#### A5: Limpar

Após análise:
- Não deixar clone no workspace (exceto se implementação seguirá).
- Registrar em `memory/evolution-log.md` que análise concluída.

### Fluxo B — Proposta de alteração no ERP

#### B1: Identificar oportunidade

Via Modo EE ou pedido direto do proprietário.

#### B2: Análise técnica

1. Consultar `knowledge/erp/` para descrever:
   - O que existe hoje (com `source_ref` ao arquivo).
   - A mudança proposta.
   - Impacto, risco, estratégia de rollback.
2. Se necessário, executar Fluxo A para verificar fonte atual.

#### B3: Proposta formal

Escrever em `permissions/erp-changes-proposed.md`:

```markdown
## Proposta: [proposal_id]

**Target system:** erp.b2a
**Target artifact:** [caminho do arquivo/migration]
**Change type:** add | modify | remove
**Date proposed:** [ISO-8601]
**Proposed by:** agente-arquiteto

**Description:**
[descrição detalhada]

**Current state:**
[o que existe hoje, com source_ref]

**Proposed state:**
[o que deve ficar após mudança]

**Impact analysis:**
- Affected modules: [lista]
- Breaking changes: [sim/não + detalhe]
- Data migration: [necessária? detalhe]
- Performance: [impacto esperado]
- Security: [impacto esperado]

**Rollback plan:**
[como reverter se der errado]

**Test plan:**
- [teste 1]
- [teste 2]

**Estimated effort:** [S/M/L/XL] ([N] dias)

**Status:** pending_approval
```

#### B4: Apresentar ao proprietário

Usar `ask_user` para notificar. Apresentar resumo da proposta.

#### B5: Aguardar decisão

- **Não** executar nada até aprovação explícita.
- Registrar decisão em `permissions/erp-changes-proposed.md`.

### Fluxo C — Implementação em fork isolado (após aprovação)

#### C1: Receber credenciais temporárias

O proprietário concede:
- Acesso ao fork (não ao repo main).
- Branch isolada para trabalho.
- Janela de tempo limitada.
- Scope Git: `[read, commit-to-branch]`.

**Regras:**
- **Nunca** clone o repo main diretamente.
- **Nunca** faça push para `main`.
- **Nunca** faça merge direto.

#### C2: Configurar fork isolado

```bash
cd ~/OpenBot/Agents/agente-arquiteto/workspaces/erp-b2a-fork/
git clone --branch [branch-autorizada] https://github.com/<fork>.git
cd <repo>
git checkout -b feature/[nome-descritivo]
```

#### C3: Implementar mudança

- Seguir `ESTRUTURA_PADRAO_DESENVOLVIMENTO.md` do ERP (feature folder pattern).
- Não adicionar comentários ao código (regra do `AGENTS.md` do ERP).
- Não adicionar dependências sem validação.
- Usar `.split("T")[0]` para datas.
- Validação Zod quando aplicável.

#### C4: Testar localmente

- `npm run typecheck` (em Node 22+).
- `npm run build` (deve passar).
- `npm run lint` (pode falhar por incompatibilidade ESLint 10 — documentar).
- Testes específicos da área alterada (se existirem).

#### C5: Validar RLS e migrations

Se a mudança envolve migration:
- Verificar que RLS está habilitada.
- Verificar que policies são apropriadas.
- Preparar plano de rollback.

#### C6: Commit em branch

```bash
git add .
git commit -m "[scope] Descrição da mudança

[Body explicando rationale]

Refs: [proposal_id]
Reviewed-by: pending"
```

#### C7: Push para fork

```bash
git push origin feature/[nome-descritivo]
```

#### C8: Abrir PR

Criar PR do fork para o repo main:
- Título: `[scope] Descrição`
- Body: seguir template do ERP.
- Referenciar `proposal_id`.
- Listar testes executados.
- Declarar limitações não verificadas.

**Não** fazer merge. Aguardar revisão humana.

#### C9: Revisão pelo proprietário

- Proprietário revisa diff.
- Pode aprovar merge, pedir ajustes, ou rejeitar.

Se pedir ajustes: voltar a C3.
Se aprovar merge: proprietário faz merge (não o Arquiteto).
Se rejeitar: descartar branch, atualizar knowledge base com razão da rejeição.

#### C10: Pós-merge

Após merge pelo proprietário:
1. Atualizar `knowledge/erp/` com nova versão do artifact.
2. Registrar em `memory/decisions.md`.
3. Atualizar `permissions/erp-changes-proposed.md` com status `merged`.

#### C11: Expirar acesso

Após janela de acesso:
- Credenciais temporárias revogadas pelo proprietário.
- Arquiteto retorna ao estado default: zero acesso ao ERP B2A.
- Registrar expiração em `permissions/granted.md`.

### Fluxo D — Revisão de PR

#### D1: Receber PR para revisão

Proprietário pede para revisar PR de terceiro (ex.: desenvolvedor humano).

#### D2: Análise do PR

1. Ler descrição do PR.
2. Ler diff completo.
3. Para cada arquivo alterado:
   - Verificar que segue `ESTRUTURA_PADRAO_DESENVOLVIMENTO.md`.
   - Verificar que não introduz comentários (regra do ERP).
   - Verificar que não bypass CSP.
   - Verificar que não usa `@ts-ignore` sem justificativa.
   - Verificar que não adiciona dependência sem necessidade.
4. Validar que testes passam (se aplicável).
5. Verificar que RLS/migrations estão adequadas.

#### D3: Relatório de revisão

```markdown
## Revisão de PR: [PR URL]

**Autor:** [autor]
**Arquivos alterados:** [N]
**Linhas:** +[X] -[Y]

### Achados
- [achado 1: severidade, arquivo, linha]

### Recomendação
[approve | request_changes | reject]

### Rationale
[1 parágrafo]
```

#### D4: Submeter revisão

- Postar revisão como comentário no PR (se autorizado).
- Ou apresentar ao proprietário via `ask_user`.

## Ferramentas necessárias

- `Read`, `Write`, `Edit`, `Glob`, `Grep` (manipular código).
- `Bash` (git, npm, typecheck, build).
- OpenBot nativas: `ask_user`, `remember`, `search_memories`.

## Limites de autorização

- **Pode executar autonomamente:**
  - Implementar ferramenta própria em `tools/developed/`.
  - Analisar código no próprio workspace do Arquiteto.
  - Propor alteração no ERP (sem executar).

- **Exige aprovação do proprietário:**
  - Acessar repositório GitHub externo (mesmo read-only).
  - Implementar mudança no ERP B2A (em fork).
  - Abrir PR para merge.
  - Instalar nova dependência.

- **Proibido:**
  - Acessar ERP B2A sem autorização específica.
  - Fazer merge direto em `main` de qualquer repo.
  - Push para `main` de qualquer repo.
  - Alterar produção diretamente.
  - Auto-conceder acesso a si próprio.

## Critérios de qualidade

- Análise cita `source_ref` para toda afirmação.
- Proposta tem plano de rollback explícito.
- Implementação segue padrões do ERP (`ESTRUTURA_PADRAO_DESENVOLVIMENTO.md`).
- Testes executados antes de PR.
- PR tem descrição clara e referência à proposal.
- Sem comentários no código (regra do ERP).
- Sem `@ts-ignore` sem justificativa.

## Critérios de conclusão

A skill é concluída quando:

- Para análise: relatório registrado em `memory/evolution-log.md`.
- Para proposta: `permissions/erp-changes-proposed.md` atualizado com `pending_approval`.
- Para implementação: PR aberto, aguardando revisão humana.
- Para revisão: relatório submetido.
- Para todos: knowledge base atualizada se aplicável.

## Tratamento de erros

| Erro | Ação |
|---|---|
| Acesso ao repo negado | Verificar `permissions/granted.md`; se expirou, solicitar renovação |
| Build falha | Diagnosticar, corrigir, re-buildar |
| TypeCheck falha | Corrigir tipos, não usar `@ts-ignore` como atalho |
| Teste falha | Diagnosticar, corrigir implementação ou teste, re-executar |
| Merge conflict | Rebase com main, resolver conflitos, re-push |
| PR rejeitado | Descartar branch, registrar razão, atualizar knowledge base |
| Acesso expirou durante implementação | Salvar progresso em branch, solicitar renovação |

## Exemplos práticos

### Exemplo 1: Propor preencher campos de conciliação

**Cenário:** Lacuna L2 em `knowledge/erp/09-limitacoes-problemas.md` — `conciliacao_financeira` tem campos vazios.

**Fluxo B:**
1. B1: Identificar via Modo EE.
2. B2: Análise técnica — consultar `knowledge/erp/03-processos.md` seção "Processo 7" e `05-modelo-dados.md`.
3. B3: Proposta formal em `permissions/erp-changes-proposed.md`.
4. B4: Apresentar ao proprietário.
5. B5: Aguardar decisão.

### Exemplo 2: Implementar após aprovação

**Cenário:** Proposta erp-change-001 aprovada.

**Fluxo C:**
1. C1: Receber credenciais do fork.
2. C2: Clonar fork em `workspaces/erp-b2a-fork/`.
3. C3: Implementar mudança em `src/lib/financial-service.ts`.
4. C4: Testar (`npm run typecheck` ✓, `npm run build` ✓).
5. C5: Sem migrations envolvidas.
6. C6: Commit em `feature/conciliacao-auto-match`.
7. C7: Push para fork.
8. C8: Abrir PR com referência a `erp-change-001`.
9. C9: Aguardar revisão humana.
10. C10 (após merge): Atualizar `knowledge/erp/03-processos.md` e `08-estado-implementacao.md`.
11. C11: Acesso expira, retorno ao estado default.

## Referências a documentos de conhecimento

- `knowledge/erp/09-limitacoes-problemas.md` — lacunas que podem gerar propostas.
- `permissions/erp-changes-proposed.md` — propostas formais.
- `permissions/granted.md` — acessos ativos.
- `memory/evolution-log.md` — histórico de análises e mudanças.
