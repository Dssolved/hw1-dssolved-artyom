# HW1 submission

**Name: Artyom Ostanin**

**Student ID: 66803**

**Group: CSS4007-ENG-8**

**Repository: https://github.com/Dssolved/hw1-dssolved-artyom**

## AI tool disclosure


> Used Claude (Anthropic) to help implement the TODO functions across all three sublabs, debug one OpenRouter error 
> (429 rate limit), and draft the written analysis below from my own run output.

---

## Sublab Easy — the registration bot and its bill

**How I laid the catalogue out inside the system prompt, and why:**

> One line per course as plain text: code, title, credits, prerequisites,
> schedule, seats used/total/remaining, instructor. Chose plain text over JSON
> because the model only needs to read the facts, not parse structured data,
> and prose is easier to verify by eye. Put the "refuse anything not listed"
> rule at the end under its own heading, worded strongly (MUST, NEVER), so it
> would not get lost among the course list.

**My turn 5 (Kazakh or Russian):**

> Какой список доступных курсов для студента 3 курса сейчас?

### Run 1 — OpenAI, `gpt-5.6-luna`

| Turn | Input tokens | Output tokens | Cost $ |
|---|---|---|---|
| 1 | 674 | 552 | 0.000797 |
| 2 | 896 | 95 | 0.000293 |
| 3 | 971 | 56 | 0.000261 |
| 4 | 1046 | 36 | 0.000252 |
| 5 | 1104 | 314 | 0.000598 |
| **total** | | | 0.002202 |

### Run 2 — OpenRouter, `google/gemma-4-26b-a4b-it:free`

| Turn | Input tokens | Output tokens | Cost $ |
|---|---|---|---|
| 1 | 767 | 471 | 0.000000 |
| 2 | 1257 | 112 | 0.000000 |
| 3 | 1387 | 114 | 0.000000 |
| 4 | 1517 | 49 | 0.000000 |
| 5 | 1582 | 466 | 0.000000 |
| **total** | | | 0.000000 |

### Turn 4, verbatim

The turn where you asked for CSS-4090, which does not exist. Paste both replies
exactly as they came back — do not tidy them.

**OpenAI:**

```
I can't add CSS-4090 Quantum Machine Learning because it does not exist in the Narxoz University 2026-FALL course catalogue.
```

**OpenRouter:**

```
I am sorry, but I cannot add that course to your schedule. CSS-4090 "Quantum Machine Learning" does not exist in the Narxoz University course catalogue for the 2026-FALL term.
```

### Written answers

**1. The two providers used almost identical code. What actually changed, and
what did not?**

> Only the client setup changed: `base_url` points to OpenRouter instead of
> OpenAI, plus a different API key and model name. Everything else is
> identical — `chat.completions.create(...)`, the `messages` list format, and
> how the reply and `.usage` fields are read out. OpenRouter speaks the same
> OpenAI-compatible wire format, so no other code had to change.

**2. Why did the input token count climb on every turn when your questions
stayed roughly the same length? Use the numbers from your own table. What
happens to the bill at fifty turns?**

> Input tokens on OpenAI went 674 → 896 → 971 → 1046 → 1104, and on OpenRouter
> 767 → 1257 → 1387 → 1517 → 1582. Each turn resends the entire history —
> system prompt plus every prior user and assistant message — because the
> model has no memory between calls. The previous turn's reply gets billed
> again as input on the next call. At fifty turns the history keeps growing
> every turn, so cost per turn keeps rising and the bill grows faster than
> linearly with turn count, even though each new question is short.

**3. Turn 4: did the bot refuse, or did it invent CSS-4090?** If it refused, what
in your system prompt held the line? If it invented, what did it make up —
credits, a room, an instructor?

> Both models refused, correctly. Neither invented credits, a schedule, or an
> instructor for CSS-4090. What held the line was the explicit "STRICT RULE"
> block at the end of the system prompt stating the course list is complete
> and the model must refuse and must never invent a course not on it.

**4. Where else was either bot wrong?** Turn 2 asks for two courses that meet at
the same hour; two courses in the catalogue are full. Did the bots notice?

> Both models caught the Tuesday 09:00–10:50 collision between CSS-4007 and
> CSS-4102 already in turn 1, before I even asked to register. Both correctly
> flagged CSS-4400 as full (0 seats) and CSS-3011/CSS-3005/MAT-2020/ECN-2101
> as already completed. I did not find a factual error in either run; Gemma's
> turn 3 answer was slightly more verbose than needed but not incorrect.

