# Assignment 5 — Can We Run This On Our Own Hardware?

## When and how

- **Opens:** Wednesday, September 23, 2026 at 12:30 PM ET (start of Week 5 class).
- **Due:** Tuesday, September 29, 2026 at 11:59 PM ET (the evening before Week 6's class).
- **Late work:** accepted up to 24 hours late with a **20% penalty**. After
  Wednesday, September 30, 2026 at 11:59 PM ET it is not accepted and scores zero.
- **This assignment counts toward your assignment grade** (30% of the course
  total, spread over ten assignments).

### What to submit

Submit **two** items to Canvas before the deadline:

1. `w5assignment.ipynb` — all cells run from the top, outputs visible.
2. Your **AI Usage Report** — see [`templates/ai-usage-report.md`](../../templates/ai-usage-report.md),
   with the tool and model version, the complete transcript of your AI
   conversations, and a short note on how you checked the output. AI use is
   required on every assignment, so this is never optional. See §7 of the
   [syllabus](../../syllabus.md).

> **Goal:** put a small language model on your own machine and find out, in
> numbers, what it costs to run, what it does with a customer's message, and
> how well it routes the desk's tickets — then write the recommendation.

---

## Setting

Northwind Outfitters' support desk receives about **2,400 tickets a week**.
Every one is currently read by a human and dropped into one of six queues.
The support operations director wants the first pass automated, and has two
proposals on the table:

1. **Call a frontier model's API** for every ticket.
2. **Run a small open-weights model on the team's own hardware** — the Mac
   Studio already sitting in the ops room — so ticket text never leaves the
   building.

Option 2 is on the table for a specific reason: tickets contain order numbers,
partial card digits, and home addresses, and Legal has questions about
sending that to a third party. If a local model is good enough, that question
disappears. "Good enough" is what you are being asked to measure.

Everything you need is in `w5assignment.ipynb`: the model loader, the chat
prompt helper, the ticket set, and the metric tables. The data loads itself
from the course's public data repository.

### Before you start

**Do the Week 5 practice notebook first** (on Canvas). It rehearses every
mechanic this assignment uses — the chat template, the generation loop, the
cache timing, sampling with a seed — on a 135M model that runs in seconds.

**The models.** Three instruction-tuned decoders, small enough for a laptop,
plus the base version of the middle one:

| Checkpoint | Learned numbers | Download | Used in |
|---|---:|---:|---|
| `HuggingFaceTB/SmolLM2-360M-Instruct` | ~362M | ~0.7 GB | Tasks 1, 3, 4 |
| `Qwen/Qwen2.5-0.5B-Instruct` | ~494M | ~1.0 GB | Tasks 1–4 |
| `Qwen/Qwen2.5-1.5B-Instruct` | ~1.54B | ~3.1 GB | Tasks 1, 3, 4 |
| `Qwen/Qwen2.5-0.5B` (base) | ~494M | ~1.0 GB | Task 2 |

- **The first run downloads about 6 GB** into `~/.cache/huggingface/hub`;
  after that everything is read from disk. Budget about 6 GB of free disk and
  do the download well before the night it is due. If disk space is a
  problem, drop the 1.5B model, say so in your notebook, and lose only the
  top rung of the size comparison.
- **Google Colab is fine.** Pick a T4 runtime; the models download in about a
  minute per session. Wherever the notebook says "your machine," name the
  Colab runtime instead.
- **Expect roughly 10–15 minutes of compute** for a filled-in notebook, most
  of it in Task 3. If one model's 60-ticket pass takes more than a few
  minutes, check that you are on the accelerator and not on the CPU.
- **Sanity anchors.** Greedy decoding is deterministic, so your accuracy
  numbers should reproduce exactly across runs on the same machine. Expect
  the largest model somewhere around 0.6–0.7 and the smallest well below 0.3.
  If your largest model scores near chance (0.167), the most likely cause is
  that `build_messages` is not sending the system prompt.

---

## Tasks

### Task 1 — Two bills, before you download anything

Write `learned_numbers`, `predicted_gb`, and `measured_gb` to compute the
**memory bill**: learned numbers × bytes per number. Then write
`time_to_write` to measure the **time bill**: seconds to generate a fixed
number of new tokens, with the KV cache on and off, on a short and a long
prompt.

Provided cells run both helpers over the three models and report memory at
4/2/1/0.5 bytes per number, and generation speed with and without the cache
remembered.

> **Deliverable:** the memory table for all three models, the cache-timing
> table (short vs. long prompt, cache on vs. off), and 4–6 sentences
> explaining the arithmetic on the 8 GB-laptop question, the long-prompt
> speed-up, and which bill grows with conversation length.

### Task 2 — What the model does with a message

Write `reply`, a thin wrapper around `model.generate` that returns only the
newly generated text. Provided cells use it to compare the **base** model
(bare text in) against the **instruct** model (the same text through the chat
template) on one ticket, and then run the instruct model under five decoding
settings: greedy twice, temperature 0.7 with top-p 0.9 under two seeds, and
temperature 1.5 with no truncation.

> **Deliverable:** both provided cells run with readable output, and a
> written answer covering what changed between the base and instruct models,
> why greedy reproduces and sampling does not, what happens at temperature
> 1.5, and which setting you would use for routing versus for drafting a
> reply.

### Task 3 — Does it actually do the job?

`support_tickets.csv` holds 60 hand-labelled support tickets across six
queues (`Shipping`, `Billing`, `Returns`, `Defect`, `Account`, `Other`), ten
per queue, split into `clear` and `ambiguous` difficulty. A fixed system
prompt gives every model the same category definitions and the instruction to
answer with one category word.

Write `parse_label` to pull a category out of a raw reply, and `classify_all`
to run one model, greedily, over all 60 tickets, timing each one. Provided
cells run all three models and build the metrics table (accuracy, invalid
replies, accuracy on clear vs. ambiguous, tokens, seconds per ticket), a
per-queue accuracy table, and a confusion matrix with every misrouted ticket
printed for your best local model.

The desk's tie-break rules — Defect beats Returns, Shipping beats Billing,
Returns before the goods move vs. Billing after, Account covers credentials
only, Other is pre-sale/out-of-scope — are given in the notebook. Read them
before arguing with a label: "the model disagreed with the rule" and "the
gold label is wrong" are different findings.

> **Deliverable:** the metrics table, the per-queue table, the confusion
> matrix and wrong-ticket listing, and a written answer that explains the
> size-vs-accuracy trend, names the hardest queue with specific ticket IDs,
> explains speed from token counts (not just parameter counts), and names the
> costliest mistake.

### Task 4 — Small vs. large, and your recommendation

You do not have an API key yet, so the frontier side is supplied as a
**fixture**: `frontier_cached.json`, authored reply strings (not sampled from
a live model) with real token counts, scored exactly like the local models.

Write `usd_cost` to turn input/output token counts into a dollar figure from
the `RATE_CARD` in the notebook. Provided cells compute the fixture's
per-ticket and weekly/yearly API cost, and build a single comparison table
across all four systems — accuracy, clear/ambiguous accuracy, invalid
replies, and a cost or machine-time column for each.

> **Deliverable:** the cost table, the comparison table, and a memo of three
> parts: **Choose** — a committed recommendation for Northwind with at least
> four measured numbers; **Evaluate** — which numbers are real measurements
> versus fixture illustration, and one further measurement you would gather;
> **Control** — the specific conditions under which a human must approve a
> prediction, and what would flip your recommendation.

---

## Ground rules

- **Every number you state must come from a cell above it.** The notebook
  runs top to bottom with nothing filled in — `SKIPPED` messages tell you
  what is still outstanding.
- Do not change `SEED` or `GREEDY`. Greedy accuracy should reproduce exactly
  on your machine; timings will not, and are not meant to.
- The frontier numbers in Task 4 are a fixture, not a product measurement.
  Say so explicitly wherever you use them.
- You may use AI assistance freely (§7 of the syllabus). You must be able to
  explain every line you submit, including what `use_cache=False` makes the
  model redo for every new token, and the AI Usage Report is required.

## Scope

Expect **10–15 minutes of compute** once everything is filled in, most of it
in Task 3, on top of the usual notebook-writing time. If a single model pass
takes more than a few minutes, check you are on an accelerator, not the CPU.

---

## Rubric

| Component | Points |
|---|---:|
| **Task 1** — helpers pass the self-check (6), memory and cache tables produced for all three models (6), written answer correct on the arithmetic, the 8 GB laptop, the long-prompt speed-up, and which bill grows (8) | 20 |
| **Task 2** — `reply` correct, returning only the new text (5); both provided cells run with readable output (3); written answer correct on the two stages, greedy vs. sampled, temperature 1.5, and the route/draft choice (7) | 15 |
| **Task 3** — `parse_label` passes the self-check and `classify_all` runs all three models (10); metrics, per-queue table, and confusion matrix present (6); error analysis grounded in actual raw replies and the tie-break rules, speed explained from tokens, costliest mistake named (14) | 30 |
| **Task 4** — `usd_cost` correct and the comparison table complete (8); memo: a committed choice with ≥4 measured numbers (8), evaluation with fixture-vs-measurement stated and a next measurement named (7), control with flip conditions (7) | 30 |
| **Reproducibility and presentation** — seeds untouched, notebook runs top to bottom, outputs visible, no stray scratch cells | 5 |
| **AI Usage Report** (complete: tool statement, full logs, validation note) | 0 (gate: −5 to −10 if incomplete, up to −20 if missing) |
| **Total** | **100** |

### What separates full marks from most marks

- Accuracy numbers that reproduce exactly on re-run, because greedy decoding
  and the fixed seed were left alone.
- Treating the Task 4 fixture honestly — using its real token counts for cost
  arithmetic while never citing its accuracy as a measurement of a live
  model.
- A control policy in Task 4 with concrete trigger conditions (which queues,
  which confidence threshold), not a general statement that "a human should
  check it."
