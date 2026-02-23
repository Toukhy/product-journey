# Product Journey — Gameball

A Claude Code-powered product management tool that walks you through Gameball's complete user story journey — from feature idea to a User Story created directly in Azure DevOps.

## What It Does

When you describe a feature idea, Claude guides you through every step of the product journey:

1. **Intake** — Capture the feature idea and source
2. **Strategic Fit** — Evaluate against OKRs, strategy, and revenue goals
3. **Product Fit** — Determine which plans and platforms it applies to
4. **Scope Check** — Decide if full market research is needed
5. **Market Research** — Analyze competitor pricing (if large feature)
6. **Competitor Analysis** — Compare with Talon.One & Capillary Tech (if large feature)
7. **Story Writing** — Generate full user story using the Azure DevOps template
8. **UX Review** — Assess design needs
9. **Tech Review** — Evaluate architecture impact and effort
10. **Create in Azure DevOps** — Push the User Story directly to Azure DevOps via API

## Setup

### Prerequisites
- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed

### Installation

```bash
# Clone the repo
git clone https://github.com/Toukhy/product-journey.git
cd product-journey

# Create your .env file with Azure DevOps credentials
cp .env.example .env
# Edit .env and add your PAT
```

### .env Configuration

Create a `.env` file in the project root with:

```
AZURE_DEVOPS_ORG=Gameballers
AZURE_DEVOPS_PROJECT=Gameball Main Project
AZURE_DEVOPS_PAT=your-personal-access-token-here
```

To get a PAT: Azure DevOps → User Settings → Personal Access Tokens → New Token (needs Work Items: Read & Write scope).

### Usage

```bash
cd product-journey
claude
```

Then just type:
- `start`
- `I have a feature idea: Add a scratch-and-win game to the loyalty widget`
- Or describe any feature you want to evaluate

Claude will walk you through all steps and create the User Story in Azure DevOps at the end.

## Project Structure

```
product-journey/
├── CLAUDE.md                        # Claude's instructions (the brain)
├── README.md                        # This file
├── .env                             # Azure DevOps credentials (not committed)
├── .env.example                     # Template for .env
├── .gitignore
├── context/
│   ├── company.md                   # Gameball OKRs, plans, competitors
│   ├── plan-features.csv            # Full feature-by-plan matrix
│   ├── user-story-template.md       # Azure DevOps user story template
│   └── flowchart.html               # Visual process flowchart
└── output/                          # Generated user stories saved here
```

## Updating Company Context

- Edit `context/company.md` to update OKRs, goals, or competitors
- Replace `context/plan-features.csv` with an updated export from the pricing spreadsheet
- Edit `context/user-story-template.md` to change the user story format