---

## Sublab Medium — one task, six models

| Model | Exact | Failed | Tokens | Cost $ |
|---|---|---|---|---|
| google/gemma-4-26b-a4b-it:free | 4 | 0 | 1769 | 0.00000 |
| qwen/qwen3.8-27b | 8 | 0 | 49045 | 0.15306 |
| deepseek/deepseek-v4-flash-0731 | 8 | 0 | 34807 | 0.00954 |
| gpt-5.6-luna | 6 | 0 | 3813 | 0.00366 |
| gpt-5.6-terra | 6 | 0 | 2321 | 0.01872 |
| gpt-5.6-sol | 5 | 0 | 3005 | 0.06732 |

### Which error types did each model repair?

| Error type | gemma | qwen | deepseek | luna | terra | sol |
|---|---|---|---|---|---|---|
| kaz_to_rus | no | yes | yes | yes | yes | yes |
| latin_homoglyph | partial | yes | yes | yes | yes | yes |
| drop_hyphen | no | yes | yes | yes | yes | yes |
| join_words | yes | yes | yes | yes | yes | yes |
| double_letter | yes | yes | yes | yes | yes | yes |

**The `latin_homoglyph` row: what happened?** Describe what you observed. The
explanation is Sublab Harder's job, not this one's.

> Two sentences carry this error: KZ-03 (pure homoglyph) and KZ-08 (homoglyph
> + doubled letter). Five of six models fixed both correctly. Gemma fixed
    > KZ-08 but failed KZ-03: instead of restoring "Алаяқтарға" it produced
    > "Алақаттарға" — wrong letters, not just an unfixed homoglyph. GPT-5.6-sol
    > restored the correct letters on KZ-03 but added a comma and question mark
    > not present in the original, so it scored non-exact despite a substantively
    > correct fix.

**Where a model returned good Kazakh that was not identical to the original,
say so here.** Exact match is not correctness.

> On KZ-04 and KZ-05, gpt-5.6-luna, gpt-5.6-terra, and gpt-5.6-sol all
> restored the correct letters but appended a "?" that is not in the
> published original. The correction itself is right; only the added
> punctuation makes it score as non-exact. Gemma did the same on KZ-04. Sol's
> KZ-03 output (letters correct, extra comma and "?") is the same pattern.

**Cheapest model that was good enough, and why:**

> `deepseek/deepseek-v4-flash-0731`: 8/8 exact at $0.00954, versus $0.15306
> for the same 8/8 from qwen (qwen burns huge amounts of hidden
> reasoning/output tokens — e.g. 18,145 output tokens for one sentence in
> KZ-08 — despite a ~170-token input each time). Gemma is free but only 4/8
> exact, including a genuine content failure on KZ-03, so it is not reliable
> enough despite the zero cost. Deepseek gives the best accuracy-per-dollar
> of the six.

---

## Sublab Harder — open the tokenizer

### A. What a language costs

**`cl100k_base`:**

| Language | Tokens | Chars | Tok/char | × English | $ per 1,000 sentences |
|---|---|---|---|---|---|
| kk | 200 | 263 | 0.760 | 3.75 | 0.1667 |
| ru | 129 | 277 | 0.466 | 2.30 | 0.1075 |
| en | 59 | 291 | 0.203 | 1.00 | 0.0492 |

**`o200k_base`:**

| Language | Tokens | Chars | Tok/char | × English | $ per 1,000 sentences |
|---|---|---|---|---|---|
| kk | 84 | 263 | 0.319 | 1.58 | 0.0700 |
| ru | 74 | 277 | 0.267 | 1.32 | 0.0617 |
| en | 59 | 291 | 0.203 | 1.00 | 0.0492 |

($ per 1,000 sentences uses `cost_per_thousand` at gpt-5.6-sol's $5.00/M
input rate, average sentence length per language from the six triplets.)

### B. What a homoglyph does

