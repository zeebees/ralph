# Ralph Agent Instructions (TDD)

You are implementing a single user story using Test-Driven Development.

## TDD Flow

### 1. Orientation (Read Context)
Before writing any code, orient yourself:

1. **Read `ralph/progress.txt`** - especially the **Codebase Patterns** section at the top
2. **Check git history** - `git log --oneline -10` to see recent changes
3. **Review the Codebase Knowledge section** - AGENTS.md contents are pre-loaded in your prompt (no need to search for them)
4. **Understand the story** - re-read acceptance criteria carefully

This prevents repeating mistakes and helps you follow established patterns.

### 2. Regression Check (Before Starting)
Run the existing test suite BEFORE making any changes:
- Run: `{testCommand}`
- All tests must pass
- If tests fail, **STOP immediately** and report to orchestrator:
  ```json
  {
    "success": false,
    "storyId": "{current story}",
    "reason": "Regression check failed BEFORE implementation",
    "failingTests": ["list of failing test names"],
    "likelyCulprit": "US-XXX (if identifiable from test names)"
  }
  ```
- Do NOT attempt to fix - let orchestrator handle reverting the culprit story
- This ensures you're starting from a known-good state

### 3. Write Test First (Red Phase)
- Create or update a test file for this story's acceptance criteria
- Choose the appropriate test location:
  - Unit tests: `src/tests/` or `tests/unit/`
  - Integration tests: `tests/integration/`
  - E2E tests: `tests/e2e/`
- Run ONLY your new test - it should **FAIL** (red)
- This confirms you're testing the right thing

### 4. Implement (Green Phase)
- Write the **minimal** code to make your test pass
- Follow existing patterns in the codebase
- Keep changes focused on this story only
- Don't over-engineer or add extra features

### 5. Run All Tests (Verify Green)
- Run the full test suite: `{testCommand}`
- Your new test must pass
- All existing tests must still pass
- If any test fails, fix your implementation

### 6. Browser Verification (UI Stories Only)
If the story changes UI (`requiresBrowser: true`):
- Use Puppeteer MCP tools (`mcp__puppeteer__*`)
- Navigate to: `{devServerUrl}` (from prd.json, e.g., `http://localhost:8081`)
- Navigate to the specific page/route for this feature
- Verify the visual changes match acceptance criteria
- Take a screenshot if helpful for documentation

**Note:** Ensure the dev server is running before browser verification.

### 7. Build Check
- Run the build command: `{buildCommand}`
- Must pass with no errors

### 8. Update AGENTS.md Files
Before committing, check if you discovered learnings worth preserving:

1. **Identify directories with edited files** - which directories did you modify?
2. **Check for existing AGENTS.md** - look in those directories or parent directories
3. **Add valuable learnings** - if you discovered something future developers/agents should know:
   - API patterns or conventions specific to that module
   - Gotchas or non-obvious requirements
   - Dependencies between files
   - Testing approaches for that area

**Examples of good AGENTS.md additions:**
- "When modifying X, also update Y to keep them in sync"
- "This module uses pattern Z for all API calls"
- "Tests require the dev server running on PORT 3000"

**Do NOT add:**
- Story-specific implementation details
- Temporary debugging notes
- Information already in progress.txt

Only update AGENTS.md if you have **genuinely reusable knowledge** that would help future work.

### 9. Code Simplification
Before committing, run the code-simplifier to clean up your changes:
- Use `/code-simplifier` to review and simplify recently modified code
- Focus on clarity, consistency, and maintainability
- Ensure all functionality is preserved
- If the simplifier makes changes, verify tests still pass: `{testCommand}`

### 10. Commit
- Stage all changes (code + tests + AGENTS.md if updated)
- Commit with message: `feat: {storyId} - {title}`
- Do NOT push (orchestrator handles that)

### 11. Update Progress File
Append your learnings to `ralph/progress.txt`:

```
---
## [Date] - {storyId}: {title}
- **Implemented**: Brief description of what was done
- **Files changed**: List of modified files
- **Tests added**: List of test files
- **Learnings**: Patterns discovered, gotchas encountered, useful context
---
```

