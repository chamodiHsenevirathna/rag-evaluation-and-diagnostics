# Limitations and research integrity

What bounds every result in this overview. The research repository these results come from is not included in this release.

## Data
* **AI-authored, no human review.** The corpus, the 40-case evaluation set, the 100-scenario development set and the 120-scenario fresh validation set were each written by an AI assistant, one author per set. Ground truth is constructed, not observed, and correlated blind spots between the author and the rules are likely. The fresh set is independent in process only (separate authoring, no code access, frozen hash), not in model family and not by human review. Its freeze timestamp is self-attested.
* **Small and templated.** 40 cases in one fictional domain, seven short documents; 120 fresh scenarios in three templated-style domains, so scenarios are not fully independent. Per-category and per-label cells are too small to test.
* **Truth conventions.** 15 fresh scenarios depend on the author's convention for extra claims; excluding them did not change the verdict, but the convention for the rest is the author's.

## Judge and reader
* **Lexical judge only.** It is blind to paraphrase, derived values (arithmetic, unit conversion) and negation. In a reliability study on 81 AI-authored items it reached accuracy 0.605 (kappa 0.42). No language-model judge was used.
* **The reader is a stand-in.** An extractive sentence-quoting rule, not a language model. Its accuracy on multi-document, reasoning and contradictory questions is 0.0 in every default condition. It rarely paraphrases or invents claims, so generation-side conclusions would need a real model.
* **Fixed-size chunking cuts words,** so some stored answers contain garbled fragments. This was left unchanged by design.

## Diagnosis and outcome
* Labels are heuristic hypotheses with fallible inputs; some merge causes (for example ranking versus chunking versus top-K). In the offline experiments, 124 of 195 full-recall incorrect rows carry no label.
* The baseline outcome rule ignores extra wrong claims that retrieved text supports.

## The failed revision
* **The revised system performed worse on the fresh set** (91/120 to 78/120; exact McNemar p = 0.0106). outcome-v2.0 failed the primary independent validation criterion. It is documented, not fixed.
* A known defect in the revised lexical judge (quantity-only claims cannot be matched) and two abstention-handling problems were left as they are: any change would need a new validation set, because the fresh set is now used up.
* The explanation of why it failed is post hoc and rests on AI annotations and counterfactuals on one set.
* The revised judge's development-set gain did not replicate. Label improvements are descriptive and not significant after correction.

## Statistics
Descriptive and paired. The same 40 cases are reused in every condition, so rows overlap heavily (only 104 and 129 distinct answers across the two readers); there is no multiple-comparison correction in the baseline experiments and no variance estimate across datasets.

## Scope
* **No real-world claim,** in either direction. Nothing here says how a real RAG system behaves.
* **Live Gemini/Groq evaluation is outside the project's scope.** Provider interfaces exist and were exercised only against mocks; no stored result used a live model.
* The independent methodology review, dashboard QA, reproducibility check and licensing review were performed by AI agents, not human reviewers.

## Reproducibility
This release is **not reproducible by itself** (no code, data or result files). In the research repository, the tests, baseline run, development-set matrix, statistics and all 64 re-scored experiment conditions reproduced exactly in a clean copy, and an independent recomputation of the fresh-set statistics agreed. The exact code state at the time of the single fresh run was never committed, so its recorded code hash cannot be regenerated.

## Ownership and licensing
No licence has been chosen for this release. The copyright holder, any institutional or employer terms, and a data licence are open decisions for the project owner.
