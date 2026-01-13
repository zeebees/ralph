# Ralph TDD Workflow

An autonomous development loop using Test-Driven Development. Based on [Geoffrey Huntley's Ralph pattern](https://ghuntley.com/ralph/), adapted for Claude Code with clean context per story.

## How It Works

```
┌─────────────────────────────────────────────────────────────┐
│ ORCHESTRATOR (main session)                                 │
│ - Reads prd.json, selects next story (passes: false)        │
│ - Spawns Task agent with fresh context                      │
│ - Updates prd.json after completion                         │
│ - Loops until all stories pass                              │
└─────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│ TASK AGENT (fresh context per story)                        │
│ 1. Orientation (read progress.txt, git log, AGENTS.md)      │
│ 2. Regression check BEFORE starting                         │
│ 3. Write test first (TDD red phase)                         │
│ 4. Implement feature (green phase)                          │
│ 5. Run all tests (verify green)                             │
│ 6. Browser verify if UI story                               │
│ 7. Build check                                              │
│ 8. Update AGENTS.md (if reusable learnings)                 │
│ 9. Commit                                                   │
│ 10. Update progress.txt                                     │
└─────────────────────────────────────────────────────────────┘
```

## Quick Start

### 1. Create Your PRD

Copy `prd.json.example` to `prd.json` and define your stories:

```json
{
  "project": "neuron-ai",
  "branchName": "ralph/my-feature",
  "description": "My Feature Description",
  "testCommand": "npm test",
  "buildCommand": "npm run build",
  "userStories": [
    {
      "id": "US-001",
      "title": "Story title",
      "description": "As a user, I want...",
      "acceptanceCriteria": ["Criteria 1", "Criteria 2"],
      "testType": "unit",
      "requiresBrowser": false,
      "priority": 1,
      "passes": false,
      "notes": ""
    }
  ]
}
```

### 2. Start the Orchestrator

In Claude Code, say one of:

```
# Option 1: Reference the prompt file
Read ralph/ORCHESTRATOR_PROMPT.md and follow its instructions to implement the stories in ralph/prd.json

# Option 2: Direct command
Run Ralph: read ralph/prd.json, spawn agents for each story using TDD, update passes when done
```

The orchestrator will:
1. Read your prd.json
2. Create/checkout the branch
3. Spawn Task agents (fresh context) for each story
4. Loop until all `passes: true` or max failures

### 3. Monitor Progress

```bash
# Check story status
cat ralph/prd.json | jq '.userStories[] | {id, title, passes}'

# View learnings
cat ralph/progress.txt

# Check git history
git log --oneline -10
```

## PRD Schema

| Field | Type | Description |
|-------|------|-------------|
| `project` | string | Project name |
| `branchName` | string | Git branch (e.g., `ralph/feature-name`) |
| `testCommand` | string | Command to run tests (e.g., `npm test`) |
| `buildCommand` | string | Command to build/typecheck (e.g., `npm run build`) |
| `devServerUrl` | string | URL for browser testing (e.g., `http://localhost:8081`) |

### User Story Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique identifier (e.g., `US-001`) |
| `title` | string | Short title |
| `description` | string | "As a..., I want..., so that..." |
| `acceptanceCriteria` | string[] | Verifiable criteria |
| `testType` | `unit` \| `integration` \| `e2e` \| `none` | What kind of test to write |
| `requiresBrowser` | boolean | Needs Puppeteer verification |
| `priority` | number | Execution order (1 = first) |
| `passes` | boolean | Set to `true` when complete |
| `failureCount` | number | Number of failed attempts (for blocking logic) |
| `notes` | string | Failure reasons, learnings |

## TDD Flow Per Story

1. **Orientation**: Read progress.txt, git history, AGENTS.md
2. **Regression Check**: Run all tests BEFORE starting (ensure baseline is green)
3. **Red Phase**: Write test that fails
4. **Green Phase**: Write minimal code to pass
5. **Verify Green**: Run all tests again
6. **Browser Check**: If UI story, verify visually
7. **Build Check**: Ensure types/compilation pass
8. **Update AGENTS.md**: Add reusable learnings to code directories
9. **Commit**: `feat: US-XXX - Title`
10. **Update progress.txt**: Append learnings

## Knowledge Persistence

| File | Purpose | Lifetime | Loading |
|------|---------|----------|---------|
| `progress.txt` | Iteration logs, session details | Per-feature (archived) | Agent reads manually |
| `AGENTS.md` | Permanent codebase knowledge | Forever (in code dirs) | **Auto-loaded by orchestrator** |

**AGENTS.md** files live near the code (e.g., `src/services/AGENTS.md`) and contain learnings that should persist beyond the current Ralph run:
- "When modifying X, also update Y"
- "This module uses pattern Z for all API calls"
- "Tests require dev server on PORT 3000"

**Auto-loading**: The orchestrator automatically finds all AGENTS.md files and includes their contents in each agent's prompt. Agents don't need to search for them.

## Story Sizing

Each story should be completable in one context window.

**Good (small)**:
- Add a validation function
- Create an API endpoint
- Add a form field

**Too big (split these)**:
- "Build the entire dashboard"
- "Add authentication"
- "Refactor the API"

## Files

| File | Purpose |
|------|---------|
| `ORCHESTRATOR_PROMPT.md` | Instructions for the main loop |
| `AGENT_PROMPT.md` | TDD instructions for each story |
| `PRD_CREATOR_PROMPT.md` | Create PRDs interactively |
| `PRD_TO_JSON_PROMPT.md` | Convert PRDs to prd.json |
| `prd.json` | Your user stories (create from example) |
| `prd.json.example` | Template/reference |
| `progress.txt` | Learnings and iteration log |

## Creating PRDs

### Option 1: Interactive PRD Creation
```
Use ralph/PRD_CREATOR_PROMPT.md to create a PRD for [your feature]
```
This will ask clarifying questions and generate `docs/plans/prd-[feature].md`

### Option 2: Convert Existing PRD to JSON
```
Use ralph/PRD_TO_JSON_PROMPT.md to convert docs/plans/prd-[feature].md to prd.json
```
This validates story sizing, ordering, and creates `ralph/prd.json`

## Completion

When all stories have `passes: true`, the orchestrator outputs:

```
<promise>COMPLETE</promise>
```

## Troubleshooting

**Story keeps failing**:
- Check `notes` field in prd.json for error details
- Review progress.txt for patterns that might help
- Consider splitting the story into smaller pieces
- After 3 failures, story is marked `BLOCKED` - may need manual intervention

**Story marked BLOCKED**:
- `failureCount >= 3` triggers blocking
- Review notes for failure patterns
- May need to: split story, fix dependencies, or clarify requirements
- Reset `failureCount` to 0 and clear BLOCKED from notes to retry

**Regression check fails at start**:
- Previous story broke something
- Orchestrator will revert culprit story to `passes: false`
- That story must fix the regression before continuing

**Tests not found**:
- Ensure test files follow project conventions
- Check testType matches where tests should go

**Browser verification fails**:
- Make sure dev server is running at `devServerUrl`
- Verify the correct URL in Puppeteer navigation
- Check that Puppeteer MCP is available
