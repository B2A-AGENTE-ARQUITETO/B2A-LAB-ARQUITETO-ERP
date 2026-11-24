# Estratégia de Integração de Ferramentas

**Última verificação:** 2026-11-24

## Sumário

Política de pesquisa, avaliação, integração e desenvolvimento de ferramentas para o ecossistema de agentes. Prioridade: **reuso antes de construir** (P5).

---

## Tipos de ferramenta suportados

| `kind` | Mecanismo | Exemplos | Verificado? |
|---|---|---|---|
| `internal` | Implementação nativa do runtime OpenBot | `ask_user`, `remember`, `search_memories` | Sim, nativo |
| `mcp` | MCP server externo (Model Context Protocol) | GitHub MCP, Filesystem MCP, Postgres MCP | Padrão aberto maduro |
| `http` | Cliente HTTP com auth header | APIs REST qualquer | Fallback para sistemas sem MCP |
| `cli` | Subprocesso local com allowlist de args | `git`, `rg`, `node` | Fallback |
| `architect_developed` | Implementação própria em `tools/developed/<tool_id>/` | Parser customizado, validador específico | Quando nada existente atende |

---

## Política de reuso (P5 enforcement)

Antes de propor desenvolvimento de ferramenta própria, o Arquiteto **deve**:

1. **Pesquisar MCP servers públicos** no catálogo comunitário (`https://modelcontextprotocol.io/servers` ou equivalentes).
2. **Pesquisar bibliotecas open-source relevantes** (npm, PyPI, crates.io) com:
   - Licença compatível (MIT, Apache 2.0, BSD preferencial).
   - Manutenção ativa (commits recentes, issues respondidas).
   - Documentação clara.
   - Comunidade ativa (stars > 100, última atualização < 6 meses).
3. **Avaliar viabilidade técnica** de cada candidato contra os requisitos.
4. **Documentar por que nenhum atende** (se for o caso) em `memory/research-log.md`.
5. **Então** propor desenvolvimento próprio.

---

## MCP como camada preferencial

**Por que MCP:**
- Padrão aberto maduro (spec 2025-06-18, doado à Linux Foundation em dez/2025).
- SDKs em todas as linguagens principais.
- Permite descoberta de ferramentas (não precisa hardcodar).
- Compatível com múltiplos modelos (Anthropic, OpenAI, Google).
- Ecossistema crescente de MCP servers públicos.

**Decisão:** ferramentas externas **devem** ser expostas via MCP quando possível; `http` e `cli` são fallback para sistemas sem MCP server disponível.

---

## Ciclo de vida de uma ferramenta externa

```
[1. Descoberta]    Arquiteto identifica necessidade durante tarefa
[2. Solicitação]   tool.request escreve em permissions/pending.md
[3. Espera]        Arquiteto suspende o caminho que depende da ferramenta
[4. Aprovação H]   Proprietário revisa; atualiza permissions/granted.md
[5. Configuração]  Humano OU Arquiteto registra conexão (secrets em env)
[6. Verificação]   Arquiteto executa tool.verify(connection_id)
[7. Ativação]      Se verify OK: status=verified
[8. Uso]           Funcionários referenciam tool em tools.yaml
[9. Auditoria]     Cada chamada logada em memória
[10. Renovação H]  Default 90 dias, requer re-confirmação
[11. Revogação H]  Humano marca status=denied; interceptor bloqueia
```

---

## Ciclo de vida de ferramenta própria (`architect_developed`)

```
[Necessidade]     Identificada em Modo EE
[Pesquisa]        develop.research(need) → candidates[]
[Decisão H]       Proprietário aprova desenvolvimento
[Design]          develop.design(spec) → tool_id
[Implementação]   develop.implement(tool_id) → commit_sha
[Teste]           develop.test(tool_id) executa T0 + T1 + T2
[Registro]        develop.register(tool_id) → catalog_entry
[Disponibilização] Funcionários referenciam em tools.yaml
[Manutenção]     Modo EE detecta bugs, novas versões, etc.
[Deprecation H]   Proprietário aprova remoção
```

---

## Permissões granulares

Cada ferramenta no catálogo declara `permissions[]` (ex.: `read`, `write`, `network`, `subprocess`). Cada funcionário declara em `tools.yaml` quais ferramentas usa e com qual escopo:

```yaml
tools:
  - id: mcp.github
    permissions_used: [read]   # subset do que a ferramenta permite
    scope:
      repositories: ["org/repo-x"]
      operations: ["list_issues", "read_file"]
```

O runtime **verifica** antes de cada chamada que:
- `permissions_used ⊆ permissions` da ferramenta (declarado no catálogo).
- `scope` está autorizado (declarado em `permissions/granted.md`).
- Rate limit respeitado.
- Host não está em denylist.

---

## Cofre de credenciais

- Secrets **nunca** em `tools.yaml`, `agent.yaml` ou memória textual.
- Referência simbólica: `secret_ref: env:GITHUB_TOKEN`.
- O runtime lê `process.env.GITHUB_TOKEN` no momento da chamada.
- Para MCPs que exigem OAuth, o token vive no MCP server (não no workspace).
- OpenBot tem password manager integration nativa (1Password, Bitwarden) e GitHub connector com device flow.
- `request_api_key` é tool nativa do OpenBot para solicitar API key ao proprietário.

---

## Acesso progressivo (P3)

O Arquiteto **não** solicita acessos em lote antecipadamente. Cada solicitação é justificada por tarefa concreta:

- O Arquiteto **nunca** solicita credencial — solicita autorização de uso.
- O Arquiteto **nunca** pede "todos os acessos possíveis" — cada pedido é específico.
- Acesso negado não bloqueia outras tarefas — apenas a tarefa dependente fica suspensa.
- Renovações não são automáticas.

---

## Denylist de sistemas

O interceptor mantém uma denylist de hosts que **nunca** podem ser acessados, mesmo com autorização:

| Host | Motivo |
|---|---|
| `appsite-erp-b2-a-engenharia.vercel.app` | ERP B2A em produção — acesso apenas via fluxo especial de §5.9 do AGENTS.md |
| IPs conhecidos do B2A | Mesmo motivo |
| Hosts sob `supabase.co` cujo project refira-se ao B2A | Banco de dados do ERP B2A |
| `n8nb2aengenharia.site` | Instância n8n do B2A — acesso apenas via fluxo especial |

Tentativa de resolver qualquer host da denylist gera:
1. Bloqueio imediato pelo interceptor.
2. Log `kind=security_violation` em `memory/incidents.md`.
3. Suspensão do funcionário que tentou.
4. Notificação ao proprietário.

---

## Dependências

- Este documento depende de: `openbot.md`, `principios-seguranca.md`, `estrategia-evolucao.md`.
- Skills relacionadas: `tool-integration` (procedimento detalhado), `skill-engineering` (para desenvolver ferramentas próprias).
