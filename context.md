# ML SYSTEM DESIGN - CASE REPO CONTEXT

This file is the spec. Every case file in this repo is written against it.
Read this before writing a new case, and before drilling an old one.

## 1. WHY THIS REPO EXISTS

RCA cases drill DIAGNOSIS: a number moved, find out why.
System design cases drill CONSTRUCTION: nothing exists yet, build it.

They fail differently, so they need different skeletons:

| | RCA | System design |
|---|---|---|
| Given | a metric that moved | a business goal |
| Core skill | localise before theorising | define the label before the model |
| Fatal move | explaining a number you never verified | drawing boxes before naming the label |
| Output | a decision (roll back / hold) | a phased plan (v0 rules -> v1 model -> v2) |
| Ends with | a pre-committed threshold | a launch plan with guardrails |

The single most common failure in a system design round is drawing a
two-tower architecture in minute three without ever saying what a positive
label is. The spine below exists to make that impossible.

---

## 2. REPO LAYOUT

```
ml_system_design/
    context.md            <- this file: the spec every case is written to
    recommendation/       no query typed - implicit feedback, feedback loops
    search/               intent is declared - relevance is judgeable
    ads/                  the score is spent as money - calibration
    fraud/                the distribution fights back - adversarial
    moderation/           the label is a policy - human in the loop
    notifications/        send or stay silent - incrementality, fatigue
    forecasting/          time is the axis - no random splits, asymmetric cost
    marketplace/          two-sided - interference breaks the standard A/B
    llm/                  no single correct output - the eval IS the design
    speech/               long expensive inputs - label efficiency
    vision/               annotation is the budget - active learning
    lifecycle/            the label is a window you chose - uplift over propensity
```

Twelve domains, chosen so that each one drills a DIFFERENT failure mode
rather than a different industry. If a new case fits an existing folder's
failure mode, it goes there; a new folder is only justified by a genuinely
new way of being wrong.

Every folder carries a `_scope.txt`: what belongs there, the one skill it
drills, and its candidate case list. Read that file before writing a case
into the folder, and update its case list as cases are written.

**File naming:** `<domain>_case_<NN>.txt`, zero-padded, e.g.
`recsys_case_01.txt`, `fraud_case_02.txt`.

**Format:** plain `.txt`, same as the RCA files. Fixed width, wrap at 76
columns, ASCII headers only. No markdown inside case files, no emojis
anywhere. `context.md` is the only markdown file in the repo.

---

## 3. CASE FILE ANATOMY

Every case file has this exact skeleton.

```
CASE NN: <Short Title>
======================
LEVEL: EASY | MEDIUM | HARD  - one line on what this case drills and
which case to do before it.

CONTEXT
-------
The business situation. Company, surface, scale. Include NUMBERS: users,
QPS, catalog size, latency budget, current baseline. Numbers are what make
the case answerable instead of a vocabulary quiz.

YOUR ROLE
---------
Who you are. Seniority changes the answer - an IC designs the pipeline, a
manager also decides staffing, sequencing and what not to build.

QUESTION
--------
The ask, phrased the way an interviewer phrases it. One or two sentences.


#######################################################################
PART A - THE GENERAL PLAYBOOK (use this for EVERY design case)
#######################################################################

Reproduce section 4 of this file, lightly adapted to the domain.
Only the FIRST case in each domain folder carries Part A in full;
later cases in that folder say "see <first case> for Part A" and go
straight to Part B.


#######################################################################
PART B - THE ANSWER (spoken, as if answering the interviewer)
#######################################################################

0.  HOW I AM GOING TO STRUCTURE THIS   (say the plan out loud, plus the
                                        one headline insight you are
                                        betting on)
1.  CLARIFYING QUESTIONS               (3-5, and name which ones change
                                        the answer entirely)
2.  THE BUSINESS GOAL AND WHETHER ML   (what we are paid in; the
    IS EVEN THE ANSWER                  heuristic baseline that must be
                                        beaten)
3.  THE LABEL                          (the hard part - what is a
                                        positive, who produces it, how
                                        delayed, how noisy)
4.  METRICS                            (offline / online / guardrails,
                                        and why they disagree)
5.  DATA AND FEATURES                  (sources, leakage, point-in-time,
                                        train-serve skew)
6.  SYSTEM ARCHITECTURE                (ASCII diagram, with the latency
                                        budget written on it)
7.  MODEL CHOICE                       (baseline first, then earn the
                                        complexity)
8.  TRAINING AND EVALUATION            (splits, retrain cadence, offline
                                        eval that correlates with online)
9.  LAUNCH PLAN                        (shadow -> 1% -> ramp, with a
                                        pre-committed stop rule)
10. FAILURE MODES AND MONITORING       (cold start, feedback loops,
                                        drift, fairness, adversarial)
11. WHAT I WOULD BUILD IN WEEK 1 vs    (the phasing - this is the senior
    MONTH 3 vs MONTH 6                  signal)
12. THE 60-SECOND VERSION              (the whole design compressed)
```

