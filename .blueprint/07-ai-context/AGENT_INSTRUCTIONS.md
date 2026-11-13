`.blueprint/07-ai-context/AGENT_INSTRUCTIONS.md`
# AI agent instructions

1. Start with [../DOCUMENT_AUTHORITY.md](../DOCUMENT_AUTHORITY.md),
   [../BASELINE-V1.md](../BASELINE-V1.md), and `.execution/current-state.md`.
2. Locate the assigned task ID in [../04-tasks/task-index.md](../04-tasks/task-index.md)
   and read its dedicated task file (`04-tasks/Tnnn-*.md`, `V2-TASKS.md`, or
   `V3-TASKS.md`) along with [../04-tasks/EXECUTION_CONTRACTS.md](../04-tasks/EXECUTION_CONTRACTS.md).
3. Every task file contains a **Self-contained implementation context** section
   so you do not need to read the entire repository to execute one task.
4. Before edits, check actual readiness in `.execution/task-status.md` and
   preserve existing user changes. Implement only the assigned bounded task, run
   its verification commands, and record honest evidence in `.execution/reports/`.
5. Do not equate documentation completion, owner acceptance, or runtime tests
   with a working compiler, passing release, or universal formal proof.
