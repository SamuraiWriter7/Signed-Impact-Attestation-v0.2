# Signed Impact Attestation v0.2

A minimal, portable, and auditable envelope for attaching cryptographic accountability to an Impact Score Profile.

Signed Impact Attestation v0.2 extends signed impact evaluation with **Existence Proof binding**, so that the responsible issuer's **existence, authority, and validity** can be verified at the time of issuance.

This specification is designed as a foundational accountability layer for **Communication Royalty OS**, **Royalty OS**, **Trust OS**, **Gratitude OS**, and related AI-civilization trace infrastructures.

---

## Why this spec matters

An Impact Score Profile can express **how much contribution is claimed**.

A Signed Impact Attestation expresses **who takes responsibility for that claim**.

Version 0.2 goes one step further: it binds that responsibility to an **Existence Proof**, so that downstream systems can verify:

- that the issuer actually exists,
- that the issuer had the relevant authority,
- that the issuer was valid at issuance time,
- and that the credential or key was not revoked.

In short:

- **Impact Score Profile** = how much contribution is claimed  
- **Signed Impact Attestation** = who is responsible for that evaluation  
- **Existence Proof Binding** = whether that responsible issuer is real, valid, and authorized  

This moves the model from simple cryptographic signing toward **civilizational accountability**.

---

## Core idea

Signed Impact Attestation v0.2 is not a claim of universal truth.

It is an **accountable assertion**.

Its purpose is to make impact evaluation socially usable by attaching:

- cryptographic integrity,
- issuer identity,
- existence proof binding,
- revocation-aware validity,
- and optional authority scope.

---

## Design goals

- Keep the attestation envelope **minimal and portable**
- Separate **scoring logic** from **signature logic**
- Bind issuer responsibility to an **Existence Proof**
- Support human, organization, AI agent, and hybrid issuers
- Enable revocation-aware and time-bounded verification
- Stay lightweight enough for practical adoption

---

## What is new in v0.2

Compared with v0.1, version 0.2 adds an explicit `existence_proof` block.

This means the attestation can now carry or reference:

- existence proof identifier
- existence proof hash
- subject identity binding
- subject type binding
- credential status
- validity window
- authority scope
- assurance level
- public key binding
- revocation check result

This turns the attestation from:

> “someone signed this evaluation”

into:

> “a verifiable and valid responsible issuer signed this evaluation under a recognizable authority context”

---

## Repository structure

