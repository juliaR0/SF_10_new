# Preregistration: Bilingual Agrammatism Simulated Patient

**Author:** Julia Rodrigues Angrisano
**Version:** 1.0
**Date written:** 2026-11-27
**Commit hash at freeze:** 83acca0f11078a988624dbe025e85198ee0906a8
**AI assistance:** the template for this document was drafted with Claude (Anthropic); all predictions, thresholds and decisions are mine. Prompts are in my AI log.

## 1. Purpose

This document lists, before building toy language 2.0 and before running any test battery on it, what I expect the model to do and how I will decide whether it did successfully. It exists so that I can't (even accidentally) adjust the language, the damage or the scoring until the results look the way I want.

## 2. Research questions

- **Research Question 1 (agrammatism):** Does damaging the grammar path of the Bilingual Dual-path model produce the error patterns of agrammatic aphasia, in both Spanish and English?
- **Research Question 2 (transfer):** When therapy is given in one language, does the other language also improve, and does it depend on whether a sentence structure is shared by both languages?
- **Research Question 3 (patients):** Do twins built from real patient language histories (LEX-BADAT) differ in a way that changes which therapy language is predicted to help most?

_These questions are meant to optimize therapy transfer prediction for Spanish-English bilinguals with Broca's aphasia. To do so, it is required that the model realistically, generally simulates healthy individuals and aphasic individuals._

## 3. Predictions

**Confirmatory** = I commit to this prediction and a pass/fail rule now. **Exploratory** = I'll look at it and report it, but it can't pass or fail.

