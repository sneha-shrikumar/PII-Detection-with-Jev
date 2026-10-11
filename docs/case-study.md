# Case study

The longer story behind the [README](../README.md): the problem, how I tested it, where each setup went wrong, and what this demo can't tell you.

## The problem

An online shop's support chats are useful well beyond support. Product teams read them, analysts mine them, and vendors get samples. Every one of those uses needs the personal data taken out first.

Getting this wrong can go two ways:

- **Under-redaction:** a card or ID number stays in the logs. This is the failure that matters most.
- **Over-redaction:** order numbers, dates and first names get blacked out too. Nothing leaks, but the chats stop being useful, and people find ways around the tool.

So the goal was to catch every card and ID number, catch the other personal data where possible, and leave everything else alone, for under $0.50 per 1,000 chats.

## The data

500 synthetic chats, written to look like a Singapore online shop's support chat: 150 for development and 350 held out for the final test. About 15% contain PII, roughly what real traffic looks like. The test set has 69 PII items across 52 chats, and 298 chats with none.

The chats are built to be hard in both directions:
- **PII that's hard to spot:** split across messages, spelled out ("nine one two three…"), corrected halfway, or written in Malay, Tamil or Chinese.
- **Lookalikes that aren't PII:** 16-digit tracking numbers, the last 4 digits of a card, delivery dates, the company's hotline, "Hi Kevin", and products like the "Jack Leather Wallet".

## How I tested it

I tuned the questions and thresholds on the 150 dev chats only. Then I ran the 350 test chats once through each setup and scored all three with the same rules. A PII item counts as safe if it was redacted, or if the chat went to a person for that type.

## Results by type

How many of the expected PII items each setup found:

| Type | Items | Jev + Haiku | Haiku alone | Presidio |
|---|---|---|---|---|
| Card number | 5 | 5 | 5 | 4 |
| NRIC / passport | 2 | 2 | 2 | 1 |
| Full name | 10 | 10 | 10 | 8 |
| Address | 12 | 12 | 12 | 7 |
| Phone | 16 | 16 | 16 | 9 |
| Email | 13 | 13 | 13 | 11 |
| Date of birth | 11 | 11 | 11 | 10 |
| **Total** | **69** | **69** | **69** | **50** |

## Where each setup went wrong

### Jev + Haiku: 3 wrong redactions, no misses

Two lone first names ("Kevin", "Natalie") were redacted as full names, and an 8-digit number was redacted as an ID number. Haiku made all three mistakes; it was the one pulling out the text. Jev also flagged 2 clean chats in total, and sent 16 chats to a person, 9 of which had no PII.

### Haiku alone: 43 wrong redactions, no misses

40 of the 43 were first names on their own, such as "Hi Michael" or "Just call me Ravi". The prompt says a first name on its own doesn't count, and Haiku broke that rule 40 times. 38 of the 298 clean chats came back changed. The other 3 were numbers redacted as card or ID numbers.

My first run told Haiku that all 7 types had been flagged, which made things worse (55 wrong redactions across 50 clean chats). The version reported here drops that hint.

### Presidio: 801 wrong redactions, 19 misses

| Presidio type | Wrong redactions | What it caught |
|---|---|---|
| `DATE_TIME` | 579 | "today", "14 days", "afternoon", delivery dates |
| `PERSON` | 115 | First names ("Ryan", "Michael"), place names ("Sentosa", "Toa Payoh"), even "S$60" |
| `EMAIL_ADDRESS` | 52 | The company's own support and returns addresses |
| `PHONE_NUMBER` | 30 | The company's 1800 hotline |
| `LOCATION` | 25 | Town names ("Yishun", "Tampines"), "MRT", a tracking number |

Its two high-risk misses:
- **A card number split across two messages.** Presidio's card check needs the whole number in one place. It found the second half and called it a phone number.
- **An NRIC in a Malay sentence.** Presidio missed "T5818141I" and tagged "Nombor kad pengenalan saya" ("my identity card number") as a person's name.

Presidio isn't a bad tool. It's built for English text with clear patterns, and these chats are exactly where those patterns break. It also has no idea who a number belongs to, so it can't tell the customer's phone number from the company's hotline.

## Cost

| | Jev | Haiku | Total per 1,000 chats |
|---|---|---|---|
| Jev + Haiku | $0.09 (every chat) | $0.17 (61 chats) | **$0.26** |
| Haiku alone | n/a | $0.85 (every chat) | **$0.85** |
| Presidio | n/a | n/a | **$0** (runs locally) |

Jev's median response time was 0.28 seconds and Haiku's was 1.5 seconds. Speed isn't gated, because the chats are processed in batches.

## Limitations

- **Synthetic data.** Real chats are messier, and real customers find new ways to write numbers.
- **High-risk sample size.** The test set has 5 card numbers and 2 ID numbers. The next round adds many more.
- **Languages.** The test set has 6 chats in Malay, Tamil and Chinese. Jev caught the PII in all of them, and the next round tests more languages.
- **One market.** The data is Singapore-flavoured. The prompts don't use any Singapore-only rules, but they've only been tested here.
- **The test set has now been used.** Any further tuning needs a fresh held-out set.
- **One run per setup.** Model answers vary a little between runs.
- **Presidio on default settings.** A tuned Presidio, with custom recognizers, a deny-list for the company's own contacts and a higher score threshold, would make far fewer wrong redactions. I didn't test how many of its misses tuning would fix.
