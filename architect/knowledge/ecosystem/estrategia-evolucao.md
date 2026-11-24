# Estratégia de Evolução e Manutenção

**Última verificação:** 2026-11-24

## Sumário

Como o ecossistema evolui: modo Engenharia & Evolução (EE), atualização incremental de conhecimento, manutenção de funcionários e ferramentas, gestão de versões.

---

## Modo Engenharia & Evolução (EE)

O Arquiteto opera continuamente em modo EE, paralelo às missões pontuais. **Não é deflagrado por uma missão específica** — é estado permanente.

### Gatilhos do Modo EE

- Nova thread do proprietário sem missão específica.
- Falha recorrente em testes T0/T2 de funcionário existente.
- Divergência detectada entre knowledge base e fonte original (após consulta autorizada).
- Disponibilidade de nova versão de ferramenta open-source em uso.
- Identificação de lacuna de conhecimento durante tarefa.

### Fluxo EE

```
[EE1] Monitorar conversas (C11) e memória operacional (M2)
      para identificar necessidades, divergências ou oportunidades
      ↓
[EE2] Consultar Knowledge Base (M5) para identificar capacidades
      existentes que atendam à necessidade
      ↓
[EE3] Decisão de caminho:
      (a) Capacidade existe → propor reuso (encerrar EE aqui)
      (b) Capacidade não existe → pesquisar open-source
          (skill tool-integration)
      (c) Nada adequado → propor desenvolvimento próprio
          (skill skill-engineering)
      (d) Mudança no ERP necessária → abrir proposta (ver AGENTS.md §11)
      ↓
[EE4] Apresentar alternativas ao proprietário via conversa
      (kind=alternative) com trade-offs explícitos
      ↓
[EE4H] Proprietário decide via conversa (kind=decision)
      ↓
[EE5] Executar decisão dentro dos limites autorizados:
      - Reuso: configurar ferramenta existente em tools.yaml
      - Open-source: solicitar integração via ciclo de §5.2
      - Desenvolvimento próprio: skill-engineering
      - Mudança no ERP: requer approval separada (AGENTS.md §11)
      ↓
[EE6] Atualizar Knowledge Base com novo aprendizado
      (knowledge.propose_update com diff justificado)
      ↓
[EE7] Registrar evolução em M2 (kind=decision + linked_artifacts)
```

---

## Atualização incremental de conhecimento

### Princípio fundamental (P12)

A Knowledge Base contém **referências estruturadas** às fontes, **nunca** réplicas integrais do código.

| Tipo de conhecimento | O que é armazenado em M5 | O que NÃO é armazenado |
|---|---|---|
| Arquitetura do ERP | Sumário de componentes + diagrama textual + `source_ref` a `ARCHITECTURE.md` | Conteúdo integral do `ARCHITECTURE.md` |
| Módulos | Lista de módulos + 1 parágrafo cada + `source_ref` à rota | Código das páginas |
| Processos de negócio | Passos principais + regras críticas + `source_ref` aos arquivos | Implementação completa |
| Schema de banco | Lista de tabelas + campos-chave + `source_ref` às migrations | DDL integral |
| APIs | Endpoints + schemas de input/output + `source_ref` aos route handlers | Código dos handlers |
| Regras de negócio | Descrição estruturada + `source_ref` ao código que as implementa | Código |
| Decisões técnicas | Decisão + rationale + `source_ref` à conversa onde foi tomada | Transcrição integral |

### Quando detalhe é necessário

O Arquiteto solicita `knowledge.consult(entry_id)`:
- **Para fontes internas ao workspace** (specs, manifests): retorna o conteúdo do arquivo referenciado (auto).
- **Para fontes externas** (ERP B2A, repositórios privados): requer autorização prévia (P3). Após autorização, consulta a fonte original sob a janela de acesso concedida e retorna apenas o trecho relevante ao contexto da tarefa.

### Atualização

- Entradas têm `last_verified_at` e `confidence`.
- Entradas marcadas `outdated` (após mudança conhecida) são re-consultadas sob autorização.
- Histórico de versões preservado em Git (commits de Knowledge Base são atômicos e justificados).

### Registro de divergências

Quando o Arquiteto detecta divergência entre Knowledge Base e fonte original:

1. **Identifica** a divergência específica (citação de ambos).
2. **Abre proposta de atualização** em `memory/divergences.md`.
3. **Propõe** atualização do documento defasado.
4. **Após autorização**, atualiza e commita.

---

## Manutenção de funcionários digitais

### Tipos de manutenção

