# AIX — AI Interchange Standard

**Version 2.4 · August 2026**

| Author: Michael Tchong, Ubertrends LLC · License: Apache 2.0 (Open Standard) |
| --- |
| *AIX is an open standard for naming, exchanging data, and establishing interoperability between generative AI tools, agents, and applications. This document is the single, converged authoritative reference — one standard, one version — superseding the separately-versioned AIX Standard v2.2 and the AIX 2.3 Canonical Registry Contract, both physically inlined here.* |
| Heritage: CrossARM (2014) · Ubertrends LLC · Solo Trillion |

---

> **Version invariant.** AIX is one standard with one version. There is no envelope standard and a separate content or registry standard carrying its own number — that model produced the divergence AIX 2.4 exists to close. Every surface in this repository that stamps an AIX version — code literals, database rows, seed scripts, frontmatter, primitive contracts — moves together when the version changes. None moves independently by surface, by document, or by "addendum" status. A future minor version that adds capability without changing this document's own naming/envelope/registry rules still bumps this one number, in this one document, everywhere at once.

---

| Chapter | Contents |
| --- | --- |
| 1. Introduction | Problem statement, CrossARM lineage, Schema.org relationship |
| 2. Core Naming Principles | camelCase, hierarchy, no abbreviations, validation tests |
| 3. Persona Field Registry | Schema.org base fields + AIX extensions for all persona fields |
| 4. Component & Agent Registry | All registered AIX components, Agent Behavioral Modes, Soul Documents |
| 5. Module Manifest Standard | Module key hierarchy, manifest schema, `permissionScopes`-vs-registry-grant relationship |
| 6. Reserved Segment Vocabulary | Object types, identifiers, property suffixes |
| 7. Canonical Key Reference | All ratified AIX keys, UMA primitive/telemetry/billing keys, deprecation policy |
| 8. AIX Envelope & Task Types | Interchange format, task schema, metadata |
| 9. Task-Specific Schemas | Voice, image, text, video, code, content_authoring |
| 10. Workflows & Chaining | Multi-step pipeline schema |
| 11. Provider Comparison Format | Side-by-side output schema |
| 12. Canonical Registry Contract | Scoped, capability-gated, auditable reads for canonical registries |
| 13. UMA 3.0 Primitive Contract Structure | Primitive registry; contract structure in fourteen numbered subsections, ten of them the sections every contract must carry; AIX key hierarchy for UI primitives |
| 14. Security & Governance | Auth, safety, versioning, amendment process |
| 15. Appendix | Provider list, language codes, media types, implementation notes |
| 16. Changelog | Version history |

---

# 1. Introduction

## 1.1 The Problem

The generative AI revolution has produced hundreds of specialized tools — each with its own API format, authentication method, and response structure. This fragmentation creates developer burden through custom integration code for each provider, vendor lock-in when switching providers requires rewriting applications, workflow friction when chaining tools requires format conversion, and no standard way to compare outputs across providers.

A parallel problem exists inside AI agents themselves: without a naming standard, every team invents its own key vocabulary for fields, configuration properties, and persona data. Keys proliferate without governance — some snake_case, some camelCase, some abbreviated, some redundant — until the system becomes unmaintainable and incompatible with external implementers.

## 1.2 The Solution

AIX addresses both problems with a single specification. It defines a standard envelope for AI task requests and responses so any AIX-compliant tool can accept standardized inputs, return standardized outputs, be swapped with any other tool in the same category, and chain seamlessly with other AIX tools in workflows. It also defines a canonical naming convention so every field, key, and identifier in the AIX ecosystem is unambiguous, hierarchical, and implementable by developers with no internal context.

## 1.3 Heritage: CrossARM (2014)

AIX follows the pattern established by CrossARM, a data interchange standard developed by Ubertrends LLC in 2014 for cross-platform marketing data exchange. CrossARM's governing principle was: use Schema.org canonical names where they exist, and invent proprietary extensions for what Schema.org does not cover.

```
// CrossARM 2014 — Schema.org base vocabulary
givenName, familyName, worksFor, address, email, telephone  (Person)
brand, legalName, duns, isicV4                              (Organization)

// CrossARM 2014 — Proprietary extensions (Schema.org had no equivalent)
facebookName, twitterName, twitterFollowers, engagementRate
uniqueVisits, timeOnSite, bounceRate, influencerRating
```

AIX v2.0 formalizes this pattern. Schema.org canonical names are used wherever applicable. Fields that Schema.org does not cover are defined as AIX extensions, noted as such in the registry.

## 1.4 Relationship to Existing Standards

| Standard | Focus | AIX Relationship |
| --- | --- | --- |
| MCP (Model Context Protocol) | Connecting AI to data sources | Complementary — MCP handles inputs, AIX handles outputs |
| Schema.org | Structured web data vocabulary | Foundation — AIX uses Schema.org canonical names as base layer |
| Eden AI Schemas | Traditional AI APIs (OCR, NLP) | Predecessor — AIX extends into generative AI task types |
| JSON Schema | Data validation | Implementation — AIX uses JSON Schema for field validation |
| CrossARM (2014) | Cross-platform marketing data | Heritage — established the Schema.org base + extension pattern |

---

# 2. Core Naming Principles

These principles govern every field name, key, and identifier in the AIX specification and in any AIX-compliant implementation.

## 2.1 camelCase Throughout

All field names and identifiers use camelCase. No snake_case, no kebab-case, no PascalCase for field names. This aligns with JSON, JavaScript, and modern API conventions.

```
// ✓ Correct
givenName  familyName  jobTitle  telephoneMobile  linkedinUrl

// ✗ Incorrect
given_name    // snake_case
given-name    // kebab-case
GivenName     // PascalCase
```

> **Layer-specific exceptions:**
> - **Database columns** use snake_case by PostgreSQL convention. API responses, JSON exports, and all specification field names use camelCase. ORM/serialization layers handle the translation (e.g., `given_name → givenName`).
> - **Slugs and URLs** use dash-case (e.g., `newsletter-studio`, `rss-discovery`). Slugs are identifiers in routing and storage contexts, not AIX field names, and follow web convention. Slug values are never used as AIX field keys.
> - **Schema version strings** use the format `{type}-v{N}` (e.g., `studio-json-v1`, `canvas-document-v1`). These are opaque version identifiers, not field keys, and are exempt from the camelCase rule.

## 2.2 Hierarchical Namespace

Every internal AIX identifier (agent configs, format types, quality passes, module keys) follows a strict four-part hierarchy. Each segment narrows scope from broad to specific. Underscores serve exclusively as namespace separators — they are not snake_case within a segment.

```
{component}_{objectType}_{identifier}_{property}

writingAgent_format_default_systemPrompt
writingAgent_format_newsletter_outputRules
researchAgent_qualityPass_hallucinationSweep_systemPrompt
personaAgent_voice_brand
publishingModule_newsletterStudio_studioJson
```

## 2.3 Schema.org Base + AIX Extensions

For persona data and interchange fields, AIX uses Schema.org canonical property names where they exist. When Schema.org does not have a suitable property, AIX defines extensions, marked 'AIX extension' in the Source column.

| Layer | Example Fields | Rule |
| --- | --- | --- |
| Schema.org canonical | givenName, familyName, jobTitle, legalName, telephone, email, naics, foundingDate | Use as-is, no modification |
| AIX extensions | telephoneMobile, emailBusiness, linkedinUrl, twitterUrl | Only when Schema.org has no equivalent |
| AIX hierarchy keys | writingAgent_format_default_systemPrompt | Four-part camelCase hierarchy for agent/config keys |

> **Amended 2026-10-20.** `taxId` (§3.4) is the Schema.org property `taxID` written in this standard's camelCase form, with `id` registered at §2.4; the property name is otherwise used as-is.

## 2.4 No Abbreviations

All segments use full English words. The only exceptions are established industry acronyms registered in the AIX permitted acronym list.

| ✗ Prohibited (abbreviated) | ✓ Required (full word) |
| --- | --- |
| liUrl | linkedinUrl |
| sp | systemPrompt |
| or | outputRules |
| docType | documentType |
| qPass | qualityPass |
| ra | researchAgent |

Permitted acronyms: `aix`, `api`, `url`, `html`, `json`, `uuid`, `naics`, `rss`, `uma`, `id`

> `id` added 2026-10-20. `id` is an abbreviation of "identifier", not an acronym; it is registered here as a permitted exception under this section's No Abbreviations rule. Ratified keys have carried `Id` segments since v2.1: `mediumId` (§3.5), `requestId` (§8.3) and the manifest's `moduleId` (§5.2), and at 2.4 the §7.6 keys, including `{capabilityId}` and the telemetry and billing keys ending in `moduleId`, `primitiveId` or `sessionId`. The list now records that existing practice. `rss` added 2026-05-12. RSS (Really Simple Syndication / RDF Site Summary) has been the W3C-canonical name for the format since 2002. `uma` added 2026-05-09. UMA (Unified Module Architecture) is the registered module platform layer in the AIX ecosystem.

## 2.5 Identifier Names the Format — Properties Define It

The identifier segment (position 3 in the hierarchy) uses a short, stable handle. The systemPrompt and outputRules properties carry the full behavioral specification. Never encode behavior into the identifier name.

```
// ✓ Correct — identifier is a short stable handle
writingAgent_format_newsletter_systemPrompt
writingAgent_format_newsletter_outputRules

// ✗ Wrong — behavior encoded into identifier name
writingAgent_format_formattedNewsletterWithCuratedSections_systemPrompt
```

## 2.6 Naming Validation Tests

Before any new key is ratified, it must pass all six tests. A key that fails any test must be revised.

| Test | Question | Fail Condition |
| --- | --- | --- |
| 1. Stranger Test | Could a developer with no internal context understand this key from its name alone? | Requires docs or explanation to parse |
| 2. Hierarchy Test | Does the key follow {component}_{objectType}_{identifier}_{property}? | Missing segment when more specificity exists |
| 3. Abbreviation Test | Does the key contain any prohibited abbreviated segments? | Any segment not a full word or registered acronym |
| 4. Case Test | Is every segment in camelCase? | Any snake_case, kebab-case, or UPPER_SNAKE within a segment |
| 5. Redundancy Test | Does any segment restate or mirror an adjacent segment? | e.g. documentType_document — concept repeated |
| 6. Conflict Test | Does the key duplicate or closely mirror an existing canonical key? | Duplicate or near-duplicate of a ratified key |

---

# 3. Persona Field Registry

The Persona Field Registry defines every field in the AIX identity vault. Each field is either a Schema.org canonical property (used as-is) or an AIX extension (defined because Schema.org has no equivalent). This registry is the authoritative source for all Persona Vault implementations.

> **Implementations (amended 2026-10-20).** The tables in this chapter are the normative registry. An implementation may keep its own project field-key list; that list conforms to this chapter.

> Schema.org canonical fields are used as-is. AIX extension fields are marked 'AIX extension' in the Source column. The field key is the bare name — e.g., `linkedinUrl`, not `aix:linkedinUrl`.

## 3.1 Essentials

| AIX Field Key | Source | Description | Example Value |
| --- | --- | --- | --- |
| givenName | Schema.org | Legal first name | Michael |
| familyName | Schema.org | Legal last name | Tchong |
| preferredName | AIX extension | Preferred name or nickname if different from legal name | Mike |
| jobTitle | Schema.org | Professional title | Founder & CEO |
| worksFor | Schema.org | Primary organization name | Ubertrends LLC |
| industry | AIX extension | Industry or sector | Technology Consulting |
| addressHome | AIX extension | Home or personal display location (City, State) | Las Vegas, NV |
| timezone | AIX extension | IANA timezone identifier | America/Los_Angeles |

## 3.2 Professional

| AIX Field Key | Source | Description |
| --- | --- | --- |
| activeProject1 | AIX extension | Current primary project name and one-line description |
| activeProject2 | AIX extension | Current secondary project |
| currentGoal1 | AIX extension | Active goal with target date |
| currentGoal2 | AIX extension | Active goal with target date |
| currentGoal3 | AIX extension | Active goal with target date |
| expertise | AIX extension | Core areas of expertise (comma-separated or prose) |
| toolsUsed | AIX extension | Primary tools and platforms currently in use |
| resume | AIX extension | Short professional bio or resume summary |

## 3.3 Organization

These fields describe the author's organization or business entity. They use Schema.org Organization vocabulary where available.

| AIX Field Key | Source | Description |
| --- | --- | --- |
| legalName | Schema.org | Full legal name of the organization — see §3.4 for authoritative definition |
| description | Schema.org | Short organization description (elevator pitch) |
| companyBackgrounder | AIX extension | Longer organization backgrounder for press or partners |
| productServiceDescription1 | AIX extension | Primary product or service description |
| productServiceDescription2 | AIX extension | Secondary product or service description |
| url | Schema.org | Primary website URL |
| websiteUrl2 | AIX extension | Secondary website URL |

> **`legalName` is a shared field** defined authoritatively in §3.4 Financial & Legal. It appears here because Organization context is where most implementations first encounter it. §3.4 is the canonical definition.

## 3.4 Financial & Legal

> **Amended 2026-10-20.** `bankName` and `businessReferences` are ratified, and `bank` and `references` are deprecated in their favor (§7.7, §7.8); `taxId` (Schema.org `taxID`) is ratified. All three are tier private in the reference implementation. Each passed the five §2.6 tests that apply: Stranger, Abbreviation, Case, Redundancy and Conflict (`bankName` and `businessReferences` replace keys deprecated in their favor, and `taxId` is distinct from `taxCategory`). Hierarchy does not apply to a bare persona field key (§2.3). `taxId` passes Abbreviation with `id` registered at §2.4.
>
> Two implementation matters are recorded here, not ratified. The reference implementation stores the legal entity name under a non-conformant key, `organizationName`, alongside the conformant `legalName`; it is not ratified, because it duplicates `legalName` (§2.6 test 6), and the implementation will resolve it. The reference implementation also stores three bank-detail keys for invoicing, `bankAddress`, `routingNumber` and `accountNumber`, which are pending ratification and not registered: `bankAddress` departs from the Disambiguation Pattern below, `routingNumber` names a jurisdiction-specific code, and `accountNumber` does not say from its name alone that it is a bank account (§2.6 test 1). Conformant names are to be settled before they are registered.

| AIX Field Key | Source | Description |
| --- | --- | --- |
| legalName | Schema.org | Legal entity name (also in Organization — shared field) |
| entityType | AIX extension | Legal entity type (LLC, Corp, Sole Prop, etc.) |
| foundingDate | Schema.org | Date organization was founded or incorporated |
| naics | Schema.org | NAICS industry classification code |
| taxCategory | AIX extension | Tax classification (e.g., Disregarded Entity, S-Corp) |
| addressBusiness | AIX extension | Full business mailing address |
| addressHome | AIX extension | Home or personal display location |
| bank | AIX extension | `[DEPRECATED, 2.4]` Primary banking institution name. Deprecated in favor of `bankName`; see §7.7. |
| references | AIX extension | `[DEPRECATED, 2.4]` Professional references. Deprecated in favor of `businessReferences`; see §7.7. |
| bankName | AIX extension | Primary banking institution name |
| businessReferences | AIX extension | Professional references (names, titles, contact info) |
| taxId | Schema.org (`taxID`) | Tax identifier of the organization |
| telephoneBusiness | AIX extension | Business/office telephone number |
| telephoneMobile | AIX extension | Mobile telephone number |

> **AIX Disambiguation Pattern** — Schema.org has single-value properties for address, telephone, and email. AIX extends each with a qualifier suffix: `addressBusiness` / `addressHome`, `telephoneBusiness` / `telephoneMobile`, `emailBusiness` / `emailPersonal`. The Schema.org base property is not used directly in AIX implementations.

## 3.5 Media Collections

AIX v2.1 introduced the `media` array: a collection of Medium entries, each typed with controlled vocabulary, carrying its own name, optional brand, optional tagline, optional URL, and a primary flag.

### Field Definitions

| AIX Field Key | Source | Type | Required | Description |
| --- | --- | --- | --- | --- |
| media | AIX extension | array | No | Ordered collection of Medium objects representing the author's media surfaces |
| mediumType | AIX extension | string | Yes (per entry) | Controlled vocabulary — see table below |
| mediumName | AIX extension | string | Yes (per entry) | User-facing display name of this media surface |
| mediumBrand | AIX extension | string | No | Optional umbrella brand if distinct from mediumName |
| mediumTagline | AIX extension | string | No | Per-medium tagline; defaults to persona tagline if absent |
| mediumUrl | AIX extension | string (URL) | No | Canonical URL for this media surface |
| isPrimary | AIX extension | boolean | Yes (per entry) | Exactly one entry per persona must be true |

