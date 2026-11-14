# 2CBOM report (strict profile)

- Serial number: `urn:uuid:fbc4bc57-c381-5966-a9c3-908e9222ffc5`
- Digest: `sha256:369d9c33fd90c1365d0a3c363e18b7e5b6a9317ce15a1b9ccf7d24eb7d52f5be`
- Generated: 2026-11-14T03:46:33Z by twocbom 1.0.0
- Conformant: **no** (30 errors, 60 warnings, 24 infos)
- Assets: 79 | Components: 16 | Relations: 61

## Sources

| id | tool | format | spec | trust | assets | digest |
|---|---|---|---|---|---|---|
| src-4c14dc79cf35 | cbomkit  | cyclonedx-json | 1.6 | 80 | 137 | `sha256:4c14dc79cf35a855…` |
| src-83f9a6b9b0ac | dosai-linux-amd64  | cyclonedx-json | 1.7 | 50 | 12 | `sha256:83f9a6b9b0ac1dc4…` |
| src-acc93be4b00d | cyclonedx-python-lib 4.2.2 | ibm-cbom-json | 1.4-cbom-1.0 | 50 | 22 | `sha256:acc93be4b00dbe6c…` |

## Post-quantum readiness

| quantum class | assets |  | migration priority | assets |
|---|---|---|---|---|
| classically-weak | 8 |  | immediate | 8 |
| quantum-vulnerable | 17 |  | high | 10 |
| unknown | 20 |  | medium | 7 |
| quantum-safe | 34 |  | low | 28 |
|  |  |  | none | 6 |
|  |  |  | unknown | 20 |

## Assets

