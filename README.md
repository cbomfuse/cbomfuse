# CBOM generators scored against `cbomfuse/baseline`: asset-level table, now with the local LLM

Eight outputs are scored: the seven generators (CBOMkit, cbomkit-theia, cbom-lens, cdxgen, cryptobom-forge, cryptoscan and the local LLM, Ollama `qwen3.5:4b`) and the merged twocbom file.

## 1. Source-code corpus (5 languages, 408 explicit ground-truth assets, strict matching)

| Rank | CBOM | Precision | Recall | F1 | Accuracy¹ | Attribute accuracy² | Mean rank³ |
|:-:|---|--:|--:|--:|--:|--:|--:|
| 1 | llm (qwen3.5:4b) | 53.7% | 61.0% | 57.1% | 40.0% | 56.3% | 2.00 |
| 2 | twocbom (merged) | 54.7% | 52.7% | 53.7% | 36.7% | 75.5% | 2.13 |
| 3 | cryptoscan | 63.9% | 20.8% | 31.4% | 18.6% | 28.7% | 3.38 |
| 4 | CBOMkit | 51.6% | 19.6% | 28.4% | 16.6% | 64.5% | 4.00 |
| 5 | cdxgen | 97.5% | 9.6% | 17.4% | 9.5% | 47.0% | 4.25 |
| 6 | cryptobom-forge | 60.4% | 7.8% | 13.9% | 7.5% | 35.1% | 5.25 |
| 7= | cbomkit-theia | — | 0.0% | 0.0% | 0.0% | — | 7.50 |
| 7= | cbom-lens | — | 0.0% | 0.0% | 0.0% | — | 7.50 |

¹ Accuracy = TP / (TP + FP + FN). A classic accuracy needs true negatives, which do not exist for "assets that are not in the code"; this Jaccard-style form is the usual substitute.  
² Share of ground-truth attributes of the matched assets (primitive, parameter set / key size, mode, padding, curve, OID; material type and size) that the component reports correctly in its own CycloneDX fields. A missing attribute counts as not correct. Matched = lenient matching. "—" = the tool matched nothing.  
³ Mean of the tool's rank over eight criteria (1 = best, ties share the average rank, undefined or zero = worst): precision, recall, F1, accuracy, lenient F1, coverage-adjusted F1, evidence-level F1, attribute accuracy. Rank orders the tools by this mean (ties broken by F1; "=" marks a tie).  
"—" in Precision = the tool reported no source-code components, so precision is undefined. cbomkit-theia and cbom-lens scan files and images, not source code, so they are not meaningful on this corpus; see section 2.

llm (qwen3.5:4b) and twocbom (merged) are close: mean rank 2.00 vs 2.13. The LLM leads on recall, F1 and accuracy; the merged file leads on precision, attribute accuracy and evidence-level F1. Both are far ahead of the rest (F1 57.1% and 53.7% vs 31.4% for the third).

Counts behind the table (a claim is one reported component; TP + FN = 408 for every tool):

| CBOM | Claims | TP | FP | FN | TP (lenient) | FP (lenient) | Output for |
|---|--:|--:|--:|--:|--:|--:|---|
| llm (qwen3.5:4b) | 464 | 249 | 215 | 159 | 278 | 186 | Python, Java, Go, TypeScript, C# |
| twocbom (merged) | 393 | 215 | 178 | 193 | 238 | 155 | Python, Java, Go, TypeScript, C# |
| cryptoscan | 133 | 85 | 48 | 323 | 85 | 48 | Python, Java, Go, TypeScript, C# |
| CBOMkit | 155 | 80 | 75 | 328 | 89 | 66 | Python, Java, Go, C# |
| cdxgen | 40 | 39 | 1 | 369 | 39 | 1 | Go, C# |
| cryptobom-forge | 53 | 32 | 21 | 376 | 32 | 21 | Python, Java |
| cbomkit-theia | 0 | 0 | 0 | 408 | 0 | 0 | — |
| cbom-lens | 0 | 0 | 0 | 408 | 0 | 0 | — |

The eight criteria and the rank each tool gets on each (value, rank):

