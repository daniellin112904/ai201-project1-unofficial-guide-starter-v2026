# Acceptance criteria — The Unofficial Guide

Five criteria that say what "working" means for this system, written in unit 1
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"Retrieval works"* is an opinion. *"For at
least 4 of my 5 test questions, the top results include a chunk containing the
answer"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter or looser one. A reason that says something about your corpus or your
pipeline earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## 1. Retrieved chunks contain the answer

For at least 4 of my 5 test questions, the retrieved chunks include one that
contains the answer.

**Why this target:**
I expect my question about the Aldridge Hall wash price to be the hardest, since
there are 7 nearly identical `_laundry.txt` documents that differ only in their
prices, so retrieval could pull the wrong hall. My other four questions each ask
about a fact in a short post on a single topic, so if more than one question
failed, it would mean retrieval is missing even the easy cases.

---

## 2. Every answer names a source

Every answer the system produces names at least one source document.

**Why this target:**
Every chunk already carries its filename, and the grounding instruction tells the
model to name its source. A missing source would mean the model ignored its
instructions, and an answer without a source can't be checked, so I want this to
hold for all 5 answers rather than 4 of 5.

---

## 3. The relevance gate stops out-of-corpus questions

When I ask a question my documents clearly don't cover, the relevance gate
stops it and the system returns "I don't have enough information about that" —
in at least 4 of 5 tries.

<!-- The five questions are the ones in `OUT_OF_SCOPE` at the bottom of
     `questions.py`, and `run_eval.py` puts them through the gate and writes
     what happened into your run log. Swap them for your own if you'd rather —
     just keep five of them, or the "4 of 5" above has nothing to be 4 of. -->

**Why this target:**
Most of the out-of-scope questions are clearly unrelated to campus life, but the question about writing a for loop in Rust shares programming vocabulary with the CS
course posts, so it could land under the cutoff. I allow one miss for that
possibility, but more than one would mean the gate is letting unrelated questions
through.

---

## 4. Something about your chunks

When I inspect 20 chunks, at least 18 should focus on one clear topic and not combine unrelated topics.



**Why this target:**
I chose this because some posts in the corpus contain multiple unrelated topics, which could make it harder for retrieval to identify the information needed for a question. I would allow 2 chunks to fail, but most chunks should stay focused on one topic.


---

## 5. Your choice

For at least 4 of my 5 test questions, the final answer contains the `expects`
phrase listed for that question in `questions.py`.



**Why this target:**
This is different from criterion 1: criterion 1 checks that the retrieved chunk
contains the answer, while this checks that the model picks the right detail out
of it. The laundry post lists both a wash and a dry price, and the printing post
has several numbers in the same few sentences, so a chunk can be correct while the
answer is wrong. Getting a confident answer with the wrong price is the mistake
that would bother me most as a student. I allow one miss because a correct answer
can be worded differently from my `expects` phrase, like "week two" instead of
"second week," but more than one would mean the model is regularly picking the
wrong detail.


---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 2 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 1. Retrieved chunks contain the answer

         For at least 4 of my 5 test questions, the retrieved chunks include
         one that contains the answer.

         **Why this target:** ...

         > **Revised in unit 2:** For at least 4 of 5 questions, the top three
         > results contain the answer.
         >
         > **Why revised:** I couldn't judge "the chunks include one that
         > contains the answer" the same way twice — I scored two questions
         > differently on Monday than on Wednesday. The new version is
         > something I can actually check.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said 4 of 5 but got 2 of 5, so 2 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.

     The whole reason the originals stay visible is so someone can see what you
     said before you knew the answer.
     ───────────────────────────────────────────────────────────────────────── -->
