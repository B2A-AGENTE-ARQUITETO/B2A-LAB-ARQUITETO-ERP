---
name: erp-knowledge
description: Consultar, compreender, analisar e atualizar o conhecimento sobre o ERP B2A. Use quando precisar responder perguntas sobre o ERP, identificar dependências, compreender funcionamento de módulos, ou propor atualizações na base de conhecimento após verificar nova informação.
---

# Skill: ERP Knowledge

## Objetivo

Operar a base de conhecimento sobre o ERP B2A Engenharia: consultar com precisão, distinguir fatos verificados de inferências, identificar lacunas e propor atualizações quando nova informação verificada é obtida.

## Situações de uso

Use esta skill quando:

- O proprietário pergunta sobre o ERP (módulos, processos, regras, APIs, dados).
- Outra skill precisa de contexto sobre o ERP antes de agir.
- Você precisa identificar dependências entre módulos.
- Você detecta divergência entre base de conhecimento e fonte original.
- Você obteve nova informação verificada (após consulta autorizada) e precisa atualizar a base.

**Não use** esta skill para:
- Acessar o ERP diretamente (use `software-engineering` mediante autorização).
- Criar funcionários (use `agent-design`).
- Pesquisar ferramentas externas (use `tool-integration`).

## Entradas necessárias

- Pergunta ou necessidade de informação.
- Domínio relevante (visão-geral, módulos, processos, regras, dados, APIs, fluxos, estado, limitações, histórico).
- Se atualizando: nova informação verificada + fonte.

## Procedimento operacional

### Passo 1: Classificar a pergunta

Identifique a qual dos 10 domínios a pergunta pertence:

| Domínio | Documento |
|---|---|
| Visão geral, stack, arquitetura | `knowledge/erp/01-visao-geral.md` |
| Módulos e funcionalidades | `knowledge/erp/02-modulos.md` |
| Processos operacionais | `knowledge/erp/03-processos.md` |
| Regras de negócio | `knowledge/erp/04-regras-negocio.md` |
| Modelo de dados | `knowledge/erp/05-modelo-dados.md` |
| APIs e integrações | `knowledge/erp/06-apis-integracoes.md` |
| Fluxos entre módulos | `knowledge/erp/07-fluxos-entre-modulos.md` |
| Estado de implementação | `knowledge/erp/08-estado-implementacao.md` |
| Limitações e problemas | `knowledge/erp/09-limitacoes-problemas.md` |
| Histórico de decisões | `knowledge/erp/10-historico-decisoes.md` |

### Passo 2: Consultar o documento

1. Abra o documento relevante.
2. Use o sumário interno do documento para localizar a seção.
3. Leia apenas o necessário para a pergunta.
4. Verifique a etiqueta de verificação:
   - `[VERIFICADO]` — confirmado no código, pode afirmar com confiança.
   - `[INFERIDO]` — inferência razoável, declare como hipótese.
   - `[DESCONHECIDO]` — lacuna, declare que não sabe e proponha como preencher.

### Passo 3: Compor a resposta

Estruture a resposta:

```
**[VERIFICADO | INFERIDO | DESCONHECIDO]**

[Resposta objetiva]

**Fonte:** [caminho do documento + seção]
```

Se a informação é `[DESCONHECIDO]`:
```
**[DESCONHECIDO]**

Esta informação não está na base de conhecimento atual.

**Para verificá-la, preciso de:** [acesso/autorização a X]
**Alternativa sem acesso:** [o que posso fazer]
```

Registre a lacuna em `memory/knowledge-gaps.md` se for recorrente.

### Passo 4: Verificar dependências

Após responder, identifique se a resposta tem implicações para outros domínios:

- A resposta menciona uma regra de negócio? Cruze com `04-regras-negocio.md`.
- A resposta menciona uma tabela? Cruce com `05-modelo-dados.md`.
- A resposta menciona um fluxo? Cruze com `07-fluxos-entre-modulos.md`.

Se houver divergência, registre em `memory/divergences.md`.

### Passo 5 (se atualizando): Propor atualização

Quando você obtém nova informação verificada:

1. **Valide a fonte:** a informação vem de consulta autorizada ao repositório ou do proprietário?
2. **Localize o documento** relevante.
3. **Identifique a seção** a atualizar.
4. **Proponha diff** em `memory/evolution-log.md`.
5. **Aguarde aprovação** do proprietário se a mudança afeta decisões anteriores.
6. **Aplique a atualização** mantendo a etiqueta de verificação.
7. **Atualize `last_verified_at`** no topo do documento.
8. **Commit** em branch isolada.

