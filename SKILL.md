---
name: skill-forge
description: >
  Build a new agent skill (SKILL.md) from a one-line idea and ship it tested and installed for both
  Claude Code and Codex, using T3 multi-agent orchestration: grill the user into a BRIEF, draft,
  cross-provider review plus a blind trigger test against installed skills, a fresh-agent dry run,
  then install. Use when the user says "write a skill", "make a skill for X", "turn this into a
  skill", "make this workflow reusable for Claude and Codex", "写个 skill", "做一个 skill",
  "把这个做成 skill", "把这套流程沉淀下来装到 agent 里", or wants a new member of the steal-* family.
  Prefer this over create-skill and skill-creator when the skill should be built, tested and
  installed. Not for SKILL.md format questions (create-skill), books/courses/transcripts
  (cangjie-skill), finding an existing skill (find-skills), or stealing a UI or flow right now
  (steal-skill-ui, steal-skill-prototype).
---

# Skill Forge

One idea in, one installed skill out. You (the parent) own the interview, the draft and every
decision; other agents only review, test and dry-run, each from files alone.

```
0 Preflight ─▶ 1 Grill → BRIEF ─[user approves]─▶ 2 Draft ─▶ 3 Review ∥ Trigger test ─▶ 4 Dry run ─▶ 5 Install
                                                     ▲                 │ (other provider)
                                                     └ fix, ≤2 rounds ─┘
```

Editing, reviewing or re-installing an *existing* skill is not this pipeline; do just the part
asked for (e.g. only step 3 for "check when my skill triggers").

## Layout

`<repo>` is the git repo where the user keeps their own skills. Ask once if you don't know it (and
remember the answer), e.g. `~/my-skills`. New skills are symlinked from there into place:

```
<repo>/<name>/                 # symlinked into ~/.agents/skills (Codex and others) and ~/.claude/skills
  SKILL.md
  .forge/                      # process files, kept so tests can be re-run after edits
    BRIEF.md  REVIEW.md  trigger-test-r<n>.md  trigger-result-r<n>.md  dryrun.md
```

Add `.forge/` to the repo's `.gitignore` unless the user wants the test history published; either
way never put real customer data, secrets or private paths in it.

Bodies default to **English**; a BRIEF may override it. Trigger phrases in `description` follow
what users will actually type, in every language they use.

## 0. Preflight (before asking anything)

- `<name>` is a lowercase-hyphen slug. If `<repo>/<name>` exists and its `.forge/BRIEF.md`
  isn't this run's, it's someone's work: pick another name or ask.
- `orchestrator_capabilities` once: confirm a cross-provider reviewer is available now, so the user
  doesn't sit through an interview for a pipeline that can't run.
- Find neighbours and duplicates. There are 200+ skills, so grep, don't dump. Use 3–6 keywords from
  the idea in both languages, plus the bare name list:
  ```sh
  find -L ~/.agents/skills ~/.claude/skills ~/.codex/skills <repo> -maxdepth 3 -name SKILL.md 2>/dev/null |
    while IFS= read -r f; do realpath "$f"; done | sort -u |
    while IFS= read -r f; do awk '/^---$/{n++; next} n==1' "$f" | tr -s '\n ' ' ' | grep -iE 'kw1|kw2|kw3' && echo "  ($f)"; done
  ```
  If the harness shows an available-skills list, check it too; it is what routing really sees.
- If a skill already does the job, say so and offer to extend it instead.
- Keep the 3–8 closest competitors (zero is fine). They are the trigger-test field.

## 1. Grill → BRIEF (the only human gate)

Interview until answers stop changing what you would write.

- Show a strawman first, then grill against it. Don't ask what you can decide or read.
- ≤4 questions per round, each self-contained, concrete options, recommended first. Claude Code:
  AskUserQuestion. Codex Plan mode: its question tool (≤3). Codex Default mode or no tool: ask in
  plain text and end the turn.
- Cover: the job in one sentence, trigger wording, near misses and who owns them, inputs, outputs,
  steps, red lines, template, and one realistic dry-run task.

Write `.forge/BRIEF.md`:

```markdown
# <name>
Job: <one sentence>
Approved: no | yes (<date>, user's words)
Template: generic | steal        Language: English | ...
Should trigger (≥6, varied, incl. ones that never say "skill" or the name): ...
Should NOT trigger (≥4 near misses → owner skill, or `none`): ...
Holdout (2–3 extra paraphrases, used only in the last trigger round): ...
Inputs: ...   Outputs: ...
Steps: ...
Red lines: ...
Dry-run task: <concrete, fake data, outputs inside scratch, no outward-facing actions>
Competitors: ...
```

