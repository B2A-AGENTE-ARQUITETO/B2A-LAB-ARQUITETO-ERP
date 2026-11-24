---
name: skill-engineering
description: Desenvolver, validar, documentar e atualizar skills reutilizáveis para o ecossistema de agentes. Use quando precisar criar nova skill (procedimento operacional especializado), corrigir skill existente, ou empacotar skill para distribuição.
---

# Skill: Skill Engineering

## Objetivo

Produzir skills no formato Agent Skills (SKILL.md + recursos) que podem ser carregadas por qualquer agente no OpenBot. Skills são procedimentos operacionalmente executáveis que estendem a capacidade de agentes sem exigir código runtime.

## Situações de uso

Use esta skill quando:

- O proprietário pede para criar uma nova capacidade operacional reutilizável.
- Um procedimento recorrente justifica virar skill (usado por 2+ funcionários).
- Uma skill existente precisa ser corrigida ou atualizada.
- Você precisa empacotar skill para distribuição (zip, plugin).

**Não use** esta skill para:
- Criar funcionários completos (use `agent-design`).
- Pesquisar ferramentas externas (use `tool-integration`).
- Implementar ferramenta com código runtime (use `software-engineering`).

## Entradas necessárias

- **Nome da skill:** identificador único (kebab-case).
- **Descrição:** 1–1024 chars, explica quando usar (importante para implicit invocation).
- **Objetivo:** o que a skill alcança.
- **Procedimento:** passos concretos que um agente pode seguir.

## Procedimento operacional

### Passo 1: Definir escopo

Antes de criar, valide:

1. **Já existe skill similar?** Consulte `skills/` no workspace do Arquiteto e skills nativas do OpenBot.
2. **Pode ser função nativa do OpenBot?** Verifique `knowledge/ecosystem/openbot.md` para tools nativas.
3. **É recorrente o suficiente?** Se usado apenas 1 vez, não justifica skill.

Se respondeu "sim" a 1 ou 2, não crie. Se respondeu "não" a 3, considere inline no `AGENTS.md` do funcionário.

### Passo 2: Estruturar a skill

Crie pasta `skills/<skill-name>/` com:

```
skills/<skill-name>/
├── SKILL.md              # Arquivo principal (obrigatório)
├── assets/               # Recursos opcionais
│   └── icon.png          # Ícone 256x256 (opcional, preview local)
├── references/           # Documentos de referência (opcional)
│   └── *.md
└── scripts/              # Scripts auxiliares (opcional)
    └── *.{sh,ts,py}
```

### Passo 3: Escrever SKILL.md

Use o template:

```markdown
---
name: <skill-name>                          # deve bater com nome da pasta
description: <1-1024 chars, quando usar>    # crítico para implicit invocation
---

# Skill: <Display Name>

## Objetivo
[1 parágrafo]

## Situações de uso
Use quando:
- [caso 1]
- [caso 2]

**Não use para:**
- [quando não usar]

## Entradas necessárias
- [input 1]
- [input 2]

## Procedimento operacional
### Passo 1: [ação]
[descrição concreta]

### Passo 2: [ação]
[descrição concreta]

## Ferramentas necessárias
- [tool 1]
- [tool 2]

## Limites de autorização
**Pode executar autonomamente:**
- [ações auto]

**Exige aprovação:**
- [ações com approval]

**Proibido:**
- [ações bloqueadas]

## Critérios de qualidade
- [critério 1]
- [critério 2]

## Critérios de conclusão
- [critério de aceite 1]
- [critério de aceite 2]

## Tratamento de erros
| Erro | Ação |
|---|---|
| [erro] | [ação] |

## Exemplos práticos
### Exemplo 1: [cenário]
[passo a passo]

## Referências a documentos de conhecimento
- [caminho 1]
- [caminho 2]
```

### Passo 4: Validações obrigatórias

#### 4.1: Validar frontmatter

```yaml
name: <string>            # obrigatório, kebab-case, igual ao nome da pasta
description: <string>     # obrigatório, 1-1024 chars
```

Frontmatter YAML inválido faz a skill ser ignorada pelo OpenBot.

#### 4.2: Validar description

A `description` é o que permite implicit invocation. Regras:
- **Primeira frase:** declara o caso de uso principal.
- **Trigger words:** inclua palavras que o modelo pode associar à skill.
- **Limites:** declare quando NÃO usar.

#### 4.3: Validar procedimento

Verificar:
- Passos são concretos (não aspiracionais).
- Cada passo tem ação definida.
- Não há "seja especialista" ou adjetivos sem operacionalização.
- Limites de autorização explícitos.
- Critérios de conclusão mensuráveis.

#### 4.4: Validar referências

Para cada referência citada:
- Verificar que o arquivo existe.
- Verificar que o caminho relativo está correto.
- Verificar que a referência é atual (não defasada).

### Passo 5: Testar a skill

#### 5.1: Teste funcional (T2)

Crie caso de teste em `tests/cases.yaml` do próprio Arquiteto:
```yaml
- id: skill-<name>-001
  description: Skill deve ser ativada quando [cenário]
  input:
    type: message
    content: "mensagem que deve trigger a skill"
  expected:
    skill_activated: <skill-name>
    procedure_followed: true
  criteria:
    must_execute_steps: [passo 1, passo 2, ...]
    max_tokens: 3000
```

