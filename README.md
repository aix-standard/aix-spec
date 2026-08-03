# AIX — AI Interchange Standard

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Version](https://img.shields.io/badge/version-2.3-green.svg)](spec/AIX-v2.2.md)
[![Status](https://img.shields.io/badge/status-ratified-brightgreen.svg)](spec/AIX-v2.2.md)

**An open standard for naming, exchanging data, and establishing interoperability between generative AI tools, agents, and applications.**

Website: [aix-standard.org](https://aix-standard.org) · Contact: [aix@aix-standard.org](mailto:aix@aix-standard.org)

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
  "aix": "2.2",
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
publishingModule_emailStudio_studioJson
```

---

## Specification

| Version | Status | Date | Summary |
| --- | --- | --- | --- |
| [v2.3](spec/aix-2.3-canonical-registry-contract.md) | Ratified | August 2026 | Canonical Registry Contract addendum — capability, tier, lifetime, resource-scope, and obligations axes for scoped agent reads; grant tuple; audit contract; cross-agent handoff fix. Additive — does not modify v2.2's naming/envelope rules. |
| [v2.2](spec/AIX-v2.2.md) | Ratified | May 2026 | Module manifest standard, `format` replaces `documentType`, Published Works registry, UMA Platform keys |
| [v2.1](spec/AIX-v2.1.md) | Draft | April 2026 | Media Collections, Agent Behavioral Modes, Soul Documents |
| [v2.0](spec/AIX-v2.0.md) | Draft | March 2026 | Merged interchange + naming convention into single spec |
| v1.1 | Draft | February 2026 | `content_authoring` task type |
| v1.0 | Draft | December 2025 | Initial specification |

**[→ Read the full v2.2 specification](spec/AIX-v2.2.md)**

---

## What's in v2.2

- **Module Manifest Standard** — Formally ratifies the `{moduleType}Module_{moduleId}_{property}` hierarchy for the Unified Module Architecture (UMA) platform layer
- **`format` replaces `documentType`** — `writingAgent_format_*` is now canonical; `writingAgent_documentType_*` is deprecated with a full migration guide
- **Published Works registry** — 10 new fields for book chapters, sidebars, and long-form structured works
- **UMA Platform canonical keys** — `umaModule_registry_manifest`, `umaModule_entitlement_status`, and more
- **FutureSight Studio module keys** — `publishingModule_emailStudio_*` keys ratified
- **Naming clarifications** — Slug/URL casing exception documented; schema version string format clarified; metadata JSON examples corrected to camelCase throughout

---

## Key Sections

| Section | What's There |
| --- | --- |
| [§2 Core Naming Principles](spec/AIX-v2.2.md#2-core-naming-principles) | camelCase, hierarchy, no abbreviations, six validation tests |
| [§3 Persona Field Registry](spec/AIX-v2.2.md#3-persona-field-registry) | All persona fields with Schema.org or AIX extension designation |
| [§4 Component & Agent Registry](spec/AIX-v2.2.md#4-component--agent-registry) | Registered components, behavioral modes, Soul Documents, module manifests |
| [§5 Reserved Segment Vocabulary](spec/AIX-v2.2.md#5-reserved-segment-vocabulary) | Format identifiers, quality pass identifiers, property suffixes |
| [§6 Canonical Key Reference](spec/AIX-v2.2.md#6-canonical-key-reference) | All ratified AIX keys + deprecated key registry |
| [§7–8 Envelope & Task Schemas](spec/AIX-v2.2.md#7-aix-envelope--task-types) | Task types, metadata, error object, per-task input/output schemas |
| [§9 Workflows & Chaining](spec/AIX-v2.2.md#9-workflows--chaining) | Multi-step pipeline with variable substitution |

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
2. Run the [six naming validation tests](spec/AIX-v2.2.md#26-naming-validation-tests) against any proposed keys
3. Submit a pull request with changes to the relevant spec file
4. Keys are ratified in the next minor version per the [amendment process](spec/AIX-v2.2.md#113-amendment-process)

Retroactive changes to ratified keys are prohibited. Deprecation is handled by adding a replacement key and marking the old key `[DEPRECATED]` — not by renaming it.

---

## License

Apache 2.0 · Copyright 2026 Michael Tchong, Ubertrends LLC

*AIX heritage: [CrossARM (2014)](https://ubertrends.com) · Ubertrends LLC · Solo Trillion*
