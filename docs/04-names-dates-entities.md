# 4. Names, Dates and Entities

---

## A messaging profile name is not the customer's name

**Symptom:** CRM leads were named with emoji, nicknames and business names, often in Latin or Hebrew script while the customer wrote in Arabic. Sales reps opened every conversation with the wrong name.

**Cause:** AI name extraction worked on the happy path, but the **fallback** silently used the WhatsApp profile name. Existing leads were never updated when the customer later gave their real name in chat.

**Fix:**
- Order: name extracted by the model → a name stated in the current message → a stored name → the phone number. **Never** the profile name.
- Validate: 1–4 words, at most 40 characters.
- When a valid name is captured, update existing leads too.

---

## Clean names with Unicode classes, not ASCII rules

Strip everything except letters and combining marks (`\p{L}\p{M}`), spaces, apostrophes and hyphens. That keeps Arabic, Hebrew (with or without vowel marks) and Latin names intact and drops emoji. Emoji-only names fall back to the phone number.

**Find every writer before fixing the field.** The same bug lived in a second flow (ad-click enrichment). A leftover `#` in a name showed it couldn't have come from the bot, which is how the second writer was found.

---

## Give extraction prompts the date

**Symptom:** dates extracted from Arabic calls ("next March", "after the holidays") landed in the wrong year.

**Fix:** inject the **call date** into the extraction prompt. The model can't resolve relative dates without an anchor, and it won't tell you it guessed.

---

## Month-only answers

A customer answering "in month 12" (common phrasing in Arabic) left the date field empty, because the parser expected a full date. Resolve month-only answers to the 1st of that month, and keep the original wording in a notes field.

---

## Checklist

- [ ] Names: model-extracted or stated; never the profile name; validated; Unicode-safe cleaning.
- [ ] Every flow that writes the field is known.
- [ ] Extraction prompts include "today" or the source date.
- [ ] Partial dates have a defined resolution rule.
