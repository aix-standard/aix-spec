# AIX — AI Interchange Standard · v2.2

**Version 2.2 · May 2026**
**Author:** Michael Tchong, Ubertrends LLC · **License:** Apache 2.0

*AIX is an open standard for naming, exchanging data, and establishing interoperability between generative AI tools, agents, and applications. This document merges the AIX Interchange Format Specification and the AIX Naming Convention Standard into a single authoritative reference.*

*Heritage: CrossARM (2014) · Ubertrends LLC · Solo Trillion*

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Core Naming Principles](#2-core-naming-principles)
3. [Persona Field Registry](#3-persona-field-registry)
   - [3.5 Media Collections](#35-media-collections)
4. [Component & Agent Registry](#4-component--agent-registry)
   - [4.7 Agent Behavioral Modes](#47-agent-behavioral-modes)
   - [4.8 Module Manifest Standard](#48-module-manifest-standard)
5. [Reserved Segment Vocabulary](#5-reserved-segment-vocabulary)
6. [Canonical Key Reference](#6-canonical-key-reference)
7. [AIX Envelope & Task Types](#7-aix-envelope--task-types)
8. [Task-Specific Schemas](#8-task-specific-schemas)
9. [Workflows & Chaining](#9-workflows--chaining)
10. [Provider Comparison Format](#10-provider-comparison-format)
11. [Security & Governance](#11-security--governance)
12. [Appendix](#12-appendix)
13. [Changelog](#13-changelog)

---

## 1. Introduction

### 1.1 The Problem

The generative AI revolution has produced hundreds of specialized tools — each with its own API format, authentication method, and response structure. This fragmentation creates developer burden through custom integration code for each provider, vendor lock-in when switching providers requires rewriting applications, workflow friction when chaining tools requires format conversion, and no standard way to compare outputs across providers.

A parallel problem exists inside AI agents themselves: without a naming standard, every team invents its own key vocabulary for fields, configuration properties, and persona data. Keys proliferate without governance — some snake_case, some camelCase, some abbreviated, some redundant — until the system becomes unmaintainable and incompatible with external implementers.

### 1.2 The Solution

AIX addresses both problems with a single specification. It defines a standard envelope for AI task requests and responses so any AIX-compliant tool can accept standardized inputs, return standardized outputs, be swapped with any other tool in the same category, and chain seamlessly with other AIX tools in workflows. It also defines a canonical naming convention so every field, key, and identifier in the AIX ecosystem is unambiguous, hierarchical, and implementable by developers with no internal context.

### 1.3 Heritage: CrossARM (2014)

AIX follows the pattern established by CrossARM, a data interchange standard developed by Ubertrends LLC in 2014 for cross-platform marketing data exchange. CrossARM's governing principle was: use Schema.org canonical names where they exist, and invent proprietary extensions for what Schema.org does not cover.

```
// CrossARM 2014 — Schema.org base vocabulary
givenName, familyName, worksFor, address, email, telephone  (Person)
brand, legalName, duns, isicV4                              (Organization)

// CrossARM 2014 — Proprietary extensions (Schema.org had no equivalent)
facebookName, twitterName, twitterFollowers, engagementRate
uniqueVisits, timeOnSite, bounceRate, influencerRating
```

AIX v2.0 formalizes this pattern. Schema.org canonical names are used wherever applicable. Fields that Schema.org does not cover are defined as AIX extensions.

### 1.4 Relationship to Existing Standards

| Standard | Focus | AIX Relationship |
| --- | --- | --- |
| MCP (Model Context Protocol) | Connecting AI to data sources | Complementary — MCP handles inputs, AIX handles outputs |
| Schema.org | Structured web data vocabulary | Foundation — AIX uses Schema.org canonical names as base layer |
| Eden AI Schemas | Traditional AI APIs (OCR, NLP) | Predecessor — AIX extends into generative AI task types |
| JSON Schema | Data validation | Implementation — AIX uses JSON Schema for field validation |
| CrossARM (2014) | Cross-platform marketing data | Heritage — established the Schema.org base + extension pattern |

---

## 2. Core Naming Principles

These principles govern every field name, key, and identifier in the AIX specification and in any AIX-compliant implementation.

### 2.1 camelCase Throughout

All field names and identifiers use camelCase. No snake_case, no kebab-case, no PascalCase for field names.

```
// ✓ Correct
givenName  familyName  jobTitle  telephoneMobile  linkedinUrl

// ✗ Incorrect
given_name    // snake_case
given-name    // kebab-case
GivenName     // PascalCase
```

**Layer-specific exceptions:**

- **Database columns** use snake_case by PostgreSQL convention. API responses, JSON exports, and all specification field names use camelCase. ORM/serialization layers handle the translation (`given_name → givenName`).
- **Slugs and URLs** use dash-case (`futuresight-studio`, `rss-discovery`). Slugs are identifiers in routing and storage contexts, not AIX field names, and follow web convention.
- **Schema version strings** use the format `{type}-v{N}` (`studio-json-v1`, `canvas-document-v1`). These are opaque version identifiers, not field keys, and are exempt from the camelCase rule.

### 2.2 Hierarchical Namespace

Every internal AIX identifier follows a strict four-part hierarchy. Underscores serve exclusively as namespace separators — they are not snake_case within a segment.

```
{component}_{objectType}_{identifier}_{property}

writingAgent_format_default_systemPrompt
writingAgent_format_newsletter_outputRules
researchAgent_qualityPass_hallucinationSweep_systemPrompt
personaAgent_voice_brand
publishingModule_emailStudio_studioJson
```

### 2.3 Schema.org Base + AIX Extensions

| Layer | Example Fields | Rule |
| --- | --- | --- |
| Schema.org canonical | givenName, familyName, jobTitle, legalName, telephone, email | Use as-is, no modification |
| AIX extensions | telephoneMobile, emailBusiness, linkedinUrl | Only when Schema.org has no equivalent |
| AIX hierarchy keys | writingAgent_format_default_systemPrompt | Four-part camelCase hierarchy for agent/config keys |

### 2.4 No Abbreviations

All segments use full English words. Permitted acronyms: `aix`, `api`, `url`, `html`, `json`, `uuid`, `naics`, `rss`, `uma`

| ✗ Prohibited | ✓ Required |
| --- | --- |
| liUrl | linkedinUrl |
| sp | systemPrompt |
| docType | documentType |
| qPass | qualityPass |

### 2.5 Identifier Names the Format — Properties Define It

The identifier segment (position 3) uses a short, stable handle. Never encode behavior into the identifier name.

```
// ✓ Correct
writingAgent_format_newsletter_systemPrompt

// ✗ Wrong — behavior encoded into identifier
writingAgent_format_formattedNewsletterWithCuratedSections_systemPrompt
```

### 2.6 Naming Validation Tests

Before any new key is ratified, it must pass all six tests:

| Test | Question | Fail Condition |
| --- | --- | --- |
| 1. Stranger Test | Could a developer with no internal context understand this key? | Requires docs to parse |
| 2. Hierarchy Test | Does the key follow the four-part hierarchy? | Missing segment when more specificity exists |
| 3. Abbreviation Test | Does the key contain prohibited abbreviated segments? | Any segment not a full word or registered acronym |
| 4. Case Test | Is every segment in camelCase? | Any snake_case, kebab-case, or UPPER_SNAKE within a segment |
| 5. Redundancy Test | Does any segment restate an adjacent segment? | e.g. `documentType_document` — concept repeated |
| 6. Conflict Test | Does the key duplicate an existing canonical key? | Duplicate or near-duplicate of a ratified key |

---

## 3. Persona Field Registry

The Persona Field Registry defines every field in the AIX identity vault. Fields are either Schema.org canonical properties (used as-is) or AIX extensions.

### 3.1 Essentials

| AIX Field Key | Source | Description | Example |
| --- | --- | --- | --- |
| givenName | Schema.org | Legal first name | Michael |
| familyName | Schema.org | Legal last name | Tchong |
| preferredName | AIX extension | Preferred name or nickname | Mike |
| jobTitle | Schema.org | Professional title | Founder & CEO |
| worksFor | Schema.org | Primary organization name | Ubertrends LLC |
| industry | AIX extension | Industry or sector | Technology Consulting |
| addressHome | AIX extension | Home or personal display location | Las Vegas, NV |
| timezone | AIX extension | IANA timezone identifier | America/Los_Angeles |

### 3.2 Professional

| AIX Field Key | Source | Description |
| --- | --- | --- |
| activeProject1 | AIX extension | Current primary project name and one-line description |
| activeProject2 | AIX extension | Current secondary project |
| currentGoal1 | AIX extension | Active goal with target date |
| currentGoal2 | AIX extension | Active goal with target date |
| currentGoal3 | AIX extension | Active goal with target date |
| expertise | AIX extension | Core areas of expertise |
| toolsUsed | AIX extension | Primary tools and platforms currently in use |
| resume | AIX extension | Short professional bio or resume summary |

### 3.3 Organization

> **`legalName` is a shared field** defined authoritatively in §3.4. It appears here because Organization context is where most implementations first encounter it.

| AIX Field Key | Source | Description |
| --- | --- | --- |
| legalName | Schema.org | Full legal name of the organization — see §3.4 for authoritative definition |
| description | Schema.org | Short organization description (elevator pitch) |
| companyBackgrounder | AIX extension | Longer organization backgrounder for press or partners |
| productServiceDescription1 | AIX extension | Primary product or service description |
| productServiceDescription2 | AIX extension | Secondary product or service description |
| url | Schema.org | Primary website URL |
| websiteUrl2 | AIX extension | Secondary website URL |

### 3.4 Financial & Legal

| AIX Field Key | Source | Description |
| --- | --- | --- |
| legalName | Schema.org | Legal entity name |
| entityType | AIX extension | Legal entity type (LLC, Corp, Sole Prop, etc.) |
| foundingDate | Schema.org | Date organization was founded or incorporated |
| naics | Schema.org | NAICS industry classification code |
| taxCategory | AIX extension | Tax classification (e.g., Disregarded Entity, S-Corp) |
| addressBusiness | AIX extension | Full business mailing address |
| addressHome | AIX extension | Home or personal display location |
| bank | AIX extension | Primary banking institution name |
| references | AIX extension | Professional references |
| telephoneBusiness | AIX extension | Business/office telephone number |
| telephoneMobile | AIX extension | Mobile telephone number |

> **AIX Disambiguation Pattern** — Schema.org has single-value properties for address, telephone, and email. AIX extends each with a qualifier suffix: `addressBusiness`/`addressHome`, `telephoneBusiness`/`telephoneMobile`, `emailBusiness`/`emailPersonal`.

### 3.5 Media Collections

AIX v2.1 introduced the `media` array: a typed collection of the author's media surfaces.

#### Field Definitions

| AIX Field Key | Source | Type | Required | Description |
| --- | --- | --- | --- | --- |
| media | AIX extension | array | No | Ordered collection of Medium objects |
| mediumType | AIX extension | string | Yes (per entry) | Controlled vocabulary — see below |
| mediumName | AIX extension | string | Yes (per entry) | User-facing display name |
| mediumBrand | AIX extension | string | No | Optional umbrella brand |
| mediumTagline | AIX extension | string | No | Per-medium tagline |
| mediumUrl | AIX extension | string (URL) | No | Canonical URL for this surface |
| isPrimary | AIX extension | boolean | Yes (per entry) | Exactly one entry must be true |

> **`mediumName` vs. `worksFor`** — `worksFor` names the organization (`"Ubertrends LLC"`). `mediumName` names the media surface (`"Ubertrends"`).

#### `mediumType` Controlled Vocabulary

| Value | Description |
| --- | --- |
| `newsletter` | Email-delivered periodic publication |
| `blog` | Web-native long-form writing surface |
| `podcast` | Audio episodic series |
| `channel` | Video episodic series — platform-neutral |
| `social` | Broadcast-style social presence |
| `platform` | Product or service where content appears |
| `publication` | Formal editorial container |
| `community` | Membership or discussion space |
| `other` | Any surface not covered above |

Implementations MUST accept unknown `mediumType` values without error.

#### Template Substitution Resolution Rule

| Step | Condition | Action |
| --- | --- | --- |
| 1 | Request specifies `mediumId` or `mediumType` | Substitute from matching `media[]` entry |
| 2 | No request-level override | Substitute from `isPrimary: true` entry |
| 3 | No primary entry | Substitute from `media[0]` |
| 4 | `media[]` empty or absent | Fall back to `worksFor` for `{mediumName}` |

This rule is normative. All AIX-compliant Writing Agents must implement it identically.

#### Reference Implementation

```json
"media": [
  {
    "mediumType": "newsletter",
    "mediumName": "Ubertrends",
    "mediumBrand": "FutureSight",
    "mediumTagline": "America's Most Influential Trend Spotter",
    "mediumUrl": "https://ubertrends.com",
    "isPrimary": true
  },
  {
    "mediumType": "platform",
    "mediumName": "Solo Trillion",
    "mediumUrl": "https://solotrillion.ai",
    "isPrimary": false
  }
]
```

### 3.6 Social Media

| AIX Field Key | Source | Description | Format |
| --- | --- | --- | --- |
| linkedinUrl | AIX extension | LinkedIn profile URL | https://linkedin.com/in/handle |
| twitterUrl | AIX extension | X/Twitter profile URL | https://x.com/handle |
| facebookUrl | AIX extension | Facebook profile or page URL | https://facebook.com/handle |
| instagramUrl | AIX extension | Instagram profile URL | https://instagram.com/handle |
| youtubeUrl | AIX extension | YouTube channel URL | https://youtube.com/@handle |
| tiktokUrl | AIX extension | TikTok profile URL | https://tiktok.com/@handle |
| substackUrl | AIX extension | Substack publication URL | https://name.substack.com |

### 3.7 Contact (Email)

| AIX Field Key | Source | Description |
| --- | --- | --- |
| email | Schema.org | Primary email address |
| emailBusiness | AIX extension | Business email address |
| emailPersonal | AIX extension | Personal email address |

### 3.8 Preferences

| AIX Field Key | Source | Description |
| --- | --- | --- |
| communicationStyle | AIX extension | Preferred communication style with this agent |
| checkinFrequency | AIX extension | How frequently and in what format to receive check-ins |
| interests | AIX extension | Personal and professional interests |
| offLimitsTopics | AIX extension | Topics or subjects to avoid in generated content |
| responseFormat | AIX extension | Preferred response format (prose, bullets, structured, etc.) |

### 3.9 Conversational Agent

| AIX Field Key | Source | Description |
| --- | --- | --- |
| conversationalAgentName | AIX extension | Display name of the conversational agent. Default: Rosie |
| conversationalAgentVoiceId | AIX extension | TTS voice identifier (e.g., ElevenLabs voice ID) |
| conversationalAgentSystemPrompt | AIX extension | System prompt defining the conversational agent persona |

### 3.10 Published Works

*New in v2.2.* Fields for authors who publish long-form structured works (books, field guides, reference manuals).

| AIX Field Key | Source | Description |
| --- | --- | --- |
| bookTitle | AIX extension | Full title of the book or long-form work |
| bookKey | AIX extension | Stable slug identifier for the book (dash-case) |
| chapterKey | AIX extension | Stable slug identifier for a chapter |
| chapterTitle | AIX extension | Display title of the chapter |
| chapterNumber | AIX extension | Sequential chapter number (integer) |
| chapterContent | AIX extension | Full prose content of the chapter |
| chapterSummary | AIX extension | Short summary for navigation and indexing |
| chapterSubsections | AIX extension | Array of subsection headings |
| contentType | AIX extension | `chapter` \| `sidebar` \| `appendix` \| `introduction` |
| parentChapterKey | AIX extension | For sidebars: the `chapterKey` of the containing chapter |

---

## 4. Component & Agent Registry

Components are the top-level namespace segment. New components require a spec amendment before use.

| Component Key | Full Name | Description |
| --- | --- | --- |
| researchAgent | Research Agent | Web research, source synthesis, hallucination detection |
| writingAgent | Writing Agent | Content generation, format management, output formatting |
| personaAgent | Persona Agent | Identity vault, voice, guidelines, guardrails |
| conversationalAgent | Conversational Agent | Conversational assistant (Telegram + web) — default name: Rosie |
| rosie | Rosie | Rosie-specific configuration keys. Use `conversationalAgent` for generic interchange. |
| orchestra | Orchestra | Platform-level orchestration, agent chaining, workflow config |
| aix | AIX Standard | Specification-level fields (version, schema, envelope) |

### 4.7 Agent Behavioral Modes

#### 4.7.2 The behavioralMode Field

```json
{
  "aix": "2.2",
  "agent": {
    "id": "rosie",
    "role": "chief_of_staff",
    "behavioralMode": "observer",
    "allowedModes": ["observer", "orchestrator", "briefing", "off"]
  }
}
```

#### 4.7.3 Standard Mode Vocabulary

| Mode | Description | Agent Behavior |
| --- | --- | --- |
| `observer` | Default monitoring state | Listens, classifies, surfaces signals. Does not initiate external actions without explicit trigger. |
| `orchestrator` | Active pipeline execution | Runs workflows, coordinates other agents, executes scheduled tasks. |
| `briefing` | Delivery state | Formats and delivers digests, summaries, and reports. Read-only relative to source data. |
| `off` | Silent state | No proactive outreach. Responds to direct queries only. |

#### 4.7.4 Mode Transitions

Mode transitions must be explicit. An agent must not self-transition based on conversation context alone. Mode changes require a direct user instruction, a scheduled trigger, or an orchestration signal from another AIX-compliant agent.

#### 4.7.6 Soul Documents

A Soul Document is a persistent identity file loaded before any system prompt, conversation history, or user input.

| Section | Required | Description |
| --- | --- | --- |
| Identity | Yes | Who the agent is and what role it serves |
| Role | Yes | Specific functions the agent performs |
| Constraints | Yes | Explicit list of what the agent never does |
| Behavioral Modes | Recommended | Mapping to §4.7.3 standard vocabulary |
| Tone Examples | Recommended | Compliant and non-compliant response examples |

### 4.8 Module Manifest Standard

*New in v2.2.* Formally ratifies the module manifest hierarchy for the Unified Module Architecture (UMA) platform layer.

#### 4.8.1 Module Key Hierarchy

```
{moduleType}Module_{moduleId}_{property}

publishingModule_emailStudio_studioJson
researchModule_rssDiscovery_feedUrl
```

**Module type segments:** `publishingModule`, `researchModule`, `writingModule`, `personaModule`

**Module ID segment:** The registered `moduleId` in camelCase (e.g., `emailStudio`, `rssDiscovery`). Note: the manifest's `moduleId` field stores the dash-case slug (`"futuresight-studio"`); the AIX hierarchy key uses its camelCase equivalent (`emailStudio`). These are the same module — different representations for different contexts.

#### 4.8.2 Module Manifest Schema

```json
{
  "aixVersion": "2.2",
  "moduleId": "futuresight-studio",
  "moduleType": "publishing",
  "displayName": "FutureSight Studio",
  "version": "1.0.0",
  "requires": ["writingAgent", "personaAgent"],
  "inputContracts": ["content_authoring"],
  "outputContracts": ["studio-json-v1"],
  "permissionScopes": ["persona:read", "writing:generate", "publishing:send"],
  "billing": { "model": "per_use", "unit": "generation" }
}
```

---

## 5. Reserved Segment Vocabulary

### 5.1 Object Type Segments (Position 2)

| Object Segment | Usage | Example |
| --- | --- | --- |
| format | Output format variants for the Writing Agent | writingAgent_format_default_systemPrompt |
| qualityPass | Error-check and verification passes | researchAgent_qualityPass_hallucinationSweep_systemPrompt |
| config | General agent operational configuration | researchAgent_config_default_systemPrompt |
| voice | Author identity and brand voice data | personaAgent_voice_brand |
| output | Generation output schema fields | writingAgent_output_draft_headline |
| input | Generation input schema fields | writingAgent_input_contentBrief_format |
| schema | JSON schema definitions | aix_schema_contentAuthoring_input |
| meta | Metadata (cost, latency, tokens) | aix_meta_cost_amount |

> **`documentType` deprecated:** Keys using `writingAgent_documentType_*` are deprecated in v2.2. New keys use `format`. See §6.6 for the full deprecated key registry and migration mapping.

### 5.2 Identifier Segments (Position 3)

#### Format Identifiers (under `writingAgent_format`)

| Identifier | What It Produces |
| --- | --- |
| default | Open-form output — structure defined by persona and config. Replaces `article` and `fieldGuide`. |
| meetingBrief | Condensed pre-meeting intelligence brief — structured, actionable |
| executiveBrief | Executive summary — structured for senior audience |
| competitiveAnalysis | Competitive analysis — structured comparison report |
| newsletter | Newsletter edition — curated, conversational |
| linkedinPost | LinkedIn post — compressed authority, plain text output |

#### Quality Pass Identifiers

| Identifier | Description |
| --- | --- |
| researchBuild | Initial data gathering and source synthesis |
| factCheck | Verify all claims against cited sources |
| hallucinationSweep | Detect and flag fabricated or unsupported content |
| complianceEdit | Apply guardrails, voice rules, and editorial standards |
| qualityScore | Final quality assessment and confidence scoring |
| optimizationPass | Refine output for performance, cost, or length targets |

#### Voice Identifiers (under `personaAgent_voice`)

| Identifier | Description |
| --- | --- |
| brand | Author brand voice DNA — tone, vocabulary, sentence rhythm, signature phrases |
| guidelines | Editorial writing structure and style rules |
| guardrails | Hard stops, non-negotiables, topics requiring human review |

### 5.3 Property Segments (Position 4)

| Property Suffix | Value Type | Description |
| --- | --- | --- |
| systemPrompt | string | LLM system prompt for the given context |
| outputRules | string | Formatting and structure rules for generated output |
| config | object | Configuration object |
| schema | object | JSON Schema definition |
| template | object | Complete template bundle (prompt + rules + metadata) |
| label | string | Human-readable display label |
| description | string | Human-readable description |
| version | string | Semantic version string |
| enabled | boolean | Feature flag |

---

## 6. Canonical Key Reference

### 6.1 Persona Agent — Voice

| Canonical Key | Type | Ver | Description |
| --- | --- | --- | --- |
| personaAgent_voice_brand | string | 1.1 | Author brand voice DNA |
| personaAgent_voice_guidelines | string | 1.1 | Editorial writing guidelines |
| personaAgent_voice_guardrails | string | 1.1 | Hard stops and non-negotiables |

### 6.2 Writing Agent — Format (v2.2)

| Canonical Key | Type | Ver | Description |
| --- | --- | --- | --- |
| writingAgent_format_default_systemPrompt | string | 2.2** | Default open-form system prompt |
| writingAgent_format_default_outputRules | string | 2.2** | Default open-form output rules |
| writingAgent_format_meetingBrief_systemPrompt | string | 2.2** | Meeting Brief system prompt |
| writingAgent_format_meetingBrief_outputRules | string | 2.2** | Meeting Brief output rules |
| writingAgent_format_executiveBrief_systemPrompt | string | 2.2** | Executive Brief system prompt |
| writingAgent_format_executiveBrief_outputRules | string | 2.2** | Executive Brief output rules |
| writingAgent_format_competitiveAnalysis_systemPrompt | string | 2.2** | Competitive Analysis system prompt |
| writingAgent_format_competitiveAnalysis_outputRules | string | 2.2** | Competitive Analysis output rules |
| writingAgent_format_newsletter_systemPrompt | string | 2.2** | Newsletter system prompt |
| writingAgent_format_newsletter_outputRules | string | 2.2** | Newsletter output rules |
| writingAgent_format_linkedinPost_systemPrompt | string | 2.2** | LinkedIn Post system prompt |
| writingAgent_format_linkedinPost_outputRules | string | 2.2** | LinkedIn Post output rules |
| writingAgent_config_default_systemPrompt | string | 2.0* | Default Writing Agent system prompt |
| writingAgent_config_default_outputRules | string | 2.0* | Default output rules |

### 6.3 Research Agent

| Canonical Key | Type | Ver | Description |
| --- | --- | --- | --- |
| researchAgent_config_default_systemPrompt | string | 1.2 | Research Agent default system prompt |
| researchAgent_qualityPass_researchBuild_systemPrompt | string | 2.0** | Pass 1: Research Build |
| researchAgent_qualityPass_factCheck_systemPrompt | string | 2.0** | Pass 2: Fact Check |
| researchAgent_qualityPass_hallucinationSweep_systemPrompt | string | 2.0** | Pass 3: Hallucination Sweep |
| researchAgent_qualityPass_complianceEdit_systemPrompt | string | 2.0** | Pass 4: Compliance Edit |
| researchAgent_qualityPass_qualityScore_systemPrompt | string | 2.0** | Pass 5: Quality Score |

### 6.4 UMA Platform Keys

*New in v2.2.*

| Canonical Key | Type | Ver | Description |
| --- | --- | --- | --- |
| umaModule_registry_manifest | object | 2.2** | Module manifest JSON object |
| umaModule_entitlement_status | string | 2.2** | Entitlement state: `active` \| `suspended` \| `pending` |
| umaModule_billing_ref | string | 2.2** | Billing reference ID for usage tracking |
| umaModule_usage_event | object | 2.2** | Metering event payload |

### 6.5 Publishing Module — FutureSight Studio

*New in v2.2.*

| Canonical Key | Type | Ver | Description |
| --- | --- | --- | --- |
| publishingModule_emailStudio_studioJson | object | 2.2** | AIX Canvas envelope for email Studio rendering |
| publishingModule_emailStudio_canvasDocument | object | 2.2** | Canvas document schema [RESERVED — v1.5] |
| publishingModule_emailStudio_publishRecord | object | 2.2** | Record of a completed publish operation |
| publishingModule_emailStudio_deliveryTarget | string | 2.2** | Target delivery address or list identifier |
| publishingModule_emailStudio_issueSubject | string | 2.2** | Email subject line for this issue |
| publishingModule_emailStudio_issueDate | string | 2.2** | Publication date (ISO-8601) |

### 6.6 Deprecated Keys

> Do not use for new implementations. Retained for backwards compatibility only.

| Deprecated Key | Replacement | Deprecated In |
| --- | --- | --- |
| writingAgent_documentType_article_systemPrompt | writingAgent_format_default_systemPrompt | 2.2 |
| writingAgent_documentType_article_outputRules | writingAgent_format_default_outputRules | 2.2 |
| writingAgent_documentType_fieldGuide_systemPrompt | writingAgent_format_default_systemPrompt | 2.2 |
| writingAgent_documentType_fieldGuide_outputRules | writingAgent_format_default_outputRules | 2.2 |
| writingAgent_documentType_meetingBrief_systemPrompt | writingAgent_format_meetingBrief_systemPrompt | 2.2 |
| writingAgent_documentType_meetingBrief_outputRules | writingAgent_format_meetingBrief_outputRules | 2.2 |
| writingAgent_documentType_executiveBrief_systemPrompt | writingAgent_format_executiveBrief_systemPrompt | 2.2 |
| writingAgent_documentType_executiveBrief_outputRules | writingAgent_format_executiveBrief_outputRules | 2.2 |
| writingAgent_documentType_competitiveAnalysis_systemPrompt | writingAgent_format_competitiveAnalysis_systemPrompt | 2.2 |
| writingAgent_documentType_competitiveAnalysis_outputRules | writingAgent_format_competitiveAnalysis_outputRules | 2.2 |
| writingAgent_documentType_newsletter_systemPrompt | writingAgent_format_newsletter_systemPrompt | 2.2 |
| writingAgent_documentType_newsletter_outputRules | writingAgent_format_newsletter_outputRules | 2.2 |
| writingAgent_documentType_linkedinPost_systemPrompt | writingAgent_format_linkedinPost_systemPrompt | 2.2 |
| writingAgent_documentType_linkedinPost_outputRules | writingAgent_format_linkedinPost_outputRules | 2.2 |
| writingAgent_template_* | writingAgent_format_* | 2.0 |
| researchAgent_config_passOne_* | researchAgent_qualityPass_researchBuild_* | 2.0 |
| researchAgent_config_passTwo_* | researchAgent_qualityPass_factCheck_* | 2.0 |
| researchAgent_config_passThree_* | researchAgent_qualityPass_hallucinationSweep_* | 2.0 |

---

## 7. AIX Envelope & Task Types

### 7.1 The AIX Envelope

```json
{
  "aix": "2.2",
  "id": "uuid-v4",
  "timestamp": "ISO-8601",
  "task": { "type": "...", "provider": "...", "model": "..." },
  "input": { },
  "output": { },
  "meta": { "cost": { }, "latency": { }, "tokens": { } },
  "error": { }
}
```

### 7.2 Task Type Registry

| Category | Task Type | Description |
| --- | --- | --- |
| Text | text_generation | Generate text from prompts |
| Text | text_completion | Complete partial text |
| Text | text_summarization | Summarize longer text |
| Text | text_translation | Translate between languages |
| Voice | voice_synthesis | Generate speech from text |
| Voice | voice_cloning | Clone a voice from samples |
| Voice | speech_to_text | Transcribe audio to text |
| Image | image_generation | Generate images from prompts |
| Image | image_editing | Modify existing images |
| Image | image_upscaling | Increase image resolution |
| Video | video_generation | Generate video from prompts |
| Video | video_editing | Modify existing video |
| Video | image_to_video | Animate a static image |
| Code | code_generation | Generate code from description |
| Code | code_completion | Complete partial code |
| Code | code_review | Analyze and review code |
| Music | music_generation | Generate music / audio |
| 3D | model_generation | Generate 3D models |
| Content | content_authoring | Generate authored content with voice, guidelines, and guardrails |

### 7.3 Metadata Object

```json
"meta": {
  "cost":    { "amount": 0.018, "currency": "USD", "unit": "per_request" },
  "latency": { "totalMs": 8400, "processingMs": 8100, "networkMs": 300 },
  "tokens":  { "input": 4200, "output": 1850, "total": 6050 },
  "modelVersion": "claude-sonnet-4-6",
  "requestId": "req_abc123",
  "region": "us-east-1",
  "retention": { "inputStored": false, "outputStored": true, "retentionDays": 30 }
}
```

### 7.4 Error Object

```json
"error": {
  "code": "rate_limited",
  "message": "Rate limit exceeded for this model.",
  "retryable": true,
  "retryAfterMs": 5000,
  "providerError": { }
}
```

---

## 8. Task-Specific Schemas

### 8.1 content_authoring

The `content_authoring` task type enables AI-generated content carrying full author identity, editorial voice, structural guidelines, and hard guardrails.

#### Input: author object

```json
"author": {
  "givenName": "Michael",
  "familyName": "Tchong",
  "displayName": "Michael Tchong",
  "jobTitle": "Founder, Solo Trillion",
  "bio": "Short author bio for attribution",
  "socialProfiles": {
    "linkedinUrl": "https://linkedin.com/in/michaeltchong",
    "twitterUrl":  "https://x.com/michaeltchong",
    "url":         "https://ubertrends.com"
  },
  "media": [
    { "mediumType": "newsletter", "mediumName": "Ubertrends", "mediumBrand": "FutureSight", "isPrimary": true }
  ]
}
```

#### Input: voice, research, brief

```json
{
  "voiceBrand":      "...",
  "voiceGuidelines": "...",
  "voiceGuardrails": "...",
  "researchPacket": {
    "topic": "...", "thesis": "...", "narrativeAngle": "...",
    "keyFindings": [{ "finding": "...", "sourceUrl": "...", "confidence": 8, "isContrarian": true }],
    "auditSummary": { "auditPassed": true, "hallucinationsFound": 0 }
  },
  "contentBrief": {
    "format": "default",
    "toneOverride": "contrarian",
    "targetLength": "standard_1200"
  }
}
```

#### Output

```json
"output": {
  "draft": {
    "headlineOptions": ["...", "...", "..."],
    "lede": "...",
    "sections": [{ "heading": "...", "content": "..." }],
    "cta": "...",
    "wordCount": 1247,
    "citations": [{ "text": "...", "url": "...", "title": "..." }]
  },
  "guardrailFlags": [{ "flag": "...", "location": "...", "recommendation": "..." }],
  "finishReason": "complete"
}
```

### 8.2 voice_synthesis

```json
"input":  { "text": "...", "voice": "aria", "language": "en-US", "speed": 1.0, "outputFormat": "mp3" },
"output": { "audio": { "url": "...", "format": "mp3", "durationMs": 3200 } }
```

### 8.3 image_generation

```json
"input":  { "prompt": "...", "width": 1024, "height": 1024, "style": "photorealistic", "outputFormat": "png" },
"output": { "images": [{ "url": "...", "format": "png", "width": 1024, "height": 1024 }] }
```

### 8.4 text_generation

```json
"input":  { "prompt": "...", "systemPrompt": "...", "maxTokens": 2000, "temperature": 1.0, "responseFormat": "markdown" },
"output": { "text": "...", "finishReason": "complete" }
```

### 8.5 video_generation

```json
"input":  { "prompt": "...", "durationSeconds": 10, "aspectRatio": "16:9", "resolution": "1080p" },
"output": { "video": { "url": "...", "format": "mp4", "durationMs": 10000 } }
```

### 8.6 code_generation

```json
"input":  { "prompt": "...", "language": "python", "framework": "fastapi", "styleGuide": "pep8" },
"output": { "code": "...", "language": "python", "files": [{ "filename": "main.py", "content": "..." }] }
```

---

## 9. Workflows & Chaining

AIX supports chaining multiple tasks via `${step_id.output.field}` variable substitution.

```json
{
  "aix": "2.2",
  "workflow": {
    "id": "uuid",
    "name": "Research → Write → Synthesize Voice",
    "steps": [
      {
        "id": "step_research",
        "task": { "type": "text_generation", "provider": "anthropic" },
        "input": { "prompt": "Research AI agents replacing knowledge workers" }
      },
      {
        "id": "step_article",
        "task": { "type": "content_authoring", "provider": "anthropic" },
        "input": {
          "researchPacket": "${step_research.output.text}",
          "contentBrief": { "format": "default", "toneOverride": "contrarian" }
        }
      },
      {
        "id": "step_audio",
        "task": { "type": "voice_synthesis", "provider": "elevenlabs" },
        "input": { "text": "${step_article.output.draft.lede}", "voice": "aria" }
      }
    ]
  }
}
```

---

## 10. Provider Comparison Format

```json
{
  "aix": "2.2",
  "comparison": {
    "task": { "type": "voice_synthesis" },
    "input": { "text": "Hello, this is a comparison test.", "language": "en-US" },
    "results": [
      { "provider": "elevenlabs", "model": "eleven_multilingual_v2",
        "output": { "audio": { "url": "...", "durationMs": 2100 } },
        "meta": { "cost": { "amount": 0.002 }, "latency": { "totalMs": 650 } } },
      { "provider": "openai", "model": "tts-1-hd",
        "output": { "audio": { "url": "...", "durationMs": 2080 } },
        "meta": { "cost": { "amount": 0.003 }, "latency": { "totalMs": 580 } } }
    ]
  }
}
```

---

## 11. Security & Governance

### 11.1 Security

AIX does not define authentication — this is provider-specific. AIX messages must never contain API keys, passwords, or personal access tokens.

```json
"safety": {
  "flagged": true,
  "categories": { "violence": false, "sexual": false, "hate": false, "selfHarm": false, "dangerous": true },
  "actionTaken": "content_blocked"
}
```

### 11.2 Versioning

| Version Level | Trigger | Example |
| --- | --- | --- |
| Major (X.0) | Breaking changes to envelope, required fields, or task type removal | 1.x → 2.0 |
| Minor (2.X) | New task types, new optional fields, additive features | 2.1 → 2.2 |
| Patch (2.0.X) | Clarifications, documentation fixes, no schema changes | 2.0 → 2.0.1 |

### 11.3 Amendment Process

1. Propose the new key with full four-part hierarchy and plain-English description
2. Run all six validation tests from §2.6
3. Check for conflicts with §6
4. Add to candidate keys in next minor version draft
5. Upon ratification, move to canonical reference

> Retroactive changes to ratified keys are prohibited. Deprecation is handled by adding a replacement key and marking the old key `[DEPRECATED]` — not by renaming it.

### 11.4 Governance

| Body | Responsibility |
| --- | --- |
| Technical Steering Committee | Specification changes and ratification |
| Provider Council | Implementation feedback and provider registry |
| Community Contributors | Pull requests, issues, and proposals |

---

## 12. Appendix

### A. Provider Registry

| Slug | Company | Task Types |
| --- | --- | --- |
| anthropic | Anthropic | text_generation, text_completion, content_authoring |
| openai | OpenAI | text_generation, image_generation, voice_synthesis, speech_to_text, code_generation |
| google | Google | text_generation, image_generation, speech_to_text |
| elevenlabs | ElevenLabs | voice_synthesis, voice_cloning, speech_to_text |
| playht | PlayHT | voice_synthesis, voice_cloning |
| runway | Runway | video_generation, image_to_video, image_editing |
| midjourney | Midjourney | image_generation |
| stability | Stability AI | image_generation, image_editing, image_upscaling |
| pika | Pika Labs | video_generation, image_to_video |
| replicate | Replicate | (various hosted models) |
| huggingface | Hugging Face | (various hosted models) |

### B. Language Codes (BCP-47)

| Code | Language |
| --- | --- |
| en-US | English (United States) |
| en-GB | English (United Kingdom) |
| es-ES | Spanish (Spain) |
| fr-FR | French (France) |
| de-DE | German (Germany) |
| ja-JP | Japanese |
| zh-CN | Chinese (Simplified) |
| zh-TW | Chinese (Traditional) |

### C. Media Type Mappings

| Format | MIME Type |
| --- | --- |
| mp3 | audio/mpeg |
| wav | audio/wav |
| mp4 | video/mp4 |
| webm | video/webm |
| png | image/png |
| jpeg | image/jpeg |
| webp | image/webp |

### D. References

1. [Model Context Protocol (MCP)](https://github.com/modelcontextprotocol)
2. [Eden AI APIs](https://github.com/edenai/edenai-apis)
3. [Schema.org](https://schema.org)
4. [JSON Schema](https://json-schema.org)
5. [BCP-47 Language Tags](https://www.rfc-editor.org/info/bcp47)
6. CrossARM Data Interchange Standard (2014) — Ubertrends LLC

### E. Implementation Notes (Non-Normative)

#### E.1 Media Collections — Single-Primary Subset Pattern

Implementations MAY ship a single-primary subset of `media[]` before supporting the full array. The subset stores one Medium object and applies Resolution Rule Step 2 directly. The full array is purely additive.

#### E.2 Module Manifest — Staged Rollout

Module manifests MAY omit optional fields (`billing`, `permissionScopes`) on initial implementation. Required fields for a v1 manifest: `aixVersion`, `moduleId`, `moduleType`, `version`.

#### E.3 writingAgent_format_* Migration Guide

| Deprecated Key (v2.1) | Canonical Key (v2.2) |
| --- | --- |
| writingAgent_documentType_article_* | writingAgent_format_default_* |
| writingAgent_documentType_fieldGuide_* | writingAgent_format_default_* |
| writingAgent_documentType_meetingBrief_* | writingAgent_format_meetingBrief_* |
| writingAgent_documentType_{other}_* | writingAgent_format_{other}_* |

Both key sets resolve identically during the transition. Remove deprecated keys after confirming all consumers have migrated.

---

## 13. Changelog

### v2.2 — May 2026

Minor version: additive.

| Area | Change |
| --- | --- |
| §2.1 | Slug/URL casing exception added (dash-case for slugs/URLs); schema version string format documented as exempt from camelCase |
| §2.4 | `uma` added to permitted acronym list |
| §3.3 | `legalName` cross-reference to §3.4 added |
| §3.10 (NEW) | Published Works field registry — 10 fields for book chapters and sidebars |
| §4 | `rosie` added as registered component key |
| §4.8 (NEW) | Module Manifest Standard — `{moduleType}Module_{moduleId}_{property}` hierarchy ratified; manifest schema defined |
| §5.1 | `documentType` deprecated as object type segment; `format` is canonical replacement |
| §5.2 | Format identifiers ratified under `writingAgent_format_*` |
| §6.2 | 12 new `writingAgent_format_*` canonical keys |
| §6.4 (NEW) | UMA Platform canonical keys (4 keys) |
| §6.5 (NEW) | FutureSight Studio publishing module keys (6 keys) |
| §6.6 (NEW) | Deprecated key registry with full replacement mapping |
| §7.3 | Metadata object example corrected to camelCase throughout |
| Appendix E.2/E.3 (NEW) | Module manifest staged rollout + format key migration guide |

### v2.1 — April 2026

| Area | Change |
| --- | --- |
| §3.5 (NEW) | Media Collections — `media[]` typed collection, `mediumType` vocabulary, four-step substitution rule |
| §4.7 (NEW) | Agent Behavioral Modes — `behavioralMode` field, standard mode vocabulary |
| §4.7.6 (NEW) | Soul Documents — persistent agent identity file standard |

### v2.0 — March 2026

Major version: merges AIX Interchange Format Specification and AIX Naming Convention Standard.

| Area | Change |
| --- | --- |
| Document structure | Merged two specs (v1.1 interchange + naming convention) into single document |
| §3 (NEW) | Persona Field Registry — 40+ fields |
| §4 (NEW) | Component & Agent Registry |
| §5 (NEW) | Reserved Segment Vocabulary |
| Deprecated | `writingAgent_template_*` \| `researchAgent_config_passOne/Two/Three_*` |

### v1.1 — February 2026

Added `content_authoring` task type, `author` object, voice fields, `researchPacket` chaining, `guardrailFlags`.

### v1.0 — December 2025

Initial specification. Core task types: text, voice, image, video, code. Metadata, workflows, provider comparison.

---

*Apache 2.0 Licensed · [aix-standard.org](https://aix-standard.org) · [github.com/aix-standard/aix-spec](https://github.com/aix-standard/aix-spec) · [aix@ubertrends.com](mailto:aix@ubertrends.com)*

*Copyright 2026 Michael Tchong, Ubertrends LLC*
