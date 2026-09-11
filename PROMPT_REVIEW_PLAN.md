# Implementation Plan: Coherent Knowledge Generation

Status: implementation plan ready; runtime changes and live comparisons
are pending.

- Repository: `obafgkm42/obsidian-knowledge-overview`
- Working branch: `codex/prompt-coherence-plan`
- Reviewed runtime: `3e9181e3eb2c2525719dcd746489564c4b44d493`
- Previous proposal: `8b063a8e9a8e970d60fc64f0a0a6b822b0e4db5c`
- Audit date: 2026-09-11

The previous proposal and this plan use the same runtime source. Its earlier
baseline, `4299505329999bc7306a76766a2c542072efb4a7`, changed documentation only.
Record the actual starting revision again before implementation.

## 1. Outcome and scope

A reader should understand how a chapter's ideas fit together without having
to reconstruct the relationship between individually useful sections.
Examples, formulas, objections, and limitations should advance that
understanding at the point where they are needed.

This plan combines the writing proposal with the pipeline audit in one
implementation document. It defines separate changes for shared writing
instructions and domain guidance, with tests and comparisons for each.
Shorter prompts are supporting evidence, not the primary success criterion.

The first runtime change will:

1. Strengthen the organizing purpose carried by the existing course goal and
   chapter focus.
2. Explain how the writer should develop that purpose.
3. Clarify the difference between course-wide terminology consistency and
   the concepts that this chapter must teach.
4. Consolidate shared instructions without weakening the output contract.

A subsequent, separately evaluated change will address domain applicability,
primary/secondary teaching roles, and positional adapter merging.

This document does not authorize paid generation. Implementation can proceed
through offline checks; live comparisons use the existing explicit budget
and confirmation workflow. Publication of this plan does not mean that any
runtime task below is complete.

## 2. Current pipeline and evidence

The production path is:

```text
Course name + language + depth
  -> one model-generated course blueprint
  -> parse and normalize chapter specifications
  -> select primary and secondary knowledge adapters
  -> assemble an independent request for each chapter
  -> generate chapter prose
  -> normalize formulas and number headings
  -> run local structural diagnostics
  -> save the note and report generation success
```

Relevant sources:

- [Prompt construction](src/prompts.ts)
- [Runtime orchestration](src/plugin.ts)
- [Blueprint parsing and fallback](src/courseBlueprint.ts)
- [Adapter definitions and merging](src/domainAdapters.ts)
- [Adapter selection](src/instructionalPlanner.ts)
- [Density settings](src/densityPresets.ts)
- [Local chapter diagnostics](src/chapterQuality.ts)
- [Evaluation corpus](tests/eval/corpus/v1/manifest.json)

The findings below distinguish observed instructions/data flow from possible
effects on generated prose. No live A/B generation was performed for this
audit.

### E1. Coverage order does not fully specify an explanatory path

The current prompt already orders prerequisites before dependent material,
asks for meaningful headings and connected paragraphs, and gives each
chapter a focus. Several adapters also describe relationships and argument
chains. These useful instructions must be retained in substance.

What remains underspecified is how the required topics contribute to the
chapter's overall purpose. A sequence of definitions, examples, and
exceptions can satisfy those local instructions while leaving the reader
to infer why the next section is needed.

This is a plausible mechanism, not an observed failure rate. Address it in
the shared-prompt change and assess the resulting prose.

### E2. Mixed adapters introduce instructions from unrelated subject matter

`mergeAdapters` currently takes the first three section roles and unit
fields, all example requirements, and the first two reliability rules from
each secondary adapter. It removes only exactly identical strings.

Offline assembly of the existing `stem-fourier-aliasing` context produced:

| Adapter selection | Total adapter items |
| --- | ---: |
| Mathematical only, as a controlled comparison | 21 |
| Mathematical + conceptual + empirical, as in the corpus | 41 |

The count is the sum of `requiredSections`, `unitFields`,
`exampleRequirements`, and `reliabilityRules` after merging. It is not a
token count, and not every item is an unconditional obligation.

