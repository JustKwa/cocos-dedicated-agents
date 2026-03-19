---
name: grill-me
description: Interview the user relentlessly about a plan or design until reaching shared understanding, resolving each branch of the decision tree. Use when user wants to stress-test a plan, get grilled on their design, or mentions "grill me".
---

Interview me relentlessly about every aspect of this plan until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one.

If a question can be answered by exploring the codebase, explore the codebase using @explorer subagent instead.

After I'm satisfied with the plan, consult @oracle subagent to review the plan and approve the plan.
If the plan is approved, save to docs/plans/YYYY-MM-DD-<topic>-plan.md.
