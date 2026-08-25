# RC1 Ratification & Identity Statement

## Ratification

The Secoya protocol owner ratified `SECOYA-TRUST-RECORD-RC1.md` on 8 August 2026 as the frozen normative prose authority for `secoya-trust-record/3` RC1.

- **Format identifier:** `secoya-trust-record/3`
- **Release identifier:** `secoya-trust-record/3-rc1`
- **Normative authority:** `SECOYA-TRUST-RECORD-RC1.md`
- **Required SHA-256:** `596FA623A3297599099ACCD8D83A2E43FC29E2D4A6CAF26CFFB77B0102C37B0A`
- **Owner identity:** © 2026 Secoya Systems
- **Contact:** https://secoya.systems

The authority freezes the existing RC1 signing input, RFC 8785 processing, signature representation, algorithms, policy modes, record bytes and verification semantics. It does not convert RC1 to JOSE or COSE.

## Subordinate material

`trust-record-3.schema.json`, `example-record.json` and `README.md` are subordinate aids. They do not amend RC1. If any subordinate material conflicts with the frozen prose authority, the prose authority prevails and the conflict must be reported as a publication or conformance defect.

## Patent and licensing notice

Secoya Systems identifies UK patent application `GB2619031.4`, filed 14 August 2026, as relevant to aspects of its full attestation and independent-verification method. This statement records an application, not a granted patent or an infringement determination.

The public envelope specification and structural conformance material are intended for independent implementation. Final copyright and patent licensing terms remain subject to legal review; no `LICENSE` file is included in this package.

## Integrity check

Before relying on the authority, calculate SHA-256 over the exact bytes of `SECOYA-TRUST-RECORD-RC1.md` and require:

```text
596FA623A3297599099ACCD8D83A2E43FC29E2D4A6CAF26CFFB77B0102C37B0A
```
