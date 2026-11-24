# PROMPT-INICIALIZACAO-OPENBOT.md

**Versão:** 1.0.0 · **Data:** 2026-11-24

## O que é

Prompt de bootstrap para criar o agente **Arquiteto B2A** no OpenBot, conectado ao monorepositório `B2A-LAB-ARQUITETO-ERP`.

## Pré-requisitos (a verificar antes de enviar o prompt)

- [ ] OpenBot v0.72.0+ instalado em `~/OpenBot/` (ou `%USERPROFILE%\OpenBot\` no Windows).
- [ ] Repositório `B2A-LAB-ARQUITETO-ERP` clonado localmente em um caminho estável (ex.: `~/projects/B2A-LAB-ARQUITETO-ERP`).
- [ ] Provider Codex configurado no OpenBot (Settings → Providers → Codex → sign in).
- [ ] Arquivo `.env` criado na raiz do repositório a partir de `.env.example` (ver `CONFIGURACAO-CREDENCIAIS.md`).
- [ ] Dependências do ERP instaladas: `cd erp && npm install`.
- [ ] Supabase de testes criado e migrations aplicadas (ver `CONFIGURACAO-CREDENCIAIS.md`).

Se qualquer item não estiver pronto, complete antes de enviar o prompt abaixo.

## Passo 1 — Criar o agente no OpenBot

Abra o OpenBot e:

1. Clique em **Create agent** (ou **Novo agente**).
2. **Nome:** `Arquiteto B2A`
3. **Provider:** `Codex` (recomendado). Fallback: `Claude Code`.
4. **Model:** `gpt-5` (ou o mais recente disponível no seu plano Codex).
5. **Workspace:** selecione a pasta onde o repositório foi clonado (ex.: `~/projects/B2A-LAB-ARQUITETO-ERP`).
6. **Permissões:** `Workspace only` (recomendado para início).
7. **Computer Use:** desativado (não necessário para esta fase).
8. Salve o agente.

## Passo 2 — Verificar carregamento de AGENTS.md

Após criar o agente, abra uma conversa e envie:

```
Olá. Confirme que você consegue ler o arquivo AGENTS.md na raiz do seu workspace. Liste suas 6 skills disponíveis.
```

**Resposta esperada:**
- Confirmação de leitura do `AGENTS.md`.
- Lista das 6 skills: `erp-knowledge`, `agent-design`, `skill-engineering`, `tool-integration`, `software-engineering`, `agent-evaluation`.

Se a resposta não listar as skills ou não confirmar leitura do `AGENTS.md`:

- Verifique que o workspace do agente aponta para a raiz do repositório (onde está o `AGENTS.md`).
- Codex lê skills de `.agents/skills/`. Confirme que `.agents/skills/` existe na raiz com symlinks ou cópias das skills.
- Se necessário, crie symlinks manuais:
  ```bash
  cd <workspace>
  mkdir -p .agents/skills
  for skill in architect/skills/*/; do
    name=$(basename "$skill")
    ln -sf "../../architect/skills/$name" ".agents/skills/$name"
  done
  ```
- Reinicie o agente no OpenBot após criar os symlinks.

## Passo 3 — Enviar o prompt de inicialização

Copie e cole o prompt abaixo na conversa do agente no OpenBot:

---

```
Você é o Agente Arquiteto B2A, Engenheiro Sênior de Sistemas Multiagentes e Arquiteto de Software da B2A Engenharia & Construção.

Estou ativando você no OpenBot com o workspace do monorepositório B2A-LAB-ARQUITETO-ERP.

## Sua identidade

Você NÃO é um chatbot. Você é um parceiro técnico permanente, com autonomia controlada para desenvolver código real.

## O que você tem neste workspace

1. Código completo do ERP B2A em `erp/` (cópia independente para testes).
2. Suas instruções detalhadas em `architect/AGENTS.md`.
3. Knowledge base em `architect/knowledge/` (ERP + ecossistema).
4. 6 skills operacionais em `architect/skills/`.
5. Templates, memória, permissões e testes em `architect/`.
6. Documentação na raiz: README, PROMPT-INICIALIZACAO-OPENBOT, CONFIGURACAO-CREDENCIAIS, RELATORIO-AUDITORIA-ERP, RELATORIO-VALIDACAO, PENDENCIAS-INTEGRACAO, INVENTARIO.

## Primeira missão — Auto-diagnóstico

Execute os passos abaixo sequencialmente e me reporte ao final:

1. Leia `AGENTS.md` na raiz deste workspace (já deve ter sido carregado automaticamente).
2. Leia `architect/AGENTS.md` para suas instruções completas.
3. Leia `architect/knowledge/erp/INDEX.md` e `architect/knowledge/ecosystem/INDEX.md`.
4. Liste a estrutura de `erp/` para confirmar que o código está disponível (use `ls erp/` e `ls erp/src/`).
5. Leia `RELATORIO-AUDITORIA-ERP.md` para conhecer as adaptações feitas no laboratório (n8n desativado, credenciais rotacionadas, etc.).
6. Leia `PENDENCIAS-INTEGRACAO.md` para conhecer o que ainda falta.
7. Verifique se `.env` existe na raiz (não leia o conteúdo, apenas confirme existência).
8. Tente rodar `cd erp && npm run typecheck` e reporte o resultado.
9. Identifique 3 lacunas de conhecimento que você gostaria de preencher primeiro.
10. Proponha 3 primeiras missões concretas que você conseguiria executar neste laboratório.

## Formato do relatório

Apresente o resultado no formato:

### Status do ambiente
[OK / PARCIAL / BLOQUEADO] com justificativa

### Estrutura do workspace confirmada
[lista das pastas principais encontradas]

### Resultado do typecheck
[passou / falhou com erro específico]

### Lacunas de conhecimento identificadas
1. ...
2. ...
3. ...

### Primeiras missões propostas
1. ...
2. ...
3. ...

### Pendências que dependem de mim (proprietário)
[lista de ações que só eu posso tomar]

## Regras desta conversa

- Não faça nenhuma alteração no código nesta primeira conversa.
- Não crie branches ainda.
- Não solicite acessos externos ainda.
- Apenas faça o diagnóstico e proponha próximos passos.
- Se encontrar erros, reporte-os; não tente corrigir agora.

Aguarde meu OK antes de iniciar qualquer missão de desenvolvimento.
```

---

## Passo 4 — Avaliar resposta do Arquiteto

Após enviar o prompt, o Arquiteto deve responder com o relatório de auto-diagnóstico. Avalie:

| Critério | Esperado | Ação se falhar |
|---|---|---|
| Confirma leitura de `AGENTS.md` raiz | Sim | Verificar workspace do agente |
| Lista 6 skills | Sim | Verificar `.agents/skills/` |
| Confirma `erp/` com código | Sim | Verificar se o clone do ERP foi feito |
| Reporta resultado do typecheck | Passa ou erro específico | Se erro de env, ver `CONFIGURACAO-CREDENCIAIS.md` |
| Identifica 3 lacunas de conhecimento | Sim | Aceitável |
| Propõe 3 missões concretas | Sim | Aceitável |
| Lista pendências do proprietário | Sim | Ver `PENDENCIAS-INTEGRACAO.md` |
| Não faz alterações no código | Sim | Recomeçar se modificou algo |

## Passo 5 — Verificações manuais do proprietário

Após o relatório do Arquiteto, verifique manualmente:

1. **`.env` configurado corretamente** — variáveis preenchidas com credenciais novas.
2. **Supabase de testes acessível** — acesse o painel e confirme que as migrations foram aplicadas.
3. **ERP rodando localmente** — `cd erp && npm run dev` deve iniciar em `http://localhost:3000`.
4. **Login no ERP** — criar usuário no Supabase Auth de testes, fazer login no ERP local.
5. **Denylist respeitada** — confirmar que o Arquiteto não tenta acessar `appsite-erp-b2-a-engenharia.vercel.app`.

## Passo 6 — Próximas conversas

Após o auto-diagnóstico, próximas conversas podem ser:

- **"Analise o módulo X do ERP e proponha melhorias."** → usa skill `software-engineering`.
- **"Crie um funcionário fictício que faça Y."** → usa skill `agent-design`.
- **"Pesquise ferramentas open-source para Z."** → usa skill `tool-integration`.
- **"Atualize o conhecimento sobre o módulo W."** → usa skill `erp-knowledge`.

## Limitações conhecidas deste prompt

- Não testa a skill `agent-evaluation` (exige runtime de sandbox ainda não implementado).
- Não valida acesso ao Supabase (exige configuração manual do proprietário).
- Não cria funcionários digitais (apenas prepara o Arquiteto para criá-los).

## Verificação final

Antes de considerar o Arquiteto operacional:

- [ ] Agente criado no OpenBot com workspace correto.
- [ ] Provider Codex (ou Claude) conectado.
- [ ] `AGENTS.md` raiz carregado.
- [ ] 6 skills detectadas pelo provider.
- [ ] Auto-diagnóstico executado com sucesso.
- [ ] `.env` preenchido.
- [ ] ERP roda localmente (`npm run dev` funciona).
- [ ] Supabase de testes acessível.
- [ ] Arquiteto respeita denylist (não acessa produção).

Marcando todos os itens acima, o Arquiteto está pronto para sua primeira missão real de desenvolvimento.

## Solução de problemas

| Problema | Solução |
|---|---|
| Codex não lê `AGENTS.md` | Verificar que está na raiz do workspace; reiniciar agente |
| Skills não aparecem | Criar symlinks em `.agents/skills/` (ver Passo 2) |
| Typecheck falha por env | Preencher `.env` corretamente |
| ERP não inicia | Verificar `npm install` em `erp/`; verificar Node 22+ |
| Supabase não conecta | Verificar URL e chave em `.env`; verificar projeto ativo |
| Arquiteto acessa produção | Verificar denylist em `architect/permissions/tools-catalog.yaml` |
| Erro "module not found" | Rodar `npm install` na raiz e em `erp/` |

## Próximos passos após estabilização

1. **Primeira missão de desenvolvimento:** escolher junto com o proprietário uma melhoria concreta no ERP.
2. **Primeiro funcionário fictício:** criar um funcionário de teste usando skill `agent-design`.
3. **Primeira integração:** avaliar um MCP externo útil via skill `tool-integration`.
4. **Documentação contínua:** o Arquiteto atualiza `architect/knowledge/erp/` conforme aprende.

## Notas

- Este prompt pressupõe que o repositório `B2A-LAB-ARQUITETO-ERP` já está criado no GitHub privado e clonado localmente. Se não estiver, ver `PENDENCIAS-INTEGRACAO.md`.
- O provider Codex é recomendado por seu bom suporte a skills e AGENTS.md. Claude Code também funciona (lê `.claude/skills/` — symlinks adicionais podem ser necessários).
- Não use Gemini como provider inicial para o Arquiteto — suporte a skills via OpenBot é menos testado.
