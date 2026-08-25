# Secoya Trust Record `/3` RC1

Status: frozen compatibility baseline consolidated from the existing RC1 conformance specification, schemas, vectors and previously frozen design decisions.

This document is intended to become the single standalone prose authority for RC1 after owner ratification. Existing schemas, vectors and SDK behavior are subordinate conformance evidence and must be reconciled to this document. This consolidation does not change RC1 bytes or semantics.

## 1. Scope

RC1 normatively specifies `secoya-trust-record/3` only.

`secoya-trust-record/2` remains a legacy format that implementations may continue to verify using its original `secoya-jcs/2` rules. `/2` is not part of the Trust Archive or Gate 6 normative scope. A verifier MUST NOT process `/2` using `/3` rules or `/3` using `/2` rules.

RC1 defines:

- the JSON record structure;
- strict JSON and RFC 8785 canonicalisation requirements;
- payload-digest calculation;
- per-signature signing input;
- initial algorithm identifiers;
- embedded signature policy;
- contextual acceptance fields; and
- the inputs required by a relying-party verification decision.

RC1 does not itself define an online service, key registry implementation, replay database, timestamp authority, transparency service, trust archive, or originating workflow.

## 2. Requirements terminology

The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT, RECOMMENDED, NOT RECOMMENDED, MAY and OPTIONAL are to be interpreted as described by BCP 14 when they appear in uppercase.

## 3. Record structure

An RC1 record is an I-JSON object with exactly three top-level members:

```json
{
  "protected": { "...": "..." },
  "payload": { "...": "..." },
  "signatures": [
    {
      "protected": { "...": "..." },
      "signature": "base64url"
    }
  ]
}
```

The members `protected`, `payload` and `signatures` are REQUIRED. Unknown top-level members MUST be rejected.

`payload` MAY be any value permitted by I-JSON and the applicable defensive limits. The record MUST contain at least one signature for acceptance. Signature order has no acceptance significance.

## 4. Strict JSON input

At an untrusted wire boundary, a verifier MUST reject input before ordinary object construction when it contains:

- duplicate object member names;
- malformed JSON;
- trailing non-whitespace data;
- non-I-JSON strings, including lone Unicode surrogates;
- non-finite numbers;
- prohibited implementation-safety keys; or
- values exceeding the verifier's declared defensive resource limits.

An implementation MAY offer an in-memory verification function for objects that have already passed an equivalent trusted parser boundary. It MUST NOT claim that such a function detects duplicate names that an earlier parser discarded.

## 5. Common protected header

The record `protected` object MUST contain exactly:

- `format`;
- `evidenceUnitType`;
- `canonicalization`;
- `digest`;
- `policy`;
- `purpose`;
- `context`;
- `recordId`;
- `nonce`; and
- `predecessor`.

### 5.1 Format and canonicalisation

`format` MUST equal `secoya-trust-record/3`.

`canonicalization` MUST equal `RFC8785`.

The pair MUST be dispatched together. An unknown or mixed pair MUST be rejected. A verifier MUST NOT infer canonicalisation from the record shape, algorithm, key, provider or historical default.

### 5.2 Evidence-unit type and purpose

`evidenceUnitType` and `purpose` MUST be non-empty identifiers from the applicable core registry or use the namespaced form:

```text
ns:<namespace>/<term>
```

Matching is exact and case-sensitive. A relying party MAY further restrict accepted evidence-unit types and purposes.

### 5.3 Context

`context` MUST contain:

- `environment`: a registered or namespaced identifier;
- `tenant`: a string or explicit `null`; and
- `workflow`: an object containing non-empty `definition` and `instance` identifiers.

`context` MAY contain:

- `audience`: a non-empty string; and
- `extensions`: a JSON object whose member names use `ns:<namespace>/<term>`.

An omitted tenant is not equivalent to `null` and MUST be rejected.

Unknown extension members are semantically ignored by RC1 but remain included in the protected signing bytes. RC1 defines no critical-extension mechanism. Security-relevant new extension semantics require a separately versioned profile.

### 5.4 Record identity and replay inputs

`recordId` MUST contain non-empty `id` and `scope` identifiers.

