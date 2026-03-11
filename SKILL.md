---
name: modelling-team
description: >
  Build Excel financial models using a three-agent modelling team: Architect (designs structure),
  Coder (writes Python/openpyxl), and Challenger (stress-tests logic). Use this skill whenever the
  user asks to build, create, or construct an Excel model, financial model, investment model,
  valuation model, DCF, LBO, P&L, or any quantitative spreadsheet. Also trigger when the user says
  "model this", "build me a spreadsheet for", or mentions building projections, scenarios, or
  forecasts in Excel. Even if the user doesn't say "Excel" explicitly — if they're asking for a
  quantitative model with assumptions, formulas, and outputs, use this skill.
---

# Modelling Team

Three specialist agents collaborate to produce a polished, error-free Excel model. Each agent has a distinct role, runs on a specific Claude model, and hands off a clear deliverable to the next.

## Workflow Overview

```
User request
    │
    ▼
┌─ Step 0: Clarify scope ─────────────────────────────────┐
│  Ask: Heavy (multi-tab) or Light (single-tab)?           │
│  Gather context, confirm output path and key metrics.    │
└──────────────────────────────────────────────────────────┘
    │
    ▼
┌─ Step 1: MODEL ARCHITECT  (Opus) ────────────────────────┐
│  Designs the full model blueprint: tabs, row layout,     │
│  assumption cells, formula logic, scenario selector,     │
│  dashboard design, formatting plan.                      │
│  Output: structured blueprint in markdown.               │
└──────────────────────────────────────────────────────────┘
    │
    ▼
┌─ Step 2: MODEL CODER  (Sonnet) ─────────────────────────┐
│  Translates the blueprint into a Python/openpyxl script. │
│  Runs it. Produces the .xlsx file.                       │
└──────────────────────────────────────────────────────────┘
    │
    ▼
┌─ Step 3: MODEL CHALLENGER  (Opus) ──────────────────────┐
│  Reviews the .xlsx: checks formula logic, stress-tests   │
│  assumptions, validates cross-references and scenario    │
│  selector wiring, checks dashboard links.                │
│  Output: PASS / PASS WITH NOTES / FAIL + fix list.      │
└──────────────────────────────────────────────────────────┘
    │
    ▼
  If FAIL → fixes back to Coder → re-run Challenger
  If PASS → deliver to user
```

## Step 0: Clarify Scope

Before spawning agents, ask the user:

> **Is this a heavy model (multiple tabs, scenarios, dashboard) or a light model (single tab)?**

- **Light**: Single worksheet. Assumptions at the top, calculations below, outputs/KPIs at the bottom.
- **Heavy**: Multiple worksheets with a **Dashboard** tab as the front page, an **Assumptions** tab, one or more calculation/P&L tabs, and a **Scenario Selector** on the Dashboard that drives all scenario-dependent formulas across the model.

Also confirm: output file path, data/assumptions to feed in, time horizon, and key metrics (IRR, NPV, PBP, EBITDA, etc.).

## Step 1: Model Architect

Spawn a **general-purpose Agent**. This agent does NOT write code — it designs the model structure.

Read `references/architect-prompt.md` for the full agent prompt template. Fill in: `{USER_REQUEST}`, `{MODEL_MODE}`, `{OUTPUT_PATH}`, `{ADDITIONAL_CONTEXT}`.

### Blueprint Structure

1. **Model metadata**: file name, mode, time horizon, currency/units
2. **Tab plan** (heavy mode): tab names, purposes, data flow between tabs
3. **Row-by-row layout per tab**: row number, label, cell type (`input` / `formula` / `cross-sheet` / `header`), formula logic in pseudo-Excel, number format, style
4. **Assumption register**: every hardcoded number, its value, source, cell location, and downstream references
5. **Key outputs**: final-answer cells (IRR, NPV, PBP, etc.) and where they sit
6. **Formatting plan**: column widths, freeze panes, section headers

### Heavy Mode Requirements

The Architect MUST include in any heavy-mode blueprint:

