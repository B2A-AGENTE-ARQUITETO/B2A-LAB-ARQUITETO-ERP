# .agents/skills/ — Skills para o provider (Codex/OpenBot)

**Status:** vazio por design (v1.0.0). Symlinks a criar pós-instalação.

## Finalidade

O OpenBot e o Codex CLI leem skills do diretório `.agents/skills/` (no workspace do agente). O Claude Code lê de `.claude/skills/`. Esta pasta existe para receber symlinks das skills em `architect/skills/`.

## Como criar os symlinks (após clonar o monorepo)

```bash
cd B2A-LAB-ARQUITETO-ERP

# Para Codex e providers compatíveis
mkdir -p .agents/skills
for skill in architect/skills/*/; do
  name=$(basename "$skill")
  ln -sf "../../architect/skills/$name" ".agents/skills/$name"
done

# Para Claude Code (também)
mkdir -p .claude/skills
for skill in architect/skills/*/; do
  name=$(basename "$skill")
  ln -sf "../../architect/skills/$name" ".claude/skills/$name"
done
```

## Verificação

```bash
# Listar symlinks criados
ls -la .agents/skills/
ls -la .claude/skills/

# Confirmar que cada symlink aponta para SKILL.md válido
for skill in .agents/skills/*/SKILL.md; do
  test -f "$skill" && echo "OK: $skill" || echo "ERRO: $skill"
done
```

## Por que symlinks e não cópias

- **Single source of truth:** as skills vivem em `architect/skills/`. Symlinks evitam duplicação.
- **Manutenção:** ao atualizar uma skill em `architect/skills/`, ela automaticamente está disponível em `.agents/skills/`.
- **Versionamento Git:** symlinks são commitados; o Git preserva o link relativo.

## Notas

- Em Windows, pode ser necessário habilitar "Developer Mode" para criar symlinks sem privilégios de admin.
- Como alternativa ao symlink, copiar as skills manualmente:
  ```bash
  cp -r architect/skills/* .agents/skills/
  ```
  Mas atenção: mudanças em `architect/skills/` não serão refletidas automaticamente.

## Após criar symlinks

Reiniciar o agente no OpenBot para que ele detecte as skills. Ver `PROMPT-INICIALIZACAO-OPENBOT.md` Passo 2 para validação.