Both the mixed aliasing and chemical-equilibrium prompts contain requirements
about intention-to-treat, churn/non-login, and an empirical or backtest
example. Those instructions enter through the empirical adapter. Their
presence in the assembled prompt is verified; whether a model follows them
into an irrelevant digression needs generation evidence.

Positional selection also drops potentially useful protections. For example,
the mathematical adapter's later pricing-measure and stationary-point
cautions disappear when mathematics is secondary. Array order is not a
sufficient policy for deciding which protections a chapter needs.

Address applicability and semantic ownership in the adapter change. Merely
removing duplicate sentences cannot resolve this finding.

### E3. Course terminology can be mistaken for mandatory chapter coverage

Every chapter receives the full course terminology list. The non-English
contract calls both course and chapter terms mandatory bilingual terminology.
Elsewhere, it restricts coverage to this chapter and limits the final table
to terms actually taught.

This creates an ambiguity: a course-wide naming reference may be interpreted
as a list of concepts to introduce in every chapter. Clarify its role in the
shared-prompt change. Preserve natural first-use bilingual terminology and
the final table; do not solve this by removing either learning feature.

### E4. Chapters do not know their neighbors' generated prose

The context object contains the blueprint, but the rendered chapter prompt
includes only neighboring titles and focus statements, not the full chapter
sequence or generated prose. Concurrency one does not create shared memory.

The demands for a self-contained chapter, definitions before use, and no
repetition of neighboring material can therefore pull in different
directions. Allow a brief prerequisite bridge, while prohibiting invented
references to unseen examples or wording.

Fallback specifications are weaker still: they can contain only a title-like
focus and empty subtopics, objectives, prerequisites, and exclusions. A
writer instruction cannot recover missing scope information. Test graceful
behavior, but do not claim that fallback guarantees course-wide boundaries.

### E5. Shared instructions and tests reinforce repeated wording

Bilingual introduction, heading limits, QA boundaries, and source anchors
are stated in multiple places, including the final self-check. Some prompt
tests assert several equivalent long sentences independently.

Consolidate by meaning. Distinguish duplicate reminders from complementary
requirements, applicability conditions, and exact parser-facing syntax.
Tests should protect the contract rather than preserve every formulation.

### E6. Density and chapter-count requirements can encourage fragmentation

Subtopic coverage, objectives, examples, failure modes, terminology, review
questions, and heading limits all compete for finite space. Core-unit counts
are already qualified as planning guidance, but the combined requirements
can still encourage a series of equally sized teaching packets.

At course level, onboarding enforces at least ten chapters even for a narrow
topic. This is a possible source of overdivision.

Keep existing counts and density settings unchanged in the first two runtime
changes so comparisons remain interpretable. Record quota pressure as an
unresolved hypothesis, especially when a narrow-topic case remains weak.

### E7. Structural diagnostics cannot certify semantic coherence

Local checks count and inspect length, headings, lists, questions, tables,
and source anchors. An existing H2 anchor does not prove that the answer is
taught, and valid headings do not establish an explanatory relationship.

The runtime saves chapters with diagnostic warnings and reports generation
success. This is a missing semantic safeguard, not itself the cause of poor
prose. Retain that execution behavior in this work; use independent review
of saved evaluation outputs to judge meaning.

## 3. Writing behavior to implement

A chapter's organizing purpose may be an explanatory question, practical
goal, comparison, or interpretive problem. It need not appear as a literal
question. The opening establishes what the reader is trying to understand;
the body develops the required material toward that understanding.

A survey may have parallel branches. A historical explanation may preserve
several causes. A literature chapter may end with competing readings.
Coherence does not require a single argument or an artificial resolution.

For example, a sampling chapter can start from recovering information from
discrete observations, explain why different signals can yield the same
samples, establish the relevant constraint under stated assumptions, and
then motivate filtering before sampling. This is an illustrative structure,
not evidence from a reviewed generated chapter.

