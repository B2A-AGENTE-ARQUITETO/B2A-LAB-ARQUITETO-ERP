# Template — Funcionário Digital

**Versão:** 1.0.0 · **Como usar:** copie esta pasta para `agents/<novo-agent-id>/` e substitua os placeholders.

## Estrutura

```
templates/funcionario-digital/
├── README.md              # Este arquivo (guia de uso do template)
├── agent.yaml             # Manifesto principal (template)
├── AGENTS.md              # Instruções principais (template)
├── tools.yaml             # Ferramentas autorizadas (template)
├── tests/
│   ├── cases.yaml         # Casos de teste (template)
│   └── fixtures/
│       └── example.json   # Fixture fictício (template)
└── knowledge/
    └── INDEX.md           # Índice de conhecimento específico (template)
```

## Como usar

### Passo 1: Copiar template

```bash
cp -r templates/funcionario-digital/ agents/<novo-agent-id>/
cd agents/<novo-agent-id>/
```

### Passo 2: Preencher agent.yaml

Substituir todos os `<PLACEHOLDERS>`:

- `id`: identificador único (kebab-case). Para fictícios: prefixo `ficticio-`.
- `display_name`: nome exibido no OpenBot.
- `role`: cargo/role curto.
- `version`: iniciar em `1.0.0`.
- `created_at`: data ISO atual.
- `parent_commit`: SHA do commit do Arquiteto que criou.
- `mission`: missão em 1 frase.
- `responsibilities`: 3–7 responsabilidades concretas.
- `limits`: limites explícitos.
- `provider.primary`: codex | claude | gemini | ...
- `provider.fallback`: lista de fallbacks.
- `provider.model`: modelo específico.
- `permissions_profile`: workspace_only (padrão) | ask_first.

### Passo 3: Escrever AGENTS.md

**Regras obrigatórias:**
- Procedimentos concretos, não aspiracionais.
- "Seja especialista" é proibito.
- Máximo 2000 palavras.
- Estrutura: Identidade → Missão → Princípios → Procedimentos → Limites.

### Passo 4: Preencher tools.yaml

- Apenas ferramentas com `status: verified` em `permissions/tools-catalog.yaml`.
- `permissions_used` deve ser subset do que a ferramenta permite.
- `scope` deve ser subset do `approved_scope` em `permissions/granted.md`.
- Se ferramenta necessária não está aprovada: abrir solicitação em `permissions/pending.md`.

### Passo 5: Criar casos de teste

- Mínimo 3 casos em `tests/cases.yaml`.
- Cobrir: situação normal, situação de borda, situação de erro.
- Fixtures em `tests/fixtures/` são **fictícios**.

### Passo 6: Conhecimento específico (opcional)

Se o funcionário precisa de conhecimento além do que está em `knowledge/erp/`:
- Criar entradas em `knowledge/` do funcionário.
- Manter mesmo padrão de etiquetas: `[VERIFICADO]`, `[INFERIDO]`, `[DESCONHECIDO]`.

### Passo 7: Validar e commitar

- Verificar que todos os placeholders foram substituídos (buscar `<PLACEHOLDER`).
- Verificar que `tests/cases.yaml` tem pelo menos 3 casos.
- Verificar que `tools.yaml` não tem ferramentas não aprovadas.
- Commit em branch isolada: `agents/<novo-agent-id>/v1`.

### Passo 8: Provisionar e testar

Seguir Fase P (provisionamento) e Fase X (execução em sandbox) da skill `agent-design`.

## Regras de qualidade

| Regra | Como verificar |
|---|---|
| Sem placeholders não substituídos | `grep -r "<PLACEHOLDER" agents/<id>/` deve retornar vazio |
| AGENTS.md ≤ 2000 palavras | `wc -w agents/<id>/AGENTS.md` |
| Mínimo 3 casos de teste | contar entradas em `tests/cases.yaml` |
| Tools apenas aprovadas | cruzar com `permissions/tools-catalog.yaml` |
| Sem credenciais em arquivos | regex scan por padrões de secret |
| Sem "seja especialista" | `grep -i "seja especialista\|atue como especialista" agents/<id>/AGENTS.md` |

## Limites do template

Este template **não** é para:
- Funcionários de produção (requer aprovação adicional do proprietário).
- Funcionários com `permissions_profile: full_access`.
- Funcionários que acessam ERP B2A (requer fluxo especial).

## Versionamento

- Mudanças no template incrementam versão em `README.md`.
- Mudanças que quebram funcionários existentes: MAJOR.
- Novos campos opcionais: MINOR.
- Correções: PATCH.
