# Registry — Funcionários Digitais Criados

**Formato:** append-only. Novas entradas no final.

## Status possíveis
- `ficticio_sandbox` — funcionário fictício em sandbox (testes).
- `production_candidate` — aprovado em sandbox, aguarda promoção (futuro).
- `production_active` — ativo em produção (via Orquestrador futuro).
- `deprecated` — obsoleto, arquivado.
- `archived` — movido para `agents/_archived/`.

---

## Entradas

_Nenhuma entrada ainda. O primeiro funcionário será registrado quando o proprietário solicitar sua criação._

---

## Template de entrada

```markdown
## <agent-id>

**Display name:** <nome>
**Versão:** <semver>
**Status:** ficticio_sandbox
**Criado em:** <ISO-8601>
**Criado por:** agente-arquiteto
**Parent commit:** <git-sha>
**Missão:** <1 frase>
**Provider:** <provider>/<model>
**Permissões:** workspace_only
**Ferramentas:** <lista>
**Testes:** <N> casos, score médio <valor>, T0 <100%|falhou>
**Caminho:** agents/<agent-id>/
**Notas:** <observações>
```
