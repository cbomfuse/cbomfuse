# Reconciling Multi-Tool CBOMs with a CLIPS Expert System on Top of BF-CBOM: A Candid Feasibility and Research Assessment

Bottom line: the idea is buildable and has a narrow, defensible core, which is an explainable, auditable rule layer that reconciles disagreeing CBOM generators into one schema-valid CycloneDX document with per-field provenance. As currently framed, though ("CLIPS expert system + confidence scores + risk scores + PQC classification"), it would not survive peer review. BF-CBOM is a comparison harness, not a data model. CLIPS is not where the hard problems live. Confidence and risk numbers produced without a labeled corpus and deployment context would be arbitrary.

## TL;DR

- **Verdict:** Reframe the work as "explainable, provenance-preserving reconciliation of heterogeneous CBOMs, evaluated against a hand-labeled ground-truth corpus." Treat confidence as calibrated, measured output, not designed-in. Restrict "risk" to a transparent, context-parameterized classification rather than a single number. PQC classification is the most deterministic and defensible piece, but CBOMkit already ships a basic quantum-safe compliance check and commercial platforms already do it at scale.
- **Foundations are weaker than they look:** BF-CBOM (University of Bern, ICPC 2026 tool demo) is an orchestration and comparison framework. It defines no data model, explicitly does not resolve discrepancies, has only 12 GitHub stars, and is GPL-3.0. The tools it wraps emit different CycloneDX versions: CryptobomForge's documented output is the pre-standard IBM "1.4-cbom-1.0" format, sonar-cryptography emits 1.6, and newer tools emit 1.7. So the premise that "each tool produces CycloneDX-compliant output" is already false for at least one priority tool.
- **CLIPS is acceptable but not the bottleneck, and probably not the best choice:** CLIPS 6.4.2 (January 2025, public domain) gives you Rete, modules, logical support and traces. It has no JSON I/O and no native uncertainty, and FuzzyCLIPS is a legacy fork. Entity resolution, normalization, calibration and serialization belong in Python. Use CLIPS (or Soufflé/OPA) only for the declarative conflict-resolution and classification rules, where explanation traces are the real value.

## 1. Conventions

**[Verified]** = confirmed against a primary or near-primary source (spec, repository, paper, vendor documentation). **[Reported]** = secondary source only. **[Analysis]** = my own reasoning. Research date: October 23, 2026.

## 2. Background

### 2.1 BF-CBOM: what it actually is (and is not)

**Disambiguation.** "BF" collides with NIST's **Bugs Framework (BF)**, Irena Bojanova's formal classification of software weaknesses (NIST SP 800-231, July 2024), which includes cryptography-related bug classes (ENC, VRF, KMN).\[1\]\[2\] The NIST Bugs Framework has nothing to do with CBOMs. The user's "BF-CBOM" is, with high confidence, the University of Bern artifact below. **[Verified]**

**Identity.** **[Verified]**
- **Paper:** Roman Bögli, Jonas Spieler, Timo Kehrer, "BF-CBOM: Uncovering Cryptographic Assets Through Comparative CBOM Analysis at Scale," ICPC '26 Tool Demonstration, Rio de Janeiro, April 12–13, 2026, DOI 10.1145/3794763.3794831 (5 pages, CC-BY 4.0).\[3\]
- **Repository:** `SEG-UNIBE/BF-CBOM` on GitHub (GPL-3.0, Python 3.12, `uv`, Streamlit UI, Redis, Docker Compose).\[4\] Zenodo DOIs 10.5281/zenodo.17220030 (v1.0.1, September 28, 2025) and 10.5281/zenodo.17853399 (v1.0.3, December 2025).\[5\]\[6\]
- **Name drift:** the acronym itself changed meaning. Early Zenodo releases expand it as "**Benchmarking Framework** for Cryptography Bill of Material Generator Tools." Later releases and the paper use "**Best Friend** for Generating, Understanding, and Comparing CBOMs."\[6\]\[7\] Cite the ICPC paper and a specific release tag.

**What it provides.** **[Verified]**
- A coordinator (Streamlit GUI plus Typer CLI) that dispatches jobs over Redis to containerized "workers." Each worker wraps one generator: CBOMkit v2.1.10, cdxgen, CryptobomForge v1.1.0 (which first runs CodeQL to produce SARIF), and a zero-shot DeepSeek LLM worker.\[3\]
- Batch inspection of GitHub repositories, runtime and component-count statistics, and drill-down into raw CBOM fragments.\[3\]
- A comparative analysis that treats cross-tool matching as an **n-way model-matching problem**. It uses **RaQuN** (Schultheiß et al., SoSyM 2023) over components vectorized from selected JSON fields embedded with **MiniLM-v2**.\[3\]

**What it explicitly does not do.** The authors write that BF-CBOM "is not designed to resolve discrepancies but to actively surface them." Their pilot found that "reported cryptographic assets were strikingly inconsistent… Overlaps were rare, with almost no lexically exact matches." Ground-truth datasets "with manually verified cryptographic inventories" are listed as future work, and a systematic validation is reserved for a later full paper.\[3\] **[Verified]**

**Data model and relation to CycloneDX.** BF-CBOM defines **no data model, schema extension, or merged-output format**. It consumes whatever JSON the tools emit and compares them at the component level. It neither extends nor constrains the CycloneDX CBOM model. **[Verified from paper and README; Analysis on implication]**

**Maturity, adoption and maintenance.** At fetch time the repo showed 12 stars, 1 fork, 0 watchers, 1 open issue and about 202 commits.\[4\] It is an academic prototype from one research group. A related supervised thesis ("CBOMbench") and a "Dynamic CBOM" runtime thesis exist on the first author's page.\[8\] I could not verify commit activity after December 2025. **[Verified counts; maintenance cadence unverified]**

**What it would contribute as a foundation [Analysis]:**
- *Useful:* reproducible containerized execution of heterogeneous generators, which saves weeks of toolchain pain (CodeQL alone is heavy). It also gives you an existing cross-tool matcher you can use as a **baseline** to beat, and a natural host for a "reconciler" stage.
- *Not useful:* any normative model of what a reconciled asset is, ground truth, provenance semantics, or schema-valid output.

**Risks of building on it [Analysis]:**
1. **License contagion.** GPL-3.0 means a tightly coupled derivative inherits GPL obligations. That matters if an industry partner is involved.
2. **Bus factor and drift.** It pins specific generator versions, and generators change output formats quickly (CycloneDX 1.6→1.7 migrations are in progress; see §2.2). An unmaintained harness silently goes stale.
3. **Methodological coupling.** If your evaluation reuses BF-CBOM's embedding-based matcher both to build *and* to evaluate merges, you have a circularity problem.
4. **Novelty optics.** Reviewers will read "expert system on top of BF-CBOM" as "the resolution step BF-CBOM's authors deferred," which is a thin delta unless done rigorously.

**Recommendation:** treat BF-CBOM as an *optional execution substrate*, consumed through its output artifacts rather than its code. Your reconciler should accept arbitrary CBOM files so it is not hostage to the harness.

### 2.2 The CycloneDX CBOM data model (1.6 → 1.7)