**Dashboard tab** (always the first tab):
- Title row and model description
- **Scenario Selector** in a prominent cell (e.g., `Dashboard!B3`): a cell containing 1, 2, or 3 (mapped to Low / Base / High), with Data Validation dropdown
- Key output metrics pulled from calculation tabs via cross-sheet links
- Summary tables/KPIs that give the user the full picture without opening other tabs

**Scenario selector mechanism**:
- The Assumptions tab stores Low / Base / High values in three adjacent columns (e.g., B = Low, C = Base, D = High)
- Every formula in the model that references a scenario-dependent assumption uses `INDEX()` to select the right column based on the scenario selector: `=INDEX(Assumptions!B5:D5, 1, Dashboard!$B$3)`
- The scenario selector cell uses Data Validation (List: "1,2,3") with a label cell next to it showing `=CHOOSE(B3,"Low","Base","High")`
- When the user changes the selector from 2 to 1 or 3, the entire model recalculates for that scenario

### What makes a good blueprint
- Every formula is written out — the Coder never invents formula logic
- Cross-sheet references are explicit (e.g., `='Assumptions'!C10`)
- Edge cases handled (division by zero, IRR on all-negative CFs)
- No magic numbers in formulas — all assumptions in dedicated cells
- Scenario selector wiring is specified for every assumption that varies by scenario
- **Cell reference $-notation** is specified for every formula so the model can be extended manually (see Code Standards below)

## Step 2: Model Coder

Spawn a **general-purpose Agent**. This agent writes and runs the Python script.

Read `references/coder-prompt.md` for the full agent prompt template. Fill in: `{BLUEPRINT}`, `{OUTPUT_PATH}`, `{SCRIPT_PATH}`.

### Design Language (mandatory)

The Coder MUST use this exact visual design system — derived from an established house style. Include these definitions at the top of every script:

```python
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment, Border, Side
from openpyxl.utils import get_column_letter
from openpyxl.worksheet.datavalidation import DataValidation

# ── Palette ─────────────────────────────────────────
NAVY    = "1F3864"; MED_BLUE = "2F5496"; LT_BLUE = "D9E1F2"
WHITE   = "FFFFFF"; LT_GREY  = "F2F2F2"; MED_GREY = "D9D9D9"
INPUT_BG = "EBF3FB"; LT_GRN = "E2EFDA"; LT_OG = "FCE4D6"
LT_YLW  = "FFF2CC"; AMBER = "F4B942"; HDR_GRN = "375623"

# ── Fonts ───────────────────────────────────────────
F_INPUT = Font(name="Arial", size=10, color="0000FF")       # blue: editable inputs
F_CALC  = Font(name="Arial", size=10, color="000000")       # black: formulas
F_XSHT  = Font(name="Arial", size=10, color="008000")       # green: cross-sheet links
F_HDR   = Font(name="Arial", size=10, bold=True, color="FFFFFF")  # white bold: headers
F_LBL   = Font(name="Arial", size=10)                       # regular labels
F_BOLD  = Font(name="Arial", size=10, bold=True)             # bold labels / totals
F_TITLE = Font(name="Arial", size=14, bold=True, color="FFFFFF")  # title rows
F_NOTE  = Font(name="Arial", size=9,  italic=True, color="595959") # notes

# ── Borders & fills ─────────────────────────────────
ts = Side(style="thin", color="B8B8B8")
def tb(): return Border(left=ts, right=ts, top=ts, bottom=ts)
def fill(c): return PatternFill("solid", start_color=c)

# ── Reusable helpers ────────────────────────────────
def title_row(ws, row, text, end_col="G", h=28):
    ws.merge_cells(f"A{row}:{end_col}{row}")
    c = ws[f"A{row}"]
    c.value = text; c.font = F_TITLE; c.fill = fill(NAVY)
    c.alignment = Alignment(horizontal="center", vertical="center")
    ws.row_dimensions[row].height = h

def sec(ws, row, text, end_col="G", bg=MED_BLUE):
    ws.merge_cells(f"A{row}:{end_col}{row}")
    c = ws[f"A{row}"]
    c.value = text; c.font = F_HDR; c.fill = fill(bg)
    c.alignment = Alignment(horizontal="left", vertical="center")
    c.border = tb(); ws.row_dimensions[row].height = 20

def hc(ws, r, col, text, bg=MED_BLUE):
    c = ws.cell(r, col, text)
    c.font = F_HDR; c.fill = fill(bg)
    c.alignment = Alignment(horizontal="center", vertical="center", wrap_text=True)
    c.border = tb(); return c

def lbl(ws, r, col, text, bold=False, bg=None):
    c = ws.cell(r, col, text)
    c.font = F_BOLD if bold else F_LBL
    c.fill = fill(bg or (LT_GREY if bold else WHITE))
    c.alignment = Alignment(horizontal="left", vertical="center", wrap_text=True)
    c.border = tb(); return c

def inp(ws, r, col, val, fmt="$#,##0;($#,##0);-"):
    c = ws.cell(r, col, val)
    c.font = F_INPUT; c.fill = fill(INPUT_BG)
    c.number_format = fmt
    c.alignment = Alignment(horizontal="center", vertical="center")
    c.border = tb(); return c

def calc(ws, r, col, formula, fmt="$#,##0;($#,##0);-", xsh=False):
    c = ws.cell(r, col, formula)
    c.font = F_XSHT if xsh else F_CALC; c.fill = fill(WHITE)
    c.number_format = fmt
    c.alignment = Alignment(horizontal="center", vertical="center")
    c.border = tb(); return c

def bold_calc(ws, r, col, formula, fmt="$#,##0;($#,##0);-", bg=LT_GRN):
    c = ws.cell(r, col, formula)
    c.font = F_BOLD; c.fill = fill(bg)
    c.number_format = fmt
    c.alignment = Alignment(horizontal="center", vertical="center")
    c.border = tb(); return c

def note_row(ws, row, text, end_col="G", h=50):
    ws.merge_cells(f"A{row}:{end_col}{row}")
    c = ws[f"A{row}"]
    c.value = text; c.font = Font(name="Arial", size=10, italic=True, color="1F3864")
    c.alignment = Alignment(wrap_text=True, horizontal="left", vertical="top")
    ws.row_dimensions[row].height = h
```