`nonce` MUST be a string or explicit `null`.

The tuple used by a deployment replay policy is external to base mathematical verification. RC1 protects these values against alteration but does not itself provide durable uniqueness state.

### 5.5 Predecessor

`predecessor` MUST be either `null` or:

```json
{
  "digestAlgorithm": "SHA-384",
  "value": "base64url"
}
```

For an RC1 renewal created by the reference construction, `value` is the unpadded Base64url encoding of SHA-384 over the RFC 8785 canonical bytes of the predecessor's common protected header.

The link binds a successor to the referenced prior protected header. It does not by itself establish that the predecessor was the latest record, that no record was omitted, or that all expected workflow events were recorded.

## 6. Payload digest

`protected.digest` MUST contain exactly:

```json
{
  "algorithm": "SHA-384",
  "value": "base64url"
}
```

The digest is:

```text
payloadBytes = UTF8(RFC8785(record.payload))
digestBytes  = SHA-384(payloadBytes)
digest.value = BASE64URL-NOPAD(digestBytes)
```

A verifier MUST recompute the digest and MUST reject a mismatch. It MUST select SHA-384 exclusively from the protected identifier and MUST reject unknown or unavailable identifiers.

## 7. Embedded signature policy

`protected.policy` MUST contain exactly:

- `mode`;
- `acceptedClassical`; and
- `acceptedPostQuantum`.

RC1 permits:

- classical algorithm `Ed25519`; and
- post-quantum algorithm `ML-DSA-65`.

Unknown algorithms and duplicate entries MUST be rejected.

The policy modes are:

| Mode | Required trusted signature classes |
|---|---|
| `classical` | At least one accepted classical signature |
| `pq-observe` | At least one accepted classical signature; PQ results may be observed but do not replace the classical requirement |
| `hybrid-preferred` | At least one accepted classical or accepted post-quantum signature |
| `hybrid-required` | At least one accepted classical and at least one accepted post-quantum signature |
| `pq-only` | At least one accepted post-quantum signature |

An embedded issuer policy is necessary but not sufficient for acceptance. The relying party MUST also explicitly accept the embedded mode under its own policy.

## 8. Signature structure

Each element of `signatures` MUST contain exactly:

- `protected`: a per-signature protected object; and
- `signature`: the unpadded Base64url encoding of the raw signature bytes.

The per-signature protected object MUST contain:

- `algorithm`: `Ed25519` or `ML-DSA-65`;
- `keyId`: a non-empty identifier;
- `authorityId`: a non-empty identifier; and
- `createdAt`: a UTC timestamp in `YYYY-MM-DDTHH:mm:ss.sssZ` form.

It MAY contain `keyFingerprint` as historical supplementary key-binding data.

A verifier MUST reject duplicate `(keyId, algorithm)` signature entries.

## 9. Signing input

For each signature independently:

```text
signingObject = {
  "recordProtected": record.protected,
  "sigProtected": signature.protected
}

signingInput = UTF8(RFC8785(signingObject))
```

The signature algorithm is applied directly to `signingInput` according to its RC1 algorithm profile.

This construction is frozen. A conformant implementation MUST NOT replace it with JWS Signing Input, COSE `Sig_structure`, a prehash variant, a locale-sensitive representation, provider defaults or inferred framing.

## 10. Algorithms

### 10.1 Ed25519

`Ed25519` means Ed25519 as defined by RFC 8032/FIPS 186-5 over the exact RC1 signing input.

### 10.2 ML-DSA-65

`ML-DSA-65` means ML-DSA-65 as defined by FIPS 204 over the exact RC1 signing input. The context parameter is empty. A prehash or external-mu variant is not silently interchangeable.

### 10.3 Algorithm changes

An implementation MUST select the algorithm only from the protected per-signature identifier. It MUST reject unsupported, unavailable, unknown, inferred, substituted or downgraded algorithms.

New signature or digest algorithms require a separately versioned algorithm profile and conformance vectors. Existing records retain their original algorithm interpretation.

## 11. RFC 8785 processing

All `/3` payload and signing-input canonicalisation MUST apply RFC 8785 in full and encode the resulting string as UTF-8.