> **`mediumName` vs. `worksFor`** — `worksFor` names the organization (`"Ubertrends LLC"`). `mediumName` names the media surface (`"Ubertrends"`). For solo operators whose company IS their media brand, the values may converge. Template substitution contexts differ: `{worksFor}` = organizational affiliation; `{mediumName}` = publication/media byline.

### `mediumType` Controlled Vocabulary

| Value | Description | Examples |
| --- | --- | --- |
| `newsletter` | Email-delivered periodic publication | Ubertrends, The Hustle, Morning Brew |
| `blog` | Web-native long-form writing surface | Waitbutwhy, Stratechery |
| `podcast` | Audio episodic series | The Tim Ferriss Show, Lex Fridman Podcast |
| `channel` | Video episodic series — platform-neutral | Veritasium (YouTube), MKBHD |
| `social` | Broadcast-style social presence | LinkedIn, X/Twitter, Threads, Bluesky |
| `platform` | Product or service where content appears | Solo Trillion, a SaaS product blog |
| `publication` | Formal editorial container | Substack, Ghost, Medium publication |
| `community` | Membership or discussion space | Discord server, Slack community, Circle |
| `other` | Any surface not covered above | — |

Implementations MUST accept unknown `mediumType` values without error, treating them as `other`.

### Template Substitution Resolution Rule

When a Writing Agent resolves `{mediumName}`, `{mediumBrand}`, or `{mediumTagline}` at generation time:

| Step | Condition | Action |
| --- | --- | --- |
| 1 | Generation request specifies `mediumId` or `mediumType` | Substitute from the matching `media[]` entry |
| 2 | No request-level override | Substitute from the entry where `isPrimary: true` |
| 3 | No primary entry | Substitute from `media[0]` |
| 4 | `media[]` is empty or absent | Fall back to `worksFor` for `{mediumName}`; log a warning |

This resolution rule is normative. Any AIX-compliant Writing Agent must implement it identically.

### Reference Implementation Example

```json
"media": [
  {
    "mediumType": "newsletter",
    "mediumName": "Example Weekly",
    "mediumBrand": "Example Media",
    "mediumTagline": "A fictional newsletter for illustration",
    "mediumUrl": "https://example.com",
    "isPrimary": true
  },
  {
    "mediumType": "platform",
    "mediumName": "Example Platform",
    "mediumUrl": "https://platform.example.com",
    "isPrimary": false
  }
]
```

### Backwards Compatibility
AIX v2.1 is purely additive. The non-canonical `{publication_name}` placeholder is deprecated in favor of `{mediumName}`. `{publication_name}` will be removed in v3.0.

## 3.6 Social Media

| AIX Field Key | Source | Description | Format |
| --- | --- | --- | --- |
| linkedinUrl | AIX extension | LinkedIn profile URL | https://linkedin.example.com/in/handle |
| twitterUrl | AIX extension | X/Twitter profile URL | https://x.example.com/handle |
| facebookUrl | AIX extension | Facebook profile or page URL | https://facebook.com/handle |
| instagramUrl | AIX extension | Instagram profile URL | https://instagram.com/handle |
| youtubeUrl | AIX extension | YouTube channel URL | https://youtube.com/@handle |
| tiktokUrl | AIX extension | TikTok profile URL | https://tiktok.com/@handle |
| substackUrl | AIX extension | Substack publication URL | https://name.substack.com |

> **Naming pattern:** `{platform}Url` — the `Url` suffix is required. Without it, `linkedin` is ambiguous (handle string, URL, or boolean). Schema.org equivalent: `sameAs` (array) — insufficient for per-platform agent use.

## 3.7 Contact (Email)

| AIX Field Key | Source | Description |
| --- | --- | --- |
| email | Schema.org | Primary email address |
| emailBusiness | AIX extension | Business email address |
| emailPersonal | AIX extension | Personal email address |

## 3.8 Preferences

| AIX Field Key | Source | Description |
| --- | --- | --- |
| communicationStyle | AIX extension | Preferred communication style with this agent |
| checkinFrequency | AIX extension | How frequently and in what format to receive check-ins |
| interests | AIX extension | Personal and professional interests |
| offLimitsTopics | AIX extension | Topics or subjects to avoid in generated content |
| responseFormat | AIX extension | Preferred response format (prose, bullets, structured, etc.) |

## 3.9 Conversational Agent

| AIX Field Key | Source | Description |
| --- | --- | --- |
| conversationalAgentSystemPrompt | AIX extension | System prompt defining the conversational agent persona |

## 3.10 Published Works

AIX v2.2 introduces fields for authors who publish long-form structured works (books, field guides, reference manuals). These fields enable book chapter chaining, sidebar attribution, and cross-referencing within multi-chapter publications.

| AIX Field Key | Source | Description |
| --- | --- | --- |
| bookTitle | AIX extension | Full title of the book or long-form work |
| bookKey | AIX extension | Stable slug identifier for the book (dash-case, e.g., `ai-power-tools`) |
| chapterKey | AIX extension | Stable slug identifier for a chapter (e.g., `chapter-01-foundations`) |
| chapterTitle | AIX extension | Display title of the chapter |
| chapterNumber | AIX extension | Sequential chapter number (integer) |
| chapterContent | AIX extension | Full prose content of the chapter |
| chapterSummary | AIX extension | Short summary of the chapter for navigation and indexing |
| chapterSubsections | AIX extension | Array of subsection headings within the chapter |
| contentType | AIX extension | Type of content unit: `chapter` \| `sidebar` \| `appendix` \| `introduction` |
| parentChapterKey | AIX extension | For sidebars: the `chapterKey` of the containing chapter |

> **Stranger Test validation (all 10 fields):** All fields use full English words, follow the `{concept}{Qualifier}` camelCase pattern, and are self-describing to a developer with no internal context. `bookKey` and `chapterKey` use `Key` as a suffix indicating a stable slug identifier — consistent with the AIX extension pattern for opaque handles.

---

# 4. Component & Agent Registry

Components are the top-level namespace segment in all AIX hierarchy keys. New components require a spec amendment before use.

| Component Key | Full Name | Description |
| --- | --- | --- |
| researchAgent | Research Agent | Web research, source synthesis, hallucination detection |
| writingAgent | Writing Agent | Content generation, format management, output formatting |
| personaAgent | Persona Agent | Identity vault, voice, guidelines, guardrails |
| conversationalAgent | Conversational Agent | Conversational assistant (Telegram + web) — default name: Rosie |
| rosie | Rosie | Registered component key for Rosie-specific configuration keys. Use `conversationalAgent` for generic interchange; use `rosie` for Rosie-specific implementation keys. |
| orchestra | Orchestra | Platform-level orchestration, agent chaining, workflow config |
| aix | AIX Standard | Specification-level fields (version, schema, envelope) |

## 4.7 Agent Behavioral Modes

AIX-compliant agents may declare a behavioral mode that governs how they respond to inputs and initiate actions.

### 4.7.1 The Problem

Without a standard mode declaration, agents default to their training bias — typically "helpful general assistant" — regardless of the role they are assigned. A naming standard alone does not solve this; behavior must be declarable and queryable at the protocol level.

### 4.7.2 The behavioralMode Field

```json
{
  "aix": "2.4",
  "agent": {
    "id": "rosie",
    "role": "chief_of_staff",
    "behavioralMode": "observer",
    "allowedModes": ["observer", "orchestrator", "briefing", "off"]
  }
}
```

### 4.7.3 Standard Mode Vocabulary

| Mode | Description | Agent Behavior |
| --- | --- | --- |
| `observer` | Default monitoring state | Listens, classifies, surfaces signals. Does not initiate external actions without explicit trigger. |
| `orchestrator` | Active pipeline execution | Runs workflows, coordinates other agents, executes scheduled tasks. |
| `briefing` | Delivery state | Formats and delivers digests, summaries, and reports. Read-only relative to source data. |
| `off` | Silent state | No proactive outreach. Responds to direct queries only. Executes no scheduled tasks. |

### 4.7.4 Mode Transitions

Behavioral mode transitions must be explicit. An agent must not self-transition between modes based on conversation context alone. Mode changes require a direct user instruction, a scheduled trigger, or an orchestration signal from another AIX-compliant agent.

### 4.7.5 Mode Declaration in Task Envelopes

```json
{
  "aix": "2.4",
  "task": { "type": "content_authoring", "provider": "anthropic", "model": "claude-sonnet-4-6" },
  "meta": { "requestId": "req_abc123", "timestamp": "2026-05-01T09:00:00Z", "agentMode": "orchestrator" }
}
```

### 4.7.6 Soul Documents

A Soul Document is a persistent identity file that defines an agent's role, constraints, personality, and behavioral boundaries. Loaded before any system prompt, conversation history, or user input.

| Section | Required | Description |
| --- | --- | --- |
| Identity | Yes | Who the agent is and what role it serves |
| Role | Yes | Specific functions the agent performs |
| Constraints | Yes | Explicit list of what the agent never does |
| Behavioral Modes | Recommended | Mapping to AIX 4.7.3 standard vocabulary |
| Tone Examples | Recommended | Compliant and non-compliant response examples |

> The Soul Document pattern was formalized in AIX 2.1 based on implementation experience with Solo Trillion's Rosie agent and the OpenClaw project's SOUL.md architecture.

### 4.7.7 Relationship to Existing Standards

| Standard | Relationship |
| --- | --- |
| MCP (Model Context Protocol) | MCP defines what data an agent can access. AIX behavioral modes define how the agent acts on that data. Complementary. |
| OpenAI Assistants API | Assistants have instructions but no standard mode vocabulary. AIX modes are interoperable across providers. |
| LangChain Agent Types | LangChain defines execution strategies (ReAct, Plan-and-Execute). AIX modes define behavioral scope. Complementary. |

---

# 5. Module Manifest Standard

AIX formally ratifies the module manifest hierarchy for the Unified Module Architecture (UMA) platform layer. Modules are self-describing units that declare their identity, requirements, and contracts in a standardized manifest.

## 5.1 Module Key Hierarchy

Module keys follow the four-part AIX hierarchy with a specialized pattern:

```
{moduleType}Module_{moduleId}_{property}

publishingModule_newsletterStudio_studioJson
publishingModule_newsletterStudio_publishRecord
researchModule_rssDiscovery_feedUrl
```

**Module type segments** (position 1):
- `publishingModule` — Content publishing and distribution modules
- `researchModule` — Research, discovery, and source modules
- `writingModule` — Writing assistance and generation modules
- `personaModule` — Identity and voice customization modules

**Module ID segment** (position 2): The registered `moduleId` in camelCase (e.g., `newsletterStudio`, `rssDiscovery`). This corresponds to the module's `app_id` slug converted to camelCase. Note: the manifest's `moduleId` field stores the dash-case slug (`"newsletter-studio"`); the AIX hierarchy key uses its camelCase equivalent (`newsletterStudio`). These are the same module — different representations for different contexts.

**Property segment** (position 3+): The specific property or data schema key.

## 5.2 Module Manifest Schema

Every AIX-compliant module carries a manifest object:

```json
{
  "aixVersion": "2.4",
  "moduleId": "newsletter-studio",
  "moduleType": "publishing",
  "displayName": "Newsletter Studio",
  "version": "1.0.0",
  "requires": ["writingAgent", "personaAgent"],
  "inputContracts": ["content_authoring"],
  "outputContracts": ["studio-json-v1"],
  "permissionScopes": ["persona:read", "writing:generate", "publishing:send"],
  "billing": { "model": "per_use", "unit": "generation" }
}
```

> **Slug-to-camelCase rule:** The module's `app_id` is dash-case (`newsletter-studio`). The AIX module key uses camelCase for the identifier segment (`newsletterStudio`) — this is the first module identifier in this document where that conversion actually derives mechanically; `emailStudio` (retired at 2.4, §7.7) never did, under either app_id this module has carried. The dash-case slug is used in routing and database storage only; the camelCase form is the canonical AIX key.

### 5.2.1 `permissionScopes` and canonical registry grants — a composable relationship, not a merged vocabulary

A module's `permissionScopes` (§5.2) declares its access needs at registration — a flat list of bare capability strings, with no tier, lifetime, resource-scope, or obligation attached to any entry. A canonical registry's grant (chapter 12.3/12.6.1) is never a bare string: it is always the tuple `(namespace, capability, tier, resource-scope, lifetime, obligations[])`, issued per-agent and revocable. A `permissionScopes` entry is satisfied when at least one active grant's `(namespace, capability)` maps to it. A manifest declaring an entry with no possible mapping fails registration-time validation; a dispatch attempting an action no current grant covers fails at runtime with the result codes chapter 12.7 already defines. `permissionScopes` is never itself a grant, and a module manifest is never rewritten to carry capability strings directly — composition, not replacement.

`permissionScopes` is coarser than a registry grant not because its own vocabulary happens to look different — a `permissionScopes` entry (`"persona:read"`) and a chapter-12 capability string (`"records:read"`) are both bare `{namespace-or-domain}:{verb}` strings, same shape — but because `permissionScopes` structurally *cannot* carry tier, lifetime, resource-scope, or obligations. Two grants can carry the identical capability string and differ in every other respect; `permissionScopes` has no way to express that difference. A manifest's `permissionScopes` entry is a registration-time statement of intent; a chapter-12 grant is the runtime credential that intent is checked against.

> **UMA 3.0 UI Primitive Contract Structure:** §5 defines module-level manifests. For UI primitive contracts — the shared components consumed by modules — see chapter 13, which registers the full UMA 3.0 contract structure and its own key families (`umaModule_primitive_*`, `umaModule_telemetry_usageEvent_*`, `umaModule_billing_transactionEvent_*`, `umaModule_tier_capability_*`). These keys are formally registered in chapter 13 (UMA 3.0 Primitive Contract Structure) — no longer a forward-pointer to a future minor version, since that minor version is this document.

---

# 6. Reserved Segment Vocabulary

## 6.1 Object Type Segments (Position 2)

| Object Segment | Usage | Example |
| --- | --- | --- |
| format | Output format variants for the Writing Agent | writingAgent_format_default_systemPrompt |
| qualityPass | Error-check and verification passes (any agent) | researchAgent_qualityPass_hallucinationSweep_systemPrompt |
| config | General agent operational configuration | researchAgent_config_default_systemPrompt |
| voice | Author identity and brand voice data | personaAgent_voice_brand |
| output | Generation output schema fields | writingAgent_output_draft_headline |
| input | Generation input schema fields | writingAgent_input_contentBrief_format |
| schema | JSON schema definitions | aix_schema_contentAuthoring_input |
| meta | Metadata (cost, latency, tokens) | aix_meta_cost_amount |

> **`documentType` deprecated:** The `documentType` object segment (used in v2.0–v2.1 keys like `writingAgent_documentType_article_systemPrompt`) is deprecated in v2.2. New keys use `format`. Deprecated keys remain in the registry marked [DEPRECATED]; do not use for new implementations.

## 6.2 Identifier Segments (Position 3)

### Format Identifiers (under writingAgent_format)

`default` is the open-form workhorse — shaped entirely by the user's persona and config, producing articles, proposals, field guides, business plans, or any freeform document. Named formats have a defined structure.

| Identifier | What It Produces |
| --- | --- |
| default | Open-form output — structure defined by persona and config. Produces articles, field guides, business plans, proposals, or any freeform document. |
| meetingBrief | Condensed pre-meeting intelligence brief — structured, actionable |
| executiveBrief | Executive summary — structured for senior audience |
| competitiveAnalysis | Competitive analysis — structured comparison report |
| newsletter | Newsletter edition — curated, conversational |
| linkedinPost | LinkedIn post — compressed authority, plain text output |

> **Migration note:** `article` and `fieldGuide` identifiers (previously under `writingAgent_documentType_*`) are deprecated. `writingAgent_format_default_systemPrompt` is the canonical replacement. Existing implementations using these identifiers should migrate; deprecated keys remain in §7 for reference.

### Quality Pass Identifiers (under {agent}_qualityPass)

Universal quality control vocabulary shared across all agents.

| Identifier | Description |
| --- | --- |
| researchBuild | Initial data gathering and source synthesis pass |
| factCheck | Verify all claims against cited sources |
| hallucinationSweep | Detect and flag fabricated or unsupported content |
| complianceEdit | Apply guardrails, voice rules, and editorial standards |
| qualityScore | Final quality assessment and confidence scoring |
| optimizationPass | Refine output for performance, cost, or length targets |