In a read-only mode (Claude or Codex plan mode), show the BRIEF in chat and write the file once
edits are allowed. **Approval must be an explicit user reply to this BRIEF**, not silence; it also
covers the install targets in step 5. If BRIEF.md already exists
and is approved (resumed session), continue from where `.forge/` shows you stopped. After approval,
run to the end without asking unless only the user can unblock you.

## 2. Draft

- **`description` is the trigger.** Routing sees only name + description. Say what it does, when,
  real trigger phrases (including one implicit one), and "Not for X (use Y)" only for competitors
  that would actually steal its prompts. Every extra clause costs context in every session.
  Lean slightly pushy: under-triggering is the common failure.
- Explain *why* behind rules instead of stacking MUSTs; a model that knows the reason handles
  cases you didn't list.
- Body under ~300 lines; bulky material goes to `references/*.md` with when-to-read notes. Scripts
  only for deterministic, repeated steps.
- Name concrete output files; end with a check the agent runs on its own output.
- Tool instructions must work in both harnesses: name the Claude and the Codex way, or describe the
  capability ("ask the user", "start a fresh subagent") instead of one harness's tool name.

Templates (both start with frontmatter: `name` = directory name, non-empty `description`):

**generic**
```
# Title
<one paragraph: what this does and the core idea>
## When to use / when not
## Steps (numbered, each with its output)
## Outputs
## Final check
```

**steal** (the steal-skills family: `npx skills add HackingU0/steal-skills`)
```
# Steal Skill · <Domain>
> Good artists copy, great artists steal.
<Copy vs Steal in this domain: copy takes the artifact, steal takes the decision and its why>
## Red lines: what never gets taken (brand, assets, copy, anything that could pass as the original)
## 1. Extract: observe with evidence (open the real thing, measure, record raw tables)
## 2. Abstract: turn observations into rules, each with "why it works"; pick the best 3–5
## 3. Re-apply: map rules onto the user's own problem; deviate on purpose at least once, say why
## Outputs: <DOMAIN>-STOLEN.md (source → observations → rules → mapping → deviations) + artifact
## Final check: side by side with the reference; it must match in quality, never pass as the original
```
Name the steal-* siblings as "Not for" when their prompts overlap.

**Package check** before any delegation and before install: frontmatter parses, `name` matches
the directory (≤64 chars, lowercase-hyphen), description ≤1024 chars, and every bundled file the
body tells the agent to read or run exists. Files the skill *produces* are not checked here.

## 3. Review ∥ Trigger test

Both run in parallel on **another provider** than yours, via `delegate_task` with
`mode: "async"`, `isolation: "shared"` and an explicit `target`:
- You are Claude → `{providerInstanceId: "codex", model: "gpt-6.2-sol", options: {reasoningEffort: "high"}}`
- You are Codex → `{providerInstanceId: "claudeAgent", model: "claude-opus-5-5", options: {effort: "high"}}`

Pick an instance with `canRunCrossProviderChildTask`; a model ID alone can exist under several
drivers. If the pair for your direction isn't available, say so and stop. Don't fall back to your own provider: the
point is a reviewer with different blind spots.

T3 rules:
- A child sees only its `task`, so hand off file paths. Give the reviewer the draft, never your
  reasoning; an anchored reviewer just confirms you.
- Don't poll (`task_status`, sleep, watchers). Follow the `next` field `delegate_task` returns:
  completion wakes you, so do other work or end your turn.
- On wake, check the child completed and its output file is from this round (round-specific
  names make stale results impossible to mistake). A crashed or write-denied child is a tooling
  failure: re-run it with a new task, don't score it as a test failure.
- Every round is a **new** `delegate_task`, never a follow-up to an old child. Children spawn
  nothing. If you abandon the run, `task_cancel` children still running.

**Review brief** (role `review`):
> Review the agent skill at `<abs>/SKILL.md`. It will be loaded by Claude Code and Codex running in
> T3 Code. Find: triggers that are too narrow or collide with common requests; steps an agent
> could misread or skip; missing failure/edge handling; anything that only works in one of the two
> harnesses; rules without a reason; wasted context. Rank most severe first, each with a concrete
> fix. Write them to `<abs>/.forge/REVIEW.md` under `## Round <n>`. Don't edit SKILL.md, don't
> open other `.forge/` files, spawn no agents.

