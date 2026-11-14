# Architecture (design overview)

A conceptual description of the framework built in the research repository. The implementation is **not** part of this release.

## Data flow
```
documents -> chunking -> embeddings -> vector store
question -> retrieval -> (optional rerank / interventions) -> prompt -> reader -> answer
        run record: per question { answer, retrieved chunks with source, offsets, rank, score }
                 |
   evaluation: retrieval inspector -> evidence inspector (judge) -> generation inspector (judge) -> diagnosis
                 |
        report: per-case outcome, metrics, labels with cited measurements, plus aggregates
                 |
   experiments: one condition = one run + one report; compared case by case
```
Running and evaluating are separate stages, so judges can be swapped or re-run without regenerating answers.

## Components
* **Baseline pipeline:** loads documents, splits them into fixed-size character chunks with offsets, embeds them into a cosine vector store, retrieves the top K chunks, and builds a prompt for a reader. In every reported experiment the reader is a deterministic extractive stand-in, not a language model. Retrieval uses a deterministic tie-break so repeated runs agree.
* **Three inspectors:** independent functions over plain data. Retrieval needs no judge; evidence and generation use a judge. A judge failure is recorded, the case becomes "not evaluated", and the run continues.
* **Judge:** a pluggable interface. Only a lexical heuristic judge produced results here; interfaces for model-based judges exist but were only exercised against mocks, and no live provider was used.
* **Diagnosis:** turns the three inspector reports into an outcome and up to seven heuristic labels, each with its observed measurements kept apart from its inferred cause.
* **Experiment framework:** one-variable experiments with fingerprints of parameters, data and code, resumable runs, and read-only paired analysis over saved files.
* **Validation framework:** scenarios with a designed, known failure; an evaluator-reliability study of the judge; a component matrix that refuses to overwrite results and refuses to rerun a frozen set.
* **Dashboard:** a read-only viewer of saved result files across nine tabs. It never calls a model.

## Versioned revised components
Each revised component is implemented as a versioned, independently selectable alternative that defaults to the baseline behaviour, and the baseline implementation is kept frozen, so the baseline cell of the matrix cannot drift. The revised components are documented as a record of what was tried, not as a recommendation: the combined revised system performed worse than the baseline on the fresh validation set (see [findings](findings.md)).

## Extension points (design)
Judges, language-model providers, embedders, rerankers, readers, experiment specifications and diagnosis rules are each a small contract that a new implementation can satisfy. These were hard-coded lists, not a plugin system.
