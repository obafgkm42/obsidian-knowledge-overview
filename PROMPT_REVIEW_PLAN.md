# Proposal: Make Generated Chapters Follow a Clear Line of Explanation

Status: proposed; implementation and generation comparison have not started.  
Repository: `obafgkm42/obsidian-knowledge-overview`  
Working branch: `codex/prompt-coherence-plan`  
Review baseline: `4299505329999bc7306a76766a2c542072efb4a7`

## Purpose and recommendation

A reader should finish a chapter understanding how its ideas fit together, rather than having to assemble that relationship from a collection of individually useful sections. This proposal addresses that outcome by making the chapter's organizing question explicit and giving the model clear instructions for developing an answer. Prompt consolidation supports that change: it reduces repeated demands that currently compete with the explanation itself.

The recommended first change is deliberately contained. Reuse the existing course blueprint, the structured outline that defines each chapter's scope, to express what each chapter contributes to the reader's understanding. Rewrite the shared writing instructions around that contribution, and consolidate repeated rules without weakening the output contract. Keep domain-specific reliability rules unchanged during this first comparison. If the result is promising, simplify domain guidance in a separate change whose effects can be assessed independently.

This document is a handoff for a human or another agent implementing the proposal. It explains the evidence, the intended behavior, the order of work, and the conditions for accepting the result. Repository documentation and implementation commentary should be written in English. Chinese text may remain in automated test material where it exercises language-specific behavior.

## Why the current instructions leave a gap

The current prompt already contains useful guidance about coherence. In [src/prompts.ts](src/prompts.ts), the outline must place prerequisites before dependent material, and chapter headings must form a clear learning progression. The chapter writer is also told to prefer connected explanatory paragraphs. These instructions should be retained in substance. The missing element is an explicit relationship between the chapter's overall purpose and the work each section performs.

A prerequisite order establishes that one concept can be understood before another. It does not explain why the reader needs the second concept to answer the chapter's central question. Similarly, paragraphs can be grammatically connected while still presenting unrelated definitions, examples, and exceptions. The present prompt specifies many properties of the finished chapter, but gives less direction on how to choose and develop its explanatory path.

Consider a chapter about sampling and aliasing. A chapter can cover sampling rate, the Nyquist condition, aliasing, and filtering under sensible headings while leaving the reader to infer their relationship. A more coherent explanation starts with the problem of recovering information from discrete observations, shows how different signals can produce the same samples, derives the relevant sampling constraint under stated assumptions, and then explains why filtering is needed before sampling. The same required topics remain, but each section creates the reason for the next. This is an illustrative design example, not a finding from a generated chapter reviewed in this task.

Several features of the assembled prompt make this gap worth addressing. The terminology block repeats the requirement to introduce English equivalents in the body through multiple formulations, then repeats it in the final self-check. Heading limits and review-question requirements are also restated. In [src/domainAdapters.ts](src/domainAdapters.ts), a domain adapter is a set of additional teaching instructions selected for a knowledge type. Definitions, examples, assumptions, and misconceptions can appear across its section roles, unit fields, and example requirements. Mixing adapters adds further instructions, while the current deduplication removes only identical strings. None of this proves that repetition harms a particular model, but it creates a plausible pressure to satisfy a checklist of ingredients.

There is also a concrete limitation on chapter continuity. [src/plugin.ts](src/plugin.ts) builds each chapter request independently. Although the context object contains the blueprint, the chapter prompt renders only the neighboring chapters' titles and focus statements, not their generated prose or the complete chapter sequence. The writer therefore has enough information to position the chapter approximately, but cannot know what example or wording a previous chapter actually used. The instruction not to repeat neighboring material should allow a brief conceptual bridge without inviting fabricated references to unseen content.

These are findings about the instructions and data flow. No new generation comparison has been run, and this proposal does not claim an established quality improvement, a measured token saving, or a benefit across all models.

## The intended writing behavior

The organizing question should come from the chapter's existing focus and required coverage. It may be an explanatory question, a practical goal, or an interpretive problem; it need not be written as a literal question in the output. The opening should establish what the reader is trying to understand and why it matters. Subsequent sections should make progress toward that understanding through dependencies, causes, comparisons, evidence, or procedural steps appropriate to the subject.

