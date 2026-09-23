# The Unofficial Guide

<!-- Replace this line with your name and which corpus you picked. -->

> **This file is your submission.** Fill it in as you go — most sections get
> written during the milestone that produces them, not at the end.
>
> How the starter works, and every command you'll need, is in `RUNNING.md`.
> Leave that file alone.
>
> **Paste everything as text.** No screenshots, no video. A typed table gets
> full credit; a picture of the same table gets none.
>
> Delete these instruction blocks as you replace them. The `<!-- -->` comments
> are notes to you and don't show up when the page renders — you can leave them
> or remove them.

---

# Unit 1

## What This Does

This system answers questions about student life at the university using advice from real student discussion threads. The corpus (`advice_threads`) contains 23 threads covering topics like laundry timing, commuting strategies, meal planning, course policies, and campus resources. You can ask questions like "When is laundry free?" or "What's the best way to commute long distances?" and the system retrieves relevant student advice, cites the source documents, and synthesizes an answer. It refuses to answer questions about topics outside the corpus (like cooking techniques or programming languages) by checking whether the retrieved information is actually relevant before answering.

## Chunking Strategy

**Chunk size:** 250 characters
**Overlap:** 0 characters

Replies in advice_threads are self-contained advice points, ranging from 68 to 179 characters each. I observed that each reply is a complete thought meant to stand alone—splitting on the `--- reply N ---` boundaries respects this structure rather than artificially cutting at character counts. A 250-character limit keeps most replies intact as single chunks, and zero overlap avoids the tiny fragments (like the 2-character tail) that the starter's generic 800-character strategy produced. The resulting 75 chunks average 121 characters with no fragments under 68 characters, meeting criterion 4: each chunk is a complete thought usable without context.

## Sample Chunks

**Chunk 1** — source: `thread_bike_commute.txt#0` — produced by: `chunker.py::split_documents`

```
Yeah. Cuts an 18 minute walk to about 6. The thing nobody mentions is storage — covered bike parking exists at three buildings and is full by 9am at all three.
```

**Chunk 2** — source: `thread_first_gen.txt#1` — produced by: `chunker.py::split_documents`

```
The thing I'd say: the unwritten rules are the hard part, not the coursework. Ask about the unwritten rules explicitly. People are happy to explain them and nobody volunteers them.
```

**Chunk 3** — source: `thread_laptop_specs.txt#2` — produced by: `chunker.py::split_documents`

```
I did two years on an 8GB machine and it was fine until the last project, at which point it very much wasn't. 16 is the answer.
```

**Chunk 4** — source: `thread_parking.txt#1` — produced by: `chunker.py::split_documents`

```
Street parking on Verrill is legal and free and unmarked, which is why half the upper years do it.
```

**Chunk 5** — source: `thread_sleep_schedule.txt#1` — produced by: `chunker.py::split_documents`

```
The library being open until 2am is a trap. It's a resource, not a schedule.
```

## Sample Answer

**Question:** When can students declare the pass/fail option for a course, and what should they consider first?

**Answer:**

According to student advice (thread_first_year_regret.txt), you can declare pass/fail "late — up to week eight." Before declaring, consider that "if you're applying to graduate programmes, some want a letter grade for prerequisites and a P doesn't satisfy it" (thread_pass_fail.txt). Additionally, there's a limit of "two per year and eight across the degree," so use them strategically (thread_pass_fail.txt).

**My relevance cutoff:** 0.60

Measured best distances for all ten questions:

| Question | In corpus? | Best distance |
|---|---|---|
| When is laundry free in the dorms and which days should students avoid? | Yes | 0.520 |
| What time window can students change their meal plan, and what happens if they miss it? | Yes | 0.536 |
| What are the benefits of stacking courses for students who commute long distances? | Yes | 0.465 |
| What resource in the student centre can make commuting easier, and how much does it cost? | Yes | 0.482 |
| When can students declare the pass/fail option for a course, and what should they consider first? | Yes | 0.348 |
| What is the capital of Mongolia? | No | 0.891 |
| How do I change the oil in a diesel engine? | No | 0.720 |
| Who won the 1994 World Cup? | No | 0.929 |
| What is the recommended dosage of ibuprofen for a headache? | No | 0.760 |
| How do I write a for loop in Rust? | No | 0.909 |

