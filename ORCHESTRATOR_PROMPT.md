# Ralph Orchestrator Instructions

You are the orchestrator for a TDD-based Ralph development loop. Your job is to manage the execution of user stories by spawning Task agents with fresh context for each story.

## Your Task

1. Read `ralph/prd.json` to get the user stories
2. Read `ralph/progress.txt` for context from previous iterations
3. Ensure you're on the correct branch from PRD `branchName`
4. Find the **highest priority** story where `passes: false`
5. Spawn a Task agent to implement that story (using the AGENT_PROMPT)
6. Wait for the agent to complete
7. Update `ralph/prd.json` to set `passes: true` for the completed story
8. Append the agent's learnings to `ralph/progress.txt`
9. Repeat until all stories pass

## Spawning Task Agents

### Before Spawning: Load AGENTS.md Files

**IMPORTANT**: Before spawning each Task agent, you MUST:

1. **Find all AGENTS.md files** in the project:
   ```bash
   find . -name "AGENTS.md" -type f 2>/dev/null
   ```

2. **Read each AGENTS.md file** and collect their contents

3. **Include them in the agent prompt** under the `## Codebase Knowledge` section (see template below)

This ensures agents have access to accumulated project knowledge without relying on them to manually discover these files.

### Agent Prompt Template

For each story, spawn a Task agent using the `general-purpose` subagent type with this prompt structure:

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

## Codebase Knowledge (from AGENTS.md files)

{For each AGENTS.md file found, include:}
### {relative/path/to/AGENTS.md}
{contents of that AGENTS.md file}

{If no AGENTS.md files found, include:}
No AGENTS.md files found yet. Create them in directories where you discover reusable patterns.

## TDD Flow

1. **Orientation**:
   - Read `ralph/progress.txt` (especially Codebase Patterns section)
   - Check `git log --oneline -10` for recent changes
   - The AGENTS.md contents are included above - no need to search for them

2. **Regression Check (Before Starting)**:
   - Run: {prd.testCommand}
   - ALL tests must pass before you start
   - If tests fail, STOP and report - codebase is broken

3. **Write Test First (Red)**:
   - Create/update test file for this story's acceptance criteria
   - Test type: {story.testType}
   - Run the test - it should FAIL (red phase)

4. **Implement (Green)**:
   - Write the minimal code to make the test pass
   - Follow existing patterns in the codebase
   - Keep changes focused and minimal

5. **Run All Tests (Verify Green)**:
   - Run: {prd.testCommand}
   - Your new test must pass
   - All existing tests must still pass

6. **Browser Verification** (if requiresBrowser: true):
   - Use Puppeteer MCP tools to verify UI changes
   - Navigate to the relevant page
   - Verify the acceptance criteria visually

7. **Build Check**:
   - Run: {prd.buildCommand}
   - Must pass with no errors

8. **Update AGENTS.md** (if you discovered reusable patterns):
   - Add learnings to AGENTS.md in the directories you modified
   - Only add genuinely reusable knowledge

9. **Commit**:
   - Stage all changes (code + tests + AGENTS.md)
   - Commit with message: feat: {story.id} - {story.title}

10. **Update Progress**:
   - Append to ralph/progress.txt with learnings
   - Add reusable patterns to Codebase Patterns section

11. **Report Results**:
   Return a JSON object:
   {
     "success": true/false,
     "storyId": "{story.id}",
     "filesChanged": ["list of files"],
     "testsAdded": ["list of test files"],
     "learnings": "What you learned that future iterations should know"
   }

## Important Rules

- Work on ONLY this one story
- Run regression check BEFORE starting implementation
- If baseline tests fail, STOP and report - don't start on broken code
- If tests fail after implementation, fix your code or report failure
- If you cannot complete the story, report failure with reason
- Follow existing code patterns in the codebase
- Keep changes minimal and focused
- **Clean Exit**: If context is getting long, commit work so far and report partial progress
```

## Updating prd.json

After a successful agent completion:
1. Parse the agent's result
2. If `success: true`, update the story's `passes` to `true`
3. Add any notes to the story's `notes` field

## Updating progress.txt

**Note:** The agent updates progress.txt as part of its workflow (step 10).
The orchestrator should:
1. Verify the agent added an entry
2. If agent failed before updating, add a failure note:
   ```
   ## [Date/Time] - {story.id} (FAILED)
   - Reason: {from agent failure report}
   - Failing tests: {if applicable}
   ---
   ```
3. If agent discovered patterns, verify they were added to Codebase Patterns section

## Stop Condition

After each iteration, check if ALL stories have `passes: true`.

If ALL stories are complete, output:
```
<promise>COMPLETE</promise>
```

If there are still stories with `passes: false`, continue to the next iteration.

## Error Handling

### If agent reports `success: false` (implementation failed):
1. **Do NOT set `passes: true`** - leave it as `false`
2. Log the failure reason to progress.txt (with details for next agent)
3. Add the error to the story's `notes` field
4. **Increment `failureCount`** in prd.json for this story
5. **Current agent terminates** (does NOT retry itself)

**Retry happens via NEW agent (fresh context)**:
- Next iteration spawns a NEW Task agent
- New agent reads failure notes from progress.txt and story.notes
- New agent tries a different approach based on learnings
- This prevents context window exhaustion

**Blocking logic** (check `failureCount` field in prd.json):
- If `failureCount < 3`: Spawn new agent for same story
- If `failureCount >= 3`: Add `"BLOCKED"` to notes, move to next story with `passes: false`

### If regression check fails BEFORE implementation:
This means a **previous story** broke something. Handle as follows:

1. **Current story stays `passes: false`** - it never started
2. **Identify the culprit**: Check which tests failed and which story likely caused it
3. **Mark the culprit story as `passes: false`** again (revert its completion)
4. **Log to progress.txt**: "Regression detected: [test names] failed, likely caused by [story ID]"
5. **Next iteration**: Will retry the culprit story, which must now also fix the regression

Example:
```
Agent starting US-003, regression check finds 2 failing tests.
Tests are related to email validation (US-001's feature).
→ Mark US-001 as passes: false
→ Add note: "Regression: email tests failing, needs fix"
→ Next iteration picks up US-001 again
```

## Example Orchestration Loop

```
1. Read prd.json → Found US-001 (passes: false)
2. Read progress.txt → No previous patterns yet
3. Spawn Task agent for US-001
4. Agent returns: { success: true, filesChanged: [...], learnings: "..." }
5. Update prd.json: US-001.passes = true
6. Append to progress.txt
7. Read prd.json → Found US-002 (passes: false)
8. Spawn Task agent for US-002
... repeat ...
N. All stories pass → output <promise>COMPLETE</promise>
```
