# CBOM generators scored against `cbomfuse/baseline`: asset-level table

## 1. Source-code corpus (5 languages, 408 explicit ground-truth assets, strict matching)

| Rank | CBOM | Precision | Recall | F1 | Accuracy¹ | Attribute accuracy² | Mean rank³ |
|:-:|---|--:|--:|--:|--:|--:|--:|
| 1 | twocbom (merged) | 62.7% | 30.9% | 41.4% | 26.1% | 60.3% | 1.50 |
| 2 | cryptoscan | 63.9% | 20.8% | 31.4% | 18.6% | 28.7% | 2.63 |
| 3 | CBOMkit | 51.6% | 19.6% | 28.4% | 16.6% | 64.5% | 3.00 |
| 4 | cdxgen | 97.5% | 9.6% | 17.4% | 9.5% | 47.0% | 3.38 |
| 5 | cryptobom-forge | 60.4% | 7.8% | 13.9% | 7.5% | 35.1% | 4.50 |
| 6= | cbomkit-theia | — | 0.0% | 0.0% | 0.0% | — | 6.50 |
| 6= | cbom-lens | — | 0.0% | 0.0% | 0.0% | — | 6.50 |

¹ Accuracy = TP / (TP + FP + FN). A classic accuracy needs true negatives, which do not exist for "assets that are not in the code"; this Jaccard-style form is the usual substitute.  
² Share of ground-truth attributes of the matched assets (primitive, parameter set / key size, mode, padding, curve, OID; material type and size) that the component reports correctly in its own CycloneDX fields. A missing attribute counts as not correct. Matched = lenient matching. "—" = the tool matched nothing.  
³ Mean of the tool's rank over eight criteria (1 = best, ties share the average rank, undefined or zero = worst): precision, recall, F1, accuracy, lenient F1, coverage-adjusted F1, evidence-level F1, attribute accuracy. Rank orders the tools by this mean (ties broken by F1; "=" marks a tie).  
"—" in Precision = the tool reported no source-code components, so precision is undefined. cbomkit-theia and cbom-lens scan files and images, not source code, so they are not meaningful on this corpus; see section 2.

Counts behind the table (a claim is one reported component; TP + FN = 408 for every tool):

| CBOM | Claims | TP | FP | FN | TP (lenient) | FP (lenient) | Output for |
|---|--:|--:|--:|--:|--:|--:|---|
| twocbom (merged) | 201 | 126 | 75 | 282 | 131 | 70 | Python, Java, Go, TypeScript, C# |
| cryptoscan | 133 | 85 | 48 | 323 | 85 | 48 | Python, Java, Go, TypeScript, C# |
| CBOMkit | 155 | 80 | 75 | 328 | 89 | 66 | Python, Java, Go, C# |
| cdxgen | 40 | 39 | 1 | 369 | 39 | 1 | Go, C# |
| cryptobom-forge | 53 | 32 | 21 | 376 | 32 | 21 | Python, Java |
| cbomkit-theia | 0 | 0 | 0 | 408 | 0 | 0 | — |
| cbom-lens | 0 | 0 | 0 | 408 | 0 | 0 | — |

The eight criteria and the rank each tool gets on each (value, rank):

| Criterion | twocbom (merged) | cryptoscan | CBOMkit | cdxgen | cryptobom-forge | cbomkit-theia | cbom-lens |
|---|--:|--:|--:|--:|--:|--:|--:|
| Precision (strict) | 62.7% (3.00) | 63.9% (2.00) | 51.6% (5.00) | 97.5% (1.00) | 60.4% (4.00) | — (6.50) | — (6.50) |
| Recall (strict) | 30.9% (1.00) | 20.8% (2.00) | 19.6% (3.00) | 9.6% (4.00) | 7.8% (5.00) | 0.0% (6.50) | 0.0% (6.50) |
| F1 (strict) | 41.4% (1.00) | 31.4% (2.00) | 28.4% (3.00) | 17.4% (4.00) | 13.9% (5.00) | 0.0% (6.50) | 0.0% (6.50) |
| Accuracy (strict) | 26.1% (1.00) | 18.6% (2.00) | 16.6% (3.00) | 9.5% (4.00) | 7.5% (5.00) | 0.0% (6.50) | 0.0% (6.50) |
| F1 (lenient) | 43.0% (1.00) | 31.4% (3.00) | 31.6% (2.00) | 17.4% (4.00) | 13.9% (5.00) | 0.0% (6.50) | 0.0% (6.50) |
| F1, coverage-adjusted (strict) | 41.4% (1.00) | 31.4% (4.00) | 33.2% (3.00) | 38.4% (2.00) | 29.5% (5.00) | — (6.50) | — (6.50) |
| Evidence-level F1 (strict) | 48.3% (2.00) | 50.1% (1.00) | 25.2% (4.00) | 12.2% (5.00) | 30.0% (3.00) | — (6.50) | — (6.50) |
| Attribute accuracy | 60.3% (2.00) | 28.7% (5.00) | 64.5% (1.00) | 47.0% (3.00) | 35.1% (4.00) | — (6.50) | — (6.50) |

