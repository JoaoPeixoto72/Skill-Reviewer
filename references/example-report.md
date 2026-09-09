# Worked example

One finished review, at `standard` depth, so the shape of the output is not
left to inference. The skill under review is invented; the defects in it are
ones that occur.

Read this once. It is not part of the review procedure.

The skill being reviewed, `changelog-writer`, has this frontmatter:

```yaml
---
name: changelog-writer
description: Writes changelogs. Use for release notes, changelogs, version summaries, git history, release management, and publishing releases to GitHub.
allowed-tools: Read Grep Glob Bash Write
hooks:
  PostToolUse:
    - matcher: Write
      command: git add -A
---
```

---

# Skill review: `changelog-writer`

**Verdict:** Needs revision
**Target:** Claude Code · **Depth:** Standard
**Model profile:** Opus 5 — the reviewing model, inferred: `changelog-writer`
declares no `model:` of its own. Override with `--model` if it will ship
elsewhere.
**Reviewed:** `SKILL.md` (118 lines), `references/format.md`
**Not inspected:** None
**Assumptions:** The repository is a git repository, as the skill assumes throughout

## Findings

### [Major] The description promises publishing; the body never publishes

- **Type:** Defect
- **Area:** Coverage
- **Evidence:** `SKILL.md:3` lists "publishing releases to GitHub" as a use
  case. The body has four sections — read history, group commits, write the
  entry, format — and none of them creates a release, tags, or pushes.
- **Impact:** The skill fires on "publish the release", produces a changelog
  file, and stops. The user believes the release is out because the skill
  claimed that use case. Nothing says otherwise.
- **Fix:** Remove "and publishing releases to GitHub" from the description, or
  add the branch. Removing is smaller and matches what the skill is for.
- **Confidence:** High

### [Major] A hook is registered that the description never mentions

- **Type:** Defect
- **Area:** Safety
- **Evidence:** `SKILL.md:6-9` registers a `PostToolUse` hook running
  `git add -A` after every `Write`.
- **Impact:** Claude Code keeps registered hooks for the rest of the session,
  not the turn. After this skill runs once, every later `Write` in that session
  stages the entire working tree — including files the user was deliberately
  keeping unstaged. Nothing in the description tells them this will happen.
- **Fix:** Say it in the description, and narrow the matcher to the changelog
  path. A session-scoped side effect has to be visible before the skill is
  invoked, not after.
- **Confidence:** High

### [Minor] `allowed-tools` grants Bash and Write for a workflow that reads

- **Type:** Concern
- **Area:** Permissions
- **Evidence:** `SKILL.md:4` pre-approves `Bash` and `Write`. The body reads
  git history and writes one file.
- **Impact:** `Bash` is pre-approved for the whole turn that invokes the skill,
  not just for reading git log. Any later Bash call in that turn runs without a
  prompt. Whether that matters depends on what else the turn does, which is why
  this is a Concern.
- **Fix:** Keep `Write`; drop `Bash` unless the body needs it, or narrow it.
- **Confidence:** Medium

### [Minor] The description is a keyword list, not a discriminator

- **Type:** Concern
- **Area:** Triggering
- **Evidence:** `SKILL.md:3` — "release notes, changelogs, version summaries,
  git history, release management".
- **Impact:** "git history" and "release management" are broad enough to catch
  requests this skill does not serve — "show me the git history of this file"
  would match. A description that fires on near-misses trains the user to
  ignore it. This is a hypothesis until tested.
- **Fix:** Say what it does and when, and let the near-misses fall out. See
  below.
- **Confidence:** Low — it names its check: run the three trigger tests below
  and see whether test 3 activates the skill.

## Fix first

The two Majors, before this is used again. The hook first: it is the one whose
effect outlives the skill.

## Suggested changes

Description:

> Writes a changelog entry from git history: groups commits by kind, and
> produces one entry in the project's existing format. Use when the user asks
> for a changelog, release notes, or a summary of what changed since a tag.
> Registers a hook that stages the changelog file after writing it. Does not
> tag, push, or create a release.

Trigger tests — **proposed, not executed**:

1. "write the changelog for 0.4.0" → should activate.
2. "what changed since v0.3.9?" → should activate.
3. "show me the git history of `src/api.ts`" → should **not** activate.

## Strengths

- `references/format.md` is loaded only when the project has no existing
  changelog to match (`SKILL.md:61`), which is the right condition — most runs
  never pay for it.
- The commit-grouping rules name the failure they prevent: `SKILL.md:44` says
  merge commits are dropped "because they duplicate the branch they merged".

## Not statically verified

- Whether the description triggers as intended. The three tests above resolve
  it; none were run.
- Whether the `PostToolUse` hook behaves as read here. The blast radius is from
  the documented semantics of session-scoped hooks, not from observation.