### Scenario Selector Implementation (heavy mode)

On the Dashboard tab, the Coder must create:
```python
# Scenario selector (cell B3 = 1/2/3 → Low/Base/High)
lbl(ws_dash, 2, 1, "Select scenario:", bold=True)
inp(ws_dash, 2, 2, 2, "0")  # default = 2 (Base)
calc(ws_dash, 2, 3, '=CHOOSE(B2,"Low","Base","High")', "@")
dv = DataValidation(type="list", formula1='"1,2,3"', allow_blank=False)
dv.prompt = "1=Low, 2=Base, 3=High"
ws_dash.add_data_validation(dv)
dv.add(ws_dash["B2"])
```

Then every scenario-dependent formula throughout the model references it:
```python
# Instead of hardcoding base column:
calc(ws, row, col, "=INDEX(Assumptions!B10:D10,1,Dashboard!$B$2)", fmt)
```

### Number formats
- Currency ($K): `'$#,##0;($#,##0);-'`
- Currency ($M): `'$#,##0.0;($#,##0.0);-'`
- Percentages: `'0.0%;(0.0%);-'`
- Multiples: `'0.0x;(0.0x);-'`
- Years: `'0'` (not `'#,##0'` — avoids `2,026`)
- PBP in years: `'0.0'`
- Negatives in parentheses. Zeros as dashes.

### Cell Reference $-Notation (manual extensibility)

The generated model must be ready for a human analyst to extend — e.g., dragging formulas to new columns/periods. Every formula must use proper mixed/absolute references:

- **Period/year header rows** (e.g., row 10 with years 2026, 2027, ...): use **row-fixed mixed ref** like `C$10`. The column changes when dragged right (C→D→E), but the row stays pinned to the header row.
- **Fixed assumption cells** (e.g., discount rate in D5): use **absolute ref** like `$D$5`. Both row and column are locked — dragging the formula in any direction still points to the same assumption cell.
- **Same-column references** within a calculation block (e.g., Revenue in the same column): use a plain column letter with a fixed row if the row is structural, e.g., `C$15` for "Revenue is always in row 15 of this tab."
- **Cross-sheet fixed cells** like the scenario selector: already absolute — `Dashboard!$B$2`.