**Trigger test.** Write `.forge/trigger-test-r<n>.md` yourself: name + full description of the new
skill and each competitor, shuffled, then the BRIEF prompts shuffled and numbered with **no**
labels. Every expected owner in the BRIEF must be in the skill list, or the right answer is
impossible. The holdout prompts run once, in the test of the version you will install, even if
round 1 already passed. The answer key stays in BRIEF.md.

**Trigger brief** (role `test`):
> Read only `<abs>/.forge/trigger-test-r<n>.md`. You are an agent that can load one of the listed
> skills. For each numbered prompt give the skill you would load, or `none`, and one line why.
> Judge only from the names and descriptions in that file; ignore your own harness's skills and
> open no other files. Write a table `# | skill | why` to `<abs>/.forge/trigger-result-r<n>.md`.
> Spawn no agents.

**Score** (one row per prompt, or it's a tooling failure): each prompt must route to its expected owner from the BRIEF (the new skill, a named
competitor, or `none`). A miss usually means the description is wrong, but check the expected
label and the competitor's description too. Report it as a curated routing test, not proof of
live routing in both harnesses.

**Fix loop**: apply findings you agree with; log each decision (applied / rejected + why) under
the round in `REVIEW.md`. Re-run with new delegates. **What you install must be what passed**: any
later edit to name or description re-runs the trigger test; a material change to the body or to a bundled script/reference re-runs review, and anything that
changes behaviour after the dry run re-runs the dry run.
Budget is 2 rounds, to cap token cost; if checks still fail, stop and report instead of installing.

## 4. Dry run

Start a fresh same-provider agent **in the foreground** with no inherited conversation (Claude:
`Agent` with `subagent_type: "general-purpose"`, `run_in_background: false`, never a fork; Codex: `spawn_agent` with `fork_turns: "none"`, then wait).
Give it only:

> Read `<abs>/SKILL.md` and follow it to do this task: `<dry-run task>`. Work only in
> `<scratch dir>`, with the fake inputs copied there. Stop before anything outward-facing (posting,
> purchases, logins, installs, writes outside scratch). If the skill would delegate to other
> agents, write the brief you would send instead of sending it. Then write `<scratch dir>/dryrun.md`:
> what you produced, every place the skill was ambiguous or you had to guess, every step you
> skipped or simulated. Spawn no agents.

Copy `dryrun.md` into `.forge/`, and open the produced files yourself: a pass needs the core output
to exist and meet the BRIEF, not just a good self-report. A guess that changed the result, a permission or a required step
is a failure even if the output looks fine; ordinary choices the skill leaves to the agent are not.
A simulated step is reported as simulated, not as passed. Fix, re-run once; if it still fails,
stop and report. Fixes here obey the "install what passed" rule above.

## 5. Install

Preflight both targets first, then link, so a collision never leaves a half install:

```sh
name=my-skill   # the approved slug
src=$(cd -P <repo>/"$name" && pwd) && [ -f "$src/SKILL.md" ] || { echo "STOP: no $name/SKILL.md"; false; } &&
( ok=1
  for d in ~/.agents/skills ~/.claude/skills; do t="$d/$name"
    if [ -e "$d" ] && [ ! -d "$d" ]; then echo "STOP: $d is not a directory"; ok=0
    elif [ -L "$t" ] && [ "$(cd -P "$t" 2>/dev/null && pwd)" != "$src" ]; then echo "STOP: $t links elsewhere"; ok=0
    elif [ -e "$t" ] && [ ! -L "$t" ]; then echo "STOP: $t is not a symlink"; ok=0; fi
  done
  [ $ok = 1 ] || exit 1
  for d in ~/.agents/skills ~/.claude/skills; do mkdir -p "$d" && ln -sfn "$src" "$d/$name" || exit 1; done
  for d in ~/.agents/skills ~/.claude/skills; do [ "$(cd -P "$d/$name" && pwd)" = "$src" ] || exit 1; done
  echo "installed $name" )
```

`~/.agents/skills` is what Codex lists (verified); `~/.claude/skills` serves Claude Code. On a STOP,
ask the user and never delete their files. A new link may only show up in fresh sessions; say so.
Don't commit unless asked.

## Final report

Briefly: skill path, trigger score per round (incl. holdout), what review changed, what the dry
run surfaced or simulated, what stayed unverified, and open items. Link `SKILL.md` and `.forge/`.
