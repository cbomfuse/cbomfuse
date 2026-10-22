# Project 2 Kickoff Report: Measuring Cryptographic Readiness for the Post-Quantum Era (as of 22 October 2026)

The best target for your new tool is **multi-language, configuration-aware cryptography discovery that normalizes everything into one CycloneDX 1.7 CBOM, starting with C#/.NET and JavaScript/TypeScript**. None of the three starting tools covers all five of your benchmark languages at source level. Their outputs barely agree with each other, and none of them reliably handles config-selected algorithms or tells you whether crypto code is actually used. Treat this as a hypothesis to confirm with your benchmark, not as a conclusion. The rest of this report supplies the foundations, the tool-by-tool facts, a benchmark design and a phased plan to get you there.

## TL;DR

- **The three tools differ in kind, not just in quality.** CBOMkit (sonar-cryptography) does AST-based detection with deep API models for Java, Python (pyca/cryptography only) and Go. cdxgen is an SBOM generator with uneven CBOM add-ons: JS/TS source analysis, .NET via dosai, Go via a separate evinse step, plus Java keystores and certificates. CryptoBOM-Forge is a small, mostly dormant post-processor that turns CodeQL SARIF into an older, pre-standard IBM CBOM 1.4 format.\[1\] A peer-reviewed comparison (BF-CBOM, ICPC 2026) found "overlaps were rare, with almost no lexically exact matches" between their outputs.
- **The benchmark needs two oracles and two granularities.** One oracle says what is *present* (for inventory) and one says what is *reachable or used*. Score both at project level (algorithm present: yes/no) and at occurrence level (file plus line window). Normalize names into CycloneDX registry names plus OIDs before matching. The only published CBOM benchmark with ground truth, Cryben (2026), shows that counting-model mismatches can explain most apparent misses.
- **PQC test cases are now feasible natively in all five languages.** Java has ML-KEM/ML-DSA since JDK 24. Go has crypto/mlkem since 1.24 and crypto/mldsa since 1.27. Python's pyca/cryptography has ML-KEM/ML-DSA since 47/48, but not SLH-DSA. .NET 10 has MLKem, MLDsa and SlhDsa. Node.js has ML-DSA/ML-KEM/SLH-DSA since 24.6–24.8.\[2\]\[3\]\[4\]\[5\]\[6\]\[7\] For the prototype, build the C#/JS-TS and config-driven coverage gap first, plus a normalizer/diff layer you will need anyway for evaluation.

---

## Part A — Foundations and post-quantum migration review

### A.1 What quantum computers change

| Family | Examples | Quantum attack | Practical consequence |
|---|---|---|---|
| Public-key encryption / key exchange | RSA, DH, ECDH (X25519, P-256) | Shor's algorithm solves factoring and discrete logs in polynomial time | Broken outright. Recorded traffic can be decrypted later ("harvest now, decrypt later"), so key exchange is the most urgent part. |
| Signatures | RSA-PSS/PKCS#1, DSA, ECDSA, EdDSA | Shor's algorithm | Signatures can be forged once a cryptographically relevant quantum computer (CRQC) exists. Less urgent for past data, but long-lived roots of trust (firmware, CA roots) need long lead times. |
| Symmetric ciphers | AES-128/192/256, ChaCha20 | Grover gives roughly a square-root speed-up | Not broken. AES-256 keeps a large margin, and AES-128 is generally still considered acceptable. |
| Hashes / MACs | SHA-2, SHA-3, HMAC | Grover / collision-search speed-ups are limited | Generally fine at current sizes. MD5 and SHA-1 are already classically broken, which is a separate issue your tool should still flag. |

**What this means for your tool:** "quantum-vulnerable" is a property of *asymmetric* primitives. A good CBOM classifier must separate primitive types (`pke`, `kem`, `signature`, `key-agree` vs `ae`, `hash`, `mac`). Calling every non-PQC algorithm "vulnerable" is a common usability failure.

### A.2 NIST PQC standards (FIPS 203/204/205, finalized 13 August 2024)

| Standard | Purpose | Replaces | Parameter sets (NIST category) | Sizes (bytes) |
|---|---|---|---|---|
| **FIPS 203 ML-KEM** (Kyber) | Key encapsulation | RSA key transport, DH/ECDH key agreement | ML-KEM-512 (1), **ML-KEM-768 (3)**, ML-KEM-1024 (5) | ek/dk/ct: 800/1,632/768; **1,184/2,400/1,088**; 1,568/3,168/1,568 |
| **FIPS 204 ML-DSA** (Dilithium) | General-purpose signatures | RSA, ECDSA, EdDSA, DSA signatures | ML-DSA-44 (2), ML-DSA-65 (3), ML-DSA-87 (5) | pk/sk/sig: 1,312/2,560/2,420; 1,952/4,032/3,309; 2,592/4,896/4,627 |
| **FIPS 205 SLH-DSA** (SPHINCS+) | Conservative hash-based signatures | Same as ML-DSA, for high-assurance or long-lived uses | 12 sets: SHA2/SHAKE × 128/192/256 × s (small) / f (fast) | pk 32–64. Signatures from about 7.9 KB (128s) to about 49.9 KB (256f). |

Practical implications for test design: KEMs are not drop-in replacements for DH APIs. They use encapsulate/decapsulate, not agree. Key and signature sizes grow 5–100×, which breaks fixed-size buffers, database columns and certificate size limits. A good CBOM records the **parameter set** (`parameterSetIdentifier` such as "768" or "ML-DSA-65"), not just the family.

**Related NIST work — status (October 2026):**

- **FN-DSA / Falcon (FIPS 206):** NIST submitted the draft for approval on 28 August 2025. The secondary sources disagree on whether an Initial Public Draft has appeared. One tracker reports that as of 30 September 2026 there was "no public draft on NIST's site and no Federal Register notice," and NIST's project page lists FIPS 206 as "in development." Treat it as **not final**.\[8\]\[9\]\[10\]
- **HQC:** selected on 11 March 2025 as a backup code-based KEM. Draft standardization is underway, reportedly as FIPS 207. No public draft is verified.\[9\]\[11\]
- **NIST IR 8547 (Transition to PQC Standards):** the only version I could verify on NIST CSRC is NISTIR 8547 (Draft), "Transition to Post-Quantum Cryptography Standards," by Dustin Moody, Ray Perlner, Andrew Regenscheid, Angela Robinson and David Cooper (NIST), published 12 November 2024 (DOI 10.6028/NIST.IR.8547.ipd), with comments closed on 10 January 2025. It proposes that quantum-vulnerable public-key algorithms (RSA, ECDSA, EdDSA, DH, ECDH) at 112-bit security be **deprecated after 2030** and **all be disallowed after 2035**. One aggregator claims a "final" version (May 2025) and a second draft (November 2025). I could not confirm either on CSRC, so cite the IPD and flag the conflict.
- **SP 800-227 (KEM recommendations):** NIST SP 800-227, "Recommendations for Key-Encapsulation Mechanisms" (Noah Waller, NIST), final version published 18 September 2025 (DOI 10.6028/NIST.SP.800-227), gives NIST's guidance on using KEMs correctly.

### A.3 Migration guidance and timelines

| Body | Key dates | Inventory emphasis |
|---|---|---|
| NIST IR 8547 (draft) | Deprecate 2030 / disallow 2035. Hybrids are allowed in the transition if the PQC component is approved.\[12\] | Identify quantum-vulnerable algorithm use |
| UK NCSC "Timelines for migration to PQC" (20 March 2025) | **2028** discovery and initial plan; **2031** highest-priority migrations; **2035** complete\[13\]\[14\] | Phase 1 is explicit: by 2028, "Carry out a full discovery exercise (assessing your estate to understand which services and infrastructure that depend on cryptography need to be upgraded to PQC)" (NCSC guidance) |
| NSA CNSA 2.0 | All National Security Systems quantum-resistant by 2035.\[15\] Requires ML-KEM-1024 and ML-DSA-87 levels. | Vendor and product transition dates |
| Canada (Cyber Centre roadmap) | High-priority systems by end of 2031, remaining systems by end of 2035\[15\] | Start with an initial, incomplete inventory and iterate\[16\] |
| EU coordinated roadmap (NIS Cooperation Group, adopted June 2025) | Per the European Commission's Digital Strategy page: "All Member States should start transitioning to PQC by the end of 2026... high-risk use cases should be transitioned to PQC as soon as possible, no later than by the end of 2030"; all feasible systems by 2035 | National inventories and roadmaps |

