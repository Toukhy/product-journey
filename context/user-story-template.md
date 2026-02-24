# Azure DevOps User Story Template

## Title
{Clear, concise title — what is being added or changed}

---

## User Story
**As a** {role}
**I want** {capability}
**So that** {outcome / value}

---

## Description
{2–4 sentences max. State the current limitation, what is changing, and what modules are affected.
Do not repeat the acceptance criteria here. Focus on context a developer needs to understand the "why".}

---

## Acceptance Criteria

{Write as a flat checklist. Each item should be a testable, unambiguous statement.
Only group into sub-sections if the feature has genuinely distinct areas (e.g., UI vs. API vs. plan gating).
Do not add sections just to fill the template — if it is a small feature, 3–5 bullet points is enough.
If it is a large feature, group logically but only include groups that add real criteria.}

- [ ] {What the system must do — specific and testable}
- [ ] {What the user can do — from their perspective}
- [ ] {Edge case or error behavior — only if non-obvious}

---

## Plans
{Only list plans where behavior differs. If all plans behave the same, one line is enough.}

| Platform | Plans |
|----------|-------|
| Shopify | {e.g., Pro, Guru} |
| Salla | {e.g., Pro, Guru} |
| Self-Serve | {e.g., Growth, Enterprise} |

---

## Notes
{Optional. Only include if there are real open questions, constraints, or decisions deferred to dev/design.
If there is nothing meaningful to add, remove this section entirely.}
- {Assumption or open question}

---

<!--
GUIDANCE FOR WRITING THIS STORY (remove before publishing):

DESCRIPTION LENGTH:
- UI tweak / small addition → 1–2 sentences
- New setting or configuration option → 2–3 sentences
- New module or integration → 3–4 sentences with impacted modules listed

ACCEPTANCE CRITERIA:
- Small feature (UI tweak, new config, minor addition): flat list, 3–7 items, no sub-sections
- Medium feature (new workflow, new page, API changes): flat list or 2–3 logical groups, 8–15 items
- Large feature (new module, integration, major flow change): grouped by area (e.g., Configuration,
  Behavior, Error Handling, Security), 15–25 items max

ONLY add these groups when the feature actually requires them:
- "Error Handling" → only if there are non-trivial failure states
- "Security & Permissions" → only if access control is part of the feature scope
- "Performance" → only if there is a latency, scale, or load requirement specific to this feature
- "Backward Compatibility" → only if the change could break existing behavior

ALWAYS include:
- What the user/merchant can do (functional)
- What the system does in response (behavioral)
- Plan gating (which plans get access)

NEVER include (unless directly relevant):
- Generic statements like "existing functionality must not break" with no specifics
- Documentation requirements (handled separately)
- Obvious validations (e.g., "required fields must not be empty") unless the field has non-standard rules
-->
