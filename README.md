# Contribution 2: Auto-run code suggested by AI

**Contribution Number:** 2  
**Student:** Ahnaf Labib
**Issue:** https://github.com/marimo-team/marimo/issues/5196  
**Status:** Phase I – In Progress  

***

## Why I Chose This Issue

I chose this issue because it combines AI-assisted coding, notebook workflows, and frontend developer experience, which lines up well with my interests in AI tools and software engineering. The requested feature is small enough to be approachable for an open-source contribution, but meaningful enough to improve how users interact with Marimo’s AI completion flow.

This issue also fits my learning goals because it is labeled `good first issue (typescript)` and appears to involve Marimo’s TypeScript frontend and CodeMirror-based completion system. Working on it would help me better understand how AI-generated suggestions are applied in the UI, how notebook cell execution is triggered, and how product decisions around usability and safety get reflected in open-source code.

***

## Understanding the Issue

### Problem Description

When using Marimo’s AI completion, a user must accept the suggested code before they can run the cell and inspect the output. That means the user has to commit to the generated code before seeing whether it actually produces the desired result, which makes iteration slower and less convenient.

### Expected Behavior

A better workflow would let users accept AI-generated code and immediately run the cell in a single action. Based on the issue discussion, the simplest agreed-upon improvement is an **Accept and Run** button that combines both actions while leaving the more advanced “run before accepting” flow for later discussion.

### Current Behavior

The current flow is: open AI completion, enter a prompt, accept the suggestion, and then manually run the cell. This adds an extra step and breaks the feedback loop because the user cannot see the output until after the suggestion has already been accepted.

### Affected Components

The issue discussion points to the frontend AI completion logic, particularly the CodeMirror completion area and `frontend/src/core/codemirror/completion/Autocompleter.ts`. The likely change will involve the suggestion acceptance UI and the logic that triggers cell execution from the frontend.

***

## Reproduction Process

### Environment Setup

This section will be completed in Phase II after setting up the project locally. The plan is to fork and clone `marimo-team/marimo`, follow the repository setup instructions, start the frontend development environment, and verify that AI completion works in a sample notebook before reproducing the issue.

### Steps to Reproduce

1. Open a Marimo notebook with an editable cell.
2. Trigger AI completion and enter a prompt for code generation.
3. Accept the generated suggestion.
4. Observe that the cell does not automatically run and still requires a separate manual execution step.

### Reproduction Evidence

- **Commit showing reproduction:** To be added in Phase II after local reproduction.
- **Screenshots/logs:** To be added in Phase II.
- **My findings:** Initial review of the issue suggests the current acceptance flow and cell execution flow are separate actions in the frontend.

***

## Solution Approach

### Analysis

The core issue is that accepting an AI suggestion and running the cell are currently treated as separate user actions. As a result, the AI-assisted workflow is slower than it could be, especially for users who care more about the rendered output than about reviewing the generated code first.

### Proposed Solution

The initial solution is to add an **Accept and Run** action to the AI completion UI. This action would apply the suggested code to the cell and immediately trigger execution of that cell, reducing the current four-step flow into a simpler combined action.

### Implementation Plan

Using the UMPIRE framework:

**Understand:**  
The issue asks for a smoother AI completion workflow so users can see results faster after accepting a suggestion.

**Match:**  
The implementation should reuse existing patterns in Marimo for accepting AI suggestions and for running cells, rather than introducing a separate execution path.

**Plan:**
1. Identify where the existing **Accept** action is implemented in the AI completion UI, likely near `Autocompleter.ts`.
2. Add a new **Accept and Run** button in the same UI area.
3. Reuse the existing accept logic, then call the current cell execution mechanism.
4. Verify that the original **Accept** behavior remains unchanged.
5. Add or update tests for the new action.

**Implement:**  
To be completed in later phases with branch and commit links.

**Review:**  
The final change should follow Marimo’s contribution guidelines, maintain existing behavior for users who only want **Accept**, and keep the UI consistent with the rest of the application.

**Evaluate:**  
Success means that clicking **Accept and Run** applies the suggestion and executes the correct cell without requiring a second manual step.

***

## Testing Strategy

### Unit Tests

- [ ] Test case 1: The existing **Accept** action still applies code without auto-running.
- [ ] Test case 2: **Accept and Run** applies the AI suggestion and executes the cell.
- [ ] Test case 3: The correct cell is executed when multiple cells are present.

### Integration Tests

- [ ] Integration scenario 1: AI completion followed by **Accept and Run** in a notebook with visible output.
- [ ] Integration scenario 2: **Accept and Run** behavior when the generated code raises an execution error.

### Manual Testing

Manual testing will be documented in later phases after local setup. Planned checks include basic code generation, visible outputs such as plots or tables, and error-handling behavior after execution.

***

## Implementation Notes

### Week 1 Progress

Selected the issue, reviewed the issue description and comments, and scoped the likely first implementation target to the simpler **Accept and Run** behavior instead of the more complex “run before accepting” flow.

### Week 2 Progress

To be added during implementation.

### Code Changes

- **Files modified:** To be added after implementation begins.
- **Key commits:** To be added after implementation begins.
- **Approach decisions:** To be added after implementation begins.

***

## Pull Request

**PR Link:** To be added after submission.

**PR Description:**  
The PR will add an **Accept and Run** button to Marimo’s AI completion UI so that users can apply an AI-generated suggestion and immediately execute the current cell in one step.

**Maintainer Feedback:**
- [Date]: Feedback to be added once received.
- [Date]: Response to feedback to be added once addressed.

**Status:** Not submitted yet.

***

## Learnings & Reflections

### Technical Skills Gained

This contribution is expected to strengthen understanding of TypeScript frontend development, notebook UI architecture, and how AI-assisted features are integrated into developer tools.

### Challenges Overcome

To be added as the work progresses.

### What I'd Do Differently Next Time

To be added after the contribution is complete.

***

## Resources Used

- Marimo issue #5196: https://github.com/marimo-team/marimo/issues/5196 
- Issue discussion/comments for #5196 
- CodePath AI Open Source Capstone Phase I course page [1]
