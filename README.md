# The Unofficial Guide

**Daniel Lin** · Corpus: `campus_life` (88 short student posts about campus life)

---

# Unit 1

## What This Does

<!-- Three or four sentences. Which corpus you picked, and the kinds of
     questions your system answers. Write it for someone who has never seen
     this repo.

     Milestone 5. -->

## Chunking Strategy

**Chunk size:** not a fixed number. One body paragraph per chunk, with the
document's title line prepended. 88 documents become 183 chunks, averaging
167 characters (shortest 63, longest 397).

**Overlap:** none. I split on paragraph breaks rather than a character count,
so no sentence gets cut in half and there is nothing for overlap to repair.
Every fact I tested for sits complete inside one paragraph. With paragraphs
this short, overlapping would make neighbouring chunks near duplicates of each
other and waste retrieval slots.

### Starting point (Milestone 1)
The starter chunker (`chunker.py::fallback_split`) turned 88 documents into 88 chunks,
averaging 317 characters (shortest 178, longest 549). Its 800 character window is
longer than every post, so it never split anything. Reading the posts, each one is
2 to 3 sentences and the useful fact usually sits in a single sentence.

### What I changed and why
The starter's 800 character window never split anything, since no post in
campus_life reaches 800. That left 20 of the 88 posts covering more than one
topic in a single chunk, such as `course_cs_340.txt`, which holds class format,
exams, workload and advice together.

I split on paragraph breaks instead, because the authors already separated
their topics that way. Each chunk gets the document's title line prepended:
the seven housing posts have nearly identical laundry paragraphs that differ
only in price, and without the title a chunk reading "Laundry costs $1.75
wash" contains no mention of Aldridge Hall for retrieval to match on.

I planned to merge paragraphs below a minimum length into their neighbours,
but printed the length of all 183 body paragraphs before writing the code. The
shortest ones turned out to be the corpus at its most useful: single facts like
course workload hours, exam formats, and dining hall times. One of them, at 69
characters, is the answer to one of my own test questions. Any minimum above 70
would have buried it inside a chunk about something else, so I dropped the
minimum entirely and kept every paragraph as its own chunk.

This does not split posts where two topics share one paragraph, such as
`admin_add_drop_deadline.txt`, which states the add and drop deadlines in
consecutive sentences. That is a known limitation.

## Sample Chunks

**Chunk 1** — source: `course_hist_118.txt#1` — produced by: `chunker.py::split_documents`

```
HIST 118 Modern World History

Expect a lot of reading, about 120 pages a week, but no problem sets.
```

A single fact standing on its own. Under the starter's chunker this sentence was
buried in a chunk that also covered seminar format, assessment and essay advice.
It is the answer to one of my five test questions.

**Chunk 2** — source: `housing_aldridge_hall_noise.txt#0` — produced by: `chunker.py::split_documents`

```
Noise levels in Aldridge Hall

Asked about this a lot so writing it down. Quiet floors on 3 and 4 are genuinely enforced.
```

The seven housing noise posts share the sentence "Asked about this a lot so writing
it down" verbatim. Without the prepended title, nothing in this chunk would
identify Aldridge Hall for retrieval to match on.

**Chunk 3** — source: `course_phys_130_workload.txt#0` — produced by: `chunker.py::split_documents`

```
Workload for PHYS 130 Mechanics

People keep asking so: 7 hours a week, plus 3 on lab weeks. That's real time, not optimistic time.
```

One of the short paragraphs I had planned to merge into a neighbour before looking
at the data. On its own it answers a workload question directly.

**Chunk 4** — source: `admin_add_drop_deadline.txt#0` — produced by: `chunker.py::split_documents`

```
On the add/drop deadline

You can add a course through the end of the second week. Dropping is a longer window — through the end of week six — but adrop after week two shows as a W on your transcript. Nothing anywhere on the registrar's site says this plainly, and students find out from each other.
```

A chunk my strategy does not fix. The add deadline and the drop deadline are two
separate questions stated in consecutive sentences of one paragraph, so paragraph
splitting leaves them together.

**Chunk 5** — source: `housing_tamsin_court.txt#3` — produced by: `chunker.py::split_documents`

```
Tamsin Court — what it's actually like

Laundry costs in-unit washer-dryer. On noise: quiet, structurally — concrete floors between units.
```

The same limitation reaching a laundry paragraph. Seven housing posts pair laundry
cost and noise level in one paragraph, which is the shape my Aldridge wash-price
test question has to retrieve through.

### Criterion 4 check

Of the 20 chunks printed by `python app.py chunks -n 20`, 18 cover only one topic
under a strict reading, which meets my target of 18 of 20. The two that fail are
`admin_add_drop_deadline.txt#0` and `housing_tamsin_court.txt#3`, both paragraphs
where the author put two separate facts in consecutive sentences. Three more
(`admin_parking_permits.txt#0`, `course_biol_160.txt#0`, `dining_halden_hall.txt#0`)
are borderline: each holds several facts a student might ask about separately. If
those count as failures the result is 15 of 20 and my target is missed. All five
fail for the same reason, which is the limitation I noted above.

## Sample Answer

<!-- One complete question and answer, pasted as text, with the source line
     visible. Milestone 4. -->

**Question:**

**Answer:**

```
```

**My relevance cutoff:**

<!-- The number you set in config.py, and how you got there.

     You ran five questions your corpus covers and the five in OUT_OF_SCOPE
     that it clearly doesn't, and wrote down the best distance for each. What
     did those two groups look like? Where was the gap? Put the actual numbers
     here — the table below wants all ten rows.

     Milestone 4. -->

| Question | In corpus? | Best distance |
|---|---|---|
|  |  |  |

## How I Used AI

<!-- Two specific moments. For each: what you asked for, what came back, and
     what you changed about it.

     "I asked Claude to write the chunking function from my notes. It ignored
     the overlap, so I added that myself" is the level of detail we're after.
     "I used AI to help me code" is not.

     Milestone 5. -->

**1.**

**2.**

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
