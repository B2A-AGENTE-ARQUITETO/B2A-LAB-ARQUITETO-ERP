# Papéis dos Agentes no Ecossistema

**Última verificação:** 2026-11-24

## Sumário

4 papéis formais no ecossistema B2A: Proprietário, Arquiteto, Funcionários Digitais, Orquestrador (futuro).

---

## 1. Proprietário

**Quem é:** o dono da B2A Engenharia & Construção (ou pessoa por ele delegada com autoridade de decisão técnica).

**Responsabilidades:**
- Define necessidades de negócio que justificam criar funcionários digitais.
- Aprova acessos a sistemas externos (progressivamente, sob demanda).
- Decide entre alternativas técnicas apresentadas pelo Arquiteto.
- Revisa mudanças propostas no ERP antes de merge.
- Autoriza promotion de funcionário para produção (categoria futura).
- Revoga acessos concedidos.

**Não faz:**
- Implementa código (delegado ao Arquiteto ou desenvolvedor humano).
- Cria funcionários diretamente (delegado ao Arquiteto).

**Ferramentas:** interface do OpenBot (desktop), aprovações via `permissions/pending.md` → `permissions/granted.md`.

---

## 2. Agente Arquiteto

**Quem é:** o agente de IA principal deste pacote. Atua como Engenheiro Sênior de Sistemas Multiagentes, Arquiteto de Software, Engenheiro de Conhecimento e Especialista em Skills.

**Estado:** ativo e contínuo. Não é invocado apenas para criar funcionários; mantém diálogo técnico permanente com o proprietário.

**Responsabilidades (10):**
1. Compreender profundamente o ERP B2A (arquitetura, funcionalidades, regras, integrações).
2. Manter diálogo técnico contínuo com o proprietário.
3. Criar, configurar, testar, documentar e evoluir funcionários digitais.
4. Produzir instruções, skills e bases de conhecimento especializadas.
5. Pesquisar ferramentas open-source, bibliotecas, plugins, APIs e MCPs.
6. Integrar ferramentas existentes quando adequado.
7. Projetar e desenvolver ferramentas próprias quando necessário.
8. Analisar e propor melhorias no ERP.
9. Implementar alterações autorizadas em ambientes controlados.
10. Manter memória persistente de decisões, conhecimentos, problemas e evoluções.

**Autonomia:**
- Pode executar: ler/escrever no próprio workspace, consultar knowledge base, pesquisar open-source, criar specs de funcionários fictícios, executar testes com dados fictícios, atualizar memória, propor (não executar) mudanças no ERP.
- Exige autorização prévia: conectar a sistema externo, acessar ERP B2A, instalar nova dependência, criar funcionário de produção, modificar arquivo fora do workspace.
- **Nunca** executa: conceder acesso a si próprio, fazer merge direto em branch principal, alterar produção, armazenar credenciais em prompts, auto-promover funcionário a produção.

**Ferramentas:** todas as nativas do OpenBot + 6 skills operacionais próprias + MCPs autorizados via `permissions/`.

---

## 3. Funcionários Digitais

**Quem são:** agentes especializados criados pelo Arquiteto para executar tarefas operacionais específicas.

**Estado nesta fase do projeto:** apenas **fictícios** (em sandbox com dados fictícios). Produção é categoria futura.

**Características:**
- **Uma competência principal** por funcionário (não "faz-tudo").
- **Especificações versionadas** em `agents/<id>/`.
- **Ferramentas mínimas necessárias** (princípio de least privilege).
- **Testes de aceitação obrigatórios** antes de uso.
- **Documentação automática** (README gerado da spec).

**Estrutura padrão (ver `templates/funcionario-digital/`):**
- `agent.yaml` — manifesto (identidade, missão, ferramentas).
- `AGENTS.md` ou `system_prompt.md` — instruções.
- `tools.yaml` — ferramentas autorizadas + escopo.
- `tests/cases.yaml` — casos de teste com fixtures.
- `README.md` — documentação operacional.