Section numbers may merge or reorder to fit the case, but none may be
silently dropped. If a section does not apply, say so in one line and say
why - that is itself an answer.

---

## 4. PART A - THE GENERAL PLAYBOOK (THE SPINE)

Every ML system design question is the same eight questions in the same
order. The discipline is in the ORDER. Most candidates fail not from
ignorance of models but from reaching question 6 before answering 3.

```
1. WHAT ARE WE PAID IN?   Business goal -> ML task. Or no ML at all.
2. WHAT IS THE LABEL?     Positive definition, source, delay, noise.
3. HOW IS IT SCORED?      Offline metric, online metric, guardrails.
4. WHAT SHAPE IS IT?      Batch vs online, funnel stages, latency, scale.
5. WHAT GOES IN?          Features, sources, leakage, train-serve skew.
6. WHAT MODEL?            Baseline first. Earn every extra parameter.
7. HOW DOES IT SHIP?      Offline eval, shadow, A/B, staged ramp.
8. HOW DOES IT ROT?       Drift, feedback loops, cold start, abuse.
```

Compressed: **Paid? -> Label? -> Score? -> Shape? -> Input? -> Model? ->
Ship? -> Rot?**

### STEP 1 - WHAT ARE WE PAID IN?

Translate the business ask into one sentence of the form: *"predict X for
Y at time Z, in order to move business metric M."* If you cannot write that
sentence, you do not have a problem yet - keep clarifying.

Then say out loud what the non-ML version is. Popularity ranking, a rules
engine, a lookup table. If a heuristic gets 80% of the value, the honest
design ships the heuristic in week one and uses it to collect data.
Interviewers reward the candidate who is willing to not use ML.

Name the horizon and the unit of decision: per request, per user per day,
per item per week. This determines everything downstream.

### STEP 2 - WHAT IS THE LABEL?

This is where seniority shows. Four sub-questions, always:

- **What counts as a positive?** Click, 30s dwell, 80% completion,
  next-day return, a purchase that was not returned. Different choices
  build different products out of the same architecture.
- **Where does the label come from?** Implicit user behaviour, human
  annotation, a downstream business system, a weak/proxy rule.
- **How delayed is it?** A fraud chargeback lands 60 days later. A churn
  label needs a 30-day window. Label delay caps your retrain cadence and
  determines whether online learning is even possible.
- **How noisy and how biased?** Implicit feedback only ever tells you
  about what you SHOWED. That is position bias and it is a feedback loop,
  not a nuisance.

State the class imbalance and the volume. "Roughly 0.2% positives, about
4M labelled events a month" changes the model choice more than any
architectural preference.

**The no-data branch (see section 5 - this is the most common curveball).**

### STEP 3 - HOW IS IT SCORED?

Three tiers, and say all three:

- **Offline:** AUC/PR-AUC, NDCG@k, recall@k, calibration, MAE. Pick one
  primary and justify it against the decision the model drives. Calibration
  matters whenever the score feeds an economic decision (bidding,
  thresholds, expected value); pure ranking does not need it.
- **Online (the OEC):** the one number the launch is judged on. Define it
  BEFORE the design, and make it purchase-, retention- or revenue-based,
  never engagement alone. Engagement as a primary metric is how you get
  clickbait - that is Goodhart and it is the single most common way an ML
  launch destroys value while the dashboard goes green.
- **Guardrails:** latency p99, cost per 1k requests, coverage/diversity,
  fairness across segments, complaint rate, the counter-metric for the
  obvious degenerate solution.

Then say the uncomfortable part out loud: offline and online metrics will
disagree. The design goal is to make offline eval CORRELATE with the online
OEC, and to track that correlation over launches.

### STEP 4 - WHAT SHAPE IS IT?

Draw the system only once steps 1-3 are settled. Establish first:

