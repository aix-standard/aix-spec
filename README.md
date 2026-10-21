# AIX — AI Interchange Standard

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Version](https://img.shields.io/badge/version-2.4-green.svg)](spec/AIX-Standard.md)
[![Status](https://img.shields.io/badge/status-ratified-brightgreen.svg)](spec/AIX-Standard.md)

**An open standard for naming, exchanging data, and establishing interoperability between generative AI tools, agents, and applications.**

Website: [aix-standard.org](https://aix-standard.org) · Contact: [aix@aix-standard.org](mailto:aix@aix-standard.org)

> **Mirror policy:** This README is maintained by hand in the canonical AIX repository; only its Provenance section, at the end, is generated with each release. `spec/AIX-Standard.md` is generated from the canonical standard by a sanitizer that removes the sections the canonical source marks internal (`aix:internal`). The earlier versions under `spec/` are frozen historical records, and no release regenerates them. Changes flow from the canonical repository to this mirror, never back. Citations of reference-implementation code (file paths and line numbers) stay in the public spec deliberately: the sanitizer removes internal process and private pointers, not the evidence a claim rests on.

---

## What is AIX?

The generative AI ecosystem has hundreds of tools — each with its own API format, field names, and response structure. AIX addresses two problems:

**1. Interchange:** A standard envelope for AI task requests and responses so any AIX-compliant tool can accept standardized inputs, return standardized outputs, be swapped with any other tool in the same category, and chain seamlessly in workflows.

**2. Naming:** A canonical naming convention so every field, key, and identifier in the AI ecosystem is unambiguous, hierarchical, and implementable by developers with no internal context.

AIX follows the pattern established by [CrossARM (2014)](https://ubertrends.com), a data interchange standard for cross-platform marketing data: use Schema.org canonical names where they exist, and define extensions for what Schema.org does not cover.

---

## Quick Start

### The AIX Envelope

Every AIX message wraps in a standard envelope:

```json
{
  "aix": "2.4",
  "id": "uuid-v4",
  "timestamp": "2026-05-01T09:00:00Z",
  "task": { "type": "content_authoring", "provider": "anthropic", "model": "claude-sonnet-4-6" },
  "input": { },
  "output": { },
  "meta": { "cost": { }, "latency": { }, "tokens": { } }
}
```

### Naming Convention

All AIX field keys use **camelCase** and a **four-part hierarchy**:

```
{component}_{objectType}_{identifier}_{property}

writingAgent_format_newsletter_systemPrompt
researchAgent_qualityPass_hallucinationSweep_systemPrompt
personaAgent_voice_brand
publishingModule_newsletterStudio_studioJson
```

---

## Specification

> **Companion specification:** The UMA 3.0 companion specification is maintained separately and is available on request from [aix@aix-standard.org](mailto:aix@aix-standard.org).

AIX is one standard with one version — there is no separately-versioned envelope standard and a separate content or registry standard carrying its own number. Every surface that stamps an AIX version moves together when the version changes. Prior versions below are a historical record of what was ratified at each point; their descriptions are not rewritten as later versions supersede them.

| Version | Status | Date | Summary |
| --- | --- | --- | --- |
| [v2.4](spec/AIX-Standard.md) | Ratified | August 2026 | Converges AIX Standard v2.2, the AIX 2.3 Canonical Registry Contract, and the UMA 3.0 Contract Meta-Framework's contract structure (its §3, in full) and key families into one document, one standard, one version — supersedes v2.2 and v2.3's two-standards framing (both physically inlined here). Module Manifest Standard promoted to its own chapter; Canonical Registry Contract and UMA 3.0 Primitive Contract Structure added as new chapters. |
| [v2.3](spec/aix-2.3-canonical-registry-contract.md) | Ratified | August 2026 | Canonical Registry Contract addendum — capability, tier, lifetime, resource-scope, and obligations axes for scoped agent reads; grant tuple; audit contract; cross-agent handoff fix, additive at the time — did not modify v2.2's naming/envelope rules. **Superseded by v2.4**, which physically inlines this contract as chapter 12. |
| [v2.2](spec/AIX-v2.2.md) | Ratified | May 2026 | Module manifest standard, `format` replaces `documentType`, Published Works registry, UMA Platform keys. **Superseded by v2.4**, which physically inlines this document. |
| [v2.1](spec/AIX-v2.1.md) | Draft | April 2026 | Media Collections, Agent Behavioral Modes, Soul Documents |
| [v2.0](spec/AIX-v2.0.md) | Draft | March 2026 | Merged interchange + naming convention into single spec |
| v1.1 | Draft | February 2026 | `content_authoring` task type |
| v1.0 | Draft | December 2025 | Initial specification |

**[→ Read the full specification](spec/AIX-Standard.md)**

---

## What's New in 2.4

- **One converged standard** — AIX Standard v2.2, the AIX 2.3 Canonical Registry Contract, and the UMA 3.0 Contract Meta-Framework's contract structure (its §3, in full) and key families are physically inlined into this single document. One standard, one version, always — a shared version number across physically separate files was structurally what produced version divergence before this release.
- **Module Manifest Standard** — promoted from a subsection to its own chapter (§5); manifest schema, and the `permissionScopes`-vs-canonical-registry-grant relationship stated explicitly for the first time (composable, not merged).
- **Canonical Registry Contract** (§12, new) — scoped, capability-gated, auditable reads for canonical registries: capability, tier, lifetime, resource-scope, and obligations axes; the formal grant tuple; an audit contract; a cross-agent handoff contract.
- **UMA 3.0 Primitive Contract Structure** (§13, new) — registers the sixteen-primitive registry, the contract shape, and the AIX key hierarchy for UI primitives (primitive, telemetry, billing, tier-gated-capability key families). Required subsection content is in the standard, and §13.2.13 states the conformance file's shape: a JSON file at `docs/aix/conformance/{primitiveId}.test.json`, its minimum `required` scenarios and its pass/fail semantics. The companion remains the source for the test-specification format, the deprecation procedure, and implementation-specific monetization extensions and worked platform examples.
- **Newsletter Studio keys corrected** — `publishingModule_emailStudio_*` deprecated in favor of `publishingModule_newsletterStudio_*`, the form that actually derives from the module's `app_id` under the standard's own slug-to-camelCase rule; `htmlExport` ratified as a new key.
- **Deprecation policy** (§7.8, new) — AIX's first formally stated deprecation policy.

---

## Key Sections

| Section | What's There |
| --- | --- |
| [§2 Core Naming Principles](spec/AIX-Standard.md#2-core-naming-principles) | camelCase, hierarchy, no abbreviations, six validation tests |
| [§3 Persona Field Registry](spec/AIX-Standard.md#3-persona-field-registry) | All persona fields with Schema.org or AIX extension designation |
| [§4 Component & Agent Registry](spec/AIX-Standard.md#4-component--agent-registry) | Registered components, behavioral modes, Soul Documents, module manifests |
| [§5 Module Manifest Standard](spec/AIX-Standard.md#5-module-manifest-standard) | Module key hierarchy, manifest schema, `permissionScopes`-vs-registry-grant relationship |
| [§6 Reserved Segment Vocabulary](spec/AIX-Standard.md#6-reserved-segment-vocabulary) | Format identifiers, quality pass identifiers, property suffixes |
| [§7 Canonical Key Reference](spec/AIX-Standard.md#7-canonical-key-reference) | All ratified AIX keys, UMA primitive/telemetry/billing keys, deprecation policy |
| [§8 AIX Envelope & Task Types](spec/AIX-Standard.md#8-aix-envelope--task-types) | Interchange format, task schema, metadata |
| [§9 Task-Specific Schemas](spec/AIX-Standard.md#9-task-specific-schemas) | Per-task input/output schemas — voice, image, text, video, code, `content_authoring` |
| [§10 Workflows & Chaining](spec/AIX-Standard.md#10-workflows--chaining) | Multi-step pipeline with variable substitution |
| [§11 Provider Comparison Format](spec/AIX-Standard.md#11-provider-comparison-format) | Side-by-side output schema across providers |
| [§12 Canonical Registry Contract](spec/AIX-Standard.md#12-canonical-registry-contract) | Scoped, capability-gated, auditable reads for canonical registries |
| [§13 UMA 3.0 Primitive Contract Structure](spec/AIX-Standard.md#13-uma-30-primitive-contract-structure) | Primitive registry, contract structure, AIX key hierarchy for UI primitives |
| [§14 Security & Governance](spec/AIX-Standard.md#14-security--governance) | Auth, safety, versioning, amendment process |

---

## Relationship to Other Standards

| Standard | Relationship |
| --- | --- |
| **MCP** (Model Context Protocol) | Complementary — MCP handles inputs, AIX handles outputs |
| **Schema.org** | Foundation — AIX uses Schema.org canonical names as the base layer |
| **JSON Schema** | Implementation — AIX uses JSON Schema for field validation |
| **OpenAI Assistants API** | AIX behavioral modes are provider-neutral; Assistants have instructions but no standard mode vocabulary |

---

## Contributing

1. Open an issue describing the new key, field, or feature
2. Run the [six naming validation tests](spec/AIX-Standard.md#26-naming-validation-tests) against any proposed keys
3. Submit a pull request with changes to the spec file
4. Keys are ratified in the next minor version per the [amendment process](spec/AIX-Standard.md#143-amendment-process)

Retroactive changes to ratified keys are prohibited. Deprecation is handled by adding a replacement key and marking the old key `[DEPRECATED]` — not by renaming it.

---

## License

Apache 2.0 · Copyright 2026 Michael Tchong, Ubertrends LLC

*AIX heritage: [CrossARM (2014)](https://ubertrends.com) · Ubertrends LLC · Solo Trillion*

---

## Provenance

Everything above this section is maintained by hand. This section is generated
with each release, and the standard itself carries no provenance of its own: a
document that describes its own build has bytes that depend on how it was
built, and those bytes are what implementers compare.

**Version 2.4 · August 2026**

| | |
| --- | --- |
| Bundle | `f429f7349c856a2c751b75cb811d74c2` |
| Canonical sha256 | `264065a4d7aee8d728d9d66f83a115f59c6b99ca4f1cc65d265730189dd9c873` (sha256 of the canonical standard this release was derived from) |
| Derived from | `7b65491d657da9fb52fcc6f532a54136c0a133fb12ba09c266c08b28c953fd70` (sha256 of `spec/AIX-Standard.md`) |

`spec/AIX-Standard.md` is derived from a single canonical source by removing the
internal-only sections that source marks. No copy of the standard is edited by
hand, and no copy is authoritative on its own: if two disagree, the derivation
is wrong and the canonical source settles it. `aix-publication.json` carries the
same identifiers in machine-readable form. The earlier versions under `spec/` are
historical records of what was ratified, and no release regenerates them. The
bundle identifier changes whenever any input to a release changes, so two
releases of the same version can carry different bundle identifiers.