| Criterion | llm (qwen3.5:4b) | twocbom (merged) | cryptoscan | CBOMkit | cdxgen | cryptobom-forge | cbomkit-theia | cbom-lens |
|---|--:|--:|--:|--:|--:|--:|--:|--:|
| Precision (strict) | 53.7% (5.00) | 54.7% (4.00) | 63.9% (2.00) | 51.6% (6.00) | 97.5% (1.00) | 60.4% (3.00) | — (7.50) | — (7.50) |
| Recall (strict) | 61.0% (1.00) | 52.7% (2.00) | 20.8% (3.00) | 19.6% (4.00) | 9.6% (5.00) | 7.8% (6.00) | 0.0% (7.50) | 0.0% (7.50) |
| F1 (strict) | 57.1% (1.00) | 53.7% (2.00) | 31.4% (3.00) | 28.4% (4.00) | 17.4% (5.00) | 13.9% (6.00) | 0.0% (7.50) | 0.0% (7.50) |
| Accuracy (strict) | 40.0% (1.00) | 36.7% (2.00) | 18.6% (3.00) | 16.6% (4.00) | 9.5% (5.00) | 7.5% (6.00) | 0.0% (7.50) | 0.0% (7.50) |
| F1 (lenient) | 63.8% (1.00) | 59.4% (2.00) | 31.4% (4.00) | 31.6% (3.00) | 17.4% (5.00) | 13.9% (6.00) | 0.0% (7.50) | 0.0% (7.50) |
| F1, coverage-adjusted (strict) | 57.1% (1.00) | 53.7% (2.00) | 31.4% (5.00) | 33.2% (4.00) | 38.4% (3.00) | 29.5% (6.00) | — (7.50) | — (7.50) |
| Evidence-level F1 (strict) | 41.9% (3.00) | 48.2% (2.00) | 50.1% (1.00) | 25.2% (5.00) | 12.2% (6.00) | 30.0% (4.00) | — (7.50) | — (7.50) |
| Attribute accuracy | 56.3% (3.00) | 75.5% (1.00) | 28.7% (6.00) | 64.5% (2.00) | 47.0% (4.00) | 35.1% (5.00) | — (7.50) | — (7.50) |

## 2. Artifacts case (config files, keystores, certificates; 61 explicit assets, strict matching)

| Rank | CBOM | Precision | Recall | F1 | Accuracy¹ | Attribute accuracy² | Mean rank³ |
|:-:|---|--:|--:|--:|--:|--:|--:|
| 1 | cbom-lens | 88.9% | 78.7% | 83.5% | 71.6% | 88.6% | 1.20 |
| 2 | twocbom (merged) | 45.5% | 90.2% | 60.4% | 43.3% | 79.8% | 2.20 |
| 3 | cbomkit-theia | 40.3% | 50.8% | 44.9% | 29.0% | 59.4% | 3.40 |
| 4 | llm (qwen3.5:4b) | 51.1% | 37.7% | 43.4% | 27.7% | 42.1% | 3.80 |
| 5 | cryptoscan | 76.9% | 16.4% | 27.0% | 15.6% | 31.6% | 4.40 |
| 6= | CBOMkit | — | 0.0% | 0.0% | 0.0% | — | 7.00 |
| 6= | cdxgen | — | 0.0% | 0.0% | 0.0% | — | 7.00 |
| 6= | cryptobom-forge | — | 0.0% | 0.0% | 0.0% | — | 7.00 |

Same footnotes. Mean rank is over the five criteria that apply here (precision, recall, F1, accuracy, attribute accuracy): the case has one "language" and no line-level evidence, so the lenient, coverage-adjusted and evidence-level criteria would only repeat F1.

| CBOM | Claims | TP | FP | FN | TP (lenient) | FP (lenient) | Output for |
|---|--:|--:|--:|--:|--:|--:|---|
| cbom-lens | 54 | 48 | 6 | 13 | 48 | 6 | Artifacts |
| twocbom (merged) | 121 | 55 | 66 | 6 | 55 | 66 | Artifacts |
| cbomkit-theia | 77 | 31 | 46 | 30 | 31 | 46 | Artifacts |
| llm (qwen3.5:4b) | 45 | 23 | 22 | 38 | 24 | 21 | Artifacts |
| cryptoscan | 13 | 10 | 3 | 51 | 10 | 3 | Artifacts |
| CBOMkit | 0 | 0 | 0 | 61 | 0 | 0 | — |
| cdxgen | 0 | 0 | 0 | 61 | 0 | 0 | — |
| cryptobom-forge | 0 | 0 | 0 | 61 | 0 | 0 | — |

## 3. Overall ranking (scorecard, 45 metrics in families; code and artifacts tracks weighted equally)

All eight outputs:

| Rank | CBOM | Overall score⁴ | Code track | Artifacts track | Rank range over 22 variants | Previous run |
|:-:|---|--:|--:|--:|:-:|:-:|
| 1 | twocbom (merged) | 1.89 | 2.04 | 1.74 | 1–1 | 1 (1.93) |
| 2 | llm (qwen3.5:4b) | 2.94 | 2.44 | 3.44 | 2–3 | new |
| 3 | cryptoscan | 4.66 | 4.73 | 4.60 | 3–5 | 2 (3.85) |
| 4 | cbom-lens | 4.76 | 7.05 | 2.47 | 2–7.5 | 3 (4.13) |
| 5 | cdxgen | 5.30 | 3.90 | 6.71 | 3–8 | 5 (4.40) |
| 6 | CBOMkit | 5.30 | 3.90 | 6.71 | 3–7 | 4 (4.29) |
| 7 | cbomkit-theia | 5.35 | 7.05 | 3.65 | 4–8 | 6 (4.60) |
| 8 | cryptobom-forge | 5.80 | 4.90 | 6.71 | 6–8 | 7 (4.81) |