**Common thread:** every roadmap puts *discovery and inventory* first. This is where CBOM tooling fits, and it justifies your project. **Crypto-agility** (algorithms chosen through configuration rather than hard-coded) is the target state. Ironically, it also makes discovery harder, because the algorithm lives in config.

**Hybrid key exchange** is the dominant real-world deployment. **X25519MLKEM768** in TLS 1.3 is the default in Go 1.24+ crypto/tls, OpenSSL 3.5+ and major browsers.\[17\]\[18\]\[19\] Your benchmark must treat a hybrid group as one protocol-level asset composed of a classical part and a PQC part.\[20\] A naive scanner will flag "X25519 = vulnerable."

### A.4 Native PQC support per target language (for writing test cases)

| Language | Built-in / mainstream PQC | Version and API names | Gaps / caveats |
|---|---|---|---|
| **Java** | JCA: `KeyPairGenerator`/`KEM.getInstance("ML-KEM")`, `Signature.getInstance("ML-DSA")` | JDK 24 (JEP 496 ML-KEM, JEP 497 ML-DSA). JDK 26: jarsigner with ML-DSA. JDK 27 (GA 15 September 2026): hybrid TLS 1.3 (JEP 527), per secondary sources.\[21\]\[22\] | No SLH-DSA in the JDK; use **Bouncy Castle** (ML-KEM/ML-DSA/SLH-DSA). Note that CodeQL's Java quantum model does not model `javax.crypto.KEM`.\[23\] |
| **Go** | `crypto/mlkem` (`GenerateKey768`, `NewEncapsulationKey1024`, …), `crypto/mldsa` | mlkem: Go 1.24. Go 1.26 adds `crypto.Encapsulator`/`Decapsulator` interfaces and SecP256r1MLKEM768 / SecP384r1MLKEM1024 TLS groups. mldsa + x509 + TLS ML-DSA: Go 1.27 (August 2026).\[6\]\[7\]\[24\] | `crypto/tls` uses X25519MLKEM768 **by default**, but an old `go` line in go.mod (e.g. `go 1.23`) or explicit `CurvePreferences` silently disables it.\[24\] This is a perfect config-driven test case. No SLH-DSA in stdlib. |
| **Python** | `pyca/cryptography`: `cryptography.hazmat.primitives.asymmetric.mlkem` / `.mldsa` | 47.0.0 (24 April 2026, AWS-LC/BoringSSL only); 48.0.0 (4 May 2026, OpenSSL 3.5+); 49.0.0 (12 June 2026, ML-DSA X.509); 50.0.0 (31 July 2026)\[3\] | ML-KEM-768/1024 only, no ML-KEM-512. **No SLH-DSA.**\[25\] For SLH-DSA use `liboqs-python` (Open Quantum Safe). `hashlib`/`ssl` are stdlib and outside CBOMkit's model. |
| **C#/.NET** | `System.Security.Cryptography.MLKem`, `MLDsa`, `SlhDsa`, `CompositeMLDsa` | .NET 10 | Needs OpenSSL 3.5+ or Windows CNG with PQC (`IsSupported`). `MLDsa`, `SlhDsa` and `CompositeMLDsa` are `[Experimental]` (SYSLIB5006).\[26\]\[27\] **Bouncy Castle C#** is an alternative. |
| **JS/TS (Node)** | `node:crypto` `generateKeyPair('ml-kem-768')`, `encapsulate`/`decapsulate`, `sign`/`verify` with `ml-dsa-65`; WebCrypto `ML-KEM-768`, `ML-DSA-65` | ML-DSA v24.6.0, ML-KEM v24.7.0 (incl. WebCrypto), SLH-DSA v24.8.0, JWK formats v26.1.0. Newer docs list hybrid `MLKEM768-X25519` in WebCrypto.\[4\]\[28\]\[29\] | Requires Node built against OpenSSL 3.5. For browsers and older runtimes use pure-TS `@noble/post-quantum`, which is a dependency-only case scanners may miss.\[30\]\[31\] |
| **C / cross-cutting** | OpenSSL 3.5+ (ML-KEM, ML-DSA, SLH-DSA; hybrid TLS groups), OQS liboqs + oqs-provider\[32\] | — | liboqs is for prototyping and is not FIPS-validated.\[32\] Good for algorithms not yet in mainstream libraries. |

### A.5 CycloneDX CBOM: what your outputs and ground truth should look like

- CBOM arrived in **CycloneDX 1.6 (April 2024; ECMA-424)**.\[33\] The current spec is **1.7**, which cdxgen and the 2026 Cryben benchmark already emit.\[34\]\[35\] sonar-cryptography and cbomkit-theia still emit **1.6**.\[36\]
- Components have `type: "cryptographic-asset"` and `cryptoProperties.assetType` ∈ {`algorithm`, `certificate`, `protocol`, `related-crypto-material`}.\[37\]
- Key algorithm fields: `primitive` (e.g. `ae`, `hash`, `signature`, `kem`, `pke`, `key-agree`), `parameterSetIdentifier`, `mode`, `padding`, `curve`/`ellipticCurve`, `cryptoFunctions`, `classicalSecurityLevel` and **`nistQuantumSecurityLevel`**. The last uses NIST categories 1–5, where "0 indicates that none of the categories are met"; RSA/ECC therefore get 0.\[20\]\[38\]\[39\] Also `oid`, plus `algorithmFamily` in 1.7.
- Location comes from `evidence.occurrences` (file and line). Protocols use `protocolProperties` (type, version, cipherSuites).
- The **CycloneDX Cryptography Registry** defines naming patterns (e.g. `RSA-PKCS1-1.5-SHA-256-2048` with `algorithmFamily: RSASSA-PKCS1`).\[40\] Use it as your normalization target.

### A.6 Discovery-tool landscape (beyond the starting three)

- **CBOMkit family (PQCA, Linux Foundation):** sonar-cryptography ("Hyperion"), cbomkit-lib, cbomkit-action, the cbomkit web app (Coeus/Themis/Mnemosyne) and **cbomkit-theia**. Theia is a Go CLI that scans directories and container images for certificates, keys, secrets, `java.security` and `openssl.cnf`.\[36\]\[41\] IBM donated the toolset on 23 June 2025.\[42\]
- **CodeQL `experimental/quantum`:** a shared crypto model (`codeql/quantum`) with confirmed C/C++ (OpenSSL) and Java (JCA) coverage, plus inventory slices and "QuantumVulnerable" demo queries.\[23\] A September 2026 issue notes that it maps ML-KEM to "Other" and does not model `javax.crypto.KEM`.\[23\]\[43\] Separately, GitHub's `advanced-security/cbom-action` runs CodeQL to build CBOMs.\[44\]\[45\]
- **Crypsy/Crypistry/Cryben (Näther & Hirsch, ICICS 2026):** a rule-based scanner covering Go, Ruby, nginx config and crypto files.\[35\]\[46\]
- **Other open source:** Wind River Crypto Detector (keyword/pattern), Semgrep crypto rules, many small 2026 "PQC readiness" scanners (quipuu in Rust, quantakrypto, QuReddy for TLS/SSH endpoints).\[35\]\[47\]\[48\]\[49\] Treat these as unvetted.
- **Commercial:** IBM Quantum Safe Explorer, Keyfactor/InfoSec Global AgileSec, SandboxAQ AQtive Guard and others. These are closed, so they can't be compared reproducibly; mention them for context only.

### A.7 Academic literature to anchor the review

