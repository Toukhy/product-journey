# Product Journey — Gameball

A Claude Code-powered product management tool that walks you through Gameball's complete user story journey — from feature idea to Azure DevOps-ready work item.

## What It Does

When you describe a feature idea, Claude guides you through every step of the product journey:

1. **Intake** — Capture the feature idea and source
2. **Strategic Fit** — Evaluate against OKRs, strategy, and revenue goals
3. **Product Fit** — Determine which plans and platforms it applies to
4. **Scope Check** — Decide if full market research is needed
5. **Market Research** — Analyze competitor pricing (if large feature)
6. **Competitor Analysis** — Compare with Talon.One & Capillary Tech (if large feature)
7. **Story Writing** — Generate user story with acceptance criteria
8. **UX Review** — Assess design needs
9. **Tech Review** — Evaluate architecture impact and effort
10. **Output** — Generate Azure DevOps-ready markdown

## Setup

### Prerequisites
- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed

### Usage

```bash
# Clone the repo
git clone <repo-url>
cd product-journey

# Start Claude Code
claude

# Then just describe your feature:
# "I have a feature idea: Add a scratch-and-win game to the loyalty widget"
# Or simply type: "start"
```

Claude will read the Gameball context (OKRs, plans, competitors, feature matrix) and walk you through each step, auto-evaluating decisions and generating the final output.

## Project Structure

```
product-journey/
├── CLAUDE.md              # Claude's instructions (the brain)
├── README.md              # This file
├── context/
│   ├── company.md         # Gameball OKRs, plans, competitors
│   ├── plan-features.csv  # Full feature-by-plan matrix
│   └── flowchart.html     # Visual process flowchart
└── output/                # Generated user stories go here
```

## Output

Generated user stories are saved as markdown files in the `output/` folder, formatted for direct paste into Azure DevOps work items.

## Updating Company Context

- Edit `context/company.md` to update OKRs, goals, or competitors
- Replace `context/plan-features.csv` with an updated export from the pricing spreadsheet
