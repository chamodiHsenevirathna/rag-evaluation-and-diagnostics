# Research findings

All numbers were produced offline in the research repository (not included in this release) and are bounded by [limitations](limitations.md). They concern a small, fictional, AI-authored corpus, a deterministic extractive reader stand-in and a lexical judge. They say nothing about real RAG systems or language-model readers and judges.

## Evidence tiers (never pooled)
| Tier | What it is | Used for |
|---|---|---|
| Fresh validation set | 120 AI-authored scenarios, hash frozen before evaluation, analysis pre-registered, run once | The only independent-in-process evidence about the revision |
| Development sets | 100 scenarios and 81 judge items whose errors shaped the revisions | Not evidence for the revisions |
| Stored-experiment re-scoring | 64 stored experiment conditions re-scored by revised components | Impact only; there is no ground truth |

## 1. Baseline experiments (40 cases, extractive reader, lexical judge)
* **Recall was necessary but not sufficient.** Over 16 configurations of the 27 answerable questions (432 question-configuration rows), 0 of 48 rows were correct when evidence recall was below 1. At recall 1, 49% of answers were correct, and all 195 failures were multi-document or reasoning questions. The reader copies sentences and cannot combine facts, so how often this occurs with a language model is unknown, and rows are not independent.
* **Top-K traded recall for noise.** From K = 1 to 8, evidence recall rose from 0.58 to 1.00, irrelevant chunks per question from 0.35 to 6.7, and context precision fell from 0.81 to 0.21. Accuracy on answerable questions went from 0.37 to 0.44 and stopped improving at K = 3.
* **Removing evidence was the only large effect.** Removing half or all of the gold evidence cut accuracy from 0.375 to 0.075 (12 questions worse, none better; exact sign test p = 0.0005). Chunk size, overlap, embedding model and reranking changed 0 to 3 of 40 cases, indistinguishable from no effect.
* **A contradicting document exposed a blind spot in the outcome rule.** One conflicting sentence was injected into 18 cases (for example a battery stated as 810 Wh beside the true 540 Wh). It reached the reader in all 18 and was quoted verbatim in 13 of the 18 answers. The original rule still scored 0.375, because it ignored extra wrong claims the retrieved text supports; a stricter measure fell from 0.35 to 0.15.

## 2. The revision and its fresh-set validation
The blind spot motivated a claim-level outcome rule (outcome-v2.0) and four other revised components. On the frozen fresh set, under the pre-registered rule:

| System | Correct | Accuracy (Wilson 95%) |
|---|---:|---|
| Baseline | 91/120 | 75.8% (67.5 to 82.6) |
| Revised | 78/120 | 65.0% (56.1 to 73.0) |

* 5 scenarios improved and 18 regressed (exact McNemar p = 0.0106); paired-bootstrap 95% interval of the accuracy difference −0.183 to −0.033. The guardrail on ordinary control scenarios also failed (0 improved, 2 regressed). The pre-registered verdict is **worse**, and the verdict did not change when 15 convention-dependent scenarios were excluded (p = 0.0074).
* **outcome-v2.0 is an implemented methodology that failed the primary independent validation criterion.** It is not an improvement, not validated and not recommended on this evidence.
* **Where the loss came from.** Adding components one at a time: outcome-v2.0 alone 2 improved and 15 regressed (−0.108); the other four each changed outcome accuracy by at most 0.017. Removing outcome-v2.0 from the full system recovered 11 of the 18 regressions.
* **Why, post hoc and exploratory (AI annotations, not human review).** Reading the 18 regressions: 5 were extra claims the documents do not support (the revised rule is meant to fail these, though the author's truth had accepted them), 5 were correct derived, converted or multi-source claims the lexical judge could not confirm, 2 were true in the corpus but absent from the gold text, 3 were reference claims the revised lexical judge cannot match, and 3 were other judge or abstention-parsing failures.
* **A concrete defect.** In the revised lexical judge, a claim made only of numbers and units has nothing left to match. A verbatim copy of the reference answer then failed the revised rule in 28 of 84 fresh answerable scenarios, against 0 of 70 on the development set.
* **Development success did not predict fresh-set success.** On the 100-scenario development set the revised system scored 0.88 to 0.90. Development answers were much more literal (13 of 27 correct ones were near-verbatim copies of the reference text, against 2 of 37 on the fresh set); this is a descriptive contrast, not a proven cause.
* **Secondary results, descriptive only.** Exact label-set match rose from 0.458 to 0.550 but is not significant after Holm correction (Holm p = 0.105). `irrelevant_context` was never correctly detected (0 of 9) by either version. The revised lexical judge's +5-point development gain did not replicate on fresh data.

## 3. Re-scoring the stored experiments
Only one conclusion moves under the revised components: under contradiction injection, accuracy falls from 0.375 to 0.150 (0 improved, 9 worse; exact sign test p = 0.0039). Evidence removal, top-K saturation, chunking, embedding, reranking and oracle conclusions are unchanged. This confirms the rule does what it was built to do on its design cases; it is not independent evidence that the rule is right, and the fresh-set result shows it over-penalises on other data.

## 4. What these findings support, and what they do not
**Support:** the three-inspector design separates retrieval from generation effects in a controlled harness; component-level attribution located the regression in one component, which a single before/after comparison would not have shown; **a stricter evaluator is not automatically a better evaluator**, because a rule that asks a fallible judge to confirm every claim punishes good answers for the judge's weakness.

**Do not support:** anything about how real RAG systems or language-model readers and judges behave, whether any revised component helps on real systems, or the accuracy of the labels on real failures. The original question (why real RAG systems fail) is not answered; this project builds and stress-tests an offline instrument for it.

## 5. Not done
A new, human-reviewed validation set before any further rule change; a judge that can verify paraphrase and derived values; a human audit of truth labels; real RAG failures instead of constructed ones; live Gemini/Groq evaluation (out of scope).
