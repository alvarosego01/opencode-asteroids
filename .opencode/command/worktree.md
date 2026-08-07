---
description: Gestiona git worktrees - crea o elimina worktrees en .worktrees/<nombre>
agent: build
---

Ejecuta este comando bash, donde `$ARGUMENTS` puede ser un nombre para crear un worktree o un nombre seguido de "remove" para eliminarlo:

```bash
ARGS="$ARGUMENTS"
LAST=$(echo "$ARGS" | rev | cut -d' ' -f1 | rev | tr '[:upper:]' '[:lower:]')

if [ "$LAST" = "remove" ]; then
  NAME=$(echo "$ARGS" | rev | cut -d' ' -f2- | rev | tr '[:upper:]' '[:lower:]' | tr -s ' ' '-')
  if [ ! -d ".worktrees/$NAME" ]; then
    echo "Error: No existe el worktree '.worktrees/$NAME'"
    exit 1
  fi
  git worktree remove ".worktrees/$NAME"
  git branch -D "$NAME"
  echo "Worktree '$NAME' eliminado"
else
  NAME=$(echo "$ARGS" | tr '[:upper:]' '[:lower:]' | tr -s ' ' '-')
  git worktree add ".worktrees/$NAME"
fi
```
