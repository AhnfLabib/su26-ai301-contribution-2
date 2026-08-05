# Contribution #2: Auto-run / run AI suggested code before accept

**Contribution Number:** 2  
**Student:** Ahnaf Labib  
**Issue:** https://github.com/marimo-team/marimo/issues/5196  
**Status:** Phase IV In Progress (draft PR open; demo video recorded for maintainers)

---

## Why I Chose This Issue

I chose [#5196](https://github.com/marimo-team/marimo/issues/5196) because it sits at the intersection of AI UX and notebook reactivity—exactly where marimo differentiates itself. The workflow friction (accept → run → maybe reopen AI and retype the prompt) is easy to feel as a user and maps cleanly onto a frontend TypeScript change, which matches the issue’s `good first issue (typescript)` label and my learning goals for Capstone.

I also wanted practice contributing under a maintainership culture that requires discussion for UX changes (`needs discussion`), so I scoped to the safer interpretation the author and maintainers converged on: **manual** run-before-accept (not auto-execute on generation), while leaving Accept-and-run (already shipping) alone.

---

## Understanding the Issue

### Problem Description

When using AI completion on a cell, the user must Accept the suggestion before they can run the cell and see output. For exploratory notebook work (especially plots), output is often the real acceptance criterion. Rejecting and iterating forces reopening AI completion and re-entering the prompt.

### Expected Behavior

After a suggestion appears, the user can **Run** the suggested code, see cell output, keep the AI panel/prompt open, then Accept or Reject. Reject restores the pre-session cell code. Accept keeps the suggestion. Suggestions are never executed automatically when they arrive.

### Current Behavior (before this contribution)

- Accept applies code and closes the AI panel; a separate run is needed unless using Accept+▶ (accept-then-run), which already exists in `main`.
- There is no way to execute the **suggestion** while the AI panel stays open for further prompt iteration.

### Affected Components

- `frontend/src/components/editor/ai/ai-completion-editor.tsx`
- `frontend/src/components/editor/ai/completion-handlers.tsx`
- `frontend/src/components/editor/ai/completion-preview.ts` (new)
- Cell editor wiring already exposes `onChange` / `runCell` / `acceptChange` / `declineChange`

---

## Reproduction Process

### Environment Setup

- Cloned fork: `https://github.com/AhnfLabib/marimo.git` → `/Users/ahnaflabib/marimo`
- Branch: `issue-5196-ai-run-before-accept` from `main`
- Frontend: Node 22+, `pnpm` (via shim/`npx pnpm@10`); Python via `uv sync --group dev`
- Local demo run: Vite on `:3000` + `marimo edit` on `:2718` with notebook `/tmp/marimo-ai-demo.py`
- Followed `CONTRIBUTING.md` / `AGENTS.md` (maintainer approval culture; draft PR; agent disclosure; CLA signed on PR)

### Steps to Reproduce (pre-fix behavior on `main`)

1. Configure AI completion and open a notebook with `marimo edit`.
2. Focus a cell → open AI completion → enter a prompt that changes code (e.g. plot).
3. When the suggestion appears, note Accept / Accept+▶ / Reject only (no Run-before-accept).
4. Accept → then run the cell to see output.
5. If wrong: reopen AI completion; previous prompt is gone → re-enter prompt.

### Reproduction Evidence

- **Working branch:** `issue-5196-ai-run-before-accept` on fork `AhnfLabib/marimo`
- **My findings:** Accept+▶ already chains accept+run (see `AcceptCompletionButton` + `runCell`). The gap that remains for #5196 is **run while panel stays open** with clean reject restore—the author’s clarified ask and the safer alternative to auto-run.

---

## Solution Approach

### Analysis

Running a cell always executes whatever code is in the editor. Preview therefore must temporarily apply the suggestion, run, and keep AI UI open, while remembering a session baseline for Reject / resubmit.

### Proposed Solution

Add a **Run** button that applies the suggestion via existing `onChange`, calls `runCell()`, and leaves the panel open. Track `hasPreviewed` + session baseline helpers. Reject/close restores baseline; Enter and Send both restore before regenerating. Aligned with upstream: Reject keeps the prompt open for refinement; X closes the panel.

### Implementation Plan (UMPIRE)

**Understand:** Users need to judge AI suggestions by output without accepting first; auto-run is unsafe; Accept+▶ already covers accept-then-run.

**Match:** Staged AI cells already restore previous code on reject (`StagedAICell.tsx`); Accept+▶ pattern lives in `completion-handlers.tsx`.

**Plan:**
1. Pure helpers for baseline / restore / merge original (`completion-preview.ts`) + tests
2. `RunCompletionButton` + RTL tests
3. Wire into `AiCompletionEditor` (preview run, restore on decline/close/resubmit, merge baseline)

**Implement:** Branch `issue-5196-ai-run-before-accept` (commits below)

**Review:** Self-review + task reviews against plan; draft PR per CONTRIBUTING + agent rules; issue still `needs discussion` so PR uses `Related to #5196` not `Closes`.

**Evaluate:** Unit tests for helpers and Run button; manual video demo of Run → Accept, Accept+▶, and Reject.

---

## Testing Strategy

### Unit Tests

- [x] `captureSessionBaseline` keeps first baseline
- [x] `codeToRestoreOnReject` / `shouldRestoreBeforeResubmit` / `originalCodeForMerge` branches
- [x] `RunCompletionButton` calls `onRun` and disables while loading

### Integration Tests

- [ ] E2E Playwright for AI completion (optional; not added—requires live AI config)

### Manual Testing

- [x] Open AI completion (**Refactor with AI** / `⌘⇧E`) → generate → **Run** → output updates, panel stays open
- [x] **Reject** after suggestion / preview discards suggestion (upstream keeps prompt open)
- [x] **Accept** after Run keeps suggestion
- [x] Accept+▶ still accept-then-runs
- [x] Recorded demo video covering: Run then Accept, Accept+▶, and Reject (for maintainer review)
- [ ] After Run, edit prompt and Send/Enter regenerates from restored baseline (not re-checked in latest recording session)

**Local demo notes (Aug 5, 2026):**
- Gemini free quota exhausted mid-setup → switched to **Ollama**
- Catalog model `gemma4:31b` 404’d (not installed); used custom model `ollama/llama3.2:latest`
- Prompt `Plot x against y` produced matplotlib suggestion; **Run** showed plot while merge UI stayed open

---

## Implementation Notes

### Progress

- Implemented run-before-accept on branch `issue-5196-ai-run-before-accept`
- Fixed Send path to share restore-before-resubmit with Enter (`handlePromptSubmit`)
- Fixed restore when AI panel closes via hotkey / switch cell (`hasPreviewedRef` + `[enabled]` effect)
- Merged upstream `main` and resolved conflict: Reject keeps prompt open; X uses `handleCloseCompletion` with baseline restore
- Signed CLA on PR; responded to maintainer request for a demo video

### Code Changes

- **Files modified/added:**
  - `frontend/src/components/editor/ai/completion-preview.ts`
  - `frontend/src/components/editor/ai/__tests__/completion-preview.test.ts`
  - `frontend/src/components/editor/ai/completion-handlers.tsx`
  - `frontend/src/components/editor/ai/__tests__/completion-handlers.test.tsx`
  - `frontend/src/components/editor/ai/ai-completion-editor.tsx`
  - `.gitignore` (local `.worktrees/`)
- **Key commits:**
  - `56abb18a8` feat(ai): add helpers for run-before-accept preview state
  - `08c272ad4` feat(ai): add Run button for previewing AI completions
  - `d66a65d71` feat(ai): run suggested code before accepting completion
  - `ff36ab002` fix(ai): restore baseline on Send after preview run
  - `8d2a3202b` fix(ai): restore preview baseline when AI panel closes
  - `493631fb3` fix(ai): keep prompt open on reject after preview run
  - `5c24254a0` merge: resolve ai-completion-editor conflict with upstream main
- **Approach decisions:** Manual Run only (not auto-execute); freeze session baseline for merge Original after preview; do not change Accept+▶; Reject ≠ close (match upstream).

---

## Pull Request

**PR Link:** https://github.com/marimo-team/marimo/pull/10458

**PR Description:** Combines CodePath `sample-pr.md` sections with upstream PR template; agent disclosure; draft; `Related to #5196`.

**Maintainer Feedback:**
- **Light2Dark** asked for a video demo (`@AhnfLabib could you share a video?`)
- Recorded local demo covering Run → Accept, Accept+▶, and Reject; uploading / posting on the PR

**Status:** Draft PR open — demo video ready for maintainer review; awaiting further feedback (`needs discussion` still on issue)

---

## Learnings & Reflections

### Technical Skills Gained

AI completion UI state (merge editor vs live cell editor), preview/restore patterns, Vitest + RTL in marimo frontend, merging with upstream UX changes, local marimo + Vite + Ollama demo setup.

### Challenges Overcome

Accept+▶ already existed—needed to re-scope from “Accept and Run” to run-before-accept. Send button initially bypassed restore; fixed via shared `handlePromptSubmit`. Hotkey-close left previewed code; fixed via `[enabled]` safety net. Merge conflict with upstream Reject-keeps-prompt-open. Gemini quota → Ollama; wrong catalog model name vs locally installed tags.

### What I'd Do Differently Next Time

Confirm shipped behavior on `main` before claiming a scoped slice; ask maintainers to flip `needs discussion` → `ready` earlier; set a known-good local Ollama model (`ollama/llama3.2:latest`) before relying on provider catalogs.

---

## Resources Used

- https://github.com/marimo-team/marimo/issues/5196
- https://github.com/marimo-team/marimo/discussions/5037
- https://github.com/marimo-team/marimo/blob/main/CONTRIBUTING.md
- https://github.com/marimo-team/marimo/pull/10458
- Local design/plan under `docs/superpowers/` (not committed)
- Ollama local models for demo after Gemini free-tier limit
