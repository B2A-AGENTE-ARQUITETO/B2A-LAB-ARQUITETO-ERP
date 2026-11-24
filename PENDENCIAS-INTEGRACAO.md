# PENDENCIAS-INTEGRACAO.md

**Versão:** 1.1.0 · **Data:** 2026-11-24

## Sumário

Lista de pendências que **exigem ação do proprietário** para completar a integração do monorepo `B2A-LAB-ARQUITETO-ERP`. Itens ordenados por prioridade.

---

## Pendências bloqueadoras (alta prioridade)

### P1 — Criar repositório GitHub privado ✓ CONCLUÍDO

**Status:** concluído
**Quem:** proprietário
**Quando:** antes de qualquer push

**Resultado:** Repositório criado em `B2A-AGENTE-ARQUITETO/B2A-LAB-ARQUITETO-ERP` (conta nova, privada).

---

### P2 — Copiar código do ERP original para `erp/`

**Status:** pendente
**Quem:** proprietário (fornece acesso) + Arquiteto (executa clone após acesso)
**Quando:** antes do primeiro uso do laboratório

**Por quê:** durante a montagem do monorepo, o código-fonte completo do ERP **não foi copiado fisicamente** para `erp/` porque a conta autenticada no plugin GitHub (`B2A-AGENTE-ARQUITETO`) NÃO tem acesso ao repo original (`anderson110288-afk/APPSITE-ERP-B2A-Engenharia`).

**Como executar:**
1. Proprietário fornecer acesso de leitura ao repositório `anderson110288-afk/APPSITE-ERP-B2A-Engenharia` para a conta `B2A-AGENTE-ARQUITETO` (ou token de acesso pessoal com escopo `repo:read` para o repo original).
2. Após acesso concedido, Arquiteto clona localmente:
   ```bash
   git clone https://github.com/anderson110288-afk/APPSITE-ERP-B2A-Engenharia.git /tmp/erp-original
   ```
3. Copiar conteúdo para `erp/` (sem `.git` e sem `.kilocode`):
   ```bash
   cd B2A-LAB-ARQUITETO-ERP
   cp -r /tmp/erp-original/* erp/
   cp -r /tmp/erp-original/.github erp/ 2>/dev/null || true
   rm -rf /tmp/erp-original
   ```
4. Aplicar adaptações de `erp/INTEGRACAO-LAB.md` (itens 1–6).
5. Commit das alterações no monorepo.
6. Validar com procedimento de `erp/INTEGRACAO-LAB.md` seção "Validação pós-aplicação".

**Autorização necessária do proprietário:**
- Confirmar permissão de leitura ao repo original.
- Fornecer acesso (convite de colaborador read-only OU PAT com escopo mínimo).

---

### P3 — Criar projeto Supabase NOVO

**Status:** pendente
**Quem:** proprietário
**Quando:** antes de rodar o ERP localmente

**Por quê:** o ERP depende de Supabase para Auth + PostgreSQL. O projeto de produção B2A **não deve ser reutilizado**. É necessário criar projeto novo, aplicar schema e migrations.

**Como executar:** ver `CONFIGURACAO-CREDENCIAIS.md` Passo 2 (passo a passo completo).

**Autorização necessária do proprietário:**
- Confirmar criação de conta NOVA no Supabase.
- Confirmar criação de projeto novo.
- Fornecer (em cofre seguro, não no chat): URL, anon key, service_role key.

---

### P4 — Obter Google Gemini API key

**Status:** pendente
**Quem:** proprietário
**Quando:** antes de testar o endpoint `/api/agent`

**Por quê:** o endpoint `/api/agent` do ERP usa Gemini 2.0 Flash via `@google/generative-ai`. É necessário API key própria do laboratório.

**Como executar:** ver `CONFIGURACAO-CREDENCIAIS.md` Passo 3.

**Autorização necessária do proprietário:**
- Acessar `https://aistudio.google.com/apikey`.
- Criar chave.
- Armazenar em cofre (não no chat).

---

### P5 — Preencher `.env`

