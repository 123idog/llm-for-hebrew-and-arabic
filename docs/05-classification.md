# 5. Classification in Arabic and Hebrew

Case study: one WhatsApp number receives both **customers** and **job applicants**. An LLM classifies the first message and routes it to the sales bot or the recruitment flow.

---

## One misclassification silenced a customer

**Symptom:** a garbled Arabic message from a real customer was classified as a job inquiry. They received the recruitment message twice, and then nothing, permanently.

**Numbers:** about 0.5% of classified turns, a handful of people. But each case is a lost sale, and nobody notices.

**Causes (two):**
- the classifier mistake itself;
- the real bug: once in "recruit mode" there was **no exit**. The gate had nothing wired after it.

**Fix:**
- a **deterministic customer-word veto** that runs before the LLM verdict is trusted;
- a new explicit class for the ambiguous case, instead of forcing a binary choice;
- the gate became *recruit AND NOT customer-intent*, with a path back.

Design rule: **a classification mistake must be recoverable**. Ask what happens to the person after a wrong label, not only how often it occurs.

---

## Arabic has no usable `\b`

JavaScript's `\b` is defined for ASCII word characters. With Arabic it either never matches or matches in the wrong place. A bare stem meaning "workers" matched **inside** words meaning "usage" and "business".

**Fix:** anchor stems yourself: start of text, whitespace, or Arabic/Latin punctuation on each side. The same applies to Hebrew, where prefixes (ו, ה, ב, ל, מ, ש) attach to the word: decide explicitly whether a prefixed form should match.

---

## Widening a lexicon causes regressions

**Symptom:** to catch more applicants, phrase-level job terms ("sales representative") were loosened into bare stems ("representative", "worker"). Two customers then got the job message. One was a **typo** of "three families".

**Fix: a two-tier lexicon.**
- **Strong** terms: trusted for everyone.
- **Weak** terms: count only for someone already flagged as an applicant.

---

## Corpus first, then rules

1. Collect **every real positive** (all genuine job messages) and a set of hard negatives (customer messages with ambiguous words, typos, dialect).
2. Write the rule.
3. Run it on both. Re-run after **every** edit.

The corpus showed that a few of the real job messages used a stem the original lexicon missed, and that the "simple" fix would have blocked real customers. The test set grew to a few dozen cases, all passing before each deploy.

---

## Exact matching of single phrases is brittle

- **Handoff detection** looked for one specific Arabic sentence. When the model phrased it differently, complaints restarted the sales funnel. It became an OR over the three handoff sentences the prompt actually uses.
- **Acknowledgements** ("ok", "great") ended conversations because they were matched as **substrings** inside longer messages. Fixed with whole-message matching, unit-tested.

In both languages, short words collide with longer words more often than in English. Prefer whole-message or explicitly anchored matching.
