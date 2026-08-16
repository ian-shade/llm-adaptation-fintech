# Final numbers — v9.1 re-score

Re-score of the v9 predictions with the unit-aware numeric matcher. **No inference re-run**, so every prediction, latency and retrieval statistic is unchanged from v9; only the scoring rule differs, and it was applied identically to all four conditions.

`summary_scoring_fix.csv` records the before/after. **0 previously-correct answers were lost in any condition.**

---

## Metric 1 — correctness

| Condition | Objective (n=240) | Clean subset (n=233) | short_answer (136) | single_choice (44) | multiple_choice (36) | true_false (24) |
|---|---|---|---|---|---|---|
| D — Baseline | 22.1% | 22.3% | 5.2% | 56.8% | 8.3% | 75.0% |
| A — RAG | 64.2% | 63.5% | 68.4% | **72.7%** | **36.1%** | 66.7% |
| B — Fine-tuned | 27.5% | 27.9% | 10.3% | 61.4% | 22.2% | 70.8% |
| C — Hybrid | **66.3%** | **65.7%** | **71.3%** | **72.7%** | 30.6% | **79.2%** |

Ranking is identical on the clean subset (all deltas < 0.7 points), so the 7 contaminated questions are not driving the comparison.

### Open-ended (n=60)

| Condition | token F1 | ROUGE-L | BERTScore | Judge 1–5 |
|---|---|---|---|---|
| Baseline | 0.305 | 0.176 | 0.837 | 1.68 |
| RAG | 0.382 | 0.259 | 0.860 | 2.98 |
| Fine-tuned | 0.382 | 0.221 | **0.873** | **1.62** |
| Hybrid | **0.421** | **0.270** | 0.872 | **3.10** |

Fine-tuned ranks top on BERTScore and last on the judge — the clearest demonstration in the study that surface-similarity metrics reward register over correctness.

### Multiple choice, scored two ways

| Condition | exact set | letter Jaccard | gap |
|---|---|---|---|
| Baseline | 0.083 | 0.461 | 0.377 |
| RAG | 0.361 | 0.569 | 0.208 |
| Fine-tuned | 0.222 | **0.722** | **0.500** |
| Hybrid | 0.306 | 0.597 | 0.292 |

---

## Metrics 2, 3, 4

| Condition | Halluc. proxy | Claim-level halluc. | Judge faithfulness | Adv. abstention | fabricated | out-of-scope yr | bank absent | Paraphrase Δ (n=36) |
|---|---|---|---|---|---|---|---|---|
| Baseline | 76.3% | 51.9% | — | 30.0% | 16.7% | 22.2% | 55.6% | +2.8pp |
| RAG | **32.9%** | **36.7%** | **4.10** | **76.7%** | 91.7% | 66.7% | 66.7% | +5.6pp |
| Fine-tuned | 72.5% | **85.4%** | — | **0.0%** | 0% | 0% | 0% | −5.6pp |
| Hybrid | 33.8% | 47.3% | 3.98 | **0.0%** | 0% | 0% | 0% | +5.6pp |

Retrieval bank-precision **100%**, off-diagonal confusion **0.0%**, on all 300 questions.

---

## Retrieval and extraction (short-answer, n=114 with a numeric reference)

| | RAG | Hybrid |
|---|---|---|
| Answer-in-context ceiling (k=6) | 86.0% | 86.0% |
| Accuracy | 66.7% | 70.2% |
| **Extraction efficiency** | **75.5%** | **78.6%** |
| Lost — never retrieved | 12.3% | 11.4% |
| Lost — figure on screen | 21.1% | 18.4% |
| Correct without the figure | 1.8% | 2.6% |

Per bank:

| Bank | Ceiling | RAG acc | RAG extraction eff. | Hybrid extraction eff. |
|---|---|---|---|---|
| HSBC | 82.4% | 58.8% | 71.4% | 67.9% |
| Lloyds | **92.9%** | 66.7% | 71.8% | 76.9% |
| NatWest | 81.6% | **73.7%** | **83.9%** | **90.3%** |

**Note this changed from the pre-re-score picture.** With digit-string matching HSBC's ceiling read 76.5% and looked worst on both axes. Unit-aware matching lifts it to 82.4% — HSBC states figures in more varied units — and NatWest now has the *lowest* ceiling while converting best. The remaining HSBC weakness is extraction, not retrieval.

---

## Metric 5 — cost

| Condition | Inference | Training | Total | Colab units |
|---|---|---|---|---|
| Baseline | 0.402 h | — | 0.402 h | 0.433 |
| RAG | 1.414 h | — | 1.414 h | 1.521 |
| Fine-tuned | 0.456 h | 0.501 h | 0.958 h | 1.031 |
| Hybrid | 1.785 h | 0.501 h | 2.286 h | 2.460 |
| **TOTAL** | | | **4.557 h** | **4.907** |

Median latency: Baseline 0.59 s, Fine-tuned 1.12 s, RAG 5.69 s, Hybrid 6.69 s.

**RQ3:** RAG costs 3.5× baseline GPU spend and 9.6× median latency for 2.9× the accuracy. Hybrid costs 5.7× spend for 3.0× accuracy — 62% more than RAG for 2.1 extra points.

---

## Hypotheses (ToR §4.3)

- **RAG > FT on numeric/lookup** — supported. 0.647 vs 0.270 pooled across short_answer, single_choice and true_false.
- **FT > RAG on analytical/open** — not supported. ROUGE-L 0.221 vs 0.259; judge 1.62 vs 2.98.

---

## The three claims worth building the discussion around

1. **Retrieval is not the bottleneck; reading is.** Bank precision 100%, answer-in-context 86%, extraction efficiency 76%. The residual loss decomposes into ~12% never retrieved, ~21% retrieved-and-misread, and the misread portion splits by document — Lloyds fails on format adherence (10 of 14 table dumps), HSBC on row selection (5 of 9 wrong-cell picks).

2. **Fine-tuning buys accuracy and sells safety.** Hybrid beats RAG by 2.1 points and 0.12 judge points, and takes abstention from 76.7% to **zero** while raising claim-level hallucination from 36.7% to 47.3%. For financial QA that is not a favourable trade, and it is a stronger conclusion than "Hybrid won".

3. **Metric choice changes the answer.** BERTScore ranks the fine-tuned model first and the judge ranks it last; multiple-choice looks broken at 22% exact-set and competent at 0.72 Jaccard; and a unit-blind numeric matcher understated every condition by 1–7 points. All three are RQ1 evidence that the evaluation framework is itself a research object.

---

## Caveats to carry into the write-up

- **7 evaluation questions (2.3%) appear in the training set with the same gold figure.** Ranking is unchanged without them; cite `leakage_audit.csv`.
- **`accuracy_when_not_retrieved` is 12.5% (RAG) and 18.8% (Hybrid)** — 2 and 3 answers respectively are correct without the figure in the stored context. Some of that is the 8,000-character cap on the stored `context` column rather than parametric memory, but inspect those rows before claiming retrieval is the sole source of correctness.
- **The abstention pattern was widened in v9**, which moves roughly +10 points on its own. Report the v9/v9.1 figure; do not present the v8.1 → v9 change as improved behaviour.
- **Paraphrase deltas are all positive and tiny** at n=36. Report as "no measurable paraphrase sensitivity at this sample size".
- **9% of RAG and 12% of Hybrid answers still hit the 512-token cap**, almost all of them the table-dump failures.
- **Judge and claim-level numbers in this re-score were carried over from the v9 session** (no DeepSeek key present). Valid, because the predictions they grade are unchanged — but say so if asked.
