# Ralph PRD Converter

Converts existing PRDs to the prd.json format that Ralph uses for autonomous execution.

Based on the battle-tested amp-ralph converter skill, with TDD enhancements.

---

## The Job

Take a PRD (markdown file or text) and convert it to `ralph/prd.json`.

---

## Output Format

```json
{
  "project": "[Project Name]",
  "branchName": "ralph/[feature-name-kebab-case]",
  "description": "[Feature description from PRD title/intro]",
  "testCommand": "npm test",
  "buildCommand": "npm run build",
  "devServerUrl": "http://localhost:8081",
  "userStories": [
    {
      "id": "US-001",
      "title": "[Story title]",
      "description": "As a [user], I want [feature] so that [benefit]",
      "acceptanceCriteria": [
        "Criterion 1",
        "Criterion 2",
        "Typecheck passes"
      ],
      "testType": "unit",
      "requiresBrowser": false,
      "priority": 1,
      "passes": false,
      "failureCount": 0,
      "notes": ""
    }
  ]
}
```

---

## Story Size: The Number One Rule

**Each story must be completable in ONE Ralph iteration (one context window).**

Ralph spawns a fresh agent per iteration with no memory of previous work. If a story is too big, the LLM runs out of context before finishing and produces broken code.

### Right-sized stories:
- Add a database column and migration
- Add a UI component to an existing page
- Update a server action with new logic
- Add a filter dropdown to a list

### Too big (split these):
- "Build the entire dashboard" - Split into: schema, queries, UI components, filters
- "Add authentication" - Split into: schema, middleware, login UI, session handling
- "Refactor the API" - Split into one story per endpoint or pattern

**Rule of thumb:** If you cannot describe the change in 2-3 sentences, it is too big.

---

## Story Ordering: Dependencies First

Stories execute in priority order. Earlier stories must not depend on later ones.

**Correct order:**
1. Schema/database changes (migrations)
2. Server actions / backend logic
3. UI components that use the backend
4. Dashboard/summary views that aggregate data

**Wrong order:**
1. UI component (depends on schema that does not exist yet)
2. Schema change

---

## Acceptance Criteria: Must Be Verifiable

Each criterion must be something Ralph can CHECK, not something vague.

### Good criteria (verifiable):
- "Add `status` column to tasks table with default 'pending'"
- "Filter dropdown has options: All, Active, Completed"
- "Clicking delete shows confirmation dialog"
- "Typecheck passes"
- "Tests pass"

### Bad criteria (vague):
- "Works correctly"
- "User can do X easily"
- "Good UX"
- "Handles edge cases"

### Always include as final criterion:
```
"Typecheck passes"
```

For stories with testable logic, also include:
```
"Tests pass"
```

### For stories that change UI, also include:
```
"Verify in browser"
```

Frontend stories are NOT complete until visually verified.

---

## Test Type Mapping

| PRD says | testType value | requiresBrowser |
|----------|----------------|-----------------|
| "unit test", "unit" | `"unit"` | `false` |
| "integration test", "API test" | `"integration"` | `false` |
| "e2e", "browser test", "UI test" | `"e2e"` | `true` |
| "no test", config changes | `"none"` | `false` |
| Story mentions "verify in browser" | (keep testType) | `true` |

---

## Conversion Rules

1. **Each user story becomes one JSON entry**
2. **IDs**: Sequential (US-001, US-002, etc.)
3. **Priority**: Based on dependency order, then document order
4. **All stories**: `passes: false` and empty `notes`
5. **branchName**: Derive from feature name, kebab-case, prefixed with `ralph/`
6. **Always add**: "Typecheck passes" to every story's acceptance criteria
7. **testType**: Map from PRD's Test Type field
8. **requiresBrowser**: Set true for e2e or UI stories

---

## Splitting Large PRDs

If a PRD has big features, split them:

**Original:**
> "Add user notification system"

**Split into:**
1. US-001: Add notifications table to database
2. US-002: Create notification service for sending notifications
3. US-003: Add notification bell icon to header
4. US-004: Create notification dropdown panel
5. US-005: Add mark-as-read functionality
6. US-006: Add notification preferences page

Each is one focused change that can be completed and verified independently.

---

## Example

**Input PRD:**
```markdown
# Task Status Feature

Add ability to mark tasks with different statuses.

## User Stories

### US-001: Add status field to tasks table
**Description:** As a developer, I need to store task status in the database.
**Test Type:** unit

**Acceptance Criteria:**
- Add status column: 'pending' | 'in_progress' | 'done' (default 'pending')
- Generate and run migration successfully
- Typecheck passes

### US-002: Display status badge on task cards
**Description:** As a user, I want to see task status at a glance.
**Test Type:** e2e

**Acceptance Criteria:**
- Each task card shows colored status badge
- Badge colors: gray=pending, blue=in_progress, green=done
- Typecheck passes
- Verify in browser
```

**Output prd.json:**
```json
{
  "project": "neuron-ai",
  "branchName": "ralph/task-status",
  "description": "Task Status Feature - Track task progress with status indicators",
  "testCommand": "npm test",
  "buildCommand": "npm run build",
  "devServerUrl": "http://localhost:8081",
  "userStories": [
    {
      "id": "US-001",
      "title": "Add status field to tasks table",
      "description": "As a developer, I need to store task status in the database.",
      "acceptanceCriteria": [
        "Add status column: 'pending' | 'in_progress' | 'done' (default 'pending')",
        "Generate and run migration successfully",
        "Typecheck passes"
      ],
      "testType": "unit",
      "requiresBrowser": false,
      "priority": 1,
      "passes": false,
      "failureCount": 0,
      "notes": ""
    },
    {
      "id": "US-002",
      "title": "Display status badge on task cards",
      "description": "As a user, I want to see task status at a glance.",
      "acceptanceCriteria": [
        "Each task card shows colored status badge",
        "Badge colors: gray=pending, blue=in_progress, green=done",
        "Typecheck passes",
        "Verify in browser"
      ],
      "testType": "e2e",
      "requiresBrowser": true,
      "priority": 2,
      "passes": false,
      "failureCount": 0,
      "notes": ""
    }
  ]
}
```

---

## Archiving Previous Runs

**Before writing a new prd.json, check if there is an existing one from a different feature:**

1. Read the current `ralph/prd.json` if it exists
2. Check if `branchName` differs from the new feature's branch name
3. If different AND `ralph/progress.txt` has content beyond the header:
   - Create archive folder: `ralph/archive/YYYY-MM-DD-feature-name/`
   - Copy current `prd.json` and `progress.txt` to archive
   - Reset `progress.txt` (keep Codebase Patterns section)

---

## Checklist Before Saving

Before writing prd.json, verify:

- [ ] **Previous run archived** (if prd.json exists with different branchName, archive it first)
- [ ] Each story is completable in one iteration (small enough)
- [ ] Stories are ordered by dependency (schema → backend → UI)
- [ ] Every story has "Typecheck passes" as criterion
- [ ] UI stories have "Verify in browser" as criterion
- [ ] UI stories have `requiresBrowser: true`
- [ ] testType is set appropriately for each story
- [ ] Acceptance criteria are verifiable (not vague)
- [ ] No story depends on a later story
- [ ] All stories have `passes: false` and `failureCount: 0`
- [ ] `devServerUrl` is set for browser testing