**Example**: A formula in row 20 (EBITDA), column C (Year 1), that multiplies revenue (row 15) by a margin assumption (Assumptions!D8):
```
=C$15 * $D$8          ← light model (assumptions on same tab)
=C$15 * INDEX(Assumptions!$B$8:$D$8,1,Dashboard!$B$2)   ← heavy model
```
When dragged to column D (Year 2), this becomes `=D$15 * ...` — correct.

### Code Standards
- **openpyxl only** (no pandas for model construction)
- **All calculations as Excel formulas** — never compute in Python and hardcode
- **Cell reference $-notation**: every formula must use proper mixed/absolute references as described above — this is critical for manual extensibility
- **Error prevention**: `IF(x=0,0,...)` for divisions, `IFERROR(IRR(...),"N/A")` for IRR
- **Zoom level**: set `ws.sheet_view.zoomScale = 85` on every worksheet so the model fits comfortably on screen when first opened
- **IRR / NPV cash flow stream**: IRR requires a negative initial outflow (Year 0 = initial investment or negative enterprise value). Never compute IRR on only positive cash flows — the result is meaningless. Build a dedicated CF row that starts with the negative investment in the first cell.
- Write the script to disk, run with `py -3 <script_path>`, confirm .xlsx saved

## Step 3: Model Challenger

Spawn a **general-purpose Agent**. This agent is the quality gate.

Read `references/challenger-prompt.md` for the full agent prompt template. Fill in: `{XLSX_PATH}`, `{BLUEPRINT_SUMMARY}`, `{USER_REQUEST}`.

### Review Checklist

1. **Formula Reference Audit**: off-by-one errors, dangling cross-sheet links, SUM range coverage
2. **Scenario Selector Validation** (heavy mode):
   - Does Dashboard!B2 (or wherever the selector lives) have Data Validation?
   - Do ALL scenario-dependent formulas use `INDEX(..., Dashboard!$B$2)` (or equivalent)?
   - Change the selector value mentally to 1 and 3 — do all references still resolve correctly?
   - Are there any formulas that hardcode a specific scenario column (e.g., always column C) instead of using the selector?
3. **Dashboard Link Validation**: does every metric on the Dashboard correctly link to its source cell in the calculation tabs?
4. **Assumption Stress Test**: boundary values, implicit assumptions, reasonableness
5. **Structural Integrity**: orphan inputs, total consistency, time horizon alignment
6. **Output Validation**: IRR/PBP/DPBP correctness, cumulative CF logic
7. **Formatting Check**: color coding, number formats, freeze panes

### Verdict
- **PASS** — clean, no material issues
- **PASS WITH NOTES** — works correctly, has improvement suggestions
- **FAIL** — specific errors with cell references and fix instructions

## Agent Model Selection

| Agent | Claude Model | Why |
|---|---|---|
| Architect | **Opus** (`claude-opus-4-6`) | Conceptual design requires deep reasoning about model structure, edge cases, and formula interdependencies |
| Coder | **Sonnet** (`claude-sonnet-4-6`) | Fast, precise code generation following a detailed blueprint |
| Challenger | **Opus** (`claude-opus-4-6`) | Adversarial review requires the same depth of reasoning as architecture — finding subtle formula bugs and logic gaps |

When spawning agents, you do NOT need to specify the model explicitly — the orchestrator (you) runs on whatever model the user has set. But in the agent prompt, tell each agent what it is and reference the appropriate prompt template.

## Handling Iterations

If the Challenger returns FAIL:
1. Extract fix instructions (specific cells and corrections)
2. Re-spawn the Coder with the original blueprint PLUS the fix list
3. Re-spawn the Challenger on the updated file
4. Maximum 3 iterations — if still failing, surface issues to the user

## Delivering the Result

Once the Challenger passes:
- State where the `.xlsx` file was saved
- Summarize: tabs, key assumptions, scenario selector location, dashboard metrics
- Share any Challenger notes worth mentioning
- Remind the user: blue-font cells are editable inputs; change Dashboard scenario selector (1/2/3) to flip between Low/Base/High