**Status:** pendente
**Quem:** proprietário
**Quando:** antes de rodar o ERP ou ativar o Arquiteto

**Por quê:** sem `.env` preenchido, o ERP não conecta ao Supabase nem ao Gemini.

**Como executar:** ver `CONFIGURACAO-CREDENCIAIS.md` Passo 5.

**Verificação:**
```bash
cd B2A-LAB-ARQUITETO-ERP
test -f .env && echo "OK" || echo "ERRO: .env não existe"
# Confirmar que não está no Git
git status .env
```

---

## Pendências não bloqueadoras (média prioridade)

### P6 — Instalar OpenBot e configurar provider

**Status:** pendente
**Quem:** proprietário
**Quando:** antes de ativar o Arquiteto

**Como executar:** ver `CONFIGURACAO-CREDENCIAIS.md` Passo 6 e `PROMPT-INICIALIZACAO-OPENBOT.md`.

**Itens:**
- [ ] Baixar e instalar OpenBot v0.72.0+.
- [ ] Configurar provider Codex (recomendado) ou Claude Code.
- [ ] Criar symlinks em `.agents/skills/` (e `.claude/skills/` se Claude).
- [ ] Criar agente "Arquiteto B2A" apontando para o workspace.
- [ ] Enviar prompt de inicialização (`PROMPT-INICIALIZACAO-OPENBOT.md` Passo 3).

---

### P7 — Aplicar migrations no Supabase novo

**Status:** pendente (após P3)
**Quem:** proprietário
**Quando:** após criar projeto Supabase

**Como executar:** ver `CONFIGURACAO-CREDENCIAIS.md` Passo 2.3.

**Itens:**
- [ ] Exportar schema-base do Supabase de produção (apenas schema, sem dados) — `pg_dump --schema-only`.
- [ ] Aplicar schema-base no Supabase novo.
- [ ] Aplicar 12 migrations locais de `erp/supabase/migrations/` em ordem cronológica.
- [ ] Criar 1 usuário admin em Authentication → Users.
- [ ] Verificar RLS com query em `pg_policies`.

**Autorização necessária do proprietário:**
- Acesso temporário ao Supabase de produção (apenas para `pg_dump --schema-only`).
- Ou: fornecer schema-base.sql exportado (sem dados).

---

### P8 — Auditar histórico Git por segredos

**Status:** pendente (após P2)
**Quem:** proprietário
**Quando:** antes do primeiro push para o GitHub privado

**Como executar:** ver `erp/INTEGRACAO-LAB.md` item 6.

**Recomendação:** usar **início limpo** (descartar histórico antigo, commitar apenas estado atual). Esta abordagem elimina qualquer risco de segredos antigos no histórico.

---

### P9 — Configurar Computer Use no OpenBot (opcional)

**Status:** opcional
**Quem:** proprietário
**Quando:** se o Arquiteto precisar controlar browser

**Como executar:** OpenBot Settings → Computer Use → ativar (requer permissões adicionais no macOS).

**Não recomendado para v1.0.0.** Avaliar necessidade após primeira missão real do Arquiteto.

---

## Pendências de baixa prioridade (pós-estabilização)

### P10 — Definir primeira missão concreta do Arquiteto

**Status:** pendente
**Quem:** proprietário + Arquiteto
**Quando:** após estabilização do laboratório

**Sugestões iniciais** (a definir com o proprietário):
- Corrigir L10 (`marcarPedidoPago` não atualiza lançamento financeiro).
- Implementar L2 (preencher campos `conciliacao_financeira` com match automático).
- Implementar L1 (CRUD para `PendenciaComentario`).
- Corrigir S1 (comparação constant-time do bearer token).
- Implementar L6 (cache com invalidação seletiva).

Cada uma é uma missão concreta que o Arquiteto pode executar usando a skill `software-engineering`.

---

### P11 — Primeiro funcionário digital fictício

**Status:** pendente
**Quem:** proprietário + Arquiteto
**Quando:** após P10

**Sugestão:** criar funcionário fictício que valide CNPJ (skill `agent-design` + `skill-engineering`).

---

