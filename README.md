# The Unofficial Guide

**Daniel Lin** · Corpus: `campus_life` (88 short student posts about campus life)

---

# Unit 1

## What This Does

This is a retrieval-augmented question answering system over `campus_life`, a
corpus of 88 short posts about student life at a fictional university: dining
halls, dorms, course workloads, and administrative deadlines. You ask a plain
question like "how much does a wash cost at Aldridge Hall?" and it retrieves the
closest chunks from those posts, answers from them only, and names the file the
answer came from. If nothing retrieved is close enough to the question, it
refuses instead of guessing, so asking about something the corpus doesn't cover
gets you an honest "I don't have enough information" rather than an invented
answer.

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

**Question:** How much does a wash cost at Aldridge Hall?

**Answer:**

```
  (best distance 0.262, cutoff 0.6)

A wash costs $1.75 at Aldridge Hall.

Source: housing_aldridge_hall.txt (also found in housing_aldridge_hall_laundry.txt)

Sources retrieved: housing_aldridge_hall.txt, housing_aldridge_hall_laundry.txt, housing_innisfree_hall.txt
```

**Top-k:** 3, lowered from the starter's 5. The correct chunk came back at rank 1
for all five of my test questions, and for the printing question ranks 2 to 5 sat
at 0.68 or worse, so the extra slots were adding unrelated material rather than
context. Lowering top-k limits how many chunks reach the model, not how relevant
they are: even at 3, the printing question still passes two chunks at roughly 0.68
and 0.70.

**Grounding instruction:** I read `GROUNDING_INSTRUCTION` in `generate.py` with
`--show-prompt` and left it unchanged. It already restricts the model to the
supplied documents, tells it to refuse when they don't cover the question, and
requires it to name the file. It says nothing about choosing between near-identical
facts, which is the live risk in this corpus: the three chunks retrieved for the
question above contain six laundry prices across three halls, four of them $1.75.
The model picked correctly here, so I left the instruction alone rather than adding
a rule I had no evidence was needed. If criterion 5 misses in unit 2, this is the
first place I would look.

**My relevance cutoff:** 0.6, unchanged from the starter's default.

I ran all five of my test questions and all five OUT_OF_SCOPE questions through
`python app.py retrieve` and recorded the best distance for each. The two groups
are far apart: everything my corpus covers landed between 0.262 and 0.329, and
everything it doesn't landed between 0.787 and 0.923. The gap between them is
about 0.46 wide, so 0.6 sits comfortably in the middle and classified all ten
questions correctly. I considered tightening it to around 0.45 for extra margin,
but my in-corpus questions all sat under 0.33 and nothing in the data suggested
0.6 was too loose.

In `criteria.md` I predicted the Rust and ibuprofen questions would come closest
to the cutoff, because one shares programming vocabulary with my CS course posts
and the other is about health, which `health_center.txt` covers. Both were wrong:
they landed at 0.860 and 0.849, among the furthest of the five. The closest
out-of-corpus question was the capital of Mongolia at 0.787, which retrieved
HIST 118 chunks, presumably on shared history and geography vocabulary.

| Question | In corpus? | Best distance |
|---|---|---|
| How much does a wash cost at Aldridge Hall? | Yes | 0.262 |
| What time does the North Kitchen close on weekdays? | Yes | 0.269 |
| How many pages of reading per week does HIST 118 assign? | Yes | 0.292 |
| How much more does colour printing cost than black and white? | Yes | 0.294 |
| What is the deadline to add a course? | Yes | 0.329 |
| What is the capital of Mongolia? | No | 0.787 |
| Who won the 1994 World Cup? | No | 0.847 |
| What is the recommended dosage of ibuprofen for a headache? | No | 0.849 |
| How do I write a for loop in Rust? | No | 0.860 |
| How do I change the oil in a diesel engine? | No | 0.923 |

## How I Used AI

