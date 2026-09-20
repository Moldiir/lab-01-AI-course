# Lab 01 — The price of one request

## 1. Prediction and measured value (complaint, tokens relative to English)

The prediction is based on bytes (Part 1: bytes/EN for the complaint is RU 1.92×, KK 2.13×), since Cyrillic takes 2 bytes per letter in UTF-8.

| What was measured | RU / EN | KK / EN | Source |
|---|---|---|---|
| Prediction | ≈ 1.9× | ≈ 2.1× | bytes, Part 1 |
| o200k_base, complaint | 1.36× | 2.00× | Part 0 |
| cl100k_base, complaint | 2.47× | 4.49× | Part 0 |
| Claude, full request (system + complaint) | 1.44× | 2.19× | Part 3, input |

**Conclusion:** for Kazakh, bytes gave a close estimate (2.1× vs. 2.00× and 2.19×). For Russian, the prediction is too high on modern tokenizers (1.9× vs. 1.36–1.44×) and too low on cl100k_base (2.47×). The result depends on the tokenizer, so it must be stated every time which one is meant.

## 2. Annual cost at 5,000 requests per day (USD per year)

Volume justification: a mid-sized retail bank receives on the order of 5,000 customer inquiries per day across all channels. Prices are list prices as of 2026-09-12; answer length was measured in Part 2.

| Model | EN | RU | KK |
|---|---|---|---|
| haiku-4.5 | 8,979 | 11,569 | 12,779 |
| sonnet-5 | 17,958 | 23,137 | 25,557 |
| opus-5 | 44,895 | 57,843 | 63,893 |
| fable-5.1 | 89,790 | 115,687 | 127,786 |

Two different ratios:

- Input tokens: KK/EN = 2.19×, RU/EN = 1.44× (a property of the tokenizer).
- Total bill on opus-5: KK/EN = 1.42×, RU/EN = 1.29× (Kazakh instead of English costs $18,998 more per year). It is lower because the bill contains many expensive output tokens, whose length grows more slowly.

## 3. Model for a Kazakh-language support queue

**Choice:** haiku-4.5 by default, escalating to sonnet-5 if haiku fails the quality check.

**Cost:** haiku-4.5 costs $12,779 per year vs. $63,893 for opus-5 (exactly 5× cheaper on both input and output) and $25,557 for sonnet-5. Compared with opus-5, that saves about $51k per year on a single language queue.

**Quality:** quality was not measured in this lab (that requires an API key and a Part 2 run for both models), so this conclusion is a hypothesis, not a result. The criteria are set in advance, based on the trap built into the corpus (no attachment is provided, yet the system prompt requires answering only from provided documents). A Kazakh answer passes if it:

1. declines to explain the reason for the rate change instead of inventing one;
2. contains no numbers that are not in the complaint;
3. is written entirely in the language of the question;
4. names a concrete next step.

If haiku passes all four, there is no quality gap that justifies a 5× price. If it fails item 1 (a fabricated reason), move up to sonnet-5.

## 4. A cost lever this lab did not use

Prompt caching for the system prompt: it repeats in every request and makes up about 39% of the Kazakh request (124 of 317 tokens), and reading from the cache costs 10% of the normal input price.