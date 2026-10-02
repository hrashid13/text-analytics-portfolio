# Assignment 1 — A Tokenization Report

## When and how

- **Opens:** Wednesday, August 26, 2026 at 12:30 PM ET (start of Week 1 class).
- **Due:** Tuesday, September 1, 2026 by 11:59 PM ET (the evening before Week 2's class).
- **Late work:** accepted up to 24 hours late with a **20% penalty**. After
  Wednesday, September 2 at 11:59 PM ET it is not accepted and scores zero.
- **This assignment counts toward your assignment grade** (30% of the course
  total, spread over ten assignments).

### What to submit

Submit **two** items to Canvas before the deadline:

1. `assignment.ipynb` — all cells run, outputs visible.
2. Your **AI Usage Report** — see [`templates/ai-usage-report.md`](../../templates/ai-usage-report.md).
   AI use is required on every assignment, so this is never optional.
   See §7 of the [syllabus](../../syllabus.md).

> **Goal:** produce the tokenization report a platform team would actually
> read before committing to a text stack — measured vocabulary growth,
> measured out-of-vocabulary rates, measured token cost, and a recommendation
> you can defend with numbers rather than with adjectives.

---

## Setting

You have joined the data platform team at an online retailer that also runs a
large public community forum. Three projects are waiting on one decision —
**how will we tokenize?** — and nobody has measured anything yet.

- **Project A — Lexical search.** An inverted index over the forum archive, so
  a support agent can find every post containing `LC-475` or `pericarditis`.
  Exact terms matter.
- **Project B — Thread summarization.** Send forum threads to a frontier LLM
  through an API and store the summary. Finance wants a cost number before
  they will sign anything.
- **Project C — International rollout.** The forum is opening to German,
  Spanish, Hindi, and Chinese speaking markets on the same paid plan. Product
  has asked, in writing, whether "the same plan" means "the same product" for
  those users.

The archive stands in for the forum: four newsgroups from the 20 Newsgroups
collection, fetched for you in the notebook. The **train** split is the
existing archive; the **test** split is "next month's posts" — text your
vocabulary has never seen.

Everything is set up in `assignment.ipynb`: the corpus loader, the parallel
multilingual corpus, the stress-test strings, and the rate card. Your job is
the measurement and the judgement.

---

## Tasks

### Task 1 — Baseline: how many words are there?

Tokenize the **archive** (train split) three ways:

- `str.split()` — whitespace only,
- a regular expression tokenizer you write yourself,
- `nltk.word_tokenize` — rule-based (Penn Treebank).

For each, report **total tokens**, **distinct types**, **type/token ratio**,
**mean tokens per document**, and the **15 most frequent types**.

Then hunt: find **three concrete strings in the corpus** where `str.split()`
produces something no human would call a word — one URL or e-mail address, one
piece of punctuation-glued text, and one of your own choosing. Print the
original string and what each of the three tokenizers does with it.

> **Deliverable:** one comparison table, the three examples with all three
> tokenizations shown, and ≤150 words on which differences would change a
> search index and which would not.

### Task 2 — Vocabulary growth

Walk the archive in document order. Every 25,000 tokens, record the cumulative
token count and the cumulative number of **distinct** types seen so far, for
three tokenizers: whitespace, `bert-base-uncased` (WordPiece), and `gpt2`
(byte-level BPE).

Plot all three curves on **log–log axes**. For the whitespace curve only, fit
Heaps' law

$$V = k N^{\beta}$$

by least squares on $\log V$ against $\log N$, and report $k$ and $\beta$.

> **Deliverable:** one figure with all three curves labelled, the fitted $k$
> and $\beta$, and ≤150 words explaining what the two subword curves do that
> the whitespace curve does not, and why that is a design property rather than
> an accident.

### Task 3 — Out of vocabulary

**Part A — word-level.** Build closed vocabularies from the archive by keeping
only the $V$ most frequent whitespace types, for
$V \in \{2000, 5000, 10\,000, 25\,000, 50\,000, \text{all}\}$. For each, measure
on the **held-out** split:

- **OOV token rate** — share of held-out token *occurrences* not in the vocabulary,
- **OOV type rate** — share of held-out distinct *types* not in the vocabulary.

**Part B — subword, on the same held-out split.** Count how many `[UNK]`
tokens `bert-base-uncased` produces, and how many `gpt2` produces. Report the
actual counts, not an impression.

**Part C — the stress set.** The notebook supplies `STRESS_SET`, eight short
strings with emoji, Devanagari, Greek, accented Latin, and currency symbols.
For each tokenizer report the token list, the `[UNK]` count, and whether
`decode(encode(s)) == s`.

> **Deliverable:** the OOV table (both rates, all six vocabulary sizes), the
> Part B counts, the Part C table, and ≤200 words reconciling Part B with Part
> C — they do not tell the same story, and the reason is the point.

### Task 4 — Reversibility and normalization

Two questions a pipeline has to answer before it processes a single document.

1. **Is tokenization reversible?** For six strings of your choice (include at
   least one accented word and one emoji), check whether
   `decode(encode(s)) == s` for `bert-base-uncased`, `gpt2`, and
   `tiktoken`'s `cl100k_base`. Where it fails, print what came back.
2. **Which Unicode form?** `"café"` can be stored as four code points (NFC) or
   five (NFD) and the two are not equal in Python. Report the token count of
   each form under `bert-base-uncased`, `gpt2`, `cl100k_base`, and
   `o200k_base`.

> **Deliverable:** a reversibility table, the NFC/NFD token counts, and a
> ≤100-word normalization **policy** for the platform: which form you write to
> storage, at which point in the pipeline you normalise, and the specific
> failure that decision prevents.

### Task 5 — Token cost, and who pays it

**Part A — the archive.** Total `cl100k_base` and `o200k_base` tokens for the
whole archive. Using the `RATE_CARD` in the notebook and stated assumptions
(the notebook proposes one pass per month with output tokens at 15% of input —
change it if you justify the change), compute the monthly API cost of Project
B under each encoding. Show the arithmetic, not just the total.

**Part B — the same words in five languages.** Load
`data/udhr_parallel.json` (loaded for you into `parallel`). Every entry is the
same declaration in English, German, Spanish, Hindi, and Chinese. For each
language and each encoding, compute characters, tokens, tokens per character,
and **tokens relative to English**. Plot tokens-relative-to-English as a bar
chart, both encodings side by side.

**Part C — convert it to money.** Fix a support conversation at 2,000 input
tokens and 500 output tokens *for an English speaker*. Assume the same
conversation in another language scales by that language's token ratio from
Part B. Using `RATE_CARD`, what does one conversation cost for each language,
and what is the ratio between the most and least expensive?

> **Deliverable:** the cost table, the figure, the per-language conversation
> cost, and ≤200 words. Name the group of users who are worst off, quantify
> how much worse, and say what changed between `cl100k_base` and `o200k_base`.

### Task 6 — Recommendation

Write a memo of **400–600 words** to the platform lead. It must:

1. Make one **specific** recommendation for each of Projects A, B, and C, each
   citing at least one number you computed in Tasks 1–5.
2. Name **one thing you would measure before committing** that this analysis
   does not cover, and say what result would change your recommendation.
3. Explain **one risk** created by pairing text tokenized one way with a model
   trained on another tokenizer, and how you would prevent it in code.
4. State plainly **what your analysis does not establish.** A report that
   claims more than its evidence supports is worse than no report.

> **Deliverable:** the memo, in a markdown cell at the end of the notebook.
> Prose, not bullets, for at least the first and last paragraphs.

---

## Ground rules

- **Every number in your report must come from code in your notebook.** If a
  number appears in prose, the cell that produced it must be visible above it.
- Set the seeds the notebook sets and do not change the corpus selection —
  your numbers should be reproducible by the grader.
- Where a task says "≤150 words", that is a ceiling, not a target. Short and
  specific beats long and hedged.
- You may use AI assistance freely (§7 of the syllabus). You must be able to
  explain every line you submit, and the AI Usage Report is required.

## Scope

This is designed as **4–6 hours** of work. If you are past eight, something
has gone wrong — post on the discussion board rather than grinding.

---

## Rubric

| Component | Points |
|---|---:|
| **Task 1 — Baseline tokenizers.** Three tokenizers correctly applied; complete comparison table; three genuine failure cases found in the corpus and shown through all three tokenizers; commentary distinguishes differences that matter for search from ones that do not. | 12 |
| **Task 2 — Vocabulary growth.** Correct streaming curves for all three tokenizers; readable log–log figure; Heaps' law fitted correctly with $k$ and $\beta$ reported; explanation identifies the fixed-vocabulary ceiling as a design choice. | 16 |
| **Task 3 — Out of vocabulary.** OOV token *and* type rates correct at all six vocabulary sizes; `[UNK]` counts on held-out data reported honestly; stress-set table complete; written answer reconciles the corpus result with the stress-set result. | 18 |
| **Task 4 — Reversibility and normalization.** Round-trip results correct for all three tokenizers; NFC/NFD token counts correct; policy is specific about form, pipeline stage, and the failure it prevents. | 12 |
| **Task 5 — Token cost and fairness.** Archive token totals correct under both encodings; cost arithmetic shown and assumptions stated; per-language ratios correct; figure readable; per-conversation costs computed; written answer names and quantifies the disadvantaged group and the effect of the newer encoding. | 22 |
| **Task 6 — Recommendation.** All three projects addressed with numbers cited; a named further measurement with a stated decision rule; a concrete tokenizer/model mismatch risk with a code-level control; an honest statement of the analysis's limits. | 20 |
| **AI Usage Report** (complete: tool statement, full logs, validation note) | 0 (gate: −5 to −10 if incomplete, up to −20 if missing) |
| **Total** | **100** |

### What separates full marks from most marks

- Numbers that reproduce when the grader re-runs the notebook.
- Claims that are *bounded* — "under this rate card and this traffic
  assumption" rather than "it costs $X".
- Noticing that Task 3 Part B and Part C disagree, and explaining why instead
  of picking whichever supports a tidier story.