### Voice Identifiers (under personaAgent_voice)

| Identifier | Description |
| --- | --- |
| brand | Author brand voice DNA — tone, vocabulary, sentence rhythm, signature phrases |
| guidelines | Editorial writing structure and style rules |
| guardrails | Hard stops, non-negotiables, topics requiring human review |

### Config Identifiers (under {agent}_config)

| Identifier | Description |
| --- | --- |
| default | The baseline configuration that ships with the product |

## 6.3 Property Segments (Position 4)

| Property Suffix | Value Type | Description |
| --- | --- | --- |
| systemPrompt | string | LLM system prompt for the given context |
| outputRules | string | Formatting and structure rules for generated output |
| config | object | Configuration object (when used as terminal node) |
| schema | object | JSON Schema definition |
| template | object | Complete template bundle (prompt + rules + metadata) |
| label | string | Human-readable display label |
| description | string | Human-readable description |
| version | string | Semantic version string |
| enabled | boolean | Feature flag |

---

# 7. Canonical Key Reference

The master reference for all currently ratified AIX hierarchy keys.

Key version markers:
- Keys marked `1.x` were introduced in v1.x of the specification
- Keys marked `2.0**` are newly introduced in v2.0
- Keys marked `2.0*` replace prior v1.2 keys
- Keys marked `2.2**` are newly introduced in v2.2
- Keys marked `UMA-3.0` are newly registered at 2.4 in chapter 13
- Keys marked `[DEPRECATED]` must not be used for new work; retained for backwards compatibility

## 7.1 Persona Agent — Voice

| Canonical Key | Type | Ver | Description |
| --- | --- | --- | --- |
| personaAgent_voice_brand | string | 1.1 | Author brand voice DNA |
| personaAgent_voice_guidelines | string | 1.1 | Editorial writing guidelines |
| personaAgent_voice_guardrails | string | 1.1 | Hard stops and non-negotiables |

## 7.2 Writing Agent — Format (v2.2)

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

## 7.3 Research Agent

| Canonical Key | Type | Ver | Description |
| --- | --- | --- | --- |
| researchAgent_config_default_systemPrompt | string | 1.2 | Research Agent default system prompt |
| researchAgent_qualityPass_researchBuild_systemPrompt | string | 2.0** | Pass 1: Research Build prompt |
| researchAgent_qualityPass_factCheck_systemPrompt | string | 2.0** | Pass 2: Fact Check prompt |
| researchAgent_qualityPass_hallucinationSweep_systemPrompt | string | 2.0** | Pass 3: Hallucination Sweep prompt |
| researchAgent_qualityPass_complianceEdit_systemPrompt | string | 2.0** | Pass 4: Compliance Edit prompt |
| researchAgent_qualityPass_qualityScore_systemPrompt | string | 2.0** | Pass 5: Quality Score prompt |

## 7.4 UMA Platform Keys (v2.2)

Platform-level keys for the Unified Module Architecture. These keys travel at the platform layer, not the agent layer.

| Canonical Key | Type | Ver | Description |
| --- | --- | --- | --- |
| umaModule_registry_manifest | object | 2.2** | Module manifest JSON object |
| umaModule_entitlement_status | string | 2.2** | Module entitlement state: `active` \| `suspended` \| `pending` |
| umaModule_billing_ref | string | 2.2** | Billing reference ID for usage tracking |
| umaModule_usage_event | object | 2.2** `[DEPRECATED, 2.4]` | Metering event payload. **Deprecated in favor of `umaModule_telemetry_usageEvent_*` (§7.6) — see §7.7.** Neither this key nor its replacement has ever been implemented in this codebase; live module-usage metering runs on `module_usage_events`, a mechanism this standard does not register. |

`umaModule_billing_ref` carries no deprecation marking — its coarser-grain relationship to `umaModule_billing_transactionEvent_*` (§7.6) is unchanged; only the usage-event key is deprecated. The asymmetry is deliberate, not an oversight.

## 7.5 Publishing Module — Newsletter Studio (v2.2, keys corrected at v2.4)

Keys for the Newsletter Studio publishing module (module ID segment `newsletterStudio`). Six of the seven keys below were ratified in 2.2 under module ID segment `emailStudio`, a camelCase identifier that never mechanically derived from any `app_id` this module has carried — deprecated at 2.4 in favor of the six `newsletterStudio` keys below, which do derive correctly (§5.1, §5.2). The deprecated originals are listed in full at §7.7, not repeated here, per this document's own convention for a fully-superseded key family (§7.7's `writingAgent_documentType_*` rows follow the same shape).

