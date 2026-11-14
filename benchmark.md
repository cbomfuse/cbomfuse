# CBOM tool ranking against the `cbomfuse/baseline` ground truth

Four CBOMs, scored against the ground truth of the 5-language / 24-scenario [cbomfuse/baseline](https://github.com/cbomfuse/baseline) corpus (HEAD `6e29566`).

## 1. Result

**Ranking (whole corpus, strict matching, 408 scorable ground-truth assets):**

| Rank | CBOM | Precision | Recall | F1 | Accuracy¹ | Attribute accuracy² | Mean rank³ |
|:-:|---|--:|--:|--:|--:|--:|--:|
| **1** | **twocbom (merged)** | 73.5% | 21.1% | 32.8% | 19.6% | 69.5% | 1.25 |
| **2** | **CBOMkit** | 46.7% | 15.7% | 23.5% | 13.3% | 62.8% | 2.13 |
| **3** | **cdxgen** | 94.1% | 3.9% | 7.5% | 3.9% | 45.7% | 3.00 |
| **4** | **cryptobom-forge** | 68.2% | 3.7% | 7.0% | 3.6% | 36.7% | 3.63 |

¹ Accuracy = TP / (TP + FP + FN). A classic accuracy needs true negatives, which do not exist for “assets that are not in the code”; this Jaccard-style form is a common substitute. ² Share of ground-truth attributes (primitive, key size, mode, padding, curve, OID; material type and size) that a matched component reports correctly (missing counts as not correct). ³ Mean of the tool's rank over the eight criteria in §4.1 (1 = best).

**Same table with lenient matching** (a component at the right place with the right algorithm still counts as found when its key/digest/material *size* is wrong, or a key is typed secret instead of private; the error is then scored under attribute accuracy):

| CBOM | Precision | Recall | F1 | Accuracy | Precision ignoring duplicates |
|---|--:|--:|--:|--:|--:|
| twocbom (merged) | 81.2% | 23.3% | 36.2% | 22.1% | 93.1% |
| CBOMkit | 59.9% | 20.1% | 30.1% | 17.7% | 80.4% |
| cdxgen | 94.1% | 3.9% | 7.5% | 3.9% | 94.1% |
| cryptobom-forge | 72.7% | 3.9% | 7.4% | 3.9% | 88.9% |

**What this says**

- **twocbom (merged) is first on every full-corpus measure except precision.** It is the only output that covers all five languages and finds 86 of 408 assets (21.1%; 46% of the 153 algorithm assets).
- **CBOMkit is second.** It finds 64 assets but is the least precise tool (46.7%): of its 73 strict false positives, 39 are right-place detections with a wrong key or material size or key type (e.g. AES-128 reported for AES-256, RSA-2048 for RSA-3072), 20 are material components with no ground-truth counterpart, and 14 are duplicates.
- **cdxgen is third, and is the most precise tool (94.1%, 16 of 17 claims correct)**, but it finds only 3.9% of the assets: it reports names without parameters (no key size, mode or curve), no key/IV/salt material beyond three C# constants, and nothing in Python, Java or Go.
- **cryptobom-forge is fourth only because it covers one of five languages.** On Python it finds 15 of 82 assets (18.3%) at 68.2% precision. On the whole corpus it is effectively tied with cdxgen (15 vs 16 TPs; F1 7.0% vs 7.5%); restricted to the languages each tool actually produced output for, it is ahead of cdxgen (28.8% vs 17.8% F1).
- **Nobody is close to complete.** Best recall is 21.1%. No tool detects a certificate (0/5), Argon2id or bcrypt (0/10 across all languages), and related-crypto-material (keys, IVs, salts, secrets — 245 of the 408 assets) tops out at 6.1% recall.
- **Merging helped, with losses.** twocbom beats naive concatenation of the three standalone outputs on precision (73.5% vs 52.8%) and F1 (32.8% vs 31.8%) by removing duplicates, but it loses 7 assets the standalone tools had found and introduces a mis-parsed RSA key size (§5).

## 2. What was scored

| Input | Producer / format | Languages with output | Crypto components | Library components |
|---|---|---|--:|--:|
| `5ef8776e-cbomkit.cbom.json` | CBOMkit (PQCA, sonar-cryptography static analysis), CycloneDX 1.6, scanned commit `7cb9d9a` | Java, Go, C# | 137 | 0 |
| `13f0d331-cdxgen.cbom.json` | cdxgen 13.7.0 + dosai / js-ast, CycloneDX 1.7 | TypeScript, C# | 12 | 14 |
| `262fc2a5-cryptobom-forge.cbom.json` | cryptobom-forge (IBM CBOM 1.4 draft, CodeQL-based; 5-line detection windows) | Python | 22 | 0 |
| `698d7ecb-merged.cdx.json` | twocbom merge of the three above (profile `strict`, `2cbom:conformant=false`), CycloneDX | Python, Java, Go, TypeScript, C# | 79 | 14 |

**Ground truth.** One program per language (Python, Java, Go, TypeScript, C#) implementing the same 24 crypto scenarios (S01–S24). Per language the repository lists every asset, every source occurrence (role, scope, relation), non-asset “decoy” mentions in comments/strings, and the libraries. Scored here: **408 explicit assets** (153 algorithms, 245 related-crypto-material, 5 TLS protocol configurations, 5 certificates), 23 libraries and 552 decoy mentions. The 13 *implicit* assets (e.g. SHA-384 and AES-128-GCM from the TLS 1.3 cipher-suite list) have no source line and are not scored.

The CBOMkit file was produced from commit `7cb9d9a`; I scored against HEAD `6e29566`. A `git diff` between the two shows only build artifacts, `build-all.sh` and `go.mod`/`go.sum` changed — the five source files and all `ground-truth.json` files are identical, so line numbers align.

## 3. Method

1. **Claims.** A claim is one component in one language (a component with locations in two languages is two claims). Locations are unioned from every place a tool puts them: `evidence.occurrences`, cdxgen's `cdx:crypto:sourceLocation` properties and `evidence.identity` methods, and the IBM-format `detectionContext` windows. The merged BOM's per-line expansion of forge windows is re-grouped into windows.
2. **Identity.** Each claim is normalised to an algorithm family plus the attributes it states (key size, mode, padding, curve, hash) — e.g. `SHA-256` (CBOMkit), `sha-256` (cdxgen), `SHA256` (forge) and `sha2-256` (merged) all become *SHA-256*; `AES-128-CBC-PKCS5` becomes *AES / 128 / CBC / PKCS#7-family padding*; an `unresolved:…` merged component is resolved through its alias list. Materials are identified by type (salt, password, secret-key, IV/nonce, private-key, …) with size as an attribute. An unstated attribute is consistent with anything; a stated one must agree. A few under-specified names are accepted as the ground-truth asset they plausibly denote (`EC` → ECDSA/ECDH P-256, `ChaCha20` or `Poly1305` → ChaCha20-Poly1305, `SHAKE` → SHAKE128, `ECDH` → X25519).
3. **Location gate.** A claim may only match a ground-truth asset if one of its lines falls inside the scenario function in which the ground truth places that asset (±1 line). Hard-coded constants (module-level keys, salts, passwords) are also accepted where their scenario *uses* them, because tools report use sites while the ground truth records the definition. Import lines, comments and strings earn nothing — the ground truth files those under libraries and decoys. A claim with several lines can match through any of them.
4. **One-to-one assignment.** Claims and assets are matched with the Hungarian algorithm (maximum cardinality first, then best identity, then exact-line hits), per language. I cross-checked the match counts with an independent maximum-cardinality matching (networkx): identical for all 8 tool/mode combinations.
5. **Outcomes.** *TP*: a claim matched to an asset. *FP-spurious*: an unmatched claim with no compatible asset in reach. *FP-redundant*: an unmatched claim that duplicates an asset another claim already matched. *FN*: an asset with no claim. TP + FN = 408 for every tool.
6. **Strict vs lenient.** *Strict* requires the identity to be consistent (a wrong key size, mode, padding, curve, hash or material size makes the claim a false positive **and** leaves the asset a miss) and counts duplicates as false positives. *Lenient* tolerates only a wrong **size** (key / digest / parameter-set / material size) and secret-vs-private-vs-public key typing: the claim is a TP and the error is scored under attribute accuracy. A different algorithm, mode, padding, curve, hash or material type never matches in either mode.
7. **Metrics.** Precision = TP/(TP+FP); Recall = TP/(TP+FN); F1 = their harmonic mean; Accuracy = TP/(TP+FP+FN). “Coverage-adjusted” restricts FN to the languages in which a tool produced any output (CBOMkit: Java, Go, C#; cdxgen: C#, TypeScript; forge: Python; merged: all five), which separates *coverage* from *quality on supported languages*.
8. **Secondary views.** Attribute accuracy on matched assets; *evidence-level* precision/recall (every reported line/window vs. every ground-truth occurrence line of a compatible asset, exact lines, no slack); decoy hits (reported lines that fall on a comment/string decoy and on no real occurrence); library detection by purl.

## 4. Detailed results

### 4.1 Ranking per criterion

| Criterion | CBOMkit | cdxgen | cryptobom-forge | twocbom (merged) | Order |
|---|--:|--:|--:|--:|---|
| Precision (strict) | 46.7% | 94.1% | 68.2% | 73.5% | cdxgen > merged > forge > CBOMkit |
| Recall (strict) | 15.7% | 3.9% | 3.7% | 21.1% | merged > CBOMkit > cdxgen ≈ forge |
| F1 (strict) | 23.5% | 7.5% | 7.0% | 32.8% | merged > CBOMkit > cdxgen > forge |
| Accuracy (strict) | 13.3% | 3.9% | 3.6% | 19.6% | merged > CBOMkit > cdxgen ≈ forge |
| F1 (lenient) | 30.1% | 7.5% | 7.4% | 36.2% | merged > CBOMkit > cdxgen ≈ forge |
| F1, coverage-adjusted (strict) | 33.5% | 17.8% | 28.8% | 32.8% | CBOMkit > merged > forge > cdxgen |
| Evidence-level F1 (strict) | 14.1% | 4.5% | 10.7% | 24.6% | merged > CBOMkit > forge > cdxgen |
| Attribute accuracy | 62.8% | 45.7% | 36.7% | 69.5% | merged > CBOMkit > cdxgen > forge |
| **Mean rank (1 = best)** | **2.13** | **3.00** | **3.63** | **1.25** | merged > CBOMkit > cdxgen > forge |

“≈” marks differences below 0.5 percentage points. The composite ordering equals the strict-F1 ordering. The only close call is 3rd vs 4th (cdxgen vs forge), which trades precision for coverage and flips in the coverage-adjusted view (§6).

### 4.2 Strict counts

| CBOM | Claims | TP | FP spurious | FP redundant | FN | Precision | Precision ignoring duplicates | Recall | F1 | Accuracy |
|---|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|
| twocbom (merged) | 117 | 86 | 25 | 6 | 322 | 73.5% | 77.5% | 21.1% | 32.8% | 19.6% |
| CBOMkit | 137 | 64 | 59 | 14 | 344 | 46.7% | 52.0% | 15.7% | 23.5% | 13.3% |
| cdxgen | 17 | 16 | 1 | 0 | 392 | 94.1% | 94.1% | 3.9% | 7.5% | 3.9% |
| cryptobom-forge | 22 | 15 | 3 | 4 | 393 | 68.2% | 83.3% | 3.7% | 7.0% | 3.6% |

### 4.3 By asset class

Recall (and precision of that class's claims) — strict.

| Class | GT assets | CBOMkit | cdxgen | cryptobom-forge | twocbom (merged) |
|---|--:|--:|--:|--:|--:|
| Algorithms | 153 | 48/153 (31%) · P 76% | 13/153 (8%) · P 93% | 14/153 (9%) · P 70% | 71/153 (46%) · P 80% |
| Related-crypto-material | 245 | 15/245 (6%) · P 21% | 3/245 (1%) · P 100% | 1/245 (0%) · P 50% | 14/245 (6%) · P 52% |
| Protocols (TLS 1.3) | 5 | 1/5 (20%) · P 100% | 0/5 (0%) · P — | 0/5 (0%) · P — | 1/5 (20%) · P 100% |
| Certificates | 5 | 0/5 (0%) · P — | 0/5 (0%) · P — | 0/5 (0%) · P — | 0/5 (0%) · P — |

Lenient view of the same table:

| Class | CBOMkit | cdxgen | cryptobom-forge | twocbom (merged) |
|---|--:|--:|--:|--:|
| Algorithms | 55/153 (36%) · P 87% | 13/153 (8%) · P 93% | 14/153 (9%) · P 70% | 79/153 (52%) · P 89% |
| Related-crypto-material | 26/245 (11%) · P 36% | 3/245 (1%) · P 100% | 2/245 (1%) · P 100% | 15/245 (6%) · P 56% |
| Protocols (TLS 1.3) | 1/5 (20%) · P 100% | 0/5 (0%) · P — | 0/5 (0%) · P — | 1/5 (20%) · P 100% |
| Certificates | 0/5 (0%) · P — | 0/5 (0%) · P — | 0/5 (0%) · P — | 0/5 (0%) · P — |

Material recall is low everywhere. Hard-coded constants (the most visible class) are found at ≤ 10 %; CBOMkit's best class is CSPRNG-generated salts and IVs; key-generation outputs, KDF outputs and key-agreement / KEM secrets are essentially undetected. Material recall by provenance (strict, found/total):

| Provenance | Total | CBOMkit | cdxgen | cryptobom-forge | twocbom (merged) |
|---|--:|--:|--:|--:|--:|
| Hard-coded constant | 80 | 6 | 3 | 0 | 8 |
| From CSPRNG (salt, IV, nonce) | 45 | 8 | 0 | 0 | 4 |
| Key generation (key pairs, keys) | 70 | 0 | 0 | 1 | 1 |
| KDF output | 35 | 1 | 0 | 0 | 1 |
| Key-agreement / KEM secret | 15 | 0 | 0 | 0 | 0 |

### 4.4 By language

Cells are precision / recall / F1 (strict); “—” = the tool produced no output for the language (precision undefined). Best F1 per language in **bold**.

| Language | GT assets | CBOMkit | cdxgen | cryptobom-forge | twocbom (merged) |
|---|--:|--:|--:|--:|--:|
| Python | 82 | — | — | 68% / 18% / 29% | **83% / 18% / 30%** |
| Java | 82 | 35% / 45% / 39% | — | — | **57% / 39% / 46%** |
| Go | 81 | **94% / 21% / 34%** | — | — | 88% / 19% / 31% |
| TypeScript | 81 | — | **100% / 9% / 16%** | — | 100% / 9% / 16% |
| C# | 82 | 83% / 12% / 21% | 90% / 11% / 20% | — | **89% / 21% / 34%** |

### 4.5 Coverage-adjusted (languages with output only)

| CBOM | Languages | TP | FN (in those languages) | Precision | Recall | F1 (strict) | F1 (lenient) |
|---|---|--:|--:|--:|--:|--:|--:|
| CBOMkit | Java, Go, C# | 64 | 181 | 46.7% | 26.1% | 33.5% | 42.9% |
| twocbom (merged) | Python, Java, Go, TypeScript, C# | 86 | 322 | 73.5% | 21.1% | 32.8% | 36.2% |
| cryptobom-forge | Python | 15 | 67 | 68.2% | 18.3% | 28.8% | 30.8% |
| cdxgen | TypeScript, C# | 16 | 147 | 94.1% | 9.8% | 17.8% | 17.8% |

### 4.6 Attribute accuracy (matched assets, lenient matching)

| CBOM | Attributes checked | Correct | Wrong | Missing | Accuracy | Precision of reported attributes | Completeness | Strict TPs with a fully specified identity |
|---|--:|--:|--:|--:|--:|--:|--:|--:|
| twocbom (merged) | 285 | 198 | 31 | 56 | 69.5% | 86.5% | 80.4% | 65/86 |
| CBOMkit | 231 | 145 | 34 | 52 | 62.8% | 81.0% | 77.5% | 46/64 |
| cdxgen | 46 | 21 | 5 | 20 | 45.7% | 80.8% | 56.5% | 10/16 |
| cryptobom-forge | 49 | 18 | 4 | 27 | 36.7% | 81.8% | 44.9% | 13/15 |

Attribute accuracy = correct ÷ all ground-truth attributes of matched assets; reported-attribute precision = correct ÷ (correct + wrong); completeness = (correct + wrong) ÷ all. Every tool is 81–86 % right *when it states an attribute*; they differ in how often they state one (the last column counts TPs whose name/parameters leave no required attribute unstated). By attribute (correct / wrong / missing):

| Attribute | CBOMkit | cdxgen | cryptobom-forge | twocbom (merged) |
|---|--:|--:|--:|--:|
| Primitive | 51 / 4 / 0 | 7 / 3 / 3 | 10 / 3 / 1 | 70 / 7 / 2 |
| Parameter set (key/digest size) | 21 / 10 / 8 | 0 / 0 / 11 | 0 / 0 / 11 | 27 / 11 / 21 |
| Mode | 7 / 0 / 7 | 0 / 0 / 1 | 2 / 0 / 0 | 9 / 0 / 7 |
| Padding | 5 / 0 / 7 | 0 / 0 / 1 | 3 / 0 / 0 | 12 / 0 / 2 |
| Curve | 6 / 0 / 3 | 0 / 0 / 1 | 0 / 0 / 1 | 4 / 0 / 7 |
| OID | 28 / 9 / 15 | 11 / 2 / 0 | 0 / 0 / 14 | 51 / 12 / 13 |
| Material type | 16 / 10 / 0 | 3 / 0 / 0 | 1 / 1 / 0 | 15 / 0 / 0 |
| Material size | 11 / 1 / 12 | 0 / 0 / 3 | 2 / 0 / 0 | 10 / 1 / 4 |

### 4.7 Evidence-level precision and recall

Scores every reported line (or window) and every ground-truth occurrence line, without one-to-one matching. This is where duplicate-heavy and window-based outputs are judged on the evidence they give an analyst.

| CBOM | Reported locations | Correct | Precision | GT occurrence lines found (of 1000) | Recall | F1 |
|---|--:|--:|--:|--:|--:|--:|
| twocbom (merged) | 208 | 126 | 60.6% | 154 | 15.4% | 24.6% |
| CBOMkit | 191 | 88 | 46.1% | 83 | 8.3% | 14.1% |
| cdxgen | 26 | 23 | 88.5% | 23 | 2.3% | 4.5% |
| cryptobom-forge | 33 | 27 | 81.8% | 57 | 5.7% | 10.7% |

forge and merged report 5-line windows for Python, so each Python window can cover several occurrence lines at once; their recall here is flattered relative to single-line reporters.

### 4.8 Noise: false positives, duplicates and decoys

| CBOM | Strict false positives | …right place & algorithm, wrong size / key type | …no ground-truth counterpart | …duplicate of a found asset | Claims per TP | Locations on decoy lines |
|---|--:|--:|--:|--:|--:|--:|
| twocbom (merged) | 31 | 18 | 7 | 6 | 1.36 | 1 / 208 |
| CBOMkit | 73 | 39 | 20 | 14 | 2.14 | 0 / 191 |
| cdxgen | 1 | 0 | 1 | 0 | 1.06 | 1 / 26 |
| cryptobom-forge | 7 | 1 | 2 | 4 | 1.47 | 0 / 33 |

- **CBOMkit** — 58 of its false positives are materials. It emits 73 material components and 15 match (strict). 28 of the 73 have a size of exactly 208 bits, which is the length of the 26-byte `SESSION_INFO` constant (Java line 86): 14 ‘salt’ components stacked on line 86, 13 ‘password’ components stacked on the `PBEKeySpec` line 172 (the real passwords are 64, 104 and 224 bits) and one ‘secret-key’. Its algorithm errors are consistent size mistakes: AES reported as 128-bit where the source uses a 256-bit key (in the GCM scenario the only 128 is the tag length), and RSA reported as 2048-bit where the source requests 3072.
- **cdxgen** — one false positive: a C# `rsa` reported on line 225, which is the comment line above the RSA scenario (a ground-truth decoy), three lines above the first real RSA call. Widening the location gate to include the function header turns it into a TP (F1 7.5 % → 8.0 %, no rank change).
- **forge** — three strict false positives: two import-line detections (`AES-GCM` and `RSA` windows over `from … import` lines, which the ground truth files under libraries) and an `AES-128-CBC` component over the AES-256-CBC window; plus four duplicates (a second `HKDF`, the hash components `SHA256-OAEP` and `SHA256-PKCS1V15`, and a second RSA private key).
- **merged** — the CBOMkit errors above (de-duplicated where the merger collapsed them) plus the two RSA PKCS#1 v1.5 components carrying the mis-parsed key size 15 (§5).

### 4.9 Library (SBOM) detection

23 ground-truth libraries (5 runtimes, 18 packages). Matched by purl type + name.

| CBOM | Reported | Correct | False | Missed | Precision | Recall | F1 |
|---|--:|--:|--:|--:|--:|--:|--:|
| twocbom (merged) | 14 | 11 | 3 | 12 | 78.6% | 47.8% | 59.5% |
| CBOMkit | 0 | 0 | 0 | 23 | — | 0.0% | 0.0% |
| cdxgen | 14 | 11 | 3 | 12 | 78.6% | 47.8% | 59.5% |
| cryptobom-forge | 0 | 0 | 0 | 23 | — | 0.0% | 0.0% |

cdxgen (and therefore the merged BOM) finds pyca/cryptography, PyCryptodome, PyNaCl, argon2-cffi, bcrypt, BouncyCastle (Java ×2, C#) and the three Go modules; it misses all six npm packages (`@noble/*`, `bcryptjs`, `@peculiar/x509`), the five language runtimes/stdlibs and `bcutil`, and adds three spurious PyPI names (`argon2`, `python-bcrypt`, `nacl`). Versions are correct for every matched package. CBOMkit and forge emit no library components.

## 5. Merged vs standalone

Because `merged` is built from the other three, the useful question is what the merge adds or costs.

| Strict, whole corpus | Precision | Recall | F1 | Accuracy | TP | FP | FN |
|---|--:|--:|--:|--:|--:|--:|--:|
| Naive concatenation of the three standalone CBOMs | 52.8% | 22.8% | 31.8% | 18.9% | 93 | 83 | 315 |
| twocbom merge | 73.5% | 21.1% | 32.8% | 19.6% | 86 | 31 | 322 |

- **Gain: de-duplication.** 176 claims shrink to 117; false positives fall from 83 to 31, lifting precision by 20.7 points.
- **Cost: 7 assets lost.** Standalone tools found 93 distinct assets (strict); the merge keeps 86 and gains none. Lost: `Go:rsa-oaep-sha256`, `Go:rsassa-pkcs1v15-sha256`, `Java:m.chacha-key`, `Java:m.ctr-iv`, `Java:m.ctr-key`, `Java:m.ecb-key`, `Java:m.tdes-key`. Causes: (a) distinct materials with an unknown size collapse into one component (a single Java `secret-key` component now carries 14 different locations), so only one of them can match; (b) separate CBOMkit components `RSA-3072` and `RSA-OAEP` (Go) merge into a single `rsa-3072-oaep`; (c) Go `RSA-PKCS1-1.5` is renamed `rsa-15-pkcs1v15` with `parameterSetIdentifier: 15` (and the Java variant likewise, replacing CBOMkit's 2048) — the `1.5` of “PKCS#1 1.5” was read as a key size.
- **Evidence is thinned.** cdxgen's SHA-256 has eight TypeScript lines; the merged component keeps one. Evidence-level F1: concatenation 24.8%, merged 24.6% (precision up, recall slightly down).
- **Per language (strict F1):** the merge is best in Python (30.0 % vs forge 28.8 %), Java (46.4 % vs CBOMkit 39.2 %) and C# (33.7 % vs CBOMkit 21.3 % / cdxgen 19.6 % — evidence from both tools lands on one component), ties cdxgen in TypeScript (15.9 %), and loses only in Go (30.6 % vs CBOMkit 34.3 %) because of the RSA issues above.
- The merged file uses a language-directory prefix for Java/Go/TypeScript/C# but a bare `crypto_showcase.py` for Python (the root forge was run from); harmless here, but it makes paths inconsistent inside one BOM.

## 6. Robustness of the ranking

| Variant | Ordering (best → worst) |
|---|---|
| Whole corpus, strict F1 | merged > CBOMkit > cdxgen > forge |
| Whole corpus, lenient F1 | merged > CBOMkit > cdxgen ≈ forge |
| Whole corpus, strict F1, duplicates ignored | merged > CBOMkit > cdxgen ≈ forge |
| Whole corpus, strict precision | cdxgen > merged > forge > CBOMkit |
| Whole corpus, strict recall | merged > CBOMkit > cdxgen ≈ forge |
| Algorithms only, strict F1 | merged > CBOMkit > forge > cdxgen |
| Materials only, strict F1 | merged > CBOMkit > cdxgen > forge |
| Coverage-adjusted, strict F1 | CBOMkit > merged > forge > cdxgen |
| Coverage-adjusted, lenient F1 | CBOMkit > merged > forge > cdxgen |
| Evidence-level F1 (strict) | merged > CBOMkit > forge > cdxgen |
| Attribute accuracy | merged > CBOMkit > cdxgen > forge |
| Location gate ±0 lines, strict F1 | merged > CBOMkit > cdxgen > forge |
| Location gate incl. function header (3 before / 1 after), strict F1 | merged > CBOMkit > cdxgen > forge |
| Location gate ±3 lines, strict F1 | merged > CBOMkit > cdxgen > forge |

`merged` is first in 11 of 14 variants and second in the rest (behind cdxgen on precision alone, and behind CBOMkit in the two coverage-adjusted variants). cdxgen and forge are interchangeable: cdxgen is ahead in 7 variants, forge in 4, level in 3. Tightening or widening the location gate changes no ordering.

The coverage-adjusted flip is partly an artefact of comparing tools on different language sets: the merged tool's denominator includes Python and TypeScript, where it can only be as good as forge and cdxgen. On *identical* languages the merge is ahead of each standalone tool in strict F1, and in lenient F1 too, except in Python, where forge is ahead by 0.8 points:

| Languages | CBOM | Strict precision | Strict recall | Strict F1 | Lenient F1 |
|---|---|--:|--:|--:|--:|
| Java, Go, C# | CBOMkit | 46.7% | 26.1% | 33.5% | 42.9% |
| Java, Go, C# | twocbom (merged) | 69.6% | 26.1% | 38.0% | 43.3% |
| TypeScript, C# | cdxgen | 94.1% | 9.8% | 17.8% | 17.8% |
| TypeScript, C# | twocbom (merged) | 92.3% | 14.7% | 25.4% | 26.5% |
| Python | cryptobom-forge | 68.2% | 18.3% | 28.8% | 30.8% |
| Python | twocbom (merged) | 83.3% | 18.3% | 30.0% | 30.0% |

## 7. Caveats

- **Granularity of the ground truth.** Assets are fine-grained: RSA appears as four assets (key, OAEP, PSS, PKCS#1 v1.5), AES as four (GCM, CBC, ECB, CTR), and every key, IV, salt, derived key and shared secret is its own asset. A tool that emits one component for “RSA-3072 + OAEP” can match only one of them, and materials make up 60 % of the denominator. Rankings on *algorithms only* are the more tool-relevant view; they give the same top two and put forge ahead of cdxgen.
- **Where materials are located.** The ground truth records hard-coded material at its definition and generated material at the producing call. I accept a use-site line inside the same scenario function; a stricter reading would lower every tool's material recall further (strict-mode material recall is already at most 6.1 %).
- **My normaliser is part of the measurement.** Name→identity mapping and the few accepted aliases (§3) are judgment calls; changing them moves individual numbers by a few points but not the order. Strict mode treats a wrong parameter as both a false positive and a miss, which is harsh by design; lenient mode is the counterweight.
- **Coverage is observed, not declared.** “Languages with output” means the tool emitted at least one asset in that language. A tool that silently failed on a supported language is not excused in the whole-corpus ranking.
- **cdxgen vs forge is a near-tie** on the whole corpus (3.9 % vs 3.7 % recall, 16 vs 15 TPs). Read their order as a trade-off, not a verdict.
- **One small corpus.** 24 scenarios × 5 languages of test-vector code, no frameworks or indirection; absolute numbers will not transfer to real applications.
- Library detection (§4.9) and implicit assets are reported but are not part of the asset ranking.

## Appendix A. Which tool finds which algorithm

Rows are ground-truth algorithm, protocol and certificate assets; columns are languages. **K** = CBOMkit, **X** = cdxgen, **F** = forge, **M** = merged. Upper case = found with a consistent identity (strict); lower case = found at the right place with the right family but a wrong parameter (lenient only). `·` = found by no tool; `–` = the asset is not explicit in that language.

| Asset | Python | Java | Go | TypeScript | C# |
|---|:-:|:-:|:-:|:-:|:-:|
| `sha-256` | FM | KM | KM | XM | XM |
| `sha-512` | FM | KM | KM | XM | XM |
| `sha3-256` | · | KM | KM | · | · |
| `blake2b-512` | FM | KM | · | · | · |
| `md5` | FM | KM | KM | XM | XM |
| `sha-1` | FM | KM | KM | XM | XM |
| `shake128` | FM | KM | · | XM | · |
| `hmac-sha256` | · | KM | KM | · | · |
| `hkdf-sha256` | FM | · | · | · | KM |
| `pbkdf2-hmac-sha256` | · | KM | KM | · | KXM |
| `scrypt` | FM | · | · | · | · |
| `argon2id` | · | · | · | · | · |
| `bcrypt` | · | · | · | · | · |
| `aes-256-gcm` | · | km | km | · | km |
| `chacha20-poly1305` | · | KM | · | · | KM |
| `aes-256-cbc` | FM | KM | · | · | KXM |
| `aes-128-ecb` | FM | KM | KM | · | KM |
| `aes-256-ctr` | · | km | · | · | · |
| `3des-cbc` | · | KM | KM | · | KM |
| `rsa-3072` | FM | km | KM | XM | KM |
| `mgf1-sha256` | · | KM | – | – | – |
| `rsa-oaep-sha256` | · | km | K | · | · |
| `rsassa-pss-sha256` | FM | KM | · | · | · |
| `rsassa-pkcs1v15-sha256` | FM | km | Km | · | · |
| `ecdsa-p256-sha256` | FM | KM | KM | · | KM |
| `ecdh-p256` | · | KM | KM | · | KM |
| `x25519` | · | KM | · | XM | · |
| `ed25519` | · | KM | KM | · | · |
| `ml-kem-768` | · | KM | KM | · | · |
| `ml-dsa-65` | · | KM | · | · | · |
| `csprng` | · | · | KM | · | KM |
| `cert.self-signed` | · | · | · | · | · |
| `tls13-client` | · | KM | · | · | · |
| `bc-digest-drbg` | – | – | – | – | · |

Found by no tool in any language: `argon2id`, `bcrypt`, `cert.self-signed`, `bc-digest-drbg`.

## Sources

- Ground truth: [github.com/cbomfuse/baseline](https://github.com/cbomfuse/baseline) — per-language `ground-truth.json`, reference `cbom.cdx.json`, `_tooling/spec.py`; HEAD `6e29566`, compared with the CBOMkit-scanned commit `7cb9d9a` (only build artifacts differ).
- Evaluated files: `5ef8776e-cbomkit.cbom.json`, `13f0d331-cdxgen.cbom.json`, `262fc2a5-cryptobom-forge.cbom.json`, `698d7ecb-merged.cdx.json`.