| Tipo | Gatilho | Ação |
|---|---|---|
| **Corretiva** | Bug reportado ou detectado em T2 | Diagnosticar, propor diff, re-testar |
| **Adaptativa** | Mudança em fonte externa (ex.: API mudou) | Atualizar tools.yaml + AGENTS.md, re-testar |
| **Perfectiva** | Oportunidade de melhoria identificada em EE | Propor diff, re-testar |
| **Preventiva** | Nova versão de dependência ou ferramenta | Atualizar, re-testar, validar compatibilidade |

### Versionamento de funcionários

Funcionários seguem **SemVer**:
- **MAJOR:** mudança incompatível (quebra testes anteriores).
- **MINOR:** nova capacidade, retrocompatível.
- **PATCH:** correção de bug, retrocompatível.

Cada versão tem:
- Tag Git no branch do Arquiteto.
- Entrada em `memory/evolution-log.md`.
- Resultados de T2 da versão anterior preservados para regressão.

### Deprecation de funcionário

Quando um funcionário não é mais necessário:

1. **Proposta de deprecation** em `memory/decisions.md`.
2. **Aprovação do proprietário**.
3. **Marcação** em `memory/agents-registry.md`: `status: deprecated`.
4. **Remoção de tools.yaml** de outros funcionários que referenciem.
5. **Arquivamento** da pasta `agents/<id>/` para `agents/_archived/<id>/`.
6. **Não exclusão** — histórico preservado.

---

## Manutenção de ferramentas

### Ferramentas MCP/externas

- Monitorar versões do MCP server.
- Testar compatibilidade antes de atualizar.
- Renovar autorizações a cada 90 dias (read) ou 30 dias (write).

### Ferramentas próprias (`architect_developed`)

- Monitorar dependências (`npm audit`, `pip-audit`, etc.).
- Atualizar patches de segurança.
- Re-testar T0 + T1 + T2 a cada mudança.
- Considerar deprecation se open-source melhor surgir (P5).

---

## Gestão de versões do ecossistema

### Versionamento do pacote do Arquiteto

O pacote `AGENTE_ARQUITETO_OPENBOT/` segue **SemVer**:

- **MAJOR:** mudança em princípios inegociáveis, missão ou limites.
- **MINOR:** nova skill, nova entrada de knowledge, novo template.
- **PATCH:** correção de referência, esclarecimento de procedimento.

Mudanças que afetem princípios, missão ou limites exigem:
1. Proposta de diff em branch isolada.
2. Aprovação explícita do proprietário.
3. Registro em `memory/evolution-log.md`.
4. Bump de versão no frontmatter do `AGENTS.md`.

Mudanças menores (esclarecimento de procedimento, correção de referência) podem ser feitas autonomamente, com registro no log.

### Compatibilidade com OpenBot

- O pacote é compatível com OpenBot v0.72.0 (verificado em 2026-11-24).
- Mudanças no OpenBot que afetem o pacote são detectadas via Modo EE.
- Atualizações do OpenBot são aplicadas pelo proprietário (não pelo Arquiteto).

---

## Gestão de memória

### Compactação periódica

- M1 (curta) expira ao fim da sessão.
- M2 (operacional) é append-only, mas com compactação periódica:
  - Após 1000 entradas em um escopo, o Arquiteto gera sumário `kind=observation` consolidando padrões.
  - Entradas antigas não são removidas, mas o sumário passa a ser a referência primária.
- M3 (declarativa) só muda via diff aprovado.
- M4 (estado de execução) é sobrescrevível, sem necessidade de compactação.

### Backup

- Todo o workspace do Arquiteto é versionado em Git.
- `state/` (M4) é incluído em `.gitignore` — não é versionado, mas pode ser exportado sob demanda.
- O OpenBot mantém `openbot.db` como source of truth para conversas.

---

## Quando o Arquiteto é reinicializado

Após reinicialização (nova sessão, crash, etc.), o Arquiteto reconstrói estado lendo:

1. `AGENTS.md` — identidade, missão, princípios.
2. `memory/conversation-summary.md` — resumo da última conversa.
3. `memory/pending-tasks.md` — tarefas suspensas.
4. `memory/decisions.md` — decisões recentes.
5. `memory/agents-registry.md` — funcionários existentes.
6. `permissions/granted.md` — acessos atualmente ativos.
7. `history_search` (OpenBot nativo) — recuperar contexto de conversas anteriores.

**Regra:** o Arquiteto deve conseguir retomar qualquer trabalho interrompido sem pedir ao proprietário para repetir contexto.

---

## Dependências

- Este documento depende de: `papeis-agentes.md`, `padrao-criacao-funcionarios.md`, `principios-seguranca.md`.
- Skills relacionadas: `skill-engineering` (manutenção de skills), `agent-evaluation` (manutenção de testes).
