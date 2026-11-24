---
name: tool-integration
description: Pesquisar, comparar, configurar e integrar ferramentas open-source, APIs, bibliotecas e MCPs ao ecossistema de agentes. Use quando precisar adicionar capacidade externa a um funcionário, avaliar alternativa open-source antes de desenvolver ferramenta própria, ou configurar conexão autorizada.
---

# Skill: Tool Integration

## Objetivo

Conduzir o ciclo completo de integração de ferramentas externas: pesquisa → avaliação → solicitação de autorização → configuração → verificação → uso auditado. Prioriza reuso de soluções existentes (P5) e respeita o princípio de acesso progressivo (P3).

## Situações de uso

Use esta skill quando:

- Um funcionário precisa de ferramenta externa (MCP, API, biblioteca).
- Você precisa pesquisar open-source antes de propor desenvolvimento próprio.
- Uma ferramenta precisa ser configurada após autorização.
- Uma conexão autorizada precisa ser verificada.
- Uma ferramenta existente precisa ser atualizada ou substituída.

**Não use** esta skill para:
- Desenvolver ferramenta própria do zero (use `skill-engineering` + `software-engineering`).
- Criar funcionário (use `agent-design`).
- Acessar ERP B2A (use `software-engineering` com fluxo especial do AGENTS.md §11).

## Entradas necessárias

- **Necessidade técnica:** o que a ferramenta deve fazer (1 frase precisa).
- **Contexto de uso:** qual funcionário/tarefa precisa.
- **Restrições:** licença, custo, latência, etc.

## Procedimento operacional

### Passo 1: Definir necessidade técnica

Formule em uma frase precisa, evitando vagueza:

**Exemplo bom:** "Funcionário X precisa listar issues de org/repo-y no GitHub e ler conteúdo de arquivos específicos."

**Exemplo ruim:** "Funcionário X precisa acessar o GitHub."

### Passo 2: Pesquisar candidatos

#### 2.1: Pesquisar MCP servers públicos

Fontes:
- `https://modelcontextprotocol.io/servers` (catálogo oficial).
- `https://github.com/modelcontextprotocol/servers` (repo oficial).
- `https://openbot.run/plugins/` (catálogo OpenBot).
- Busca web: `"<necessidade> MCP server"`.

Para cada candidato:
- Nome e repositório.
- Licença (MIT, Apache 2.0 preferencial).
- Stars GitHub.
- Última atualização (deve ser < 6 meses).
- Documentação clara?
- Mantido por organização confiável?

#### 2.2: Pesquisar bibliotecas open-source

Fontes:
- npm (JavaScript/TypeScript): `https://www.npmjs.com/search?q=<termo>`.
- PyPI (Python): `https://pypi.org/search/?q=<termo>`.
- crates.io (Rust): `https://crates.io/search?q=<termo>`.

Para cada candidato:
- Nome e repositório.
- Licença.
- Downloads/semana.
- Última versão (deve ser < 6 meses).
- Documentação?
- Issues abertas vs fechadas?

#### 2.3: Pesquisar APIs REST

- Documentação pública do serviço alvo.
- Suporte a OAuth 2.0 (preferencial) ou API key.
- Rate limits documentados.
- SDK oficial disponível?

### Passo 3: Avaliar candidatos

Para cada candidato viável, preencha tabela em `memory/research-log.md`:

```markdown
## [DATA] — Pesquisa: [necessidade técnica]

**Necessidade:** [frase precisa]
**Contexto:** [funcionário/tarefa]

**Candidatos MCP:**
| Candidato | Licença | Stars | Última atualização | Viável? | Por quê? |
|---|---|---|---|---|---|
| [nome] | [licença] | [n] | [data] | [sim/não] | [razão] |

**Candidatos bibliotecas:**
| Candidato | Licença | Downloads/sem | Última versão | Viável? | Por quê? |
|---|---|---|---|---|---|
| [nome] | [licença] | [n] | [data] | [sim/não] | [razão] |

**Candidatos APIs:**
| Candidato | Auth | Rate limit | SDK | Viável? | Por quê? |
|---|---|---|---|---|---|
| [nome] | [OAuth/API key] | [limites] | [sim/não] | [sim/não] | [razão] |

**Decisão:** [reusar X | desenvolver próprio]
**Rationale:** [justificativa detalhada]
```

### Passo 4: Decidir entre reusar vs desenvolver próprio

Aplicar P5 (reuso antes de construir):

| Condição | Decisão |
|---|---|
| MCP server atende 100% dos requisitos | Reusar MCP |
| Biblioteca open-source atende + fácil de embrulhar | Reusar biblioteca |
| API REST bem documentada + SDK oficial | Reusar API |
| Nenhum candidato atende, mas candidate próximo existe com pequeno gap | Considerar fork + PR upstream |
| Nenhum candidato minimamente viável | Desenvolver próprio (use `skill-engineering`) |