```text
.
├── .github/
│   └── workflows/
│       └── validate-specs.yml
├── examples/
│   └── signed-impact-attestation.sample.json
├── schemas/
│   └── signed-impact-attestation-v0.2.schema.json
├── spec/
│   └── signed-impact-attestation-v0.2.yaml
└── README.md
Start here

If you are new to this repository, read the files in this order:

spec/signed-impact-attestation-v0.2.yaml
The human-readable normative specification.
schemas/signed-impact-attestation-v0.2.schema.json
The machine-readable validation schema.
examples/signed-impact-attestation.sample.json
A valid example document for testing and implementation.
.github/workflows/validate-specs.yml
The automated validation workflow used by GitHub Actions.
Specification summary
One-line definition

Signed Impact Attestation v0.2 defines a portable and auditable envelope that binds signed impact evaluation to the existence proof and credential validity of the responsible issuer.

Human meaning

This spec answers all of the following:

What evaluation is being attested?
Who issued the attestation?
Does that issuer exist?
Was that issuer valid at issuance time?
Was the issuer authorized to make this kind of evaluation?
Can downstream systems trust this attestation for action?
Document model

A Signed Impact Attestation v0.2 document contains these top-level blocks:

version
attestation_id
issued_at
subject
issuer
existence_proof
review
signature
integrity

Optional fields may include:

ctr_id
review.review_policy_ref
existence_proof.authority_scope
existence_proof.assurance_level
existence_proof.revocation_check
integrity.transparency_log_entry
Existence Proof binding

The main extension introduced in v0.2 is Existence Proof binding.

This requires that:

issuer.issuer_id matches existence_proof.subject_id
issuer.issuer_type matches existence_proof.subject_type
issued_at falls within the declared validity window when present
revoked or expired credentials are not treated as actionable
declared authority scope is respected by downstream systems

This is the core of the v0.2 model.

Effective meaning of status

The review.decision field expresses the issuer’s explicit evaluation state:

approved
rejected
disputed
pending
withdrawn

In addition, downstream verifiers may derive an effective status such as:

actionable
non_actionable
disputed
indeterminate

Example:

approved + credential_status = valid + revocation_check.result = not_revoked
→ likely actionable
approved + credential_status = revoked
→ non_actionable

This distinction is important for Trust OS, Gratitude OS, and Royalty OS integrations.

Supported issuer types

The schema currently supports the following issuer categories:

human
organization
ai_agent
hybrid_agent
automated_system

This allows the specification to operate across both present-day institutional systems and future AI-agent ecosystems.

Schema Usage
Validate locally

You can validate the example document locally with Python.

Install dependencies:

python -m pip install --upgrade pip
pip install PyYAML jsonschema

Then run:

python - <<'PY'
import json
import sys
from pathlib import Path

import yaml
from jsonschema import Draft202012Validator


def fail(message: str) -> None:
    print(f"ERROR: {message}")
    sys.exit(1)


def load_yaml(path: Path):
    if not path.exists():
        fail(f"Spec file not found: {path}")
    try:
        with path.open("r", encoding="utf-8") as f:
            return yaml.safe_load(f)
    except yaml.YAMLError as e:
        fail(f"Invalid YAML in {path}: {e}")


def load_json(path: Path):
    if not path.exists():
        fail(f"JSON file not found: {path}")
    try:
        with path.open("r", encoding="utf-8") as f:
            return json.load(f)
    except json.JSONDecodeError as e:
        fail(f"Invalid JSON in {path}: {e}")


def main() -> None:
    spec_path = Path("spec/signed-impact-attestation-v0.2.yaml")
    schema_path = Path("schemas/signed-impact-attestation-v0.2.schema.json")
    sample_path = Path("examples/signed-impact-attestation.sample.json")

    print("=== Validating Signed Impact Attestation repository ===")
    print()

    print("=== Validating YAML specification ===")
    print(f"Spec: {spec_path}")
    spec = load_yaml(spec_path)

    if not isinstance(spec, dict):
        fail(f"YAML spec must be a mapping/object: {spec_path}")

    expected_spec_id = "signed-impact-attestation-v0.2"
    actual_spec_id = spec.get("spec_id")
    if actual_spec_id != expected_spec_id:
        fail(
            f"Unexpected spec_id in {spec_path}: "
            f"expected '{expected_spec_id}', got '{actual_spec_id}'"
        )

    print("OK: YAML specification loaded successfully.")
    print()

    print("=== Validating JSON Schema ===")
    print(f"Schema: {schema_path}")
    schema = load_json(schema_path)

    try:
        Draft202012Validator.check_schema(schema)
    except Exception as e:
        fail(f"Invalid JSON Schema in {schema_path}: {e}")

    print("OK: JSON Schema is valid.")
    print()

    print("=== Validating example document ===")
    print(f"Sample: {sample_path}")
    sample = load_json(sample_path)

    validator = Draft202012Validator(schema)
    errors = sorted(validator.iter_errors(sample), key=lambda e: list(e.path))

    if errors:
        print("Validation errors found:")
        for error in errors:
            path = ".".join(str(p) for p in error.path) or "<root>"
            print(f" - {path}: {error.message}")
        sys.exit(1)

    print("OK: Signed Impact Attestation v0.2 sample is valid.")
    print()
    print("Validation succeeded.")


if __name__ == "__main__":
    main()
PY
What this validation checks

The local validation flow checks:

that spec/signed-impact-attestation-v0.2.yaml exists and loads correctly
that spec_id is signed-impact-attestation-v0.2
that schemas/signed-impact-attestation-v0.2.schema.json is valid Draft 2020-12 JSON Schema
that examples/signed-impact-attestation.sample.json conforms to the schema
What JSON Schema does not fully enforce

JSON Schema is excellent for structure validation, but some semantic checks still require an additional verifier layer.

Examples include:

issuer.issuer_id == existence_proof.subject_id
issuer.issuer_type == existence_proof.subject_type
signature.key_id being truly bound to existence_proof.public_keys
actual cryptographic signature verification
actual revocation lookup
actual hash re-computation of external referenced payloads

These should be enforced by a dedicated verifier implementation.

GitHub Actions

This repository includes an automated validation workflow:

.github/workflows/validate-specs.yml

It validates:

the YAML spec
the JSON Schema
the example document

on:

push
pull request
manual dispatch
Conformance levels
Minimal conformance

A document is minimally conformant to v0.2 when it includes the required structural fields, including:

version
attestation ID
issuance timestamp
subject hash
issuer identity
existence proof reference and binding fields
review decision
signature block
integrity block
Actionable conformance

A document is stronger for downstream use when it also includes:

validity window
revocation check
key binding
authority scope
assurance level
review policy reference

This stronger profile is more suitable for:

trust propagation
gratitude acknowledgment
royalty preparation
governance review
dispute-aware trace infrastructures
Security considerations

Implementers should pay special attention to the following risks:

1. Issuer impersonation

A valid-looking signature is not enough if the existence proof is unrelated or forged.

2. Expired or revoked credentials

A once-valid issuer may become invalid later.

3. Key substitution

A signing key should be bound to the existence proof subject whenever possible.

4. Reference swapping

External references without hash anchoring can be silently replaced.

5. Authority overreach

An issuer valid for impact review may not be authorized for royalty governance or trust adjudication.

6. Replay across contexts

An attestation may be replayed outside the governance context it was originally intended for.

Privacy considerations

This specification can support privacy-preserving implementations.

For example:

issuers may use organizational or pseudonymous identifiers
existence proofs may use privacy-preserving credential systems
downstream systems may verify authority without exposing all issuer metadata

Still, enough information must remain available to verify:

identity binding
validity
authority scope
revocation state
Intended downstream integrations

Signed Impact Attestation v0.2 is designed to serve as an accountability layer for systems such as:

Communication Royalty OS
Royalty OS
Trust OS
Gratitude OS
Existence Proof OS
CTR-ID linked trace systems

In these ecosystems, the attestation functions as a responsibility-bearing bridge between:

trace,
evaluation,
trust,
and value circulation.
Versioning and compatibility

This repository currently defines:

signed-impact-attestation-v0.2

Version 0.2 is designed to remain conceptually compatible with v0.1 while extending it with issuer existence binding.

A v0.1-style implementation may still parse part of the document, but it should not claim full v0.2 validation unless it also verifies the existence proof layer.

Example file

A valid example document is included here:

examples/signed-impact-attestation.sample.json

This example shows:

a signed impact evaluation
an organization issuer
a bound existence proof
a validity window
authority scope
revocation check
integrity metadata
Status

Current status:

Version: 0.2
Specification state: draft
Document type: normative specification

This repository is intended as a foundational accountability primitive for future AI-civilization infrastructures.

Philosophy in one sentence

If v0.1 was responsible assertion, v0.2 is existence-backed responsible assertion.

License

Unless otherwise specified in the repository, this specification text is released under:

CC-BY-4.0

Check repository files for the final applicable license configuration.