The chapter-level instruction should convey:

> Organize the chapter around the central question or learning purpose
> expressed by its focus. Establish that purpose naturally near the beginning,
> then develop the required topics in an order that helps the reader answer
> it. Show how each section builds on, qualifies, contrasts with, or applies
> what has already been established. Place examples and limitations where
> they advance that explanation. Briefly recall necessary prerequisites
> without reteaching neighboring chapters or claiming knowledge of their
> actual prose. Bring the teaching body back to its purpose before the review
> questions, preserving unresolved disagreements and uncertainty where
> appropriate.

Do not require a transition phrase at every section, a recurring example,
or an additional summary heading. Prefer reusing an example when it reduces
cognitive effort; allow a new one when the new concept needs it.

Apply these scope rules together:

- Required subtopics and objectives define chapter coverage. A stronger focus
  must not silently remove them.
- Course terminology is a consistency reference: when a relevant concept is
  used, preserve its canonical wording and symbol.
- A course term's presence alone is not a requirement to teach it here.
- Teach relevant chapter terms within the stated scope. If supplied metadata
  conflicts with an explicit exclusion, respect the exclusion and do not
  manufacture an aside to satisfy the terminology list.
- Recall prerequisites only as needed for the current explanation. When
  metadata is absent, avoid pretending that an earlier chapter established
  a particular fact, convention, or example.

## 4. Contracts and exclusions

Preserve the following through the shared-prompt change:

| Area | Required behavior |
| --- | --- |
| Coverage | Existing subtopics, objectives, prerequisites, and exclusions |
| Density | Existing depth presets, chapter ranges, and example/question counts |
| Non-English prose | Accurate English equivalents at first useful occurrence |
| Bilingual minimum | Non-English: at least five final-table terms used bilingually |
| Terminology table | Eight to fifteen taught terms; existing language columns |
| QA | Body-grounded questions; no new concepts first introduced in questions |
| QA derivation | At most two explicit body claims combined per question |
| Boundaries | Exact QA and terminology markers and source-anchor syntax |
| Headings | Existing H2/H3 limits and application-supplied numbering |
| Rendering | Existing Obsidian math syntax and optional Mermaid behavior |
| Execution | One outline plus N chapter requests in a normal run |
| Recovery | Existing concurrency, cancellation, resume, and diagnostic behavior |

The terminology table remains the final model-authored content. The plugin's
existing provenance footer remains outside that model-authored contract.

The adapter change may revise which domain-specific examples apply. It must
preserve density-preset counts and necessary domain protections, and document
each changed applicability rule. Do not conceal such changes as deduplication.

Excluded from this implementation:

- Adding previous chapter prose, a memory store, or a dependency graph.
- Adding required blueprint fields or migrating saved outlines.
- Building a topic classifier or a semantic deduplication service.
- Changing chapter minima, heading limits, density presets, or output budgets.
- Adding automatic model judging or automatic chapter rewriting.
- Making provider, retry, model-default, or release-version changes.

## 5. Implementation tasks

Each stage should be a separate reviewable change. Complete its offline
checks before requesting a live comparison. Keep successful and unsuccessful
changes independently reversible.

### A. Establish the baseline and the review method

Files: `tests/prompts.test.ts`, `tests/eval-config.test.ts`,
`tests/eval/corpus/v1/manifest.json`, `tests/eval/CODEX_REVIEW_SCHEMA.md`,
`tests/eval/reviewPacket.ts`, and `tests/eval/README.md`.

- [ ] Record runtime revision, corpus version, case IDs, prompt fingerprints,
  and sanitized model/provider settings before changing runtime prompts.
- [ ] Reproduce E2 using the existing aliasing and chemical-equilibrium
  contexts and the production prompt/adapter functions.
- [ ] Record adapter item counts and system/user prompt lengths separately.
  Use Unicode code-point counts or UTF-8 bytes and label the unit explicitly.
- [ ] Add the fixed cases and selection tests in Section 6. Freeze their
  contents before generating either comparison arm.