**Regra:** se decidir desenvolver próprio, **documente na pesquisa** por que nenhum candidato atende.

### Passo 5: Solicitar autorização (se reusar)

Se reusar ferramenta externa, abrir solicitação em `permissions/pending.md`:

```markdown
## Solicitação: [tool_id]

**Recurso:** [MCP/biblioteca/API]
**Motivo:** [tarefa concreta que depende deste acesso]
**Escopo solicitado:**
  - operations: [lista específica, ex.: "list_issues", "read_file"]
  - repositories: [lista, se GitHub]
  - rate: [limite máximo por minuto]
**Risco:** [avaliação honesta: read-only/write, escopo, latência]
**Alternativa sem acesso:** [o que acontece se negado]
**Risk assessment:** [read-only scoped é baixo; write é médio; admin é alto]
**Pesquisa:** [link para entrada em memory/research-log.md]
**Requested at:** [ISO-8601]
**Status:** pending_authorization
```

### Passo 6: Aguardar aprovação

- **Não prossiga** com a tarefa que depende do acesso.
- Pode continuar outras tarefas na mesma missão.
- Use `ask_user` para notificar proprietário sobre a solicitação pendente.

### Passo 7: Configurar conexão (após aprovação)

Após proprietário aprovar em `permissions/granted.md`:

```markdown
## Granted: [tool_id]

**Approved by:** [proprietário]
**Approved at:** [ISO-8601]
**Approved scope:**
  - operations: [lista final, pode ser subset do solicitado]
  - repositories: [lista final]
  - rate: [limite final]
**Expires at:** [data de renovação, default 90 dias para read, 30 para write]
**Status:** verified (após passo 9) | pending_verification (antes)
```

Registrar conexão (sem secrets):

```yaml
# permissions/tools-catalog.yaml
- id: [tool_id]
  kind: mcp | http | cli
  status: verified
  connection:
    transport: stdio | http | sse
    auth_method: oauth2 | pat | none
    secret_ref: env:GITHUB_TOKEN    # referência simbólica, nunca o valor
    scope: [escopo aprovado]
  last_approved_by: [proprietário]
  last_approved_at: [data]
  last_verified_at: [data]
```

**Regras:**
- Secrets **nunca** em arquivo versionado.
- Usar OpenBot password manager integration quando possível (1Password, Bitwarden).
- Para GitHub: usar OpenBot GitHub Connector com device flow.
- Para APIs com API key: usar `request_api_key` tool nativa.

### Passo 8: Verificar conexão

Execute `tool.verify(connection_id)`:

1. **Ping MCP server** (list_tools) — confirmar que está respondendo.
2. **Chamar 1 operação read-only com dados fictícios** — confirmar que auth funciona.
3. **Validar resposta** — schema esperado está correto?
4. **Verificar scope** — nenhuma chamada extrapola o aprovado?
5. **Logar** — latência, tokens, success/falha em `memory/incidents.md` (se falha) ou `memory/evolution-log.md` (se sucesso).

Se verify OK: marcar `status: verified` em `permissions/granted.md`.
Se verify falha: manter `pending_verification`, diagnosticar, refazer config ou propor alternativa.

### Passo 9: Disponibilizar para funcionários

Adicionar entrada em `permissions/tools-catalog.yaml` com `status: available`.

Funcionários podem agora referenciar a ferramenta em seus `tools.yaml`:
```yaml
tools:
  - id: [tool_id]
    permissions_used: [read]    # subset do aprovado
    scope: [escopo específico do funcionário, subset do aprovado]
```

### Passo 10: Uso auditado

Cada chamada da ferramenta por funcionário:
1. Interceptor valida `identity + scope + permission` antes de despachar.
2. Se bloqueada: log `kind=security_violation` em `memory/incidents.md`.
3. Se sucesso: log `kind=action` em `memory/agents-registry.md` (entrada da instância).
4. Rate limit enforced pelo interceptor.

### Passo 11: Renovação e revogação

- **Renovação:** 90 dias (read) ou 30 dias (write) antes da expiração, re-apresentar ao proprietário.
- **Revogação antecipada:** proprietário marca `status: denied` em `permissions/granted.md`. Interceptor bloqueia imediatamente.
- **Atualização de versão:** se MCP server lança nova versão, avaliar compatibilidade antes de atualizar.

## Ferramentas necessárias

- `Read`, `Write`, `Edit` (manipular arquivos de permissão).
- `Grep`, `Glob` (validar referências).
- OpenBot nativas: `ask_user`, `request_api_key`, `remember`.
- Acesso à web para pesquisa (via OpenBot browser embutido ou `WebSearch` se disponível).

## Limites de autorização

- **Pode executar autonomamente:**
  - Pesquisar candidatos em fontes públicas.
  - Registrar pesquisa em `memory/research-log.md`.
  - Abrir solicitação em `permissions/pending.md`.
  - Verificar conexão após aprovação (com dados fictícios).