**Two groups with a clear gap:**
- In-corpus: 0.348–0.536 (all well below cutoff)
- Out-of-scope: 0.720–0.929 (all well above cutoff)
- Gap: 0.536 to 0.720 (0.184 margin)

I set the threshold at 0.60, squarely in the middle of the gap. This ensures in-corpus questions (criterion 1) will retrieve their answers, while out-of-scope questions (criterion 3) will be refused with high confidence.

## How I Used AI

**1. Chunking function for reply-aware splitting**

I asked Claude to replace `split_documents` with a function that splits on `--- reply N ---` boundaries instead of using the starter's blind 800-character strategy. I gave it actual numbers from my corpus: replies range 68–179 characters, each is a standalone thought. Claude returned code that splits on boundaries, applies the 250-char limit from config, and filters fragments under 10 characters. I tested the output (75 chunks, no 2-char fragments vs. 26 with the starter's approach). The logic was solid, but I noticed it calculated chunk index inefficiently by looping through all previous chunks. I left it as-is because correctness mattered more than optimization for this pipeline.

**2. Tightening the grounding instruction**

I asked Claude to review the generic GROUNDING_INSTRUCTION in generate.py and tighten it for my corpus. Claude came back with the original instruction but suggested four improvements. I accepted most of them: changed "don't guess" to "don't guess or infer," added "quote directly from documents," specified the filename format (e.g., `thread_laundry_timing.txt`), and clarified what to do with multi-source answers. I skipped one suggestion to add "explain confidence" because my corpus is small and advice is usually clear-cut. The tighter rules push the model toward direct quotes and explicit citations, which matters when documents are already well-written student advice.

<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

---

# Unit 2

<!-- These sections get ADDED to what's already above. Don't delete or rewrite
     unit 1 — the point is that someone can see what you said before you knew
     how it went. -->

## Run Log — Before

<!-- Your five criteria, three runs each. `python run_eval.py --label before`
     runs the questions, puts the OUT_OF_SCOPE ones through the gate, and
     writes it all into results/ for you. Targets come from criteria.md; the
     verdict column is your call.

     Criterion 3 is measured in one deterministic pass rather than three, so
     the same number goes in all three run columns. That's correct, not lazy.

     Milestone 1. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

<!-- Underneath, paste the REAL output for each criterion from one of your
     runs — the actual text your system produced, not a description of it.
     Name the file and function that produced it. -->

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 |  |  |  |
| 2 |  |  |  |
| 3 |  |  |  |
| 4 |  |  |  |
| 5 |  |  |  |

## Diagnoses

<!-- For each miss: which stage caused it, and how. The stage alone isn't
     enough — you need the mechanism.

     Not a diagnosis: "Question 3 didn't work."
     A diagnosis:     "Question 3 asks about laundry costs. The answer is in
                       one sentence that got split across two chunks, so
                       neither chunk on its own contains it."

     The five stages: loading → chunking → embedding → retrieval → generation.

     Look for a pattern. If three misses all ask about numbers, that's one
     problem, not three.

     Missed nothing? Say so, then say honestly whether your targets were set
     low, and which one you'd tighten and to what.

     Milestone 3. -->

## The Improvement

**What I changed:**

**Why I picked it:**

<!-- Connect it to a specific diagnosis above in one sentence. If you can't,
     you picked a fix because it sounded impressive. -->

### Run Log — After

<!-- Same format, same five criteria, three runs each.
     `python run_eval.py --label after` -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

**Did it help?**

<!-- Say plainly whether it did, and how you know. If it made things worse,
     say that — a change that backfired, honestly reported, earns full credit
     and is more interesting than one that worked. What matters is that you can
     tell.

     Milestone 4. -->

## What's Still Broken

<!-- For each criterion still missed after your fix: what you'd do about it,
     and why you stopped where you did.

     "I ran out of time" is fine if it's true. Pretending nothing is left is
     not.

     Milestone 5. -->

## What I'd Do Differently

<!-- Knowing what you know now — which of your five criteria would you write
     differently, and why?

     Milestone 5. -->