Consequently:

- input is constrained to I-JSON;
- object member names are sorted by UTF-16 code units;
- ECMAScript-compatible finite IEEE-754 number serialisation is used;
- insignificant whitespace is absent;
- duplicate names are rejected before canonicalisation;
- lone surrogates are rejected; and
- locale-sensitive ordering and runtime map order are forbidden.

## 12. Trust resolution inputs

Mathematical verification requires candidate public-key material. Trusted acceptance additionally requires externally established trust state.

For each signature, the relying environment supplies or resolves:

- public verification key;
- key authority binding;
- key status;
- validity interval;
- revocation state and, when present, effective time; and
- optional independently defined key thumbprint/fingerprint evidence.

RC1 does not define how that state is hosted. A resolver MUST NOT treat `keyId` or a record-supplied key as self-authenticating.

`createdAt` is issuer-declared and signature-protected. It prevents later alteration but is not external proof of time.

Historical-state publication, time-indexed resolution, retirement, transition and external-time artifacts are specified outside RC1 by the Trust Archive profile. They MUST NOT alter the RC1 signing input or reinterpret existing signature bytes.

## 13. Verification procedure

A conformant relying-party verification performs these layers in order and fails closed where a mandatory layer cannot be established:

1. Strictly parse untrusted JSON and enforce structure/resource limits.
2. Require the exact `/3` and `RFC8785` pair.
3. Validate protected fields, registries, identifiers, policy and signature encodings.
4. Recompute and compare the payload digest.
5. For each signature, resolve candidate key material for the protected `keyId` and `algorithm`.
6. Reconstruct the exact RC1 signing input and mathematically verify the raw signature.
7. Establish authority by matching the protected `authorityId` to authenticated external trust material unless the relying policy explicitly performs mathematical-only verification.
8. Evaluate key usability under the trust state supplied by the applicable trust-resolution profile.
9. Determine the set of trusted classical and post-quantum signatures.
10. Require the relying-party policy to accept the embedded mode.
11. Require the trusted signature classes to satisfy the embedded mode.
12. Apply exact relying-party constraints for context, purpose and evidence-unit type.
13. Apply external-time and replay requirements only when required by the relying-party profile; report unevaluated optional layers explicitly.
14. Return layered reasons and an overall relying-party decision.

Mathematical signature validity MUST NOT be reported as authority, time, replay, rollback or completeness assurance.

## 14. Relying-party policy

A relying-party policy supplies at least:

- accepted embedded modes;
- accepted purposes;
- accepted evidence-unit types;
- required context, if any;
- whether authority establishment is mandatory;
- whether external time is mandatory; and
- whether replay, continuity, rollback or completeness layers are in scope.

RC1 does not collapse issuer policy and relying-party policy into one policy.

## 15. Security boundaries

RC1 establishes cryptographic binding between:

- payload and protected digest;
- common protected metadata and every signature; and
- per-signature algorithm, key ID, authority ID and declared time and that signature.

RC1 alone does not establish:

- real-world truth of payload claims;
- secure private-key custody;
- authority identity without an external anchor;
- independently evidenced time;
- live replay prevention;
- latest-state/rollback resistance;
- log non-equivocation;
- deletion/omission resistance; or
- continuity through key, policy, algorithm or anchor transition.

Those properties require explicit external evidence and MUST be reported separately.

## 16. Conformance

A conforming RC1 implementation MUST:

- pass every applicable committed `/3` manifest entry;
- reproduce expected RFC 8785 bytes, SHA-384 values and RC1 signing inputs;
- reproduce expected signature and parser decisions;
- reject unknown algorithms and format/canonicalisation pairs;
- preserve the exact semantics in this document; and
- avoid dependence on undocumented SDK behavior.

Partial vector success is not conformance.

## 17. Normative references

- RFC 8032, Edwards-Curve Digital Signature Algorithm (EdDSA)
- RFC 8785, JSON Canonicalization Scheme
- FIPS 180-4, Secure Hash Standard
- FIPS 186-5, Digital Signature Standard
- FIPS 204, Module-Lattice-Based Digital Signature Standard

COSE and JOSE are not normative containers for RC1 records.
