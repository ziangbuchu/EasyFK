# Rationale

## What was wrong with `Helps find bugs in code`

The old description fails on every dimension Claude uses to auto-invoke a skill:

1. **No trigger surface.** It names no artifact, no verb, no symptom. "Bugs" and "code" match nothing in particular, so the matcher has nothing concrete to latch onto when the user pastes a stack trace or says "it throws NullPointerException."
2. **Ambiguously broad.** Literally every coding turn involves code that might have bugs. A matcher that interprets this generously would fire on every edit; a conservative matcher (which is what Claude actually does) gives up and fires on nothing. The result is the observed behavior: silence until `/bug-hunter` is typed.
3. **No negative space.** It doesn't say when *not* to run. Without exclusions, the model can't distinguish "writing new code" from "debugging broken code," so it plays safe and skips.
4. **Wrong verb framing.** "Helps find bugs" is passive and generic. Skill descriptions that auto-invoke well tend to name a **situation** ("when the user pastes X") and a **deliverable** ("returns the root cause at file:line").

## What I changed

The new description does four things the old one did not:

- **Names the trigger artifacts explicitly**: stack traces, error messages, exception names, crash logs, failing test output, described symptoms of the form "X throws / returns wrong / hangs / crashes when Y." These are the exact tokens the matcher will see in the user's message when this skill should fire.
- **States the direction of work**: "signal backward into the codebase." This distinguishes debugging from forward work (features, refactors) without having to enumerate every non-debugging activity.
- **Adds a SKIP clause.** Explicitly rules out feature work, refactoring clean code, style/perf tuning, code review without a reported symptom, and the adjacent case of "user already knows the line, just wants the patch written." The last exclusion matters — it prevents the skill from stealing turns from normal editing work after a bug has already been localized.
- **Splits into `description` + `when_to_use`.** The description carries the scope and SKIP rules; `when_to_use` is a tight restatement focused purely on trigger signals, which is what the matcher weights most. Both are well under the ~1.5k combined cap.

## Expected triggers

Should match:
- "I'm getting `TypeError: Cannot read property 'x' of undefined` when I click submit — why?"
- Pasted stack trace with no other context.
- "This test fails: [output]. Find the cause."
- "The API returns 500 after the third request, here's the log."
- "Users report the modal sometimes doesn't close. What's wrong?"

Should NOT match:
- "Add pagination to the users list."
- "Refactor this function to use async/await."
- "Review my PR."
- "Is this the idiomatic way to write this in Rust?"
- "I found the bug on line 42, help me write the fix." (symptom localized — skill's job is done; patching is normal editing.)

## Residual risk

The one borderline case is "help me debug this" with no artifact attached. The description leans toward requiring a concrete signal, so that phrasing may not trigger. That's the right trade: firing on vague "debug help" is what caused the over-broad matcher to get confused in the first place. If the user provides the artifact on a follow-up turn, the skill will fire then.