### P12 — Primeira integração MCP externa

**Status:** pendente
**Quem:** proprietário + Arquiteto
**Quando:** após P11

**Sugestão:** avaliar MCP server de filesystem ou GitHub read-only (skill `tool-integration`).

---

## Pendências documentais

### P13 — Atualizar `RELATORIO-VALIDACAO.md` após popular `erp/`

Após o proprietário concluir P2 (copiar ERP), atualizar `RELATORIO-VALIDACAO.md` com:
- Confirmação de que `erp/` está populado.
- Resultado do `npm run typecheck` em `erp/`.
- Resultado do `npm run build` em `erp/`.
- Validação das adaptações de `INTEGRACAO-LAB.md`.

---

### P14 — Atualizar `architect/knowledge/erp/` com base no código real

Após P2, o Arquiteto deve:
1. Ler código em `erp/`.
2. Comparar com conhecimento em `architect/knowledge/erp/`.
3. Atualizar entradas marcadas `[DESCONHECIDO]` para `[VERIFICADO]` quando confirmar.
4. Registrar divergências em `architect/memory/divergences.md`.
5. Atualizar `last_verified_at` em cada documento.

---

## Ordem recomendada de execução

```
1. P1 (criar GitHub conta + repo privado) ✓ CONCLUÍDO
   ↓
2. P2 (clonar ERP original para erp/) — PRÓXIMA AÇÃO
   ↓
3. P8 (auditar histórico Git por segredos)
   ↓
4. P3 (criar Supabase novo)
   ↓
5. P7 (aplicar migrations)
   ↓
6. P4 (obter Gemini API key)
   ↓
7. P5 (preencher .env)
   ↓
8. Validar ERP rodando localmente (npm run dev)
   ↓
9. P6 (instalar OpenBot + provider)
   ↓
10. Criar agente Arquiteto no OpenBot
    ↓
11. Enviar PROMPT-INICIALIZACAO-OPENBOT.md
    ↓
12. P10 (primeira missão de desenvolvimento)
    ↓
13. P11 (primeiro funcionário fictício)
    ↓
14. P12 (primeira integração MCP)
    ↓
15. P13 + P14 (atualizações documentais)
```

---

## Verificação final (checklist)

Após concluir todas as pendências, validar:

- [x] P1: repositório GitHub privado criado e acessível.
- [ ] P2: `erp/` populado com código do ERP original.
- [ ] P3: projeto Supabase novo criado.
- [ ] P4: Gemini API key obtida.
- [ ] P5: `.env` preenchido (e NÃO commited).
- [ ] P6: OpenBot instalado e provider conectado.
- [ ] P7: migrations aplicadas no Supabase novo.
- [ ] P8: histórico Git auditado (ou início limpo).
- [ ] ERP roda localmente: `cd erp && npm run dev` funciona.
- [ ] Login no ERP com usuário admin do Supabase novo funciona.
- [ ] Endpoint `/api/agent` responde (Gemini ativo).
- [ ] Arquiteto criado no OpenBot e responde ao prompt de inicialização.
- [ ] Arquiteto respeita denylist (não acessa produção).
- [ ] `architect/knowledge/erp/` atualizado com base no código real.

Marcando todos os itens, o laboratório está **100% operacional**.

---

## Ação imediata solicitada

Para destravar a próxima fase (P2 — clonar ERP original), preciso que o proprietário:

1. **Forneça acesso de leitura ao repositório `anderson110288-afk/APPSITE-ERP-B2A-Engenharia`** para a conta `B2A-AGENTE-ARQUITETO`.
   - Opção A: convidar `B2A-AGENTE-ARQUITETO` como colaborador read-only no repo original.
   - Opção B: criar Personal Access Token (PAT) na conta `anderson110288-afk` com escopo mínimo `repo:read` para o repo específico, e fornecer via canal seguro (não no chat).
   - Opção C: proprietário clona localmente e faz upload do conteúdo.

Após acesso concedido, o Arquiteto clona o ERP, aplica as adaptações de `erp/INTEGRACAO-LAB.md`, e faz commit no monorepo.