Measures: 
- strict score (right meaning and no other-language words)
- error categories from prototype 2.2 (switch, structure, tense/aux, wrong word, article, agreement)
- sham-adjusted gain (therapy gain minus the same twin's sham gain)
- 95% intervals across twins.

### A. Healthy twins (3a)

| ID | Prediction | Measure | Pass rule | Type |
|---|---|---|---|---|
| A1 | Healthy twins are near ceiling in their dominant language on every test-battery section | strict % per section | ≥ 85% on every section | Confirmatory |
| A2 | The weaker language is weaker in proportion to the patient's exposure | strict % in the non-dominant language vs pre-stroke use | the twin with more L2 exposure scores higher in L2 | Confirmatory |
| A3 | Healthy twins rarely switch languages in single-language test mode | switch % | ≤ 5% | Confirmatory |

### B. Aphasic twins (3b)

| ID | Prediction | Measure | Pass rule | Type |
|---|---|---|---|---|
| B1 | **Argument-structure effect:** more arguments = more errors | strict % for 1, 2, 3-argument sentences | 1-arg > 2-arg > 3-arg in both languages, interval for (1-arg − 3-arg) above 0 | Confirmatory |
| B2 | **Canonicity effect:** passives are harder than actives | agrammatism ratio = passive ÷ active accuracy | ratio < 1, interval below 1 | Confirmatory (2.0 only; 1.0 has no passives) |
| B3 | **Grammar, not words:** grammar-path damage raises structure + tense/aux + article + agreement errors more than wrong-word errors | change from healthy, in points | grammar errors rise ≥ 10 points more than wrong-word errors | Confirmatory |
| B4 | **Double dissociation:** meaning-path damage gives the opposite pattern (mostly wrong words) | same as B3, meaning-path lesion | wrong-word errors rise more than grammar errors | Confirmatory |
| B5 | **Chen pattern 1:** more errors in the non-dominant language | error % per language | non-dominant > dominant for each patient | Confirmatory |
| B6 | **Chen pattern 2:** error-type mix is similar in both languages | proportions of covered error types | same top-2 error types in both languages | Confirmatory |
| B7 | **Chen pattern 3:** switches go mostly into the dominant language | direction of switches | > 50% into the dominant language | Confirmatory |
| B8 | Tense errors are more common than agreement errors | tense/aux vs agreement % | — | Exploratory (Tree Pruning idea; not yet verified in the literature) |
| B9 | English drops endings more; Spanish swaps them more | omission vs substitution share | — | Exploratory |


### C. Therapy (3c)

| ID | Prediction | Measure | Pass rule | Type |
|---|---|---|---|---|
| C1 | Therapy helps the trained language more than sham | sham-adjusted gain, trained language | interval above 0 | Confirmatory |
| C2 | **Hidden transfer (2.2 Finding 2):** with single-language testing, the untrained language's structure errors fall more than with sham | sham-adjusted change in structure errors, untrained language | interval below 0 (fewer errors) | Confirmatory |
| C3 | Transfer is larger for structures shared by both languages than for language-specific ones | C2 measure, shared vs specific structures | shared > specific, interval of the difference above 0 | Confirmatory |
| C4 | Adding 20% maintenance sentences in the other language reduces forgetting | forgetting with vs without maintenance | less forgetting with maintenance; interval of the difference above 0 | Confirmatory |
| C5 | Training complex sentences improves simpler ones (CATE) | sham-adjusted gain on untrained simpler structures | — | Exploratory (later) |

## 4. Methods frozen before testing

| Item | Frozen value |
|---|---|
| Toy language version | filled in at freeze (Mission 2, Step 8): git tag toylang-v2.0 + its commit hash — before the full battery runs |
| Dominant language | the language with the higher pre-stroke use (LEX-BADAT LUQ); if tied, the first-acquired language (L1) |
| Lesion site and method | random seeded zeroing of hidden → pred_compress weights (grammar); random seeded zeroing of hidden → pred_role weights (meaning control) |
| Damage calibration | **only** the overall strict score (scored in single-language mode) is used: each twin's damage = the level whose accuracy is closest to 60% of its healthy accuracy; levels [0, .2, .3, .4, .5, .6, .65, .7, .75, .8] |
| Twins | N per patient (N comes from the twins-needed formula (M = 5 points) using the sham-adjusted spread from 2.0-small, capped at 20 per patient for compute, and decided before the full battery runs); hidden/compress sizes sampled in 76-84 hidden, 52-60 compress; seeds 1 to N |
| Screening rule | a healthy twin is dropped if its dominant-language strict score is below 70% and if more than a third of twins are dropped, the screening rule itself is reported as a problem; every dropped twin is reported |
| Test mode | single-language (only the target language active) for all battery and therapy tests; `--cs` results reported separately |
| Scoring | strict score; 2.2 error categories; **Spanish subject omission (pro-drop) is not an error** |
| Statistics | 95% intervals across twins now; mixed-effects models (twins and sentences as random effects) in prototype 5 |
| Therapy | 5 sessions, learning rate 0.02, --nodlr; options EN, ES, MIXED, SHAM, EN + maintenance, ES + maintenance; every option the same size |

## 5. Decision rules

- A confirmatory prediction **passes** only if its pass rule is met; otherwise it **fails** or is **inconclusive** (interval too wide to decide). All three outcomes get reported.
- Gate 3a must pass (A1–A3) before 3b results are interpreted; gate 3b (B1–B4) before 3c.
- If a gate fails: 
    - 3a fails → fix training (more epochs or a larger hidden layer) as a logged deviation, then retest 3a before 3b.
    - 3b fails → report it as the result, then try grammar_B as a logged, exploratory follow-up.
    - 3c fails → report it as the result ("the model shows no transfer"); it's still a finding.
- A language is testable for B and C predictions only if the healthy twin's strict score in it is at least 50%. Untestable languages are reported as untestable, not as failures.

## 6. Anti-tampering rules

1. This file is committed to GitHub before toy language 2.0 is tested; the commit hash goes at the top.
2. After seeing battery results, the toy language, lesion method, calibration rule and scoring rules can't change unless logged as a deviation (Section 8) and every affected test is rerun and both versions are reported.
3. Every run is reported, including failures and dropped twins (no cherry-picking).
4. Exploratory results are labeled exploratory everywhere (lab notebook, board, write-up).

## 7. Testing schedule

| Stage | Which predictions can be tested | Why limited |
|---|---|---|
| Toy language 1.0, existing 2.1 twins (baseline) | B1 (partly: arguments, no passives), B3, B4, B5, B6 (partly), B7 | no passives, few tense forms, tiny test sets |
| Toy language 2.0, small (a few twins) | all of A and B | small samples, so intervals are wide |
| Toy language 2.0, full (thousands of sentences, full twin population) | all of A, B, C | — |

## 8. Deviations log

| Date | What changed | Why | Which tests were rerun | Commit |
|---|---|---|---|---|
| | | | | |

## 9. Results summary (filled in after testing)

| ID | 1.0 baseline | 2.0 small | 2.0 full |
|---|---|---|---|
| A1 | | | |
| … | | | |