- [ ] Keep the same test harness and corpus revision for baseline and
  candidate; vary only the identified production source revision.
- [ ] Extend the existing review schema guidance and review packet with
  Section 7's rubric. Keep the report schema and score field names.
- [ ] Add an offline prompt-composition inspection test/helper if needed.
  Reuse production builders; do not reimplement prompt assembly in the test.
- [ ] If adding a test file, import it from `tests/all.test.ts` so `npm test`
  actually executes it.

Keep API keys, authorization headers, full request bodies, and generated
notes out of committed artifacts. Preserve the harness's existing artifact
policy. Use hashes, counts, case IDs, and selected sanitized evidence rather
than adding request-body logging.

**Exit:** reproducible baseline composition, frozen cases, and a review rubric.
No claim about generated improvement is possible yet.

### B. Revise shared prompts and terminology scope

Files: `src/prompts.ts` and `tests/prompts.test.ts`.
Add fallback assertions to `tests/course-blueprint.test.ts` if necessary.

- [ ] In `buildOutlinePrompt`, clarify `courseGoal` as intended understanding
  and `focus` as the chapter's central problem and contribution to that goal.
  Retain schema, types, coverage fields, and all chapter-count rules.
- [ ] In `buildInstructionalSystemPrompt`, state the broad priority briefly:
  develop a connected explanation within the blueprint's boundaries.
- [ ] In `buildChapterPrompt`, place operational continuity instructions near
  the chapter context and apply Section 3's terminology/scope distinction.
- [ ] Give each shared requirement one clear home. Keep terminology together,
  QA grounding and metadata together, and rendering constraints together.
- [ ] Reduce the final self-check to coverage, explanatory continuity, and
  output validity without reciting the full contract.
- [ ] Leave all adapter definitions and merging behavior unchanged.
- [ ] Replace assertions tied to obsolete wording with focused contract
  assertions. Keep exact parser markers and localized table/QA syntax exact.
- [ ] Cover first, middle, last, descriptive-focus, and sparse fallback
  contexts; confirm missing neighbors do not add invented context.
- [ ] Test an irrelevant course term and a conflicting excluded term.
  Verify the scope instruction survives construction; reserve judgment
  about model compliance for the live cases.
- [ ] Run the offline commands in Section 8, then compare the fixed cases
  against the baseline using Section 7.

**Exit:** retain the writer revision only when its generated comparison meets
the gate. This fixed-context comparison does not validate the outline edit:
keep that edit pending until D passes, or split it from the writer change.
If live evaluation has not run, label it
"prompt construction verified; generated quality not yet validated."

This comparison evaluates the shared-prompt bundle. It does not separately
attribute improvement to focus wording, continuity instructions, terminology
scope, and shorter repetition. If the bundle fails, reduce the candidate
change before commissioning further comparisons.

### C. Make domain composition respect purpose and applicability

Files: `src/domainAdapters.ts`, `src/instructionalPlanner.ts`,
`src/prompts.ts` only if needed for role framing, and focused adapter tests.
Register new tests in `tests/all.test.ts`.

Start from the accepted B revision. If B is rejected or inconclusive, an
adapter-only candidate can instead start from the original baseline; record
that choice and do not mix unvalidated B changes into the comparison.

- [ ] Inventory each domain instruction as a teaching contribution,
  reliability protection, or repetition of a shared rule.
- [ ] Keep the primary type responsible for the chapter's main explanatory
  structure. State that secondary types enrich relevant parts instead of
  creating additional miniature chapters.
- [ ] Rewrite broad empirical guidance around evidence, measurement,
  assumptions, examples, and inference limits. Qualify experimental
  reminders by applicability to the actual chapter.
- [ ] Remove unconditional demands for a backtest, online-experiment workflow,
  or other specialized example from unrelated empirical applications.
- [ ] Replace positional `slice` selection with explicit, named secondary
  guidance for each knowledge type. Keep the public adapter interface if
  practical and reuse shared strings rather than copying entire adapters.
