# AI Failure-Mode Notes

Short write-ups of real failures I've hit while building with and evaluating LLMs and AI
coding agents — analyzed the way an evaluator would: the **failure mode**, the **signal**
that catches it, and the **mitigation**.

My working premise across all of my projects is one sentence: **I treat AI output as
untrusted input.** I build the validation, bounding, fallback, and audit layers around
unreliable models — and I keep notes on where they fail, because noticing and categorizing
failure is the core of evaluation work.

These are real cases from my own work.

---

## 1. Output that ignores target constraints

**What happened.** Machine-translating a game's UI from Japanese to English, the output was
reasonable English but wouldn't fit the engine's fixed half-width character cells. The model
translated for *meaning* with no awareness of the display's byte and pixel budget, so
"correct" output was still unusable. Brute-forcing a narrower custom font sort-of worked but
looked wrong; the real fix was rebuilding the English glyphs on the engine's native 24px
letterforms, so the text fit and looked native to the game.

**Failure mode.** The model optimizes the objective it was given (meaning) and is blind to
the hard constraints of the target system (rendering width, byte budget).

**Signal.** A "correct" output is not a *usable* one until it satisfies the target's
constraints — correctness in isolation is not sufficiency.

**Mitigation.** Make the width/byte budget a gate the translation must pass, and adapt the
rendering — engineering the font onto the engine's native letterforms — rather than tolerating
overflow.

---

## 2. Untrusted structured output

**What happened.** In a live lead pipeline, a model qualifies each inquiry into strict JSON
(priority, score) that feeds a CRM and a sales alert. Left unchecked, a malformed response or
an out-of-range score would flow straight into the system of record — one bad generation
corrupting real business data.

**Failure mode.** Treating a model's structured output as trusted because it *usually* comes
back well-formed.

**Signal.** Never let raw model output cross into a system of record; structural plausibility
is not a guarantee.

**Mitigation.** A JSON schema constrains the model, a validator checks every field, and a
deterministic fallback takes over on any malformed or out-of-range output — graceful
degradation instead of a poisoned pipeline.

---

## 3. Plausible-but-wrong code

**What happened.** Working with AI coding agents every day, the most common failure isn't a
crash — it's confident code that reads correctly but calls a non-existent API, mishandles an
edge case, or quietly changes behavior. It compiles, it looks reasonable, and it's wrong.

**Failure mode.** Fluency is the default: the model produces plausible code regardless of
correctness, and "looks right" masks subtle defects.

**Signal.** "Looks right" is not a signal — appearance and correctness are independent.

**Mitigation.** Treat every AI-generated change as untrusted input — verify against tests,
docs, and real runtime behavior before it ships, and never merge on appearance.

---

*More notes as I collect them. If you are evaluating me for AI-evaluation or LLM-reliability
work and want to talk through any of these, I am at zackwildebusiness@gmail.com.*