**1. Deciding whether to merge short chunks.** I was designing my chunker and
wanted short paragraphs merged into their neighbours so I wouldn't end up with
fragments. I asked Claude how to set the minimum length and proposed taking a
percentage of the shortest paragraph in each file. It pointed out that at any
percentage below 100, the shortest paragraph is by definition above the minimum,
so the rule could never fire in any file. It suggested I print the actual length
distribution instead of guessing. I ran that over all 183 body paragraphs and
found the shortest ones were the most useful pieces in the corpus: single facts
like course workload hours, exam formats, and dining hall times. One of them, at
69 characters, is the answer to one of my own test questions. I dropped the
minimum entirely rather than implementing what I had originally planned.

**2. Writing my test questions.** I drafted six questions from documents I had
read and asked Claude to tighten the wording and the `expects` phrases. Two
problems came back that I hadn't seen. My colour printing question expected the
answer 75, which I had worked out myself as 600 divided by 8; no chunk in the
corpus contains that number, so the question would have been testing arithmetic
rather than retrieval. My CS 340 workload question had two separate facts in one
`expects` field, and my version of the answer disagreed with what the document
actually says. I cut both questions, then opened each source file and copied the
`expects` phrases straight from the text, shortening them so they would still
match if the model reworded the answer. "120 pages" instead of "about 120 pages
per week", for example.

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

Produced by `run_eval.py::main`, three runs per question with caching off.
Full output committed in `results/run_2026-09-29_2318_before.md`.

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Chunks are at most two sentences (revised) | 90% | 77.0% | 77.0% | 77.0% | MISSED |
| 5. Answer contains the `expects` phrase | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |

Criteria 3 and 4 show the same number in all three columns. Criterion 3 is a
single deterministic pass of the gate, as `run_eval.py` explains. Criterion 4
does not vary between runs either, since it reads the stored chunks.

Criterion 4 is scored here against the revised version of the criterion, for the
reason set out under The Improvement below. The figure is 141 of 183 chunks, and
it comes from running the same measurement with `SENTENCE_SPLIT_ABOVE` disabled,
which reproduces the unit 1 chunker exactly. My original scoring of the unit 1
criterion, 17 of 20 by hand, is kept in the Verdicts table and in the criterion 4
evidence below, since that is the scoring that showed me the criterion could not
be applied consistently.

### Criterion 1 — retrieved chunk contains the answer

Retrieval is deterministic, so the same chunks came back on all three runs. For
each question the file holding the answer was among the retrieved sources, from
`store.py::search` over chunks made by `chunker.py::split_documents`:

```
What is the deadline to add a course?
  Sources retrieved: admin_add_drop_deadline.txt, admin_pass_fail_option.txt, advising_registration.txt

What time does the North Kitchen close on weekdays?
  Sources retrieved: dining_north_kitchen.txt, dining_north_kitchen_followup.txt

How much does a wash cost at Aldridge Hall?
  Sources retrieved: housing_aldridge_hall.txt, housing_aldridge_hall_laundry.txt, housing_innisfree_hall.txt

How many pages of reading per week does HIST 118 assign?
  Sources retrieved: course_hist_118.txt, course_hist_118_workload.txt

How much more does colour printing cost than black and white?
  Sources retrieved: admin_printing_quota.txt, housing_calder_annexe.txt, money_textbooks.txt
```

### Criterion 2 — every answer names a source

All fifteen answers named a file. Produced by `generate.py`. Run 1, all five
questions:

```
You can add a course through the end of the second week (admin_add_drop_deadline.txt).

North Kitchen closes at 7:00pm on weekdays, according to *dining_north_kitchen.txt*.

A wash at Aldridge Hall costs $1.75.

Source: housing_aldridge_hall_laundry.txt (also mentioned in housing_aldridge_hall.txt)

HIST 118 assigns about 120 pages of reading per week.

Source: course_hist_118.txt (also mentioned in course_hist_118_workload.txt)

Colour printing costs eight times as much per page as black-and-white printing (admin_printing_quota.txt).
```

### Criterion 3 — the gate on out-of-corpus questions

Produced by `run_eval.py::check_out_of_scope`, cutoff 0.6. Refused 5 of 5:

```
refused  (best distance 0.787)  What is the capital of Mongolia?
refused  (best distance 0.923)  How do I change the oil in a diesel engine?
refused  (best distance 0.847)  Who won the 1994 World Cup?
refused  (best distance 0.849)  What is the recommended dosage of ibuprofen for a headache?
refused  (best distance 0.860)  How do I write a for loop in Rust?
```