- [ ] Include distinctive secondary teaching needs and applicability-qualified
  protections deliberately. Do not drop a safeguard simply because it appears
  late in an array. If a shorter formulation loses protection, keep the
  qualified original and accept its prompt cost.
- [ ] Preserve the specific A/B experiment and mathematical interpretation
  protections where relevant. Include mathematics as a secondary type in
  regression coverage, not just as the primary type.
- [ ] Test primary-only, primary-plus-one, primary-plus-two, duplicate types,
  and `hybrid` selection through the actual planner and adapter functions.
- [ ] Test aliasing and chemical equilibrium for absence of unconditional
  ITT/churn/backtest obligations. A clearly conditional caution is not the
  same as an instruction to teach that subject.
- [ ] Test A/B validity for retained assignment, missing-outcome, interference,
  leakage, SRM, and repeated-A/A distinctions.
- [ ] Test relevant mathematical contexts for retained stationary-point,
  across-step equilibrium, and pricing-measure distinctions, including when
  mathematical guidance is secondary.
- [ ] Confirm interpretive/argumentative combinations retain textual evidence,
  serious objections, and unresolved alternatives.
- [ ] Run offline checks and a separate generated comparison before acceptance.

**Exit:** relevant secondary contributions remain, irrelevant obligations are
removed or clearly limited, and the reviewed outputs meet the same semantic
and regression gate. A lower item count alone does not pass this stage.

### D. Compare generated outlines and adjacent chapters

The current evaluation runner generates chapters from fixed blueprints. It
does not yet provide a paired live outline comparison. Do not imply that
`eval:generate --profile smoke` tests outline generation.

- [ ] Prepare a bounded evaluation-only outline comparison using the production
  outline builder and parser, including minimum-chapter validation.
- [ ] Reuse the existing provider client, confirmation, cancellation, logical
  and physical request caps, token limits, and sanitized artifact rules.
  Do not use an ad hoc uncapped provider script.
- [ ] If a new mode or helper is required, implement and document its CLI,
  fingerprinting, budget accounting, and offline tests before any live run.
- [ ] Compare old and revised outlines for the same course input, language,
  depth, model, provider settings, and output limit.
- [ ] Select the same intended learning transition in both outlines. Do not
  equate chapter numbers when their scopes differ.
- [ ] Generate two adjacent chapters from each outline with the same accepted
  chapter writer revision. This holds the writer fixed while examining
  outline-driven differences.
- [ ] Assess purpose, prerequisite order, boundaries, recalled concepts,
  repeated teaching, and invented references to neighboring prose.
- [ ] Report whether coverage is comparable. If the outlines changed scope
  materially, describe that difference rather than scoring unlike chapters
  as a clean coherence comparison.

**Exit:** paired evidence supports any claim about outline improvement.
Reading only a new outline can establish plausibility, not improvement over
the old version. Even this paired check does not establish shared memory of
generated chapter prose.

## 6. Test and generation matrix

At the audited revision, all 12 corpus cases use Traditional Chinese,
`onboarding` depth, and mixed knowledge types for the selected chapter.
The existing four-case smoke profile already covers mixed STEM/humanities;
it does not cover English or single-type generation.

### Offline matrix

No provider calls are needed for these tests:

- English and Traditional Chinese, each with single and mixed types.
- All four depth presets, preserving counts and length instructions.
- First, middle, and last chapters, including both missing neighbors.
- Descriptive focus with valid coverage, and a real sparse fallback.
- Course terminology outside chapter scope, including an explicit conflict.
- Primary/secondary combinations and the domain regressions listed in C.
- Unchanged QA/terminology boundaries, source anchors, numbering, and formulas.
- New corpus/profile selection and prompt-fingerprint changes.

Use synthetic content only. These tests verify construction and preservation
of context; they cannot prove that generated prose obeys an instruction.

### Shared-prompt live pilot: nine cases per version

Use the four existing smoke cases unchanged:

