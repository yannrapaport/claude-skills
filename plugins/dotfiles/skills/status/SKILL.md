---
name: dotfiles:status
description: Check dotfiles config drift and symlink health across machines
---

# Dotfiles Status Check

Run a health check on the dotfiles installation.

## Steps

1. **Check symlinks** — verify all expected symlinks from `~/projects/admin/dotfiles/install.sh` are intact:
   ```bash
   for f in ~/.zshrc ~/.gitconfig ~/.vimrc ~/.profile ~/.tmux.conf ~/.claude/settings.json ~/.claude/statusline.sh ~/.p10k.zsh; do
     if [ -L "$f" ]; then echo "OK: $f -> $(readlink $f)"; else echo "BROKEN: $f"; fi
   done
   ```

2. **Check profile symlinks**:
   ```bash
   for f in ~/.claude/profiles/*.json; do
     if [ -L "$f" ]; then echo "OK: $f -> $(readlink $f)"; else echo "BROKEN: $f"; fi
   done
   ```

3. **Check op template symlinks**:
   ```bash
   for f in ~/.secrets/op.*.env; do
     if [ -L "$f" ]; then echo "OK: $f -> $(readlink $f)"; else echo "MISSING: $f"; fi
   done
   ```

4. **Check git status** of dotfiles repo:
   ```bash
   cd ~/projects/admin/dotfiles && git status --short && git log --oneline -3
   ```

5. **Run drift check** if available:
   ```bash
   command -v dotfiles-drift-check && dotfiles-drift-check
   ```

## Output

Present results as a table with PASS/WARN/FAIL status for each category.