### Criterion 4 — chunks cover one topic

Measured with `python app.py chunks -n 20`, chunks from
`chunker.py::split_documents`. Three of the twenty combine facts a student
would ask about separately:

```
Chunk 1  |  source: admin_add_drop_deadline.txt#0

On the add/drop deadline

You can add a course through the end of the second week. Dropping is a longer window — through the end of week six — but a drop after week two shows as a W on your transcript. Nothing anywhere on the registrar's site says this plainly, and students find out from each other.
```

```
Chunk 2  |  source: admin_parking_permits.txt#0

On the parking permits

Student permits for the west lots go on sale in August and sell out in about three days. The east lot never sells out because it's a 12-minute walk. There is no waitlist — people who miss the window park on Verrill Street and walk in, which is legal but unmarked and confuses everyone.
```

```
Chunk 19  |  source: housing_tamsin_court.txt#3

Tamsin Court — what it's actually like

Laundry costs in-unit washer-dryer. On noise: quiet, structurally — concrete floors between units.
```

### Criterion 5 — answer contains the `expects` phrase

Every answer on all three runs contained its `expects` phrase from
`questions.py`. The wording around it varied; the phrase itself did not.
The Aldridge question across all three runs:

```
run 1: A wash at Aldridge Hall costs $1.75.
run 2: A wash costs $1.75 at Aldridge Hall (housing_aldridge_hall_laundry.txt and housing_aldridge_hall.txt).
run 3: A wash at Aldridge Hall costs $1.75.
```

## Verdicts

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 | Retrieved chunk contains the answer | MET | For every question, the file holding the answer was among the retrieved sources on all three runs. 5 of 5 against a target of 4 of 5. |
| 2 | Every answer names a source | MET | I read all fifteen answers and each named at least one file. The format varied between inline parentheses, a `Source:` line and backticks, but the criterion asks that a source is named, not how. |
| 3 | Gate stops out-of-corpus questions | MET | The gate refused all five OUT_OF_SCOPE questions, the closest at 0.787 against a cutoff of 0.6. 5 of 5 against a target of 4 of 5. |
| 4 | Chunks cover one topic | MISSED | 17 of 20 against a target of 18 of 20, so it misses by one. I counted a chunk as failing when it holds two facts a student would ask about separately: the add and drop deadlines, the permit sale window and the unofficial Verrill Street parking, and Tamsin Court's laundry setup alongside its noise level. I counted `course_biol_160.txt#0` and `dining_halden_hall.txt#0` as passing even though both open with "I lived here my sophomore year", because that line is reused verbatim across unrelated posts as flavour text and carries no fact anyone would ask about, so it is noise inside a chunk rather than a second topic. In unit 1 I counted this same sample as 18 of 20 by treating the parking chunk as one topic. Re-reading it for this verdict, the permit sale window and the unofficial Verrill Street workaround are things a student would ask about separately, so I counted it as a failure here, which moves the count to 17 of 20. |
| 5 | Answer contains the `expects` phrase | MET | I checked each of the fifteen answers for the literal phrase in `questions.py`. All fifteen contained it. 5 of 5 against a target of 4 of 5. |

## Diagnoses

**Criterion 4 — chunks cover one topic (17 of 20, target 18 of 20)**

The failure happened at the **chunking** stage. `chunker.py::split_documents`
cuts only at blank lines between paragraphs, but in
`admin_add_drop_deadline.txt`, `admin_parking_permits.txt` and
`housing_tamsin_court.txt` the separate facts are divided by sentence
boundaries inside a single paragraph, so the splitter had nothing to cut on and
each chunk kept two or three topics.

All three failures are the same problem, not three different ones: the authors
of these posts wrote several separately-askable facts as consecutive sentences
rather than as separate paragraphs, and my splitting rule only sees the blank
lines. I named this limitation in unit 1 before running any test, using
`admin_add_drop_deadline.txt` as the example, so the test confirmed a weakness
I already expected rather than finding a new one.

