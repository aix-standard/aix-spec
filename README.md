# AIX — AI Interchange Standard

**Version 2.0 Draft · April 2026**
Apache 2.0 License · Open Standard · [aix-standard.org](https://aix-standard.org)

---

## The Problem

The generative AI revolution has produced hundreds of specialized tools — each with its own API format, authentication method, and response structure. This fragmentation creates:

- **Developer burden** — custom integration code for every provider
- **Vendor lock-in** — switching providers requires rewriting applications
- **Workflow friction** — chaining tools requires format conversion at every step
- **No comparability** — no standard way to compare outputs across providers

A parallel problem exists inside AI agents: without a naming standard, every team invents its own key vocabulary. Keys proliferate without governance — some snake_case, some camelCase, some abbreviated, some redundant — until the system becomes unmaintainable and incompatible with external implementers.

## The Solution

AIX addresses both problems with a single specification:

- Any AIX-compliant tool can **accept standardized inputs** and **return standardized outputs**
- Tools can be **swapped** with any other tool in the same category
- Tools **chain seamlessly** in multi-step workflows
- Every field, key, and identifier is **unambiguous, hierarchical, and self-documenting**

## Relationship to Existing Standards

| Standard | Focus | AIX Relationship |
|---|---|---|
| **MCP** (Model Context Protocol) | Connecting AI models to data sources | Complementary — MCP handles context inputs, AIX handles interchange outputs |
| **Schema.org** | Structured web data vocabulary | Foundation — AIX uses Schema.org canonical names as its base layer |
| **JSON Schema** | Data validation format | Implementation — AIX uses JSON Schema for all field validation |
| **OpenAPI** | REST API description | Compatible — AIX task schemas can be expressed as OpenAPI components |

---

## Quick Example

```json
{
  "aix": "2.0",
  "task": {
    "type": "text_generation",
    "provider": "anthropic",
    "model": "claude-sonnet-4-20250514",
    "input": {
      "prompt": "Summarize the following document in three sentences.",
      "context": "..."
    }
  },
  "persona": {
    "agentName": "writingAgent",
    "voiceProfile": "authoritative, concise, no em dashes"
  },
  "metadata": {
    "requestId": "aix_req_20260418_001",
    "timestamp": "2026-04-18T00:00:00Z",
    "cost": { "currency": "USD", "amount": 0.0024 },
    "latency": { "ms": 1240 },
    "tokens": { "input": 512, "output": 128 }
  }
}
```

---

## Table of Contents

| # | Section |
|---|---|
| 1 | [Introduction](#1-introduction) |
| 2 | [Core Naming Principles](#2-core-naming-principles) |
| 3 | Persona Field Registry |
| 4 | Agent & Component Registry |
| 5 | Reserved Segment Vocabulary |
| 6 | Canonical Key Reference |
| 7 | Task Interchange Format |
| 8 | Task Type Schemas |
| 9 | Workflow Chaining |
| 10 | Validation & Testing |
| 11 | Amendment Process |
| 12 | Provider Registry |
| 13 | Changelog |

---

## 1. Introduction

### 1.1 Purpose

AIX (AI Interchange) is an open specification for naming, exchanging data, and establishing interoperability between generative AI tools, agents, and applications.

The specification has two equal objectives:

1. **Interchange format** — A standardized JSON envelope for sending tasks to AI providers and receiving structured responses, enabling tool substitution and multi-step workflow chaining without custom integration code.

2. **Naming convention** — A hierarchical, self-documenting grammar for all field names, keys, and identifiers used within AI agent systems, eliminating the ambiguity and key sprawl that accumulates inside production agent architectures.

Both objectives are inseparable. An interchange format without a naming standard produces well-structured data with poorly governed keys. A naming standard without an interchange format governs internal architecture but cannot cross system boundaries.

### 1.2 Scope

AIX covers the following generative AI task types:

- Text generation and content authoring
- Voice synthesis and transcription
- Image generation and analysis
- Video generation
- Code generation and execution
- Multi-step workflow chaining across any combination of the above

AIX does not define model training, fine-tuning procedures, or provider-specific authentication mechanisms beyond what is required to construct a valid AIX request envelope.

### 1.3 Heritage

AIX inherits directly from **CrossARM**, a structured social media data interchange format developed by Michael Tchong at Ubertrends LLC in 2014. CrossARM introduced canonical field naming for social platform data — including `facebookName`, `twitterName`, and `engagementRate` — at a time when every analytics vendor invented its own vocabulary.

The same fragmentation problem CrossARM addressed in social data, AIX addresses in generative AI.

### 1.4 Relationship to Schema.org

AIX uses Schema.org as its canonical naming foundation. Where a Schema.org term exists for a concept, AIX uses that term without modification. Where no Schema.org term exists, AIX defines an extension using the `aix:` namespace prefix.

This layered approach means:

- AIX fields are recognizable to any developer familiar with structured data
- AIX extensions are clearly distinguishable from base Schema.org fields
- Future Schema.org additions can be adopted into AIX without breaking changes

### 1.5 Governance

AIX is stewarded by **Ubertrends LLC** under the `aix-steward` GitHub account and the `aix-standard` GitHub organization. The specification is licensed Apache 2.0. Contributions, amendments, and extensions follow the process defined in Section 11.

Official channels:
- Specification: [aix-standard.org](https://aix-standard.org)
- Repository: [github.com/aix-standard/aix-spec](https://github.com/aix-standard/aix-spec)
- Contact: aix@aix-standard.org

---

## 2. Core Naming Principles

### 2.1 The Naming Problem

Without a naming standard, AI agent systems develop key sprawl: `user_name`, `userName`, `username`, `personName`, `agentId`, `agent_id`, `agentIdentifier` — all representing the same concept, in different files, written by different contributors, on different days.

Key sprawl has compounding costs:

- Developers cannot predict field names without consulting documentation
- Data cannot be joined across subsystems without transformation
- New contributors make inconsistent choices that persist in production
- External consumers cannot implement against an unstable vocabulary

AIX eliminates key sprawl through a strict, learnable grammar applied to every field name in the system.

### 2.2 The Four-Segment Grammar

Every AIX key follows this pattern:

```
agentName_objectType_identifier_property
```

| Segment | Role | Example |
|---|---|---|
| `agentName` | Which agent owns this key | `writingAgent`, `rosie`, `personaAgent` |
| `objectType` | What kind of thing is being described | `article`, `voiceProfile`, `outputRule` |
| `identifier` | Which specific instance | `draft`, `final`, `primary` |
| `property` | What attribute of that instance | `title`, `wordCount`, `status` |

Segments are joined with underscores. Each segment uses camelCase internally — never snake_case, never PascalCase.

**Valid:** `writingAgent_article_draft_wordCount`
**Invalid:** `writing_agent_article_draft_word_count`
**Invalid:** `WritingAgent_Article_Draft_WordCount`

Not every key requires all four segments. A key uses the minimum number of segments needed to be unambiguous in context.

**Valid two-segment key:** `agentName_voiceProfile`
**Valid three-segment key:** `writingAgent_article_status`

### 2.3 camelCase Is Mandatory

All segment values use camelCase. This is not a style preference — it is a validation requirement.

Rationale:

- Underscore is the segment separator. Allowing underscores within a segment value makes parsing ambiguous.
- camelCase produces self-documenting compound terms: `voiceProfile` is immediately readable; `voiceprofile` is not.
- Schema.org uses camelCase throughout. AIX alignment with Schema.org requires matching case convention.

### 2.4 No Abbreviations

AIX prohibits abbreviations in key names. Every segment value must be spelled in full.

**Valid:** `writingAgent_outputRule_compliance_status`
**Invalid:** `wa_or_comp_stat`

Abbreviations require shared context to decode. A key that is self-documenting to the original author becomes opaque to a new contributor six months later and unreadable to an external implementer entirely.

The only exception is widely recognized initialisms that have entered common use as standalone words: `url`, `id`, `api`. These may be used in lowercase as segment values where appropriate.

### 2.5 Identifier Names the Format, Properties Define It

When an identifier segment names a document type or format, the property segment must not repeat that name.

**Valid:** `writingAgent_article_draft_wordCount` (identifier = `draft`, property = `wordCount`)
**Invalid:** `writingAgent_article_documentDraft_draftWordCount` (redundant — `draft` appears in both segments)

The identifier segment names the instance. The property segment describes an attribute of that instance. They operate at different levels and must not overlap.

### 2.6 Validation Tests

Every proposed AIX key must pass all six validation tests before ratification.

---

**Test 1 — Stranger Test**

> Could a developer who has never seen this codebase read this key and know what it means?

A key fails the Stranger Test if it requires internal knowledge, tribal context, or documentation lookup to interpret. The key must be self-documenting from its segments alone.

---

**Test 2 — Hierarchy Test**

> Does the key correctly communicate ownership and specificity from left to right?

The leftmost segment must be the most general (agent or system owner). Each subsequent segment must narrow scope. A key fails the Hierarchy Test if a more specific segment appears before a less specific one.

---

**Test 3 — Abbreviation Test**

> Does the key contain any abbreviations or truncated terms?

A key fails the Abbreviation Test if any segment value is shortened from its full form. Run the test by asking: does expanding this term produce a different, more complete word? If yes, the key fails.

---

**Test 4 — Case Test**

> Is every segment value in camelCase?

A key fails the Case Test if any segment value contains an underscore, begins with a capital letter, or is written in all lowercase as a compound term.

---

**Test 5 — Redundancy Test**

> Does any segment value appear in substance in more than one segment?

A key fails the Redundancy Test if two segments carry the same semantic content, even if worded differently. `article_document` fails because `article` and `document` refer to the same object. `draft_draftStatus` fails because `draft` is repeated.

---

**Test 6 — Conflict Test**

> Does this key conflict with any existing ratified AIX key?

A key fails the Conflict Test if it is a synonym, near-synonym, or alternative representation of an already-ratified key. Before ratifying a new key, the full canonical key reference (Section 6) must be checked for conflicts.

---

> **Full specification coming soon — watch this repo.**
> Sections 3–13 cover the Persona Field Registry, Agent & Component Registry, Reserved Segment Vocabulary, Canonical Key Reference, Task Interchange Format, Task Type Schemas, Workflow Chaining, Validation & Testing, Amendment Process, Provider Registry, and Changelog.

---

## Heritage

AIX descends from CrossARM (2014), a social media data interchange standard developed by Michael Tchong at Ubertrends LLC. CrossARM addressed the same fragmentation problem in social analytics that AIX addresses in generative AI.

## Governance

The AIX specification is stewarded by Ubertrends LLC. Contributions are welcome via pull request. The amendment process (Section 11) governs versioning, deprecation, and ratification. No retroactive changes to ratified keys.

## License

Copyright 2026 Michael Tchong, Ubertrends LLC

Licensed under the Apache License, Version 2.0. You may obtain a copy at:
https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.

---

*AIX · aix-standard.org · aix@aix-standard.org*
