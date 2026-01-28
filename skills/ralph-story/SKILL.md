---
name: ralph-story
description: Implement a single user story using TDD. Use when manually implementing one story from prd.json.
---

# Implement Story with TDD

Implement a single user story using strict Test-Driven Development.

## Usage

Specify which story to implement:
- By ID: `/ralph-story US-003`
- Next pending: `/ralph-story` (picks highest priority with passes: false)

## TDD Flow

### 1. Orientation

```
1. Read prd.json to get story details and commands
2. Read progress.txt (especially Codebase Patterns section at top)
3. Run: git log --oneline -10
4. Find and read any AGENTS.md files in the codebase
```

### 2. Regression Check

**Before writing any code**, run the test suite:
```bash
{testCommand from prd.json}
```

All tests must pass. If tests fail:
- STOP immediately
- Do not attempt to fix
- Report which tests failed
- This indicates a previous story broke something

### 3. Write Test First (Red Phase)

1. Create/update test file for this story's acceptance criteria
2. Test location by type:
   - `unit`: `tests/unit/` or `src/__tests__/`
   - `integration`: `tests/integration/`
   - `e2e`: `tests/e2e/`
3. Run ONLY your new test
4. It must FAIL (red phase confirms you're testing the right thing)

If `testType: "none"`, skip to step 5.

### 4. Implement (Green Phase)

- Write **minimal** code to make the test pass
- Follow existing patterns in the codebase
- Keep changes focused on this story only
- No over-engineering or extra features

### 5. Verify All Tests Pass

```bash
{testCommand from prd.json}
```

- Your new test must pass
- All existing tests must still pass
- Fix any regressions before proceeding

### 6. Browser Verification (if requiresBrowser: true)

Use Puppeteer MCP tools:
1. Navigate to `{devServerUrl from prd.json}`
2. Navigate to the specific page for this feature
3. Verify acceptance criteria visually
4. Screenshot if helpful

### 7. Build Check

```bash
{buildCommand from prd.json}
```

Must pass with no errors.

### 8. Update AGENTS.md

If you discovered reusable patterns:
1. Find/create AGENTS.md in directories you modified
2. Add genuinely reusable knowledge:
   - API patterns specific to that module
   - Non-obvious conventions
   - Testing gotchas
   - File dependencies

**Do NOT add**: Story-specific details, temporary notes, obvious info.

### 9. Commit

```bash
git add .
git commit -m "feat: {storyId} - {story.title}"
```

Do NOT push (orchestrator handles that).

### 10. Update progress.txt

Append:
```
---
## [Date] - {storyId}: {title}
- **Implemented**: Brief description
- **Files changed**: List of files
- **Tests added**: List of test files
- **Learnings**: Patterns discovered, gotchas encountered
---
```

Add reusable patterns to the `## Codebase Patterns` section at top.

### 11. Update prd.json

Set `passes: true` for this story.

### 12. Report

Output results:
```json
{
  "success": true,
  "storyId": "US-XXX",
  "filesChanged": ["list"],
  "testsAdded": ["list"],
  "learnings": "What you learned"
}
```

## Quality Checklist

Before reporting success:
- [ ] Regression check passed before starting
- [ ] Test exists and covers acceptance criteria
- [ ] Test failed before implementation (red phase)
- [ ] Implementation is minimal and focused
- [ ] All tests pass (new + existing)
- [ ] Build passes
- [ ] Code follows existing patterns
- [ ] AGENTS.md updated (if applicable)
- [ ] Commit made with proper message
- [ ] progress.txt updated
- [ ] prd.json updated

## If Something Goes Wrong

**Test won't fail (red phase)**: Functionality may already exist. Check for duplicates.

**Can't make test pass**: Story may be too complex. Report failure, suggest splitting.

**Existing tests break**: You have a regression. Fix before proceeding.

**Build fails**: Fix type/compilation errors before committing.

Report failures with clear reasons:
```json
{
  "success": false,
  "storyId": "US-XXX",
  "reason": "Clear explanation",
  "suggestion": "Potential solution"
}
```
