# pep8-physics-skill

A Claude / Codex / Cursor **Agent Skill** that teaches the agent to name physical-quantity variables in Python correctly.

> There is no single "Python physics naming standard". The closest thing that *does* carry authority is the composition:
>
> **PEP 8** (formatting) + **Google Python Style §3.16** (short-name license) + **ISO/IEC 80000** (symbols) + **unit suffixes or `Quantity` objects** (visible units).
>
> `pep8-physics-skill` is that composition, packaged as a skill.

## What it does

When activated, the skill instructs the agent to:

- Default to descriptive `snake_case` for variables and functions (PEP 8).
- Use short / Greek-letter names (`r`, `v`, `rho`, `lambda_`) **only** when they match an established mathematical convention, and **only** when the source is cited in the docstring. (Google Python Style §3.16.)
- Draw the symbols from ISO/IEC 80000 unless a specific paper dictates otherwise.
- Make units visible with a `Quantity` object (`astropy.units`, `pint`, `unyt`), a module-level unit declaration, or a unit suffix (`_m`, `_kg`, `_nm`, `_eV`, ...).
- Spell Greek letters in Latin (`alpha`, `omega`, `lambda_`). Never use non-ASCII identifiers.
- Avoid the PEP 8 banned identifiers `l`, `O`, `I`.

Full rules, decision flow, symbol tables, and worked examples live in [`pep8-physics-skill/SKILL.md`](pep8-physics-skill/SKILL.md).

## Install

### Via [`skills.sh`](https://skills.sh) (recommended)

```sh
npx skills add Fanzzzd/pep8-physics-skill
```

This drops the skill into your active agent's skills directory (`~/.claude/skills/pep8-physics-skill/`, `.cursor/skills/pep8-physics-skill/`, etc.).

Target a specific agent:

```sh
npx skills add Fanzzzd/pep8-physics-skill -a claude-code
```

### Manual

Copy the `pep8-physics-skill/` directory into your skills directory:

```sh
# Claude Code (personal)
cp -R pep8-physics-skill ~/.claude/skills/pep8-physics-skill

# Claude Code (project)
cp -R pep8-physics-skill <your-repo>/.claude/skills/pep8-physics-skill

# Cursor (personal)
cp -R pep8-physics-skill ~/.cursor/skills/pep8-physics-skill
```

## Repository layout

```
pep8-physics-skill/
├── README.md               ← this file
├── LICENSE                 ← MIT
└── pep8-physics-skill/                 ← the skill itself
    ├── SKILL.md            ← rules, decision flow, core examples
    └── references/
        ├── iso80000.md     ← full ISO/IEC 80000 symbol tables (parts 3–12)
        ├── unit-suffixes.md ← `_m`, `_kg`, `_nm`, `_eV`, ... catalogue
        └── examples.md     ← worked snippets (mechanics, EM, thermo, astro, fluids)
```

## Scope

`pep8-physics-skill` is language-agnostic about the *physics* — it covers classical mechanics, EM, thermodynamics, optics, physical chemistry, atomic/nuclear, dimensionless numbers, and condensed matter (ISO 80000 parts 3–12). It is *Python-specific* in the formatting rules it enforces.

If you want only one subfield, you can delete the unused reference tables — the main `SKILL.md` stands on its own.

## Why not just write a comment at the top of the file?

Because a comment is read by humans, not by the agent. A skill becomes part of the agent's system prompt as soon as its description matches the task, so every function the agent writes follows the rules — no reminder needed.

## License

MIT — see [`LICENSE`](LICENSE).