This does not require every chapter to become a single argument or every section to start with a transition phrase. A survey chapter may have parallel branches under one organizing purpose. A historical chapter may distinguish several contributing causes. A literature chapter may end with competing readings. Coherence means that the relationships are intelligible, not that uncertainty disappears.

Examples, formulas, objections, and limitations should appear where they advance the explanation. A recurring example is useful when it reduces the reader's effort, but should not be mandatory. Necessary prerequisites may be recalled briefly; material outside the chapter's scope should not be taught merely to make a transition smoother. Before the review questions, the teaching body should resolve the opening problem as far as the material permits and make any remaining limits clear. This closure belongs in the explanation and does not require another fixed summary heading.

The following passage captures the proposed chapter-level instruction. Its wording may be refined during implementation, but its meaning is the acceptance target:

> Organize the chapter around the central question or learning purpose expressed by its focus. Establish that purpose naturally near the beginning, then develop the required topics in an order that helps the reader answer it. Show how each section builds on, qualifies, contrasts with, or applies what has already been established. Place examples and limitations where they advance that explanation. Briefly recall necessary prerequisites without reteaching neighboring chapters or claiming knowledge of their actual prose. Bring the teaching body back to its purpose before the review questions, preserving unresolved disagreements and uncertainty where appropriate.

## How to express this in the existing design

The outline should carry the purpose, and the chapter prompt should explain how to develop it. In `buildOutlinePrompt`, retain the existing `courseGoal` and `focus` fields and their types. Clarify that the course goal describes the understanding the course should establish, while each chapter's focus identifies its central problem and contribution to that goal. The focus should remain concise enough to be useful when passed to neighboring chapters. Subtopics and learning objectives continue to define required coverage; a stronger focus must not silently narrow them away.

This choice avoids a blueprint migration and keeps saved outlines usable. Older outlines and fallback chapter specifications may have only a descriptive focus. The chapter instruction must still work by deriving an organizing purpose from the title, focus, and required topics, without expanding their scope. Missing neighbors should require no invented introduction or transition. There is no need to add previous chapter prose, additional model requests, or new required metadata for this first attempt.

Within `buildInstructionalSystemPrompt`, state the broad writing priority briefly: build a connected explanation within the blueprint's boundaries. Put the operational guidance in `buildChapterPrompt`, close to the chapter context. The two messages should have distinct responsibilities rather than repeating the full passage.

The rest of the shared prompt should have one clear home for each requirement. Keep bilingual introduction and the final terminology table together. Keep review-question grounding, section boundaries, and source anchors together. Keep heading and mathematical formatting constraints together. A short final self-check can direct attention to coverage, explanatory continuity, and output validity without reciting every count and prohibition again. Shortening must be done by comparing meanings, not by deleting every repeated phrase mechanically; some reinforcement may prove necessary for reliable output.

The first implementation must preserve the following contract so that any observed difference can reasonably be attributed to organization and consolidation:

| Area | Behavior to preserve |
| --- | --- |
| Coverage and density | Existing scope boundaries, chapter ranges, depth settings, length targets, example requirements, and question counts |
| Terminology | Required first-use bilingual wording, relevant canonical terms, existing table columns and term-count requirements |
| Review questions | Questions grounded in the teaching body, existing source anchors, and the marked question and terminology boundaries |
| Rendering | Existing heading limits, application-supplied numbering, Obsidian formula syntax, and optional diagram behavior |
| Execution | One outline request plus one request per chapter in a normal run, existing concurrency and resume behavior, no automatic rewriting |

Domain guidance should be handled after this shared-prompt change. The immediate task there is editorial: identify which instructions add a distinct teaching requirement and which merely restate shared guidance. Preserve the current adapter interfaces initially. Do not build a semantic deduplication service or a topic classifier as part of this proposal. Specialized reliability reminders, such as experiment-design or pricing-measure cautions, deserve separate attention because they may protect against known errors. Inspect their existing regression cases before generalizing or removing them. If a general rule cannot preserve that protection, retain the specific reminder and document the remaining prompt cost.

