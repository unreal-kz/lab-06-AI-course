# Lab 06 — The harness that says no

Content: Harness and Reliable AI Systems, Security and Responsible AI · Narxoz University

A scholarship fund has a small open language model that screens applications. It works: it reads a
profile and answers *shortlist* or *reject* with a score. Your job is not to improve it. Your job is
to find out **what it does to applicants who differ only in something that must not matter**, to build
a **harness** around it (controls that validate, refuse, record and escalate), and to defend that
design in writing: which ethical principle and which law each control answers to.

Every applicant, name and university in this lab is **invented**. The universities are fictional.
No real person, institution or application appears anywhere in the data.

**No API key. No cost.** One open model (`Qwen2.5-3B-Instruct`) runs inside the notebook.

**Due: Saturday 21 November 2026, 23:59 (Asia/Almaty).** 15 points.

## Setup

Open the notebook in Google Colab — one click:

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/unreal-kz/lab-06-AI-course/blob/main/lab06_harness_says_no.ipynb)

1. **Runtime → Change runtime type → T4 GPU.** On a CPU the model is far too slow.
2. Run the first code cell once. It installs the Python packages the lab was prepared with.
3. Run the cells in order, one at a time. A ✏️ cell is a cell where **you** write.

**What it costs you.** The model is downloaded once per Colab session. Lab 05 uses the same model, and
its README gives the download as about 6.2 GB in float16 (a T4 has 16 GB). **How long this notebook
takes on a Colab T4 has not been measured by the instructor**, and it has not been checked there
end to end. Start early, do not leave the first run for the last evening, and do not close the tab
while the model is loading.

## What the notebook does

- **Part 1 — measure.** You get a fixed set of applicant profiles in **matched pairs**. Two
  profiles in a pair are identical on every merit field (grades, entrance exam, household income
  bracket, achievements, programme, age, consent) and differ in **one** thing that must not
  matter: the language the name comes from, the gender the first name signals, or the applicant's
  university. Two further kinds of pair change only something irrelevant (another name of the same
  language group and gender coding; another university of the same tier): their divergence is the
  **floor** each real attribute has to clear. The agent decides on both members of every pair. You
  report where the two outputs diverge, with counts and sizes, read against the floors. **You may find
  no effect**; if so, you report that, with counts, intervals and what your sample could and could not detect.
- **Part 2 — build the controls.** Four controls, written by you against what Part 1 found:
  **input validation**, a **refusal path**, a **structured decision log** (one record per
  decision, refusals and escalations included) and an **escalation route** to human review. You decide
  what the agent is not shown, when it must refuse, and which cases a person must see. For each
  control you fill one row of `CONTROL_REGISTER`: the control, the **ethical principle** it serves,
  the **legal instrument and provision** that bears on it, and why it belongs here given what you measured.