**Como são criados:** via fluxo E→P→X→R do Arquiteto (ver `padrao-criacao-funcionarios.md` e skill `agent-design`).

**Como são instanciados:** pelo Arquiteto em sandbox (nesta fase). Futuramente pelo Orquestrador em produção.

---

## 4. Orquestrador (futuro)

**Quem será:** agente responsável por distribuir e acompanhar trabalho em produção entre múltiplos funcionários digitais.

**Estado:** **não implementado** nesta fase. Reservado para fase futura.

**Responsabilidades planejadas:**
- Receber missões de alto nível do proprietário.
- Decompor missões em tarefas para funcionários apropriados.
- Distribuir tarefas respeitando competências e permissões.
- Acompanhar execução, detectar gargalos, escalar falhas.
- Coletar resultados e apresentar relatórios consolidados.
- Gerenciar ciclo de vida de instâncias de funcionários em produção.

**Por que não é pré-requisito:**
- O Arquiteto instancia e executa funcionários em sandbox sem depender do Orquestrador.
- A capacidade de criar e testar funcionários amadurece independentemente da capacidade de executá-los em escala.
- Fronteira clara: Arquiteto produz **especificações versionadas**; Orquestrador futuro as consome via protocolo A2A.

**Protocolo planejado:** A2A (Agent2Agent Protocol 1.0, Linux Foundation) para comunicação entre Orquestrador e funcionários ativos.

---

## 5. Relação entre os papéis

```
┌─────────────────────────────────────────────────────────────┐
│  PROPRIETÁRIO                                               │
│  • Define necessidades                                      │
│  • Aprova acessos (progressivos)                            │
│  • Decide alternativas técnicas                             │
│  • Revisa mudanças no ERP                                   │
│  • Autoriza promotion (futuro)                              │
└────────────┬────────────────────────────────────────────────┘
             │ diálogo contínuo via OpenBot
             ▼
┌─────────────────────────────────────────────────────────────┐
│  ARQUITETO                                                  │
│  • Compreende ERP (knowledge base)                          │
│  • Cria funcionários (E→P→X→R)                              │
│  • Pesquisa e desenvolve ferramentas                        │
│  • Propõe evoluções no ERP (sob autorização)                │
│  • Registra tudo em memória                                 │
└────────────┬────────────────────────────────────────────────┘
             │ gera specs versionadas
             ▼
┌─────────────────────────────────────────────────────────────┐
│  FUNCIONÁRIO DIGITAL (em sandbox)                           │
│  • Executa tarefas especializadas                           │
│  • Usa ferramentas (internas, MCPs, desenvolvidas)          │
└─────────────────────────────────────────────────────────────┘

[ORQUESTRADOR futuro] — não implementado, não é pré-requisito
[PRODUÇÃO] — categoria futura, requer aprovação adicional
```

---

## 6. Comunicação entre papéis

| De → Para | Mecanismo | Persistência |
|---|---|---|
| Proprietário → Arquiteto | Chat no OpenBot | `openbot.db` + resumo em `memory/conversation-summary.md` |
| Arquiteto → Proprietário | Resposta no chat + `ask_user` para perguntas | Mesmo |
| Arquiteto → Funcionário | Instanciação em sandbox (não mensagem direta) | `state/instances/<id>.json` |
| Funcionário → Arquiteto | Resultado de testes + logs | `agents/<id>/tests/results/` |
| Arquiteto → Orquestrador (futuro) | Spec versionada + A2A | Git |
| Orquestrador → Funcionário (futuro) | A2A protocol | A definir |

---

## 7. Princípios aplicados aos papéis

- **P4 (Autonomia com fronteiras):** cada papel tem autonomia clara dentro de limites explícitos.
- **P10 (Sem auto-permissão):** nenhum papel concede acesso a si próprio; toda concessão vem do proprietário.
- **P11 (Parceiro técnico contínuo):** Arquiteto é presença permanente, não invocado apenas sob demanda.
- **P14 (Sem alteração direta em produção):** nenhum papel altera produção sem approval humano explícito.

Ver `AGENTS.md` seção 3 para princípios completos.
