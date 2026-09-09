# Skill-Reviewer

A Claude Code skill that reviews **other skills**.

It reads a `SKILL.md` and everything it references, then reports what is
actually wrong with it: specification compliance, use-case coverage,
triggering, instruction quality, context efficiency, resources, permissions,
security, portability, and model fit.

It is **static and read-only**. It does not run the skill under test, and it
does not claim to have. What that buys — and what it costs — is written down in
[POLICE.md](POLICE.md), the standard it judges by.

---

## Install from the command line

The skill lives inside the project that uses it, in `.claude/skills/`:

```bash
# from the root of your project
mkdir -p .claude/skills
git clone https://github.com/JoaoPeixoto72/Skill-Reviewer.git .claude/skills/skill-reviewer

# it belongs to the project now: without its own .git it goes into the
# project's version control like any other file
rm -rf .claude/skills/skill-reviewer/.git
```

With the GitHub CLI:

```bash
gh repo clone JoaoPeixoto72/Skill-Reviewer .claude/skills/skill-reviewer
rm -rf .claude/skills/skill-reviewer/.git
```

> Removing `.git` is not optional housekeeping. Leave it and you have a
> repository inside a repository: the outer project ignores the folder, and the
> skill quietly vanishes from version control.

To install it for every project on the machine instead, clone into
`~/.claude/skills/skill-reviewer` — reviewing skills is not project-specific
work, so a global install is reasonable here.

The folder name must be `skill-reviewer`, lowercase. The Agent Skills format
requires the directory name to match the `name:` field, and this skill will
tell you so if you get it wrong somewhere else.

## Install from inside Claude

Claude Code discovers anything under `.claude/skills/`. After cloning, **start
a new conversation** — the skill list is read at startup.

Or just ask:

> install the skill-reviewer from `github.com/JoaoPeixoto72/Skill-Reviewer`
> into `.claude/skills/skill-reviewer` and drop the `.git`

## Use it

```text
/skill-reviewer .claude/skills/my-skill
```

Or describe the job and let it fire:

> review my-skill before I publish it
>
> why does this skill trigger on the wrong requests?
>
> check this skill still holds up after moving it to Opus 5

### Options

| Flag | Values | Default |
|---|---|---|
| `--target` | `claude-code`, `portable`, `claude-upload` | `claude-code` |
| `--model` | `opus-5`, `fable-5.1`, `gpt-6-astra`, `generic` | `opus-5` |
| `--depth` | `quick`, `standard`, `deep` | `standard` |

`--target portable` applies the Agent Skills specification strictly — name
length and charset, description limits, the permitted frontmatter fields.
`--target claude-upload` adds the Claude.ai and API packaging rules.

`--depth quick` reports only Blockers and Majors; `deep` adds minor
consistency, maintenance and efficiency findings.

### Model profiles

A profile asks whether the skill accounts for how the model that will run it
behaves. Exactly one applies per review, and the report names which — the same
instruction can be a defect under one and a correction under another: Opus 5
looks for unlimited delegation, GPT-6 Astra looks for *under*-delegation.

They live in [`references/model-profiles.md`](references/model-profiles.md),
one section each, with the source cited. That is the part of this skill with a
shelf life: when a model ships, adding or replacing a profile is one section in
one file, and nothing else moves. Changing the default is one line in
`## Defaults`.

### One thing to know before you invoke it

The skill declares `model: opus` and `effort: high`. In Claude Code a `model`
override **applies for the rest of the turn**, not just for the review — so the
turn you review in stays on Opus afterwards. That is deliberate: the failure
mode of a cheap review is not a worse report, it is a wrong verdict someone
acts on, or false positives that teach you to stop reading it. But it is your
turn and your budget, so it should not be a surprise.

### What a report looks like

[`references/example-report.md`](references/example-report.md) is one finished
review at `standard` depth — a real-shaped skill with real-shaped defects, so
you can see the density of evidence expected before you run it on your own
work.

## What you get back

A report ordered by severity, with one verdict:

```text
Verdict:  Ready | Ready with minor changes | Needs revision |
          Not ready | Unable to assess
```

Every finding cites `file:line`, names the requirement it violates or the risk
it runs, states the concrete consequence, and proposes the smallest correction
that resolves it. Findings that cannot do all four are downgraded or dropped
before the report is written — see [POLICE.md](POLICE.md) §4.

The report also lists what the review **could not** establish. A skill that
looks correct on paper can still trigger on the wrong requests or fail at
runtime, and saying so is part of the output rather than a caveat buried at the
end.

## What it will not do

- It will not run the skill, its scripts, or its assets.
- It will not benchmark, or claim an improvement over a baseline. Behavioural
  evaluation belongs to `skill-creator`.
- It will not change your files unless you ask it to apply the corrections.
- It will not help you improve a skill that steals credentials, exfiltrates
  data, or ships malware. It reports the evidence and stops.

## Licence

MIT — see [LICENSE](LICENSE).
