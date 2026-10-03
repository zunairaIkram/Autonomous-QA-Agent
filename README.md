# Autonomous QA Exploration Agent

**Status: in development (V1 scope).** This README describes the current, intentionally-scoped-down build target, plus where the project could go next. See "Roadmap" below for the honest difference between what's built, what's planned, and what's deliberately out of scope.

## The idea

Frontend bugs, broken buttons, dead links, silent console errors, are tedious to find by hand and expensive to catch late. This project is an agent that autonomously explores a website the way a QA tester would: clicking around, following links, and flagging what's actually broken, without a human driving the browser.

## Scope of this version (V1)

**In scope:**
- Input: a single URL (staging or public site)
- The agent explores autonomously — decides what to click or navigate next, within a capped number of actions
- Captures: console errors, failed network requests (4xx/5xx), dead clicks (an action that produced no visible change), broken links
- Output: a structured bug report with what was clicked, what was expected vs. observed, and a screenshot per issue

**Explicitly out of scope for this version** (see Roadmap):
- No GitHub repo or local codebase input
- No automatic code fixing
- No writing back to any repository

This scope was chosen deliberately. Autonomous bug-fixing against an arbitrary, unfamiliar codebase is a genuinely hard, still largely unsolved problem even for frontier models — this version focuses on the part that's achievable and still genuinely useful on its own: **finding and clearly reporting** real issues.

## How it works

```
Load target URL
      │
      ▼
┌─────────────────────────┐
│  PERCEIVE                 │   Extract clickable elements on the
│  (indexed element list)    │   current page into a short, numbered
└─────────────┬───────────────┘  list the model can act on.
              ▼
┌─────────────────────────┐
│  DECIDE                    │   Model chooses: click element N,
│  (model picks next action)  │   navigate to a URL, or finish.
└─────────────┬───────────────┘
              ▼
┌─────────────────────────┐
│  ACT                        │   Executes the chosen action via
│  (click / navigate)          │   Playwright.
└─────────────┬───────────────┘
              ▼
┌─────────────────────────┐
│  OBSERVE                    │   Captures console errors, failed
│  (errors, response codes,    │   requests, and whether the page
│   visible change check)       │   actually changed.
└─────────────┬───────────────┘
              │
     loop until max actions
     or model signals "done"
              ▼
      Compile bug report
```

Same Perceive → Decide → Act → Observe loop structure as a typical ReAct agent — applied to browsing instead of code generation.

## Tech stack

- **Playwright (Python, async)** — headless browser control, console/network event capture
- **OpenAI API** — decides exploration actions from the current page state
- *(Planned)* MCP tool boundary around the browser actions, consistent with the companion code-agent project

## Current limitations

- Element detection is based on a simple DOM query (`button`, `a`, `input`) — doesn't yet handle complex dynamic/JS-rendered content reliably.
- "Dead click" detection is based on whether visible page state changed — not a perfect signal, can have false positives on intentionally static UI.
- No persistence between runs yet — each exploration starts fresh.

## Roadmap

- **V1 (this version):** URL-only exploration and bug reporting.
- **V2 (planned):** Auto-generate a Playwright regression test script that reproduces each found bug, so issues become immediately actionable, testable artifacts.
- **V3 (future work, not currently planned for near-term build):** For a small, known codebase (not an arbitrary client repo), propose a fix as a diff and open it as a pull request for human review — never an automatic push. Deliberately deferred: reliably fixing bugs in an unfamiliar, arbitrary codebase is a hard, open problem, and auto-pushing code without human review is a real trust/safety issue regardless of difficulty.

## AI tools used

Built with the help of Claude for architecture planning and implementation guidance.