# 7. Evaluation with Native Speakers

The engineer building the system often doesn't speak the customer's language natively, or speaks one of the two languages but not the other. Evaluation has to be designed around the people who do.

---

## Method before bot

An Arabic sales bot was switched off after the Arabic-speaking team was unhappy with it. The rebuild runs in a fixed order:

1. **A written method with the Arabic team:** how they actually sell, what they ask first, what they never say.
2. **Native approval of example conversations**, good and bad, against that method.
3. **Only then** a new bot.

Asking the team to approve raw bot conversations without agreed rules was rejected: without rules, every review is an argument about taste. The hardest and most valuable input is **examples of good and bad conversations** from the team itself.

---

## Expert annotation turns into rubric rules

For call scoring in Arabic, the Arabic team's feedback on a handful of calls narrowed one rubric definition. The fix was tested in a clone on the exact calls they flagged, and fixed all of them, while one requested enforcement rule still wasn't achieved and was reported as such. Report partial results honestly; the team will check.

---

## Judges aren't reproducible, in any language

Re-rating the same recording with the same prompt at temperature 0 changed most dimensions and moved the overall score by up to ~1.7 points. Measure the noise before comparing agents, and report scores as bands. (More in [LLM Evaluation in Production, chapter 5](https://github.com/123idog/llm-evaluation-in-production/blob/main/docs/05-llm-as-judge.md).)

---

## Audit the automated checks too

An audit of a few hundred commitments the model extracted from calls (quotes like "I'll send you the price tomorrow") found:
- **a few real defects**, including one quote cut just before a negation, which turned a **refusal** into a "promise";
- the automated Hebrew keyword checks raised **11 flags, 9 of them false alarms**.

Lessons:
- In Hebrew and Arabic, negation can come after the phrase it negates in ways a naive cut misses. Always keep the full clause in extracted quotes.
- Treat automated language checks as **leads for a human**, not verdicts.

---

## Silent drops

New sales agents who weren't yet on the scoring allowlist were silently skipped, so their calls were never evaluated. Any allowlist in an evaluation pipeline needs a "not on the list" report, or coverage quietly shrinks as the team changes.