The seven generators only (the merged file left out):

| Rank | CBOM | Overall score⁴ | Code track | Artifacts track | Rank range over 22 variants | Previous run |
|:-:|---|--:|--:|--:|:-:|:-:|
| 1 | llm (qwen3.5:4b) | 2.15 | 1.74 | 2.56 | 1–2 | new |
| 2 | cryptoscan | 3.83 | 3.92 | 3.75 | 2–4 | 1 (3.02) |
| 3 | cbom-lens | 3.92 | 6.05 | 1.79 | 1–6.5 | 2 (3.32) |
| 4 | CBOMkit | 4.37 | 3.04 | 5.71 | 2–6 | 3 (3.44) |
| 5 | cbomkit-theia | 4.42 | 6.05 | 2.79 | 3–7 | 5 (3.68) |
| 6 | cdxgen | 4.43 | 3.16 | 5.71 | 3–7 | 4 (3.59) |
| 7 | cryptobom-forge | 4.88 | 4.05 | 5.71 | 5–7 | 6 (3.95) |

⁴ Each track's score is the mean, over its metric families, of the tool's mean rank on the metrics in the family (lower is better); the overall score averages the code track and the Artifacts track. "Rank range" is the best and worst position over 22 variants of the scoring (track weights, one weight per case, exact-parameter or lenient metrics only, and dropping each metric family in turn). Ranks are relative, so adding the LLM moves every other tool's score; "Previous run" is the rank and score without it.

cdxgen (5.302) and CBOMkit (5.305) are tied for practical purposes; their order depends on the third decimal.

## 4. What changed since the previous run

- The scores of CBOMkit, cdxgen, cryptobom-forge, cryptoscan, cbomkit-theia, cbom-lens are unchanged in both asset-level tables (same TP, FP, FN and attribute accuracy). Their ranks move only because there is one more competitor.
- The merged file is a different file: it now fuses the LLM output too (provenance label `ollama`).

| CBOM | Code table: previous rank → now | F1 previous → now | Artifacts table: previous rank → now | F1 previous → now |
|---|:-:|:-:|:-:|:-:|
| llm (qwen3.5:4b) | new | — → 57.1% | new | — → 43.4% |
| twocbom (merged) | 1 → 2 | 41.4% → 53.7% | 2 → 2 | 69.3% → 60.4% |
| cryptoscan | 2 → 3 | 31.4% → 31.4% | 4 → 5 | 27.0% → 27.0% |
| CBOMkit | 3 → 4 | 28.4% → 28.4% | 5= → 6= | 0.0% → 0.0% |
| cdxgen | 4 → 5 | 17.4% → 17.4% | 5= → 6= | 0.0% → 0.0% |
| cryptobom-forge | 5 → 6 | 13.9% → 13.9% | 5= → 6= | 0.0% → 0.0% |
| cbomkit-theia | 6= → 7= | 0.0% → 0.0% | 3 → 3 | 44.9% → 44.9% |
| cbom-lens | 6= → 7= | 0.0% → 0.0% | 1 → 1 | 83.5% → 83.5% |

Effect of the LLM on the merged file:

| Twocbom (merged file) | Precision | Recall | F1 | Accuracy | Attribute accuracy | Claims |
|---|--:|--:|--:|--:|--:|--:|
| Source code, previous run | 62.7% | 30.9% | 41.4% | 26.1% | 60.3% | 201 |
| Source code, now | 54.7% | 52.7% | 53.7% | 36.7% | 75.5% | 393 |
| Artifacts, previous run | 57.6% | 86.9% | 69.3% | 53.0% | 81.8% | 92 |
| Artifacts, now | 45.5% | 90.2% | 60.4% | 43.3% | 79.8% | 121 |

On source code the merge gains 89 true positives and 103 false positives. On the Artifacts case it adds 27 false positives for 2 extra true positives; the LLM recovers only 3 Artifacts assets that no static tool found.

## 5. The LLM in detail

| Language | Assets | LLM claims | LLM TP / FP / FN | LLM P / R / F1 | Merged TP / FP / FN | Merged P / R / F1 |
|---|--:|--:|--:|--:|--:|--:|
| Python | 82 | 117 | 48 / 69 / 34 | 41.0% / 58.5% / 48.2% | 44 / 45 / 38 | 49.4% / 53.7% / 51.5% |
| Java | 82 | 97 | 58 / 39 / 24 | 59.8% / 70.7% / 64.8% | 50 / 52 / 32 | 49.0% / 61.0% / 54.3% |
| Go | 81 | 78 | 49 / 29 / 32 | 62.8% / 60.5% / 61.6% | 44 / 26 / 37 | 62.9% / 54.3% / 58.3% |
| TypeScript | 81 | 65 | 38 / 27 / 43 | 58.5% / 46.9% / 52.1% | 35 / 25 / 46 | 58.3% / 43.2% / 49.6% |
| C# | 82 | 107 | 56 / 51 / 26 | 52.3% / 68.3% / 59.3% | 42 / 30 / 40 | 58.3% / 51.2% / 54.5% |

