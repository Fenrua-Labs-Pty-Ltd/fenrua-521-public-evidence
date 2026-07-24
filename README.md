# Fenrua-521 public verified evidence

## What this repository is

This is the public, evidence-only release boundary for Fenrua-521.

It contains only final, sanitized results that have already passed private
verification. A public entry is an attestation of a completed `VERIFIED` run;
it is never a proposal, a work-in-progress, a model endpoint, a development
repository, or a mirror of the private mediator.

The private Fenrua Labs mediator repository is the sole place for design,
implementation, formula work, fixtures, execution, diagnostics, and evidence
review. Nothing is moved here merely because it exists or because it produced
a safe failure. It moves here only after the private evidence says `VERIFIED`.

## Public-release admission rule

A result may be published only when all of the following are true:

1. Its private evidence package has `build_state: "VERIFIED"`.
2. The private package and all required receipts passed their integrity checks.
3. The public version is redacted to public-safe outcome data only.
4. Every public result is SHA-256 bound and listed in a SHA-256-bound release
   manifest.
5. The release contains no private or sensitive material.

If any condition is missing, the result stays private. In particular,
`PROPOSAL`, `NEEDS_EVIDENCE`, `INCOMPLETE`, `DIVERGENT`, `BLOCKED`, and failed
or interim runs are never public pass results.

## Required release layout

Each published release uses this layout:

```text
releases/<release-id>/
  release-manifest.json
  results/<result-id>.json
  SHA256SUMS
```

`release-manifest.json` is the public index. It must declare the release as
`VERIFIED`, identify every published result, and bind each result by digest.
Each file under `results/` is a bounded public result or receipt summary.
`SHA256SUMS` records the byte-level SHA-256 hash of every released file for
independent download verification.

## SHA-256 evidence-binding process

Every published result has two complementary hash bindings:

- `result_digest` binds the canonical public JSON content of that result.
- `file_sha256` binds the exact UTF-8 bytes of the result file included in the
  release.

The manifest records both values for every result and has its own
`manifest_digest`. A missing, malformed, duplicate, or non-matching digest
means the item is not a public pass result.

### Canonical public JSON v1

`result_digest` and `manifest_digest` use this deterministic procedure:

1. Start with the public JSON object only. It must contain no private fields.
2. Set the object’s own digest field (`result_digest` or `manifest_digest`) to
   the empty string.
3. NFC-normalize all strings, then serialize as UTF-8 using the [JSON
   Canonicalization Scheme (RFC 8785)](https://www.rfc-editor.org/rfc/rfc8785).
   This fixes object-key order, number representation, string escaping, and
   whitespace; no trailing newline is included in the hash input.
4. Compute SHA-256 over those UTF-8 bytes and write it as lowercase
   `sha256:<64 hexadecimal characters>`.
5. Re-serialize the released JSON file and compute its exact-byte
   `file_sha256`. Record that same byte hash in `SHA256SUMS`.

The digest is calculated after public redaction. No private digest, formula
commitment, runtime inventory, prompt, or raw receipt is reused as a public
substitute.

### Required public manifest fields

```json
{
  "schema_version": "fenrua-521-public-evidence/v1",
  "release_id": "<public-release-id>",
  "build_state": "VERIFIED",
  "published_at": "<ISO-8601 UTC timestamp>",
  "results": [
    {
      "result_id": "<public-result-id>",
      "result_path": "results/<public-result-id>.json",
      "result_digest": "sha256:<64 lowercase hex>",
      "file_sha256": "sha256:<64 lowercase hex>"
    }
  ],
  "manifest_digest": "sha256:<64 lowercase hex>"
}
```

Every result file must contain its own `result_id`, `build_state: "VERIFIED"`,
bounded public outcome data, and a matching `result_digest`.

## Independent verification steps

Anyone receiving a public release can verify it without access to the private
mediator:

1. Confirm the manifest and every result say `build_state: "VERIFIED"`.
2. Confirm that every manifest result entry resolves to one unique file.
3. Recompute each result’s canonical `result_digest` with its digest field
   blank and compare it with the result file and manifest entry.
4. Recompute each `file_sha256` from the exact downloaded file bytes and
   compare it with both the manifest and `SHA256SUMS`.
5. Recompute `manifest_digest` with that field blank.
6. Reject the complete release if any check fails or any required digest is
   absent.

## Material that never belongs here

- Formula contracts, formula inputs, private source code, or implementation
  details.
- Raw prompts, raw model responses, API keys, canaries, tenant material,
  private filesystem paths, model-weight inventories, or local runtime data.
- Private evidence packages, diagnostic logs, failed attempts, blocked runs,
  or interim test results.

## Current release status

Published releases:

- [`F521-PUB-BASELINE-001`](releases/f521-public-baseline-001/) records the
  verified 52-case deterministic KRN mediator conformance run: 23 verified
  outcomes, 3 contained outcomes, 26 policy refusals, and 0 errors.
- [`F521-PUB-ENVELOPE-001`](releases/f521-public-envelope-001/) records the
  verified six-case inter-engine envelope conformance run: 1 accepted
  envelope, 5 correctly rejected envelopes, and 0 errors.

Each release directory contains its own manifest, result files, and
`SHA256SUMS`. Neither release attests a live capability-model response or
production authority.