## Implementation sequence

The work should proceed as three reviewable steps. Each step has a concrete result that the next one depends on; completing a task list alone is not evidence that chapter quality improved.

First, establish the baseline and the review method. Record the implementation starting commit and select fixed chapter contexts from the existing evaluation material. Inspect the assembled English and Traditional Chinese prompts for a single knowledge type and a mixed type, recording lengths as characters or bytes rather than calling them tokens. Do not change the evaluation harness to persist full request bodies; follow its existing artifact and credential-handling rules. Extend the guidance in [tests/eval/CODEX_REVIEW_SCHEMA.md](tests/eval/CODEX_REVIEW_SCHEMA.md) to assess explanatory continuity within the existing `conceptSequence`, `headingUtility`, and `learningEfficiency` scores. Use existing sequence-related issue records for evidence. This makes the desired outcome reviewable without introducing a new report schema.

Second, implement the shared-prompt change in `src/prompts.ts` and update the relevant tests in `tests/prompts.test.ts`. Cover a normal chapter, a first or last chapter with a missing neighbor, and a legacy or fallback specification with a descriptive focus. Tests should confirm that context and required output markers survive prompt construction. Assertions tied to long obsolete sentences should be rewritten around the requirement they protect, while exact strings required by parsers remain exact. Prompt assertions can verify that the model receives an instruction; they cannot verify that its prose follows it. Complete the repository's `npm test`, `npm run lint`, and `npm run build` checks before opening an implementation pull request, and handle generated build artifacts according to repository practice.

Third, compare the shared-prompt revision with the baseline before changing adapters. If it meets the acceptance conditions below, adapter consolidation can follow as a separate commit or pull request with its own comparison. Review `src/domainAdapters.ts` alongside `src/instructionalPlanner.ts` to verify both single-type and mixed-type assembly. A secondary type must still contribute its distinctive teaching needs. If consolidation weakens those needs or a reliability safeguard, revert that portion rather than expanding the refactor to compensate.

The current task revises this proposal only. It does not execute those implementation steps, change runtime prompts, or authorize a paid evaluation run.

## Evaluation and decision

Use the existing [evaluation workflow](tests/eval/README.md). Its smoke profile provides an initial comparison across science and humanities; inspect the selected cases and add only the missing language or mixed-type coverage needed for this change. Keep chapter contexts, model, provider settings, and output limits fixed between versions. Holding those settings fixed reduces confounding, but does not eliminate generation variability. Run planning first, review its request and token ceilings, and obtain the required live-run authorization before generating.

A fixed chapter comparison isolates the writer change, but cannot establish that the revised outline instructions produce better course structure. To assess that claim, separately inspect an outline generated with the revised instructions and a pair of adjacent chapters from it. Check whether their focus statements establish a useful progression, whether necessary concepts are recalled without being retaught, and whether either chapter invents details about its neighbor. This small end-to-end check should be planned explicitly rather than folded into an unbounded full-course run.

The reviewer should read the teaching body before looking at its formal completeness. Identify the central purpose, then explain what each section contributes to it. When a transition fails, cite the relevant passages and describe the missing conceptual connection. Check whether examples support the current explanation and whether the ending answers or appropriately limits the opening purpose. A high score requires these relationships in the prose; sensible headings and frequent words such as “therefore” are insufficient. Apply the same rubric to both versions and, where practical, conceal which version produced each output until the review is complete.

Accept the shared-prompt change when the comparison provides concrete evidence of improved continuity in the cases that needed it, without new material failures in coverage, factual reliability, terminology, question answerability, or rendering. A previously strong case need not improve, but should not materially regress. Reduced prompt length is useful supporting information, not the deciding metric. If the result is mixed or differs only in style, report it as inconclusive and investigate the affected case before broadening the change.

The implementation handoff should state what changed, which checks ran, which generated cases were reviewed, and what remains uncertain. If only offline checks have run, record “prompt construction verified; generated quality not yet validated.” Keep successful shared-prompt changes separable from later adapter edits so that a regression can be reverted without discarding the whole improvement.
