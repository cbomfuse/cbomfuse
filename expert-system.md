# Reconciling Multi-Tool CBOMs with a CLIPS Expert System on Top of BF-CBOM

auditable rule layer that reconciles disagreeing CBOM generators into one schema-valid CycloneDX document with per-field provenance.

CLIPS expert system + confidence scores + risk scores + PQC classification

BF-CBOM is a comparison harness, not a data model. 

Confidence and risk numbers produced without a labeled corpus and deployment context would be arbitrary.

explainable, provenance-preserving reconciliation of heterogeneous CBOMs, evaluated against a hand-labeled ground-truth corpus.

Treat confidence as calibrated, measured output, not designed-in.

Restrict risk to a transparent, context-parameterized classification rather than a single number. 

CBOMkit already ships a basic quantum-safe compliance check.

BF-CBOM  is an orchestration and comparison framework. 

The tools emit different CycloneDX versions.

CryptobomForge's output is the pre-standard IBM "1.4-cbom-1.0" format.

CLIPS has no JSON I/O and no native uncertainty.


BF-CBOM: Uncovering Cryptographic Assets Through Comparative CBOM Analysis at Scale.

A comparative analysis that treats cross-tool matching as an n-way model-matching problem. 

It uses RaQuN over components vectorized from selected JSON fields embedded with MiniLM-v2.

BF-CBOM is not designed to resolve discrepancies but to actively surface them.
reported cryptographic assets were strikingly inconsistent.


Overlaps were rare, with almost no lexically exact matches.
 

BF-CBOM defines no data model, schema extension, or merged-output format. 


BF-CBOM consumes whatever JSON the tools emit and compares them at the component level.


BF-CBOM reproducible containerized execution of heterogeneous generators, which saves weeks of toolchain pain (CodeQL alone is heavy). 


It pins specific generator versions, and generators change output formats quickly.

 
BF-CBOM is merely an execution substrate.

reconciler should accept arbitrary CBOM files apart from BF-CBOM.


Practically, a 1.6 document and a 1.7 document from two tools describing the same asset may legitimately use different fields.


Model elements relevant to reconciliation.

































