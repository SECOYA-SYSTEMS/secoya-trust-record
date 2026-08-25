
# secoya-trust-record / 3 RC1 - Independent Cryptographic Evidence Records

The authoritative specification is [`SECOYA-TRUST-RECORD-RC1.md`](./SECOYA-TRUST-RECORD-RC1.md), the frozen ratified standard for `secoya-trust-record/3`. All other files in this repository are subordinate references. If any subordinate file conflicts with the prose authority, the prose authority prevails.

## What the format defines

`secoya-trust-record/3` defines a strict JSON evidence-record envelope. It binds a payload digest, protected record context and protected per-signature metadata through Ed25519, ML-DSA-65, or policy-governed combinations of those signatures. It uses RFC 8785 canonicalisation and SHA-384.

A record can be mathematically verified without contacting Secoya or the originating application when the verifier possesses the required public-key material. Trusted acceptance additionally requires authenticated authority, key-state and relying-party policy material. RC1 does not make trust self-authenticating and does not by itself establish real-world truth, external time, replay prevention, rollback resistance or completeness.

The format is relevant to evidence controls used in operational resilience, security and automated-decision traceability programmes, including contexts addressed by NIS2 Article 21, FCA SYSC 15A and EU AI Act Article 12. Publication of this format is not a representation that using it alone establishes regulatory compliance.

## Implementation and intellectual-property boundary

- **Public format material:** The envelope structure, RFC 8785 processing, SHA-384 digest construction and verification semantics are published so independent implementations can be evaluated against the frozen authority.
- **Secoya implementation and services:** Production evidence capture, event-time integration, key custody, trust publication, replay state, monitoring, historical verification operations and customer deployment controls are separate implementation concerns and are not included here.
- **Patent notice:** Secoya Systems identifies UK patent application `GB2619031.4`, filed 14 August 2026, as relevant to aspects of its full attestation and independent-verification method. The existence of an application is not a statement that a patent has been granted or that any particular activity infringes a claim.
- **Licence status:** No licence file is included in this publication candidate. Secoya intends the record-format material to be independently implementable, but final copyright and patent licensing terms remain subject to legal review. Do not infer a broader licence grant from repository access alone.

## Policy modes

RC1 section 7 defines these exact embedded policy modes:

- `classical`
- `pq-observe`
- `hybrid-preferred`
- `hybrid-required`
- `pq-only`

An embedded issuer policy is necessary but not sufficient for acceptance. A relying party must explicitly accept the embedded mode under its own policy.

## Files

- [`SECOYA-TRUST-RECORD-RC1.md`](./SECOYA-TRUST-RECORD-RC1.md) - frozen normative prose authority.
- [`RC1-RATIFICATION.md`](./RC1-RATIFICATION.md) - ratification identity, integrity hash and publication status.
- [`trust-record-3.schema.json`](./trust-record-3.schema.json) - subordinate Draft 7 structural aid.
- [`example-record.json`](./example-record.json) - structurally conforming illustrative record with deliberately invalid placeholder signatures.
- [`README.md`](./README.md) - repository landing page and publication boundary.

## Status

The prose authority was ratified on 8 August 2026. Its required SHA-256 is:

```text
596FA623A3297599099ACCD8D83A2E43FC29E2D4A6CAF26CFFB77B0102C37B0A
```

© 2026 Secoya Systems. Ratified 8 August 2026. Publication package prepared 25 August 2026.
