# 1. Encoding and Plumbing

The first Hebrew/Arabic bugs aren't in the model. They're in the pipes between your editor, your shell, your automation platform and the API. They almost never raise an error.

---

## A shell turned a live bot's prompt into `???`

**Symptom:** a routine GET → edit → PUT of a live chatbot workflow replaced every Hebrew character (over 10k characters of prompts, keyword regexes and month names) with `?`. The bot kept running, now with a broken prompt.

**Cause:** an older Windows shell's HTTP client didn't reliably use UTF-8 for the response or the request body. The JSON serializer kept the Hebrew, then the transport encoded it as ASCII.

**Fix:**
- Don't send RTL payloads from a shell one-liner. Write a UTF-8 (no BOM) file with a script and send it with `curl --data-binary @file`, or use a Node/Python client.
- Never put a long Hebrew or Arabic prompt **inline** in a command string; the shell can mangle it before the request is even built.

**Detect:** count code points in the Hebrew (U+0590–05FF) or Arabic (U+0600–06FF) blocks before and after every write. Any drop means corruption. Keep version history so you can roll back.

---

## Hebrew model output stored as Chinese

**Symptom:** about 1% of Hebrew call evaluations reached the CRM as Chinese characters. No errors anywhere.

**Cause:** the HTTP client **guessed** the response charset and decoded Hebrew UTF-8 bytes as GBK, which maps each Hebrew letter onto a Chinese character. The tell: hard-coded Hebrew strings written into the same field stayed Hebrew; only the model's text was damaged.

**Fix:** fetch model responses as raw bytes and decode explicitly as UTF-8. Already-damaged text is reversible (`encode('gbk').decode('utf-8')`), so repair beats re-running.

**Debugging note:** compression settings looked like the obvious suspect and weren't. A cheap probe call ruled them out in minutes.

---

## Regexes that lose their backslashes

**Symptom:** an Arabic keyword regex matched nothing. `\s` had become a literal `s`.

**Cause:** writing the file through a shell heredoc stripped backslashes. The result was still valid JavaScript, so nothing complained.

**Fix:** write regex code with a proper file-writing tool, and print `RegExp.source` after loading to check it's what you meant. Arabic and Hebrew regexes are hard to eyeball, so verify them mechanically.

---

## The model mangles percent-encoded Hebrew URLs

**Symptom:** an SEO article generator corrupted internal links whose slugs were Hebrew. One letter changed and the link 404'd.

**Cause:** the model regenerates URLs token by token, and long percent-encoded strings are easy to get almost right.

**Fix:** a deterministic step after generation: check every internal link against the real URL list, repair close matches, and turn unknown links into plain text. Never trust a model to copy a URL.

---

## Checklist

- [ ] UTF-8 explicitly, at every hop: file, shell, HTTP client, database, CRM.
- [ ] Code-point count before/after every write of RTL content.
- [ ] Payloads built by scripts and sent as files, not inline strings.
- [ ] Regexes verified via their compiled source.
- [ ] URLs, IDs and codes copied by code, not by the model.