- scale: QPS, peak/average, users, items, events per day
- latency budget, split across stages, p99 not mean
- batch (precompute, cheap, stale) vs online (fresh, expensive)
- the funnel: retrieval / candidate generation -> filtering -> ranking ->
  re-ranking for business rules and diversity

The retrieval-then-rank split is the load-bearing idea in almost every
large-scale design: a cheap model over millions of candidates, an expensive
model over hundreds. Say WHY the split exists (latency x cardinality), not
just that it does.

Write the latency budget on the diagram: `retrieval 20ms -> rank 30ms ->
rerank 5ms -> 45ms headroom`. That single line separates people who have
shipped from people who have read.

### STEP 5 - WHAT GOES IN?

- **Sources:** user profile, item metadata, interaction history,
  contextual (time, device, location), aggregates, embeddings.
- **Leakage:** any feature computed after the label event. Point-in-time
  correctness is not optional - features must be reconstructed as they
  were at decision time, not as they are now.
- **Train-serve skew:** the same code path must compute the feature in
  both places, or they will drift. This is the most common silent killer
  in production ML and naming it is a strong signal.
- **Freshness:** which features are real-time (counters, session), which
  are daily batch, which are static. Each tier has a different failure
  mode and a different cost.
- **Missing values at serving time:** a null feature for a new user is the
  normal case, not the edge case. Say what the model does with it.

### STEP 6 - WHAT MODEL?

Order matters as much as choice:

```
rules/heuristic -> logistic regression or GBDT on tabular features
                -> embeddings / two-tower retrieval
                -> deep ranker / sequence model -> LLM in the loop
```

Start at the left. State what each step buys you and what it costs in
latency, infra and debuggability. GBDT is the correct answer for most
tabular problems and saying so confidently is a positive signal, not a
weak one.

For each candidate model, say: training cost, inference latency,
interpretability, cold-start behaviour, retrain cost. Choose against the
constraints from step 4, not against fashion.

### STEP 7 - HOW DOES IT SHIP?

- offline eval on a time-based split (never random split when there is any
  temporal structure - random splits leak the future)
- backtest / replay against logged traffic
- shadow mode: score live traffic, serve nothing, compare distributions
- A/B: randomisation unit, power calculation, minimum runtime, CUPED for
  variance reduction
- staged ramp 1% -> 5% -> 20% -> 50% -> 100%, with a retained holdout
- a pre-committed stop rule, written down BEFORE the launch, plus
  automated alerting so a bad launch halts without a human noticing a
  dashboard

Say the retrain cadence and what triggers it: a schedule, a drift alarm,
or a performance drop. Say who is on the hook when it fires at 3am.

### STEP 8 - HOW DOES IT ROT?

Every model degrades. The design must say how you find out:

- **Cold start:** new user, new item, new market. Three separate problems
  with three separate fallbacks.
- **Feedback loops:** the model's own output becomes tomorrow's training
  data. Popularity concentrates, the tail starves, the model confirms
  itself. Mitigation: exploration traffic, propensity logging,
  position-bias correction (IPS), diversity constraints.
- **Drift:** feature drift vs label drift vs concept drift. Monitor input
  distributions, not just output accuracy - accuracy needs labels, which
  arrive late.
- **Adversarial pressure:** anywhere there is money (fraud, ads, spam,
  marketplace ranking), the distribution fights back. Retrain cadence
  becomes a security property.
- **Fairness and harm:** segment the primary metric, not just the average.

Close with the degradation dashboard: what you watch daily, what pages
someone, and what the automatic fallback is when the model is unavailable.
Every design needs a defined answer to "what do we serve if the model
service is down."

---

## 5. THE NO-DATA BRANCH (COLD START FROM ZERO)

Interviewers deliberately frame problems where no dataset exists - a new
product, a new market, a brand-new surface. It is a favourite because it
cannot be answered from memorised architectures. Any case in this repo may
be run in this mode, and at least one case per domain should be.

When there is no data, the answer is a data STRATEGY, in this order:

1. **Define the label before anything else.** No data makes this MORE
   urgent, not less. You cannot collect what you have not defined.
2. **Ship a heuristic v0 that is instrumented to generate labels.**
   Popularity, recency, hand-tuned rules, editorial curation. The point of
   v0 is not the ranking, it is the log.
3. **Design the logging before the model.** Impressions, positions,
   outcomes, request context, and the identifiers that let you join them.
   If you cannot attribute an outcome back to an impression, no amount of
   modelling later will save you.
