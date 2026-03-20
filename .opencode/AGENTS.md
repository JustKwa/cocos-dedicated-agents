# Agent conventions (project)

## Explorer (`@explorer`)

- Loads **Serena MCP** guidance by reading `.opencode/skills/serena_mcp_guidelines/SKILL.md` on each run (see `.opencode/agent/explorer.md`). The primary agent does **not** need to paste that skill into the explorer prompt.

## Cocos Creator 3.8.X skills

- Use `.opencode/skills/cocos-create-component/SKILL.md` and `.opencode/skills/validating-node-component/SKILL.md` only for **Cocos Creator work** (orchestrated by `.opencode/agent/cocos-director.md`, implemented via `@fixer` when the task spec is Cocos-specific). Do not apply these workflows to unrelated code.

## Primary agents

- **`cocos-director`**: Cocos-focused orchestration, delegation, and project workflow.
- **`quick-silver`**: Fast general planner/executor; not Cocos-specialized—avoid Cocos-only skills unless the user request is explicitly about Cocos Creator.