**Versions and status.** **[Verified]**
- CycloneDX 1.6 (April 9, 2024) introduced CBOM: the `cryptographic-asset` component type with `cryptoProperties`. It was ratified as ECMA-424 1st edition.\[9\]
- CycloneDX **1.7** (released October 21, 2025, described as "the final version in the 1.x series") is ECMA-424 2nd edition (December 2025). It has patch releases up to **1.7.2**.\[9\]\[10\]\[11\]
- 1.7 adds a **Cryptography Registry** (`cryptography-defs.schema.json`) with an `algorithmFamily` enum and an `ellipticCurve` enum of namespaced curves (e.g., `nist/P-256`). The free-text `curve` field is deprecated.\[12\] Per the sonar-cryptography upgrade epic (issue #505), 1.7 also deprecates `algorithmRef`, `signatureAlgorithmRef`, `subjectPublicKeyRef`, `protocolProperties.cryptoRefArray` and `certificateExtension`. The same epic counts 93 family identifiers and 246 curve identifiers.\[13\]
- The registry evolves independently of the spec. Recent patch releases added SSH-KDF, HPKE, OPAQUE, Argon2, scrypt and AES-GCM-SIV, and deduplicated MD4/MD5 entries.\[14\]\[15\]

Practically, a 1.6 document and a 1.7 document from two tools describing the same asset may *legitimately* use different fields.

**Model elements relevant to reconciliation** **[Verified at the level of field existence; semantics summarized]**:

| Element | What it carries | Relevance to the merger |
|---|---|---|
| `cryptoProperties.assetType` | `algorithm`, `certificate`, `protocol`, `related-crypto-material` | First-level blocking key for matching |
| `algorithmProperties` | `primitive`, `parameterSetIdentifier`, `curve` (deprecated in 1.7 → `ellipticCurve`), `mode`, `padding`, `cryptoFunctions`, `executionEnvironment`, `implementationPlatform`, `certificationLevel`, `classicalSecurityLevel`, `nistQuantumSecurityLevel`; 1.7 adds `algorithmFamily` | Attribute-level conflicts live here |
| `oid` | Algorithm OID | Strongest normalization anchor when present |
| `bom-ref` | Document-local identifier | **Not** stable across tools; must be re-minted |
| `dependencies` (`dependsOn`, `provides`) | Component→crypto "uses/implements" graph | Must be re-wired after merging |
| `evidence.occurrences` | Locations (file, line, offset, symbol, additional context) | Basis for call-site-level identity |
| `evidence.callstack` | Frames | cdxgen notes only one callstack is retained per component\[16\] |
| `evidence.identity[]` | `field`, `confidence` (0–1), `concludedValue`, `methods[]` (`technique`, `confidence`, `value`), `tools` | Closest native carrier for a confidence score\[12\]\[17\]\[18\] |
| `annotations` | Free-text notes by annotators (person/org/tool), optionally signed | Human-readable rationale and explanation traces |
| `properties` | Name/value pairs; names governed by the CycloneDX Property Taxonomy (`cdx:` namespace plus registered vendor namespaces) | Carrier for anything non-native |
| `vulnerabilities[].ratings` | `score`, `severity`, `method` (CVSS versions, OWASP, SSVC, `other`), `vector`, `justification` | Possible but semantically awkward carrier for "risk" |
| `metadata.tools` | Since 1.5, an object with `components`/`services` (the legacy array is deprecated) | Where each source generator plus the reconciler are declared |
| **1.7 `citations`** | Top-level array; each citation has `bom-ref`, exactly one of `pointers` (RFC 6901 JSON Pointers) or `expressions` (path expressions), at least one of `attributedTo` (bom-ref of a tool/person/org/component) or `process` (bom-ref into `formulation`), plus `timestamp` and optional `note` | **Native per-field provenance**, the single most important 1.7 feature for this project |

The `citations` structure is verified from the CycloneDX 1.7 schema documentation. The spec states: "A collection of attributions indicating which entity supplied information for specific fields within the BOM," and "Exactly one of the 'pointers' or 'expressions' elements must be present."\[12\]\[19\] **[Verified]**

**Which desired outputs are native vs. custom [Analysis grounded in the schema]:**

| Desired output | Native & schema-valid? | Recommended encoding | Caveat |
|---|---|---|---|
| Per-tool provenance (which tool reported the asset) | **Yes** | `metadata.tools.components[]` for each generator; `evidence.identity[].tools` referencing those bom-refs | `identity.field` is designed for component identity fields (name/purl/cpe/hash etc.), not crypto attributes |
| Per-*field* provenance (which tool supplied `mode`) | **Yes, in 1.7 only** | `citations[]` with JSON Pointers to `/components/i/cryptoProperties/algorithmProperties/mode`, `attributedTo` = tool bom-ref | JSON Pointers are index-based and break if components are re-ordered; serialize first, then emit citations. Downgrading to 1.6 loses this |
| Confidence of existence/identity | **Partially** | `evidence.identity[].confidence` plus `methods[].confidence` with technique (`source-code-analysis`, `binary-analysis`, `filename`, `instrumentation`…) | Semantics are "confidence in the identity claim," not "probability the asset is really used." Stretching it is defensible only if documented |
| Confidence per attribute (e.g., key size) | **No** | `properties` in a registered/own namespace (e.g., `acme:cbom:confidence:keySize`), or 1.7 `citations` + `annotations` | Not interoperable; consumers will ignore it |
| PQC classification | **Partially** | `nistQuantumSecurityLevel` (0–6) is native but is a *security strength category*, not a migration verdict. A categorical label ("quantum-vulnerable-shor") needs `properties` | See §6.4 on the field's limits |
| Risk score | **Only by abuse** | `vulnerabilities[]` with `ratings.method: "other"` and `justification`, or `properties` | Quantum exposure is not a vulnerability in the CVE sense. Putting it in `vulnerabilities` will pollute downstream VEX/vulnerability tooling |
| Explanation of decisions | **Yes** | `annotations` (annotator = the reconciler tool) | Free text; not machine-checkable |

**What is lost or broken when merging [Analysis, partly Verified]:**
- **Signatures:** JSF/XML signatures on inputs are invalidated by any transformation. The merged BOM must be re-signed by the reconciler, and the original signatures can at best be preserved by reference (external references to the originals).
- **serialNumber/version:** a merged document is a *new* BOM and needs a new `urn:uuid` serial number. Link the inputs via BOM-Link external references or `formulation` rather than reusing their serials.
- **bom-ref uniqueness:** bom-refs are only document-unique. Two tools commonly produce colliding or semantically unrelated refs (UUIDs vs. `crypto/algorithm/aes-128-gcm@...` path-like strings). All refs must be re-minted, and every `dependencies` edge rewritten. Hierarchical merge in `cyclonedx-cli` is known to break path-like ref conventions (Festo cyclonedx-editor-validator issue #385).\[20\]
- **Per-property provenance:** lost entirely in 1.6 unless you invent properties. Recoverable only with 1.7 `citations`.
- **Version mixing:** merging a 1.6 and a 1.7 input forces a target choice. Targeting 1.7 means mapping `curve` → `ellipticCurve` (registry identifiers only, so a wrong value invalidates the whole document).\[21\] Targeting 1.6 discards registry normalization and citations.

### 2.3 CLIPS today

**[Verified]**
- CLIPS **6.4.2** was released January 27, 2025, with new functions including `try`, `with-open-file`, `str-byte-length` and `fact-index-to-fact`.\[22\]\[23\] CLIPS 6.4.1 dates from April 2023.\[24\]
- Development is essentially by one maintainer (Gary Riley), with an active Google Group.\[25\]
- CLIPS has been **public domain** since 1996 per clipsrules.net.\[26\]
- It was developed at NASA JSC from 1985, written in C, and provides forward-chaining production rules plus a procedural language and the COOL object system.\[26\]\[27\]

**[Reported/known from the CLIPS reference manuals; not re-verified line-by-line]** CLIPS offers a Rete-based matcher; `deftemplate` facts and COOL `defclass` instances; `salience` plus conflict-resolution strategies (depth, breadth, simplicity, complexity, LEX, MEA, random); `defmodule` with focus stacks; the `logical` conditional element for truth maintenance (supported facts are retracted when their support disappears); and tracing via `watch`, `matches` and `dribble`. It has **no native JSON parser or serializer** and **no native certainty factors or probability**. The CLIPSpy Python binding is maintained: clipspy 1.0.6 was released October 26, 2025 and is described on PyPI as "Python CFFI bindings for the C Language Integrated Production System CLIPS 6.42." It followed 1.0.5 (February 17, 2025) and 1.0.4 (May 26, 2024).

## 3. The problem, and evidence that it is real

**The problem is real, and it is documented.** **[Verified]**
- BF-CBOM's pilot found almost no lexically exact cross-tool matches and "strikingly inconsistent" asset sets and metadata.\[3\]
- The CycloneDX project itself says the 1.7 registry was created because "inconsistent naming and classification of cryptographic algorithms across different tools and organizations… hindered policy enforcement, PQC readiness assessments, and meaningful cross-organizational analysis."\[14\]
- Adjacent SBOM research shows the same pathology at larger scale: differential analysis of metadata-based SBOM generators (Yu et al., DSN 2024), Python SBOM generator comparisons (Cofano et al., TrustCom 2024; Benedetti et al., ACNS 2025), and "Comparing Bills of Materials" (Tate et al., arXiv 2411.10384).\[3\]

**But "real" ≠ "needs an expert system."** Most of the observed divergence comes from three things [Analysis]:
1. **Coverage differences.** Tools look at different languages, libraries and artifact types, so they are *complementary*, not *conflicting*.
2. **Naming differences.** This is largely a normalization problem, which the 1.7 registry now targets.
3. **Granularity differences.** One component per algorithm vs. per call site.

Only a minority of cases are true attribute conflicts (e.g., one tool says AES-128-GCM and another says AES-256-CBC for the same call site). The research contribution has to be located there, and in honest evidence about how often it happens.

### 3.1 How the priority tools actually differ

| Dimension | sonar-cryptography / CBOMkit (PQCA) | CBOMkit-theia (PQCA) | cdxgen `--include-crypto` / `cbom` | CryptobomForge (Santander) |
|---|---|---|---|---|
| Technique | Static AST analysis via SonarQube language parsers; rule per API pattern | Filesystem/container artifact scan (certs, keys, secrets via gitleaks, `java.security`, `openssl.cnf`, problematic CAs) | Package/manifest analysis plus crypto-library detection, keystores/certificates, JS/TS source constant propagation; .NET via `dosai` | Post-processes CodeQL SARIF output\[28\]\[29\]\[30\] |
| Languages | Java (JCA, BouncyCastle LW API), Python (pyca/cryptography), Go (stdlib `crypto/*` except `crypto/x509`; partial `x/crypto`, added 2026); C# in development; JS/TS proposed (issue #521, August 2026) | Language-agnostic (artifacts) | **Conflicting sources:** BF-CBOM paper says Java + Python; current README says Java keystores/certs plus JS/TS source; a sbomify PR reports 0 assets from a Python project using hashlib/AES/RSA | Whatever CodeQL queries cover\[30\]\[31\]\[32\]\[33\] |
| Spec emitted | CycloneDX **1.6** (upgrade to 1.7 tracked in issue #505) | CycloneDX **1.6** | 1.6 historically; current docs show `--spec-version 1.7` in the `cbom` alias | README example shows `"bomFormat": "CBOM"`, `"specVersion": "1.4-cbom-1.0"` with IBM's pre-standard schema; SandboxAQ docs say they validated CycloneDX 1.4 input from CryptoBOM Forge\[30\] |
| Granularity | One asset node per detection, with occurrences carrying file and line; `INode` tree with `origin` = detected / **inferred** / **enriched** | One per artifact (cert, key, config) | Component-level, evidence-mode | Per SARIF result\[30\] |
| Enrichment | Enricher adds OIDs and may **assign a default mode when none is explicit** | Can enrich an input BOM; `javasecurity` plugin assigns a 0–1 confidence of executability based on `jdk.tls.disabledAlgorithms` | – | Rules-based weak-algorithm checks\[30\] |
| Built-in policy | CBOMkit `BasicQuantumSafeComplianceService` (whitelist of quantum-safe OIDs/names; non-listed → non-compliant; unclassifiable → unknown); optional OPA sidecar and external regulator API | Flags problematic CAs | – | Weak algorithm/mode checks\[30\] |
| Maintenance | Active (2026 releases, Go support, PQCA governance after IBM's June 2025 donation) | Active | Very active | v1.1.0 (January 25, 2024), the version BF-CBOM pinned, is still marked "Latest" on GitHub Releases. There are only three releases in total (1.0.0, 1.0.1, 1.1.0). The repo shows an update on May 20, 2026 but no new tag |

**Implications [Analysis]:**
- **The input premise is false for at least one tool.** CryptobomForge's documented output is not CycloneDX 1.6+.\[34\] Your ingestion stage needs a legacy adapter (IBM CBOM 1.4-cbom-1.0 → 1.6/1.7) or you must drop the tool.
- **Correlated provenance.** CBOMkit-theia is *designed* to enrich a sonar-cryptography BOM. If you feed both outputs to the reconciler, the theia output already contains the sonar assets, so counting them as two votes is double-counting.\[30\]
- **Injected defaults are a trap.** If sonar-cryptography fills a default mode that the code never specified, a "majority vote" can be won by an inference, not an observation. The `origin` attribute must be carried into your facts and down-weighted.\[30\]
- **Naming divergence is structural.** JCA transformation strings ("AES/GCM/NoPadding", "SHA256withRSA"), pyca class names, and CodeQL query labels yield different decompositions. "SHA256withRSA" is *two* assets (a hash and a signature scheme) plus a composition. A string-level merge will get this wrong.

### 3.2 Other tools worth knowing

- **Open source / academic:**
  - IBM's Cryptoscope (arXiv 2503.19531): CBOM-consistent, algorithms and related material only.\[35\]
  - OmniTrust **CBOM Lens** 1.1.0: claims to be the first producer to emit the 1.7 registry; scans filesystems, images and ports; "omits values it cannot prove."\[21\] That design philosophy matters for voting.
  - `aclater/pqc-readiness`: 1.7 CBOM by default. It notes HQC and FN-DSA are *absent* from the 1.7 registry.\[36\]
  - Santander's CADI index of crypto-discovery tools.\[37\]
- **Commercial aggregators** **[Verified from vendor docs unless noted]**:
  - **IBM Guardium Cryptography Manager** "ingests data from… network scanners and Kubernetes; Vault…; Qualys and Nessus…; Cryptographic Bill of Materials (CBOM) for code assets; and IBM Quantum Safe Explorer." It assigns "an exploitability score… using CVSS metrics" and an "enterprise risk score" (no formula published).\[38\]\[39\] IBM Quantum Safe Explorer emits CycloneDX 1.6 CBOMs with Minor/Major/Critical severities.\[40\]
  - **SandboxAQ AQtive Guard** imports CBOM JSON. It "validated" CycloneDX 1.4 from CryptoBOM Forge and 1.6 from IBM's Sonar plugin, alongside its own filesystem, Java-tracer and network sensors and many integrations. Issues are "ranked by Severity and the number of Occurrences," and context can raise or lower severity.\[41\]\[42\]\[43\]
  - **Keyfactor AgileSec Analytics** exports CycloneDX 1.6 CBOM per source. Policies compute a combined `score_value` (0–10), `severity_score` (0–3), and a separate PQC `priority_score` (0–3).\[44\]\[45\] I found no documented CBOM *import*.

**Implication [Analysis]:** multi-sensor aggregation plus scoring is **commercially solved enough** that "we merge CBOMs and assign risk scores" is not novel. What none of these vendors publish is a **transparent, reproducible, evaluated** reconciliation method with calibrated confidence. That is the gap.

## 4. Existing merge tooling and why it is insufficient

- **`cyclonedx-cli merge`** **[Verified]**: flat and hierarchical modes. Hierarchical requires `--name`/`--version` and a `metadata.component` in every input. Known issues include misplaced properties producing invalid output (issue #304), missing-metadata exceptions (issue #377), and practitioners reporting that neither cyclonedx-cli nor sbomasm "merge duplicate components" (cdxgen discussion #1835).\[46\]\[47\]\[48\]\[49\]\[50\]
- **sbomasm** (Interlynk) and **bomctl**/**sbom-utility**: general SBOM assembly and manipulation; I did not verify any CBOM-aware deduplication in them. **[Unverified]** The CycloneDX RFC discussion #320 catalogs flat, hierarchical and reference merges, but no crypto-semantic merge.\[51\]
- **CBOM-specific:**
  - cbomkit-lib's `CBOM.merge()` merges per-language CBOMs from *the same engine*, which is a trivial case.\[30\]
  - cbomkit-theia's `provider/cyclonedx` has component/dependency merge utilities for enrichment.\[30\]
  - BF-CBOM's RaQuN matching *aligns* but does not *merge*.

**Verdict [Analysis]:** there is no published, crypto-semantics-aware, provenance-preserving reconciler. That gap is genuine but narrow. Generic merge is "just engineering." Semantic identity resolution, conflict resolution and calibrated confidence over crypto assets are the research-worthy parts.

## 5. Proposed architecture

| Stage | Where | Why |
|---|---|---|
| 1. Ingest/validate per version | Python (cyclonedx-python-lib or `jsonschema` against bundled 1.6/1.7.2 schemas; legacy IBM 1.4-cbom adapter) | CLIPS cannot parse JSON. Validation is a solved library problem |
| 2. Normalize | Python | Registry lookup (`algorithmFamily`, `ellipticCurve`), OID tables, transformation-string parsing ("SHA256withRSA" → {SHA-256, RSA-PKCS1v1.5-sig}), parameter extraction. Deterministic table-driven code; testable with property tests |
| 3. Blocking + matching | Python | Pairwise matching is O(n²) per block. Blocking on (assetType, family, file) makes it tractable. Similarity features and possibly RaQuN or learned matchers sit poorly in Rete |
| 4. Conflict resolution | **CLIPS** (or Soufflé) | Declarative, auditable precedence rules: observed > inferred > enriched; specific > generic; technique priors. This is where `watch rules` traces and `logical` support pay off |
| 5a. PQC classification | **CLIPS** | Small, stable, explainable decision table keyed on family/parameter set/function |
| 5b. Confidence | Python (calibration) + CLIPS (evidence features) | Rules assert *evidence facts*. A fitted, calibrated model turns them into numbers. Never hand-tune CF algebra |
| 5c. "Risk" | CLIPS rules over *external context facts* | Only meaningful with deployment/data context (§6.3) |
| 6. Provenance | Python | Build `citations` after final ordering. Turn rule-firing traces into `annotations` |
| 7. Serialize/validate/sign | Python | Deterministic ordering, schema validation, JSF signature |

**Illustrative CLIPS fragment** for the one place CLIPS genuinely shines (auditable conflict resolution with truth maintenance) **[Analysis; illustrative, not tested]**:

```clips
(deftemplate claim
  (slot asset) (slot attr) (slot value)
  (slot tool) (slot origin (allowed-symbols detected inferred enriched))
  (slot technique))

(deftemplate resolved (slot asset) (slot attr) (slot value) (slot rule))

(defrule prefer-observed-over-inferred
  (declare (salience 100))
  (logical (claim (asset ?a) (attr ?k) (value ?v1) (origin detected) (tool ?t1))
           (claim (asset ?a) (attr ?k) (value ?v2&~?v1) (origin inferred|enriched)))
  (not (resolved (asset ?a) (attr ?k)))
  =>
  (assert (resolved (asset ?a) (attr ?k) (value ?v1)
                    (rule prefer-observed-over-inferred))))
```

If the supporting `detected` claim is later retracted (e.g., the occurrence is reclassified as test code), the `logical` CE retracts the conclusion automatically. That is real, explainable non-monotonic behavior that is clumsy in a plain Python pipeline.

**Illustrative 1.7 output fragment** (per-field provenance plus evidence confidence) **[Analysis; structure per the 1.7 schema]**:

```json
{
  "bom-ref": "ca:alg:aes-256-gcm:src/Enc.java:42",
  "type": "cryptographic-asset",
  "name": "AES-256-GCM",
  "cryptoProperties": {
    "assetType": "algorithm",
    "algorithmProperties": { "algorithmFamily": "AES", "primitive": "ae",
      "mode": "gcm", "parameterSetIdentifier": "256", "nistQuantumSecurityLevel": 1 }
  },
  "evidence": {
    "occurrences": [ { "location": "src/Enc.java", "line": 42 } ],
    "identity": [ { "field": "name", "confidence": 0.82,
      "methods": [ { "technique": "source-code-analysis", "confidence": 0.9, "value": "sonar-cryptography" } ],
      "tools": [ "tool:sonar-cryptography", "tool:cryptobom-forge" ] } ]
  },
  "properties": [ { "name": "acme:cbom:pqc-class", "value": "grover-affected-adequate" } ]
}
```

## 6. Capability-by-capability analysis

### 6.1 Unification / merging (entity resolution + truth discovery)

**Identity is the crux, and there is no single right answer [Analysis].** Candidate identity levels:

1. **Algorithm-type identity** (family + parameter set + mode + function): "the application uses AES-256-GCM." This is good for PQC posture and bad for remediation.
2. **Call-site identity** (algorithm + file + line/symbol): needed for remediation and for matching static tools. It breaks across tools that report different lines for the same call (constructor vs. `init` vs. `doFinal`) and across code versions.
3. **Library/provider identity** (the algorithm as implemented by OpenSSL 3.5 vs. BouncyCastle): needed for FIPS/certification questions.
4. **Material identity** (a specific certificate or key, by fingerprint): the only level where identity is *crisp* (hash equality).

A defensible design keeps **two levels explicitly**: an asset-type node, and occurrence children that link to it. Matching then happens per level with level-specific keys. Do not let one tool's granularity define identity for all tools.

**Normalization.** Map every name to a registry `algorithmFamily` plus a structured parameter tuple. Decompose composite strings. Keep the original string in a property. Unmappable names become an explicit `unknown` state, not dropped data.

**Granularity mismatch.** When tool A reports one component with 12 occurrences and tool B reports 12 components, do not merge B's components into one by name. Match each B component to A's occurrences by location, and treat unmatched ones as candidate additional occurrences.

**Conflicting attributes.** Classic truth discovery applies:
- source-reliability-weighted voting (Dawid–Skene, 1979)
- copy-aware truth discovery (Dong, Berti-Equille & Srivastava, VLDB 2009, which models exactly the "sources that copy each other" problem)
- surveys (Li et al., "A Survey on Truth Discovery," arXiv 1505.02463)

Rule precedence (observed > inferred; specific > default) is a reasonable *prior* that a learned reliability model can refine.

**Dependency graphs.** Re-mint refs, union the edges, then deduplicate edges whose endpoints were merged. Watch for cycles introduced by inconsistent `provides`/`dependsOn` semantics across tools.

**Complexity.** Naive pairwise matching across k tools with n components each is O(k²n²). Blocking reduces it to near-linear in practice. n-way optimal matching is NP-hard in general, which is why BF-CBOM uses the RaQuN heuristic.\[3\] At realistic CBOM sizes (thousands of occurrences), runtime is not the bottleneck; label quality is.

**Correlated errors (the most important methodological point).** **[Analysis grounded in Verified tool relationships]**
- sonar-cryptography, cbomkit-lib, cbomkit-action and cbomkit share one detection engine.\[30\]
- cbomkit-theia enriches that engine's output.\[30\]
- An LLM worker may have been trained on the same public rule repos.

Agreement among correlated sources is **not** corroboration. Without modeling dependence, majority voting systematically inflates confidence in shared false positives (e.g., a test-only fixture flagged by every static tool) and shared false negatives (e.g., reflection-based `Cipher.getInstance(cfg.get("alg"))` that no static tool resolves).

### 6.2 Confidence scores

**What would it mean?** Pick one and state it:
- (a) P(asset truly present in the analyzed artifact)
- (b) P(attribute value correct | asset present)
- (c) P(asset reachable at runtime)

These differ, and conflating them is the most common flaw in such proposals. CycloneDX's `evidence.identity.confidence` most naturally carries (a). Use per-attribute properties or citations for (b). Do not claim (c) from static evidence alone. **[Analysis]**

**Candidate features:**
- number of *independent* tool families agreeing
- detection technique (AST with resolved symbols > SARIF pattern > filename/regex > LLM)
- `origin` (detected > inferred > enriched)
- field completeness (OID present, parameter set present)
- location class (test/vendor/generated paths)
- tool-specific priors estimated on labeled data

CycloneDX's own guidance gives technique-level ranges, e.g., filename matching at 0–0.1 and instrumentation at 0.3–0.8.\[52\] **[Verified for the guide's existence and examples]**

**Why scores are arbitrary without ground truth.** Any hand-assigned weight (0.9 for AST, 0.5 for LLM) is a *design opinion*. Without a labeled corpus you cannot estimate tool reliabilities, cannot check calibration, and cannot answer the reviewer's first question: "is 0.8 actually right 80% of the time?" The unsupervised alternative, Dawid–Skene-style latent-class estimation, needs *conditional independence of sources*, which §6.1 shows is violated.

**Calibration and evaluation.** Hold out labeled assets. Report:
- Brier score and log loss
- reliability diagrams and Expected Calibration Error (Guo et al., arXiv 1706.04599)
- selective-prediction curves (precision at confidence ≥ τ)

Recalibrate with isotonic or Platt scaling.

**Why not MYCIN-style certainty factors?** CF algebra lacks a consistent probabilistic interpretation except under strong independence assumptions. Heckerman (1986) and Horvitz & Heckerman (1986) showed that CF combination is equivalent to a probabilistic update only under restrictive conditions, and that it double-counts correlated evidence. That is exactly your correlated-tool scenario. FuzzyCLIPS (NRC Canada) implements CF propagation (e.g., CF_conclusion = CF_rule × min CF_facts), but it is a legacy fork tied to old CLIPS versions (6.04A/6.10d documentation).\[53\]\[54\]\[55\]\[56\]\[57\] I found no evidence of active maintenance. **Do not build on it.** **[Verified for FuzzyCLIPS CF semantics from its user guide; maintenance status: no evidence of activity found]**

**Dempster–Shafer** handles "don't know" mass but misbehaves under high conflict (Zadeh's example) and still assumes independent evidence. **Probabilistic soft logic** (Bach et al., arXiv 1505.04406) and Markov logic encode rule-like soft constraints with learned weights, the principled upgrade path for "rules + uncertainty."

### 6.3 Risk scores

**A CBOM lacks most of the inputs that quantum risk depends on [Analysis]:**

| Risk driver | In a CBOM? |
|---|---|
| Data sensitivity and secrecy lifetime (Mosca's X in X + Y > Z) | **No**\[58\]\[59\] |
| Migration time (Y) | **No** |
| Threat timeline (Z) | External estimate only |
| HNDL relevance (high for key establishment/encryption of long-lived data; low for signatures, whose forgery threat is prospective) | Partly (via `cryptoFunctions`/primitive) |
| Exposure (internet-facing vs. internal) | **No** |
| Reachability / dead code / test code | Weakly (paths; callstack rarely populated) |
| First-party vs. dependency code | Partly (via component graph) |
| Negotiated protocol parameters (TLS groups actually agreed) | **No** for static tools; only network sensors see it |

**Threat timeline evidence.** The Global Risk Institute / evolutionQ *Quantum Threat Timeline Report 2025* (Mosca & Piani, published March 9, 2026; 26 experts) puts a cryptographically relevant quantum computer at "quite possible (28–49%) within the next 10 years, and likely (51–70%) in the next 15."\[60\]\[61\]\[62\] **[Verified]** These are elicited expert opinions with wide spread, not measurements.

**Existing frameworks.** Commercial scores are opaque: IBM uses "CVSS metrics" for exploitability, Keyfactor a 0–10 combined score plus a 0–3 PQC priority, and AQtive Guard severity × occurrences.\[39\]\[42\]\[45\] There is **no standard quantum-risk scoring method**. CVSS fits poorly because it scores exploitability of a *flaw* in a product. "Uses RSA-2048 key transport" is not a flaw today; its impact depends on future adversary capability and on data lifetime, neither of which CVSS models. **[Analysis]**

**Blunt warning.** A single number like "risk 7.3" computed from a static CBOM is **authoritative-looking noise**. It will be gamed and misread, and reviewers will reject it. Defensible alternatives:
- a **categorical priority** (e.g., HNDL-critical / signature-migrate-by-deadline / Grover-adequate / broken-now / insufficient-info) derived by transparent rules
- *explicit external context facts* (data lifetime, exposure) supplied by the user, with "unknown" propagated rather than defaulted

### 6.4 PQC classification

**Taxonomy and mapping** **[Verified for listed standards' content where stated; Reported where flagged]**:

| Class | Examples | Authority / basis |
|---|---|---|
| **Q-vulnerable (Shor)**: key establishment | RSA-KEM/OAEP, (EC)DH, X25519/X448 | NIST IR 8547 (initial public draft, Nov 2024): 112-bit quantum-vulnerable algorithms deprecated after 2030, all disallowed after 2035 *(still a draft as of mid-2026, per secondary sources)*\[63\] |
| **Q-vulnerable (Shor)**: signatures | RSA-PSS/PKCS#1, ECDSA, EdDSA | Same; CNSA 2.0 software/firmware signing timelines\[64\] |
| **Grover-affected, adequate** | AES-256, SHA-384/512 | CNSA 2.0 mandates AES-256 and SHA-384/512\[64\]\[65\] |
| **Grover-affected, policy-dependent** | AES-128, SHA-256 | NIST generally treats AES-128 as adequate (category 1 baseline); CNSA 2.0 does not permit it for NSS. **Agencies disagree** |
| **Quantum-resistant, standardized** | ML-KEM (FIPS 203), ML-DSA (FIPS 204), SLH-DSA (FIPS 205), all final August 13, 2024; LMS/XMSS (SP 800-208) | NIST\[63\]\[64\] |
| **Quantum-resistant, pending standard** | FN-DSA (FIPS 206), HQC | The FIPS 206 draft was submitted for approval August 28, 2025. ArcQubit's NIST PQC Standards Guide (status as of September 30, 2026) found "no public draft on NIST's site and no Federal Register notice, so no comment period has opened," citing NIST's project page (updated August 5, 2026), which lists FIPS 206 as "in development." HQC was selected March 11, 2025 (NIST IR 8545). Dustin Moody's NIST deck "NIST PQC: The Road Ahead" lists "FIPS 207 – in 2026" as a next step; that is a NIST plan, not a published draft. Neither is in the CycloneDX 1.7 registry per `pqc-readiness` |
| **Hybrid** | X25519MLKEM768, SecP384r1MLKEM1024; composite ML-DSA+ECDSA | Quantum-resistant *if* the PQ component meets policy. CNSA 2.0 requires ML-KEM-1024, and an NSA FAQ (as quoted secondhand) says hybrid "will not be integrated into eventual deployable solutions." BSI/ANSSI *recommend* hybrid. NCSC UK treats it as interim\[64\]\[66\]\[67\] |
| **Classically broken/deprecated** | MD5, SHA-1 (signatures), DES/3DES, RC4, RSA-1024, ECB mode | SP 800-131A transitions. Rev. 2 (March 2019) is still in force. The Rev. 3 initial public draft (Barker & Roginsky) was published October 21, 2024, with comments closing December 4, 2024. NIST states "The final version is currently being reviewed internally" |
| **Insufficient information** | "RSA" with no key size; "AES" with no mode; TLS without negotiated group | Must be explicit; never default |

**Hard cases [Analysis]:**
- **Missing parameters.** "RSA" is Shor-vulnerable regardless of size, so missing size does not change the PQC class. It *does* change the classical-strength verdict. Classify along two independent axes (quantum class, classical adequacy), not one.
- **Hybrid naming.** `X25519MLKEM768` must be decomposed into two components plus a composition relation. Otherwise a tool reporting only "X25519" (because it saw the classical half) will be "corrected" toward vulnerable.
- **Configuration-dependent protocols.** A TLS 1.3 library *supporting* ML-KEM hybrids says nothing about what is *negotiated*. Static CBOMs can only report capability. Mark these as "capability-dependent; requires runtime/network evidence."
- **Agency disagreement.** Make the policy a *parameter*: a rule module per regime (NIST-general, CNSA 2.0, BSI TR-02102, ANSSI). Emit verdicts per regime rather than one global label. This is where CLIPS modules (or OPA bundles) are a natural fit.
- **Other agencies.** I did not re-verify the current text of the Canadian Centre for Cyber Security roadmap, NCSC UK milestones, BSI TR-02102 or ETSI hybrid specifications in this session. Encode them only from their primary documents. A secondary source reports BSI no longer recommends classical-only key agreement from end-2031.\[66\] **[Reported]**

**`nistQuantumSecurityLevel` limits.** It encodes a NIST *security strength category* (0–6, with 0 for quantum-vulnerable), not migration status, compliance, or HNDL relevance. It does not distinguish "Shor-broken" from "unknown." It is defined per algorithm, not per protocol configuration, and it has no slot for regime-specific verdicts (AES-128 is level 1 to NIST and non-compliant to CNSA 2.0). Tools that populate it may also apply different conventions. Validate rather than trust it. **[Analysis]**

**Novelty check.** CBOMkit already ships whitelist-based quantum-safe compliance with an "unknown" class and an OPA hook.\[30\]\[68\] Keyfactor computes a PQC priority.\[45\] PQC classification alone is **not** a contribution. Multi-regime, explainable classification with explicit "insufficient information" handling, evaluated against expert labels, is a small but real one.

## 7. Limitations and Challenges (candid)

**Technical**
1. Identity is ill-defined across granularities. Every downstream metric depends on a choice you must defend.
2. JSON Pointer–based citations are brittle under re-ordering, and per-attribute confidence has no native home.
3. Enricher-injected defaults masquerade as observations unless `origin` is preserved, and most tools do not emit it.
4. Static analysis cannot see negotiated parameters, reflection/config-driven algorithm names, or native/binary crypto. Merging static tools does not fix shared blind spots.

**Methodological**
5. No ground truth exists for CBOM inventory. BF-CBOM's own authors list it as future work.\[3\] Without it, merged-vs-single comparisons are relative only.
6. Correlated sources invalidate naive voting and Dawid–Skene independence assumptions.
7. Confidence, risk and "correctness" can be circular if the same rules both produce and judge outputs.

**Standards-related**
8. Version churn: 1.6 vs. 1.7 field deprecations, a registry that changes independently (with frequent patch-level additions), and HQC/FN-DSA not yet in it.\[13\]\[14\]\[36\]
9. Policy churn: IR 8547 is still a draft, FIPS 206 timing is uncertain, and agencies disagree on hybrids and AES-128.\[69\]\[70\]
10. Regulatory pull is real but unsettled. Secondary sources report a June 2026 U.S. Executive Order (EO 14412) directing CISA/NIST to publish CBOM minimum elements within 270 days.\[71\] If that happens, your output schema targets may move. **[Reported; not verified against the Federal Register]**

**Tooling**
11. CryptobomForge is pre-standard format and possibly unmaintained. cdxgen's crypto language coverage is inconsistently documented.\[34\]\[72\]\[73\]
12. CLIPS lacks JSON, uncertainty and a modern testing ecosystem. It is maintained effectively by one person, and team familiarity is low.

**Evaluation**
13. Crypto-misuse benchmarks (CryptoAPI-Bench: 181 Java test cases across 18 misuse categories; CamBench; MASC) label *misuses*, not *inventories*, and have documented ground-truth errors. FPPredictor's authors found cases labeled non-vulnerable that were in fact insecure. They are synthetic and Java-centric.\[74\]\[75\]

**Novelty**
14. Merge = engineering. Multi-tool comparison = BF-CBOM. Aggregation + scoring = IBM/SandboxAQ/Keyfactor. PQC whitelist = CBOMkit. Rule-based expert system = 1980s technology. The defensible novelty is **calibrated, provenance-preserving, correlation-aware reconciliation with an evaluated ground truth**, not "an expert system."

**Claims that would not survive peer review:**
- "Our risk score quantifies quantum risk."
- "Agreement between tools increases confidence."
- "The merged CBOM is more accurate." (without a labeled corpus)
- "CLIPS enables explainable AI for CBOM." (without a user study or a concrete explanation-quality metric)

## 8. Alternatives and comparison

| Option | Strengths | Weaknesses for this task | Verdict |
|---|---|---|---|
| **CLIPS 6.4.2** (+ CLIPSpy/C API) | Mature Rete, modules, `logical` TMS, traces, public domain, deterministic under fixed assertion order | No JSON, no uncertainty, niche skills, single maintainer, weak test tooling | Fine for stages 4–5 only |
| **Drools** | Industrial Rete (Phreak), decision tables, JVM ecosystem | JVM weight (JDK 17 minimum for the 10.x series). Actively released: Apache KIE (Incubating) 10.2.0, which includes Drools, was announced April 29, 2026 and "fixed critical issues with activation groups and agenda groups" | Better if the team is JVM-based |
| **Datalog / Soufflé** | Set semantics give determinism by construction; recursion over dependency graphs; provenance (proof trees) | No native non-monotonic TMS (stratified negation instead), no uncertainty | **Strong alternative** for graph rules and explanation |
| **OPA/Rego** | Policy-as-code over JSON natively; CBOMkit already integrates OPA | Not designed for iterative inference/TMS | **Best fit for PQC/compliance policy modules**\[30\] |
| **experta** (pyknow fork) / **durable_rules** | Python-native | Secondary sources describe pyknow as stalled with experta as the community fork; durable_rules depends on Redis for state; I could not verify recent releases for either | Avoid for a research artifact needing longevity\[76\]\[77\] |
| **Plain Python + SQL/DuckDB** | Fast to build, testable, JSON-native, ubiquitous | Explanation must be hand-built | **Simplest baseline; you must beat it** |
| **Dempster–Shafer / Bayesian truth discovery / Dawid–Skene / PSL / MLN** | Principled uncertainty; learnable source reliability | Need labels or independence assumptions; less legible | **Required for the confidence contribution** |
| **ML/LLM-assisted matching** | Handles naming chaos | Non-deterministic, uncalibrated, hard to audit | Use only as a feature generator with a deterministic decision layer |

Prolog and graph databases are viable but offer no clear advantage here.

**Blunt verdict on CLIPS [Analysis].** CLIPS is *not* the bottleneck. Labels, normalization tables and identity definitions are. CLIPS would beat a Python `if`-ladder on explainability, `logical` retraction and modular policy regimes. It loses on I/O, uncertainty, testability and team cost. If the research claim is explainability, a Soufflé/OPA design gives comparable traceability with JSON-native tooling. Choose CLIPS only if you will exploit `logical` support and rule traces as *evaluated* features, for example by comparing explanation completeness against a Python baseline.

## 9. Evaluation plan

**Ground truth:**
1. **Synthetic corpus:** generated projects (Java/Python/Go, matching tool coverage) with known crypto calls, including configuration-driven, reflective, test-only, dead-code, wrapper-library, and hybrid-TLS configurations. Labels come by construction.
2. **Hand-annotated real repositories:** 20–40 OSS repositories, dual-annotated, with inter-annotator agreement (Cohen's κ) reported. Use the schema "asset type × parameters × location × reachable?"
3. **Repurposed misuse benchmarks:** CryptoAPI-Bench and CamBench, re-labeled for *inventory*. Acknowledge that they are Java-only, synthetic and misuse-oriented, with known labeling errors.
4. **Artifact corpus:** container images with known certificates and keys for theia-style tools.

**Metrics:**
- Asset-level precision/recall/F1 for each single tool vs. naive union vs. majority vote vs. the reconciler, at both type and occurrence level.
- Attribute-level accuracy (family, parameter set, mode, padding, curve).
- Confidence calibration (Brier, ECE, reliability diagrams).
- PQC-class agreement with two expert annotators (κ), per policy regime.
- Explanation completeness (fraction of output fields with a citation and a rule trace).

**Ablations:** without normalization, without origin-awareness, without correlation modeling (treat sonar and theia as independent), without rule precedence (pure voting), and without the LLM worker.

**Threats to validity:**
- Synthetic corpora favor pattern-matching tools.
- Annotators share tool-induced bias if they see tool output first, so annotate blind.
- Tool versions drift; pin and archive containers.
- Construct validity: "risk" has no ground truth, so evaluate only the classification and the rule transparency, not the risk "accuracy."

## 10. Feasibility and recommended scope

**One-month prototype (realistic):**
- Ingest 1.6/1.7 (plus a CryptobomForge legacy adapter, or explicitly drop it).
- Registry-based normalization for roughly 30 common families.
- Two-level identity with blocking.
- About 20 CLIPS conflict-resolution rules using `logical` support.
- A multi-regime PQC classifier (NIST-general plus CNSA 2.0).
- 1.7 output with `citations`, `annotations` and identity evidence, validated against the 1.7.2 schema.
- A 10–15 repository hand-labeled pilot to report *preliminary* asset-level P/R.
- Confidence as a documented heuristic, explicitly labeled uncalibrated.
- No risk score; a categorical priority with user-supplied context only.

**Longer research effort (3–9 months):** a labeled corpus at scale, correlation-aware truth discovery with calibrated confidence, cross-language coverage, network/runtime sensors for negotiated parameters, user study of explanations, and a comparison against Soufflé/OPA and plain-Python baselines.

**Biggest risks:**
1. No ground truth, so no defensible claim.
2. Identity definition disputes.
3. Tool churn (1.7 upgrades in flight).
4. Scope creep into risk scoring.
5. Novelty being dismissed as engineering.

**Recommended reframing:** *"Correlation-aware, provenance-preserving reconciliation of CycloneDX CBOMs: a ground-truth study."* Contributions:
- (i) a labeled CBOM inventory corpus (likely the most cited artifact you could produce)
- (ii) an explicit two-level identity model and normalization mapping to the 1.7 registry
- (iii) a reconciler whose every field carries a 1.7 citation and a rule trace
- (iv) calibrated confidence that measurably beats voting
- (v) multi-regime PQC classification with explicit insufficient-information handling

Consider collaborating with the BF-CBOM authors. Their stated future work, ground-truth datasets, is your prerequisite.\[3\]

## 11. Open research questions

1. What is the right unit of identity for crypto assets across static, artifact and network evidence, and can it be standardized (e.g., proposed to the CycloneDX working group)?
2. How much do CBOM generators' errors correlate, and can source dependence be estimated without labels?
3. Does reconciliation measurably beat the best single tool, or is a union with deduplication sufficient?
4. Can confidence be calibrated across projects and languages, or is it corpus-specific?
5. Do rule traces and citations improve analyst decisions (time-to-triage, error rate) in a user study?
6. How should regime-dependent PQC verdicts be represented natively in CycloneDX: a property-taxonomy namespace, or a spec proposal?

## Sources

1. [Bugs Framework (BF): Formalizing Cybersecurity Weaknesses and Vulnerabilities](https://www.nist.gov/publications/bugs-framework-bf-formalizing-cybersecurity-weaknesses-and-vulnerabilities)
2. [Cryptography Classes in Bugs Framework (BF): Encryption Bugs (ENC), Verification Bugs (VRF), and Key Management Bugs (KMN)](https://www.nist.gov/publications/cryptography-classes-bugs-framework-bf-encryption-bugs-enc-verification-bugs-vrf-and)
3. <https://romanboegli.ch/assets/pdf/Boegli_2026_BFCBOM_ICPC.pdf>
4. [GitHub - SEG-UNIBE/BF-CBOM: BF-CBOM: Your Best Friend for Generating, Understanding, and Comparing Cryptography Bills of Material (CBOMs)](https://github.com/SEG-UNIBE/BF-CBOM)
5. [BF-CBOM: Benchmarking Framework for Cryptography Bill of Material Generator Tools](https://zenodo.org/records/17220030)
6. [BF-CBOM: Your Best Friend for Generating, Understanding, and Comparing Cryptography Bill of Material (CBOMs)](https://zenodo.org/records/17853399)
7. [BF-CBOM: Benchmarking Framework for Cryptography Bill of Material Generator Tools](https://zenodo.org/records/17215778)
8. [academia](https://romanboegli.ch/academia/)
9. [What Is CycloneDX?](https://www.encryptionconsulting.com/education-center/what-is-cyclonedx/)
10. [CycloneDX Newsroom](https://cyclonedx.org/news/)
11. [CBOM Examples: Annotated CycloneDX 1.7 JSON for a TLS Endpoint, an Application and an HSM Key](https://quantumsentinel.uk/cbom-examples-cyclonedx-json/)
12. [cyclonedx.v1\_7 - CycloneDX/specification - protodoc.io](https://protodoc.io/CycloneDX/specification/cyclonedx.v1_7)
13. [\[EPIC\] Upgrade CBOM output from CycloneDX 1.6 to 1.7 · Issue #505 · cbomkit/sonar-cryptography · GitHub](https://github.com/cbomkit/sonar-cryptography/issues/505)
14. [Cryptography Registry](https://cyclonedx.org/registry/cryptography/)
15. [Releases · CycloneDX/specification · GitHub](https://github.com/CycloneDX/specification/releases)
16. [cdxgen-docs/cdxgen-for-bots.jsonl · CycloneDX/cdx-docs at e4783f7babbbd5414fcdc0a6832828535739b88a](https://huggingface.co/datasets/CycloneDX/cdx-docs/blame/e4783f7babbbd5414fcdc0a6832828535739b88a/cdxgen-docs/cdxgen-for-bots.jsonl)
17. [cyclonedx.model.component\_evidence — CycloneDX Python Library 11.0.0 documentation](https://cyclonedx-python-library.readthedocs.io/en/stable/autoapi/cyclonedx/model/component_evidence/)
18. [CycloneDX v1.7 JSON Reference](https://cyclonedx.org/docs/1.7/json/)
19. [CycloneDX v1.7 Protobuf Reference](https://cyclonedx.org/docs/1.7/proto/)
20. [Prepend parent bom-refs in hierarchical merge · Issue #385 · Festo-se/cyclonedx-editor-validator · GitHub](https://github.com/Festo-se/cyclonedx-editor-validator/issues/385)
21. [CBOM Lens Emits the CycloneDX 1.7 Crypto Registry - OmniTrust](https://omnitrust.com/blog/cbom-lens-cyclonedx-1-7/)
22. [CLIPS 6.4.2 Released](https://groups.google.com/g/CLIPSESG/c/ZYOlXhv_sp4)
23. [CLIPS Rule Based Programming Language - Browse /CLIPS/6.4.2 at SourceForge.net](https://sourceforge.net/projects/clipsrules/files/CLIPS/6.4.2/)
24. [CLIPS](https://en.wikipedia.org/wiki/CLIPS)
25. [CLIPSESG - Google Groups](https://groups.google.com/g/CLIPSESG)
26. [CLIPS: A Tool for Building Expert Systems](https://www.clipsrules.net/)
27. [CLIPS Rule Based Programming Language download](https://sourceforge.net/projects/clipsrules/)
28. [@cyclonedx/cdxgen - JSR](https://jsr.io/@cyclonedx/cdxgen)
29. [cbomkit-theia command - github.com/cbomkit/cbomkit-theia - Go Packages](https://pkg.go.dev/github.com/cbomkit/cbomkit-theia)
30. [PQCA CBOMkit Architecture – Post-Quantum Cryptography Alliance](https://pqca.org/blog/2026/pqca-cbomkit-architecture/)
31. [feat: optional CBOM generation via cdxgen, plus CBOM pipeline recipes by aurangzaib048 · Pull Request #300 · sbomify/sbomify-action · GitHub](https://github.com/sbomify/sbomify-action/pull/300)
32. [cdxgen/cdxgen: Creates CycloneDX Bill of Materials (BOM) for your projects from source and container images. Supports many languages and package managers. Integrate in your CI/CD pipeline with automatic submission to Dependency Track server - Codeberg.org](https://codeberg.org/cdxgen/cdxgen)
33. [BOM Types and Formats](https://deepwiki.com/cdxgen/cdxgen/2.1-bom-types-and-formats)
34. [GitHub - Santandersecurityresearch/cryptobom-forge: Tools and utilities needed to parse GitHub Multi-Repository Variant Analysis output · GitHub](https://github.com/Santandersecurityresearch/cryptobom-forge)
35. [Cryptoscope: Analyzing cryptographic usages in modern software](https://arxiv.org/pdf/2503.19531)
36. [feat(cbom): CycloneDX 1.7 CBOM by default, --cbom-version 1.6 kept byte-identical by aclater · Pull Request #163 · aclater/pqc-readiness · GitHub](https://github.com/aclater/pqc-readiness/pull/163)
37. [PQCTools/CADI.md at main · Santandersecurityresearch/PQCTools · GitHub](https://github.com/Santandersecurityresearch/PQCTools/blob/main/CADI.md)
38. [Taking Control of Cryptographic Risk in the Quantum Era: Introducing IBM Guardium Cryptography Manager](https://www.ibm.com/new/announcements/control-of-cryptographic-risk-in-the-quantum-era-introducing-ibm-guardium-cryptography-manager)
39. <https://webobjects2.cdw.com/is/content/CDW/cdw/on-domain-cdw/brands/ibm/ibm-guardium-cryptography-manager-data-sheet.pdf>
40. <https://www.ibm.com/downloads/documents/fr-fr/137a1e22185bac92>
41. [CBOM - AQtive Guard User Guide](https://docs.aqtiveguard.com/data-sources/cbom/)
42. [Getting started - AQtive Guard User Guide](https://docs.aqtiveguard.com/getting-started/)
43. [Cryptographic Alert Fatigue Solved: Context-Aware Security Operations](https://www.aqtiveguard.com/blogs/enriched-perspectives-for-enhanced-cryptographic-security-operations)
44. [AgileSec 3.4 Release Notes](https://docs.keyfactor.com/agilesec/latest/agilesec-3-4-release-notes)
45. [Cryptographic Data Fields Reference](https://docs.keyfactor.com/agilesec/latest/cryptographic-data-fields-reference)
46. [GitHub - CycloneDX/cyclonedx-cli: CycloneDX CLI tool for SBOM analysis, merging, diffs and format conversions. · GitHub](https://github.com/CycloneDX/cyclonedx-cli)
47. [Documention: Hierarchical merge metadata requirement · Issue #310 · CycloneDX/cyclonedx-cli · GitHub](https://github.com/CycloneDX/cyclonedx-cli/issues/310)
48. [Hierarchical merge misplaces \<properties\> nodes of toplevel components · Issue #304 · CycloneDX/cyclonedx-cli · GitHub](https://github.com/CycloneDX/cyclonedx-cli/issues/304)
49. [Merging vcpkg created sboms · Issue #377 · CycloneDX/cyclonedx-cli · GitHub](https://github.com/CycloneDX/cyclonedx-cli/issues/377)
50. [Tools for properly merging sboms? · cdxgen/cdxgen · Discussion #1835 · GitHub](https://github.com/cdxgen/cdxgen/discussions/1835)
51. [RFC: merging multiple CycloneDX documents · CycloneDX/specification · Discussion #320 · GitHub](https://github.com/CycloneDX/specification/discussions/320)
52. [OWASP CycloneDX Authoritative Guide to SBOM](https://cyclonedx.org/guides/sbom/evidence/)
53. [(somewhat) Quick introduction to FuzzyCLIPS](http://alumni.cs.ucr.edu/~vladimir/cs171/quickfuzzy.pdf)
54. [(PDF) A fuzzy knowledge-based system for intelligent retrieval](https://www.researchgate.net/publication/3336292_A_fuzzy_knowledge-based_system_for_intelligent_retrieval)
55. [FuzzyCLIPS Version 6.04A User's Guide](https://aikernel.org/files/FuzzyClips/fzdocs.pdf)
56. [FuzzyCLIPS10d Fuzzy CLIPS Users Guide](https://usermanual.wiki/Document/FuzzyCLIPSusersguide.2105868129/html)
57. [A Framework for Comparing Uncertain Inference Systems to Probability](https://arxiv.org/pdf/1304.3430)
58. [Quantum Threat Timeline - Global Risk Institute](https://globalriskinstitute.org/publication/quantum-threat-timeline/)
59. [Michele Mosca](https://en.wikipedia.org/wiki/Michele_Mosca)
60. [Quantum Threat Timeline Report 2025 - Global Risk Institute](https://globalriskinstitute.org/publication/quantum-threat-timeline-report-2025b/)
61. [Quantum Threat Timeline Report 2025: Record Predictions, But Can the Survey Keep Up? - Quantum Security](https://quantum.security/post-quantum/quantum-threat-timeline-report-2025-record-predictions-but-can-the-survey-keep-up/)
62. [Quantum Threat Timeline Research Report 2025 - Publication](https://www.evolutionq.com/publications/quantum-threat-timeline-research-report-2025)
63. [Post-Quantum Cryptography Standards and Audit Index: algorithms, dates, standards bodies, auditors](https://pqaudit.org/)
64. [CNSA 2.0: U.S. National Security Systems: dates and requirements](https://pqaudit.org/timelines/cnsa-2-0/)
65. [Exploring CNSA 2.0: The Core Algorithms for Next-Gen Security](https://www.encryptionconsulting.com/exploring-cnsa-2-0-the-core-algorithms-for-next-gen-security/)
66. [Hybrid vs. Pure PQC: Why Global Requirements Conflict](https://postquantum.com/pqc-migration-timelines/hybrid-vs-pure-pqc-compliance/)
67. [A Scenario-Based Evaluation of CRQC+AI Vulnerability Spectrum for TLS 1.3 Cryptographic Dependencies](https://arxiv.org/pdf/2608.23785)
68. [CBOMkit Now Supports Go (Golang)](https://pqca.org/blog/2026/cbomkit-now-supports-go/)
69. [How To Prepare For Quantum Computing, The Complete Readiness Guide](https://quantumzeitgeist.com/how-to-prepare-for-quantum-computing/)
70. [NIST PQC Standards Guide: FIPS 203, 204, 205](https://www.arcqubit.ai/blog/nist-pqc-standards-guide)
71. [What Is a Cryptographic Bill of Materials (CBOM)? - Cycode](https://cycode.com/blog/cryptographic-bill-of-materials-cbom/)
72. [OWASP cdxgen](https://owasp.github.io/www-project-cdxgen/)
73. [ci: generate the CBOM via the action (GENERATE\_CBOM) by aurangzaib048 · Pull Request #1052 · sbomify/sbomify · GitHub](https://github.com/sbomify/sbomify/pull/1052)
74. [FPPredictor](https://arxiv.org/pdf/2603.10558)
75. [CryptoAPI-Bench](https://www.alphaxiv.org/benchmarks/university-of-wisconsin-madison/cryptoapi-bench)
76. [7 Best Python Rule Engines Compared (2026 Guide)](https://www.nected.ai/us/blog-us/python-rule-engines-automate-and-enforce-with-python)
77. [pypi.org](https://pypi.org/project/durable-rules/0.33.74/)
