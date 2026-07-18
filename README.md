# Safety Stock 2.0 — Excel calculator audit & build toolkit

![License](https://img.shields.io/github/license/ojekjojek-creator/safety-stock-toolkit)
![Claude Skill](https://img.shields.io/badge/Claude-Skill-D97757?logo=anthropic&logoColor=white)
![Domain](https://img.shields.io/badge/Domain-Supply%20Chain-blue)

## What it is

A Claude AI skill (and a methodology for manual use in Excel) for calculating and
**auditing** safety stock (SS) across SKU portfolios. It includes 16 safety stock
methods (King, DDMRP, Croston/SBA, Fill Rate/ESC, empirical quantile, King with
σ_err correction), an automatic per-SKU method-selection decision tree, and a
catalog of **real-world Excel formula errors** encountered in practice (e.g.
comparing text to a number inside `IF`, non-existent functions like
`POISSON.INV`, or an incorrect unary minus in `EXP(-A1^2/2)`).

## Why it works better

Built and validated on a real-world project: **907 SKUs**, suppliers across
Turkey/China/Europe, an Excel file with ~56,000 formulas, six rounds of expert
review.

| Metric | Without the skill | With the skill |
|---|---|---|
| Safety stock formula audit accuracy | 51% | **100%** |

## Who it's for

- Logistics and procurement managers building their own SS/ROP calculators in Excel
- Anyone reviewing someone else's Excel inventory calculations ("are these formulas correct?")
- Claude/Claude Code users who want to add supply chain domain knowledge as a skill

## How to use it

### As a Claude skill
```
1. Clone this repository or copy the folder into ~/.claude/skills/
2. Ask Claude: "audit this Excel file with safety stock calculations"
   or "build a safety stock calculator for my SKU portfolio"
3. The skill automatically detects the mode: AUDIT / BUILD / SINGLE SKU
```

### As a standalone methodology (no AI)
The files in `references/` (`metody_zb_16.md`, `struktura_v36.md`) describe the
formulas and calculator structure — they can be used directly as a checklist when
building your own Excel spreadsheet.

## Repository structure

```
├── SKILL.md              # skill definition (frontmatter + methodology)
├── references/
│   ├── metody_zb_16.md   # 16 safety stock calculation methods
│   └── struktura_v36.md  # Excel calculator structure
├── LICENSE               # MIT
└── README.md
```

## Methods covered

King's Formula, DDMRP, Croston/SBA (intermittent demand), Fill Rate/ESC,
empirical quantile, King with σ_err correction, supplier fill-rate correction
(effective lead time), supplier profiles for Turkey/China/Europe, ABC/XYZ
classification.

## License

MIT — see [LICENSE](LICENSE).

## Contact

Questions and collaboration proposals: [ojekjojek@gmail.com](mailto:ojekjojek@gmail.com)
