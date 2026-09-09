# Model profiles

Load **one** — the one selected by `--model`. The others do not apply to the
review in hand and only cost context.

Each profile asks the same question about the skill under review: **does it
account for how the model that will run it actually behaves?** None of them is
a compliance list. A skill that never triggers the behaviour a profile
describes has nothing to answer for.

None of these describes the model performing the review. That one needs no
profile — it is already the model, and it reads its own instructions from this
skill's frontmatter.

Profiles age. Each carries its source so that replacing one when a model ships
is a contained edit, not an archaeology exercise.

---

## Opus 5

*Source: Anthropic guidance for Claude Opus 5.*

Assess whether the skill provides clear scope and output constraints without
unnecessary scaffolding.

Look for:

- excessive progress narration;
- repeated self-verification;
- mandatory verifier subagents without concrete need;
- unlimited delegation;
- unrelated improvements;
- incomplete upfront specifications for long tasks;
- suggestions requested when implementation is intended;
- rigid step-by-step reasoning;
- unclear boundaries on files or systems changed.

Treat narration as excessive only when it does not communicate material
progress, decisions, blockers, risks, or results.

Use additional verification when concrete acceptance criteria or high
consequences justify it. Do not assume any effort level is universally optimal.

---

## Fable 5.1

*Source: Anthropic guidance for Claude Fable 5.1.*

Assess whether the skill supports:

- completion of the requested task;
- useful progress updates during long work;
- batching independent tool calls;
- targeted rather than whole-file edits;
- control of unrelated changes and excessive tests;
- explicit retrieval when required;
- readable, structured output;
- sufficient output space for long deliverables;
- append-only conversation history where relevant.

---

## GPT-6 Astra

*Source: OpenAI model guidance,
<https://developers.openai.com/api/docs/guides/latest-model>. Retrieved
2026-09-09; six documented behaviours, quoted below.*

Astra reads skill files, so a skill written for it is inside the behaviour it
has to account for. The instruction-precedence point below is the one that
matters most to a skill and has no equivalent in the other profiles.

Assess whether the skill:

- **States that the user outranks it.** OpenAI documents Astra as *"more
  sensitive to information in context"* such as skill files, and recommends
  making explicit that *"The user's instructions take precedence over
  guidelines provided in a skill."* A skill that never says this can have an
  obsolete line in its own body outrank what the user just asked for. Its
  absence is a **Major** finding for this profile, and it costs one sentence.
- **Pushes toward action rather than approval.** Astra *"is more likely to ask
  the user a question when additional input could materially change the
  result."* A skill built on many confirmation gates compounds that into
  stalling. The documented correction is to *"bias towards action and carry the
  user's intended task to completion"*, and to treat *"can you…"* as an
  instruction rather than a capability check.
- **Says where the work stops, not where the proposal stops.** The guidance is
  to *"complete all the necessary work until the intended outcome is
  fulfilled"* and to *"complete the work that is already authorized from
  context"* before seeking approval, leaving user review as the final step. A
  skill whose success condition is a plan rather than an outcome will produce
  one.
- **Constrains formatting when it matters.** Astra *"tends toward detailed,
  formatted responses."* If the skill's output should be prose, it has to ask
  for it — *"clear, concise paragraphs, each developing one main idea"* — and
  reserve lists for what is *"genuinely parallel, sequential, or easier to
  compare."* A skill silent on output shape gets tables it did not want.
- **Scopes testing to risk.** The documented calibration is *"Do not write
  tests for reversible, low-impact changes"*, broadening *"only when new
  changes, failures, or unresolved concerns justify it."* A skill that demands
  tests unconditionally fights the model rather than steering it.
- **Says when to parallelise.** Astra *"may delegate less often than
  desired."* Where a skill has independent work, saying so is what gets it
  done in parallel.

Note the inversion against Opus 5: there, unlimited delegation and mandatory
verification are the failure modes to look for. Here, under-delegation and
under-scoped completion are. The same instruction can be a defect under one
profile and a fix under the other — which is why exactly one profile applies
to a review, and the report names which.

---

## Generic

Use when the target model is unknown, or when the skill must hold up across
several.

Assess clarity, context, output constraints, tool requirements, autonomy,
examples, and resistance to overfitting.

Recommend examples, XML, or additional structure only when they solve an
observed problem.
