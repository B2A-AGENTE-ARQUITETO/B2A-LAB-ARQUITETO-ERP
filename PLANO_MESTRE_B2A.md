# PLANO MESTRE B2A

**Versão:** 1.0.0  
**Data de consolidação:** 2026-11-24  
**Status:** laboratório operacional; primeiro funcionário digital em preparação  
**Documento estratégico de referência:** `VISAO_E_DIRETRIZES_B2A.md`

## 1. Regra de governança

`VISAO_E_DIRETRIZES_B2A.md` contém a visão estratégica aprovada pelo proprietário e não deve ser alterado unilateralmente. Este documento contém o planejamento operacional, o estado verificável, as pendências e a próxima entrega. Decisões novas devem preservar o histórico e registrar data, evidência e impacto.

Em toda missão de desenvolvimento, ler primeiro:

1. `VISAO_E_DIRETRIZES_B2A.md`;
2. `PLANO_MESTRE_B2A.md`;
3. `AGENTS.md` e `architect/AGENTS.md`;
4. os índices aplicáveis de `architect/knowledge/`.

## 2. Ponto atual

### Confirmado no código e em testes locais

- O código do ERP foi incorporado em `erp/` a partir do commit de origem `0920bb4`, com adaptação de laboratório registrada em `erp/ORIGIN.md`.
- `npm ci`, `npm run typecheck` e `npm run build` passam em `erp/`.
- `npm run lint` permanece bloqueado pela incompatibilidade documentada entre ESLint 10 e `eslint-config-next`.
- O laboratório Supabase é separado do ambiente de produção; usa schema sintético sanitizado e dados fictícios.
- O ciclo de obra e planejamento foi validado por SQL e por REST: obra de R$ 500.000, 14 etapas, 12 meses, totais financeiros consistentes, persistência real, rejeição de dados inválidos e idempotência.
- O fluxo funcional ampliado da mesma obra validou cronograma, pendências, compras, cotações, financeiro, conciliação e isolamento por obra.
- A operação mínima `lab_criar_obra_planejamento` e o registro `lab_operation_registry` existem no laboratório, com RLS e escopo de execução documentado nos artefatos de `architect/tests/local-lab/`.
- Existe uma especificação ainda em rascunho para `Obras e Planejamento — Piloto 01` em `architect/agents/drafts/ficticio-obras-planejamento-piloto-01/`. Nenhum funcionário OpenBot foi criado ou ativado.

### Documentado, mas ainda não equivalente ao ambiente oficial

- O schema de laboratório é sintético e não prova equivalência com o schema oficial completo.
- As migrations locais não reproduzem sozinhas todas as tabelas centrais identificadas na base de conhecimento.
- A integração com o funcionário no runtime OpenBot ainda depende de wrapper/interceptor com operações fixas; não se deve expor `execute_sql` genérico.
- O estado de produção, suas credenciais, policies e integrações externas permanece fora do escopo do laboratório.

## 3. Histórico preservado

- 2026-11-24: pacote inicial do Arquiteto e knowledge base publicados.
- 2026-11-24: código do ERP incorporado e sanitizado para o laboratório.
- 2026-11-24: dependências locais instaladas; typecheck e build aprovados; lint bloqueado e documentado.
- 2026-11-24: schema sintético, fixtures e testes funcionais do laboratório criados.
- 2026-11-24: Supabase exclusivo de laboratório criado e validado sem reutilizar produção.
- 2026-11-24: operação estruturada de criação de obra e planejamento implementada no laboratório e validada por banco e REST.
- 2026-11-24: especificação do primeiro funcionário preparada como draft, aguardando aprovação antes de criação/ativação.

## 4. Próxima entrega

Preparar a camada final de operação do funcionário `Obras e Planejamento — Piloto 01`, sem criar ainda o agente:

1. implementar/verificar wrapper de ferramentas com escopo fixo para consulta, preparação, criação e conferência;
2. executar T0 de segurança e T2 funcional com três casos, incluindo dados faltantes, repetição e erro;
3. apresentar ao proprietário a especificação, permissões, ferramentas e evidências;
4. somente após aprovação explícita, criar o funcionário no OpenBot e executar o teste no runtime real.

## 5. Critérios de conclusão da próxima entrega

- O funcionário usa somente ferramentas estruturadas e autorizadas do laboratório.
- Nenhuma ferramenta aceita SQL arbitrário ou credencial administrativa.
- A criação retorna identificador oficial e evidência independente no banco.
- Planejamento persiste com pesos, arredondamento e totalização corretos.
- Repetição com a mesma chave não duplica registros.
- Dados faltantes geram pergunta humana; decisões empresariais não são inventadas.
- Erros de validação e gravação são comunicados e registrados.
- T0 não apresenta falha eliminatória e os casos T2 atingem o limiar definido em `architect/AGENTS.md`.
- O agente só é criado/ativado após aprovação do proprietário.

## 6. Conflitos e decisões pendentes

| Conflito identificado | Fonte anterior | Situação | Decisão necessária |
|---|---|---|---|
| Relatórios antigos afirmam que `erp/` contém apenas documentação e que typecheck/Supabase ainda não foram executados. | `RELATORIO-VALIDACAO.md`, `PENDENCIAS-INTEGRACAO.md` | Contradito pelo código e pelos testes atuais do laboratório; manter como histórico da fase anterior. | Atualizar esses relatórios em missão documental própria, sem apagar o histórico. |
| `erp/INTEGRACAO-LAB.md` marca ambiente e migrations como pendentes, embora o Supabase sintético e o `.env.local` do laboratório já estejam operacionais. | `erp/INTEGRACAO-LAB.md` | Parcialmente desatualizado; não deve ser interpretado como prova de equivalência oficial. | Atualizar status e separar schema sintético de schema oficial. |
| A knowledge base descreve n8n/Vercel como componentes do ERP original, enquanto a visão determina n8n desativado e laboratório sem produção. | `architect/knowledge/erp/01-visao-geral.md`, `06-apis-integracoes.md`, `architect/AGENTS.md` | Não é conflito quando o escopo é explicitado: origem/produção documentada versus arquitetura ativa do laboratório. | Preservar referências históricas e rotular o escopo em futuras revisões. |
| A visão atribui ao OpenBot execução e conversa, ao ERP registros e regras, e ao Supabase persistência; a integração atual ainda chama diretamente uma RPC do laboratório. | `VISAO_E_DIRETRIZES_B2A.md`, `architect/agents/drafts/.../tools.yaml` | Compatível como prova de laboratório, mas wrapper controlado ainda é necessário antes de ativar o agente. | Aprovar ou ajustar o contrato de ferramentas antes da ativação. |

Nenhuma dessas divergências foi resolvida silenciosamente nesta missão.

## 7. Fora do escopo desta versão

- Alterar código funcional do ERP.
- Conectar ou modificar produção.
- Criar o funcionário no OpenBot.
- Criar Orquestrador.
- Reativar n8n, Vercel ou integrações antigas.
- Migrar dados fictícios para produção.

## 8. Registro de atualização

Esta versão foi criada porque não existia `PLANO_MESTRE_B2A.md`. Os resultados anteriores foram incorporados como histórico e estado atual, sem substituir decisões registradas nos documentos anteriores.
