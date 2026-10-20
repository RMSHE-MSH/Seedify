# Agent System Prompt Optimizer SKILL

A [Claude Code skill](https://docs.anthropic.com/en/docs/claude-code/skills) that automates agent system prompt engineering using evals as quality gates. It runs evals, diagnoses failures, makes targeted prompt edits, and validates improvements — in a closed loop.

The human provides the quality gates (test cases + scorers). The machine iterates toward them.

```
/optimize-prompt <agent> <max-runs> [baseline.json] ["focus area"]
```

## How it works

```
Run baseline eval → Diagnose failures → Edit prompt → Run targeted eval → Keep or revert → Repeat
```

The skill simulates how a human prompt engineer iterates — but with a budget, revert-on-regression discipline, and structured cost tiers so it doesn't burn through eval runs unnecessarily.

### Phases

1. **Setup** — Create a git branch, read the prompt, load the failure taxonomy
2. **Baseline** — Run eval (or use a provided result file) to establish the starting point
3. **Diagnose** — Classify failures by pattern, map to prompt sections, rank by impact
4. **Optimize** — Iterative loop: edit → spot check → keep or revert. Budget-controlled with cost tiers
5. **Validate** — Final eval at high confidence (full suite, 5 samples)
6. **Commit** — Present diff and report. User chooses: commit, merge, or discard

### Cost tiers

Each eval run invokes your agent end-to-end for every test case. The skill minimizes cost by starting cheap:

| Tier | What it runs | When |
|------|-------------|------|
| **Spot check** | 3-5 test cases, 1 sample | First check after an edit |
| **Targeted** | All failing TCs, 1 sample | Spot check passed |
| **Regression** | Full suite, 1 sample | After 2-3 successful edits |
| **Final** | Full suite, 5 samples | Validation only |

## Prerequisites

Before using this skill, you need:

1. **An eval runner** that outputs structured JSON with:
   - Test case IDs
   - Pass/fail per attempt
   - Scorer reasons explaining *why* something failed (not just "pass" or "fail")
   - Pass@k aggregation at multiple k values

2. **Test cases** that test specific behaviors, not just "does the output look right"

3. **A structured prompt** with identifiable sections, so the agent can make targeted edits without breaking unrelated behavior

## Installation

### 1. Copy the skill into your project

```bash
cp -r skill/ <your-project>/.claude/skills/optimize-prompt/
```

### 2. Customize the configuration

Edit `SKILL.md` and replace the placeholder values:

| Placeholder | Replace with | Example |
|-------------|-------------|---------|
| `YOUR_EVAL_COMMAND` | Your eval runner command | `npm run eval:planner` |
| `YOUR_PROMPT_PATH` | Path to your prompt file | `src/agents/planner/instructions.ts` |
| `YOUR_EVAL_OUTPUT_PATTERN` | Glob pattern for eval result files | `eval-results-*.json` |

### 3. Adapt the scripts

The `scripts/` directory contains two reference implementations that parse a specific JSON format. You **must** adapt these to match your eval output:

- **`parse-eval.sh`** — Reads a single eval result JSON and outputs structured text. Must output:
  - Pass@k summary
  - Per-test-case results (PASS/FLAKY/FAIL)
  - Failure details with scorer reasons
  - Comma-separated list of failing test case IDs (for `--filter`)

- **`compare-evals.sh`** — Compares two eval results (baseline vs current). Must output:
  - Pass@k deltas
  - Per-test-case changes (IMPROVED/REGRESSED/unchanged)
  - Summary counts
  - Failure details for still-failing cases

The reference implementations use `jq` and assume a specific JSON structure.

### 4. Write your methodology

Edit `methodology.md` to define:

- Your failure taxonomy (what patterns do your scorers produce?)
- The mapping from failure patterns to prompt sections
- Optimization strategies ranked by impact

### 5. Set up allowed-tools

The `allowed-tools` in `SKILL.md` controls which operations auto-approve during the optimization loop. Adjust the patterns to match your project:

```yaml
allowed-tools: Read, Edit, Grep, Glob,
  Bash(YOUR_EVAL_COMMAND*),
  Bash(bash .claude/skills/optimize-prompt/scripts/*),
  Bash(git checkout -b prompt-opt/*),
  Bash(git checkout -- YOUR_PROMPT_PATH),
  Bash(git diff*), Bash(git status*), Bash(git log*)
```

Safe operations (evals, scripts, branches, reverts) should be auto-approved. Dangerous operations (commit, merge, delete branch) should require approval.

## Usage

```bash
# Basic: optimize with 6 eval runs
/optimize-prompt planner 6

# With existing baseline (skip first eval, save money)
/optimize-prompt planner 5 eval-results-2026-02-26.json

# With focus area
/optimize-prompt planner 6 "tool trajectory failures"

# With baseline AND focus
/optimize-prompt planner 5 eval-results-2026-02-26.json "reduce prompt length"
```

## Rules the skill enforces

1. Never skip the baseline — you need a before measurement
2. Minimal edits only — change the fewest lines to fix the identified pattern
3. One hypothesis per iteration — don't bundle changes
4. No commits until the user approves
5. Revert failures immediately — don't stack changes on a failed hypothesis
6. Use the provided scripts — never write ad-hoc code to parse eval JSON
7. Never edit test cases — the test cases are ground truth
8. Ask before exceeding budget

## Project structure

```
skill/                          # Drop into .claude/skills/optimize-prompt/
├── SKILL.md                    # Main skill definition (customize placeholders)
├── methodology.md              # Failure taxonomy template (fill in your patterns)
└── scripts/
    ├── parse-eval.sh           # Reference implementation (adapt to your JSON)
    └── compare-evals.sh        # Reference implementation (adapt to your JSON)
```

## Learn more

For a deeper walkthrough of the methodology — including real optimization transcripts showing the agent reverting failed hypotheses and navigating tradeoffs — see the accompanying blog post [Closing the loop: automated prompt engineering with evals and Claude Code](https://flow-ai.com/blog/automated-prompt-engineering-cc).
