# AI Usage Report

**Course:** ISM 6564 Text Analytics, Fall 2026
**Student:** Hesham Rashid
**Assignment:** Assignment 1 A Tokenization Report
**Date submitted:** 9/1/26

---

## Part 1 — AI tool statement

| Tool and model version | Where I used it | What I asked it to do |
|---|---|---|
| Claude Sonnet 5 (Claude Code, VS Code extension) | Tasks 1a–5d | Had it follow the instructions in the codable cells, explain the code line by line that is generated, and then I manually verify outputs to make sure they were correct |

---

## Part 2 — Chat history / log

**Sessions included:**

| # | Tool | Date | Roughly what it covered |
|---:|---|---|---|
| 1 | Claude Code (VS Code extension) | 8/29/26 | Full implementation of the coding tasks 1a through 5d, see attached transcript. Markdown cells I wrote myself. |

Session 1 (Claude Code) is long and is submitted as a separate file: Assignment 1 transcript.docx


---

## Part 3 — Validation steps

**How I validated the output:**

For each task, I ran the cell myself after Claude Code wrote it and checked the actual printed output against what the task instructions and in-cell comments said to expect, rather than assuming the code was correct because it ran without error, because it could have ran something incorrect with no errors. Examples: Task 1a: checked the regex tokenizer's output against the test string "Don't ship the LC-475 -- it won't boot! see http://a.io/x" by hand, making sure that things like the apostrophe words Don't and won't stayed the same. Task 3b: the UNK counts for bert-base-uncased and gpt2 on the held-out split both came back as 0%, which looked wrong at first but after I looked at it I realized it was right since, subword tokenizers decompose unfamiliar words into smaller known pieces, so they basically never need UNK unlike. Also after each task cell, I asked for a line by line explanation before moving on, so I could better understand the code instead of just knowing it worked, this also helped in the write ups. 

**Something the AI got wrong that I corrected:**

After the Task 1a prompt, Claude Code reported that regex_tokenize was "fully implemented" in the notebook, but my VS Code editor still showed the original return None stub on screen. So I tried to debugf what was happening and had Claude Code read the notebook file directly from disk (bypassing the editor buffer) and confirmed the implementation really was saved correctly but the problem was that VS Code's editor tab hadn't picked up the external file change and was showing a unsaved buffer, so I just had to revert the file. Another mess up was that claude code randomly chose a different kernal and I had to direct it back to the correct one for our class. 

---

## Declaration

By submitting this report I confirm that:

- [x] It names every AI tool I used, and does not understate that use.
- [x] The transcripts are complete and unedited.
- [x] I can explain and defend every part of what I submitted, and could
      reproduce or modify it on request.
