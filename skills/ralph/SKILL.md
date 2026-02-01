---
name: ralph
description: Autonomous TDD development loop. Use when implementing features from a prd.json file using test-driven development with fresh-context agents per story.
---

# Ralph: Autonomous TDD Development

You are the orchestrator for a TDD-based development loop. Your job is to implement user stories by spawning Task agents with fresh context for each story.

## Prerequisites

Before starting, ensure:
1. `prd.json` exists in project root (copy from `prd.json.example` if needed)
2. The prd.json contains `testCommand`, `buildCommand`, and `userStories`

## Orchestrator Loop

Execute this loop until all stories pass:

### Step 1: Load Context

```
1. Read prd.json to get user stories and commands
2. Read progress.txt for learnings from previous iterations
3. Find all AGENTS.md files: find . -name "AGENTS.md" -type f
4. Read each AGENTS.md file (these contain permanent codebase knowledge)
```

### Step 2: Select Story

Find the highest priority story where `passes: false` and `failureCount < 3`.

If no such story exists:
- If all stories have `passes: true` → Output `<promise>COMPLETE</promise>` and stop
- If remaining stories are blocked (failureCount >= 3) → Report blocked stories and stop

### Step 3: Ensure Branch

```bash
git checkout {prd.branchName} 2>/dev/null || git checkout -b {prd.branchName}
```

### Step 4: Spawn Task Agent

Use the Task tool with `subagent_type: "general-purpose"` and this prompt:

```
You are implementing a single user story using TDD.

## Story Details
- ID: {story.id}
- Title: {story.title}
- Description: {story.description}
- Test Type: {story.testType}
- Requires Browser: {story.requiresBrowser}

## Acceptance Criteria
{story.acceptanceCriteria as bullet list}

## Commands
- Test: {prd.testCommand}
- Build: {prd.buildCommand}
- Dev Server: {prd.devServerUrl}

## Codebase Knowledge (from AGENTS.md files)
{Include contents of each AGENTS.md file found, or "No AGENTS.md files found yet."}

## TDD Flow

1. **Orientation**: Read progress.txt (especially Codebase Patterns section), run `git log --oneline -10`

2. **Regression Check**: Run {prd.testCommand}. ALL tests must pass before starting.
   - If tests fail, STOP and report: { "success": false, "reason": "Regression check failed", "failingTests": [...] }

3. **Write Test First (Red)**: Create test for acceptance criteria. Run it - must FAIL.

4. **Implement (Green)**: Write minimal code to pass the test. Follow existing patterns.

5. **Run All Tests**: Run {prd.testCommand}. All must pass.

6. **Browser Verification** (if requiresBrowser: true): Use Puppeteer to verify UI at {prd.devServerUrl}

7. **Build Check**: Run {prd.buildCommand}. Must pass.

8. **Update AGENTS.md**: Add reusable learnings to AGENTS.md in modified directories (only genuinely reusable knowledge).

9. **Commit**: `git add . && git commit -m "feat: {story.id} - {story.title}"`

10. **Update progress.txt**: Append iteration log with learnings.

11. **Report Results**:
{
  "success": true/false,
  "storyId": "{story.id}",
  "filesChanged": ["list of files"],
  "testsAdded": ["list of test files"],
  "learnings": "What you learned"
}

## Rules
- Work on ONLY this one story
- If regression check fails BEFORE implementation, STOP and report
- Keep changes minimal and focused
- Follow existing code patterns
- If context gets long, commit work and report partial progress
```

### Step 5: Process Agent Result

**If success: true**:
1. Update prd.json: Set `story.passes = true`
2. Verify agent updated progress.txt
3. Continue to next story

**If success: false**:
1. Increment `story.failureCount` in prd.json
2. Add failure reason to `story.notes`
3. Log failure to progress.txt
4. If `failureCount >= 3`: Add "BLOCKED" to notes, move to next story
5. Otherwise: Loop will spawn fresh agent to retry

**If regression detected** (tests failed before implementation):
1. Identify culprit story from failing tests
2. Set culprit story's `passes = false`
3. Log regression to progress.txt
4. Next iteration will fix the culprit

### Step 6: Loop

Return to Step 2 until all stories complete or are blocked.

## Completion

When all stories have `passes: true`:
```
<promise>COMPLETE</promise>
```

## Quick Commands

Check story status:
```bash
cat prd.json | jq '.userStories[] | {id, title, passes, failureCount}'
```

View learnings:
```bash
cat progress.txt
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Story keeps failing | Check notes field, consider splitting story |
| Story marked BLOCKED | Review notes, may need manual intervention |
| Regression at start | Previous story broke something, will be reverted |