## Ferramentas necessárias

- `Read` (ler documentos da knowledge base).
- `Grep` (buscar termos específicos).
- `Glob` (localizar arquivos).
- `Edit` (atualizar documentos após aprovação).
- OpenBot nativas: `remember`, `search_memories`, `ask_user`.

## Limites de autorização

- **Pode executar autonomamente:**
  - Ler qualquer documento em `knowledge/erp/`.
  - Registrar lacunas em `memory/knowledge-gaps.md`.
  - Registrar divergências em `memory/divergences.md`.
  - Propor atualizações em `memory/evolution-log.md`.

- **Exige aprovação do proprietário:**
  - Aplicar atualização em `knowledge/erp/` que contradiz decisão anterior.
  - Consultar repositório GitHub do ERP (ver `software-engineering` skill).
  - Acessar o ERP em produção.

- **Proibido:**
  - Inventar informação para preencher lacuna.
  - Apresentar `[INFERIDO]` como `[VERIFICADO]`.
  - Acessar ERP sem autorização específica.

## Critérios de qualidade

- Toda afirmação tem etiqueta de verificação.
- Toda resposta cita fonte (caminho do documento).
- Lacunas são declaradas explicitamente, não mascaradas.
- Divergências são registradas antes de atualizadas.
- Atualizações preservam histórico (Git).

## Critérios de conclusão

A skill é concluída quando:

- Resposta entregue ao proprietário com fonte e etiqueta de verificação.
- Lacunas (se houver) registradas em `memory/knowledge-gaps.md`.
- Divergências (se houver) registradas em `memory/divergences.md`.
- Atualizações (se houver) commitadas em branch isolada com justificativa.

## Tratamento de erros

| Erro | Ação |
|---|---|
| Documento não encontrado | Informar lacuna, registrar em `memory/knowledge-gaps.md` |
| Informação conflitante entre documentos | Registrar em `memory/divergences.md`, declarar na resposta |
| Informação marcadamente `[DESCONHECIDO]` | Não inventar; propor como preencher |
| Fonte externa necessária | Informar proprietário, solicitar autorização via `permissions/pending.md` |

## Exemplos práticos

### Exemplo 1: Pergunta factual

**Proprietário:** "Como o ERP gera IDs de pedidos de compra?"

**Procedimento:**
1. Classificar: regra de negócio → `04-regras-negocio.md` seção RN-04.
2. Consultar: abrir documento, localizar RN-04.
3. Responder:

```
**[VERIFICADO]**

O ERP gera IDs de pedidos de compra no formato `PED-NNNN` (4 dígitos zero-padded)
via RPC `get_next_sequence(text, integer)` com prefixo "PED".

A RPC é `SECURITY DEFINER` (migration `20260314_fix_sequence_rpc.sql`) e,
desde 2026-08-19, apenas `authenticated` e `service_role` podem executá-la
(migration `20260819_revoke_anon_sequence_rpc.sql`).

**Fonte:** `knowledge/erp/04-regras-negocio.md` seção RN-04.
```

### Exemplo 2: Lacuna identificada

**Proprietário:** "Quais variáveis de ambiente estão configuradas na Vercel?"

**Procedimento:**
1. Classificar: visão geral → `01-visao-geral.md`.
2. Consultar: documento menciona que variáveis **referenciadas** no código são conhecidas, mas estado real na Vercel é `[DESCONHECIDO]`.
3. Responder:

```
**[DESCONHECIDO]**

O código referencia 6 variáveis de ambiente, mas o estado real de configuração
na Vercel não está versionado no Git e não posso verificar sem acesso ao painel.

**Para verificá-la, preciso de:** acesso read-only ao painel Vercel.
**Alternativa sem acesso:** você pode me informar quais variáveis estão configuradas.

**Fonte:** `knowledge/erp/06-apis-integracoes.md` seção "Variáveis de ambiente referenciadas".
```

Registrar lacuna em `memory/knowledge-gaps.md` se for a primeira vez.

## Referências a documentos de conhecimento

- `knowledge/erp/INDEX.md` — índice navegável.
- `knowledge/erp/01-visao-geral.md` até `10-historico-decisoes.md` — 10 domínios.
- `memory/knowledge-gaps.md` — lacunas identificadas.
- `memory/divergences.md` — divergências registradas.
- `memory/evolution-log.md` — histórico de atualizações.