4. **Bootstrap labels, cheapest first:** proxy/weak supervision from
   existing business rules; transfer from a related product or market;
   pretrained embeddings on content; public datasets; LLM-generated labels
   with a human-audited holdout; paid human annotation with a written
   guideline and an inter-annotator agreement number; buy or partner.
5. **Build exploration in from day one.** A greedy v0 only ever teaches
   you about what it showed. Epsilon-greedy, bandits, or a slice of random
   traffic - and log propensities so off-policy evaluation is possible
   later. Most candidates miss this entirely; it is the strongest single
   signal in a cold-start answer.
6. **Evaluate without ground truth:** online A/B against the heuristic,
   human eval on a sample, guardrails, staged rollout. Say explicitly that
   offline eval is unavailable for the first N weeks and what you use
   instead.
7. **Give the timeline:** week 0 rules, month 1 first supervised model on
   logged data, month 3 richer model with exploration retained.

The failure mode being tested is the candidate who blocks: "I would need
data before I could build anything." The correct answer ships without ML
and treats data acquisition as a designed part of the system.

---

## 6. TIMEBOXING A 45-MINUTE ROUND

```
 ~5 min  clarify: goal, scale, constraints, what exists today
 ~5 min  framing out loud: the ML problem statement + the OEC
 ~7 min  label and data (longer if it is a cold-start case)
~10 min  architecture, drawn, with the latency budget on it
 ~7 min  model choice and training
 ~6 min  evaluation and launch plan
 ~5 min  failure modes, monitoring, phasing
```

If you are 20 minutes in and have not named the label, you are losing.

---

## 7. COMMON TRAPS

- Drawing architecture before defining the label. The cardinal sin.
- Never stating the business metric, so nothing can be traded off.
- Optimising engagement and calling it success. Goodhart.
- Jumping to the most complex model available to look impressive.
- Ignoring latency and cost entirely, then proposing a deep ranker over
  a million candidates.
- Random train/test split on temporal data. Silent future leakage.
- Forgetting cold start for new users AND new items AND new markets.
- No monitoring story, no fallback when the model service is down.
- Ignoring the feedback loop the model creates in its own training data.
- In a marketplace or two-sided market, forgetting the supply side.
- Presenting one design with no alternative and no stated trade-off.
- No phasing - describing the month-6 system as if it ships on day one.

---

## 8. WRITING STYLE FOR THE ANSWER SECTIONS

The answer is SPOKEN. Write it as a transcript of a strong candidate, not
as notes.

- First person, present tense: "I would...", "My leading bet is...".
- Take positions. "GBDT, and I would need a clear reason to go deeper."
  Hedged surveys of every option read as not having shipped anything.
- Every claim carries its consequence: not "I would add exploration" but
  "I would add exploration, because otherwise the v0 heuristic is the only
  thing the month-1 model can ever learn from."
- Put numbers on everything: latency in ms, scale in QPS, volumes,
  percentages. Invented-but-plausible numbers beat no numbers.
- Rank things, always. Ranked lists of hypotheses, models, phases.
- Name the ONE insight the case turns on, early, then earn it.
- Where an RCA answer ends with a pre-committed decision threshold, a
  design answer ends with a phased plan and a launch stop rule.
- ASCII diagrams only. Boxes and arrows, wrapped at 76 columns.
- No emojis. No markdown formatting inside `.txt` case files.
- End with the 60-second compression - assume the interviewer asks for it.

---

## 9. CASE INDEX

Cases written so far. Keep ordered by difficulty within each domain, and
mirror every entry into that folder's `_scope.txt`.

| Case | Domain | Level | Drills |
|---|---|---|---|
| [01 Listening hours for capacity planning](forecasting/forecasting_case_01.txt) | forecasting | EASY | Newsvendor quantile from an asymmetric cost; one series, four decisions, opposite postures |

Candidate cases per domain live in each folder's `_scope.txt`.

Suggested progression across the whole repo, ignoring domain:

1. An EASY case with abundant data and an unambiguous label, to make the
   spine automatic. (`recommendation/recsys_case_01.txt`)
2. The same surface with NO data, to drill the cold-start branch.
3. A case with a delayed or noisy label. (`fraud/`, `ads/`)
4. A case where the label is a human policy. (`moderation/`)
5. A two-sided case where the experiment design itself is broken.
   (`marketplace/`)
6. A case with no single correct output. (`llm/`)