- **Crypto API misuse detectors:** CryptoLint (Android), CogniCrypt/CryptoAnalysis with **CrySL** (rules over call sequences and constraints), **CryptoGuard** (backward program slicing over large Java projects), LICMA (Python/C).\[35\] These detect *misuse*, not *inventory*, but their analysis techniques (slicing, typestate, constant propagation) are exactly what config and indirection cases need.
- **Benchmarks:**
  - **CryptoAPI-Bench:** 171 Java JCA cases covering 16 misuse types in the original release (Afrose, Rahaman, Yao, IEEE SecDev 2019); the extended version has 181 cases across 18 misuse categories (arXiv 2112.04037), which were run against SpotBugs, CryptoGuard, CrySL and Coverity. Cases carry true and false misuse labels and intra-/inter-procedural, field- and path-sensitive variants.
  - **CamBench:** real-world, synthetic and analysis-capability tiers.\[50\]
  - **MASC:** mutation-based evaluation. 12 base operators (later 19), 20,303 mutants, 9 detectors, **19 previously unknown flaws**.\[51\]\[52\]\[53\]
  - **OWASP Benchmark:** v1.2, 2,740 Java programs with true- and false-positive cases.\[54\]
  - Lesson from LLM-era studies: test-case **names leaked the expected answer** (class names encoding the misuse).\[55\] Give your benchmark files neutral names.
- **CBOM-specific (2026):**
  - **BF-CBOM** (Bögli, Spieler, Kehrer, ICPC 2026) orchestrates CBOMkit v2.1.10, cdxgen, CryptobomForge v1.1.0 and a DeepSeek LLM worker in Docker. Findings: CryptobomForge has "the highest setup overhead"; runtimes range from seconds to minutes; tools fail on some repos; and outputs are "strikingly inconsistent." The authors say they are curating ground-truth datasets, so there is a direct opportunity to contribute or compare.\[33\]\[56\]
  - **Cryben/Crypsy** (arXiv 2608.04857) is the closest methodological template: a CycloneDX 1.7 ground truth built independently of the tool, scored at occurrence level.\[35\]\[46\]
  - **Cryptoscope** (arXiv 2503.19531) covers source plus domain knowledge but leaves certificates and keystores to future work.\[35\]\[57\]
  - Also: "Towards CBOM Compliance" (Springer 2026), architecture-derived CBOMs (arXiv 2603.22442), and binary-level PQC detection (arXiv 2608.25122).\[58\]\[59\]\[60\]
- **LLM-assisted discovery:** BF-CBOM's zero-shot DeepSeek worker, and LLM misuse-detection studies showing high recall but poor precision (e.g. GPT-4o precision 0.46 on CamBench before prompt tuning).\[61\] Treat LLMs as triage or classification helpers, not as oracles.

**CryptanalysisBench (Fluri, Shafran, Carlini, Jagielski, Nasr, Dunkelman, Ronen, Tramèr; arXiv 2607.18538, v2 29 July 2026):**
- 191 tasks across six primitive families (block ciphers, hash functions, etc.), mostly from NIST competitions, in three tiers.\[62\]
- Five frontier models break 65–86% of Tier-1 schemes (known practical breaks), 6–12 Tier-2 schemes at full strength, and 24–61 scaled-down variants.\[62\]
- The models found new results: a key-recovery attack on the SpoC AEAD and an error in KINDI's CCA proof.\[62\]
- **Relevance to you:** it is *motivation, not method*. If AI speeds up cryptanalysis, the set of "weak" algorithms may change faster than standards cycles. That raises the value of accurate, machine-readable inventories that can be re-queried when a primitive falls. It is also a caution: the paper evaluates attacks on primitives, not discovery of crypto in code, so don't cite it as evidence that LLMs can build CBOMs.

### A.8 Learning-path links

- **arXiv 2607.18538:** verified live (v2).
- **NIST "What Is PQC?" and nist.gov/pqcrypto:** official NIST pages. Note that NIST's PQC project page currently lists FIPS 206 as in development.
- **NCSC "Preparing for quantum-safe cryptography":** NCSC's foundational white paper. Pair it with the March 2025 "Timelines for migration" guidance, which is more actionable.
- **openquantumsafe.org:** liboqs, oqs-provider and language wrappers (liboqs-python, liboqs-java, liboqs-go, liboqs-dotnet).
- **cyclonedx.org/capabilities/cbom:** the CBOM overview. Also use the 1.7 spec docs and the Cryptography Registry.
- I fetched only the arXiv page directly. Re-click the other four before submitting your review.

---

## Part B — Deep dive: CBOMkit, cdxgen, CryptoBOM-Forge

### B.1 CBOMkit (sonar-cryptography / cbomkit-lib / cbomkit / cbomkit-theia)