**[CORRECTED, 2026-08-15 — this section previously labeled the camelCase forms above as `moduleId: newsletterStudio` / `moduleId: emailStudio`, contradicting §5.2's own rule.]** §5.2 is authoritative: the manifest's `moduleId` field is dash-case; only the AIX hierarchy key's module-ID segment is camelCase. The camelCase values above were correct in context — only the label was wrong, corrected to `module ID segment`. **Do not "fix" the live `moduleId` field to match this section's old wording — the row is conformant; the prose was stale, not the data.**

**`publishingModule_newsletterStudio_htmlExport` is the seventh key, ratified new at 2.4 — not a rename, and deliberately absent from §7.7.** The live `module_registry` manifest has carried `publishingModule_emailStudio_htmlExport` since the module's original seed (2026-05-09), but it was never ratified in any prior version of this standard — the six-key table this section has always carried never included it. §7.7's Deprecated Keys table records keys that *were* ratified and have since been superseded; there is nothing to deprecate here, since nothing was ever ratified under this name. This is a plain new ratification, closing a gap the live registry has been silently citing for over three months, not a rename.

| Canonical Key | Type | Ver | Description |
| --- | --- | --- | --- |
| publishingModule_newsletterStudio_studioJson | object | 2.4** | AIX Canvas envelope for email Studio rendering |
| publishingModule_newsletterStudio_canvasDocument | object | 2.4** | Canvas document schema [RESERVED — v1.5] |
| publishingModule_newsletterStudio_publishRecord | object | 2.4** | Record of a completed publish operation |
| publishingModule_newsletterStudio_deliveryTarget | string | 2.4** | Target delivery address or list identifier |
| publishingModule_newsletterStudio_issueSubject | string | 2.4** | Email subject line for this issue |
| publishingModule_newsletterStudio_issueDate | string | 2.4** | Publication date (ISO-8601) |
| publishingModule_newsletterStudio_htmlExport | string (text/html) | 2.4** | Rendered HTML email string generated for local browser download, metered to `module_usage_events`. Not a delivery mechanism — no ESP send is involved in producing this value; sending to a recipient is a separate, unrelated live code path with no AIX key of its own. |

## 7.6 UMA 3.0 Primitive, Telemetry, Billing, and Tier-Gated Capability Keys (UMA-3.0, new at 2.4)

Registered by chapter 13's UMA 3.0 contract structure. These keys extend the `umaModule_*` namespace from §7.4 to primitive-level contracts, telemetry, billing, and tier-gated capability. All follow §2.2 hierarchy and pass the six §2.6 validation tests.

**Primitive Contract Keys:**

| Canonical Key | Type | Ver | Description |
| --- | --- | --- | --- |
| `umaModule_primitive_{primitiveId}_contractVersion` | string | UMA-3.0 | Semver version of the AIX contract for this primitive (e.g., `"1.0.0"`) |
| `umaModule_primitive_{primitiveId}_aixVersion` | string | UMA-3.0 | AIX standard version this contract targets |
| `umaModule_primitive_{primitiveId}_status` | string | UMA-3.0 | `stable` \| `experimental` \| `deprecated` |

**Telemetry Event Keys** — shape of the `umaModule_telemetry_usageEvent_*` payload emitted by primitives via the `onUsage` callback (§13.2.11.1):

| Canonical Key | Type | Ver | Description |
| --- | --- | --- | --- |
| `umaModule_telemetry_usageEvent_primitiveId` | string | UMA-3.0 | The primitive ID that fired the event |
| `umaModule_telemetry_usageEvent_eventType` | string | UMA-3.0 | Event type from the primitive's controlled vocabulary |
| `umaModule_telemetry_usageEvent_moduleId` | string | UMA-3.0 | The `moduleId` of the host module (§5.2) |
| `umaModule_telemetry_usageEvent_timestamp` | string | UMA-3.0 | ISO-8601 timestamp of the event |
| `umaModule_telemetry_usageEvent_sessionId` | string | UMA-3.0 | Host-supplied session identifier for event grouping |

**Billing Transaction Event Keys** — shape of the `umaModule_billing_transactionEvent_*` payload emitted via the `onTransaction` callback (§13.2.11.2):

| Canonical Key | Type | Ver | Description |
| --- | --- | --- | --- |
| `umaModule_billing_transactionEvent_primitiveId` | string | UMA-3.0 | The primitive that surfaced the transaction |
| `umaModule_billing_transactionEvent_moduleId` | string | UMA-3.0 | The host module |
| `umaModule_billing_transactionEvent_tier` | string | UMA-3.0 | Subscription tier active at transaction time |
| `umaModule_billing_transactionEvent_timestamp` | string | UMA-3.0 | ISO-8601 timestamp |

**Tier-Gated Capability Keys:**

| Canonical Key | Type | Ver | Description |
| --- | --- | --- | --- |
| `umaModule_tier_capability_{primitiveId}_{capabilityId}` | boolean | UMA-3.0 | Whether capability `{capabilityId}` of primitive `{primitiveId}` is available at the current tier |

## 7.7 Deprecated Keys

> These keys remain in the registry for backwards compatibility. Do not use for new implementations.

> **Not every `publishingModule_newsletterStudio_*` key has an entry below.** `htmlExport` is absent deliberately, not by omission — it was ratified new at 2.4 (§7.5), never having been ratified under any prior name. This table records keys that *were* ratified and have since been superseded; it is not the right place for a key that was simply never here before.

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
| umaModule_usage_event | umaModule_telemetry_usageEvent_* | 2.4 |
| publishingModule_emailStudio_studioJson | publishingModule_newsletterStudio_studioJson | 2.4 |
| publishingModule_emailStudio_canvasDocument | publishingModule_newsletterStudio_canvasDocument | 2.4 |
| publishingModule_emailStudio_publishRecord | publishingModule_newsletterStudio_publishRecord | 2.4 |
| publishingModule_emailStudio_deliveryTarget | publishingModule_newsletterStudio_deliveryTarget | 2.4 |
| publishingModule_emailStudio_issueSubject | publishingModule_newsletterStudio_issueSubject | 2.4 |
| publishingModule_emailStudio_issueDate | publishingModule_newsletterStudio_issueDate | 2.4 |
| bank (persona field, §3.4) | bankName | 2.4 (2026-10-20) |
| references (persona field, §3.4) | businessReferences | 2.4 (2026-10-20) |

## 7.8 Deprecation Policy (new at 2.4)

A deprecated key remains defined in the Canonical Key Reference, marked `[DEPRECATED]`, with its replacement named — same shape §7.7 already uses.

**What deprecated means:** do not use in new work; existing usage is not broken by the deprecation itself.

**Validity:** a deprecated key remains valid indefinitely — this document never silently invalidates a ratified key (§14.3's retroactive-change prohibition already establishes this; this policy makes it explicit for this specific case).

**Removal:** a deprecated key is removed from the reference only when its own entry is amended to record zero known consumers — not on a fixed timer.

**On encountering a deprecated key:** a consumer should migrate to the replacement at its own next convenient revision; nothing requires an emergency migration, since deprecation alone never breaks an existing reader or writer.

---

# 8. AIX Envelope & Task Types

## 8.1 The AIX Envelope

Every AIX message is wrapped in an envelope. The `aix` field carries the specification version.

```json
{
  "aix": "2.4",
  "id": "uuid-v4",
  "timestamp": "ISO-8601",
  "task": { "type": "...", "provider": "...", "model": "..." },
  "input": { },
  "output": { },
  "meta": { "cost": { }, "latency": { }, "tokens": { } },
  "error": { }
}
```

## 8.2 Task Type Registry

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

## 8.3 Metadata Object

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

## 8.4 Error Object

```json
"error": {
  "code": "rate_limited",  // invalid_input | rate_limited | quota_exceeded |
                           // content_filtered | model_unavailable | timeout |
                           // provider_error | unknown
  "message": "Rate limit exceeded for this model.",
  "retryable": true,
  "retry_after_ms": 5000,
  "provider_error": { }   // Raw error object from provider
}
```

---

# 9. Task-Specific Schemas

## 9.1 content_authoring

The `content_authoring` task type enables AI-generated content carrying full author identity, editorial voice, structural guidelines, and hard guardrails.

### Input: author object

```json
"author": {
  "givenName": "Michael",
  "familyName": "Tchong",
  "displayName": "Michael Tchong",
  "jobTitle": "Founder, Solo Trillion",
  "bio": "Short author bio for attribution",
  "socialProfiles": {
    "linkedinUrl": "https://linkedin.example.com/in/example-author",
    "twitterUrl":  "https://x.example.com/example-author",
    "youtubeUrl":  "https://youtube.example.com/@example-author",
    "url":         "https://example.com"
  },
  "media": [
    {
      "mediumType": "newsletter",
      "mediumName": "Ubertrends",
      "mediumBrand": "Ubertrends",
      "isPrimary": true
    }
  ]
}
```

### Input: voice, research, brief

```json
"voiceBrand":      "...",
"voiceGuidelines": "...",
"voiceGuardrails": "...",

"researchPacket": {
  "topic":          "...",
  "thesis":         "...",
  "narrativeAngle": "...",
  "keyFindings": [
    { "finding": "...", "sourceUrl": "...", "confidence": 8, "isContrarian": true }
  ],
  "traceabilityTable": [
    { "finding": "...", "sourceUrl": "...", "supportingPassage": "...", "confidence": 8 }
  ],
  "auditSummary": { "auditPassed": true, "hallucinationsFound": 0 }
},

"contentBrief": {
  "format":            "default",
  "toneOverride":      "contrarian",
  "targetLength":      "standard_1200",
  "targetPublication": "Solo Trillion",
  "selectedFindings":  []
}
```

### Output

```json
"output": {
  "draft": {
    "headlineOptions": ["...", "...", "..."],
    "lede":    "...",
    "body":    "...",
    "sections": [{ "heading": "...", "content": "..." }],
    "cta":     "...",
    "wordCount": 1247,
    "citations": [{ "text": "...", "url": "...", "title": "..." }]
  },
  "guardrailFlags": [{ "flag": "...", "location": "...", "recommendation": "..." }],
  "finishReason": "complete"
}
```

## 9.2 voice_synthesis

```json
"input": {
  "text": "Welcome to the future of AI interchange standards.",
  "voice": "aria",  "language": "en-US",  "speed": 1.0,
  "emotion": "neutral",  "outputFormat": "mp3",  "sampleRate": 24000
},
"output": {
  "audio": { "url": "...", "format": "mp3", "sampleRate": 24000,
             "channels": 1, "durationMs": 3200, "fileSizeBytes": 51200 },
  "wordTimestamps": [{ "word": "Welcome", "startMs": 0, "endMs": 320 }]
}
```

## 9.3 image_generation

```json
"input": {
  "prompt": "A futuristic city at sunset, cyberpunk style",
  "negativePrompt": "blurry, low quality",
  "width": 1024,  "height": 1024,
  "style": "photorealistic",  "numOutputs": 1,
  "outputFormat": "png",  "seed": 42
},
"output": {
  "images": [{ "url": "...", "format": "png", "width": 1024, "height": 1024,
               "fileSizeBytes": 1048576, "revisedPrompt": "..." }],
  "nsfwDetected": false
}
```

## 9.4 text_generation

```json
"input": {
  "prompt": "...",  "systemPrompt": "...",
  "messages": [{ "role": "user", "content": "..." }],
  "maxTokens": 2000,  "temperature": 1.0,  "stream": false,
  "responseFormat": "markdown"
},
"output": {
  "text": "...",
  "finishReason": "complete",
  "citations": [{ "text": "...", "url": "...", "title": "..." }]
}
```

## 9.5 video_generation

```json
"input": {
  "prompt": "...",
  "image": { "url": "..." },
  "durationSeconds": 10,  "aspectRatio": "16:9",
  "resolution": "1080p",  "fps": 24,
  "motionIntensity": "medium",  "cameraMotion": "static"
},
"output": {
  "video": { "url": "...", "format": "mp4", "width": 1920, "height": 1080,
             "durationMs": 10000, "fps": 24, "fileSizeBytes": 8400000 },
  "thumbnail": { "url": "...", "timestampMs": 500 }
}
```

## 9.6 code_generation

```json
"input": {
  "prompt": "...",  "language": "python",
  "context": "...",  "framework": "fastapi",
  "includeTests": false,  "includeComments": true,
  "styleGuide": "pep8"
},
"output": {
  "code": "...",  "language": "python",  "explanation": "...",
  "files": [{ "filename": "main.py", "content": "...", "language": "python" }],
  "dependencies": [{ "name": "fastapi", "version": "^0.110.0" }]
}
```

---

# 10. Workflows & Chaining

AIX supports chaining multiple tasks. The output of one step becomes the input of the next via `${step_id.output.field}` variable substitution.

```json
{
  "aix": "2.4",
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

# 11. Provider Comparison Format

```json
{
  "aix": "2.4",
  "comparison": {
    "task": { "type": "voice_synthesis" },
    "input": { "text": "Hello, this is a comparison test.", "language": "en-US" },
    "results": [
      {
        "provider": "elevenlabs", "model": "eleven_multilingual_v2",
        "output": { "audio": { "url": "...", "durationMs": 2100 } },
        "meta":   { "cost": { "amount": 0.002 }, "latency": { "totalMs": 650 } }
      },
      {
        "provider": "openai", "model": "tts-1-hd",
        "output": { "audio": { "url": "...", "durationMs": 2080 } },
        "meta":   { "cost": { "amount": 0.003 }, "latency": { "totalMs": 580 } }
      }
    ]
  }
}
```

---

# 12. Canonical Registry Contract

This chapter defines an open-standard contract for how canonical registries expose scoped, capability-gated, auditable reads to agents. It was originally ratified as a separate document, AIX 2.3 (2026-08-02), framed at the time as a separate, coexisting standard alongside what was then AIX 2.2. That framing is retired at 2.4 — there is no sibling; this is one standard, and this chapter carries what was AIX 2.3's contract text, physically inlined here. STO's Persona serves as a **partial** reference implementation (§12.12 for exactly what it does and doesn't yet demonstrate).

**Scope exclusions, carried forward unchanged:** this chapter does NOT design Persona-Editorial's content itself (Stage-1-track work, see §12.2.2/§12.12). Does NOT design any domain namespace's content (Trademark schema, Investment schema, Health schema — separate future workstreams). Does NOT redesign block-first refactor Stage 1 (already ratified — this chapter conforms to it, doesn't override it). Does NOT design the `aix-standard.org` public-publication process.

## 12.1 Overview and Positioning

AIX generalizes a pattern already latent, but never made explicit, in this codebase: a **canonical registry** — a scoped, versioned, single-source-of-truth store that agents read from at dispatch time rather than caching or pinning locally. The motivating failure modes (per-agent pinning drift, per-file library divergence, disputes over which copy of a "canonical" fact is authoritative) are exactly the shape of defect this codebase has produced independently at least three times: the press-release dispatch bug (format-key drift across five enumeration surfaces), the block-first architecture refactor's own verification pass (`writing_system_prompts`/`writing_output_rules`'s `prompt_key`/`rules_key` naming, which doesn't mechanically derive from the canonical `format_key` at all), and Persona's own `ESSENTIALS_KEYS` drift (below).

**A fourth instance, found during this contract's original verification pass:** Persona's backend service module defines a field-key constant, `ESSENTIALS_KEYS` (`solo-trillion-agents/persona/app/persona_agent.py:25-29`), using Schema.org-canonical names — `givenName`, `familyName`, `preferredName`, `jobTitle`, `legalName`, `industry`, `addressHome`, `telephoneMobile`, `timezone`, `emailBusiness` (10 fields). The frontend-facing schema documentation instead lists an "essentials" category using pre-rename names — `firstName`, `lastName`, `company` (9 fields) — even though the field-key master list explicitly documents the rename this doc failed to pick up. Confirmed further at the migration layer: `migrations/20260328_aix_field_key_renames.sql:11-19` is the actual rename migration. This is not corrected in this chapter (out of scope — Persona's own documentation hygiene) but is cited deliberately: live, first-party evidence that even the reference implementation this chapter cites has already drifted internally.

**Reference implementation, precisely scoped (§12.12 for full detail):** STO's Persona (`app.solotrillion.ai/persona`) demonstrates the **tier axis** in production — a real, per-field, three-way classification (§12.4) — but does **not** currently demonstrate the capability axis, namespace/scope concept, grant lifetime, or resource-scoped grants this chapter specifies (§12.3, §12.5, §12.6, §12.9). Those are net-new design surface, using Persona's tier system as the closest existing analog, not as proof the full pattern already ships.

**External precedent:** Airtable's Personal Access Token model — orthogonal capabilities (`data.records:read`, `schema.bases:read`), base-scoped grants, per-token revocation. This is domain knowledge about an external product, not a claim about this codebase; cited as a design precedent for §12.3/§12.6, not grep-verified against any file here.

### 12.1.1 Open-standard positioning

Spec authoring targets external adoption from the start: STO as reference implementation, `aix-standard.org` as the eventual public spec site, other agent platforms encouraged to implement independently. Publication of the public-facing version is an explicit downstream workstream, not this chapter's scope.

**Gate on publication claims:** this chapter's text may publish now — the contract text doesn't need a running full-conformance system to exist first. But conformance CLAIMS, and specifically "Solo Trillion/Persona is THE reference implementation" language (unqualified), do NOT publish until a full-conformance reference actually exists somewhere in this codebase. Today, Persona satisfies only **incubation-profile** conformance (§12.12, Appendix E.5) — the shape is declared correctly, but handoff (§12.9), audit (§12.7), lifetime-expiry (§12.5), and denial paths aren't demonstrated end-to-end anywhere.

**"Universal Markdown" comparison.** The same open-contract/reference-implementation growth shape this chapter follows appears in the shipped Universal Markdown implementation: a canonical-writer-rules table implemented at `solo-trillion-relay/app/services/canvas_markdown_writer.py`, and a public validator at `solotrillion.io`, framed as "the developer adoption trojan horse."

## 12.2 Canonical-Registry Namespace Definition

A namespace is:
- A typed canonical store, single source of truth for a domain of ground facts.
- Supabase-backed or equivalent RDBMS-backed, with row-level security.
- Versioned in place (`version` column, `updated_at` timestamp) — no per-agent pinned copies allowed by contract.
- Read at dispatch time, not cached across sessions by agents, by contract.
- Scope-boundaried — agents declare needed namespaces at registration; they cannot silently drift to new ones (§12.8).

**STO's initial namespace set, with status as of this chapter's original ratification:**

| Namespace | Status |
|---|---|
| Persona-Identity | Shipping, tier-gated only — not yet namespace/capability-scoped per this contract |
| Persona-Editorial | Stage 1 addition (§12.2.2) |
| Legal-Structure | Proposed (Ubertrends LLC entity data) |
| Trademark-Filings | Proposed, for a future Trademark agent |
| Investment-Thesis | Proposed, for financial agents |
| Health-Record | Proposed, privacy-tier-critical example |
| Airtable-per-base | Proposed — each Airtable base exposed as a scoped namespace via MCP |
| `format_registry` / `block_schemas` | Not yet built — Stage 1 of the block-first refactor. These become AIX canonical registries once implemented. |
| Mobile-conducting client namespace access | See §12.10 — the mobile client is a CONSUMER profile across all namespaces above, not a namespace itself. |
| `dispatch-queue` | Operational (§12.2.3) — the pipeline MCP server's own dispatch-claim queue. First namespace registered as OPERATIONAL rather than canonical-fact under this contract. |

**Extension pattern for adding a namespace:** registration protocol, RLS setup, agent-scope-declaration re-negotiation (§12.8 — adding a namespace to an already-registered agent's scope requires the same ratification as initial registration, not a silent grant).

### 12.2.1 Reserved template namespace (fill-in-the-blank addition protocol)

A user-suggested namespace should be a fill-in-the-blank exercise against a clonable skeleton, not a fresh design cycle each time. Reserved template, instantiating every axis this contract defines:

```json
{
  "namespace_id": "{namespaceName}",
  "capabilities": ["records:read"],
  "tiers": ["public", "professional", "sensitive"],
  "lifetimes_supported": ["persistent", "session", "single-read"],
  "resource_scopes": ["{resourceClass1}", "{resourceClass2}"],
  "obligations": [],
  "audit_retention_days": 90,
  "issuance": "explicit-at-registration"
}
```

`obligations: []` — the fifth orthogonal axis (§12.6.1). Empty by default; a namespace declares which obligation types it supports/requires the same way it narrows `capabilities`/`tiers`/`lifetimes_supported`.

Addition protocol: (1) clone the template, (2) rename `namespace_id`, (3) fill `resource_scopes` with the namespace's actual resource-class boundaries, (4) narrow `capabilities`/`tiers`/`lifetimes_supported` from the full defaults if the domain requires tighter constraints (e.g., Health narrowing `lifetimes_supported` to `["session", "single-read"]` only), (5) register per §12.8.

This chapter does not attempt to prove the four axes (capability, tier, lifetime, resource-scope) are complete for every conceivable namespace. §12.3's namespace-specific-capability clause is the explicit escape valve for whatever a domain needs that the four standard axes don't express.

### 12.2.2 Persona-Editorial: Stage 1 addition

Persona-Editorial is a Stage 1 addition, migrating whatever document currently governs Rosie's editorial content rules into Supabase namespace form, read by Rosie at dispatch time alongside `format_registry` (§12.2). The source document states in its own scope section that it "governs writing, formatting, verification, terminology, article structure, and content handoffs," matching this section's editorial-content-rules domain.

**The retirement obligation, regardless of where the source document lives:** whatever currently governs Rosie's editorial content rules must be RETIRED in the same motion that lands the Persona-Editorial namespace — not left as parallel surface alongside the new namespace. Persona-Editorial's Stage 1 landing should carry its own acceptance criterion — "the source document is deleted or explicitly marked deprecated-and-empty in the same commit" — not merely "the namespace now also exists."

### 12.2.3 `dispatch-queue`: first namespace registered OPERATIONAL, not canonical-fact

**What it is:** the pipeline MCP server's own dispatch-claim queue — row-per-dispatch state (`workstream`, `stage`, `status`) that the internal agent pipeline claims and completes against.

**OPERATIONAL, not canonical-fact — capability and grant axes in force, version axis explicitly EXEMPT.** §12.2's own definition describes a namespace as a canonical store of ground facts an agent reads and trusts. `dispatch-queue` is a different kind of thing: mutable operational workflow state. The capability axis (§12.3) and grant-lifetime axis (§12.5) apply to `dispatch-queue` exactly as they do to any canonical-fact namespace. The version axis is the one that does not apply — there is no `version` column on `dispatches`, and none is owed: a `status` transition is not a fact revision needing a version history, it's the row doing its job.

**Namespace definition, per the reserved template (§12.2.1), narrowed to what this domain actually needs:**
```json
{
  "namespace_id": "dispatch-queue",
  "capabilities": ["dispatches:read", "dispatches:claim", "dispatches:complete", "dispatches:notify", "dispatches:stage"],
  "tiers": "documented-equivalent — single-classification (see below)",
  "lifetimes_supported": ["persistent"],
  "resource_scopes": "deferred to phase 2 (namespace-wide grant in v1)",
  "obligations": [],
  "audit_retention_days": null,
  "issuance": "explicit-at-registration"
}
```

Five namespace-specific capabilities (§12.3's own escape valve), not the template's generic `records:read` default. `persistent` is the only supported grant lifetime — an agent credential is a standing identity across many dispatches, not a per-dispatch grant. Resource scoping is namespace-wide in v1, per-workstream scoping deferred to Phase 2.

**Tier axis — documented-equivalent, not "not used."** Every field on a `dispatches` row sits at the same sensitivity level — there is no sub-classification within this namespace analogous to Persona's public/professional/sensitive split.

**Conformance target: incubation-profile (§12.12), not full.** `dispatch-queue`'s v1 auth model runs over `stdio` with local trust — "who is this caller really" is NOT cryptographically enforced in v1; the relay trusts its own local process boundary rather than verifying per-agent identity. Full registry conformance is explicitly Phase 2/3 work for this namespace.

**`dispatch-queue` is also the first OPERATIONAL namespace under this contract** — a genuine category distinction this chapter's own §12.2 definition did not previously draw. A future namespace-kind taxonomy amendment is where this distinction formally belongs; this entry is a registration-time workaround, not the permanent home for the taxonomy.

## 12.3 Capability Axis (orthogonal to tier)

Enumerated capabilities per namespace type:
- `records:read` — read records.
- `records:write` — write records directly (privilege-escalating, most gated).
- `schema:read` — read the shape of records (Airtable's `schema.bases:read` precedent).
- `proposals:write` — submit a proposed change to a review queue that a user or approver-agent ratifies before landing.
- `stream:read` — read canonical content as part of an active dispatch's LLM prompt, without persistent caching; short-lived and session-bound implicitly.
- Namespace-specific capabilities are allowed for special domains, narrower than an early draft framed them — see the correction below.

Contract requires namespaces to enumerate their supported capabilities explicitly. Agents cannot request capabilities a namespace doesn't declare.

**The namespace-specific-capability escape valve is narrower than an early draft implied.** Capability strings are for WHAT can be done — read/write/schema mechanics only. Health's `emergency:read`, once considered "distinct from `records:read`," smuggled an unstated CONSTRAINT into a capability string with no declared semantics for what made it different beyond its name. That constraint belongs on the Obligations axis (§12.6.1), not encoded as an ad hoc capability string. Health's emergency-access case is `capability: records:read` + `obligation: post-hoc-audit-required` — the capability stays ordinary `records:read`; the obligation carries the actual constraint.

This capability list is the runtime grant vocabulary a `permissionScopes` declaration (chapter 5.2.1) is satisfied against — see chapter 5's own composable relationship. It does not redefine `permissionScopes`' own string vocabulary; a module manifest is never rewritten to carry capability strings directly.

## 12.4 Tier Axis (orthogonal to capability)

**This axis has real, verified, shipping prior art in Persona — the one part of this contract with a genuine production reference.**

Persona implements a per-field, three-way tier classification, with two parallel naming schemes deliberately mapped to each other:
- DB column values (`permission_tier`, `migrations/001_baseline.sql:100-101`'s `CHECK (permission_tier IN ('public', 'trusted', 'private'))`): `public` / `trusted` / `private`.
- UI-facing tier names (`solo-trillion-frontend/src/data/personaData.js:108-112`): `Public` / `Professional` / `Sensitive`.
- Explicit bidirectional mapping (`solo-trillion-frontend/src/components/persona/PersonaModule.jsx:22-33`):
  ```js
  const backendToFrontendTier = { public: 'public', trusted: 'professional', private: 'sensitive' };
  const frontendToBackendTier = { public: 'public', professional: 'trusted', sensitive: 'private' };
  ```

Tier is assigned per field, not per category. Read-time gating is a simple hierarchy comparison (`personaData.js:184-187`):
```js
function shouldIncludeField(fieldTier, filterTier) {
  const hierarchy = { public: 0, professional: 1, sensitive: 2 };
  return hierarchy[fieldTier] <= hierarchy[filterTier];
}
```

There is no fixed field count per tier anywhere in this codebase — tier is a per-row, user-editable value.

## 12.5 Grant Lifetime (orthogonal to capability and tier)

- **Persistent** — grant survives across sessions until explicitly revoked.
- **Session-only** — revoked at end of the dispatch that requested it.
- **Single-read** — revoked immediately after one successful read.

Namespaces can require minimum lifetime constraints (Health might require session-only-or-less for all its capabilities). Agents cannot request longer lifetimes than a namespace allows.

Persona demonstrates **none** of these three today. Tier assignment (§12.4) is a persistent field property with no lifetime concept at all. This entire axis is net-new design surface.

## 12.6 Resource-Scoped Grants Within Namespaces

Grants scoped to specific resources, following the Airtable-PAT precedent. For non-Airtable-shaped namespaces, resource scoping means:
- Per-record-class scoping (a Trademark-Filings grant restricted to USPTO records, not international).
- Per-record scoping (an Investment-Thesis grant restricted to public-market positions, not private-company).
- Per-field-set scoping (a Persona-Identity grant restricted to the Essentials category, not addresses or biographical fields).

Grant is fully specified by the tuple: `(namespace, resource-scope, capability-set, tier-max, lifetime, obligations[])`.

Persona's existing category grouping (essentials/professional/social/financial/organization/preferences/conversationalAgent) provides UI STRUCTURE a per-field-set grant could map onto — but today, access is gated by TIER alone, not category. Implementing category-scoped grants is new work.

### 12.6.1 Obligations Axis (fifth orthogonal concept)

**Not a peer to capability/tier/lifetime/resource-scope — those four stay exactly as specified above, unchanged.** Obligations are a structured, typed constraint layer attached to a grant, enforced actively rather than checked as bare string-presence. Where a capability answers "what can be done" and a tier/lifetime/resource-scope answer "how much, how long, over what," an obligation answers "under what continuing condition."

**Obligation types:**
- `purpose-limitation` — grant valid only for a declared purpose string, checked against namespace-declared allowed purposes.
- `consent-required` — grant inactive until an explicit consent record exists.
- `human-approval-required` — grant inactive until a human (not an approver-agent) explicitly signs off.
- `redact-before-persist` — any downstream persistence of the read value must redact/mask it first.
- `session-only-use` — narrower than the lifetime axis: constrains USE within a session even if the grant's lifetime is technically persistent.
- `post-hoc-audit-required` — access (e.g., an emergency read) must be justified within N hours or gets auto-flagged.
- `no-further-forward` — marks a value that crossed into a handoff payload as usable-once but not re-forwardable to a third agent (§12.9).

**Interaction with §12.3's escape valve:** namespace-specific capabilities are for mechanics only (read/write/schema); anything constraint-shaped that used to get folded into a bespoke capability string now expresses as capability + obligation instead.

**Enforcement, not just declaration — three distinct modalities, not uniformly mechanical:** **(1) pre-read gates** (`consent-required`, `human-approval-required`) — genuinely mechanical, since the system controls whether the read executes at all. **(2) downstream-use constraints on structured data still inside system-controlled storage/transport** — mechanical, since the system controls the write path. **(3) downstream-use constraints on a value once it has entered an LLM agent's free-text context** (`session-only-use`, `no-further-forward` on a value embedded in a handoff payload/prompt/brief) — **ADVISORY ONLY, not mechanically enforceable by the system once the value is inside model-generated text.** This is a known, current limitation of LLM-mediated handoffs: enforcement of modality (3) rests on agent compliance plus post-hoc audit (§12.7), not a technical block.

**Derivative values, same modality-3 limitation:** obligations travel with the underlying FACT a value conveys — including paraphrases, summaries, or extracts that substantively restate it. Detecting that agent-generated text IS such a derivative is itself modality-3 (advisory), never a mechanical pattern-match.

## 12.7 Audit Contract

What every canonical read logs: agent identity (agent-id, dispatch-id if applicable), namespace read, resource(s) accessed, capability exercised, tier of data accessed (max tier touched during the read), grant type (persistent/session/single-read), timestamp, result (`success`, `denied-by-tier`, `denied-by-capability`, `denied-by-scope`, `denied-by-lifetime`, `denied-by-obligation`), and obligation-fulfillment state — was each attached obligation satisfied, not just whether the grant/access itself succeeded. Retention policy: 90 days minimum. Query interface must expose a readable audit trail to the user.

Persona has a real UI tab called "Access Log" (`PersonaModule.jsx:19`, rendering `AccessLog.jsx`), the right shape — but it is currently mock data, not a live audit trail. A real, if partial, audit primitive DOES already exist: the `conversations` table carries a `persona_fields_accessed TEXT[]` column (`migrations/001_baseline.sql:146`), written by `conversations.py`/`agent.py`, tracking which persona fields were read per conversation turn. It does NOT carry capability, tier-touched, grant-type, or result fields, so it satisfies only a fraction of this section's audit contract, and it is architecturally separate from the (mock) Access Log UI. Wiring the Access Log UI to `conversations.persona_fields_accessed` (extended with the missing fields) is a smaller lift than building an audit system from nothing.

## 12.8 Grant Issuance Discipline

Explicit-at-agent-registration. No blanket-future-namespace grants. Adding a namespace to an agent's scope after initial registration requires user (or authorized-approver) ratification. Grants are user-revocable per-grant, not just per-agent. Bulk-grant patterns are explicitly discouraged — an agent requesting "all namespaces" is non-compliant with this contract.

**Grant-downgrade, acknowledged as a limitation, not silently fixed:** if a namespace issues two grants of differing strictness to the same underlying resource, an agent holding the LAXER grant can legitimately access that resource with fewer obligations than a stricter grant would carry for the same fact — including by simply re-reading under its own laxer grant. This is legitimate, not a bypass, since it's a genuinely different, independently-issued grant. Namespaces that want to prevent this should declare a minimum obligation FLOOR per tier, applying to ALL grants issued at that tier for that resource. Where a namespace does not declare such a floor, grant-downgrade-via-independent-re-read is a known, accepted possibility under this contract.

## 12.9 Cross-Agent Handoff Contract

**When agent A hands a task to agent B (e.g., Rosie handing a writing dispatch to the Writing Agent), agent B does NOT inherit agent A's scopes.** Two mechanisms, differentiated by tier and by what travels with the value:

**1. Payload-embedding — Public-tier values only.** A reads a value under its own grant and embeds the raw value directly into B's handoff payload. Restricted to Public-tier data. The value carries forward, attached to it, whatever obligations were attached to A's read grant. B is REQUIRED to treat an obligation-bearing embedded value as bound by its attached obligations — `session-only-use` bars persistence past B's current session; `redact-before-persist` requires masking before any write B makes; `no-further-forward` bars B from embedding that same value into a further handoff to C. **This is squarely modality (3) — advisory, not mechanically enforced.** This section specifies the constraint and requires it be declared and carried; enforcement rests on agent compliance plus post-hoc audit (§12.7).

**2. Grant-forwarding transaction — required for Professional and Sensitive-tier values.** A requests a grant-forwarding transaction that gives B its own scoped, time-limited, capability-limited grant derived from A's grants — never broader than A's own grants, and audited as a distinct grant issuance (§12.7, including its own obligation set). Professional-tier values require grant-forwarding, not just Sensitive-tier, since Persona's own Professional-tier definition ("released to agents you specifically approve," §12.4) already implies a per-agent approval step that payload-embedding's looser obligation-carrying model doesn't provide. Only Public-tier values may cross via the lighter payload-embedding path.

**What distinguishes B's independently-granted access from an obligation-bearing embedded value:** if B separately holds its OWN grant to the same underlying namespace/resource, B's use of that independently-read copy is governed by B's OWN grant and its OWN obligations — not by whatever obligation traveled with a DIFFERENT copy A happened to embed in a payload. The obligation attaches to the specific value-instance that crossed via the handoff, not to the underlying canonical fact itself.

This is what prevents sub-agent spawn from being implicit privilege escalation, and it is what makes the Rosie-as-conductor model formally auditable.

**Grounding:** the existing Rosie→Writing-Agent handoff mechanism, the `<!--DISPATCH:-->` HTML-comment tag convention (`solo-trillion-relay/app/services/system_prompt.py:892`; parsed via `DISPATCH_RE` at `solo-trillion-frontend/src/lib/dispatch.js:18`), carries a plain JSON payload (`action`, `topic`, `format_key`, `source_count`, `brief`) with no scope, grant, or obligation concept anywhere. Today, ANY value Rosie chooses to embed in `brief` crosses with zero obligation-tracking of any kind — the unconstrained-payload-embedding shape this section closes.

## 12.10 Mobile Conducting-and-Reviewing Client Profile

**Scope, explicit: a mobile CONDUCTING-AND-REVIEWING client, not a coding client.** This section specifies what an AIX-conformant mobile client exposes, so any implementation builds to one shape rather than renegotiating platform-by-platform. Full mobile CODING access is explicitly out of scope.

**Streaming-artifact receipt.** A mobile client must be able to receive a canonical read's or a dispatch's output as it streams, not only as a completed poll result — the same SSE/streaming discipline the block-first canvas pipeline already uses server-side, extended to a mobile transport.

**Push-notification schema.** A minimal, namespace-agnostic event: draft-complete (or equivalent dispatch-complete signal) → notify. The schema must carry enough for the client to deep-link directly to the artifact (namespace, resource-id, dispatch-id) without a follow-up poll to resolve "what finished."

**Canonical-read-from-mobile: the same dispatch-time read discipline as desktop.** A mobile client is a consumer like any other under §12.2 — it reads namespaces at dispatch/render time, it does not locally pin or cache canonical facts across sessions.

**Background-dispatch state model.** A conformant client's dispatch-state model must survive suspension — a dispatch fired, then the app backgrounded, then resumed, must reflect the CURRENT state rather than the state at backgrounding time.

**Why this is a profile, not a mobile-specific contract fork:** network variability, background suspension, touch input, and limited concurrent tasks constrain HOW a client implements streaming-receipt/push/read-discipline/background-state, not WHAT those four things mean. Any AIX-conformant client implements the same four; a mobile client's constraints shape its implementation, not the contract.

## 12.11 RLS Enforcement Mechanics

Reference-implementation guidance for external implementers: Supabase RLS policies as the mechanism, JWT-scoped tokens carrying agent identity and current grants, database-layer enforcement (not application-layer) so bypass would require compromising the RDBMS. Cross-store federation for canonical stores held in different systems (e.g., Airtable via MCP).

**Core Persona identity table's RLS.** `persona_fields` is created and RLS-enabled at `migrations/001_baseline.sql:91-127`:
```sql
ALTER TABLE public.persona_fields ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view own persona fields"
  ON public.persona_fields FOR SELECT USING (auth.uid() = user_id);
CREATE POLICY "Users can insert own persona fields"
  ON public.persona_fields FOR INSERT WITH CHECK (auth.uid() = user_id);
CREATE POLICY "Users can update own persona fields"
  ON public.persona_fields FOR UPDATE USING (auth.uid() = user_id);
CREATE POLICY "Users can delete own persona fields"
  ON public.persona_fields FOR DELETE USING (auth.uid() = user_id);
```
All four policies are ownership-scoped only (`auth.uid() = user_id`) — none are tier-aware. `permission_tier` (§12.4) is enforced purely as a `CHECK` constraint on the column value; no RLS policy references it. This means tier-based access control happens entirely at the application layer today, not at the database layer — any query that bypasses the application's filtering logic would return ALL of a user's own fields regardless of tier. This section's reference-implementation guidance (database-layer, not application-layer, enforcement) is therefore a genuine gap relative to Persona's current posture (§12.12).

## 12.12 Compliance Checklist for Canonical-Registry Implementations — two named tiers

**Incubation-profile conformance** — the shape is declared correctly: namespace registration; capability enumeration; tier system (three tiers or a documented equivalent) correctly declared. This is a lower, achievable bar, and it's what Persona satisfies today.

**Full conformance** — every incubation-profile item PLUS all remaining checklist items demonstrated end-to-end in a running system, not just declared in a spec: grant lifetime support (all three) actually enforced; resource-scoped grants actually gating access; an audit log with §12.7's required fields (including obligation-fulfillment state) actually populated by real reads, not mock data; explicit-grant-issuance discipline actually preventing blanket-future grants; cross-agent handoff contract (§12.9) actually implemented; obligation declaration and enforcement (§12.6.1) actually checked at runtime; and denial paths actually exercised and logged, not just enumerated as possible states.

**Persona today satisfies incubation-profile conformance only.** Its tier system is correctly declared (three tiers, per-field), but even that one demonstrated axis has a real gap (tier enforcement is application-layer, not database-layer, §12.11). Every other line item is either partially present (Access Log UI shape without a live backend, §12.7; a partial field-access log in `conversations.persona_fields_accessed`) or entirely absent (capability enumeration, grant lifetime, resource scoping, handoff contract, obligations).

**What Persona does NOT yet have, needed for full conformance:**
- **No namespace/capability axis at all.** Access today is tier-gated only.
- **No grant lifetime concept.** Tier assignment is persistent-only.
- **No resource-scoped grants beyond the tier hierarchy.**
- **No live audit backend wired to the Access Log UI** — though `conversations.persona_fields_accessed` (§12.7) is a real, running, partial primitive.
- **No cross-agent handoff protocol** (§12.9).
- **Database-layer tier enforcement is absent** — RLS is ownership-scoped only (§12.11).

Persona-extension to close this delta is its own future workstream, not this chapter's scope — but the delta itself is the concrete input to whoever scopes that workstream.

---

# 13. UMA 3.0 Primitive Contract Structure

This chapter carries the UMA 3.0 UI primitive contract structure in full — the sixteen-primitive registry, the four ratification preambles, the ten contract sections every primitive contract must contain, and the four UMA key families. The UMA 3.0 Contract Meta-Framework stays a maintained companion document; its §3 is the working text this chapter is derived from, and a divergence between the two is reconciled here at the next AIX minor version rather than left standing.

## 13.1 Primitive Registry

Sixteen primitives are registered as of this chapter's ratification. **Fourteen are `experimental`, two are `stable`. This is not "fourteen unfinished extractions"** — every one of the fourteen has real, shipped, live consumers, and every one's own contract states its gaps honestly, by name. "Experimental" here means "shipped with named, tracked accessibility or coverage gaps," not "not yet in use."

| Primitive ID | Description | Contract status | Consumers (live) |
|---|---|---|---|
| `useModuleTabs` | Hook: tab state, routing, and history management for module-level tabs | `experimental` — automated test coverage is the named gap | 4 (Rosie, Research, Persona, Newsletter Studio) |
| `blockHoverOverlay` | Hover-revealed control cluster anchored to a rendered block | `experimental` — reduced-motion gating not honored in 1.0.0 | `canvas/BlockComplete.jsx` |
| `blockMoreMenu` | Flyout from a More (···) button | `experimental` — no initial focus management, no focus restoration, Tab moves focus outside the open menu | Writing (wrapper), Newsletter Studio, Invoice Studio |
| `moduleFields` | Field rendering surface for editing a module's typed fields | `experimental` — arrow-key navigation not implemented; section headings lack heading markup | `NewsletterStudioAside.jsx` ModuleEditor |
| `accordion` | Collapsible disclosure panel | `experimental` — no `aria-controls`/`aria-labelledby`; tall-content clipping; closed panel remains Tab-focusable | 4 Newsletter Aside consumer sites |
| `glassDropdown` | Glass-styled select/dropdown trigger | `experimental` — no arrow-key navigation, no focus-into-panel on open, no focus restoration on close | Writing's 3 inline dropdowns; Persona migration pending |
| `statusDot` | Status indicator dot with optional glow, color-parameterized | `experimental` — CSS-variable color tokens and accent-glow fallback not yet wired | Rosie, Research, Newsletter Studio |
| `skeleton` | Loading placeholder mirroring the shape of loading content | `experimental` — no automated test coverage is the sole remaining gate | 13 ad-hoc patterns consolidated across 10 files |
| `floatingInput` | Label-above text input field with floating label transform | `experimental` — reduced-motion gate not applied to `.floating-label` transition | `AuthGate.jsx`, `BetaForm.jsx`, `WaitingPage.jsx` |
| `floatingTextarea` | Label-above multiline text field | `experimental` — reduced-motion gate, no automated tests, `dotSlot` positioning unvalidated | `BetaForm.jsx`, `WaitingPage.jsx` |
| `toggle` | Toggle switch (on/off binary input) | `experimental` — no built-in focus ring via `focus-visible:`; no `label`/`labelSlot` | Subscriptions (3 call sites) |
| `toast` | Transient status notification | `experimental` — no ARIA role, no `aria-live` region, no pause-on-hover, no automated tests | `ProtectedLayout.jsx`, Subscriptions, `ModulesPage.jsx` |
| `confirmModal` | Confirmation dialog: title, body, confirm/cancel actions, optional dangerous variant | `experimental` — no focus trap, no focus restoration on close, no reduced-motion handling | Rosie, Research, Subscriptions |
| `backButton` | Back navigation button with dropdown to other agent surfaces | `experimental` — no ARIA labeling, no keyboard support beyond click, no focus management, reduced-motion gating unconfirmed, no test coverage | `useModuleTabs.jsx` (default), `ModulesPage.jsx`, `WritingModule.jsx`, `WritingHistoryReader.jsx` |
| `categoryIcon` | Category-to-icon resolver for data-driven rendering surfaces | **`stable`** — ratified at consumer #2 merge, per the Experimental → Stable Graduation Path (§13.2.3) | `NewsletterActionBar.jsx` (n=1), `blockIcons.js` (consumer #2) |
| `dragDrop` | Drag-and-drop primitive family | **`stable`** at merge — n=1 was comprehensive, the one primitive that skipped the experimental stage entirely | `NewsletterCanvas.jsx`, `NewsletterBlockPalette.jsx` |

## 13.2 Contract Structure

This section has **fourteen numbered subsections**. That number describes this chapter's own structure, not a contract requirement. §13.2.1 through §13.2.4 are ratification history — the four preambles under which the structure was ratified (PR #101–#103) — and §13.2.5 through §13.2.14 are the **ten sections every UMA 3.0 primitive's AIX contract document must contain, in this order**: Inputs, Outputs, Behaviors, Accessibility Commitments, Theming Surface, Context-Purity Audit, Monetization-Supporting Contracts, Reference Implementation, Conformance Test Suite, Versioning and Breaking Change Policy. A contract document that omits any of the ten is not AIX-compliant; combining sections is permitted only where explicitly noted below. A contract document does not carry §13.2.1–§13.2.4 as sections.

These requirements codify the contract structure already exercised by the UMA primitive library. They govern every new primitive contract; earlier component-API contracts adopt the structure as their consumers validate the applicable requirements (§13.2.1).

### 13.2.1 The canonical skeleton

Four elements are universal: the header, §13.2.12, §13.2.13 and §13.2.14. Every primitive contract, regardless of category, must include them exactly as structured: the **Header** (name, category, version, status), **§13.2.12 Reference Implementation**, **§13.2.13 Conformance Test Suite** (a machine-readable JSON file at `docs/aix/conformance/<primitive>.test.json`, scenarios split into `automated` — asserted programmatically — and `manual` — human-validatable conditions like touch velocity or visual gestalt, each carrying a `procedure` field), and **§13.2.14 Versioning Policy**.

**Behavioral primitives** additionally carry §13.2.5-§13.2.11 as a **v1-extensible** block — locked into the canonical skeleton only once a *second* behavioral primitive validates the same section shape. `dragdrop.md`'s own §13.2.5-§13.2.11 (interaction model, cursor contract, drop indicator, drag overlay, keyboard accessibility, screen-reader announcement, data transfer) are the first instance; they graduate into this skeleton when a second behavioral primitive validates and ratifies the same section shape, not before.

**Rendering primitives** author their own v1 sections independently — `categoryicon.md` was the first rendering-primitive contract and established the pattern (viewBox normalization, token contracts) that a second rendering primitive would validate the same way.

**Component-API primitives** (the majority of §13.1's table — `confirmModal`, `toast`, `backButton`, and the ten others) predate this skeleton entirely (PR #75-#84, before PR #101's ratification) and were never retrofitted to it as a formal action; they conform to the skeleton retroactively as their consumers ratify, not as a single rewrite pass. Their §13.2.5-§13.2.14 sections read as ten flat, equally-weighted sections (the meta-framework's own original v1.0.1 shape) rather than skeleton-plus-tier — both shapes are valid AIX 2.2/2.4 contracts; the skeleton concept only started governing *new* contracts from PR #101 forward.

### 13.2.2 Three-tier section split

Every contract's own sections split into three tiers, stated explicitly rather than left as an implicit "all v1 sections are equally generalizable" assumption (the assumption PR #101's own §13.2.5-§13.2.11 initially made, corrected at PR #102):

- **Generalizable sections** — candidate template for future primitives of the same category. Graduate to the canonical skeleton at the *second* primitive that shares the section's shape (same mechanism as §13.2.1's behavioral-primitive rule, generalized to any category).
- **Primitive-specific sections** — this primitive's own domain only. Never graduation candidates.
- **Candidate-future sections** — surface-area notes only, not authored. A future primitive may need something like this; it is not ratified until a primitive actually authors it.

### 13.2.3 Experimental → Stable Graduation Path

When a primitive's `n=1` reference consumer doesn't exercise the full load-bearing API surface, its contract ships `experimental` (§13.1's table shows this is true of 14 of 16 primitives today) and graduates to `stable` at the merge that lands its **consumer #2**. `categoryIcon` is the first primitive to exercise this end-to-end: `experimental` at PR #102 (n=1, `NewsletterActionBar.jsx`), `stable` at PR #103 (consumer #2, `blockIcons.js`).

**The graduation work lands in consumer #2's own PR, not a separate post-merge write, and decomposes into three commits whose atomicity matches the logical change:**

1. **Migration commit** — consumer #2's own migration to the primitive. One logical change, one commit.
2. **Atomic status-flip commit** — the AIX contract's `status` field AND the conformance JSON's own `status` field, `experimental` → `stable`, together, in one commit. Splitting these across two commits creates a self-inconsistent intermediate state (contract says stable, JSON still says experimental) that isn't a useful bisect target.
3. **Version + tag commit** — a version bump (a status-flip is metadata, not an API-surface change — MICRO/4th-digit, PATCH in the older 3-digit convention the meta-framework's own §8 still uses) plus changelog plus a paired `git tag`, in the same commit, no deferred tagging.

`dragDrop` (PR #101) is the one exception: its `n=1` was comprehensive (drag + reorder + cancel + keyboard + a11y all exercised by the Newsletter Studio migration), so it ratified `stable` at merge with no graduation path needed. That is the exception, not the default — the default for a new first-of-kind primitive is to ship `experimental` and follow the three-commit pattern above when consumer #2 lands.

### 13.2.4 Contract Claims Must Match Contract Mechanism

A contract claim of the shape "consumer is responsible for X" is only honest when the consumer actually has a mechanism to discharge that responsibility. When the primitive's own mechanism doesn't deliver what the claim promises, the claim is contract dishonesty — fixed by narrowing the claim or by adding the missing mechanism, never by shipping the gap silently. First surfaced at PR #101 (a portaled-token `:root`-scope claim the implementation didn't actually deliver) and generalized at PR #102. **This principle is why §13.1's table above states each `experimental` primitive's actual gaps by name rather than a bare status word** — a status field alone is a claim; the named deviations are the mechanism by which a reader can verify it.

---

### 13.2.5 Inputs

Describes what the primitive accepts. This section is framework-agnostic: describe shapes abstractly using prose and table notation, not React TypeScript declarations. Framework-specific declarations (TypeScript `Props` interfaces, JSX syntax) appear in the reference implementation (§13.2.12), not in the contract.

**Required content:**

- **Required inputs** — name, abstract type, and plain-English description for every required input
- **Optional inputs** — name, abstract type, default value, and description for every optional input
- **Slot composition surfaces** — named regions where the consumer injects custom content (equivalent to React `children`, render props, or named slots in other frameworks). **Naming discipline:** name slots by position, not by presumed content type. Use `leading`, `trailing`, `prefix`, `suffix` rather than `icon`, `badge`, `avatar`. Position-based names are honest about the slot shape (ReactNode-accepting, content-agnostic) and don't constrain future consumers to the mental model implied by the original use case. Example: `items.leading` in `GlassDropdown` correctly accepts both a decorative violet dot (Writing) and semantic tier-color dots (Persona) — renaming from `items.icon` was necessary because neither use is properly an icon.
- **Event handlers** — callbacks the primitive invokes, with their invocation conditions and payload shapes
- For hooks (`useModuleTabs`): input parameter object shape and all configuration fields

**Example table format:**

| Input | Type | Required | Default | Description |
|---|---|---|---|---|
| `title` | string | yes | — | Heading rendered in the dialog title region |
| `body` | string | yes | — | Body text rendered in the dialog content region |
| `confirmLabel` | string | no | `"Confirm"` | Label for the confirm action button |
| `cancelLabel` | string | no | `"Cancel"` | Label for the cancel action button |
| `onConfirm` | `() => void` | yes | — | Invoked when the user activates the confirm button |
| `onCancel` | `() => void` | no | `() => {}` | Invoked when the user activates the cancel button or dismisses via Escape |
| `isDangerous` | boolean | no | `false` | When true, the confirm button renders in destructive visual state |

**SDK minimalism discipline:** When deciding whether a variation belongs as a prop or at the consumer's call site, apply this rule: if the variation is purely visual and can be expressed via `className`, it stays at the call site. Primitives expose behavior; consumers compose visual. Example: `Skeleton` does not expose a `widths` prop even though consumers use different widths — each consumer passes its width via `className`. Adding `widths` as a prop would grow the SDK surface without adding behavioral capability. Reserve props for inputs that change behavior, accessibility state, or content — not for every visual variant a consumer might want.

**AIX naming requirement:** All input names pass the six validation tests from §2.6 of this standard — stranger test, hierarchy test, abbreviation test, case test, redundancy test, conflict test.

---

### 13.2.6 Outputs

Describes what the primitive produces. Visual output is described abstractly — not as a specific DOM structure or className string. Implementation details belong in the reference implementation (§13.2.12); the contract describes the rendered shape and semantic structure.

**Required content:**

- **Visual rendering** — abstract description: "renders a modal overlay with backdrop, title region, content region, and action region containing two buttons"
- **Returned values** (for hooks only) — the shape of the returned object, field-by-field, with types and descriptions
- **Emitted events** — events the primitive fires that consumers can listen to beyond the input handler callbacks
- **DOM semantics** — the primary ARIA role (described abstractly; the reference implementation decides whether to use native HTML or ARIA attributes)

---

### 13.2.7 Behaviors

Describes how the primitive behaves over time. Behaviors are normative — implementations must honor them. The conformance test suite (§13.2.13) verifies conformance.

**Required content:**

- **State transitions** — internal states the primitive owns, and what triggers transitions between them (e.g., `closed → open → confirmed | canceled | dismissed`). A state transition diagram is recommended but not required.
- **Side effects** — every DOM mutation, focus management operation, scroll lock, localStorage read/write, or external API call the primitive performs. The list must be exhaustive — undocumented side effects are not permitted in AIX-compliant primitives.
- **Lifecycle** — behavior on mount (initial state, initial focus placement), unmount (cleanup, restored state), and input changes (whether internal state resets or persists when inputs change)
- **Concurrency** — behavior under rapid repeated input. Required to document: double-click on confirm, rapid open/close cycles, input changes mid-animation. For each: state what the primitive does. Do not document what it "should avoid." **Multi-instance coordination discipline:** when multiple instances of a primitive coexist on the same surface (e.g., three dropdowns in a row), the default assumption is that browser event ordering handles coordination — mousedown-close fires before click-open on a sibling trigger, so independent uncontrolled instances naturally enforce "only one open at a time" without SDK-level coordination state. Verify empirically before adding coordination logic to the primitive. Document the ordering behavior in §13.2.7 Concurrency when it is the mechanism of record (see `GlassDropdown` PR #79).

---

### 13.2.8 Accessibility Commitments

Describes what accessibility guarantees the primitive makes. These are contractual obligations — not aspirational notes. An implementation that violates any commitment in this section fails the conformance test suite (§13.2.13) on `required` severity scenarios.

**Required content:**

**ARIA role and required attributes** — primary ARIA role and every ARIA attribute the primitive must set. Example:

| ARIA requirement | Value | Notes |
|---|---|---|
| `role` | `dialog` | Set on the root container element |
| `aria-modal` | `true` | Indicates content outside dialog is inert |
| `aria-labelledby` | `{id of title element}` | References the rendered title |

**Keyboard interaction contract** — complete keyboard interaction table:

| Key | Condition | Action | Event consumed |
|---|---|---|---|
| Tab | Dialog open | Moves focus forward within dialog (focus trap) | Yes |
| Shift+Tab | Dialog open | Moves focus backward within dialog (focus trap) | Yes |
| Escape | Dialog open | Invokes `onCancel`, closes dialog | Yes |
| Enter | Focus on confirm button | Invokes `onConfirm`, closes dialog | Yes |
| Enter | Focus on cancel button | Invokes `onCancel`, closes dialog | Yes |
| Space | Focus on confirm or cancel button | Same as Enter | Yes |

**Focus management contract** — where focus moves on open, where it returns on close, whether a focus trap is active.

**Screen reader announcements** — any ARIA live regions or `role="alert"` elements the primitive creates, and when they fire. **Group announcement discipline:** when a primitive renders N items that collectively represent one loading/status state (e.g., a skeleton list), the wrapper carries the single ARIA announcement (`role="status"`, `aria-label="Loading..."`) and each child carries `aria-hidden="true"`. Do not apply `role="status"` or live-region semantics to each child — N announcements for one conceptual state is an accessibility regression. `Skeleton` (PR #81) establishes this as the canonical pattern for multi-item loading primitives.

**Reduced-motion compliance** — how animations are suppressed when `prefers-reduced-motion: reduce` is active. If the primitive has no animations, state that explicitly.

---

### 13.2.9 Theming Surface

Describes the primitive's theming API — how consumers and host platforms control its visual presentation.

**Required content:**

**CSS custom properties consumed** — every CSS variable the primitive reads:

| CSS variable | Value type | Fallback | Description |
|---|---|---|---|
| `--glass-modal-bg` | color | `rgba(15, 15, 30, 0.95)` | Modal surface background |
| `--glass-modal-border` | border | `1px solid rgba(255,255,255,0.1)` | Modal surface border |
| `--btn-glow` | box-shadow | `0 0 12px rgba(99,102,241,0.3)` | Button glow ring |
| `--glass-pill-bg` | color | `rgba(255,255,255,0.08)` | Pill button surface |

These variables are the theming API surface. Consumers override these to brand the primitive. The STO glass aesthetic defines the defaults in `index.css`.

**`className` prop (canonical surface-injection pattern):** For primitives that render a root container with visual identity (card surface, pill surface, modal surface), declare a `className` prop that the consumer supplies. The primitive applies it to the root container; the consumer controls the visual surface treatment. Example: `Accordion` accepts `className` — Newsletter passes `"glass-card glass-card--no-glow"`, Invoice Studio may pass a different surface class. This is the theming-decoupled-from-behavior commitment made concrete: **behavior in the primitive, visual identity injected by the consumer.** Do not hardcode STO glass class names inside the primitive. If the primitive has a reasonable default surface, it may use one — but the consumer must be able to override it without modifying the primitive.

**Theming slots** — named regions where consumers inject fully custom chrome. Example: a title slot that accepts any content, replacing the default `<h2>` rendering. If none, state "no theming slots."

**Required vs. optional tokens** — which CSS variables must be defined for correct rendering (required) vs. which have acceptable built-in fallbacks (optional).

**Design token approximation as Known Deviation:** When a primitive uses Tailwind utility classes (e.g., `text-green-400`) to approximate hand-tuned CSS values from a module-private implementation (e.g., `.vault-dot --tier` exact color), document the approximation as a Named Deviation in §13.2.12 and specify the promotion path: either (a) extract the exact values into shared design tokens and reference them in the primitive, or (b) accept the approximation as platform-level color discipline and close the deviation by policy. Do not silently ship approximations — they accumulate into inconsistency. `StatusDot` (PR #80) Known Deviation #1 documents this tradeoff for tier-role colors. The deviation closes when the Persona vault-dot colors are extracted into shared design tokens.

**Isolation guarantee** — confirmation that the primitive uses `!important` only where documented as a deliberate isolation requirement, with an explanatory comment in the CSS at the point of use. Undocumented `!important` overrides that would prevent consumer theming from applying are forbidden.

---

### 13.2.10 Context-Purity Audit

Explicitly names every STO-internal context the primitive does NOT depend on. This section is required because STO has implicit contexts (auth, user, module registration, `ShellContext`, `ChatBotContext`, etc.) that primitives must never read. The audit is a table, not prose.

**Required content:**

**Context dependency table:**

| Context | Dependency | Required input instead |
|---|---|---|
| `AuthContext` | ❌ None | N/A |
| `UserContext` | ❌ None | N/A |
| `ShellContext` | ❌ None | N/A |
| `ChatBotContext` | ❌ None | N/A |
| Module entitlement gate | ❌ None | `tier` prop if tier-dependent behavior needed (§13.2.11.3) |
| STO analytics / Backboard | ❌ None | `onUsage` callback for caller-supplied telemetry (§13.2.11.1) |

**No implicit side effects declaration** — explicit confirmation that the primitive does not fire analytics events, write to localStorage under STO-private keys, or read from environment variables. If it does write to localStorage (e.g., a dismiss state), the key must be prefixed with a caller-supplied namespace passed as an input, never with a hardcoded STO key.

**Zero-imports discipline target:** The canonical context-purity posture for a shared primitive is zero STO imports — no `react`, no `react-router-dom`, no `lib/` utilities, no STO context imports. A primitive with zero imports is a pure render component driven entirely by props. It can be dropped into any React app without STO infrastructure assumptions — the SDK pitch in its clearest form. `ModuleFields.jsx` (PR #77) is the reference proof point. Where applicable, design toward zero-imports; document any necessary imports as explicit deviations from the zero-imports target.

When a React import IS necessary, classify it by category. Three accepted exception types established across Phase 2:
1. **SDK-coupling exceptions** — `useModuleTabs.jsx` imports `ShellContext` + `react-router-dom` (`useNavigate`, `useParams`) because the hook's routing integration is the primitive's purpose; both are declared allowed dependencies in its contract.
2. **A11y-necessity exceptions** — `FloatingInput.jsx` + `FloatingTextarea.jsx` (PR #82) import React's `useId` built-in to auto-generate stable IDs for `htmlFor`/`id` label-input linkage. Without `useId`, the burden shifts to every call site; consumers would need to manage IDs manually and could easily omit the association. `useId` is a React built-in (not STO-specific coupling), closes a real a11y gap (SR label/input association), and cannot be deferred to a prop without unacceptable consumer ergonomic cost. This is the reference proof point for the a11y-necessity exception class.
3. **Chrome-namespace exceptions** — primitives in `uma/chrome/` are shell-coupled navigation/structural components by design. `BackButton.jsx` (PR #84) imports `react-router-dom` for `useNavigate` — permitted because navigation integration is the primitive's purpose and `chrome/` is explicitly scoped to shell-structural work. Context-purity audit still required; the chrome namespace widens what counts as "zero STO coupling" to include `react-router-dom` navigation hooks.

**Verification note:** this section is a verification discipline artifact, not a design claim. Write it after reading the reference implementation, not before (the UMA 3.0 Contract Meta-Framework §9).

---

### 13.2.11 Monetization-Supporting Contracts

Defines how the primitive participates in the UMA 3.0 monetization model through billing, telemetry, and tier-gating hooks.

Not every primitive requires all three sub-sections. The contract document explicitly states which sub-sections apply and which are declared inapplicable with reasoning. Leaving any sub-section blank without declaration is not permitted.

#### 13.2.11.1 Usage Telemetry Hooks

For primitives that emit events usable for commission calculation:

**`onUsage` callback input** — optional event handler the primitive invokes when a usage event occurs. The callback receives a `umaModule_telemetry_usageEvent_*` payload (§7.6 AIX keys). The host platform or module supplies this callback; the primitive fires it; the primitive does not implement billing logic.

**Event types emitted** — controlled vocabulary of event type strings for this primitive. Example for `confirmModal`:

| Event type | When fired |
|---|---|
| `confirm_activated` | User activates confirm button |
| `cancel_activated` | User activates cancel button |
| `dismissed` | User dismisses via Escape or backdrop click |

**Inapplicable declaration:** If the primitive does not emit billable events (e.g., `Skeleton`, `StatusDot`, `Accordion`), state: "§13.2.11.1 Usage Telemetry: not applicable — this is a UI chrome primitive with no commerce-adjacent interaction surface."

#### 13.2.11.2 Billing API Integration Points

For primitives that surface transaction events back to the host platform:

**`onTransaction` callback input** — optional event handler for primitives that complete a billable action (e.g., a confirmation modal that gates a paid operation). The callback receives a `umaModule_billing_transactionEvent_*` payload (§7.6 AIX keys).

**Transaction semantics** — description of when a transaction event fires and what it represents.

**Inapplicable declaration:** Most primitives are not transaction-adjacent. State: "§13.2.11.2 Billing API: not applicable — this primitive does not gate or complete billable operations."

**v1 forward-declaration pattern:** The STO billing architecture is not yet fully specified. v1 contracts that include §13.2.11.2 forward-declare the `onTransaction` contract shape and specify a no-op host implementation. When the billing architecture is specified, the no-op is replaced with the real implementation — the primitive contract does not change. No v1 primitive contract is blocked waiting for billing architecture. Forward-declare with no-op; ship.

```js
// No-op host implementation for v1 — replaced when billing architecture is specified
const onTransaction = (payload) => { /* billing not yet wired */ };
<ConfirmModal onTransaction={onTransaction} ... />
```

#### 13.2.11.3 Tier-Gated Capability Discovery

For primitives that expose capability variations by subscription tier:

**`tier` input** — optional input carrying the host module's subscription tier string (`"free"` | `"pro"` | `"enterprise"`). When present, the primitive may expose enhanced capabilities or restrict features. When absent, the primitive renders in its most permissive state — this is the correct default for development use.

**Capability map:**

| Feature | `free` | `pro` | `enterprise` |
|---|---|---|---|
| Standard confirm variant | ✅ | ✅ | ✅ |
| Destructive confirm variant (`isDangerous`) | ✅ | ✅ | ✅ |
| Custom icon slot | ❌ | ✅ | ✅ |
| Confirm-delay input (anti-fat-finger) | ❌ | ❌ | ✅ |

> **Tier-gating framing note:** Safety-signaling props (`isDangerous`, `variant="destructive"`) must never be locked behind a paid tier — restricting a module's ability to warn users of a dangerous action is backwards safety framing. Tier-gate branding and convenience features instead.

**Inapplicable declaration:** If the primitive has no tier-dependent capabilities: "§13.2.11.3 Tier-Gated Capability: not applicable — all capabilities are available at all tiers."

---

### 13.2.12 Reference Implementation

Names and describes the canonical React implementation.

**Required content:**

- **Source location** — exact file path (e.g., `src/components/uma/primitives/ConfirmModal.jsx`)
- **Implementation version** — semver string locked to the AIX contract version it implements (e.g., `1.0.0`)
- **Export shape** — named export, default export, or hook function signature
- **Test coverage** — pointer to test files; minimum requirements: unit tests covering §13.2.7 state transitions + integration tests covering §13.2.8 keyboard contract + context-purity verification (no STO context imports)
- **Known deviations** — explicit list of any contract commitments the reference implementation does not yet honor, with tracking issue references. A conforming implementation has no entries in this table. An implementation with open deviations is labeled `experimental` in the registry (§13.1).

> **Experimental-is-honest posture:** Every Phase 2 primitive has shipped `experimental` with documented deviations (accessibility gaps, missing automated tests, open behaviors). This is the correct SDK posture. Empirical migration coverage from same-PR consumer refactors substitutes for automated tests at v1.0.0 confidence. Shipping `experimental` honestly — with a documented promotion-to-stable gate — is more trustworthy to 3rd-party developers than claiming `stable` prematurely. The Known Deviations table is not a liability; it is the contract that says "here is exactly what we haven't finished."

---

### 13.2.13 Conformance Test Suite

A framework-agnostic test specification that any implementation (React, Vue, Svelte, web component) can run against itself to verify AIX contract compliance.

**Required content:**

- **Test specification document** — a JSON file at `docs/aix/conformance/{primitiveId}.test.json` following the format in the UMA 3.0 Contract Meta-Framework §7
- **Minimum coverage** — at least five `required` scenarios covering: ARIA role and attribute correctness, keyboard interaction contract (Escape key minimum), focus management (initial focus placement), context purity (no STO context imports), theming surface (CSS variable consumption)
- **Pass/fail semantics** — a conforming implementation passes all `required` scenarios. Failing `recommended` scenarios are tracked but do not block AIX compliance certification.

**Test runner:** JSON test specification files ship with their extraction PRs as part of Phase 2. A test runner that executes these specifications automatically is a separate future deliverable and is not required before the first extraction PR ships.

---

### 13.2.14 Versioning and Breaking Change Policy

States the primitive's current version and per-primitive definition of a breaking change.

**Required content:**

- **Contract version** — semver string (initial: `1.0.0`)
- **Locked AIX version** — which AIX standard version this contract targets (`AIX v2.4`)
- **Breaking change definition** — primitive-specific list of what constitutes a breaking change. Generic lists are insufficient; each entry must name a specific input or behavior. Example for `confirmModal`:
  - Removing `onConfirm` as a required input
  - Adding a new required input
  - Changing Escape key behavior from "invokes `onCancel`" to "does nothing"
  - Removing `aria-modal="true"` from the root element
- **Deprecation procedure** — how deprecated inputs/behaviors are flagged (the UMA 3.0 Contract Meta-Framework §8.2)
- **Migration path requirements** — deprecated inputs or behaviors must include exact migration instructions

---

## 13.3 AIX Key Hierarchy for UMA Primitives

Ratified in full at §7.6 of this document. Summarized here for chapter-local reference: `umaModule_primitive_{primitiveId}_*` (contract metadata: `contractVersion`, `aixVersion`, `status`), `umaModule_telemetry_usageEvent_*` (usage telemetry payload, fired via a primitive's `onUsage` callback), `umaModule_billing_transactionEvent_*` (billing transaction payload, fired via `onTransaction`), `umaModule_tier_capability_{primitiveId}_{capabilityId}` (tier-gated capability discovery). All follow §2.2 hierarchy and pass the six §2.6 validation tests.

---

# 14. Security & Governance

## 14.1 Security

AIX does not define authentication — this is provider-specific. AIX messages must never contain API keys, passwords, or personal access tokens.

```json
"safety": {
  "flagged": true,
  "categories": { "violence": false, "sexual": false, "hate": false,
                  "selfHarm": false, "dangerous": true },
  "actionTaken": "content_blocked"
}
```

## 14.2 Versioning

| Version Level | Trigger | Example |
| --- | --- | --- |
| Major (X.0) | Breaking changes to envelope, required fields, or task type removal | 1.x → 2.0 |
| Minor (2.X) | New task types, new optional fields, additive features | 2.1 → 2.2 |
| Patch (2.0.X) | Clarifications, documentation fixes, no schema changes | 2.0 → 2.0.1 |

## 14.3 Amendment Process

To add a new key: (1) propose with full four-part hierarchy and plain-English description; (2) run all six validation tests from §2.6; (3) check for conflicts with §7; (4) add to candidate keys in next minor version draft; (5) upon ratification, move to canonical reference.

> Retroactive changes to ratified keys are prohibited. Deprecation is handled by adding a replacement key and marking the old key [DEPRECATED] in the registry — not by renaming it.

## 14.4 Governance — Aspirational External-Standard Model

| Body | Responsibility |
| --- | --- |
| Technical Steering Committee | Specification changes and ratification |
| Provider Council | Implementation feedback and provider registry |
| Community Contributors | Pull requests, issues, and proposals |

This is the external-standard governance model this project aspires to as AIX opens to outside implementers via `aix-standard.org`. It has not yet convened — no version through 2.4 has been ratified by a Technical Steering Committee or Provider Council.



---

# 15. Appendix

## A. Provider Registry

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

## B. Language Codes (BCP-47)

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

## C. Media Type Mappings

| Format | MIME Type |
| --- | --- |
| mp3 | audio/mpeg |
| wav | audio/wav |
| ogg | audio/ogg |
| flac | audio/flac |
| mp4 | video/mp4 |
| webm | video/webm |
| png | image/png |
| jpeg | image/jpeg |
| webp | image/webp |

## D. References

1. Model Context Protocol (MCP) — https://github.com/modelcontextprotocol
2. Eden AI APIs — https://docs.edenai.co
3. Schema.org — https://schema.org
4. JSON Schema — https://json-schema.org
5. BCP-47 Language Tags — https://www.rfc-editor.org/info/bcp47
6. CrossARM Data Interchange Standard (2014) — Ubertrends LLC (internal)

## E. Implementation Notes (Non-Normative)

> This appendix is non-normative. Nothing here adds requirements to the AIX standard. Material here documents staging patterns, migration sequences, and operational guidance that implementers have found useful. When the spec body and this appendix appear to disagree, the spec body governs.

### E.1 Section 3.5 Media Collections — Single-Primary Subset Pattern

AIX-compliant implementations MAY ship a single-primary subset of `media[]` before supporting the full array. The subset stores one Medium object (the primary one) and applies Resolution Rule Step 2 directly. The full array is purely additive: a later upgrade adds the array shape without breaking the subset implementation.

The Solo Trillion reference implementation uses this staging pattern — single-primary subset at beta launch (May 2026), full array post-beta.

### E.2 Module Manifest — Staged Rollout

Module manifests MAY omit optional fields (`billing`, `permissionScopes`) on initial implementation. Required fields for a v1 manifest: `aixVersion`, `moduleId`, `moduleType`, `version`. Add optional fields in subsequent releases.

### E.3 writingAgent_format_* Migration Guide

Implementations using `writingAgent_documentType_*` keys should migrate as follows:
- `writingAgent_documentType_article_*` → `writingAgent_format_default_*`
- `writingAgent_documentType_fieldGuide_*` → `writingAgent_format_default_*`
- `writingAgent_documentType_meetingBrief_*` → `writingAgent_format_meetingBrief_*`
- All other `documentType` keys → corresponding `format` key (name maps directly)

Both key sets resolve identically during the transition. Remove deprecated keys after confirming all consumers have migrated.

### E.4 Relationship to Block-First Architecture Refactor

Block-first's `format_registry` and `block_schemas`, per §12.2's own namespace table, become canonical registries under chapter 12 once implemented — Stage 1 implementation should be authored with chapter 12 conformance in mind from the start.

Persona-Editorial is in Stage 1 scope (§12.2.2) — authored on the separate Stage-1 implementation track, tracked here as a real cross-chapter dependency. The Rosie iPad conducting experience is compressed into Stage 1 — voice input as a first-class input mode, push-on-completion, background dispatch survival, and a streamed-artifact review mode, per §12.10's contract.

### E.5 Relationship to Persona — Incubation-Profile Reference

Persona is an incubation-profile reference only (§12.12), not "the reference implementation" unqualified. Persona correctly declares the SHAPE of one axis (tier) but does not demonstrate handoff, audit, lifetime-expiry, or denial paths end-to-end anywhere in a running system. Calling it "the reference implementation" unqualified would overstate what it actually proves.

---

# 16. Changelog

> **Note on section references within entries below.** Version 2.4 restructured this document's own chapter order — Module Manifest Standard was promoted from a subsection to its own chapter, and the Canonical Registry Contract and UMA 3.0 Primitive Contract Structure were inserted as new chapters 12–13. Entries for Version 2.3 and earlier cite section numbers as they stood in that version's own numbering at the time of ratification, per changelog-integrity discipline — not this document's current chapter numbers. Several of those old citations (`§5.1`, `§5.2`, `§6.2`, `§7.3`, `§8.1`) now also exist as headings in the current document but name different content there. Consult the Version 2.4 entry's own chapter-by-chapter list, immediately below, for what actually changed and where it lives today.

## Version 2.4 — August 2026

Minor version, structural: converges what had become three separately-versioned documents — AIX Standard v2.2, the AIX 2.3 Canonical Registry Contract, and the UMA 3.0 Contract Meta-Framework's key families — into this single document, one standard with one version. Retires the two-coexisting-standards framing that produced version-literal divergence across the codebase.

| Area | Change |
| --- | --- |
| Document structure (STRUCTURAL) | `AIX-Standard-v2.2.md` and `aix-2.3-canonical-registry-contract.md` retired as live files (redirect stubs in place); physically inlined here rather than cross-referenced — a shared version number across physically separate files was structurally what produced the divergence this version closes. |
| Chapter 2 (UPDATED 2026-10-20) | `id` added to §2.4's permitted list as a registered abbreviation of "identifier" (a permitted exception under §2.4's No Abbreviations rule, not an acronym). Ratified keys have carried `Id` segments since v2.1 (`mediumId` §3.5, `requestId` §8.3, the manifest's `moduleId` §5.2), and §7.6's `{capabilityId}` and its telemetry and billing keys ending in `moduleId`, `primitiveId` or `sessionId` carry them too; this records that existing practice and changes no existing key. It newly permits keys such as `taxId`, which §2.6 test 3 (Abbreviation) would otherwise reject. No version bump, although §14.2's table would make this a Patch, or a Minor because it newly permits a key: additions to the permitted list are made in place as dated amendments, as at the 2026-05-12 `rss` addition and the 2026-09-12 chapter 13 row below, and this row follows that practice. |
| Chapter 2 (UPDATED 2026-10-20, §2.3) | §2.3 notes that `taxId` (§3.4) is the Schema.org property `taxID` in this standard's camelCase form, with `id` registered at §2.4. No key changes. |
| Chapter 3 (UPDATED 2026-10-20) | Chapter 3 states that its tables are the normative registry and that an implementation's own field-key list conforms to this chapter. §3.4 Financial & Legal ratifies `bankName` and `businessReferences`, deprecating `bank` and `references` in their favor (both remain in §3.4 marked `[DEPRECATED]`, with §7.7 entries, per §7.8 and §14.3), and ratifies `taxId` (Schema.org `taxID`), each after the §2.6 tests that apply. A dated note records two implementation matters without ratifying them: the non-conformant `organizationName`, which duplicates `legalName`, and three bank-detail keys pending ratification (`bankAddress`, `routingNumber`, `accountNumber`). No version bump: Michael ruled on 2026-10-20 to publish these as 2.4 with dated in-place amendments, a deliberate exception to the minor-version path §14.2 and §14.3 set for new keys. |
| Corrections (FIXED 2026-10-20) | §13.2.1 said "five of the ten" contract sections are universal and named four elements, one of them the header, which is not one of the ten; it now says four elements are universal: the header, §13.2.12, §13.2.13 and §13.2.14. §12.4 dropped a persona field count that had gone stale. Examples were made fictional: §3.5's media example now uses invented names, tagline and example.com URLs in place of real ones; §3.6's LinkedIn and X format examples use example.com hosts, while its other format examples keep generic handles on the platforms' own hosts; §9.1's author example uses example.com hosts for its LinkedIn, X, YouTube and website URLs. |
| Chapter 5 (UPDATED) | `permissionScopes`-vs-canonical-registry-grant relationship stated explicitly for the first time: composable, not merged — a flat string list vs. a full grant tuple, with both registration-time and dispatch-time failure modes specified (§5.2.1). |
| Chapter 7 (UPDATED) | `umaModule_usage_event` marked `[DEPRECATED]` in favor of `umaModule_telemetry_usageEvent_*`; `umaModule_billing_ref` unchanged — the asymmetry is deliberate. New §7.6 registers UMA 3.0's primitive/telemetry/billing/tier-gated-capability key families. New §7.8 Deprecation Policy — AIX's first stated deprecation policy, sized to this one deprecation. |
| Chapter 12 (NEW) | Canonical Registry Contract — physically inlines what was ratified separately as AIX 2.3 (2026-08-02). Its own "Relationship to AIX 2.2" section is retired entirely; §2's "Grounding" paragraph (which asserted a merged-vocabulary model chapter 5 explicitly rejects) is rewritten to match chapter 5's composable relationship. |
| Chapter 13 (NEW) | UMA 3.0 Primitive Contract Structure — registers the full, now-current meta-framework: the sixteen-primitive registry, the ten-section contract structure (including the canonical skeleton, three-tier section split, and Experimental → Stable graduation path), and the four UMA key families. Supersedes the narrower "register only the key families" recommendation an earlier authoring pass made. |
| Chapter 13 (UPDATED 2026-09-12) | Chapter 13 now carries the UMA 3.0 Contract Meta-Framework's §3 in full as §13.2.1–§13.2.14 (four ratification preambles, then the ten contract sections, with §13.2.11's three monetization subsections) and the key summary moves from §13.4 to §13.3; the former §13.5 is absorbed by §13.2.11. The count is stated in both units, because a published copy had conflated them: fourteen numbered subsections in this chapter, ten sections a contract must contain. Cross-references at §7.6 (`§13.5.1`/`§13.5.2` → `§13.2.11.1`/`§13.2.11.2`) and §13.1 (`§13.3` → `§13.2.3`) follow. |
| Chapter 15 (UPDATED) | 2.3's own §12 (Relationship to block-first architecture refactor) and §13 (Relationship to Persona) folded in as new Appendix E.4/E.5, non-normative implementation notes — the same treatment 2.2's own Appendix E already gives comparable material. 2.3's own §14 (design-question answers), §15 (verification log), and §16 (authoring report) are NOT carried forward — they are the ratification-process record of that document's own authoring pass. Not lost: `aix-2.3-canonical-registry-contract.md` is retained as a historical file (via git history), and this changelog entry plus the Version 2.3 entry below already summarize what was ratified. |
| Chapter 16 (UPDATED) | This entry. The Version 2.3 entry below is kept verbatim per changelog-integrity discipline — a changelog is a historical record and rewriting it erases what was actually ratified — with a bracketed note marking its two-coexisting-standards framing as retired. |
| Chapter 5/7 (UPDATED) | The publishing module's product name is retired — it is now Ubertrends' Newsletter Studio, the same way "Orchestra" was dropped from Solo Trillion's own name. Chapter 5's manifest example and chapter 7.5's section title/prose updated; §3.5's two media-collection examples update `mediumBrand` to `"Ubertrends"`. **The ratified keys themselves are now affected — see the deprecation row below; this row's own original claim that they were not has been superseded, not silently left standing.** The Version 2.2 entry below is kept verbatim per the same changelog-integrity discipline, with a bracketed note marking the retirement. |
| Stale identifiers (FIXED) | §2.1's slug example, §5.1's worked example, §5.2's manifest example, and §5.2's slug-to-camelCase note all cited the retired publishing module's own dead `app_id` — the `module_registry` row was renamed to `newsletter-studio` in June 2026 (`20260612_rename_app_id_futuresight_to_newsletter_studio.sql`). All four updated to `newsletter-studio`. |
| `publishingModule_emailStudio_*` deprecated, `publishingModule_newsletterStudio_*` ratified | The derivation gap the row above flagged is closed, not merely narrowed: §5.1/§5.2's own slug-to-camelCase rule now derives correctly (`newsletter-studio` → `newsletterStudio`), because the six `publishingModule_emailStudio_*` keys — which never derived correctly under either app_id — are deprecated in favor of six `publishingModule_newsletterStudio_*` keys that do. §7.5 carries the six new ratified rows; §7.7 carries the six deprecated rows, following the same shape as the existing `writingAgent_documentType_*` deprecations (fully superseded, not duplicated in their origin table). A direct repository grep confirmed zero code references to the deprecated keys before this deprecation was ratified — documentation and one SQL seed only. **`publishingModule_emailStudio_htmlExport`, live in the newsletter-studio manifest but never ratified in any version of this standard, is deliberately not addressed by this deprecation** — addressed separately, below. |
| `publishingModule_newsletterStudio_htmlExport` ratified | Seventh key in §7.5, closing the gap the row above named. Not a deprecation — `htmlExport` was never ratified under any prior name in any version of this standard, so §7.7 carries no corresponding row; a plain new ratification, per §7.5/§7.7's own explicit reasoning. Its description is written from the code that actually produces the value (`NewsletterActionBar.jsx`'s `exportHtml`, a client-side-only blob download), not from either live document that previously described it — those two disagreed with each other (one said no ESP send, one said delivery via ESP); the code settles it in the no-send direction. The maintained module-registry description was corrected to match on 2026-08-15. |
| Broken cross-references (FIXED) | §13's own telemetry/billing key citations pointed at `§13.7.1`/`§13.7.2`, which did not exist (chapter 13 has no §13.6 or §13.7) — corrected to `§13.5.1`/`§13.5.2`, the actual Usage Telemetry Hooks / Billing API Integration Points sub-contracts, which were themselves promoted from bold paragraph leads to real headings so they resolve as addressable sections like every other numbered subsection in this document. §12.1.1's incubation-profile citation read `(§12.12, §12.12)` — a duplicated self-citation from flattening AIX 2.3's own two-target citation (its old §10 and §13) during convergence; corrected to `(§12.12, Appendix E.5)`, its actual two targets. |
| Changelog numbering ambiguity (FIXED) | Entries for Version 2.3 and earlier cite section numbers from that version's own numbering at ratification time — several of those numbers (`§5.1`, `§5.2`, `§6.2`, `§7.3`, `§8.1`) now also exist as headings in this document but name different content, since Version 2.4 inserted and promoted chapters below chapter 4. A note added at the top of this chapter states this explicitly rather than leaving old citations to silently resolve against the wrong current section; no historical entry's own text was altered. |
| External reference, resolved | §12.1.1's public-validator citation, `solotrillion.io`, is correct — verified live (2026-08-14): it serves "Solo Trillion · Validate AI markdown before you ship," exactly the validator §12.1.1 describes, while `solotrillion.ai` serves the app; both return HTTP 200, neither redirects. The split is deliberate: `.ai` for the app and Executive Briefings, `.io` for the standard and developer ecosystem. `.com` was sunsetted May 2026 and now redirects to `.ai`. |
| Internal contradiction (FIXED) | §7.5 labeled the camelCase key-hierarchy segments (`newsletterStudio`, `emailStudio`) as `moduleId: X`, contradicting §5.2's own rule that the manifest's `moduleId` field is dash-case and only the AIX hierarchy key's module-ID segment is camelCase. The live `module_registry` row was and is conformant (dash-case, re-verified live) — the prose was stale, not the data. Corrected to `module ID segment X`, matching §5.2's own terminology; the camelCase values themselves were unchanged, since they were correct in a key-hierarchy context. |

## Version 2.3 — August 2026

Minor version: additive. Ratifies the Canonical Registry Contract (ratified 2026-08-02) — a sibling addendum defining how canonical registries expose scoped, capability-gated, auditable reads to agents. Does not modify this document's own naming/envelope rules (§1-§12 unchanged); fulfills the forward-pointer this document already carried (§4.8: "These keys are addenda to v2.2; they will be formally registered in a future minor version"). **[This framing was retired at 2.4 — this document is now one standard, physically inlined; see the Version 2.4 entry, above. Recorded verbatim per changelog-integrity discipline, not rewritten.]**

| Area | Change |
| --- | --- |
| New document (NEW) | Canonical Registry Contract — full text in its own file, cataloged here per this document's own changelog convention. |
| Capability axis (NEW) | Orthogonal capability strings (e.g. `records:read`), modeled after the Airtable Personal Access Token precedent. |
| Tier axis (NEW) | Public / Professional / Sensitive, per-field classification — Persona's existing tier system is the closest reference implementation. |
| Lifetime axis (NEW) | Grant expiry and revocation — persistent, session-scoped, and one-shot grants. |
| Resource-scope axis (NEW) | Grants scoped to a specific resource, not just a namespace-wide capability. |
| Obligations axis (NEW) | Seven obligation types constraining post-read use of a value, across three enforcement modalities (pre-read gate; mechanical system-controlled downstream-use; advisory-only once a value enters an LLM agent's free-text context). |
| Grant tuple (NEW) | The formal `{namespace, capability, tier, resource_scope, lifetime, obligations}` shape a grant carries. |
| Audit contract (NEW) | What a conformant registry must log per read/grant; distinguishes partial audit primitives already live in this codebase (`conversations.persona_fields_accessed`) from full conformance. |
| Cross-agent handoff (NEW) | Closes a dispatch-payload security gap (the prior zero-scope JSON payload between agents) — grants and obligations now travel with a value across an agent handoff. |
| Conformance (NEW) | Two-tier: incubation-profile vs. full-conformance. Persona is incubation-profile only as of ratification. |

## Version 2.2 — May 2026

Minor version: additive. Formalizes module manifest hierarchy, deprecates `documentType` object segment in favor of `format`, adds slug/URL casing exception, adds Published Works field registry, and ratifies UMA Platform and FutureSight Studio canonical keys.

| Area | Change |
| --- | --- |
| §2.1 (UPDATED) | Slug/URL casing exception added: dash-case for slugs/URLs, snake_case for database columns (was already noted), camelCase for field keys. Schema version string format (`{type}-v{N}`) documented as exempt from camelCase rule. |
| §2.2 (UPDATED) | Hierarchy examples updated to use `writingAgent_format_*` pattern. |
| §2.4 (UPDATED) | `uma` added to permitted acronym list. |
| §3.10 (NEW) | Published Works field registry — 10 fields for book chapters and sidebars: `bookTitle`, `bookKey`, `chapterKey`, `chapterTitle`, `chapterNumber`, `chapterContent`, `chapterSummary`, `chapterSubsections`, `contentType`, `parentChapterKey`. |
| §4 (UPDATED) | `rosie` added as a registered component key alongside `conversationalAgent`. Use `conversationalAgent` for generic interchange; use `rosie` for Rosie-specific configuration keys. |
| §4.8 (NEW) | Module Manifest Standard — formalizes `{moduleType}Module_{moduleId}_{property}` hierarchy pattern. Module manifest schema defined. Slug-to-camelCase rule documented. Module type segments ratified: `publishingModule`, `researchModule`, `writingModule`, `personaModule`. |
| §5.1 (UPDATED) | `documentType` deprecated as object type segment. `format` is the canonical replacement. |
| §5.2 (UPDATED) | Format identifiers added under `writingAgent_format`: `default`, `meetingBrief`, `executiveBrief`, `competitiveAnalysis`, `newsletter`, `linkedinPost`. Migration note from `article`/`fieldGuide`. |
| §6.2 (UPDATED) | `writingAgent_format_*` canonical keys added (12 new keys). All `writingAgent_documentType_*` keys deprecated with direct replacement mapping. |
| §6.4 (NEW) | UMA Platform canonical keys: `umaModule_registry_manifest`, `umaModule_entitlement_status`, `umaModule_billing_ref`, `umaModule_usage_event`. |
| §6.5 (NEW) | Publishing Module — FutureSight Studio canonical keys (6 keys under `publishingModule_emailStudio_*`). **[The product name recorded here was retired at 2.4 — see the Version 2.4 entry, above. The keys themselves are deprecated as of 2.4, in favor of `publishingModule_newsletterStudio_*` — see chapter 7.7.]** |
| §6.6 (NEW) | Deprecated key registry with full replacement mapping table. |
| §7.3 (UPDATED) | Metadata object example corrected to camelCase throughout: `total_ms` → `totalMs`, `processing_ms` → `processingMs`, `network_ms` → `networkMs`, `model_version` → `modelVersion`, `request_id` → `requestId`, `input_stored` → `inputStored`, `output_stored` → `outputStored`, `retention_days` → `retentionDays`. |
| §8.1 (UPDATED) | `contentBrief.format` examples updated to use `"default"` instead of `"article"`. |
| Appendix E.2 (NEW) | Module manifest staged rollout guidance. |
| Appendix E.3 (NEW) | `writingAgent_format_*` migration guide. |

## Version 2.1 — April 2026

Minor version: additive. Introduces Media Collections standard, Agent Behavioral Modes standard, and Soul Document specification.

| Area | Change |
| --- | --- |
| §3.5 (NEW) | Media Collections — `media[]` typed collection, `mediumType` vocabulary, four-step substitution rule, backwards compatibility |
| §4.7 (NEW) | Agent Behavioral Modes — `behavioralMode` field, standard mode vocabulary (observer/orchestrator/briefing/off), mode transitions |
| §4.7.6 (NEW) | Soul Documents — persistent agent identity file standard |
| §8.1 | `media` array added to `content_authoring` author object |
| Appendix E.1 (NEW) | Single-Primary Subset Pattern — staging guidance |

## Version 2.0 — March 2026

Major version: merges AIX Interchange Format Specification and AIX Naming Convention Standard into a single authoritative document.

| Area | Change |
| --- | --- |
| Document structure | Merged two specs (v1.1 interchange + naming convention) into AIX v2.0 |
| §3 (NEW) | Persona Field Registry — 40+ fields with Schema.org base or AIX extension designation |
| §4 (NEW) | Component & Agent Registry formalized |
| §5 (NEW) | Reserved Segment Vocabulary — object types, identifiers, property suffixes |
| Author fields | givenName/familyName replace first_name/last_name (Schema.org canonical) |
| Social media fields | {platform}Url pattern; `Url` suffix required |
| Deprecated | writingAgent_template_* \| researchAgent_config_passOne/Two/Three_* |

## Version 1.1 — February 2026

| Area | Change |
| --- | --- |
| content_authoring | New task type added |
| author object | first_name, last_name, display_name, title, bio, social_media_profiles |
| voice fields | voice_brand, voice_guidelines, voice_guardrails |
| research_packet | Research Agent → Writing Agent chaining input |
| guardrail_flags | Human review signaling in output |

## Version 1.0 — December 2025

Initial specification. Core task types: text, voice, image, video, code. Metadata schema with cost/latency tracking. Workflow chaining support. Provider comparison format.

---

*Apache 2.0 Licensed · aix-standard.org · github.com/aix-standard/aix-spec · aix@ubertrends.com · Copyright 2026 Michael Tchong, Ubertrends LLC*