Criteria 1, 2, 3 and 5 were all met, so there is nothing to diagnose for them.

## The Improvement

**What I changed:** I added `SENTENCE_SPLIT_ABOVE = 260` to `config.py` and a
step to `chunker.py::split_documents` that splits any body paragraph longer
than that at sentence boundaries. Paragraphs at or below 260 characters stay
whole. The corpus went from 183 chunks to 202, and the longest chunk fell from
397 characters to 283.

**Why I picked it:** My diagnosis said the chunking stage kept two or three
separately-askable facts in one chunk because the authors wrote them as
consecutive sentences rather than separate paragraphs, and my splitter only cut
at blank lines. Before writing any code I measured the paragraphs involved: two
of the three failures were the longest paragraphs in the sample, at 274 and 285
characters, while the longest paragraph I had counted as passing was 252. A
threshold of 260 sat in that gap, so it would split the two long failures and
leave everything else alone.

I knew going in that this could not fix all three. The Tamsin Court laundry
paragraph that failed was only 98 characters, shorter than four paragraphs I
had counted as passing, so no length threshold could reach it without splitting
those too. I chose the length rule anyway because it was one small change I
could measure cleanly, rather than a sentence-splitting rule with a merge step
that would have reversed a decision I made in unit 1 and changed several things
at once.

### Criterion 4 was revised, not lowered

Scoring criterion 4 for this unit showed I could not apply it the same way
twice. In unit 1 I counted `admin_parking_permits.txt#0` as one topic; in unit
2 I counted it as two. In a single sitting I failed
`housing_tamsin_court.txt#3` for pairing laundry cost with noise level while
passing `dining_north_kitchen.txt#1` for pairing opening hours with price,
which is the same shape of chunk. "One clear topic" turned out to measure how
broadly I was defining a topic that day rather than anything about the chunks.

I replaced it with a mechanical version: at least 90% of all chunks contain no
more than two sentences of body text. Sentence count is a proxy for the same
thing and anyone can check it. 90% preserves the strictness of the original 18
of 20, and measuring every chunk rather than a sample of 20 removes a second
problem: `python app.py chunks -n 20` returned a different 20 chunks once the
chunk count changed, so I was not scoring the same chunks before and after. The
original line stays in `criteria.md` with the revision underneath it.

Both run logs below score criterion 4 against the revised version, measured over
every chunk. The before figure comes from running the same measurement with the
threshold disabled, which reproduces the unit 1 chunker exactly.

### Run Log — After

Produced by `run_eval.py::main`, three runs per question with caching off.
Full output committed in `results/run_2026-09-30_0124_after.md`.

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Chunks are at most two sentences (revised) | 90% | 82.7% | 82.7% | 82.7% | MISSED |
| 5. Answer contains the `expects` phrase | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |

**Did it help?**

Yes, but not enough to meet the target. Criterion 4 went from 141 of 183 chunks
(77.0%) to 167 of 202 (82.7%), an improvement of 5.7 points against a target of
90%, so it is still a MISS. The specific chunks that motivated the change were
fixed: `admin_add_drop_deadline.txt#0` is now just the add deadline, and the
parking permit paragraph is split into its three separate facts.

The 35 chunks that still fail show why the length rule could only go so far. The
shortest of them is 143 characters, barely half my threshold, and the twelve
shortest are almost all opening paragraphs from housing and course posts. These
pack three or four short sentences into well under 260 characters, so a length
threshold can never reach them. Length was a proxy for "this paragraph holds
several facts", and in this corpus that proxy holds for long paragraphs and
fails for short dense ones.

Two things improved that criterion 4 does not capture. The add-course question's
best distance fell from 0.329 to 0.188, the largest change of any question,
because the chunk it matches went from three sentences covering both deadlines
and a complaint about the registrar's website down to the one sentence that
answers it. Its retrieved sources also tightened: before the change the three
slots held `admin_add_drop_deadline.txt`, `admin_pass_fail_option.txt` and
`advising_registration.txt`, and after it two of the three come from the
add/drop post itself. No other question's distance moved, because no other
question's source document had a paragraph over 260 characters.

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