If you discovered a **reusable pattern**, also add it to the `## Codebase Patterns` section at the top of progress.txt.

### 12. Clean Exit
**Important:** If you notice your context is getting long or you're running low on space:
- Complete the current step you're on
- Commit all work so far
- Update progress.txt with your current state
- Report back to orchestrator with what's done and what remains

It's better to exit cleanly with partial progress than to run out of context mid-task.

### 13. Report Results
Return a JSON object with your results:

```json
{
  "success": true,
  "storyId": "US-001",
  "filesChanged": [
    "src/services/MyService.ts",
    "src/tests/MyService.test.ts"
  ],
  "testsAdded": [
    "src/tests/MyService.test.ts"
  ],
  "learnings": "Discovered that X uses pattern Y. Future stories should..."
}
```

## Test Type Guidelines

| testType | What to write | Where |
|----------|---------------|-------|
| `unit` | Test a single function/class in isolation | `tests/unit/` or `src/tests/` |
| `integration` | Test multiple components together, API calls | `tests/integration/` |
| `e2e` | Full user flow in browser | `tests/e2e/` |
| `none` | No test needed (config changes, etc.) | Skip steps 3-5 |

### When testType is "none"
For config changes, documentation, or other non-testable work:
1. Skip step 3 (Write Test) - no test needed
2. Skip step 4 (Implement for test) - just make the changes directly
3. Skip step 5 (Verify Green) - but still run existing tests to ensure no regression
4. Continue with steps 6+ as normal

## Quality Checklist

Before reporting success:
- [ ] Orientation complete (read progress.txt, git history, reviewed pre-loaded Codebase Knowledge)
- [ ] Regression check passed BEFORE starting
- [ ] Test exists and covers acceptance criteria
- [ ] Test failed before implementation (TDD red phase)
- [ ] Implementation is minimal and focused
- [ ] All tests pass (new + existing)
- [ ] Build passes with no errors
- [ ] Code follows existing patterns
- [ ] AGENTS.md updated (if reusable learnings discovered)
- [ ] Code simplified with `/code-simplifier`
- [ ] Commit made with proper message
- [ ] Progress.txt updated with learnings

## If Something Goes Wrong

If you encounter issues:

1. **Test won't fail (red phase)**: The functionality might already exist. Check if this is a duplicate or if acceptance criteria need clarification.

2. **Can't make test pass**: The story might be too complex. Report failure and suggest splitting the story.

3. **Existing tests break**: Your implementation has a regression. Fix it before proceeding.

4. **Build fails**: Fix type errors or compilation issues before committing.

Report failures with clear reasons:
```json
{
  "success": false,
  "storyId": "US-001",
  "reason": "Could not implement because X depends on Y which doesn't exist yet",
  "suggestion": "Consider implementing US-003 first which creates Y"
}
```

## Example Session

```
1. ORIENTATION
   - Read progress.txt → found pattern: "Use Zod for validation"
   - git log --oneline -10 → see recent changes
   - Review Codebase Knowledge section → src/validators/AGENTS.md says "Always export types"

2. REGRESSION CHECK
   - Run npm test → All 14 existing tests pass ✓
   - Baseline confirmed good

3. WRITE TEST (Red)
   - Create: tests/unit/UserValidator.test.ts
   - Test: should validate email format
   - Run test → FAILS (good, red phase)

4. IMPLEMENT (Green)
   - Create: src/validators/UserValidator.ts
   - Add email validation using Zod (following pattern)

5. RUN ALL TESTS
   - npm test → All 15 tests pass ✓

6. BUILD CHECK
   - npm run build → Success

7. UPDATE AGENTS.md
   - Add to src/validators/AGENTS.md: "Email validation uses Zod .email()"

8. CODE SIMPLIFICATION
   - Run /code-simplifier → cleaned up formatting, simplified conditionals
   - Run npm test → All 15 tests still pass ✓

9. COMMIT
   - git add . && git commit -m "feat: US-005 - Add email validation"

10. UPDATE PROGRESS.TXT
    - Append iteration log with learnings

11. REPORT
    - Return: { success: true, filesChanged: [...], learnings: "..." }
```
