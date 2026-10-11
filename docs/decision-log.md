# Decision log

The decisions that shaped the tool, in the order they were made, and why.

| # | Date | Decision | Why |
|---|---|---|---|
| 1 | 2026-10-11 | **Jev decides whether a chat has PII. Haiku copies out the exact text, and only for chats Jev flags.** | Jev answers questions with probabilities but doesn't return text spans. Haiku can, but running it on every chat costs more, and most chats have no PII at all. |
| 2 | 2026-10-11 | **The models only ever see the chat text.** The label columns are kept out of the model calls and out of the traces. | So the scores can be trusted: there's no way for the answers to leak into the input. |
| 3 | 2026-10-11 | **Two risk tiers.** Card and ID numbers are high risk; name, address, phone, email and birth date are medium. | Leaking a card or ID number is the worst case. The others matter, but a miss isn't as costly. |
| 4 | 2026-10-11 | **Send a type to a person when Jev is under 95% sure (high risk) or under 80% sure (medium).** | Human review is the safety net for high-risk recall. 80% for medium risk keeps reviewers from being flooded. |
| 5 | 2026-10-11 | **Zero invented text, checked in Python.** Every item Haiku returns must appear word for word in the chat. | Redaction must never put words in someone's mouth. A code check is cheaper and more reliable than asking the model to be careful. |
| 6 | 2026-10-11 | **Cost under $0.50 per 1,000 chats. Speed isn't gated.** | This has to run on every chat. The chats are processed in batches, so a second or two per chat doesn't matter. |
| 7 | 2026-10-11 | **Dropped the overall "any PII?" question.** | It was confusing next to the 7 types, and the types already answer it. |
| 8 | 2026-10-11 | **Rewrote the Jev questions as a short question with "counts" and "doesn't count" examples (v2).** The examples are invented, not copied from the dev chats. | v1 packed each rule and all its exceptions into one long paragraph. The v2 format follows TypeSafe's guidance for Jev questions. Invented examples limit overfitting to the dev set. |
| 9 | 2026-10-11 | **Keep the prompts country-neutral.** No Singapore-only rules such as the NRIC letter-digits-letter format. | The tool is meant for use anywhere, even though the test data is Singapore-flavoured. |
| 10 | 2026-10-11 | **Turn spelled-out digits into numerals before Jev reads the chat.** Haiku still gets the original. | Jev is weak at reading numbers written as words, like "nine one two three four five six seven". The rewrite only affects what Jev sees, so the redaction still matches the real text. |
| 11 | 2026-10-11 | **Keep redaction separate, and only run it when asked.** | Detection and extraction are what's being tested. Rebuilding a redacted file on every run adds noise. |
| 12 | 2026-10-11 | **Tune on the dev set only. Open the test set once, for the final test.** | Scores on data you've tuned against are always optimistic. |
| 13 | 2026-10-11 | **Compare against Haiku alone and Presidio.** | Haiku alone shows whether the screening step earns its place. Presidio is the standard free tool, and what most teams would try first. |
| 14 | 2026-10-11 | **Run Haiku alone without the screening hint.** | The first run told Haiku that all 7 types were flagged, which pushed it to find something (50 clean chats changed). Dropping the hint brought that down to 38, so that's the version reported. |
| 15 | 2026-10-11 | **Run Presidio in Google Colab, on default settings.** | Windows Smart App Control blocked spaCy on my machine. Default settings show what you get out of the box. |
| 16 | 2026-10-11 | **Added a fourth check: over 90% of redacted items must be real PII.** | Haiku alone found every piece of PII but redacted 40 extra things, mostly first names. Without a precision check, a tool that redacts everything would pass. |
| 17 | 2026-10-11 | **Ship as a demo, not a product.** | It passes the gate on synthetic data. The next steps are more card and ID numbers, more languages, and real chats from other markets. |