- Of its 464 source-code claims, 249 are true positives (131 algorithms, 110 related-crypto-material, 4 certificates, 4 protocols). The 215 false positives are 144 with no ground-truth counterpart (117 related-crypto-material and 27 algorithms; in the Python file, for example, ciphertexts, signatures and plaintexts, which the ground truth does not list as assets), 41 with a wrong attribute and 30 duplicates.
- Of the 408 source-code assets, 249 are recovered by the LLM, 160 by the six other generators together, and 275 by all seven together; 133 are found by none. 115 assets are found only by the LLM and 26 only by the static tools.
- Schema validity: 4 of 6 LLM files validate against CycloneDX 1.6 (the others have one or two enum errors in `cryptoProperties`); the merged file has 75 schema errors (required@components/evidence ×74, uniqueItems@components ×1).

## 6. Where the merged file loses detections

The merged file contains no asset that no generator found, so the union of the generators is its ceiling (275 on source code, 57 on Artifacts). It reaches 215 and 55. Of the 60 source-code assets that some generator recovered but the merged file does not, 52 are "collapsed" (the merged file has a component eligible for the asset, but one fused component covers several assets and only one can be credited; 51 of them are related-crypto-material) and 8 are "dropped" (no eligible component). 37 of the 60 were found only by the LLM. On Artifacts the merged file loses 2 (both collapsed).

## 7. How the numbers are computed

- **Claims and assets.** A claim is one component of kind algorithm, related-crypto-material, certificate or protocol in one case. Assets are the explicit (non-implicit) ground-truth assets: 153 algorithms, 245 related-crypto-material, 5 certificates and 5 TLS configurations in the five code cases (408), and 61 in the Artifacts case.
- **Location gate.** A claim can match an asset only if one of its lines lies inside the scenario function in which the ground truth places the asset (±1 line); constants are accepted where their scenario uses them. Import lines and comment/string decoys earn nothing. Artifacts case: the claim must name a file in which the asset occurs.
- **Identity.** An unstated attribute is consistent with anything, a stated one must agree. Strict: a wrong key size, mode, padding, curve, hash, protocol version or material size makes the claim a false positive and leaves the asset a miss. Lenient: same family, wrong parameter tolerated.
- **Matching.** One-to-one per case (Hungarian: cardinality first, then identity, then exact-line hits); the match size equals an independent Hopcroft-Karp maximum matching in all 96 tool × case × mode combinations. TP = matched claim, FP = unmatched claim (duplicate of a found asset, wrong attribute, or no counterpart), FN = asset without claim.
- **Evidence-level F1.** Every reported line or window against every ground-truth occurrence line of a compatible asset, exact lines, no one-to-one matching (1,000 ground-truth occurrences).

## 8. Caveats

- **The corpus labels itself.** Every scenario in all five source files carries a comment (`// S11: AES-256-GCM known-answer test`), a function named after it (`s11Aes256Gcm`) and an `emit("S11", "AES-256-GCM", …)` call that spells out the algorithm. A language model reads those labels; the static tools match API calls and do not. The LLM's lead is probably larger here than it would be on unlabelled real code. The Python output shows it using them: 20 of its 117 Python claims point only at the list of scenario functions at the end of the file (an algorithm read from a function name), which the location gate rejects.
- The LLM result is one run at temperature 0 of a 4B model; there is no variance estimate, and a different model or chunk size would score differently. It also hallucinates now and then (for example a "SHA-128" for SHAKE128).
- 43 scorable components of the merged file carry no location at all, so they belong to no language and are excluded from the claims. Counted as false positives they would lower twocbom's precision to 49.3%, F1 to 50.9% and accuracy to 34.2%; its mean rank would be 2.38 (llm (qwen3.5:4b) first, twocbom (merged) second).
- Strict identity lets names without parameters match ("AES" without a key size is not a contradiction). That is why cryptoscan scores well on recall and badly on attribute accuracy.
- This is a per-asset, location-aware score. The scorecard in section 3 also weights name-level inventory measures, so the two orders differ: the merged file is first in both scorecard tracks, while the LLM is first on source code in the asset table (it leads on recall, F1 and accuracy).
- Six cases only (five languages and one artifacts folder), so small differences in the middle of the table are not stable; the mean rank is a summary, not a significance test.
