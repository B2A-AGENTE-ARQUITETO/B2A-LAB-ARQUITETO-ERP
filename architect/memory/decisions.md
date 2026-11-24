# Memory — Decisões Técnicas

**Formato:** append-only. Novas decisões no final. Nunca editar entradas anteriores.

---

## 2026-11-24 — Inicialização do Arquiteto

**Contexto:** primeira ativação do Agente Arquiteto após entrega do pacote ao OpenBot.

**Decisão:** Arquiteto inicia em estado default:
- Sem acessos externos ativos.
- Knowledge base contém apenas referências ao Catálogo de Capacidades do ERP B2A (entrega anterior).
- Nenhum funcionário digital criado.
- Nenhuma ferramenta externa verificada.

**Rationale:** respeitar P1 (independência do ERP por padrão) e P3 (acesso progressivo). O Arquiteto começa isolado e solicita acessos conforme necessário.

**Alternativas consideradas:**
- Pré-cadastrar acessos ao ERP B2A — rejeitado por violar P1.
- Pré-criar funcionário fictício de exemplo — adiado para primeira missão do proprietário.

**Impacto:** todas as operações iniciais limitam-se a ler knowledge base interna e conversar com o proprietário.

**Effective from:** 2026-11-24T00:00:00Z

**Linked artifacts:** `AGENTS.md`, `knowledge/erp/INDEX.md`, `knowledge/ecosystem/INDEX.md`