| id | type | quantum class | NIST IR 8547 | NIST cat. | classical bits | CNSA 2.0 | priority | sources | evidence |
|---|---|---|---|---|---|---|---|---|---|
| `2cbom:alg:aes-128` | algorithm | quantum-safe | acceptable | 1 | 128 | no | low | 1 | 6 |
| `2cbom:alg:aes-128-cbc-pkcs5` | algorithm | quantum-safe | acceptable | 1 | 128 | no | low | 1 | 2 |
| `2cbom:alg:aes-128-cbc-pkcs7` | algorithm | quantum-safe | acceptable | 1 | 128 | no | low | 1 | 5 |
| `2cbom:alg:aes-128-ctr` | algorithm | quantum-safe | acceptable | 1 | 128 | no | low | 1 | 2 |
| `2cbom:alg:aes-128-ecb-pkcs5` | algorithm | quantum-safe | acceptable | 1 | 128 | no | low | 1 | 1 |
| `2cbom:alg:aes-128-ecb-pkcs7` | algorithm | quantum-safe | acceptable | 1 | 128 | no | low | 3 | 8 |
| `2cbom:alg:aes-128-gcm` | algorithm | quantum-safe | acceptable | 1 | 128 | no | low | 1 | 3 |
| `2cbom:alg:aes-256` | algorithm | quantum-safe | acceptable | 5 | 256 | yes | none | 1 | 2 |
| `2cbom:alg:aes-256-cbc-pkcs7` | algorithm | quantum-safe | acceptable | 5 | 256 | yes | none | 2 | 6 |
| `2cbom:alg:aes-gcm` | algorithm | quantum-safe | acceptable | - | - | no | low | 1 | 5 |
| `2cbom:alg:blake2b` | algorithm | quantum-safe | acceptable | 5 | 256 | no | low | 2 | 6 |
| `2cbom:alg:chacha20` | algorithm | quantum-safe | acceptable | 5 | 256 | no | low | 1 | 3 |
| `2cbom:alg:chacha20-poly1305` | algorithm | quantum-safe | acceptable | 5 | 256 | no | low | 1 | 1 |
| `2cbom:alg:drbg` | algorithm | quantum-safe | acceptable | - | - | no | low | 1 | 7 |
| `2cbom:alg:ec` | algorithm | quantum-vulnerable | disallowed-after-2035 | - | - | no | high | 1 | 3 |
| `2cbom:alg:ec-p256` | algorithm | quantum-vulnerable | disallowed-after-2035 | 0 | 128 | no | high | 2 | 19 |
| `2cbom:alg:ecdh` | algorithm | quantum-vulnerable | disallowed-after-2035 | - | - | no | high | 1 | 3 |
| `2cbom:alg:ecdh-p256` | algorithm | quantum-vulnerable | disallowed-after-2035 | 0 | 128 | no | high | 1 | 2 |
| `2cbom:alg:ecdsa` | algorithm | quantum-vulnerable | disallowed-after-2035 | - | - | no | medium | 1 | 2 |
| `2cbom:alg:ecdsa-p256` | algorithm | quantum-vulnerable | disallowed-after-2035 | 0 | 128 | no | medium | 1 | 2 |
| `2cbom:alg:ecdsa-sha2-256` | algorithm | quantum-vulnerable | disallowed-after-2035 | - | - | no | medium | 1 | 2 |
| `2cbom:alg:ed25519` | algorithm | quantum-vulnerable | disallowed-after-2035 | 0 | 128 | no | medium | 1 | 3 |
| `2cbom:alg:hkdf` | algorithm | quantum-safe | acceptable | - | - | no | low | 2 | 22 |
| `2cbom:alg:hmac-sha2-256` | algorithm | quantum-safe | acceptable | 2 | 128 | no | low | 1 | 4 |
| `2cbom:alg:md5` | algorithm | classically-weak | disallowed | 0 | 0 | no | immediate | 3 | 9 |
| `2cbom:alg:ml-dsa-65` | algorithm | quantum-safe | acceptable | 3 | 192 | no | low | 1 | 3 |
| `2cbom:alg:ml-kem-768` | algorithm | quantum-safe | acceptable | 3 | 192 | no | low | 1 | 2 |
| `2cbom:alg:pbkdf2` | algorithm | quantum-safe | acceptable | - | - | no | low | 2 | 2 |
| `2cbom:alg:pbkdf2-sha2-256` | algorithm | quantum-safe | acceptable | 2 | 128 | no | low | 1 | 2 |
| `2cbom:alg:poly1305` | algorithm | quantum-safe | acceptable | 1 | 128 | no | low | 1 | 3 |
| `2cbom:alg:rsa` | algorithm | quantum-vulnerable | disallowed-after-2035 | - | - | no | high | 3 | 8 |
| `2cbom:alg:rsa-15-pkcs1v15` | algorithm | quantum-vulnerable | disallowed-after-2035 | - | - | no | medium | 1 | 2 |
| `2cbom:alg:rsa-15-pkcs1v15-sha2-256` | algorithm | quantum-vulnerable | disallowed-after-2035 | - | - | no | medium | 1 | 2 |
| `2cbom:alg:rsa-2048` | algorithm | quantum-vulnerable | deprecated-after-2030 | 0 | 112 | no | high | 1 | 1 |
| `2cbom:alg:rsa-2048-oaep` | algorithm | quantum-vulnerable | deprecated-after-2030 | 0 | 112 | no | high | 1 | 2 |
| `2cbom:alg:rsa-3072-oaep` | algorithm | quantum-vulnerable | disallowed-after-2035 | 0 | 128 | no | high | 2 | 6 |
| `2cbom:alg:rsa-3072-pkcs1v15` | algorithm | quantum-vulnerable | disallowed-after-2035 | 0 | 128 | no | high | 1 | 5 |
| `2cbom:alg:rsa-pss` | algorithm | quantum-vulnerable | disallowed-after-2035 | - | - | no | medium | 1 | 2 |
| `2cbom:alg:scrypt` | algorithm | quantum-safe | acceptable | 2 | 128 | no | low | 1 | 10 |
| `2cbom:alg:sha1` | algorithm | classically-weak | disallowed | 0 | 80 | no | immediate | 3 | 9 |
| `2cbom:alg:sha2-256` | algorithm | quantum-safe | acceptable | 2 | 128 | no | low | 3 | 79 |
| `2cbom:alg:sha2-512` | algorithm | quantum-safe | acceptable | 5 | 256 | yes | none | 3 | 12 |
| `2cbom:alg:sha3-256` | algorithm | quantum-safe | acceptable | 2 | 128 | no | low | 1 | 2 |
| `2cbom:alg:shake` | algorithm | quantum-safe | acceptable | - | - | no | low | 1 | 5 |
| `2cbom:alg:shake-128` | algorithm | quantum-safe | acceptable | 2 | 128 | no | low | 2 | 2 |
| `2cbom:alg:tdea-64` | algorithm | classically-weak | disallowed | - | - | no | immediate | 1 | 2 |
| `2cbom:alg:tdea-64-cbc-pkcs5` | algorithm | classically-weak | disallowed | - | - | no | immediate | 1 | 2 |
| `2cbom:alg:tdea-64-cbc-pkcs7` | algorithm | classically-weak | disallowed | - | - | no | immediate | 1 | 1 |
| `2cbom:alg:unresolved:36b8de141ee4d423d533` | algorithm | unknown | unknown | - | - | no | unknown | 1 | 1 |
| `2cbom:alg:unresolved:7550960cd83390a0618a` | algorithm | unknown | unknown | - | - | no | unknown | 1 | 8 |
| `2cbom:alg:x25519` | algorithm | quantum-vulnerable | disallowed-after-2035 | 0 | 128 | no | high | 2 | 5 |
| `2cbom:key:initialization-vector:id:2a799bbf0b948fe612e567d5` | related-crypto-material | unknown | unknown | - | - | no | unknown | 1 | 1 |
| `2cbom:key:initialization-vector:loc:90e558045ca48164b2f20104` | related-crypto-material | unknown | unknown | - | - | no | unknown | 1 | 2 |
| `2cbom:key:initialization-vector:loc:9d756f230770f865a4429015` | related-crypto-material | unknown | unknown | - | - | no | unknown | 1 | 4 |
| `2cbom:key:initialization-vector:loc:ed209f7944a3ef3b4e3e5dc1` | related-crypto-material | unknown | unknown | - | - | no | unknown | 1 | 2 |
| `2cbom:key:password:loc:8d3dabda7bfab1f1705c9951` | related-crypto-material | quantum-safe | acceptable | 3 | 208 | no | low | 1 | 1 |
| `2cbom:key:private-key:loc:476393f871c4d004cf9291d0` | related-crypto-material | unknown | unknown | - | - | no | unknown | 1 | 10 |
| `2cbom:key:salt:loc:0ae7c1325801bde23c851467` | related-crypto-material | unknown | unknown | - | - | no | unknown | 1 | 1 |
| `2cbom:key:salt:loc:110590fd3066210bf8b1fae4` | related-crypto-material | unknown | unknown | - | - | no | unknown | 1 | 1 |
| `2cbom:key:salt:loc:40dc9409ccef01b91deb3423` | related-crypto-material | unknown | unknown | - | - | no | unknown | 1 | 1 |
| `2cbom:key:salt:loc:573558e0613905e1ffc5c7e7` | related-crypto-material | unknown | unknown | - | - | no | unknown | 1 | 1 |
| `2cbom:key:salt:loc:67512275dac23f8e292d52c1` | related-crypto-material | unknown | unknown | - | - | no | unknown | 1 | 1 |
| `2cbom:key:salt:loc:8d3dabda7bfab1f1705c9951` | related-crypto-material | unknown | unknown | - | - | no | unknown | 1 | 1 |
| `2cbom:key:salt:loc:9d756f230770f865a4429015` | related-crypto-material | unknown | unknown | - | - | no | unknown | 1 | 2 |
| `2cbom:key:salt:loc:d2781155d265fa1673ac51c9` | related-crypto-material | unknown | unknown | - | - | no | unknown | 1 | 1 |
| `2cbom:key:salt:loc:d700b5f20159d6052be67868` | related-crypto-material | unknown | unknown | - | - | no | unknown | 1 | 1 |
| `2cbom:key:secret-key:id:b81263d711b5a126be273ff6` | related-crypto-material | unknown | unknown | - | - | no | unknown | 1 | 1 |
| `2cbom:key:secret-key:id:f247840448065f80b95fb679` | related-crypto-material | unknown | unknown | - | - | no | unknown | 1 | 1 |
| `2cbom:key:secret-key:loc:0ae7c1325801bde23c851467` | related-crypto-material | classically-weak | disallowed | 0 | 104 | no | immediate | 1 | 1 |
| `2cbom:key:secret-key:loc:110590fd3066210bf8b1fae4` | related-crypto-material | quantum-safe | acceptable | 5 | 256 | yes | none | 1 | 2 |
| `2cbom:key:secret-key:loc:573558e0613905e1ffc5c7e7` | related-crypto-material | classically-weak | disallowed | 0 | 24 | no | immediate | 1 | 1 |
| `2cbom:key:secret-key:loc:67512275dac23f8e292d52c1` | related-crypto-material | quantum-safe | acceptable | 5 | 344 | yes | none | 1 | 1 |
| `2cbom:key:secret-key:loc:8d3dabda7bfab1f1705c9951` | related-crypto-material | quantum-safe | acceptable | 3 | 208 | no | low | 1 | 1 |
| `2cbom:key:secret-key:loc:9d756f230770f865a4429015` | related-crypto-material | quantum-safe | acceptable | 1 | 128 | no | low | 1 | 1 |
| `2cbom:key:secret-key:loc:c6255f3ddd0a65ef75e60ebb` | related-crypto-material | unknown | unknown | - | - | no | unknown | 1 | 14 |
| `2cbom:key:secret-key:loc:d2781155d265fa1673ac51c9` | related-crypto-material | quantum-safe | acceptable | 1 | 112 | no | low | 1 | 1 |
| `2cbom:key:secret-key:loc:d700b5f20159d6052be67868` | related-crypto-material | classically-weak | disallowed | 0 | 32 | no | immediate | 1 | 1 |
| `2cbom:key:secret-key:loc:fda46c6dd8584f0289d64532` | related-crypto-material | quantum-safe | acceptable | 5 | 512 | yes | none | 1 | 1 |
| `2cbom:proto:tls-1.3` | protocol | unknown | unknown | - | - | no | unknown | 1 | 1 |