| Sentence id | Foreign char (index, name) | Tokens correct | Tokens corrupted | Δ | Diverges at |
|---|---|---|---|---|---|
| KZ-03 | 0 'A' LATIN CAPITAL LETTER A; 2 'a' LATIN SMALL LETTER A; 5 't' LATIN SMALL LETTER T | 16 | 20 | +4 | 0 |
| KZ-08 | 1 'o' LATIN SMALL LETTER O; 3 'a' LATIN SMALL LETTER A; 9 'T' LATIN CAPITAL LETTER T | 21 | 24 | +3 | 1 |

**Token pieces around the divergence:**

```
KZ-03
correct  : ['А', 'лая', 'қ', 'тарға', ' ақша']
corrupted: ['A', 'л', 'a', 'я', 'қ']

KZ-08
correct  : ['Д', 'он', 'аль', 'д', ' Т', 'рамп']
corrupted: ['Д', 'o', 'н', 'a', 'л', 'ль']
```

### C. Did it get better?

| Language | cl100k_base | o200k_base | Change |
|---|---|---|---|
| kk | 0.760 | 0.319 | -0.441 |
| ru | 0.466 | 0.267 | -0.199 |
| en | 0.203 | 0.203 | 0.000 |

### Written answers

**1. What is the Kazakh tax?** The ratio against English in both encodings, the
dollar figure from A, and how much it changed between the two tokenizers.

> Kazakh costs 3.75x English under `cl100k_base` and 1.58x English under
> `o200k_base` — $0.1667 vs $0.0492 per 1,000 sentences under the older
> encoding, $0.0700 vs $0.0492 under the newer one. The Kazakh tax dropped
> from 3.75x to 1.58x, a 2.37x improvement, because `o200k_base` has a much
> larger vocabulary with more Cyrillic multi-character merges. It did not
> disappear: Kazakh still costs 58% more than English even in the newer
> tokenizer, and English's ratio never moved (1.00x in both), since English
> was already well covered by `cl100k_base`.

**2. Why did the models repair `kaz_to_rus` but struggle with
`latin_homoglyph`?** Both are single-letter substitutions and both look almost
identical on screen. Use your token streams from B as the evidence. Say what the
model actually received in each case.

> In `kaz_to_rus` (KZ-01), every letter is still a valid Cyrillic character —
> just the wrong Kazakh-specific one swapped for a plain Russian look-alike
> (ә→а, қ→к, etc). The tokenizer still sees real Cyrillic text and produces a
> token stream the model can pattern-match against ordinary Kazakh/Russian
> text it has seen a lot of, so the fix is just a spelling correction.
>
> In `latin_homoglyph`, the swapped characters are a different Unicode script
> entirely (LATIN, not CYRILLIC — as `foreign_chars` shows directly: 'A', 'a',
> 't' in KZ-03; 'o', 'a', 'T' in KZ-08). The token stream diverges from
> position 0 (KZ-03) or 1 (KZ-08): the correct word "Алаяқтарға" tokenizes as
> `['А', 'лая', 'қ', 'тарға', ...]`, four clean pieces, while the corrupted
> version fragments into `['A', 'л', 'a', 'я', 'қ']` — five tiny pieces, one
> per character, because mixed-script "words" have no merged multi-character
> token in the vocabulary. The model is not reading a slightly misspelled
> Kazakh word; it is reading a sequence of single, disconnected Latin and
> Cyrillic letter-tokens that do not statistically co-occur as a real word.
> That is a harder pattern to recognize and repair than an ordinary
> letter-for-letter spelling error within one script — and it is exactly the
> case (gemma on KZ-03) where a model produced outright wrong letters instead
> of a correction.

**3. Name one thing this measurement does not explain about your Sublab Medium
results.** You measured OpenAI's tokenizers; three of your six models were not
OpenAI's. What follows, and what would you have to do to close the gap?

> This only measures `cl100k_base` and `o200k_base`, which are OpenAI's
> tokenizers used by luna/terra/sol. Gemma, qwen, and deepseek use their own,
> different vocabularies (SentencePiece/BPE variants trained by Google,
> Alibaba, and DeepSeek respectively), so their actual token counts and
> fragmentation patterns on the same Kazakh text could differ from what is
> shown here — the `input_tokens`/`output_tokens` billed to those three
> models in Sublab Medium were never verified against these two encodings.
> To close the gap, I would need each provider's own tokenizer (e.g. a
> SentencePiece model file or an equivalent library) and re-run measurements
> A and B against it directly, rather than assuming OpenAI's tokenizer
> behavior generalizes to them.