- **Part 3 — measure again, and exercise the refusal path.** The same pairs run through your
  harness. Note that if you hide the name and the university, the two members of those pairs are
  shown identical text, so their "after" flips are zero **by construction**; that table cannot show
  that the harness worked on the model. What is informative is what happened to individual
  profiles (how many decisions and scores changed relative to the unblinded run, which is not proof
  that the model relied on a removed field, since removing lines also changes the prompt's form),
  the share of applicants sent to a person (against the share you planned: a score band calibrated
  on one configuration does not transfer to another), refusals, wrong refusals and the cost in model
  calls. The refusal path must be triggered in the submitted run: a control that nothing has
  tested is a guess, not a result. Your log, `decisions.jsonl`, comes out of this run.
- **Write-up.** `REPORT.md` (what you measured, who is affected, where your responsibility lies)
  and `JUSTIFICATION.md` (what the system may do, must refuse, records, and which trade-offs you accepted).

## How the measurement works

**Matched pairs.** Same applicant, one thing changed. If the agent's decision differs inside a pair,
the only thing that could have caused it is the attribute that differs (or noise, see below). That is
the whole logic. It says nothing about *why* the model does it.

**Flip rate and score difference.** A *flip* is a pair whose two members got different decisions (one
shortlist, one reject). The *score difference* is the difference in the 0–100 scores inside a pair. The
notebook reports, per attribute, the flips, the number of pairs whose scores differ, the signed mean
shift and the **mean absolute difference** |b − a|, with counts, an exact paired test and intervals.
A signed mean can cancel (+10 and −10 average to 0) when the sign of a difference is random, because
the two members of a pair have arbitrary roles; the absolute difference cannot. Report both, and read
each attribute against its floor on both measures. Read the
counts as well as the percentages: with a modest number of pairs, a rate with a wide interval is not yet a finding.

**Floors, and what they are not.** A *null pair* changes something that must not matter **and is not a
protected attribute**; its flip rate is the **floor** a real attribute has to exceed before you may say
it moved the decision. There are two, one per kind of field that the real attributes change:

- `null_name`: a different name **of the same language group and the same gender coding**. This is the
  floor for the two name attributes (language of the name, gender coding of the name).
- `null_university`: a different university **of the same tier**. This is the floor for the university
  attribute.

Greedy decoding is deterministic, so there is no run-to-run noise to sample; a floor measures how much
the agent's output moves when *these* irrelevant details change. It is **not** a bound on all noise:
a different wording of the prompt, or other irrelevant details, could move the outputs more. The
two members of a pair are laid out identically in the prompt, so a floor is not a layout test; an
optional extension in the notebook measures sensitivity to the order of the lines separately, on pairs
of identical applicants (this is a different kind of fragility and not one of the floors).
Say in your read-out what your floors do and do not cover.

**You may find no effect.** Read each attribute against its own floor. If no attribute moves the
decision beyond its floor, say so, with the counts, the intervals and what your number of pairs could
and could not have detected (the notebook computes the smallest one-sided count an exact test could call
significant). Absence of a measured effect is not proof of absence, and **a difference that does not
exceed your own floor is not an effect: claiming it is an unsupported claim.** A measurement that shows
nothing to remove does not make the controls pointless, but it changes what you can say they were
built against: say what they cost and what they changed.

**Runs are not guaranteed to match yours.** The model decodes greedily, so repeating a run on the
same machine gives the same text. A different GPU computes in a different numerical order, so your
counts may differ slightly from a classmate's. That has not been measured for this lab.

## What you hand in

Everything is committed to **your fork** of this repository. Submit the URL of your last commit in
Canvas. Late commits are flagged, not hidden.

1. **`lab06_harness_says_no.ipynb`**: run from top to bottom, with all outputs saved: the
   measurement before and after the harness, and the refusal path exercised. *File → Download → .ipynb*,
   and replace the template in your fork. The last cell reports anything still missing.
2. **`control_register.json`**: your `CONTROL_REGISTER`. It lives in code in the notebook, and the
   notebook also writes it to this file; check that the file is in your fork.
3. **`decisions.jsonl`**: the decision log produced by your submitted run, one JSON record per
   line. A log you wrote by hand is not evidence.
4. **`REPORT.md`**: the template in this repository, filled in.
5. **`JUSTIFICATION.md`**: the template in this repository, filled in.

In both templates the first line after the title is `<!-- STUDENT: replace -->`. **Delete that line**
when you replace the guidance with your own text; a file that still carries it counts as not written.

**No claim about a law without a citation.** Every statement about what a law requires, permits or
forbids needs a citation to **the instrument and the provision** it rests on. **You identify which
instruments apply** to a system like this one and say why; the course does not give you a list.
**A chat model's statement is not a citation**: it can name a law that does not exist or put a
provision in the wrong place. Read the primary text yourself, and cite what you read. Places where
primary texts are published (these are places to look, not a hint about which instruments apply):

- the official legal database of the Republic of Kazakhstan, `adilet.zan.kz`;
- EUR-Lex, `eur-lex.europa.eu`, for European Union law;
- the OECD (`oecd.ai` and the OECD legal instruments database) and UNESCO, for international
  principles and recommendations on AI. These are not laws; say what status the text you cite has.

A principle (ethics) and an instrument (law) are different things; the register has a column for each,
and a control that cites only one of them is half done. Whether an instrument applies to a
fictional fund is a judgement you have to argue, not assume.

## What is assessed

The assignment is evidence for two ABET Performance Indicators. **PI 4.1**, Awareness of Professional
Responsibility, is **25 %** of the task (the matched-pair measurement and what you draw from it:
3.75 of the 15 points). **PI 4.2**, Integration of Ethical and Legal Considerations, is **75 %** (the
controls and their written justification: 11.25 of the 15 points). Each indicator is graded
dimension by dimension; the weight is the share of **that indicator**. (Source of the dimensions and
weights: the ABET PI assessment document for CSS-4007, HW6, pp. 5–6.)

| PI 4.1 dimension | Weight |
|---|---|
| Evidence of Specific Impact | 45 % |
| Affected Parties and the Consequences They Bear | 30 % |
| Recognition of Professional Responsibility | 25 % |

| PI 4.2 dimension | Weight |
|---|---|
| Integration of Constraints into the Design | 25 % |
| Identification and Citation of Legal Constraints | 20 % |
| Identification of Applicable Ethical Principles | 15 % |
| Refusal Boundaries and Enforcement | 15 % |
| Accountability Record and Auditability | 15 % |
| Trade-off Analysis and Justification | 10 % |

Each dimension is placed at one of four levels by the share of its marks you earn: **Exemplary** from
90 %, **Satisfactory** from 70 %, **Developing** from 50 %, **Unsatisfactory** below 50 %. The
bands are the ones in the ABET document; the 0 / 50 / 70 / 90 % reading of marks is still awaiting
the coordinator's confirmation.

Where the evidence is: dimensions of PI 4.1 are read from `REPORT.md` and the notebook's measurement;
those of PI 4.2 from the code (`CONTROL_REGISTER`, the controls), `decisions.jsonl`, the refused
cases in the submitted run, and `JUSTIFICATION.md`. The guiding questions in the two templates are
drawn from the descriptions of the Satisfactory and Exemplary levels.

## AI use

Add half a page saying which AI tools you used, and for what, as the last section of
`JUSTIFICATION.md` (headed `AI-use declaration`); the rules are on the course's AI use policy page in Canvas. Whatever tools you use, **the legal citations in your work
must come from the instruments themselves**, and every number you quote must be one your own notebook
printed. Lab 05 required its `CONCLUSION.md` to be typed without any AI tool; this lab asks for a
declaration instead, as the Canvas assignment page says.

## Glossary

- **Matched pair**: two profiles identical on every merit field that differ in one attribute that must not matter.
- **Null pair / floor**: a pair that differs in something that must not matter and is not protected (another name of the same language group and gender coding; another university of the same tier). Its flip rate is the floor for "divergence that is not about the attribute", not the whole noise.
- **Mean absolute score difference**: the mean of |score of b − score of a| over the pairs with two scores; a magnitude that cannot cancel, unlike the signed mean.
- **Order probe** (optional): pairs of identical applicants, one shown with its lines in a different order; divergence there is sensitivity to the layout of the prompt.
- **McNemar test (exact)**: a test on the pairs whose two decisions differ; it asks whether "a shortlisted, b rejected" is as common as "b shortlisted, a rejected", using only those pairs.
- **Refusal path**: the route by which the system declines to produce a decision at all (for example, a purpose it must not serve or an input it must not process), and says why.
- **Escalation**: sending a case to a human reviewer instead of letting the system decide.
- **Audit record**: the stored account of one decision, enough for someone later to see why it was made without re-running the model.

## Files

| File | What it is |
|---|---|
| `lab06_harness_says_no.ipynb` | The lab: Parts 1–3, no outputs saved |
| `lab06_core.py`, `lab06_data.py` | The agent, harness, log writer, profiles, pairs and statistics (supplied) |
| `data/` | The fictional universities and invented names |
| `REPORT.md`, `JUSTIFICATION.md` | The templates you replace with your own text |
| `README.md` | This file |

Instructor-side material (reference solution, key, tests) is kept in the instructor's course
repository and is not needed here.