## Findings

- **error** `2CBOM-C01-IDENTITY-UNRESOLVED` `2cbom:alg:unresolved:36b8de141ee4d423d533`: algorithm name(s) ['MGF1'] could not be canonicalised
- **error** `2CBOM-C01-IDENTITY-UNRESOLVED` `2cbom:alg:unresolved:7550960cd83390a0618a`: algorithm name(s) ['PSS-3072-OAEP'] could not be canonicalised
- **error** `2CBOM-C03-ASSESSMENT-UNKNOWN` `2cbom:alg:unresolved:36b8de141ee4d423d533`: quantum security class could not be determined: identity could not be resolved by the canonical naming grammar
- **error** `2CBOM-C03-ASSESSMENT-UNKNOWN` `2cbom:alg:unresolved:7550960cd83390a0618a`: quantum security class could not be determined: identity could not be resolved by the canonical naming grammar
- **error** `2CBOM-C03-ASSESSMENT-UNKNOWN` `2cbom:key:initialization-vector:id:2a799bbf0b94...`: quantum security class could not be determined: no algorithm linked to this key material
- **error** `2CBOM-C03-ASSESSMENT-UNKNOWN` `2cbom:key:initialization-vector:loc:90e558045ca...`: quantum security class could not be determined: no algorithm linked to this key material
- **error** `2CBOM-C03-ASSESSMENT-UNKNOWN` `2cbom:key:initialization-vector:loc:9d756f23077...`: quantum security class could not be determined: no algorithm linked to this key material
- **error** `2CBOM-C03-ASSESSMENT-UNKNOWN` `2cbom:key:initialization-vector:loc:ed209f7944a...`: quantum security class could not be determined: no algorithm linked to this key material
- **error** `2CBOM-C03-ASSESSMENT-UNKNOWN` `2cbom:key:private-key:loc:476393f871c4d004cf9291d0`: quantum security class could not be determined: no algorithm linked to this key material
- **error** `2CBOM-C03-ASSESSMENT-UNKNOWN` `2cbom:key:salt:loc:0ae7c1325801bde23c851467`: quantum security class could not be determined: no algorithm linked to this key material
- **error** `2CBOM-C03-ASSESSMENT-UNKNOWN` `2cbom:key:salt:loc:110590fd3066210bf8b1fae4`: quantum security class could not be determined: no algorithm linked to this key material
- **error** `2CBOM-C03-ASSESSMENT-UNKNOWN` `2cbom:key:salt:loc:40dc9409ccef01b91deb3423`: quantum security class could not be determined: no algorithm linked to this key material
- **error** `2CBOM-C03-ASSESSMENT-UNKNOWN` `2cbom:key:salt:loc:573558e0613905e1ffc5c7e7`: quantum security class could not be determined: no algorithm linked to this key material
- **error** `2CBOM-C03-ASSESSMENT-UNKNOWN` `2cbom:key:salt:loc:67512275dac23f8e292d52c1`: quantum security class could not be determined: no algorithm linked to this key material
- **error** `2CBOM-C03-ASSESSMENT-UNKNOWN` `2cbom:key:salt:loc:8d3dabda7bfab1f1705c9951`: quantum security class could not be determined: no algorithm linked to this key material
- **error** `2CBOM-C03-ASSESSMENT-UNKNOWN` `2cbom:key:salt:loc:9d756f230770f865a4429015`: quantum security class could not be determined: no algorithm linked to this key material
- **error** `2CBOM-C03-ASSESSMENT-UNKNOWN` `2cbom:key:salt:loc:d2781155d265fa1673ac51c9`: quantum security class could not be determined: no algorithm linked to this key material
- **error** `2CBOM-C03-ASSESSMENT-UNKNOWN` `2cbom:key:salt:loc:d700b5f20159d6052be67868`: quantum security class could not be determined: no algorithm linked to this key material
- **error** `2CBOM-C03-ASSESSMENT-UNKNOWN` `2cbom:key:secret-key:id:b81263d711b5a126be273ff6`: quantum security class could not be determined: no algorithm linked to this key material
- **error** `2CBOM-C03-ASSESSMENT-UNKNOWN` `2cbom:key:secret-key:id:f247840448065f80b95fb679`: quantum security class could not be determined: no algorithm linked to this key material
- **error** `2CBOM-C03-ASSESSMENT-UNKNOWN` `2cbom:key:secret-key:loc:c6255f3ddd0a65ef75e60ebb`: quantum security class could not be determined: no algorithm linked to this key material
- **error** `2CBOM-C03-ASSESSMENT-UNKNOWN` `2cbom:proto:tls-1.3`: quantum security class could not be determined: tls 1.3: no cipher suites or algorithm references recorded
- **error** `2CBOM-C09-KEY-ALGORITHM-UNLINKED` `2cbom:key:private-key:loc:476393f871c4d004cf9291d0`: private-key is not linked to an algorithm
- **error** `2CBOM-C10-PARAMETER-SET-MISSING` `2cbom:alg:aes-gcm`: missing parameterSet for family 'aes'; security level cannot be derived
- **error** `2CBOM-C10-PARAMETER-SET-MISSING` `2cbom:alg:ec`: missing curve for family 'ec'; security level cannot be derived
- **error** `2CBOM-C10-PARAMETER-SET-MISSING` `2cbom:alg:ecdh`: missing curve for family 'ecdh'; security level cannot be derived
- **error** `2CBOM-C10-PARAMETER-SET-MISSING` `2cbom:alg:ecdsa`: missing curve for family 'ecdsa'; security level cannot be derived
- **error** `2CBOM-C10-PARAMETER-SET-MISSING` `2cbom:alg:rsa`: missing parameterSet for family 'rsa'; security level cannot be derived
- **error** `2CBOM-C10-PARAMETER-SET-MISSING` `2cbom:alg:rsa-pss`: missing parameterSet for family 'rsa'; security level cannot be derived
- **error** `2CBOM-C10-PARAMETER-SET-MISSING` `2cbom:alg:shake`: missing parameterSet for family 'shake'; security level cannot be derived
- **warning** `2CBOM-C04-DANGLING-RELATION` `src-83f9a6b9b0ac:pkg:golang/example.com/cbom-co...`: dependency 'pkg:golang/example.com/cbom-corpus' -> 'pkg:golang/github.com/cloudflare/circl@v1.6.5' (uses) references an unknown bom-ref
- **warning** `2CBOM-C04-DANGLING-RELATION` `src-83f9a6b9b0ac:pkg:golang/example.com/cbom-co...`: dependency 'pkg:golang/example.com/cbom-corpus' -> 'pkg:golang/golang.org/x/crypto@v0.58.0' (uses) references an unknown bom-ref
- **warning** `2CBOM-C04-DANGLING-RELATION` `src-83f9a6b9b0ac:pkg:maven/example.cbom/crypto-...`: dependency 'pkg:maven/example.cbom/crypto-showcase@1.0.0?type=jar' -> 'pkg:golang/example.com/cbom-corpus' (uses) references an unknown bom-ref
- **warning** `2CBOM-C04-DANGLING-RELATION` `src-83f9a6b9b0ac:pkg:maven/example.cbom/crypto-...`: dependency 'pkg:maven/example.cbom/crypto-showcase@1.0.0?type=jar' -> 'pkg:nuget/CryptoShowcase@1.0.0' (uses) references an unknown bom-ref
- **warning** `2CBOM-C04-DANGLING-RELATION` `src-83f9a6b9b0ac:pkg:maven/example.cbom/crypto-...`: dependency 'pkg:maven/example.cbom/crypto-showcase@1.0.0?type=jar' -> 'pkg:pypi/baseline@latest' (uses) references an unknown bom-ref
- **warning** `2CBOM-C04-DANGLING-RELATION` `src-83f9a6b9b0ac:pkg:nuget/CryptoShowcase@1.0.0`: dependency 'pkg:nuget/CryptoShowcase@1.0.0' -> 'pkg:nuget/BouncyCastle.Cryptography@2.7.0' (uses) references an unknown bom-ref
- **warning** `2CBOM-C04-DANGLING-RELATION` `src-83f9a6b9b0ac:pkg:pypi/baseline@latest`: dependency 'pkg:pypi/baseline@latest' -> 'pkg:pypi/argon2@0.1.10' (uses) references an unknown bom-ref
- **warning** `2CBOM-C04-DANGLING-RELATION` `src-83f9a6b9b0ac:pkg:pypi/baseline@latest`: dependency 'pkg:pypi/baseline@latest' -> 'pkg:pypi/nacl@0.0.0' (uses) references an unknown bom-ref
- **warning** `2CBOM-C04-DANGLING-RELATION` `src-83f9a6b9b0ac:pkg:pypi/baseline@latest`: dependency 'pkg:pypi/baseline@latest' -> 'pkg:pypi/python-bcrypt@0.3.2' (uses) references an unknown bom-ref
- **warning** `2CBOM-C07-PROTOCOL-SUITES-MISSING` `2cbom:proto:tls-1.3`: protocol has no cipher suites or algorithm references
- **warning** `2CBOM-C09-KEY-ALGORITHM-UNLINKED` `2cbom:key:secret-key:id:b81263d711b5a126be273ff6`: secret-key is not linked to an algorithm
- **warning** `2CBOM-C09-KEY-ALGORITHM-UNLINKED` `2cbom:key:secret-key:id:f247840448065f80b95fb679`: secret-key is not linked to an algorithm
- **warning** `2CBOM-C09-KEY-ALGORITHM-UNLINKED` `2cbom:key:secret-key:loc:0ae7c1325801bde23c851467`: secret-key is not linked to an algorithm
- **warning** `2CBOM-C09-KEY-ALGORITHM-UNLINKED` `2cbom:key:secret-key:loc:110590fd3066210bf8b1fae4`: secret-key is not linked to an algorithm
- **warning** `2CBOM-C09-KEY-ALGORITHM-UNLINKED` `2cbom:key:secret-key:loc:573558e0613905e1ffc5c7e7`: secret-key is not linked to an algorithm
- **warning** `2CBOM-C09-KEY-ALGORITHM-UNLINKED` `2cbom:key:secret-key:loc:67512275dac23f8e292d52c1`: secret-key is not linked to an algorithm
- **warning** `2CBOM-C09-KEY-ALGORITHM-UNLINKED` `2cbom:key:secret-key:loc:8d3dabda7bfab1f1705c9951`: secret-key is not linked to an algorithm
- **warning** `2CBOM-C09-KEY-ALGORITHM-UNLINKED` `2cbom:key:secret-key:loc:9d756f230770f865a4429015`: secret-key is not linked to an algorithm
- **warning** `2CBOM-C09-KEY-ALGORITHM-UNLINKED` `2cbom:key:secret-key:loc:c6255f3ddd0a65ef75e60ebb`: secret-key is not linked to an algorithm
- **warning** `2CBOM-C09-KEY-ALGORITHM-UNLINKED` `2cbom:key:secret-key:loc:d2781155d265fa1673ac51c9`: secret-key is not linked to an algorithm
- **warning** `2CBOM-C09-KEY-ALGORITHM-UNLINKED` `2cbom:key:secret-key:loc:d700b5f20159d6052be67868`: secret-key is not linked to an algorithm
- **warning** `2CBOM-C09-KEY-ALGORITHM-UNLINKED` `2cbom:key:secret-key:loc:fda46c6dd8584f0289d64532`: secret-key is not linked to an algorithm
- **warning** `2CBOM-C14-SOURCE-CONFLICT` `2cbom:alg:aes-128-cbc-pkcs5`: oid: catalogue value '2.16.840.1.101.3.4.1.2' overrides src-4c14dc79cf35='2.16.840.1.101.3.4.1'
- **warning** `2CBOM-C14-SOURCE-CONFLICT` `2cbom:alg:aes-128-ecb-pkcs5`: oid: catalogue value '2.16.840.1.101.3.4.1.1' overrides src-4c14dc79cf35='2.16.840.1.101.3.4.1'
- **warning** `2CBOM-C14-SOURCE-CONFLICT` `2cbom:alg:aes-128-ecb-pkcs7`: oid: catalogue value '2.16.840.1.101.3.4.1.1' overrides src-4c14dc79cf35='2.16.840.1.101.3.4.1'
- **warning** `2CBOM-C14-SOURCE-CONFLICT` `2cbom:alg:aes-128-gcm`: oid: catalogue value '2.16.840.1.101.3.4.1.6' overrides src-4c14dc79cf35='2.16.840.1.101.3.4.1'
- **warning** `2CBOM-C14-SOURCE-CONFLICT` `2cbom:alg:rsa`: oid: catalogue value '1.2.840.113549.1.1.1' overrides src-83f9a6b9b0ac='2.5.8.1.1'
- **warning** `2CBOM-C15-WEAK-IDENTITY` `2cbom:key:initialization-vector:loc:90e558045ca...`: identity derived from location; occurrences from other sources may not merge
- **warning** `2CBOM-C15-WEAK-IDENTITY` `2cbom:key:initialization-vector:loc:9d756f23077...`: identity derived from location; occurrences from other sources may not merge
- **warning** `2CBOM-C15-WEAK-IDENTITY` `2cbom:key:initialization-vector:loc:ed209f7944a...`: identity derived from location; occurrences from other sources may not merge
- **warning** `2CBOM-C15-WEAK-IDENTITY` `2cbom:key:password:loc:8d3dabda7bfab1f1705c9951`: identity derived from location; occurrences from other sources may not merge
- **warning** `2CBOM-C15-WEAK-IDENTITY` `2cbom:key:private-key:loc:476393f871c4d004cf9291d0`: identity derived from location; occurrences from other sources may not merge
- **warning** `2CBOM-C15-WEAK-IDENTITY` `2cbom:key:salt:loc:0ae7c1325801bde23c851467`: identity derived from location; occurrences from other sources may not merge
- **warning** `2CBOM-C15-WEAK-IDENTITY` `2cbom:key:salt:loc:110590fd3066210bf8b1fae4`: identity derived from location; occurrences from other sources may not merge
- **warning** `2CBOM-C15-WEAK-IDENTITY` `2cbom:key:salt:loc:40dc9409ccef01b91deb3423`: identity derived from location; occurrences from other sources may not merge
- **warning** `2CBOM-C15-WEAK-IDENTITY` `2cbom:key:salt:loc:573558e0613905e1ffc5c7e7`: identity derived from location; occurrences from other sources may not merge
- **warning** `2CBOM-C15-WEAK-IDENTITY` `2cbom:key:salt:loc:67512275dac23f8e292d52c1`: identity derived from location; occurrences from other sources may not merge
- **warning** `2CBOM-C15-WEAK-IDENTITY` `2cbom:key:salt:loc:8d3dabda7bfab1f1705c9951`: identity derived from location; occurrences from other sources may not merge
- **warning** `2CBOM-C15-WEAK-IDENTITY` `2cbom:key:salt:loc:9d756f230770f865a4429015`: identity derived from location; occurrences from other sources may not merge
- **warning** `2CBOM-C15-WEAK-IDENTITY` `2cbom:key:salt:loc:d2781155d265fa1673ac51c9`: identity derived from location; occurrences from other sources may not merge
- **warning** `2CBOM-C15-WEAK-IDENTITY` `2cbom:key:salt:loc:d700b5f20159d6052be67868`: identity derived from location; occurrences from other sources may not merge
- **warning** `2CBOM-C15-WEAK-IDENTITY` `2cbom:key:secret-key:loc:0ae7c1325801bde23c851467`: identity derived from location; occurrences from other sources may not merge
- **warning** `2CBOM-C15-WEAK-IDENTITY` `2cbom:key:secret-key:loc:110590fd3066210bf8b1fae4`: identity derived from location; occurrences from other sources may not merge
- **warning** `2CBOM-C15-WEAK-IDENTITY` `2cbom:key:secret-key:loc:573558e0613905e1ffc5c7e7`: identity derived from location; occurrences from other sources may not merge
- **warning** `2CBOM-C15-WEAK-IDENTITY` `2cbom:key:secret-key:loc:67512275dac23f8e292d52c1`: identity derived from location; occurrences from other sources may not merge
- **warning** `2CBOM-C15-WEAK-IDENTITY` `2cbom:key:secret-key:loc:8d3dabda7bfab1f1705c9951`: identity derived from location; occurrences from other sources may not merge
- **warning** `2CBOM-C15-WEAK-IDENTITY` `2cbom:key:secret-key:loc:9d756f230770f865a4429015`: identity derived from location; occurrences from other sources may not merge
- **warning** `2CBOM-C15-WEAK-IDENTITY` `2cbom:key:secret-key:loc:c6255f3ddd0a65ef75e60ebb`: identity derived from location; occurrences from other sources may not merge
- **warning** `2CBOM-C15-WEAK-IDENTITY` `2cbom:key:secret-key:loc:d2781155d265fa1673ac51c9`: identity derived from location; occurrences from other sources may not merge
- **warning** `2CBOM-C15-WEAK-IDENTITY` `2cbom:key:secret-key:loc:d700b5f20159d6052be67868`: identity derived from location; occurrences from other sources may not merge
- **warning** `2CBOM-C15-WEAK-IDENTITY` `2cbom:key:secret-key:loc:fda46c6dd8584f0289d64532`: identity derived from location; occurrences from other sources may not merge
- **warning** `2CBOM-C20-HINT-MISMATCH` `2cbom:alg:rsa-15-pkcs1v15-sha2-256`: name/field disagreement: ['hint-conflict:parameterSet:2048']
- **warning** `2CBOM-P01-CLASSICALLY-WEAK` `2cbom:alg:md5`: classically weak: replace now (catalogue family 'md5': classically-weak; classical strength 0 bits is below the 112-bit floor)
- **warning** `2CBOM-P01-CLASSICALLY-WEAK` `2cbom:alg:sha1`: classically weak: replace now (catalogue family 'sha1': classically-weak; classical strength 80 bits is below the 112-bit floor)
- **warning** `2CBOM-P01-CLASSICALLY-WEAK` `2cbom:alg:tdea-64`: classically weak: replace now (catalogue family 'tdea': classically-weak; 64-bit block (Sweet32); encryption disallowed by NIST after 2023.)
- **warning** `2CBOM-P01-CLASSICALLY-WEAK` `2cbom:alg:tdea-64-cbc-pkcs5`: classically weak: replace now (catalogue family 'tdea': classically-weak; 64-bit block (Sweet32); encryption disallowed by NIST after 2023.)
- **warning** `2CBOM-P01-CLASSICALLY-WEAK` `2cbom:alg:tdea-64-cbc-pkcs7`: classically weak: replace now (catalogue family 'tdea': classically-weak; 64-bit block (Sweet32); encryption disallowed by NIST after 2023.)
- **warning** `2CBOM-P01-CLASSICALLY-WEAK` `2cbom:key:secret-key:loc:0ae7c1325801bde23c851467`: classically weak: replace now (symmetric material of 104 bits is below the 112-bit floor)
- **warning** `2CBOM-P01-CLASSICALLY-WEAK` `2cbom:key:secret-key:loc:573558e0613905e1ffc5c7e7`: classically weak: replace now (symmetric material of 24 bits is below the 112-bit floor)
- **warning** `2CBOM-P01-CLASSICALLY-WEAK` `2cbom:key:secret-key:loc:d700b5f20159d6052be67868`: classically weak: replace now (symmetric material of 32 bits is below the 112-bit floor)