1. `stem-fourier-aliasing`
2. `stem-ab-test-leakage`
3. `humanities-unreliable-narrator`
4. `humanities-french-revolution`

Add five named cases before live evaluation:

5. `coherence-aliasing-zh-single`: aliasing with mathematical guidance only.
6. `coherence-aliasing-en-single`: fully localized English equivalent.
7. `coherence-aliasing-en-mixed`: the same English context with the original
   secondary types.
8. `coherence-legacy-focus`: descriptive focus with bounded subtopics and
   objectives; do not strengthen its focus as part of the candidate.
9. `coherence-scoped-terms`: unrelated course terminology and an explicit
   exclusion, with enough relevant taught terms to satisfy the final table.

These five IDs are planned additions, not currently executable commands.
Localize the complete English blueprint, not only its language selector.
Freeze fixtures, expected scope, and lexical expectations before both arms.

Run one output per case and version initially: **18 logical requests**.
The result is a bounded pilot at onboarding depth, not a claim across all
models, languages, or depths. Execute in batches compatible with approved
caps; do not silently increase the existing default limits.

### Adapter live pilot: six cases per version

Compare the chosen pre-adapter baseline with C using:

- Aliasing and chemical equilibrium: cross-topic obligations.
- A/B leakage/contamination: retained empirical protections.
- Unreliable narrator: distinct interpretive and argumentative contributions.
- No-arbitrage option pricing: mathematical interpretation protections.
- One new, fixed mathematical-secondary case where a specific protection is
  relevant, using the same learning scope in both arms.

Initial budget: **12 logical requests**. The last case must be added, reviewed,
and frozen before running either arm.

### Outline pilot: one subject, then a separate extension if needed

Start with the same sampling/signal-processing subject in both arms: one
outline plus two adjacent chapters per arm, **six logical requests total**.
Generate the full required outline but only the selected chapter pair.

A separate humanities pair would add six requests and needs its own planned
budget. Do not claim cross-domain outline improvement from the STEM pair.

### Variability and budgets

Before running, record the model, endpoint identity without credentials,
temperature or its omission, reasoning/thinking controls, output limits,
case set, and source/corpus revisions. Keep them fixed between arms and note
any provider compatibility fallback that changes the effective settings.

For a disputed fixed-chapter result in B or C, predeclare one additional
paired generation for at most two affected cases: at most **four extra
requests per stage**. Review all outputs; do not select the most favorable
sample. If the second pair still disagrees, mark the outcome inconclusive.
Repeating D's full outline-and-chapter comparison costs another six requests
and requires a separate preflight and authorization.

The shared, adapter, and initial outline pilots total **36 logical requests**
if all three stages are separately approved and run. This is planning
arithmetic, not approval or a monetary spending cap. Physical attempts and
token costs require the actual preflight plans for each stage.

## 7. Semantic review and acceptance

Use [the existing review schema](tests/eval/CODEX_REVIEW_SCHEMA.md).
Extend its guidance and the generated review packet; keep score fields and
issue categories compatible.

Read the teaching body before the questions and formal completeness checks.
For each output, record:

1. The organizing purpose in one sentence.
2. What each teaching section contributes and how it relates to another.
3. A short exact passage for every important missing connection.
4. Whether examples, formulas, objections, and limitations serve that purpose.
5. Whether the ending answers or appropriately limits the opening purpose.
6. Scope drift, repeated teaching, invented neighbor references, and QA gaps.

Use existing `sequence` issues for missing explanatory connections and
`scope` or `domain-fit` issues for irrelevant domain material.

Apply these anchors to `conceptSequence`:

| Score | Interpretation |
| --- | --- |
| 1 | Essential dependencies are missing; topic shifts block understanding |
| 3 | A purpose and usable order exist, but key links still require inference |
| 5 | Sections visibly develop the purpose; dependencies and limits are clear |

