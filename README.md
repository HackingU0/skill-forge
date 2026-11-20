# skill-forge

An agent skill that turns a one-line idea into a **tested, installed** skill, using a second AI
provider to review and test it.

```
Preflight → Grill → BRIEF ─[you approve]─▶ Draft ─▶ Review ∥ Blind trigger test ─▶ Dry run ─▶ Install
                                             ▲        (other provider)   │
                                             └──── fix, ≤ 2 rounds ──────┘
```

- **Grill**: a strawman first, then at most four sharp questions per round, until answers stop
  changing the design. The BRIEF is the only point where you sign off.
- **Cross-provider review**: Claude drafts, Codex reviews (or the other way round), so the reviewer
  has different blind spots. The reviewer gets the draft, never the author's reasoning.
- **Blind trigger test**: a fresh agent sees only the names and descriptions of the new skill and
  its closest installed neighbours, plus unlabelled should/shouldn't-trigger prompts (with
  held-back paraphrases for the final round). It catches description collisions, the most common
  way a new skill fails.
- **Dry run**: a fresh agent with only the finished skill does a realistic task in a scratch folder.
- **Install what passed**: any later edit re-runs the checks it affects. Skills are symlinked into
  `~/.agents/skills` (Codex and others) and `~/.claude/skills` (Claude Code), with a collision check.

It also carries a template for the [steal-skills](https://github.com/HackingU0/steal-skills) family.

## Install

```sh
npx skills add HackingU0/skill-forge
```

Then: "write a skill for X", "turn this workflow into a skill", "做一个 skill".

## Requirements

Built for [T3 Code](https://github.com/pingdotgg/t3code)'s multi-agent orchestration
(`delegate_task` across providers, e.g. Claude ↔ Codex). Without a second provider it stops and
says so rather than reviewing its own work.

## License

MIT
