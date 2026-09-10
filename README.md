# hany-skills

Personal archive of agent skills collected from Cursor, Codex, official Flutter packs, and the Speedly project.

This repo is a **private personal archive**, not for public redistribution. Third-party skill text remains copyright of the original authors (Cursor, Codex, Flutter, Stripe, Speedly). Copies here are for backup and reuse on your machines.

## Layout

```text
cursor/             Cursor built-in skills (~/.cursor/skills-cursor)
agents/             Codex / agents skills (~/.agents/skills)
flutter/            Official Flutter skills (from pos_vendor)
projects/speedly/   Speedly-only project skills
CATALOG.md          Name, description, and path for every skill
```

Overlapping names between `cursor/` and `agents/` are kept on purpose so neither tree is dropped.

See [CATALOG.md](CATALOG.md) for the full list (114 skills).

## Use a skill

Copy or symlink a skill folder into a personal or project skills directory:

```sh
# Personal (Cursor)
ln -s "$PWD/flutter/flutter-build-responsive-layout" ~/.cursor/skills/flutter-build-responsive-layout

# Project
ln -s /Users/hany/Mostql/hany-skills/projects/speedly/speedly-bloc-feature-architect \
  .agents/skills/speedly-bloc-feature-architect
```

Or copy instead of linking:

```sh
cp -R flutter/flutter-add-widget-test ~/.cursor/skills/
```

Do not write into `~/.cursor/skills-cursor/` — that directory is Cursor-managed.

## Not included

- Figma and Notion plugin skills
- VS Code Python extension skills
- Projects that had no skill folders (Fekra_kfc, moda_app, planner, Autobotz)
