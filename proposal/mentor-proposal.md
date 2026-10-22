Project 2: Measuring Cryptographic Readiness for the Post-Quantum Era


**Motivation**
Future quantum computers may be able to break widely used public-key methods such as RSA and elliptic-curve cryptography. Organizations will need to replace these methods, but first they must find where cryptography is used in their software.

This is harder than it sounds. Cryptography may be used directly in source code, selected through configuration, or hidden inside libraries and dependencies. Existing tools can miss real uses or report code that is present but never used. They may produce a Cryptography Bill of Materials (CBOM): a machine-readable list of cryptographic algorithms, keys, certificates, and protocols found in a system.

**Project Goal**
This applied project has four parts:

1. Learn the foundations. Develop a practical understanding of current cryptography, how quantum computers change the risk, and the new NIST standards ML-KEM, ML-DSA, and SLH-DSA. No prior knowledge of post-quantum cryptography is assumed.

2. Study existing tools. Install and use open-source tools such as CBOMkit, cdxgen, and CryptoBOM-Forge. Determine what each tool examines, what it reports, and whether its results are useful for planning a post-quantum migration.

3. Build a fair benchmark. Use two groups of software. First, create small sample programs in which the cryptography is known in advance. Include straightforward uses, cryptography hidden behind helper functions or configuration, unused code, dependencies, and both traditional and post-quantum algorithms. Second, test pinned versions of several existing open-source projects that use cryptography in different ways and programming languages. Compare how often tools find the correct items, how often they report incorrect items, their run time, and how easy they are to install and understand.

4. Make an improvement. Use the benchmark results to select one important weakness and build a focused improvement or new tool.

**Deliverables and Evaluation**
The student will produce:

- a short review of post-quantum migration and available discovery tools;

- a reusable benchmark containing controlled examples, selected open-source projects, and known expected results;

- a repeatable comparison of the selected tools;

- a working prototype that improves one measured weakness; and

- clear installation, usage, and design documentation.

The project will be successful if another person can repeat the comparison and the prototype shows a clear benefit over at least one existing tool. The goal is not to claim that one tool detects everything, but to explain what each tool can and cannot reliably tell an organization.

**Suggested Learning Path**

1. Plain-language introduction: NIST, What Is Post-Quantum Cryptography?
https://www.nist.gov/cybersecurity-and-privacy/what-post-quantum-cryptography

2. Introductory reading: UK National Cyber Security Centre, Preparing for Quantum-Safe Cryptography
https://www.ncsc.gov.uk/paper/preparing-for-quantum-safe-cryptography

3. Standards and migration overview: NIST Post-Quantum Cryptography
https://www.nist.gov/pqcrypto

4. Hands-on experimentation: Open Quantum Safe, including liboqs and test integrations https://openquantumsafe.org/

5. CBOM concept and examples: CycloneDX Cryptography Bill of Materials
https://cyclonedx.org/capabilities/cbom/

6. Advanced context on AI-assisted cryptanalysis: Fluri et al., CryptanalysisBench: Can LLMs do Cryptanalysis?
https://arxiv.org/abs/2607.18538
This paper provides broader motivation for maintaining accurate cryptographic inventories by showing emerging AI capabilities for analyzing cryptographic primitives. 

**Starting Tools**
1. CBOMkit: A toolset focused specifically on cryptographic inventories. It can scan supported Java, Python, and Go source code, produce and display CBOMs, and apply basic checks for quantum-vulnerable algorithms.

2. cdxgen: A command-line tool that generates software inventories in CycloneDX format. Its CBOM features can find some cryptographic algorithms in source code, certificates, and Java key stores, making it a convenient first tool to run.

3. CryptoBOM-Forge is an open-source tool that analyzes software code to identify cryptographic algorithms, libraries, and implementations and generates a Cryptographic Bill of Materials (CBOM) describing the cryptographic assets used in a software project.

Other tools may be added later if the first benchmark shows a need for a different analysis approach.

