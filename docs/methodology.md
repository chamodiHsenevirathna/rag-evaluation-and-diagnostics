# Methodology

This describes the method as designed and run in the research repository (not included in this release). Every result it produced is bounded by [limitations](limitations.md).

## Three evaluation dimensions
1. **Retrieval:** did the right evidence reach the model's context? Checked deterministically against labelled gold evidence, by exact text span.
2. **Evidence:** are the answer's claims supported, contradicted or unaddressed by the retrieved text? The answer is split into claims and each is checked by a judge. In every result reported here the judge is a lexical overlap heuristic, not a language model.
3. **Generation:** how faithfully and completely does the answer use the context? Derived from the first two plus the judge.

The three inspectors are independent; each reports separately, so "the evidence was there" and "the answer used it correctly" are different facts.

## Outcome and failure labels
Diagnosis runs only when the observed outcome is incorrect.

**Baseline outcome rule (outcome-v1).** For a question the corpus cannot answer, the answer is correct only if it abstains. For a case with contradictory documents, it is correct only if it flags the conflict and covers the reference claims. Otherwise it is correct if it does not abstain, covers at least 80% of the reference claims, and contains no claim the retrieved text contradicts. This rule does **not** penalise extra wrong claims that the retrieved text happens to support, which is the blind spot that motivated the revision below.

Seven heuristic labels can then be attached (multi-label). Each records the measurements that triggered it separately from an *inferred* cause, worded as a hypothesis.

| Label | Fires when (observed) | Inferred cause (hypothesis) |
|---|---|---|
| `missing_evidence` | gold evidence is absent from the indexed corpus | information not in the knowledge base |
| `retrieval_failure` | gold evidence is in the corpus but covered less than 80% by retrieved chunks | ranking, chunking or top-K (not distinguished) |
| `irrelevant_context` | unconfirmed claims are supported only by irrelevant chunks | noise may have influenced the answer |
| `unsupported_generation` | claims supported neither by context nor by the reference | background knowledge or fabrication (not distinguished) |
| `context_misinterpretation` | all gold evidence retrieved, but the answer contradicts it, abstains, or uses less than half of it | misread or under-used context (or judge error) |
| `contradictory_evidence` | case has conflicting documents, they were retrieved, and the answer does not flag it | conflict not surfaced |
| `insufficient_evidence` | the corpus has no evidence and the answer did not abstain | answered an unanswerable question |

Labels are heuristic hypotheses, not causal conclusions. No confidence score is emitted, and an incorrect answer that matches no rule is marked unexplained.

## Controlled experiments
Ten one-variable experiments (32 conditions, each run with two reader settings, 64 in all) change chunk size, overlap, embedding model, Top-K, reranking, injected irrelevant text, removed evidence, an injected contradiction, and oracle evidence. All use the same 40 cases, a deterministic extractive reader stand-in (not a language model) and the lexical judge. Conditions are compared case by case with paired exact tests; rows are not independent, so these are descriptive.

## Revised components and the component matrix
After the experiments, five independently versioned components were built: a claim-level outcome rule (outcome-v2.0), a revised lexical judge (lexical-v2.0), a claim extractor that flags and reports suspect fragments (claims-v2.0), a paraphrase-aware abstention detector (abstention-v2.0) and revised label rules (diagnosis-v2.0). A 17-cell matrix scores the same scenarios with every combination of baseline and revised components (a build-up path, a factorial grid, leave-one-out and isolation cells), so a change in accuracy can be attributed to a component and not only to the whole.

## Validation protocol
1. A fresh set of 120 AI-authored scenarios (three new fictional domains) was frozen: its SHA-256 hash was recorded before any evaluation.
2. The analysis was pre-registered: primary test (paired exact McNemar with a seeded paired bootstrap interval), a guardrail on ordinary control scenarios, secondary endpoints with Holm correction, and a convention check.
3. The baseline and revised systems were run **once**. No rule, threshold or scenario was changed afterwards, and the set is not rerun.
4. Three evidence tiers are never pooled: the fresh set, the development sets that shaped the revisions (not evidence for them), and re-scoring of stored experiments (impact analysis without ground truth).

Statistics: exact McNemar (two-sided exact binomial on discordant pairs), paired percentile bootstrap (seed fixed, 10,000 resamples, scenario as the unit), Holm correction, Wilson intervals (Clopper–Pearson for small counts).

## Principles
Deterministic first; one variable at a time; every dropped or suspect claim reported; no accuracy claim beyond what constructed-scenario validation measures, and none about real-world RAG systems.