- **Maintainer, repo, license, activity:** Post-Quantum Cryptography Alliance (Linux Foundation), originally IBM Research.\[36\] Repos at github.com/cbomkit (formerly PQCA/…), licensed Apache-2.0.\[63\] The packaged plugin also bundles sonar-java, sonar-python and sonar-go under the **Sonar Source-Available License v1.0**, which matters if you plan to fork.\[64\] Activity is high:
  - sonar-cryptography 1.5.1 (4 February 2026) added Go and AES-GCM rules for Go.\[65\]
  - cbomkit-theia was updated in September 2026.\[63\]
  - An open issue (#521, August 2026) proposes JS/TS support via an ESLint bridge.\[66\]
- **How it analyzes code:** AST-based, using Sonar's typed ASTs with resolved symbols.\[67\] `DetectionRuleBuilder` rules target specific API calls (e.g. `javax.crypto.Cipher#getInstance`), capture parameter values and follow nested detections. A translator maps results into an `INode` model (Algorithm, Key, Protocol, Mode, Padding, KeyLength). An **enricher** adds OIDs and default parameters, and every node records an origin of *detected*, *inferred* or *enriched*.\[36\] The Python module tracks variable assignments.\[68\] Java symbol resolution improves if the project is **built** and class or JAR directories are supplied.\[36\]
- **Coverage:**
  - Java: JCA and the Bouncy Castle lightweight API.\[36\]\[67\]
  - Python: **pyca/cryptography only**.\[36\]\[64\] The docs don't list hashlib, ssl, PyCryptodome or PyJWT, which is likely a large blind spot.
  - Go: `crypto/*` "full coverage (except crypto/x509)" and partial `golang.org/x/crypto` (hkdf, pbkdf2, sha3).\[69\]
  - **No C#, no JS/TS.**\[66\]
- **Output:** a CycloneDX **1.6** `cbom.json` with file and line evidence and dependency edges (e.g. key → algorithm).\[36\]
- **Quantum flagging:** the cbomkit app's `BasicQuantumSafeComplianceService` checks assets against a whitelist of quantum-safe OIDs and names, marking each compliant, non-compliant or unknown. Optional OPA or an external regulator service can be plugged in.\[36\]
- **PQC recognition:** depends on rule coverage of the new APIs (JDK 24 ML-KEM/ML-DSA, BC PQC classes, pyca mlkem/mldsa, Go mlkem/mldsa). This is **unverified**, so make it an explicit benchmark question.
- **Setup:**
  - Full plugin: SonarQube server + plugin + SonarScanner.
  - cbomkit-lib/cbomkit-action: no server needed, JDK 21 container.\[36\]
  - cbomkit web app: Docker Compose with PostgreSQL.
  - Theia: Go or Docker. Directory scans must be scoped, because unscoped scans pick up vendored CA bundles. One user saw 8,542 assets that were almost all certifi/pip CA bundles.\[70\]
  - Known Theia bug: `image` mode missed a PEM key that `dir` mode found on the same content (#195).\[71\]
- **Indirection, config and dead code:** wrapper and helper detection works through symbol resolution where Sonar resolves types. Algorithm strings built from constants are partly handled through assignment tracking. **No reachability analysis**, so dead code is reported. Configuration files are covered only by Theia (java.security, openssl.cnf), not application properties.

### B.2 cdxgen

- **Maintainer, repo, license, activity:** OWASP CycloneDX community (lead: Prabhu Subramanian). github.com/cdxgen/cdxgen, Apache-2.0, about 1.1k stars and about 4,967 commits.\[72\] **Latest release v13.4.0 (14 October 2026).**\[72\]\[73\]\[74\]\[75\] It moves fast: v13.0.0 (13 August 2026), 13.1.0, 13.2.0 and 13.3.0 all shipped since August.\[75\]\[76\] The npm package was renamed to `@cdxgen/cdxgen` and needs Node ≥ 24.\[75\] The master README already mentions v13.5.0, but that is not yet a published release.\[72\]
- **How it analyzes code:** manifest and lockfile analysis for dependencies, plus several source engines: atom (Java/JS/TS/Python/C), dosai (.NET), golem via `evinse -l go` (Go), kosi (Kotlin), rusi (Rust), and `tracebom` for runtime tracing (`--trace-crypto`).\[72\]\[73\]
- **Inputs:** source, manifests, container images, binaries, keystores and certificates.
- **`cbom` alias:** equivalent to `cdxgen --include-crypto --evidence --deep --spec-version 1.7`.\[77\]\[78\]
- **Per-language CBOM coverage:**
  - **Java:** keystores, certificates and crypto libraries as components.\[72\] Source-level algorithm extraction is *not clearly documented*.
  - **Python:** documented only as an alias example. No documented source-level algorithm detection.
  - **JS/TS:** "lightweight constant propagation through common `node:crypto`, WebCrypto, and JWT call sites."\[72\] Output includes `cdx:crypto:sourceLocation` (file:line:col) and `cdx:crypto:sourceType`.\[72\]\[73\]
  - **C#/.NET:** dosai crypto analysis. Algorithms are emitted "only when they can be mapped to a known OID."\[73\]
  - **Go:** a *separate* `evinse` step with `--golem-dataflow crypto`, producing crypto-flow properties and crypto components.\[73\]
- **Output:** CycloneDX 1.7 by default. `algorithmFamily` comes from a closed enum. Non-conforming values are moved into `cdx:crypto:*` properties ("query the properties, not the structured field, when you need every usage").\[73\]
- **PQC:** v13.2.0 verified ML-DSA, ML-KEM and all 12 SLH-DSA OIDs and resolves parameter sets from dosai call sites.\[75\] I found **no evidence that it sets `nistQuantumSecurityLevel`**. There is no built-in quantum-vulnerability verdict comparable to CBOMkit's compliance service, so you would have to derive one.
- **Setup issues:**
  - atom 3 embeds a native binary on main platforms. Elsewhere it needs Java 23+, and it "freezes with Java 8 or 11."\[72\]
  - Large repos need a big heap.
  - SBOM fidelity depends on **the project building**. `--introspect` reports when only direct dependencies were captured.\[73\]
  - Known silent false-negative: dosai crypto output above about 512 MB is dropped by cdxgen "and logs nothing" (dosai #75).\[79\]
- **Indirection, config and dead code:** JS/TS constant propagation handles simple indirection. Reachability exists as a separate "reachables" slicing feature for dependencies, not for crypto findings. Config-selected algorithms are not documented as covered.
- **Note:** BF-CBOM (written against cdxgen 11.7) says cdxgen's CBOM is limited to Python and Java.\[33\] That is **already out of date**, which shows why you must pin versions and date every claim.

### B.3 CryptoBOM-Forge (confirmed project identity)

- **What it is:** `Santandersecurityresearch/cryptobom-forge`, written by Emile El-qawas, Mark Carney and Daniel Cuthbert (Santander) for a **Black Hat EU 2023** talk. Licensed **GPL-3.0**, Python ≥ 3.10.\[1\]\[42\]\[80\]
- **Size and activity:** about 37 stars, 9 forks, 52 commits, 2 open issues and 7 open PRs.\[1\] Latest tagged release cited in the literature is **v1.1.0 (25 January 2024)**.\[33\] The organisation page shows a repo update in 2026, but there are no new releases.\[45\]\[81\] **Maturity: low to moderate, maintenance mode.**
- **What it is not:** it is **not a scanner**. It parses **CodeQL SARIF / multi-repo variant analysis (MRVA) output**\[33\]\[82\] (`cryptobom generate <path>`, `--exclude` regex for tests).\[1\] Detection quality and language coverage therefore come entirely from the CodeQL queries you run: GitHub's `cbom-action` queries or Santander's CodeQL fork.\[44\]\[81\] CodeQL itself supports all five of your languages,\[83\] but the crypto query coverage per language is **unverified**.\[84\]
- **Output:** **IBM's pre-standard CBOM schema** (`"specVersion": "1.4-cbom-1.0"`, `type: "crypto-asset"`, `detectionContext` with file, lines and code snippet).\[1\] This is *not* CycloneDX 1.6/1.7, so you will need a converter in your harness.
- **Bundled "Cryptocheck":** YAML rules (regex, string and numeric comparisons on algo, keylen, mode, padding) producing SARIF findings such as "AES in ECB mode."\[1\]\[42\] There is no built-in PQC classification, though you could write rules for it.
- **Setup:** CodeQL CLI and query packs, building compiled-language databases (Java, C#, Go need a build), then Forge. BF-CBOM calls this the highest setup overhead of the three.\[33\]
- **Inference:** CodeQL databases do give you inter-procedural dataflow, so in principle this pipeline could handle indirection best. In practice the original queries were simple "informative" ones,\[85\] and nothing models config files or reachability.\[85\]

### B.4 Comparison table

| | CBOMkit (sonar-cryptography + Theia) | cdxgen (cbom) | CryptoBOM-Forge | CodeQL experimental/quantum | Crypsy (research) |
|---|---|---|---|---|---|
| Approach | AST + symbol resolution, rule DSL, enricher; Theia = file/image parsing | Manifest + atom/dosai/golem source engines + keystore/cert parsing | Post-processes CodeQL SARIF | QL dataflow over a shared crypto model | Rule repo: AST (Go/Ruby), regex config, manifest, file parsing |
| Source languages | Java, Python, Go | JS/TS, .NET, Go (evinse), Java (limited), Python (limited) | Whatever the CodeQL queries cover | C/C++, Java confirmed | Go, Ruby |
| Config / certs / keystores | Theia: certs, keys, secrets, java.security, openssl.cnf | Java keystores, certs | No | No | Yes (nginx, TLS/SSH config, crypto files) |
| Output format | CycloneDX 1.6 | CycloneDX 1.7 | IBM CBOM 1.4-cbom-1.0 | SARIF / query results | CycloneDX 1.7 |
| Location info | file + line | file:line:col (JS); evidence | file + line range + snippet | yes | yes |
| Quantum verdict | Whitelist compliance (cbomkit app) | No explicit verdict found | Only via custom YAML rules | Demo queries | PQC migration candidates |
| PQC recognition | Unverified per API | ML-KEM/ML-DSA/SLH-DSA OIDs (v13.2+) | Depends on queries | ML-KEM mapped to "Other" | Unknown |
| Reachability | No | Dependency reachables only | No | Possible via QL | No |
| Setup effort | Medium–high (Sonar or JDK, build for Java) | Medium (Node 24, atom/Java, build) | High (CodeQL + build) | High | Unknown (research artifact) |
| License / activity | Apache-2.0 (+ Sonar SAL parts) / active | Apache-2.0 / very active | GPL-3.0 / dormant | MIT / active | Research code |

### B.5 Per-language coverage matrix (source-level crypto detection)

| Language | CBOMkit | cdxgen | CryptoBOM-Forge |
|---|---|---|---|
| **Python** | ✅ pyca/cryptography only (no hashlib/ssl/PyJWT documented) | ⚠️ alias exists, no documented source algorithm extraction | ⚠️ via CodeQL queries (README example is `hashlib`) |
| **Java** | ✅ JCA + BC lightweight (best-modeled) | ⚠️ keystores/certs/libraries; source algorithms unclear | ⚠️ via CodeQL |
| **Go** | ✅ stdlib (no x509), partial x/crypto | ⚠️ separate `evinse -l go` golem step | ⚠️ via CodeQL |
| **C#** | ❌ | ✅ via dosai (OID-mappable only; >512 MB output silently dropped) | ⚠️ via CodeQL |
| **JS/TS** | ❌ (proposed, issue #521) | ✅ node:crypto, WebCrypto, JWT with constant propagation | ⚠️ via CodeQL |

✅ = documented; ⚠️ = partial or undocumented, so measure it; ❌ = none.

### B.6 Usefulness for migration planning (assessment)

- **CBOMkit:** most migration-ready for Java, Python and Go. It records parameters, has an explicit quantum-safe compliance check, and Theia adds deployment artifacts. Its weaknesses are language breadth and dead code.
- **cdxgen:** best breadth and the newest schema. You get dependency context for free, so it's good for a "which libraries could do crypto" view. But crypto depth varies by language and there is no quantum verdict, so you need post-processing.
- **CryptoBOM-Forge:** a historical baseline and a proof that CodeQL can be a CBOM backend. Its outdated schema and dependence on queries make it the weakest of the three as an *out-of-the-box* planning tool. Include it, but report setup cost honestly.

---

## Part C — Benchmark design

### C.1 Taxonomy of controlled cases (implement for each of the 5 languages)

Write each case as a tiny, buildable program with a neutral file name (e.g. `case_017/main.go`) and a sidecar `expected.cdx.json`.

| ID | Category | Example (Java / Go / Python / C# / JS) | Tests |
|---|---|---|---|
| D1 | Direct API, literal | `Cipher.getInstance("AES/GCM/NoPadding")`, `rsa.GenerateKey(r, 2048)`, `rsa.generate_private_key(65537, 2048)`, `RSA.Create(2048)`, `crypto.createHash('md5')` | Baseline recall |
| D2 | Direct API, default parameters | `KeyPairGenerator.getInstance("ML-KEM")` (defaults to 768), `MLKem.GenerateKey(MLKemAlgorithm.MLKem768)` | Enrichment of implicit parameters |
| H1 | Helper / wrapper, same file | `encrypt(data)` → `Cipher.getInstance(ALG)` | Inter-procedural |
| H2 | Wrapper across files / modules / packages | `crypto/util.go` exports `NewSigner()` | Cross-file resolution |
| H3 | Interface / factory / DI | strategy pattern choosing `Signer` impl | Points-to |
| S1 | Constant in another class / module | `static final String ALG = "RSA"` | Constant propagation |
| S2 | String concatenation / formatting | `"AES/" + mode + "/PKCS5Padding"`, `` `sha${bits}` `` | String analysis |
| C1 | Config file selects algorithm | `application.yml: jwt.alg: RS256`, `appsettings.json`, `.env`, YAML | Config parsing |
| C2 | Environment variable | `os.Getenv("HASH_ALG")` with default | Unknown-value handling |
| C3 | Platform security config | `java.security` disabled algorithms, `openssl.cnf`, Go `GODEBUG=tlsmlkem=0`, go.mod `go 1.23` line | Platform semantics |
| T1 | TLS config: cipher suites / groups | `tls.Config{CurvePreferences: []tls.CurveID{tls.X25519MLKEM768}}`, `SSLParameters.setNamedGroups`, Node `ecdhCurve`, `SslProtocols` | Protocol assets |
| T2 | Implicit TLS defaults | plain `https.get()` / `http.Client{}` | Should tools report "TLS with default groups"? Labeled *optional* |
| K1 | Certificates (PEM/DER), RSA vs ECDSA vs ML-DSA | `certs/server.pem` | Artifact parsing |
| K2 | Keystores | JKS/PKCS#12, .NET PFX | Keystore parsing |
| K3 | Private keys / SSH keys / JWKs in repo | `id_ed25519`, `jwks.json` | Material discovery |
| U1 | Dead function (never called) | `unusedEncrypt()` | Reachability (two oracles) |
| U2 | Unreachable branch (`if false`, feature flag off) | | Path sensitivity |
| U3 | Test-only crypto | `src/test/...` | Scoping |
| U4 | Commented-out / string-in-docs | `// Cipher.getInstance("DES")` | Pattern-matcher FPs |
| P1 | Direct dependency does crypto | uses `jjwt`, `PyJWT`, `jose`, `System.IdentityModel.Tokens.Jwt` with RS256 | Library-level inventory |
| P2 | Transitive dependency only | app → lib A → BouncyCastle | SBOM-to-crypto mapping |
| P3 | Vendored / pure-language crypto | `@noble/post-quantum`, node-forge, vendored Go code | Non-standard APIs |
| R1 | Reflection / dynamic loading | `Class.forName(...)`, Python `getattr(hashes, name)`, JS `crypto[fn]` | Dynamic dispatch |
| Q1 | PQC KEM | ML-KEM-512/768/1024 in each native API | PQC recognition + parameter |
| Q2 | PQC signatures | ML-DSA-44/65/87; SLH-DSA (Java BC, .NET, Node, liboqs-python) | PQC recognition |
| Q3 | Hybrid | X25519MLKEM768 TLS; .NET `CompositeMLDsa`; WebCrypto `MLKEM768-X25519` | Hybrid modeling, not a "vulnerable X25519" FP |
| Q4 | Legacy weak | MD5, SHA-1, DES/3DES, RC4, RSA-1024, DSA, ECB | Classical-weakness classification |
| N1 | Negative controls | Non-crypto `hash()`/`HashMap`, variables named `rsa`, `"AES"` in UI string | Precision |

Mirror CryptoAPI-Bench's sensitivity dimensions (intra-/inter-procedural, field-, path- and context-sensitive). Then use **MASC-style mutation operators** (rename, wrap, move to field, split string, route through config) to multiply each base case automatically. This gives you statistical power without hand-labelling thousands of files.

### C.2 Ground truth format and matching

- **Format:** one CycloneDX 1.7 CBOM per case, using registry names and OIDs, `parameterSetIdentifier`, `primitive`, `nistQuantumSecurityLevel` and `evidence.occurrences` (file and line). Add custom properties: `bench:reachable` (true/false), `bench:category` (D1…N1), `bench:required` (must find / optional / must-not-report).
- **Normalization pipeline:**
  - Map each tool's names to a canonical tuple: (assetType, family, parameter set, mode, padding, primitive).
  - Use an alias table: `RSA/ECB/PKCS1Padding` → family RSA, padding PKCS1v1.5, mode none; `SHA256withECDSA` → ECDSA + SHA-256; `ml-kem-768` / `MLKem768` / `ML_KEM_768` → ML-KEM-768. OIDs win when present.
  - Convert CryptoBOM-Forge's 1.4 schema and cdxgen's `cdx:crypto:*` properties into the same model.
- **Granularity:** score at three levels and report all of them.
  1. **Project level:** set of canonical algorithms per case or project.
  2. **Component level:** canonical tuple, ignoring location.
  3. **Occurrence level:** tuple plus file plus line within ±k lines (or enclosing function).
  Cryben shows why this matters: 43 of its 67 false negatives were "by-design counting differences,"\[46\] and at component level recall jumped to 0.96.\[35\]
- **Counting rules:**
  - TP = a predicted item matches a `required` truth item.
  - FN = an unmatched `required` item.
  - FP = an unmatched prediction that is not `optional`, or a match to `must-not-report`.
  - Use partial credit (family right, parameter set wrong) as a separate "attribute accuracy" metric, not as TP or FP.
  - Score dead code twice: under an **inventory oracle** (dead crypto *should* be listed) and a **usage oracle** (it should not). This avoids an unfair rule about which one is "correct."

### C.3 Metrics

- Precision, recall and F1 per tool × language × category, plus micro and macro averages.
- **Attribute accuracy:** parameter set, mode, padding and OID correctness.
- **Quantum-classification accuracy:** correct `nistQuantumSecurityLevel` or verdict, especially hybrid and symmetric cases.
- **Location accuracy.**
- Wall time, peak RSS and disk (via `/usr/bin/time -v`), and failure rate.
- **Installation effort:** steps, minutes, prerequisites, need to build, as a scripted checklist.
- **Understandability:** a short rubric (schema validity, naming consistency, actionable locations). Optionally ask 3–5 classmates to answer migration questions from each CBOM.

### C.4 Real-world projects (candidates — verify tags and sizes before pinning)

| Lang | Project | Why / crypto usage | License (verify) |
|---|---|---|---|
| Python | **paramiko** | SSH: RSA/ECDSA/Ed25519 keys, KEX, ciphers via pyca | LGPL-2.1 |
| Python | **PyJWT** or **python-jose** | JWT/JOSE: HS/RS/ES algorithms chosen by string, a config-driven case | MIT |
| Python | **certbot** (or acme lib) | PKI/ACME, certificates, pyca/cryptography | Apache-2.0 |
| Python | home-assistant/core (optional, large) | Mixed hashlib/pyca. Used in CryptoBOM-Forge's own example, so good for cross-checking. | Apache-2.0 |
| Java | **jjwt** | JWT/JWS/JWE across many JCA algorithms | Apache-2.0 |
| Java | **Apache MINA SSHD** | SSH KEX/ciphers/host keys; includes PQC hybrid KEX in recent versions (verify) | Apache-2.0 |
| Java | **Keycloak** (large) | JOSE, TLS, keystores, config-driven algorithms | Apache-2.0 |
| Java | **EJBCA CE** | PKI with ML-DSA/ML-KEM/SLH-DSA via BC, a PQC-adopted project | LGPL |
| Go | **smallstep/certificates** | CA/PKI, x509 (CBOMkit blind spot), TLS | Apache-2.0 |
| Go | **FiloSottile/age** | X25519, scrypt, ChaCha20-Poly1305; recent versions add hybrid ML-KEM recipients (verify) | BSD-3 |
| Go | **Caddy** | TLS config incl. curves/groups; inherits Go hybrid default | Apache-2.0 |
| C# | **OpenIddict** | OAuth/OIDC tokens, signing/encryption keys, certificates | Apache-2.0 |
| C# | **SSH.NET** | SSH algorithms in managed code | MIT |
| C# | **MimeKit** | S/MIME, PGP via Bouncy Castle C# (dependency case) | MIT |
| JS/TS | **panva/jose** | JOSE on WebCrypto/node:crypto, many algorithms | MIT |
| JS/TS | **node-forge** | Pure-JS crypto implementation (non-standard API case), PKI | BSD/GPL dual |
| JS/TS | **ssh2** | SSH via node:crypto | MIT |
| JS/TS | **openpgp.js** | OpenPGP; check whether your pinned version has PQC (ML-KEM+X25519) support | LGPL-3.0 |

**Pinning:** record `repo@tag` *and* the full commit SHA in `projects.yaml`. Vendor the source tarball (or use `git clone --filter=blob:none` plus checkout). Record build commands and toolchain versions (JDK, Go, .NET SDK, Node, Python) per project.

**Partial ground truth for real projects:**
1. Union all tool findings and sample N per project, stratified by tool and category.
2. Manually label each sampled finding (TP/FP plus reachable or not).
3. Independently grep-audit known crypto entry points (import lists of crypto packages, JCA `getInstance`, `System.Security.Cryptography`, `node:crypto`) to find misses that *no* tool reported.
4. Have a second labeller do a 20% overlap and report Cohen's κ.
5. Report recall as **relative recall** against the pooled-plus-audit set. Cryben's real-world section is explicit that this is a "manually compiled reference list rather than an exhaustive one."\[35\]

### C.5 Reproducibility

- One Dockerfile per tool, pinned by **image digest**: cbomkit-lib/cbomkit-action, SonarQube+plugin (optional), cdxgen v13.4.0 (or the official image), CodeQL CLI version + query-pack versions + CryptoBOM-Forge v1.1.0, cbomkit-theia.
- A `run_all.sh` / Makefile that builds cases → runs tools → normalizes → scores → produces CSV and plots.
- A GitHub Actions workflow that reruns on a small subset.
- A results bundle with raw tool outputs, normalized outputs and scores. Archive it on Zenodo with a DOI, as BF-CBOM did.
- Consider plugging into **BF-CBOM** (SEG-UNIBE, Docker-based) instead of building orchestration from scratch.\[56\] Its worker interface already wraps all three tools.\[33\]

---

## Part D — Toward a new tool

### D.1 Candidate weaknesses, approaches, measurement and feasibility (ranked)

| Rank | Weakness (expected, to confirm) | Technical approach | Measured by | Feasibility (MSc) |
|---|---|---|---|---|
| **1** | **No or partial source coverage for C# and JS/TS, and inconsistent depth across languages** | Single engine on **tree-sitter** (grammars for all 5) or **Semgrep** rules plus a per-language API model (YAML: API → primitive/family/param-arg index). Intra-procedural constant propagation plus simple inter-file constant resolution. Emit CycloneDX 1.7 with registry names, OIDs and `nistQuantumSecurityLevel`. | Recall/F1 on D/H/S/Q categories for C# and JS/TS vs cdxgen (dosai/atom) | High: 6–10 weeks for a solid core |
| **2** | **Config-selected algorithms and TLS/protocol settings missed** | Parsers for application.yml/properties, appsettings.json, .env, java.security, openssl.cnf, go.mod `go` line + GODEBUG, Node TLS options. Link config keys to code reads (`@Value`, `IConfiguration["..."]`, `os.Getenv`) to resolve algorithm values. | C1–C3, T1 recall; hybrid-group correctness | High; novel; small scope |
| **3** | **Naming and normalization chaos / no cross-tool view** | A normalizer that maps any tool's output (incl. IBM 1.4 schema, cdxgen properties) to registry-canonical tuples, plus a CBOM **diff/merge** with quantum verdicts. | Matching accuracy vs manual alignment; BF-CBOM's "almost no lexically exact matches" | Very high (you need it for scoring anyway). Low novelty alone, high as a contribution. |
| **4** | **Dead code reported as live** | Call-graph reachability from entry points: Go `golang.org/x/tools/go/callgraph` (VTA), Java SootUp/WALA, Python PyCG-like or Jarvis, JS Jelly or TAJS-lite, C# Roslyn. Report `reachable: true/false/unknown` instead of dropping findings. | Precision under the usage oracle on U1–U3 without recall loss on the inventory oracle | Medium: pick 1–2 languages |
| **5** | **Crypto hidden in dependencies** | SBOM (cdxgen) → knowledge base mapping purls (BouncyCastle, jjwt, PyJWT, jose, noble, SSH.NET) to the algorithms they expose, plus call-site linking of which library APIs the app uses. | P1–P3 recall | Medium–high |
| **6** | **Weak PQC/hybrid recognition** | Add rules for JDK 24 KEM, BC PQC, pyca mlkem/mldsa, Go mlkem/mldsa, .NET MLKem/MLDsa/SlhDsa/Composite, Node ml-kem/ml-dsa/slh-dsa; model hybrids as composite assets. | Q1–Q3 F1 + classification accuracy | Very high, but low novelty: fold into #1 |
| **7** | **Triage / explanation burden** | LLM-assisted *classification of already-extracted call sites* (not free-form CBOM generation), with schema-constrained output and abstention. | Precision gain on N1/R1 with a fixed recall budget; cost/time | Medium; evaluate carefully (precision issues in prior work) |

**Recommendation:** build ranks 1+2+6 as one tool, e.g. a "polyglot, config-aware CBOM scanner" with a pluggable per-language API model. Ship rank 3 as part of your benchmark harness. If time allows, add rank 4 for one language (Go is easiest, thanks to `x/tools` call graphs) to show a measurable precision gain on dead code. Your success criterion, "a clear benefit over at least one existing tool," is then very likely: C#/JS-TS recall vs CBOMkit (zero) is trivial. The *meaningful* comparison is against cdxgen on JS/TS and C#, and against CBOMkit on config cases in Java, Python and Go.

**Building blocks:** tree-sitter (+ py-tree-sitter / go-tree-sitter), Semgrep OSS (rule format and constant propagation), cyclonedx-python-lib or cyclonedx-go (schema-valid output), the CycloneDX Cryptography Registry, OID tables (from cdxgen's verified PQC OIDs), and sonar-cryptography's rule definitions as a reference catalogue of Java, Python and Go APIs.

---

## Part E — Kickoff plan

### E.1 Phased plan (two-semester baseline, about 30 weeks)

| Phase | Weeks | Deliverables / milestones |
|---|---|---|
| 0. Setup | 1 | Repo skeleton, Docker base images, `projects.yaml`, decision log |
| 1. Foundations | 2–4 | Reading notes. **Milestone M1:** short review draft (Part A material) |
| 2. Tool install and trial | 4–7 | Dockerized CBOMkit-lib/action, Theia, cdxgen 13.4.0, CodeQL+Forge. Install-effort log. Run on 2–3 toy cases per language. **M2:** "tool fact sheet" verified against your own runs |
| 3. Benchmark v1 | 6–12 | ~25 base cases × 5 languages (prioritize D, H, S, C, U, Q, N), ground-truth CBOMs, normalizer + scorer, CI. **M3:** benchmark v1 tagged |
| 4. Evaluation | 11–15 | Synthetic results + 2 real projects per language with sampled labels. **M4:** comparison report; pick the *measured* weakness |
| 5. Prototype | 15–25 | Core engine + 1–2 weakness modules; weekly regression runs on the benchmark. **M5:** prototype beats baseline on the chosen metric |
| 6. Hardening and docs | 25–30 | Install/usage/design docs, Zenodo release, final report |

### E.2 Key risks and mitigations

| Risk | Mitigation |
|---|---|
| Tool installs fail or drift (Sonar, atom/Java, CodeQL builds) | Docker with digests, record exact versions, fall back to cbomkit-lib instead of a SonarQube server, budget a week |
| Tools need buildable projects (Java/C#/Go for CodeQL; Java symbol resolution) | Prefer real projects with simple Maven/Gradle/`dotnet build`/`go build`; record "needs build" as a metric, not a failure |
| Ground-truth labeling effort | Small, mutation-multiplied synthetic set; sampling + κ for real projects; three-level scoring |
| Limited PQC APIs (Python no SLH-DSA; .NET needs OpenSSL 3.5 / CNG; Node needs OpenSSL 3.5) | Use BC/liboqs fallbacks; base images with OpenSSL 3.5+; note "experimental" APIs (.NET SYSLIB5006) |
| Fast-moving tools (cdxgen releases every 2–3 weeks) | Freeze versions at M2; rerun latest once at the end as a sensitivity check |
| Unfair granularity comparisons | Report all three levels and both dead-code oracles |
| Over-claiming | Phrase results as "what each tool can and cannot reliably tell an organization" (your project brief) |

### E.3 Scaling scope

- **One term (about 12–14 weeks):** 3 languages (C#, JS/TS plus one CBOMkit language for contrast), about 15 cases each, 1 real project per language, tools = CBOMkit-lib + cdxgen, with CryptoBOM-Forge documented but optional. Prototype = rank 1 for C# *or* JS/TS plus rank 2 config parsing.
- **Longer (MSc thesis, 9–12 months):** all 5 languages, mutation-scaled benchmark, 3–4 real projects per language, reachability (rank 4) for Go and Java, dependency mapping (rank 5), an LLM triage experiment (rank 7), and a contribution back to BF-CBOM, sonar-cryptography (e.g. issue #521) or the CycloneDX registry.

## Caveats

- **Verified directly:** facts on the tools, library versions and papers come from primary repos, docs, release notes and papers, except where a section says otherwise.
- **Secondary sources only:** FIPS 206 and HQC status, the IR 8547 "final" claim (conflicting), JDK 27 hybrid TLS (JEP 527), Go 1.27 details and the Canada dates. Re-check these on NIST, OpenJDK, go.dev and official government pages. The EU dates are confirmed by the European Commission's Digital Strategy page.
- **Not verified, so measure them in your benchmark:** whether sonar-cryptography's rules cover the new PQC APIs; cdxgen's Java and Python source-level CBOM depth and whether it sets `nistQuantumSecurityLevel`; and which CodeQL crypto queries exist for Python, Go, C# and JS.
- **Real-world project details** (licenses, PQC adoption in MINA SSHD, age, openpgp.js) come from background knowledge and are flagged "verify." Confirm them at the tag you pin.
- **Ranking of weaknesses:** the Part D ranking is an evidence-informed *hypothesis*. Your project brief requires choosing the weakness from measured results, so keep that decision open until milestone M4.

## Sources

1. [GitHub - Santandersecurityresearch/cryptobom-forge: Tools and utilities needed to parse GitHub Multi-Repository Variant Analysis output](https://github.com/Santandersecurityresearch/cryptobom-forge)
2. [.NET 9 and .NET 10: Post-Quantum Cryptography APIs](https://quantumsequrity.com/blog/dotnet-pqc)
3. [Changelog — Cryptography 51.0.0-dev1 documentation](https://cryptography.io/en/latest/changelog/)
4. [Crypto](https://nodejs.org/api/crypto.html)
5. [Quantum-Resistant ML-KEM and ML-DSA in Java](https://www.baeldung.com/java-quantum-resistant-ml-kem-ml-dsa)
6. [Package crypto - The Go Programming Language](https://golang.google.cn/pkg/crypto/)
7. [Go 1.27 brings post-quantum signatures to the stdlib](https://quantakrypto.com/blog/go-1-27-ml-dsa-post-quantum-signatures)
8. [FN-DSA: Compact Post-Quantum Signatures](https://www.encryptionconsulting.com/fn-dsa-compact-post-quantum-signatures/)
9. [NIST PQC Standards Guide: FIPS 203, 204, 205](https://www.arcqubit.ai/blog/nist-pqc-standards-guide)
10. [FN-DSA vs ML-DSA: 666-Byte Sigs, 2-Yr Standards Gap](https://shattered.io/fn-dsa-vs-ml-dsa-falcon-dilithium-2026/)
11. [NIST FIPS & PQC Standardization - The Infinite Unknown](https://www.jaredwatkins.com/research/post-quantum-encryption/standards-policy/nist-fips-pqc/)
12. [Post-Quantum Cryptography Standards and Audit Index: algorithms, dates, standards bodies, auditors](https://pqaudit.org/)
13. [UK Post-Quantum Cryptography Regulation: What NCSC's Migration Timeline Actually Requires](https://duokey.com/en/resources/uk-post-quantum-cryptography-regulation-ncsc-migration-timeline)
14. [UK NCSC migration timeline: dates and requirements](https://pqaudit.org/timelines/uk-ncsc-pqc-timeline/)
15. [PQC Roadmaps and Transition Guidance](https://pqshield.com/pqc-transition-roadmaps-and-guidance/)
16. [PQC Migration Roadmap for Enterprises](https://www.arcqubit.ai/resources/blog/pqc-migration-roadmap)
17. [The Ultimate Guide to Post-Quantum Cryptography and TLS 1.3](https://stevenpg.com/posts/ultimate-guide-post-quantum-cryptography-tls/)
18. [ML-KEM (FIPS 203) explained: what it replaces, how big it is and how to deploy it](https://www.cryptagion.io/resources/ml-kem-fips-203-explained)
19. [The 2025 Go Cryptography State of the Union](https://words.filippo.io/2025-state/)
20. [CBOM Examples: Annotated CycloneDX 1.7 JSON for a TLS Endpoint, an Application and an HSM Key](https://quantumsentinel.uk/cbom-examples-cyclonedx-json/)
21. [Post-Quantum Cryptography in Java: ML-KEM, ML-DSA and Hybrid TLS 1.3 (JDK 27)](https://ankurm.com/post-quantum-cryptography-java-ml-kem-ml-dsa-hybrid-tls/)
22. [The Java Platform and Post-Quantum Cryptography: From Threat Models to JDK Delivery · Neat Guy Coding](https://neatguycoding.com/posts/2026-05-18-javaone-2026-java-and-post-quantum-cryptography/)
23. [Java: PQC API misuse queries for experimental/quantum - wanted, and against which layer?](https://github.com/github/codeql/issues/22253)
24. [Post-quantum TLS in Go: X25519MLKEM768 and the go.mod trap](https://sslboard.com/blog/go-post-quantum-tls-mlkem.md/)
25. [ML-KEM and ML-DSA land in Python's crypto library - Berigo](https://berigo.no/en/news/python-crypto-library-gets-quantum-resistant-algorithms)
26. [What's new in .NET libraries for .NET 10](https://learn.microsoft.com/en-us/dotnet/core/whats-new/dotnet-10/libraries)
27. [docs/docs/core/whats-new/dotnet-10/libraries.md at main · dotnet/docs · GitHub](https://github.com/dotnet/docs/blob/main/docs/core/whats-new/dotnet-10/libraries.md)
28. [Web Crypto API](https://nodejs.org/api/webcrypto.html)
29. [Web Crypto API](https://beta.docs.nodejs.org/webcrypto)
30. [Node.js — Node.js 24.7.0 (Current)](https://nodejs.org/en/blog/release/v24.7.0)
31. [Post-Quantum Cryptography Migration: A 2026 Developer's Guide](https://devtoollab.com/blog/post-quantum-cryptography-migration-guide)
32. [GitHub - open-quantum-safe/liboqs: C library for prototyping and experimenting with quantum-resistant cryptography · GitHub](https://github.com/open-quantum-safe/liboqs)
33. [BF-CBOM: Uncovering Cryptographic Assets Through ...](https://romanboegli.ch/assets/pdf/Boegli_2026_BFCBOM_ICPC.pdf)
34. [Cryptographic bill of materials](https://en.wikipedia.org/wiki/Cryptographic_bill_of_materials)
35. [Hidden Ciphers and Where to Find Them:Static Discovery and Assessment of Cryptographic Assets in Software](https://arxiv.org/html/2608.04857)
36. [PQCA CBOMkit Architecture – Post-Quantum Cryptography Alliance](https://pqca.org/blog/2026/pqca-cbomkit-architecture/)
37. [\[Feature\]: Ingest CycloneDX 1.6 cryptographic assets (CBOM) into a crypto\_assets table · Issue #415 · seebom-labs/BOMHort · GitHub](https://github.com/seebom-labs/BOMHort/issues/415)
38. [cyclonedx.v1\_7 - CycloneDX/specification - protodoc.io](https://protodoc.io/CycloneDX/specification/cyclonedx.v1_7)
39. [\[SEV2\] \[conformance\]\[cbom\] classicalSecurityLevel declares no resistance for a hash, and two of our algorithms have no algorithmFamily · Issue #959 · BreachSAFE/qureddy · GitHub](https://github.com/BreachSAFE/qureddy/issues/959)
40. [Cryptography Registry](https://cyclonedx.org/registry/cryptography/)
41. [GitHub - cbomkit/cbomkit-theia: A tool for detecting cryptographic assets in container images and directories, and generating CBOMs. · GitHub](https://github.com/cbomkit/cbomkit-theia)
42. [CBOM Tools: Open-Source Cryptographic Discovery Tools](https://intellixbom.com/resources/cbom/cbom-tools/)
43. [github/codeql](https://deepwiki.com/github/codeql)
44. [GitHub - advanced-security/cbom-action: Create a Crypto Bill of Materials using CodeQL · GitHub](https://github.com/advanced-security/cbom-action)
45. [Santandersecurityresearch repositories · GitHub](https://github.com/orgs/Santandersecurityresearch/repositories?type=all)
46. [Hidden Ciphers and Where to Find Them: Static Discovery and Assessment of Cryptographic Assets in Software](https://arxiv.org/pdf/2608.04857)
47. [GitHub - udsy19/quipuu: Find every cryptographic operation in your codebase, classified against the NIST post-quantum migration timeline. Single Rust binary, 7 languages, CycloneDX CBOM + SARIF. · GitHub](https://github.com/udsy19/quipuu)
48. [GitHub - quantakrypto/pqc-tools: Open-source post-quantum readiness tooling by quantakrypto · GitHub](https://github.com/quantakrypto/pqc-tools)
49. [GitHub - BreachSAFE/qureddy: Open-source post-quantum cryptography readiness scanner — find what's quantum-vulnerable in your TLS deployment · GitHub](https://github.com/breachsafe/qureddy)
50. [Cryptographic API Misuse Detection Tool Benchmark Suite](https://arxiv.org/pdf/2204.06447)
51. [CryptoAPI-Bench: A Comprehensive Benchmark on Java Cryptographic API Misuses](https://www.researchgate.net/publication/337528014_CryptoAPI-Bench_A_Comprehensive_Benchmark_on_Java_Cryptographic_API_Misuses)
52. [Mutation-based Evaluation of Cryptographic API Misuse Detectors](https://dl.acm.org/doi/full/10.1145/3796221)
53. [Mutation-based Evaluation of Cryptographic API Misuse Detectors](https://arxiv.org/html/2107.07065)
54. [Beyond Static Tools: Evaluating Large Language Models for Cryptographic Misuse Detection](https://arxiv.org/pdf/2411.09772)
55. [Explorng Automatic Cryptographic API Misuse Detection in the Era of LLMs](https://arxiv.org/html/2407.16576v1)
56. [BF-CBOM: Uncovering Cryptographic Assets Through Comparative CBOM Analysis at Scale](https://dl.acm.org/doi/full/10.1145/3794763.3794831)
57. [Cryptoscope: Analyzing cryptographic usages in modern software](https://arxiv.org/html/2503.19531v1)
58. [Towards Cryptography Bill of Materials Compliance](https://link.springer.com/chapter/10.1007/978-3-032-19567-8_10)
59. [Architecture-Derived CBOMs for Cryptographic](https://arxiv.org/pdf/2603.22442)
60. [Static Detection of Post-Quantum Cryptographic Algorithms in Stripped Binaries for Digital Forensic Examination and Migration Assurance](https://arxiv.org/pdf/2608.25122)
61. [Can generative AI detect and fix real-world cryptographic misuses? - ScienceDirect](https://www.sciencedirect.com/science/article/pii/S016412122500319X)
62. [CryptanalysisBench: Can LLMs do Cryptanalysis?](https://arxiv.org/abs/2607.18538)
63. [CBOMkit · GitHub](https://github.com/cbomkit)
64. [GitHub - cbomkit/sonar-cryptography: This repository contains a SonarQube Plugin that detects cryptographic assets in source code and generates CBOM. · GitHub](https://github.com/cbomkit/sonar-cryptography)
65. [Releases · cbomkit/sonar-cryptography · GitHub](https://github.com/cbomkit/sonar-cryptography/releases)
66. [Support for JavaScript / Typescript in Sonar-Cryptography plugin · Issue #521 · cbomkit/sonar-cryptography · GitHub](https://github.com/cbomkit/sonar-cryptography/issues/521)
67. [Sonar Cryptography Plugin](https://pqca.org/blog/2026/sonar-cryptography-plugin/)
68. [Sonar Cryptography Plugin. The Sonar Cryptography Plugin…](https://medium.com/@chmodshubham/sonar-cryptography-plugin-cb2a772b61ff)
69. [CBOMkit Now Supports Go (Golang)](https://pqca.org/blog/2026/cbomkit-now-supports-go/)
70. [\[SEV4\] \[cbom\]\[enhancement\] Adopt cbomkit-theia as a scan component (image/dir crypto discovery → CycloneDX), chained per #474 · Issue #749 · BreachSAFE/qureddy · GitHub](https://github.com/BreachSAFE/qureddy/issues/749)
71. [Cbomkit-theia image does not detect cryptographic material that cbomkit-theia dir detects on the same filesystem content · Issue #195 · cbomkit/cbomkit-theia · GitHub](https://github.com/cbomkit/cbomkit-theia/issues/195)
72. [GitHub - cdxgen/cdxgen: Creates CycloneDX Bill of Materials (BOM) for your projects from source and container images. Supports many languages and package managers. Integrate in your CI/CD pipeline with automatic submission to Dependency Track server](https://github.com/cdxgen/cdxgen)
73. [cdxgen: docs/ADVANCED.md | Fossies](https://fossies.org/linux/cdxgen/docs/ADVANCED.md)
74. [Release Release v13.4.0 · cdxgen/cdxgen · GitHub](https://github.com/cdxgen/cdxgen/releases/tag/v13.4.0)
75. [Releases · cdxgen/cdxgen · GitHub](https://github.com/cdxgen/cdxgen/releases)
76. [cdxgen/cdxgen on GitHub](https://releasealert.dev/github/cdxgen/cdxgen)
77. [BOM Types and Formats](https://deepwiki.com/cdxgen/cdxgen/2.1-bom-types-and-formats)
78. [cdxgen: docs/CLI.md](https://fossies.org/linux/cdxgen/docs/CLI.md)
79. [crypto fails with OutOfMemoryException once its JSON passes 2 GB, and cdxgen drops it from 512 MB · Issue #75 · owasp-dep-scan/dosai · GitHub](https://github.com/owasp-dep-scan/dosai/issues/75)
80. [Daniel Cuthbert on LinkedIn: GitHub - Santandersecurityresearch/cryptobom-forge: Tools and utilities…](https://www.linkedin.com/posts/daniel-cuthbert0x_github-santandersecurityresearchcryptobom-forge-activity-7138145659735429120-NMlb)
81. [Santander Security Research · GitHub](https://github.com/Santandersecurityresearch)
82. [The Week in Quantum Computing - December 11th - Quantum is now corrected!](https://quantumpirates.substack.com/p/the-week-in-quantum-computing-december-3fc)
83. [GitHub CodeQL Review 2026: Semantic Security Analysis](https://appsecsanta.com/github-codeql)
84. [Introduction](https://reestwick.github.io/sbomgen/tutorials/cbom/creating-cbom/creating-cryptographic-bill-of-materials.html)
85. [Addressing post-quantum cryptography with CodeQL - The GitHub Blog](https://github.blog/security/vulnerability-research/addressing-post-quantum-cryptography-with-codeql/)