Use 2 and 4 for intermediate cases. `headingUtility` assesses whether
headings reveal the teaching progression without formulaic fragmentation.
`learningEfficiency` assesses useful development versus repeated setup,
unmotivated asides, checklist-like padding, and excessive prerequisites.
A smooth transition phrase or sensible heading alone does not establish
coherence.

Use the same reviewer instructions for both arms. Where practical, label
outputs anonymously and vary reading order. Preserve a private mapping to
source revisions and case IDs for reproducibility.

Accept a candidate only when:

- Cases with baseline continuity problems show concrete improvement supported
  by passages and an explanation of the repaired relationship.
- Previously strong cases do not materially regress.
- No new major or critical scope, factual, domain-fit, terminology, rendering,
  or QA-answerability failure is introduced.
- Existing structural gates pass; known baseline failures are recorded and
  are not silently relabeled as successes.
- Coverage remains comparable; apparent coherence from dropping required
  concepts is a failure.
- The candidate's targeted mechanism is checked, including irrelevant
  obligations for C and paired outlines for D.

Do not let an average score hide a material regression. Shorter prompts,
fewer headings, longer paragraphs, or passing prompt assertions are not
acceptance evidence by themselves.

If no baseline case has a relevant weakness, the comparison can demonstrate
non-regression but not improved coherence. If differences are stylistic,
mixed, or within generation variability, record "inconclusive" and retain
the last accepted runtime revision.

## 8. Commands and validation

Run repository checks for every implementation change:

```bash
npm ci
npm test
npm run lint
npm run build
git diff --check
```

The build includes type checking. Inspect any tracked `main.js` change and
follow repository release-artifact practice; do not commit disposable test
bundles or generated evaluation notes. If a runtime build artifact is part
of the change, also run `npm run release:verify`. Do not bump the plugin
version merely to edit this plan.

Existing provider-free planning commands include:

```bash
npm run eval:plan -- --profile smoke
npm run eval:plan -- --case stem-fourier-aliasing
```

After the case additions, their individual `--case` commands use the same
workflow. Any new outline command must first be implemented and documented
in Stage D.

Only after explicit authorization of the current preflight plan:

```bash
npm run eval:generate -- --case stem-fourier-aliasing --confirm PLAN_ID
```

Replace `PLAN_ID` with the exact current plan ID. Changed prompts or settings
require a new plan; do not reuse confirmation from another source revision.

To rerun local checks on saved output without provider calls:

```bash
npm run eval:check -- --run eval/runs/RUN_ID
```

For this documentation-only update, validate source references, commands,
case names, budget arithmetic, Markdown, whitespace, and sensitive-content
hygiene. Do not claim that runtime tests or generated comparisons ran.

## 9. Deferred work and implementation handoff

The following remain visible follow-up questions, not hidden requirements
of the current prompt edits:

- **Cross-chapter prose continuity:** if paired chapters still contradict or
  repeat each other, design a compact shared-context proposal separately.
- **Sparse fallback scope:** if graceful prompt handling is insufficient,
  evaluate a separate product decision about missing outline metadata.
- **Quota-driven fragmentation:** if narrow-topic cases remain fragmented,
  compare scope-aware chapter counts or density settings in a new experiment.
- **Runtime semantic checks:** any future judge or rewrite step needs its
  own cost, failure, and user-control design.

Every implementation handoff should include:

- [ ] The completed stage, exact source revision, and comparison baseline.
- [ ] Changed requirements and preserved contracts.
- [ ] Offline checks executed and their results.
- [ ] Generated case/run IDs, settings, and authorized request/token budgets.
- [ ] Passage-level findings, per-case verdicts, and material regressions.
- [ ] A decision: accept, reject, or inconclusive.
- [ ] Remaining limitations and the revision to restore if a regression appears.

Write repository documentation and implementation commentary in English.
Chinese text may remain in test fixtures and quoted evaluation evidence where
it exercises language-specific behavior. Keep evidence sanitized and follow
[CONTRIBUTING.md](CONTRIBUTING.md) and the
[evaluation artifact rules](tests/eval/README.md).
