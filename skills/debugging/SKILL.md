---
name: debugging
description: Use this skill when implementation, debugging, or troubleshooting work has stalled after repeated failed attempts, when the user is dissatisfied with repeated non-progress, or when the assistant is at risk of giving up too early without enough evidence.
---

# Debugging

## Purpose

This skill helps the assistant recover from stalled implementation or troubleshooting work.

It is designed for situations where:
- multiple attempts have failed
- the assistant is repeating the same approach with small variations
- the root cause is still unclear
- the user signals dissatisfaction with lack of progress
- the assistant is tempted to stop before gathering enough evidence

The goal is to turn unproductive retry loops into structured investigation, evidence-based reasoning, and either:
1. a validated fix, or
2. a high-quality failure report with clear next steps.

---

## When to use

Use this skill when **one or more** of the following are true:

- Two or more attempts have failed without meaningful new information.
- The current approach is only a small variation of a previously failed approach.
- The assistant is about to say “I can’t fix this” or “you need to do this manually” without first exhausting reasonable investigation steps.
- The assistant is attributing the problem to environment, permissions, tooling, or external factors without direct evidence.
- The user expresses dissatisfaction, such as:
  - “try harder”
  - “figure it out”
  - “that still doesn’t work”
  - “why are we stuck”
  - “use another way”
- The assistant has produced a possible fix but has not verified it yet.

---

## When not to use

Do **not** use this skill when:

- The task is primarily creative writing, brainstorming, or open-ended ideation.
- The user explicitly wants a brief answer rather than deep troubleshooting.
- The task involves high-risk actions that should not be attempted without confirmation.
- The missing information can only come from the user and cannot reasonably be inferred or discovered.
- Further investigation would clearly produce no new information because tools, permissions, or inputs are unavailable.

---

## Core principles

1. **Do not repeat the same failed idea.**  
   A new attempt must be meaningfully different and should be likely to produce new information.

2. **Prefer evidence over guesswork.**  
   Do not blame tooling, environment, permissions, or undocumented behavior unless there is direct evidence.

3. **Investigate before asking the user for help.**  
   Ask only when the missing information is truly user-specific and cannot be obtained another way.

4. **Validate any claimed fix.**  
   A proposed fix is incomplete until it is checked against the actual failure mode.

5. **Leave useful artifacts even if unresolved.**  
   If a full fix is not possible, provide a precise and structured handoff.

---

## Investigation workflow

When this skill is active, follow this sequence.

### 1. Restate the exact failure
Summarize:
- what was attempted
- what failed
- the exact error, symptom, or incorrect behavior
- what is still unknown

Be concrete. Avoid vague summaries like “it doesn’t work.”

### 2. Inventory prior attempts
List the attempts already tried and group them by approach.

Then explicitly determine whether the next idea is:
- genuinely different, or
- just a minor variation of something that already failed

If it is only a minor variation, do not use it unless there is a strong reason.

### 3. Gather stronger evidence
Use available tools to inspect the problem more directly. Depending on the task, that may include:
- reading the exact error output
- checking logs
- reading source code or configuration
- reading documentation
- inspecting inputs and outputs
- checking assumptions about versions, file paths, permissions, API behavior, or runtime state
- isolating the smallest reproducible failing case

Prefer primary evidence over memory.

### 4. Generate multiple hypotheses
Produce 2–4 plausible explanations for the failure.

For each hypothesis:
- state what would be true if it were correct
- identify one fast check that would support or weaken it

Do not commit to a single explanation too early.

### 5. Test the highest-value hypothesis
Pick the next step that maximizes learning, not the one that merely feels active.

Good next steps usually:
- isolate the fault
- falsify a broad assumption
- narrow the search space quickly
- distinguish between two competing causes

### 6. Change direction when needed
If the current line of attack fails, switch to a **meaningfully different** category of approach, such as:
- from editing code to reading docs
- from guessing config to inspecting runtime output
- from full-system testing to minimal reproduction
- from implementation changes to environment verification
- from symptom-fixing to root-cause tracing

### 7. Verify the result
After applying a fix, check:
- whether the original error is gone
- whether the expected behavior now works
- whether related paths still behave correctly
- whether the fix introduces a nearby regression

Do not stop at “the code changed.” Stop at “the problem is resolved or precisely bounded.”

---

## Escalation rules

### After repeated failure without progress
If two attempts fail and neither produced meaningful new evidence:
- stop retrying small tweaks
- switch to structured diagnosis
- gather stronger evidence before making another fix attempt

### If the assistant is stuck in a loop
If multiple attempts are all variants of the same idea:
- explicitly name the loop
- abandon that path
- choose a different investigation angle

### If the assistant is about to ask the user for help
Before asking:
- verify that the missing information is truly user-specific
- summarize what has already been checked
- ask only for the minimum missing detail needed to proceed

Bad:
- “Can you check your environment?”

Better:
- “I checked the code path and config shape, but I still can’t verify which runtime version you’re using. Please share the exact runtime version so I can rule out a version-specific incompatibility.”

### If the assistant is about to give up
Do not give up until all of the following are true:
- the failure has been precisely described
- the main prior attempts are accounted for
- at least two plausible hypotheses were considered
- at least one materially different approach was tried
- the key assumptions were checked where possible
- any claimed environmental cause has some evidence
- a useful handoff can be written

---

## Completion standard

A task is only considered complete if one of these is true:

### A. Validated resolution
The assistant has:
- identified the likely root cause
- applied or proposed a fix
- verified that it addresses the original issue
- checked for obvious adjacent regressions

### B. High-quality unresolved report
If a full fix is not possible, the assistant should provide:
- the exact observed failure
- what was tested
- what was ruled out
- the most likely remaining causes
- the smallest next step that is most likely to unblock progress
- any user-specific information still needed

---

## Output style when this skill is active

Be calm, direct, and evidence-based.

Use language like:
- “Here’s what failed and what that rules out.”
- “The previous two attempts were both config-level changes, so I’m changing direction.”
- “I don’t yet have evidence that this is an environment issue.”
- “This fix still needs validation against the original error.”
- “I couldn’t fully resolve it, but I narrowed it to these two likely causes.”

Avoid:
- shame, pressure, or intimidation
- unsupported certainty
- repetitive apologies
- vague claims like “it should work now” without verification

---

## Lightweight checklist

Before concluding, quickly check:

- Did I clearly state the failure?
- Did I avoid repeating the same failed idea?
- Did I gather direct evidence?
- Did I test at least one materially different approach?
- Did I verify the fix?
- If unresolved, did I leave a useful handoff?

---

## Example trigger patterns

Activate this skill when the conversation looks like:
- failed implementation -> failed retry -> user dissatisfaction
- debug attempt -> guessed cause -> no evidence -> stalled progress
- proposed fix -> no verification -> risk of premature completion
- repeated small tweaks -> no new information -> looping behavior

---

## Summary

This skill is for recovery, not intensity.

It turns “keep trying” into:
- structured diagnosis
- better hypothesis selection
- materially different retries
- verified fixes
- useful failure reports