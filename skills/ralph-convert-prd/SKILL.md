---
name: ralph-convert-prd
description: Convert a markdown PRD to prd.json format for Ralph TDD workflow. Use when you have an existing PRD document to convert.
---

# Convert PRD to JSON

Convert an existing markdown PRD document into the prd.json format required by Ralph.

## Usage

The user should provide the path to their markdown PRD file.

## Process

### Step 1: Read the PRD

Read the provided markdown PRD file and extract:
- Project name
- Feature description
- User stories or requirements
- Acceptance criteria

### Step 2: Gather Missing Info

Ask for any missing required fields:
- `testCommand`: Command to run tests
- `buildCommand`: Command to build/typecheck
- `devServerUrl`: URL for browser testing (if UI-related)
- `branchName`: Git branch name (suggest: `ralph/feature-name`)

### Step 3: Extract Stories

Parse the PRD to identify user stories. Look for:
- Numbered requirements
- User story format ("As a... I want... so that...")
- Feature bullet points
- Acceptance criteria sections

### Step 4: Validate Story Size

Each story must be completable in one context window.

**If a story is too big**, split it:
- "Build user dashboard" → "Create dashboard layout", "Add user stats widget", "Add activity feed"
- "Add authentication" → "Add login form", "Add session management", "Add logout"

### Step 5: Assign Properties

For each story, determine:

**testType**:
- `unit`: Pure functions, utilities, validators
- `integration`: API endpoints, database operations, multi-component
- `e2e`: User flows, UI interactions
- `none`: Config, documentation, non-code changes

**requiresBrowser**:
- `true`: UI changes that need visual verification
- `false`: Backend, API, utility functions

**priority**: Order by dependencies (prerequisites get lower numbers)

### Step 6: Generate prd.json

Create the JSON structure:
```json
{
  "project": "extracted-project-name",
  "branchName": "ralph/feature-name",
  "description": "Feature description from PRD",
  "testCommand": "npm test",
  "buildCommand": "npm run build",
  "devServerUrl": "http://localhost:3000",
  "userStories": [
    {
      "id": "US-001",
      "title": "Story title",
      "description": "As a..., I want..., so that...",
      "acceptanceCriteria": ["criterion 1", "criterion 2"],
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

### Step 7: Write Output

Write to `prd.json` in the project root.

### Step 8: Summary

Output a summary:
- Number of stories created
- Estimated execution order
- Any stories that may need further splitting
- Ready to run with `/ralph`

## Validation Checklist

Before writing prd.json:
- [ ] All required fields present
- [ ] Stories ordered by dependency
- [ ] Each story is small enough (one context window)
- [ ] Acceptance criteria are specific and testable
- [ ] testType and requiresBrowser correctly assigned
- [ ] No duplicate story IDs
