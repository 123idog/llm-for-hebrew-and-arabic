# 2. Language Guards on Output

A model that answers in Hebrew 99.9% of the time will, eventually, send a customer something else: a stray English word, an internal instruction, or a paragraph explaining in English why it's ending the conversation. Prompting reduces this. Only a guard stops it.

---

## What leaked

From a Hebrew WhatsApp sales bot, before the guard:
- single English words inside Hebrew sentences
- internal control phrases from the prompt, in Hebrew, sent verbatim to the customer
- once, a whole English message explaining why the bot was stopping

**Where it happens:** the **last** turn of a conversation. Closings, refusals and edge cases are where the model breaks character.

---

## Why the first filter failed

- It matched **exact** control strings, punctuation included. A slight paraphrase passed.
- It checked "contains Hebrew". The English leak **quoted the customer's Hebrew message**, so it passed.

---

## The guard that works

All three must pass before anything is sent:

1. **The reply contains the expected script.**
2. **It matches none of a loose blocklist** of control and meta phrases, in both languages, matched after normalising punctuation and spacing.
3. **Script ratio:** at least as many Hebrew letters as Latin letters. The English leak had about 11 Hebrew letters against 140 Latin ones, an obvious fail.

If the guard fails: retry once, then hand off to a human. **Sending nothing beats sending something off-topic**, as the business owner put it.

Test the guard against real past leaks *and* real good replies (prices, links, product names in Latin script) so it doesn't block legitimate messages. See [LLM Evaluation in Production](https://github.com/123idog/llm-evaluation-in-production) for how to build that assertion suite.

---

## Detect three scripts, not two

A monitoring check computed a Hebrew-vs-Latin ratio on call transcripts. Arabic calls have neither, so they were flagged as "translated into English". That produced a wrong diagnosis (a fifth of calls supposedly translated) and paid re-transcription of calls that were fine.

**Rule:** classify text by Hebrew, Arabic **and** Latin character counts. In a multilingual system, "not Hebrew" doesn't mean English.

---

## Monitor the rate, not just the incident

Log every guard firing. A guard rate that climbs from under 1% to over 10% within days means the model or the prompt is degrading, and you want to know before customers do.