#### 5.2: Teste de não-ativação (T2 negativo)

```yaml
- id: skill-<name>-neg-001
  description: Skill NÃO deve ser ativada quando [cenário diferente]
  input:
    type: message
    content: "mensagem que NÃO deve trigger a skill"
  expected:
    skill_activated: null
```

#### 5.3: Teste de segurança (T0)

Se a skill envolve ferramentas externas:
- T0.1: tool não autorizada bloqueada.
- T0.4: sem credenciais em output.
- T0.7: auditoria completa.

### Passo 6: Documentar

Gere `references/README.md` (se aplicável) com:
- Resumo da skill.
- Quando usar (com exemplos).
- Limitações conhecidas.
- Histórico de versões.

### Passo 7: Registrar

Adicione entrada em `memory/agents-registry.md` do Arquiteto.

### Passo 8: Distribuir

#### Opção A: Uso local apenas
Skill fica em `skills/<skill-name>/` no workspace do Arquiteto. Disponível para o próprio Arquiteto.

#### Opção B: Disponibilizar para outros agentes
Use comandos nativos do OpenBot:
- `create_skill` — publica pasta do workspace como skill imutável.
- `revise_skill` — revisa skill existente (cria nova versão).
- `install_local_skill` — instala skill em outro agente.

#### Opção C: Empacotar como plugin
Se a skill inclui MCP server junto: empacotar como plugin via `openbot.run/plugins/`.

### Passo 9: Manter

- Monitorar uso (via `memory/evolution-log.md`).
- Atualizar quando procedimento muda.
- Deprecar quando não mais necessária (marcar status=deprecated em `memory/agents-registry.md`).

## Ferramentas necessárias

- `Read`, `Write`, `Edit` (manipular SKILL.md).
- `Glob`, `Grep` (validar referências).
- OpenBot nativas: `list_local_skills`, `read_local_skill`, `revise_skill`, `install_local_skill`, `create_skill`.

## Limites de autorização

- **Pode executar autonomamente:**
  - Criar skill no próprio workspace.
  - Testar skill com dados fictícios.
  - Atualizar skill em branch isolada.

- **Exige aprovação do proprietário:**
  - Publicar skill para outros agentes.
  - Empacotar skill como plugin.
  - Distribuir skill externamente.

- **Proibido:**
  - Criar skill que bypassa interceptor de segurança.
  - Skill com credenciais hardcoded.
  - Skill que acessa ERP B2A sem fluxo de autorização.

## Critérios de qualidade

- SKILL.md segue formato Agent Skills specification.
- Frontmatter válido (name + description).
- Description tem ≥ 1 trigger word clara.
- Procedimento é executável (não aspiracional).
- Limites de autorização explícitos.
- Mínimo 1 caso T2 + 1 caso T2 negativo.
- Referências validadas (arquivos existem).
- Sem credenciais ou secrets.

## Critérios de conclusão

A skill é concluída quando:

- SKILL.md criado e validado.
- Passou T2 (ativação correta).
- Passou T2 negativo (não ativação quando não deve).
- Passou T0 (segurança, se aplicável).
- Registrada em `memory/agents-registry.md`.
- Documentação em `references/README.md` (se aplicável).

## Tratamento de erros

| Erro | Ação |
|---|---|
| Frontmatter YAML inválido | Corrigir sintaxe, re-testar |
| Description vago | Reescrever com trigger words, re-testar |
| Skill não ativa em teste T2 | Refinar description, re-testar |
| Skill ativa em T2 negativo | Refinar description com limites, re-testar |
| Referência quebrada | Corrigir caminho ou criar arquivo |
| T0 falha | Identificar violação, corrigir, re-testar |

## Exemplos práticos

### Exemplo 1: Criar skill de validação de CNPJ

**Cenário:** Múltiplos funcionários precisam validar CNPJ. Justifica skill reutilizável.

**Fluxo:**
1. Passo 1: Verificar `skills/` — não existe skill similar. Justifica criação.
2. Passo 2: Criar `skills/validador-cnpj/`.
3. Passo 3: Escrever SKILL.md:
   ```yaml
   ---
   name: validador-cnpj
   description: Validar formato e dígitos verificadores de CNPJ. Use quando precisar verificar se um CNPJ é válido antes de usar em cadastro, conciliação ou operação que exija CNPJ correto.
   ---
   ```
4. Passo 4: Validar frontmatter. OK.
5. Passo 5: Testar T2 com CNPJ válido e inválido.
6. Passo 6: Documentar em `references/README.md`.
7. Passo 7: Registrar em `memory/agents-registry.md`.
8. Passo 8: Disponibilizar via `install_local_skill` para outros agentes.

## Referências a documentos de conhecimento

- `knowledge/ecosystem/openbot.md` — formato Agent Skills, localizações.
- `knowledge/ecosystem/estrategia-evolucao.md` — manutenção de skills.
- `memory/agents-registry.md` — skills existentes.
