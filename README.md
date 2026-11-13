# LLMs for Hebrew and Arabic in Production

**What breaks when customer-facing AI speaks Hebrew or Arabic, and the fixes that held up in production.**

Most LLM tooling, examples and evaluation habits assume English. Put a model in front of Hebrew- and Arabic-speaking customers and new failure modes show up: text that silently turns into `???` or Chinese characters, regexes that don't see word boundaries, English leaking into replies, names the TTS can't pronounce, classifiers confused by dialect and typos. This repo collects the ones I hit running WhatsApp agents, call analysis, content generation and voice in both languages.

---

## Contents

| # | Chapter | What's inside |
|---|---|---|
| 1 | [Encoding and Plumbing](docs/01-encoding-and-plumbing.md) | `???` from a shell, Hebrew decoded as Chinese, regexes losing backslashes, mangled RTL URLs |
| 2 | [Language Guards on Output](docs/02-language-guards.md) | Stopping English and control-phrase leaks, script ratios, three scripts not two |
| 3 | [Prompting and Register](docs/03-prompting-and-register.md) | Native prompts not translations, dialect personas, explicit "never" lists, model strength, few-shot drift |
| 4 | [Names, Dates and Entities](docs/04-names-dates-entities.md) | Profile names aren't names, Unicode-safe cleaning, relative and month-only dates |
| 5 | [Classification in Arabic and Hebrew](docs/05-classification.md) | No usable `\b`, two-tier lexicons, deterministic vetoes before the LLM |
| 6 | [Voice: Speech-to-Text and Text-to-Speech](docs/06-voice.md) | Transcription recipe, repetition loops, language support per model, pronouncing names |
| 7 | [Evaluation with Native Speakers](docs/07-native-evaluation.md) | Method before bot, expert annotation, audits of automated checks |

---

## Six rules

1. **Count the Hebrew/Arabic characters before and after every hop.** Encoding bugs return 200.
2. **Write prompts natively, per language and market.** Don't translate a tuned prompt, and don't paraphrase one when you copy it.
3. **Guard the output's language, not just its content.** Check script ratios, not "contains Hebrew".
4. **Arabic regex has no `\b`.** Anchor on whitespace and punctuation yourself.
5. **A deterministic veto beats a smarter prompt** for the classification mistakes that cost you a customer.
6. **Native speakers define "good".** Automated language checks are leads to review, not verdicts.

---

## About

Written by **Ido Guy**, AI & Automation Engineer. I design end-to-end AI solutions for sales and operations teams in Hebrew- and Arabic-speaking markets.

[LinkedIn](https://www.linkedin.com/in/ido-guy) · Also: [WhatsApp AI Agent Design](https://github.com/123idog/whatsapp-ai-agent-design) · [LLM Evaluation in Production](https://github.com/123idog/llm-evaluation-in-production) · [n8n Production Playbook](https://github.com/123idog/n8n-production-playbook)

*All examples are generalised and numbers are rounded. No client data, prompts or identifiers are included.*
