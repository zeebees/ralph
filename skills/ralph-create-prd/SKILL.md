---
name: ralph-create-prd
description: Interactively create a PRD for Ralph TDD workflow. Use when starting a new feature and need to define user stories.
---

# Create PRD for Ralph

You will help create a Product Requirements Document for the Ralph TDD workflow.

## Process

### Step 1: Gather Feature Info

Ask the user:
1. What feature do you want to build?
2. What project/codebase is this for?
3. What is the test command? (e.g., `npm test`, `pytest`)
4. What is the build command? (e.g., `npm run build`, `cargo build`)
5. Is there a dev server URL for UI testing? (e.g., `http://localhost:3000`)

### Step 2: Explore Codebase

Before writing stories:
1. Understand the existing codebase structure
2. Identify patterns and conventions
3. Find similar implementations to reference

### Step 3: Break Down into Stories

Create user stories that are:
- **Small**: Completable in one context window (2-3 sentences to describe)
- **Testable**: Clear acceptance criteria
- **Independent**: Minimal dependencies between stories
- **Ordered**: Lower priority numbers execute first

**Good story size**:
- Add a validation function
- Create an API endpoint
- Add a form field

**Too big (split these)**:
- Build the entire dashboard
- Add authentication
- Refactor the API

### Step 4: Define Each Story

For each story, specify:
```json
{
  "id": "US-001",
  "title": "Short title",
  "description": "As a [user], I want [feature], so that [benefit]",
  "acceptanceCriteria": [
    "Specific, testable criterion 1",
    "Specific, testable criterion 2"
  ],
  "testType": "unit|integration|e2e|none",
  "requiresBrowser": false,
  "priority": 1,
  "passes": false,
  "failureCount": 0,
  "notes": ""
}
```

### Step 5: Generate prd.json

Create `prd.json` with this structure:
```json
{
  "project": "project-name",
  "branchName": "ralph/feature-name",
  "description": "Feature description",
  "testCommand": "npm test",
  "buildCommand": "npm run build",
  "devServerUrl": "http://localhost:3000",
  "userStories": [...]
}
```

### Step 6: Validate

Before finalizing:
- [ ] Each story is small enough for one context window
- [ ] Stories are ordered by dependency (prerequisites first)
- [ ] Acceptance criteria are specific and testable
- [ ] testType matches what kind of test should be written
- [ ] requiresBrowser is true only for UI verification stories

## Test Types

| Type | Use For |
|------|---------|
| `unit` | Single function/class in isolation |
| `integration` | Multiple components, API calls |
| `e2e` | Full user flow in browser |
| `none` | Config changes, documentation |

## Output

Write the generated PRD to `prd.json` in the project root.

After creation, the user can run `/ralph` to start the TDD implementation loop.