## 2. Artifacts case (config files, keystores, certificates; 61 explicit assets, strict matching)

| Rank | CBOM | Precision | Recall | F1 | Accuracy¹ | Attribute accuracy² | Mean rank³ |
|:-:|---|--:|--:|--:|--:|--:|--:|
| 1 | cbom-lens | 88.9% | 78.7% | 83.5% | 71.6% | 88.6% | 1.20 |
| 2 | twocbom (merged) | 57.6% | 86.9% | 69.3% | 53.0% | 81.8% | 2.00 |
| 3 | cbomkit-theia | 40.3% | 50.8% | 44.9% | 29.0% | 59.4% | 3.20 |
| 4 | cryptoscan | 76.9% | 16.4% | 27.0% | 15.6% | 31.6% | 3.60 |
| 5= | CBOMkit | — | 0.0% | 0.0% | 0.0% | — | 6.00 |
| 5= | cdxgen | — | 0.0% | 0.0% | 0.0% | — | 6.00 |
| 5= | cryptobom-forge | — | 0.0% | 0.0% | 0.0% | — | 6.00 |

Same footnotes. Mean rank is over the five criteria that apply here (precision, recall, F1, accuracy, attribute accuracy): the case has one "language" and no line-level evidence, so the lenient, coverage-adjusted and evidence-level criteria would only repeat F1.

| CBOM | Claims | TP | FP | FN | TP (lenient) | FP (lenient) | Output for |
|---|--:|--:|--:|--:|--:|--:|---|
| cbom-lens | 54 | 48 | 6 | 13 | 48 | 6 | Artifacts |
| twocbom (merged) | 92 | 53 | 39 | 8 | 53 | 39 | Artifacts |
| cbomkit-theia | 77 | 31 | 46 | 30 | 31 | 46 | Artifacts |
| cryptoscan | 13 | 10 | 3 | 51 | 10 | 3 | Artifacts |
| CBOMkit | 0 | 0 | 0 | 61 | 0 | 0 | — |
| cdxgen | 0 | 0 | 0 | 61 | 0 | 0 | — |
| cryptobom-forge | 0 | 0 | 0 | 61 | 0 | 0 | — |

## 3. How the numbers are computed

- **Claims and assets.** A claim is one component of kind algorithm, related-crypto-material, certificate or protocol in one case. Assets are the explicit (non-implicit) ground-truth assets: 153 algorithms, 245 related-crypto-material, 5 certificates and 5 TLS configurations in the five code cases (408), and 61 in the Artifacts case.
- **Location gate.** A claim can match an asset only if one of its lines lies inside the scenario function in which the ground truth places the asset (±1 line); constants are accepted where their scenario uses them. Import lines and comment/string decoys earn nothing. Artifacts case: the claim must name a file in which the asset occurs.
- **Identity.** An unstated attribute is consistent with anything, a stated one must agree. Strict: a wrong key size, mode, padding, curve, hash, protocol version or material size makes the claim a false positive and leaves the asset a miss. Lenient: same family, wrong parameter tolerated.
- **Matching.** One-to-one per case (Hungarian: cardinality first, then identity, then exact-line hits); the match size equals an independent Hopcroft-Karp maximum matching in all 84 tool x case x mode combinations. TP = matched claim, FP = unmatched claim (duplicate of a found asset, wrong attribute, or no counterpart), FN = asset without claim.
- **Evidence-level F1.** Every reported line or window against every ground-truth occurrence line of a compatible asset, exact lines, no one-to-one matching (1,000 ground-truth occurrences).

## 4. Check against your earlier table

The scoring was rebuilt from the method text of the earlier ranking report. On the two outputs whose claim counts are the same as in that report, it reproduces its true-positive counts to within one asset: cryptobom-forge on Python 14 of 22 claims here vs 15 of 22 implied by the screenshot (68.2% precision, 3.7% recall), and CBOMkit on Java, Go and C# 65 of 137 vs 64 of 137 (46.7%, 15.7%). Attribute accuracy is within three points on both (34.0% vs 36.7%, 64.6% vs 62.8%). The other rows are not comparable with that table because the files changed: cdxgen now yields assets for Go and C# (before: C# and TypeScript), CBOMkit now has Python, cryptobom-forge now has Java, and the merged file is a larger merge (201 claims here, about 117 there).

## 5. Caveats

- 42 scorable components of the merged file carry no location at all, so they belong to no language and are excluded from the claims. Counted as false positives they would lower twocbom's precision to 51.9%, F1 to 38.7% and accuracy to 24.0%; its mean rank would be 1.63 (order of the top tool unchanged: twocbom (merged) first).
- Strict identity lets names without parameters match ("AES" without a key size is not a contradiction). That is why cryptoscan scores well on recall and badly on attribute accuracy.
- This is a per-asset, location-aware score, like your earlier table. The scorecard I sent before also weighted name-level inventory measures, so the order of cryptoscan, CBOMkit and cdxgen differs between the two.
- Six cases only (five languages and one artifacts folder), so small differences in the middle of the table are not stable; the mean rank is a summary, not a significance test.