<details><summary>24 informational findings</summary>

- `2CBOM-C14-SOURCE-CONFLICT` `2cbom:alg:ec-p256`: toolPrimitive: sources disagree (src-acc93be4b00d='other'); chose 'pke' by precedence
- `2CBOM-C14-SOURCE-CONFLICT` `2cbom:alg:sha2-256`: toolPrimitive: sources disagree (src-83f9a6b9b0ac='kdf'); chose 'hash' by precedence
- `2CBOM-C14-SOURCE-CONFLICT` `2cbom:alg:shake-128`: toolPrimitive: sources disagree (src-83f9a6b9b0ac='hash'); chose 'xof' by precedence
- `2CBOM-C17-SPEC-VERSION-NONSTANDARD` `src-acc93be4b00d`: pre-standard IBM CBOM (specVersion 1.4-cbom-1.0) lifted to the CycloneDX 1.6 model
- `2CBOM-I01-SUBSUMED` `2cbom:alg:aes-128-ecb-pkcs7`: detection 'crypto/algorithm/aes@2.16.840.1.101.3.4.1' (aes) from src-83f9a6b9b0ac absorbed into 2cbom:alg:aes-128-ecb-pkcs7 (same location)
- `2CBOM-I01-SUBSUMED` `2cbom:alg:hmac-sha2-256`: detection 'ab702b78-1c07-4555-b593-52edd68277be' (HMAC) from src-4c14dc79cf35 absorbed into 2cbom:alg:hmac-sha2-256 (same location)
- `2CBOM-I01-SUBSUMED` `2cbom:alg:rsa-3072-oaep`: detection '0b6b8c33-a93a-4b6e-945a-a5247879f5cd' (RSA-3072) from src-4c14dc79cf35 absorbed into 2cbom:alg:rsa-3072-oaep (same location)
- `2CBOM-P02-QUANTUM-VULNERABLE` `2cbom:alg:ec`: quantum-vulnerable; NIST IR 8547: disallowed-after-2035; migration priority high
- `2CBOM-P02-QUANTUM-VULNERABLE` `2cbom:alg:ec-p256`: quantum-vulnerable; NIST IR 8547: disallowed-after-2035; migration priority high
- `2CBOM-P02-QUANTUM-VULNERABLE` `2cbom:alg:ecdh`: quantum-vulnerable; NIST IR 8547: disallowed-after-2035; migration priority high
- `2CBOM-P02-QUANTUM-VULNERABLE` `2cbom:alg:ecdh-p256`: quantum-vulnerable; NIST IR 8547: disallowed-after-2035; migration priority high
- `2CBOM-P02-QUANTUM-VULNERABLE` `2cbom:alg:ecdsa`: quantum-vulnerable; NIST IR 8547: disallowed-after-2035; migration priority medium
- `2CBOM-P02-QUANTUM-VULNERABLE` `2cbom:alg:ecdsa-p256`: quantum-vulnerable; NIST IR 8547: disallowed-after-2035; migration priority medium
- `2CBOM-P02-QUANTUM-VULNERABLE` `2cbom:alg:ecdsa-sha2-256`: quantum-vulnerable; NIST IR 8547: disallowed-after-2035; migration priority medium
- `2CBOM-P02-QUANTUM-VULNERABLE` `2cbom:alg:ed25519`: quantum-vulnerable; NIST IR 8547: disallowed-after-2035; migration priority medium
- `2CBOM-P02-QUANTUM-VULNERABLE` `2cbom:alg:rsa`: quantum-vulnerable; NIST IR 8547: disallowed-after-2035; migration priority high
- `2CBOM-P02-QUANTUM-VULNERABLE` `2cbom:alg:rsa-15-pkcs1v15`: quantum-vulnerable; NIST IR 8547: disallowed-after-2035; migration priority medium
- `2CBOM-P02-QUANTUM-VULNERABLE` `2cbom:alg:rsa-15-pkcs1v15-sha2-256`: quantum-vulnerable; NIST IR 8547: disallowed-after-2035; migration priority medium
- `2CBOM-P02-QUANTUM-VULNERABLE` `2cbom:alg:rsa-2048`: quantum-vulnerable; NIST IR 8547: deprecated-after-2030; migration priority high
- `2CBOM-P02-QUANTUM-VULNERABLE` `2cbom:alg:rsa-2048-oaep`: quantum-vulnerable; NIST IR 8547: deprecated-after-2030; migration priority high
- `2CBOM-P02-QUANTUM-VULNERABLE` `2cbom:alg:rsa-3072-oaep`: quantum-vulnerable; NIST IR 8547: disallowed-after-2035; migration priority high
- `2CBOM-P02-QUANTUM-VULNERABLE` `2cbom:alg:rsa-3072-pkcs1v15`: quantum-vulnerable; NIST IR 8547: disallowed-after-2035; migration priority high
- `2CBOM-P02-QUANTUM-VULNERABLE` `2cbom:alg:rsa-pss`: quantum-vulnerable; NIST IR 8547: disallowed-after-2035; migration priority medium
- `2CBOM-P02-QUANTUM-VULNERABLE` `2cbom:alg:x25519`: quantum-vulnerable; NIST IR 8547: disallowed-after-2035; migration priority high

