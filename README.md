# Modelling Team — Claude Code Skill

A Claude Code skill that builds polished Excel financial models via a three-agent team: **Architect** (Opus) designs the blueprint, **Coder** (Sonnet) writes Python/openpyxl, **Challenger** (Opus) stress-tests formulas. Supports light (single-tab) and heavy (multi-tab with Dashboard, scenario selector, INDEX-driven scenarios) modes.

## How It Works

```
User request
    |
    v
 Step 0: Clarify scope (heavy or light?)
    |
    v
 Step 1: MODEL ARCHITECT (Opus)
         Designs full model blueprint: tabs, rows, formulas, scenarios
    |
    v
 Step 2: MODEL CODER (Sonnet)
         Translates blueprint into Python/openpyxl script, runs it
    |
    v
 Step 3: MODEL CHALLENGER (Opus)
         Reviews .xlsx: formula audit, scenario validation, design check
    |
    v
 PASS -> deliver | FAIL -> fix and re-check (up to 3 iterations)
```

## Features

- **Two modes**: Light (single tab, assumptions at top) and Heavy (Dashboard + Assumptions + calculation tabs)
- **Scenario selector** (heavy mode): Dashboard cell drives Low/Base/High via `INDEX()` — no hardcoded scenario columns
- **Professional design language**: Navy/blue palette, Arial font, color-coded cells (blue = inputs, black = formulas, green = cross-sheet links)
- **Manual extensibility**: All formulas use proper `$`-notation (mixed/absolute references) so you can drag formulas to new columns
- **IRR/NPV/PBP/DPBP**: Correctly constructed cash flow streams with negative initial investment
- **Zero formula errors**: Challenger validates every formula before delivery

## Installation

### From `.skill` file

```bash
claude install-skill modelling-team.skill
```

### From this repo

Copy the `SKILL.md` and `references/` directory into your Claude Code skills folder:

```
~/.claude/skills/modelling-team/
├── SKILL.md
└── references/
    ├── architect-prompt.md
    ├── coder-prompt.md
    └── challenger-prompt.md
```

## Usage

Just ask Claude Code to build a model:

> "Build me a DCF model for a coffee shop: $200K revenue, 5% growth, 30% EBITDA margin, 10% discount rate, 4x terminal multiple, 5-year horizon."

> "Build me a heavy model for a rental portfolio with 3 properties, scenario analysis for rent growth and vacancy, and a dashboard with IRR and cash-on-cash return."

Claude will ask whether you want a light or heavy model, then run the three-agent workflow automatically.

## File Structure

```
modelling-team/
├── SKILL.md                          # Main skill definition
├── references/
│   ├── architect-prompt.md           # Blueprint design agent prompt
│   ├── coder-prompt.md               # Python/openpyxl coding agent prompt
│   └── challenger-prompt.md          # Validation/review agent prompt
└── modelling-team.skill              # Packaged distributable
```

## Design Language

The skill enforces a consistent visual style across all models:

| Element | Color | Usage |
|---|---|---|
| Blue font (`0000FF`) | Inputs | Editable assumptions |
| Black font (`000000`) | Formulas | All calculations |
| Green font (`008000`) | Cross-sheet | Links between tabs |
| Navy fill (`1F3864`) | Title rows | Model/section titles |
| Medium blue fill (`2F5496`) | Section headers | Sub-sections |
| Light blue fill (`EBF3FB`) | Input cells | Background for editable cells |
| Light green fill (`E2EFDA`) | Totals/KPIs | Bold summary rows |

## Requirements

- [Claude Code](https://claude.com/claude-code) with access to Opus and Sonnet models
- Python 3 with `openpyxl` installed
- LibreOffice (for formula recalculation via the xlsx skill)

## License

MIT