- **Exige aprovação do proprietário:**
  - Conceder acesso (approve em `permissions/granted.md`).
  - Configurar credencial real (token, OAuth).
  - Usar ferramenta em produção.

- **Proibido:**
  - Usar ferramenta antes de verify OK.
  - Armazenar credenciais em arquivo.
  - Acessar host em denylist (ver `principios-seguranca.md`).
  - Auto-aprovar solicitação.

## Critérios de qualidade

- Pesquisa documentada com pelo menos 3 candidatos avaliados.
- Decisão entre reusar vs desenvolver justificada por escrito.
- Solicitação de autorização tem escopo mínimo necessário.
- Conexão configurada sem secrets em arquivo.
- Verify executado com sucesso antes de uso.
- Uso auditado (logs em memória).

## Critérios de conclusão

A skill é concluída quando:

- Ferramenta verificada (`status: verified` em `permissions/granted.md`).
- Entrada em `permissions/tools-catalog.yaml` com `status: available`.
- Funcionários podem referenciar em seus `tools.yaml`.
- Pesquisa registrada em `memory/research-log.md`.
- Acesso com data de renovação definida.

## Tratamento de erros

| Erro | Ação |
|---|---|
| Nenhum candidato viável encontrado | Documentar, propor desenvolvimento próprio (`skill-engineering`) |
| Solicitação negada pelo proprietário | Respeitar, suspender tarefa dependente, não insistir |
| Verify falha (MCP não responde) | Diagnosticar auth/transport/scope, reconfigurar, re-verify |
| Verify falha (scope excedido) | Reduzir scope solicitado, re-abrir solicitação |
| Host em denylist | Bloquear imediatamente, não prosseguir, notificar proprietário |
| Token expira | Renovar via fluxo de renovação, não auto-renovar |

## Exemplos práticos

### Exemplo 1: Funcionário precisa listar issues do GitHub

**Cenário:** Funcionário "classificador-issue-001" precisa listar issues de `org/repo-y` para classificá-las.

**Fluxo:**
1. **Passo 1:** Necessidade precisa: "Listar issues de org/repo-y no GitHub com filtros básicos (state, labels)."
2. **Passo 2:** Pesquisar:
   - MCP: `@modelcontextprotocol/server-github` (MIT, 5k stars, mantido).
   - Biblioteca: `octokit` (MIT, 8k stars, mantido pela GitHub).
   - API: GitHub REST API v4 (OAuth suportado, rate limit 5000/h).
3. **Passo 3:** Avaliar — MCP é mais simples (descoberta automática), API é mais flexível.
4. **Passo 4:** Decisão: reusar MCP (atende 100%, menos configuração).
5. **Passo 5:** Solicitar em `permissions/pending.md`.
6. **Passo 6:** Aguardar aprovação.
7. **Passo 7 (após approval):** Configurar via OpenBot GitHub Connector (device flow). Secret em `env:GITHUB_TOKEN`.
8. **Passo 8:** Verify: listar issues de org/repo-y. OK.
9. **Passo 9:** Disponibilizar em `permissions/tools-catalog.yaml`.
10. **Passo 10:** Funcionário referencia em `tools.yaml`.

### Exemplo 2: Pesquisa leva a reusar biblioteca

**Cenário:** Funcionário precisa validar CPF com dígito verificador brasileiro.

**Fluxo:**
1. **Passo 1:** "Validar CPF brasileiro incluindo dígitos verificadores."
2. **Passo 2:** Pesquisar:
   - MCP: nenhum específico para validação CPF.
   - Bibliotecas: `cpf-validator` (npm, 50 stars, mantido).
   - APIs: nenhuma pública gratuita.
3. **Passo 3:** Avaliar — `cpf-validator` atende, mas é simples o suficiente para inline.
4. **Passo 4:** Decisão: **reusar biblioteca** (não desenvolver próprio — P5 satisfeito).
5. Continuar com fluxo de integração.

### Exemplo 3: Atualizar versão de MCP

**Cenário:** MCP `@modelcontextprotocol/server-github` lançou v2.0 com breaking changes.

**Fluxo:**
1. Ler changelog da nova versão.
2. Avaliar impacto nos funcionários que usam.
3. Testar em sandbox com dados fictícios.
4. Se compatível: atualizar, re-verify, registrar em `memory/evolution-log.md`.
5. Se incompatível: avaliar manter versão atual ou migrar com ajustes.

## Referências a documentos de conhecimento

- `knowledge/ecosystem/estrategia-integracao-ferramentas.md` — política completa.
- `knowledge/ecosystem/principios-seguranca.md` — interceptor, denylist.
- `permissions/tools-catalog.yaml` — catálogo de ferramentas.
- `permissions/pending.md` — solicitações pendentes.
- `permissions/granted.md` — acessos concedidos.
- `memory/research-log.md` — histórico de pesquisas.