</details>

## Resolved conflicts

- `2cbom:alg:aes-128-cbc-pkcs5`.oid → `'2.16.840.1.101.3.4.1.2'` (settle.kb-authoritative); candidates: kb='2.16.840.1.101.3.4.1.2'; src-4c14dc79cf35='2.16.840.1.101.3.4.1'
- `2cbom:alg:aes-128-ecb-pkcs5`.oid → `'2.16.840.1.101.3.4.1.1'` (settle.kb-authoritative); candidates: kb='2.16.840.1.101.3.4.1.1'; src-4c14dc79cf35='2.16.840.1.101.3.4.1'
- `2cbom:alg:aes-128-ecb-pkcs7`.oid → `'2.16.840.1.101.3.4.1.1'` (settle.kb-authoritative); candidates: kb='2.16.840.1.101.3.4.1.1'; src-4c14dc79cf35='2.16.840.1.101.3.4.1'
- `2cbom:alg:aes-128-gcm`.oid → `'2.16.840.1.101.3.4.1.6'` (settle.kb-authoritative); candidates: kb='2.16.840.1.101.3.4.1.6'; src-4c14dc79cf35='2.16.840.1.101.3.4.1'
- `2cbom:alg:ec-p256`.toolPrimitive → `'pke'` (settle.tool-precedence); candidates: src-4c14dc79cf35='pke'; src-acc93be4b00d='other'
- `2cbom:alg:rsa`.oid → `'1.2.840.113549.1.1.1'` (settle.kb-authoritative); candidates: kb='1.2.840.113549.1.1.1'; src-4c14dc79cf35='1.2.840.113549.1.1.1'; src-83f9a6b9b0ac='2.5.8.1.1'
- `2cbom:alg:sha2-256`.toolPrimitive → `'hash'` (settle.tool-precedence); candidates: src-4c14dc79cf35='hash'; src-acc93be4b00d='hash'; src-83f9a6b9b0ac='kdf'
- `2cbom:alg:shake-128`.toolPrimitive → `'xof'` (settle.tool-precedence); candidates: src-4c14dc79cf35='xof'; src-83f9a6b9b0ac='hash'
