# 3. Prompting and Register

---

## Write natively; never translate a tuned prompt

A Hebrew prompt that took weeks of conversations to tune encodes register, phrasing and local expectations that don't survive translation. Write each language's prompt natively, with someone who sells in that language.

When cloning a bot to a new version: **copy prompts verbatim**. "Improving" the wording in passing is how regressions sneak in. Change prompts only as deliberate, tested edits.

---

## Arabic is many registers

There isn't one "Arabic". A Gulf customer, a Levantine customer and a formal Modern Standard Arabic reply each read differently. What worked:
- a **persona with a specific regional dialect**, matching the human team the customer would otherwise reach;
- a strict **step-by-step procedure** in the prompt, so the dialect stays natural while the flow stays controlled;
- fixed sentences (declines, handoffs) written by a native speaker, in that dialect, and used verbatim.

---

## Say what's forbidden, not only what's allowed

**Symptom:** the Arabic bot happily agreed to build a combination the business doesn't sell.

**Cause:** the prompt listed what was supported. It never said what wasn't.

**Fix:** an explicit allowlist **plus** a "never, even if the customer asks" rule, with a fixed decline sentence in the customer's dialect. Verified live afterwards.

---

## A prompt fix needs a strong enough model

**Symptom:** a bug kept returning after a correct prompt fix: hallucinated English in the Hebrew bot, missed fields in the Arabic bot.

**Cause:** that node ran on a lite or preview model. Small models follow Hebrew and Arabic instructions far less reliably than English ones.

**Fix:** run customer-facing generation and extraction on the strong model, with the cheaper one as **fallback**, not primary. Before rewriting a prompt for the third time, check which model is actually running it.

Related: when a JSON template shows a sample value (`"is_qualified": true`), weak models copy it. Show **null** defaults in templates.

---

## Few-shot examples from your own output lock in drift

**Symptom:** the rule said "3–5 sentences", yet Hebrew replies stayed at 60–80 words for weeks.

**Cause:** the few-shot examples were drawn from past published output, and the prompt told the model to match them. Concrete examples outvote abstract rules, and each cycle reinforces the drift.

**Fix, in one change:**
- tighten the rule and name the flaw explicitly;
- remove length (or whichever trait drifted) from what the examples govern;
- use fewer examples (6 → 3);
- rotate them (e.g. by a hash of the record ID);
- afterwards, check that no **new** uniform pattern has taken over.

---

## Ground generated content in real examples

For Hebrew SEO content, quality jumped when the generator received the two best existing articles on the topic as style references, plus a short brand document. The model copies tone from examples much better than from adjectives.

---

## Match the conversation's dominant language

A one-line English message inside a Hebrew conversation isn't a request to switch languages. Bots (and assistants) that flip on every short English phrase feel broken to the user. Switch only on a full message in the other language, or when asked.
