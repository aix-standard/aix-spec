**AIX**

**AI Interchange Standard**

*Version 2.1 Draft  ·  April 2026*

| Author: Michael Tchong, Ubertrends LLC License: Apache 2.0 (Open Standard) *AIX is an open standard for naming, exchanging data, and establishing interoperability between generative AI tools, agents, and applications. This document merges the AIX Interchange Format Specification (v1.1) and the AIX Naming Convention Standard into a single authoritative reference.* Heritage: CrossARM (2014) · Ubertrends LLC · Solo Trillion |
| --- |

| Section | Contents |
| --- | --- |
| 1\. Introduction | Problem statement, CrossARM lineage, Schema.org relationship |
| 2\. Core Naming Principles | camelCase, hierarchy, no abbreviations, validation tests |
| 3\. Persona Field Registry | Schema.org base fields \+ AIX extensions for all persona fields |
| 3.5 Media Collections | media\[\] typed collection, mediumType vocabulary, template substitution resolution |
| 4\. Component & Agent Registry | All registered AIX components |
| 4.7 Agent Behavioral Modes | behavioralMode field, standard mode vocabulary, Soul Documents |
| 5\. Reserved Segment Vocabulary | Object types, identifiers, property suffixes |
| 6\. Canonical Key Reference | All ratified AIX keys |
| 7\. AIX Envelope & Task Types | Interchange format, task schema, metadata |
| 8\. Task-Specific Schemas | Voice, image, text, video, code, content_authoring |
| 9\. Workflows & Chaining | Multi-step pipeline schema |
| 10\. Provider Comparison Format | Side-by-side output schema |
| 11\. Security & Governance | Auth, safety, versioning, amendment process |
| 12\. Appendix | Provider list, language codes, media types |
| 13\. Changelog | Version history |

# **1\\. Introduction**

## **1.1 The Problem**

The generative AI revolution has produced hundreds of specialized tools — each with its own API format, authentication method, and response structure. This fragmentation creates developer burden through custom integration code for each provider, vendor lock-in when switching providers requires rewriting applications, workflow friction when chaining tools requires format conversion, and no standard way to compare outputs across providers.

A parallel problem exists inside AI agents themselves: without a naming standard, every team invents its own key vocabulary for fields, configuration properties, and persona data. Keys proliferate without governance — some snake_case, some camelCase, some abbreviated, some redundant — until the system becomes unmaintainable and incompatible with external implementers.

## **1.2 The Solution**

AIX addresses both problems with a single specification. It defines a standard envelope for AI task requests and responses so any AIX-compliant tool can accept standardized inputs, return standardized outputs, be swapped with any other tool in the same category, and chain seamlessly with other AIX tools in workflows. It also defines a canonical naming convention so every field, key, and identifier in the AIX ecosystem is unambiguous, hierarchical, and implementable by developers with no internal context.

## **1.3 Heritage: CrossARM (2014)**

AIX follows the pattern established by CrossARM, a data interchange standard developed by Ubertrends LLC in 2014 for cross-platform marketing data exchange. CrossARM's governing principle was: use Schema.org canonical names where they exist, and invent proprietary extensions for what Schema.org does not cover.

| // CrossARM 2014 — Schema.org base vocabulary |
| --- |
| givenName, familyName, worksFor, address, email, telephone  (Person) |
| brand, legalName, duns, isicV4                              (Organization) |
| --- |
| // CrossARM 2014 — Proprietary extensions (Schema.org had no equivalent) |
| facebookName, twitterName, twitterFollowers, engagementRate |
| uniqueVisits, timeOnSite, bounceRate, influencerRating |

AIX v2.0 formalizes this pattern. Schema.org canonical names are used wherever applicable. Fields that Schema.org does not cover are defined as AIX extensions, noted as such in the registry.

## **1.4 Relationship to Existing Standards**

| Standard | Focus | AIX Relationship |
| --- | --- | --- |
| MCP (Model Context Protocol) | Connecting AI to data sources | Complementary — MCP handles inputs, AIX handles outputs |
| Schema.org | Structured web data vocabulary | Foundation — AIX uses Schema.org canonical names as base layer |
| Eden AI Schemas | Traditional AI APIs (OCR, NLP) | Predecessor — AIX extends into generative AI task types |
| JSON Schema | Data validation | Implementation — AIX uses JSON Schema for field validation |
| CrossARM (2014) | Cross-platform marketing data | Heritage — established the Schema.org base \+ extension pattern |

# **2\\. Core Naming Principles**

These principles govern every field name, key, and identifier in the AIX specification and in any AIX-compliant implementation.

## **2.1 camelCase Throughout**

All field names and identifiers use camelCase. No snake_case, no kebab-case, no PascalCase for field names. This aligns with JSON, JavaScript, and modern API conventions.

| // ✓ Correct |
| --- |
| givenName  familyName  jobTitle  telephoneMobile  linkedinUrl |
| --- |
| // ✗ Incorrect |
| given_name    // snake_case |
| given-name    // kebab-case |
| GivenName     // PascalCase |

| ℹ  Database exception: PostgreSQL column names use snake_case by convention.    This is layer-appropriate only. API responses, JSON exports, and all    specification field names use camelCase. ORM/serialization layers handle    the translation (e.g., given_name → givenName). |
| --- |

## **2.2 Hierarchical Namespace**

Every internal AIX identifier (agent configs, document types, quality passes) follows a strict four-part hierarchy. Each segment narrows scope from broad to specific. Underscores serve exclusively as namespace separators — they are not snake_case within a segment.

| {component}*{objectType}*{identifier}_{property} |
| --- |
| writingAgent_documentType_article_systemPrompt |
| writingAgent_documentType_newsletter_outputRules |
| researchAgent_qualityPass_hallucinationSweep_systemPrompt |
| personaAgent_voice_brand |

## **2.3 Schema.org Base \\+ AIX Extensions**

For persona data and interchange fields, AIX uses Schema.org canonical property names where they exist. When Schema.org does not have a suitable property — particularly for disambiguation (e.g., business vs. personal phone) or platform-specific fields (e.g., individual social media profiles) — AIX defines extensions, marked 'AIX extension' in the Source column.

| Layer | Example Fields | Rule |
| --- | --- | --- |
| Schema.org canonical | givenName, familyName, jobTitle, legalName, telephone, email, naics, foundingDate | Use as-is, no modification |
| AIX extensions | telephoneMobile, emailBusiness, linkedinUrl, twitterUrl | Only when Schema.org has no equivalent |
| AIX hierarchy keys | writingAgent_documentType_article_systemPrompt | Four-part camelCase hierarchy for agent/config keys |

## **2.4 No Abbreviations**

All segments use full English words. The only exceptions are established industry acronyms registered in the AIX permitted acronym list.

| ✗ Prohibited (abbreviated) | ✓ Required (full word) |
| --- | --- |
| liUrl | linkedinUrl |
| sp | systemPrompt |
| or | outputRules |
| docType | documentType |
| qPass | qualityPass |
| ra | researchAgent |

Permitted acronyms: aix, api, url, html, json, uuid, naics, rss

> `rss` added 2026-05-12. RSS (Really Simple Syndication / RDF Site Summary) has been the W3C-canonical name for the format since 2002 and predates this specification by 24 years. It already ships in production AIX-implementing systems as the `ingest_sources.source_type='rss'` value (Solo Trillion, `20260507_ingest_sources.sql:15`). Registration acknowledges existing usage and unblocks `researchModule_rssDiscovery_*` hierarchy keys for the RSS Discovery module without forcing a verbose spell-out alternative.

## **2.5 Identifier Names the Format — Properties Define It**

The identifier segment (position 3 in the hierarchy) uses a short, stable handle. The systemPrompt and outputRules properties carry the full behavioral specification. Never encode behavior into the identifier name.

| // ✓ Correct — identifier is a short stable handle |
| --- |
| writingAgent_documentType_newsletter_systemPrompt |
| writingAgent_documentType_newsletter_outputRules |
| --- |
| // ✗ Wrong — behavior encoded into identifier name |
| writingAgent_documentType_formattedNewsletterWithCuratedSections_systemPrompt |

## **2.6 Naming Validation Tests**

Before any new key is ratified, it must pass all six tests. A key that fails any test must be revised.

| Test | Question | Fail Condition |
| --- | --- | --- |
| 1\. Stranger Test | Could a developer with no internal context understand this key from its name alone? | Requires docs or explanation to parse |
| 2\. Hierarchy Test | Does the key follow {component}*{objectType}*{identifier}_{property}? | Missing segment when more specificity exists |
| 3\. Abbreviation Test | Does the key contain any prohibited abbreviated segments? | Any segment not a full word or registered acronym |
| 4\. Case Test | Is every segment in camelCase? | Any snake_case, kebab-case, or UPPER_SNAKE within a segment |
| 5\. Redundancy Test | Does any segment restate or mirror an adjacent segment? | e.g. documentType_document — concept repeated |
| 6\. Conflict Test | Does the key duplicate or closely mirror an existing canonical key? | Duplicate or near-duplicate of a ratified key |

# **3\\. Persona Field Registry**

The Persona Field Registry defines every field in the AIX identity vault. Each field is either a Schema.org canonical property (used as-is) or an AIX extension (defined because Schema.org has no equivalent). This registry is the authoritative source for all Persona Vault implementations.

| ℹ  Schema.org canonical fields are used as-is. AIX extension fields are marked 'AIX extension'    in the Source column. The field key is the bare name — e.g., linkedinUrl, not aix:linkedinUrl.    this specification. See Section 1.3 for heritage rationale. |
| --- |

## **3.1 Essentials**

| AIX Field Key | Source | Description | Example Value |
| --- | --- | --- | --- |
| givenName | Schema.org | Legal first name | Michael |
| familyName | Schema.org | Legal last name | Tchong |
| preferredName | AIX extension | Preferred name or nickname if different from legal name | Mike |
| jobTitle | Schema.org | Professional title | Founder & CEO |
| worksFor | Schema.org | Primary organization name | Ubertrends LLC |
| industry | AIX extension | Industry or sector (Schema.org has no direct property) | Technology Consulting |
| addressHome | AIX extension | Home or personal display location (e.g., City, State) | Las Vegas, NV |
| timezone | AIX extension | IANA timezone identifier | America/Los_Angeles |

## **3.2 Professional**

| AIX Field Key | Source | Description |
| --- | --- | --- |
| activeProject1 | AIX extension | Current primary project name and one-line description |
| activeProject2 | AIX extension | Current secondary project |
| currentGoal1 | AIX extension | Active goal with target date |
| currentGoal2 | AIX extension | Active goal with target date |
| currentGoal3 | AIX extension | Active goal with target date |
| expertise | AIX extension | Core areas of expertise (comma-separated or prose) |
| tools | AIX extension | Primary tools and platforms currently in use |
| resume | AIX extension | Short professional bio or resume summary |

## **3.3 Organization**

These fields describe the author's organization or business entity. They use Schema.org Organization vocabulary where available.

| AIX Field Key | Source | Description |
| --- | --- | --- |
| legalName | Schema.org | Full legal name of the organization |
| description | Schema.org | Short organization description (elevator pitch) |
| companyBackgrounder | AIX extension | Longer organization backgrounder for press or partners |
| productServiceDescription1 | AIX extension | Primary product or service description |
| productServiceDescription2 | AIX extension | Secondary product or service description |
| url | Schema.org | Primary website URL |
| websiteUrl2 | AIX extension | Secondary website URL (Schema.org url is singular) |

## **3.4 Financial & Legal**

Schema.org has Organization-level financial properties. Fields not covered use AIX extensions.

| AIX Field Key | Source | Description |
| --- | --- | --- |
| legalName | Schema.org | Legal entity name (also in Organization — shared field) |
| entityType | AIX extension | Legal entity type (LLC, Corp, Sole Prop, etc.) |
| foundingDate | Schema.org | Date organization was founded or incorporated |
| naics | Schema.org | NAICS industry classification code |
| taxCategory | AIX extension | Tax classification (e.g., Disregarded Entity, S-Corp) |
| addressBusiness | AIX extension | Full business mailing address |
| addressHome | AIX extension | Home or personal display location (City, State or full address) |
| bank | AIX extension | Primary banking institution name |
| references | AIX extension | Professional references (names, titles, contact info) |
| telephoneBusiness | AIX extension | Business/office telephone number |
| telephoneMobile | AIX extension | Mobile telephone number |

| ℹ  AIX Disambiguation Pattern — Schema.org has single-value properties for address,    telephone, and email. AIX extends each with a qualifier suffix to disambiguate context:    addressBusiness  /  addressHome    telephoneBusiness  /  telephoneMobile    emailBusiness  /  emailPersonal    The Schema.org base property (address, telephone, email) is not used directly.    The AIX pair replaces the Schema.org base property entirely for all Persona Vault implementations. |
| --- |

## **3.5 Media Collections**

AIX v2.0 shipped without a persona-level field for the author's media surface — the publication, podcast, channel, or platform where content appears. Implementations filled this gap with non-canonical placeholders like `{publication_name}`, which don't correspond to any AIX field, don't pass the Stranger Test, and can't represent authors with multiple media surfaces.

A single author commonly operates multiple surfaces, each with its own name, optional brand, optional tagline, and content voice. A flat scalar field collapses this to one value and forces manual override per generation. An enumerated approach (`mediumNewsletter`, `mediumPodcast`, `mediumYtChannel`) was considered and rejected: the taxonomy is unbounded, every new platform forces a spec amendment, field keys hardcode platform names that become stale, and most users would populate 0–2 of N fields. The correct shape is a typed collection.

AIX v2.1 introduces the `media` array on the persona object: a collection of Medium entries, each typed with controlled vocabulary, carrying its own name, optional brand, optional tagline, optional URL, and a primary flag.

### **Field Definitions**

| AIX Field Key | Source | Type | Required | Description |
| --- | --- | --- | --- | --- |
| media | AIX extension | array | No | Ordered collection of Medium objects representing the author's media surfaces |
| mediumType | AIX extension | string | Yes (per entry) | Controlled vocabulary — see table below |
| mediumName | AIX extension | string | Yes (per entry) | User-facing display name of this media surface |
| mediumBrand | AIX extension | string | No | Optional umbrella brand if distinct from mediumName |
| mediumTagline | AIX extension | string | No | Per-medium tagline; defaults to persona tagline if absent |
| mediumUrl | AIX extension | string (URL) | No | Canonical URL for this media surface |
| isPrimary | AIX extension | boolean | Yes (per entry) | Exactly one entry per persona must be true |

> **`mediumName`**** vs. \****`worksFor`** — `worksFor` names the organization or company the author is affiliated with (e.g., "Ubertrends LLC"). `mediumName` names the media surface where content appears (e.g., "Ubertrends" newsletter). For solo operators whose company IS their media brand, the values may be near-identical. For authors at larger organizations, they typically differ. Template substitution contexts differ: `{worksFor}` = organizational affiliation; `{mediumName}` = publication/media byline. Example: `worksFor = "Ubertrends LLC"`, `mediumName = "Ubertrends"`.

> **Naming rationale** — `mediumName` was chosen over alternatives (`publicationName`, `periodicalName`, `seriesName`, `outletName`) because it generalizes across newsletter, podcast, channel, blog, and platform without constraining to print-era semantics. `publicationName` implies formal editorial operation; `periodicalName` aligns with Schema.org but is not in common usage; `mediumName` is medium-neutral and passes the Stranger Test without semantic footnotes. The minor brand collision with Medium.com is noted but not disqualifying — AIX field keys are always contextually unambiguous on a Person or author object.

### **`mediumType`**** Controlled Vocabulary**

AIX v2.1 ratifies the following baseline vocabulary for `mediumType`. The vocabulary is extensible in future minor versions via the AIX amendment process (Section 11.3). Implementations MUST accept unknown `mediumType` values without error, treating them as `other`.

| Value | Description | Examples |
| --- | --- | --- |
| `newsletter` | Email-delivered periodic publication | Ubertrends, The Hustle, Morning Brew |
| `blog` | Web-native long-form writing surface | Waitbutwhy, Stratechery |
| `podcast` | Audio episodic series | The Tim Ferriss Show, Lex Fridman Podcast |
| `channel` | Video episodic series — platform-neutral | Veritasium (YouTube), MKBHD (YouTube/Rumble) |
| `social` | Broadcast-style social presence | LinkedIn, X/Twitter, Threads, Bluesky, Mastodon |
| `platform` | Product or service where content appears | Solo Trillion, a SaaS product blog |
| `publication` | Formal editorial container | Substack publication, Ghost publication, Medium publication |
| `community` | Membership or discussion space | Discord server, Slack community, Circle community |
| `other` | Any surface not covered above | — |

The distinction between `channel` and `social` is editorial cadence: channels are episodic long-form (a YouTube weekly show), social surfaces are broadcast-style (a LinkedIn post stream). When ambiguous, prefer the type that best describes the dominant content cadence.

### **Template Substitution Resolution Rule**

When a Writing Agent resolves `{mediumName}`, `{mediumBrand}`, or `{mediumTagline}` at generation time, it follows this four-step rule:

| Step | Condition | Action |
| --- | --- | --- |
| 1 | Generation request specifies `mediumId` or `mediumType` | Substitute from the matching `media[]` entry |
| 2 | No request-level override | Substitute from the entry where `isPrimary: true` |
| 3 | No primary entry | Substitute from `media[0]` |
| 4 | `media[]` is empty or absent | Fall back to `worksFor` for `{mediumName}`; log a warning recommending the user populate `media[]` |

This resolution rule is normative. Any AIX-compliant Writing Agent must implement it identically to ensure consistent output across implementations.

### **Backwards Compatibility**
AIX v2.1 is purely additive. v2.0 implementations that do not use `media[]` continue to work unchanged. The non-canonical `{publication_name}` placeholder used in some v2.0 templates is deprecated in favor of `{mediumName}`. Implementations SHOULD migrate to `{mediumName}` but MAY continue to use `{publication_name}` during transition. `{publication_name}` will be removed from the AIX specification in v3.0.

Schema.org offers no canonical flat property suitable for persona-level media surface identification. `media[]` is therefore an AIX extension. Per AIX heritage (Section 1.3), the evaluated Schema.org terms and their disposition:

| Schema.org Term | Type | AIX Relationship |
| --- | --- | --- |
| `Periodical` | Class | Represents a publication entity (magazine, journal, newsletter). Useful for JSON-LD output but not a flat Person-level property. |
| `CreativeWorkSeries` | Class | Parent class for TV series, book series, periodicals. Too broad for persona-level template substitution. |
| `publisher` | Property | Names the organization that publishes — not the publication itself. Maps to `worksFor`, not `mediumName`. |
| `isPartOf` | Relation | Links an Article to its containing publication. Schema.org's pattern is nested containment (`Article.isPartOf.Periodical`), which does not fit AIX's flat-substitution model. |

AIX implementations MAY emit Schema.org `Periodical` or `CreativeWorkSeries` markup when producing JSON-LD output, using the medium entry's `mediumName` and `mediumUrl` fields. This is a presentation-layer concern, not a persona-storage concern.

### **Stranger Test Validation**

| Field Key | Result | Rationale |
| --- | --- | --- |
| `mediumType` | ✅ PASS | "Type of medium" — self-describing; `type` suffix is standard AIX pattern |
| `mediumName` | ✅ PASS | "Name of the medium" — unambiguous |
| `mediumBrand` | ✅ PASS | "Brand associated with the medium" — clear |
| `mediumTagline` | ✅ PASS | "Tagline for the medium" — unambiguous |
| `mediumUrl` | ✅ PASS | Aligns with existing `{platform}Url` pattern; `Url` suffix required per §3.5 naming convention |
| `isPrimary` | ✅ PASS | Boolean flag — "is this the primary entry" is self-evident |

### **Reference Implementation Example**

The following example demonstrates a persona with two media surfaces. `mediumBrand` illustrates the optional umbrella brand pattern.

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

`mediumBrand` ("FutureSight") names the umbrella brand under which the newsletter publishes. `mediumName` ("Ubertrends") is the editorial identity as it appears to readers. For generation requests targeting the newsletter, `{mediumName}` resolves to "Ubertrends" and `{mediumBrand}` resolves to "FutureSight". The platform entry illustrates `mediumType: "platform"` — a product or service where content appears alongside the author's editorial surfaces.



### **Single-Primary Subset Pattern (Appendix E.1 reference)**

### **Schema.org Alignment**
For implementations that need a simple starting point before supporting the full `media[]` array, Appendix E.1 documents a single-primary subset pattern: store one `mediumName` value per persona, use the four-step resolution rule with only Step 2 and Step 4 active, and migrate to the full array in a subsequent release. This pattern is a staging approach only — production implementations should target the full `media[]` schema.

**Single-Primary Reference Implementation (STO Beta, 2026-04-25):**

- **Storage:** one `persona_fields` row with `field_key = "mediumName"`, display label "Publication Name", stored in the professional category.
- **Cascade chain for template fallback:** `mediumName` → `worksFor` → `legalName`.
- **Template usage:** Newsletter heading "The {mediumName} Take" resolves to "The Ubertrends Take". `{tagline}` is retained in author-descriptor contexts; only publication-name positions use `{mediumName}`.
- **`worksFor`**** vs. \****`mediumName`**\*\* in practice:** `worksFor = "Ubertrends LLC"`, `mediumName = "Ubertrends"` — the organization and the media surface have different names. For a solo operator whose brand IS their company, the values may converge.

## **3.6 Social Media**

Schema.org's sameAs property accepts an array of profile URLs but provides no per-platform disambiguation. AIX defines individual extension fields for each platform so agents can reference specific profiles by name without parsing URL arrays.

| AIX Field Key | Source | Description | Format |
| --- | --- | --- | --- |
| linkedinUrl | AIX extension | LinkedIn profile URL | https://linkedin.com/in/handle |
| twitterUrl | AIX extension | X/Twitter profile URL | https://x.com/handle |
| facebookUrl | AIX extension | Facebook profile or page URL | https://facebook.com/handle |
| instagramUrl | AIX extension | Instagram profile URL | https://instagram.com/handle |
| youtubeUrl | AIX extension | YouTube channel URL | https://youtube.com/@handle |
| tiktokUrl | AIX extension | TikTok profile URL | https://tiktok.com/@handle |
| substackUrl | AIX extension | Substack publication URL | https://name.substack.com |

| ℹ  Naming pattern: {platform}Url  e.g., linkedinUrl, twitterUrl, youtubeUrl    The Url suffix is required. Without it, 'linkedin' is ambiguous — it could be a    handle string, a URL, or a boolean. The Url suffix passes the Stranger Test.    Schema.org equivalent: sameAs (array) — insufficient for per-platform agent use. |
| --- |

## **3.****7 Contact (Email)**

Schema.org has a single email property. AIX extensions disambiguate by context.

| AIX Field Key | Source | Description |
| --- | --- | --- |
| email | Schema.org | Primary email address |
| emailBusiness | AIX extension | Business email address |
| emailPersonal | AIX extension | Personal email address |

## **3.8 Preferences**

| AIX Field Key | Source | Description |
| --- | --- | --- |
| commStyle | AIX extension | Preferred communication style with this agent |
| checkinPreference | AIX extension | How frequently and in what format to receive check-ins |
| interests | AIX extension | Personal and professional interests |
| offLimits | AIX extension | Topics or subjects to avoid in generated content |
| responseFormat | AIX extension | Preferred response format (prose, bullets, structured, etc.) |

## **3.9 Conversational Agent**

Fields describing the conversational agent assigned to this identity vault. The default name is Rosie — a user-configurable display name stored as a value, not a key.

| AIX Field Key | Source | Description |
| --- | --- | --- |
| conversationalAgentName | AIX extension | Display name of the conversational agent. Default: Rosie |
| conversationalAgentVoiceId | AIX extension | TTS voice identifier for the conversational agent (e.g., ElevenLabs voice ID) |
| conversationalAgentSystemPrompt | AIX extension | System prompt defining the conversational agent persona |

# **4\\. Component & Agent Registry**

Components are the top-level namespace segment in all AIX hierarchy keys. New components require a spec amendment before use.

| Component Key | Full Name | Description |
| --- | --- | --- |
| researchAgent | Research Agent | Web research, source synthesis, hallucination detection |
| writingAgent | Writing Agent | Content generation, document type management, output formatting |
| personaAgent | Persona Agent | Identity vault, voice, guidelines, guardrails |
| conversationalAgent | Conversational Agent | Conversational assistant (Telegram \+ web) — default name: Rosie |
| orchestra | Orchestra | Platform-level orchestration, agent chaining, workflow config |
| aix | AIX Standard | Specification-level fields (version, schema, envelope) |

## **4.7 Agent Behavioral Modes**

AIX-compliant agents may declare a behavioral mode that governs how they respond to inputs and initiate actions. Behavioral modes allow orchestration layers, users, and other agents to understand and control agent behavior at the protocol level without modifying the agent's underlying configuration.

### **4.7.1 The Problem**

Without a standard mode declaration, agents default to their training bias — typically "helpful general assistant" — regardless of the role they are assigned. This causes agents to exceed their intended scope, volunteer actions outside their function, and behave inconsistently across sessions. A naming standard alone does not solve this; behavior must be declarable and queryable at the protocol level.

### **4.7.2 The behavioralMode Field**

The `behavioralMode` field is an optional string property in the AIX agent object. When present, it declares the agent's current operating mode.

| { |
| --- |
| "aix": "2.1", |
| "agent": { |
| "id": "rosie", |
| "role": "chief_of_staff", |
| "behavioralMode": "observer", |
| "allowedModes": \[ |
| "observer", |
| "orchestrator", |
| "briefing", |
| "off" |
| \] |
| } |
| } |

### **4.7.3 Standard Mode Vocabulary**

AIX 2.1 ratifies four standard behavioral modes. Implementations may extend this vocabulary using the AIX extension pattern (Section 2.3), subject to the six validation tests (Section 2.6).

| Mode | Description | Agent Behavior |
| --- | --- | --- |
| `observer` | Default monitoring state | Listens, classifies, surfaces signals. Does not initiate external actions without explicit trigger. |
| `orchestrator` | Active pipeline execution | Runs workflows, coordinates other agents, executes scheduled tasks. |
| `briefing` | Delivery state | Formats and delivers digests, summaries, and reports. Read-only relative to source data. |
| `off` | Silent state | No proactive outreach. Responds to direct queries only. Executes no scheduled tasks. |

### **4.7.4 Mode Transitions**

Behavioral mode transitions must be explicit. An agent must not self-transition between modes based on conversation context alone. Mode changes require one of:

- A direct user instruction
- A scheduled trigger defined in the agent's configuration
- An orchestration signal from another AIX-compliant agent carrying a valid `behavioralMode` transition request

### **4.7.5 Mode Declaration in Task Envelopes**

When an agent sends or receives an AIX task envelope, it may include its current behavioral mode in the metadata object:

| { |
| --- |
| "aix": "2.1", |
| "task": { |
| "type": "content_authoring", |
| "provider": "anthropic", |
| "model": "claude-sonnet-4-6" |
| }, |
| "meta": { |
| "requestId": "req_abc123", |
| "timestamp": "2026-04-19T09:00:00Z", |
| "agentMode": "orchestrator" |
| } |
| } |

Receiving agents may use the `agentMode` field to calibrate their response — for example, a Writing Agent receiving a task from an agent in `observer` mode may treat it as a draft request rather than a publish request.

### **4.7.6 Soul Documents**

AIX 2.1 introduces the concept of a Soul Document — a persistent identity file that defines an agent's role, constraints, personality, and behavioral boundaries. Soul Documents are loaded before any system prompt, conversation history, or user input, establishing identity as the first layer of context.

Soul Documents are implementation-specific and not part of the AIX interchange format. However, AIX recommends the following minimum structure:

| Section | Required | Description |
| --- | --- | --- |
| Identity | Yes | Who the agent is and what role it serves |
| Role | Yes | Specific functions the agent performs |
| Constraints | Yes | Explicit list of what the agent never does |
| Behavioral Modes | Recommended | Mapping to AIX 4.7.3 standard vocabulary |
| Tone Examples | Recommended | Compliant and non-compliant response examples |

| ℹ  The Soul Document pattern was formalized in AIX 2.1 based on implementation experience    with Solo Trillion's Rosie agent and the OpenClaw project's SOUL.md architecture. |
| --- |

### **4.7.7 Relationship to Existing Standards**

| Standard | Relationship |
| --- | --- |
| MCP (Model Context Protocol) | MCP defines what data an agent can access. AIX behavioral modes define how the agent acts on that data. Complementary. |
| OpenAI Assistants API | Assistants have instructions but no standard mode vocabulary. AIX modes are interoperable across providers. |
| LangChain Agent Types | LangChain defines execution strategies (ReAct, Plan-and-Execute). AIX modes define behavioral scope. Complementary. |

# **5\\. Reserved Segment Vocabulary**

## **5.1 Object Type Segments (Position 2\\)**

| Object Segment | Usage | Example |
| --- | --- | --- |
| documentType | Output format variants for the Writing Agent | writingAgent_documentType_article_systemPrompt |
| qualityPass | Error-check and verification passes (any agent) | researchAgent_qualityPass_hallucinationSweep_systemPrompt |
| config | General agent operational configuration | researchAgent_config_default_systemPrompt |
| voice | Author identity and brand voice data | personaAgent_voice_brand |
| output | Generation output schema fields | writingAgent_output_draft_headline |
| input | Generation input schema fields | writingAgent_input_contentBrief_format |
| schema | JSON schema definitions | aix_schema_contentAuthoring_input |
| meta | Metadata (cost, latency, tokens) | aix_meta_cost_amount |

## **5.2 Identifier Segments (Position 3\\)**

### **Document Type Identifiers**

| Identifier | Format Description |
| --- | --- |
| article | Default long-form editorial article |
| fieldGuide | Solo Trillion Field Guide practitioner reference format |
| meetingBrief | Pre-meeting intelligence brief — condensed, actionable |
| executiveBrief | Executive summary format |
| competitiveAnalysis | Competitive analysis structured comparison report |
| newsletter | Newsletter edition |
| linkedinPost | LinkedIn post |

### **Quality Pass Identifiers**

| Identifier | Description |
| --- | --- |
| researchBuild | Initial data gathering and source synthesis pass |
| factCheck | Verify all claims against cited sources |
| hallucinationSweep | Detect and flag fabricated or unsupported content |
| complianceEdit | Apply guardrails, voice rules, and editorial standards |
| qualityScore | Final quality assessment and confidence scoring |
| optimizationPass | Refine output for performance, cost, or length targets |

### **Voice Identifiers (under personaAgent\_voice)**

| Identifier | Description |
| --- | --- |
| brand | Author brand voice DNA — tone, vocabulary, sentence rhythm, signature phrases |
| guidelines | Editorial writing structure and style rules |
| guardrails | Hard stops, non-negotiables, topics requiring human review |

## **5.3 Property Segments (Position 4\\)**

| Property Suffix | Value Type | Description |
| --- | --- | --- |
| systemPrompt | string | LLM system prompt for the given context |
| outputRules | string | Formatting and structure rules for generated output |
| config | object | Configuration object (when used as terminal node) |
| schema | object | JSON Schema definition |
| template | object | Complete template bundle (prompt \+ rules \+ metadata) |
| label | string | Human-readable display label |
| description | string | Human-readable description |
| version | string | Semantic version string |
| enabled | boolean | Feature flag |

# **6\\. Canonical Key Reference**

The master reference for all currently ratified AIX hierarchy keys. Keys marked * were revised in v2.0. Keys marked ** are newly introduced in v2.0.

| Canonical Key | Type | Ver | Description |
| --- | --- | --- | --- |
| personaAgent_voice_brand | string | 1.1 | Author brand voice DNA |
| personaAgent_voice_guidelines | string | 1.1 | Editorial writing guidelines |
| personaAgent_voice_guardrails | string | 1.1 | Hard stops and non-negotiables |
| writingAgent_documentType_article_systemPrompt | string | 2.0* | Article system prompt |
| writingAgent_documentType_article_outputRules | string | 2.0* | Article output rules |
| writingAgent_documentType_fieldGuide_systemPrompt | string | 2.0* | Field Guide system prompt |
| writingAgent_documentType_fieldGuide_outputRules | string | 2.0* | Field Guide output rules |
| writingAgent_documentType_meetingBrief_systemPrompt | string | 2.0** | Meeting Brief system prompt |
| writingAgent_documentType_meetingBrief_outputRules | string | 2.0** | Meeting Brief output rules |
| writingAgent_documentType_executiveBrief_systemPrompt | string | 2.0* | Executive Brief system prompt |
| writingAgent_documentType_executiveBrief_outputRules | string | 2.0* | Executive Brief output rules |
| writingAgent_documentType_competitiveAnalysis_systemPrompt | string | 2.0* | Competitive Analysis system prompt |
| writingAgent_documentType_competitiveAnalysis_outputRules | string | 2.0* | Competitive Analysis output rules |
| writingAgent_documentType_newsletter_systemPrompt | string | 2.0* | Newsletter system prompt |
| writingAgent_documentType_newsletter_outputRules | string | 2.0* | Newsletter output rules |
| writingAgent_documentType_linkedinPost_systemPrompt | string | 2.0* | LinkedIn Post system prompt |
| writingAgent_documentType_linkedinPost_outputRules | string | 2.0* | LinkedIn Post output rules |
| writingAgent_config_default_systemPrompt | string | 2.0* | Default Writing Agent system prompt |
| writingAgent_config_default_outputRules | string | 2.0* | Default output rules |
| researchAgent_config_default_systemPrompt | string | 1.2 | Research Agent default system prompt |
| researchAgent_qualityPass_researchBuild_systemPrompt | string | 2.0** | Pass 1: Research Build prompt |
| researchAgent_qualityPass_factCheck_systemPrompt | string | 2.0** | Pass 2: Fact Check prompt |
| researchAgent_qualityPass_hallucinationSweep_systemPrompt | string | 2.0** | Pass 3: Hallucination Sweep prompt |
| researchAgent_qualityPass_complianceEdit_systemPrompt | string | 2.0** | Pass 4: Compliance Edit prompt |
| researchAgent_qualityPass_qualityScore_systemPrompt | string | 2.0** | Pass 5: Quality Score prompt |

| ℹ  * Keys marked 2.0* replace v1.2* keys. The v1.2 keys used 'template' as the object    segment and 'standardArticle/preMeeting' as identifiers. These are deprecated.    ****\* Keys marked 2.0****\* are newly introduced in this version.    Deprecated: writingAgent\_template\_*  researchAgent_config_passOne/passTwo/passThree_*    Deprecation policy: old keys remain in registry marked \[DEPRECATED\]. Do not use for new work. |
| --- |

# **7\\. AIX Envelope & Task Types**

## **7.1 The AIX Envelope**

Every AIX message is wrapped in an envelope. The aix field carries the specification version. The task, input, output, and meta fields are always present; error is included only on failure.

| { |
| --- |
| "aix": "2.0", |
| "id": "uuid-v4", |
| "timestamp": "ISO-8601", |
| "task": { "type": "...", "provider": "...", "model": "..." }, |
| "input": { }, |
| "output": { }, |
| "meta": { "cost": { }, "latency": { }, "tokens": { } }, |
| "error": { } |
| } |

## **7.2 Task Type Registry**

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

## **7.3 Metadata Object**

The metadata object captures operational information enabling comparison, cost tracking, and debugging across providers.

| "meta": { |
| --- |
| "cost":    { "amount": 0.018, "currency": "USD", "unit": "per_request" }, |
| "latency": { "total_ms": 8400, "processing_ms": 8100, "network_ms": 300 }, |
| "tokens":  { "input": 4200, "output": 1850, "total": 6050 }, |
| "model_version": "claude-sonnet-4-6", |
| "request_id": "req_abc123", |
| "region": "us-east-1", |
| "retention": { "input_stored": false, "output_stored": true, "retention_days": 30 } |
| } |

## **7.4 Error Object**

| "error": { |
| --- |
| "code": "rate_limited",       // invalid_input | rate_limited | quota_exceeded |  |
| // content_filtered | model_unavailable | timeout |  |
| // provider_error | unknown |
| "message": "Rate limit exceeded for this model.", |
| "retryable": true, |
| "retry_after_ms": 5000, |
| "provider_error": { }         // Raw error object from provider |
| } |

# **8\\. Task-Specific Schemas**

## **8.1 content\_authoring**

The content_authoring task type enables AI-generated content carrying full author identity, editorial voice, structural guidelines, and hard guardrails. Designed for human-in-the-loop publishing workflows where brand consistency and claim integrity are non-negotiable.

### **Input: author object**

```json
"author": {
  "givenName":   "Michael",
  "familyName":  "Tchong",
  "displayName": "Michael Tchong",
  "jobTitle":    "Founder, Solo Trillion",
  "bio":         "Short author bio for attribution",
  "socialProfiles": {
    "linkedinUrl":  "https://linkedin.com/in/michaeltchong",
    "twitterUrl":   "https://x.com/michaeltchong",
    "youtubeUrl":   "https://youtube.com/@michaeltchong",
    "url":          "https://ubertrends.com"
  },
  "media": [
    {
      "mediumType": "newsletter",
      "mediumName": "Ubertrends",
      "mediumBrand": "FutureSight",
      "isPrimary": true
    }
  ]
}
```

The `media` array in a generation request MAY be a subset of the persona's full media collection — include only the entry relevant to this generation request. Resolution rules are defined in Section 3.5.

### **Input: voice, research, brief**

| "voiceBrand":       "...",  // Author brand voice DNA: tone, vocabulary, sentence rhythm |
| --- |
| "voiceGuidelines":  "...",  // Structural/editorial rules: formats, sections, citations |
| "voiceGuardrails":  "...",  // Hard stops: topics to avoid, claims needing sourcing |
| --- |
| "researchPacket": { |
| "topic":           "...", |
| "thesis":          "...", |
| "narrativeAngle":  "...", |
| "keyFindings":     \[ { "finding": "...", "sourceUrl": "...", "confidence": 8, "isContrarian": true } \], |
| "traceabilityTable": \[ { "finding": "...", "sourceUrl": "...", "supportingPassage": "...", "confidence": 8 } \], |
| "auditSummary":    { "auditPassed": true, "hallucinationsFound": 0 } |
| }, |
| --- |
| "contentBrief": { |
| "format":             "article",  // article | newsletter | linkedinPost | executiveBrief | opEd |
| "toneOverride":       "contrarian",  // contrarian | analytical | explainer | inspirational |
| "targetLength":       "standard_1200",  // short_600 | standard_1200 | deepDive_2000 |
| "targetPublication":  "Solo Trillion", |
| "selectedFindings":   \[ \]  // Subset of keyFindings to emphasize; omit to use all |
| } |

### **Output**

| "output": { |
| --- |
| "draft": { |
| "headlineOptions": \[ "...", "...", "..." \],  // 3 variants ranked by contrarian strength |
| "lede":    "...", |
| "body":    "...",     // Full article in markdown |
| "sections": \[ { "heading": "...", "content": "..." } \], |
| "cta":     "...", |
| "wordCount": 1247, |
| "citations": \[ { "text": "...", "url": "...", "title": "..." } \] |
| }, |
| "guardrailFlags": \[ { "flag": "...", "location": "...", "recommendation": "..." } \], |
| "finishReason": "complete"  // complete | guardrailHalt | length | contentFilter |
| } |

## **8.2 voice\_synthesis**

| "input": { |
| --- |
| "text": "Welcome to the future of AI interchange standards.", |
| "voice": "aria",  "language": "en-US",  "speed": 1.0, |
| "emotion": "neutral",  "outputFormat": "mp3",  "sampleRate": 24000 |
| }, |
| "output": { |
| "audio": { "url": "...", "format": "mp3", "sampleRate": 24000, |
| "channels": 1, "durationMs": 3200, "fileSizeBytes": 51200 }, |
| "wordTimestamps": \[ { "word": "Welcome", "startMs": 0, "endMs": 320 } \] |
| } |

## **8.3 image\_generation**

| "input": { |
| --- |
| "prompt": "A futuristic city at sunset, cyberpunk style", |
| "negativePrompt": "blurry, low quality", |
| "width": 1024,  "height": 1024, |
| "style": "photorealistic",  "numOutputs": 1, |
| "outputFormat": "png",  "seed": 42 |
| }, |
| "output": { |
| "images": \[ { "url": "...", "format": "png", "width": 1024, "height": 1024, |
| "fileSizeBytes": 1048576, "revisedPrompt": "..." } \], |
| "nsfwDetected": false |
| } |

## **8.4 text\_generation**

| "input": { |
| --- |
| "prompt": "...",  "systemPrompt": "...", |
| "messages": \[ { "role": "user", "content": "..." } \], |
| "maxTokens": 2000,  "temperature": 1.0,  "stream": false, |
| "responseFormat": "markdown"  // text | json | markdown |
| }, |
| "output": { |
| "text": "...", |
| "finishReason": "complete",  // complete | length | stopSequence | contentFilter |
| "citations": \[ { "text": "...", "url": "...", "title": "..." } \] |
| } |

## **8.5 video\_generation**

| "input": { |
| --- |
| "prompt": "...", |
| "image": { "url": "..." },    // for image_to_video |
| "durationSeconds": 10,  "aspectRatio": "16:9", |
| "resolution": "1080p",  "fps": 24, |
| "motionIntensity": "medium",  "cameraMotion": "static" |
| }, |
| "output": { |
| "video": { "url": "...", "format": "mp4", "width": 1920, "height": 1080, |
| "durationMs": 10000, "fps": 24, "fileSizeBytes": 8400000 }, |
| "thumbnail": { "url": "...", "timestampMs": 500 } |
| } |

## **8.6 code\_generation**

| "input": { |
| --- |
| "prompt": "...",  "language": "python", |
| "context": "...",  "framework": "fastapi", |
| "includeTests": false,  "includeComments": true, |
| "styleGuide": "pep8" |
| }, |
| "output": { |
| "code": "...",  "language": "python",  "explanation": "...", |
| "files": \[ { "filename": "main.py", "content": "...", "language": "python" } \], |
| "dependencies": \[ { "name": "fastapi", "version": "^0.110.0" } \] |
| } |

# **9\\. Workflows & Chaining**

AIX supports chaining multiple tasks in a workflow. The output of one step becomes the input of the next via ${step_id.output.field} variable substitution.

| { |
| --- |
| "aix": "2.0", |
| "workflow": { |
| "id": "uuid", |
| "name": "Research → Write → Synthesize Voice", |
| "steps": \[ |
| { |
| "id": "step_research", |
| "task": { "type": "text_generation", "provider": "anthropic" }, |
| "input": { "prompt": "Research AI agents replacing knowledge workers" } |
| }, |
| { |
| "id": "step_article", |
| "task": { "type": "content_authoring", "provider": "anthropic" }, |
| "input": { |
| "researchPacket": "${step_research.output.text}", |
| "contentBrief": { "format": "article", "toneOverride": "contrarian" } |
| } |
| }, |
| { |
| "id": "step_audio", |
| "task": { "type": "voice_synthesis", "provider": "elevenlabs" }, |
| "input": { |
| "text": "${step_article.output.draft.lede}", |
| "voice": "aria" |
| } |
| } |
| \] |
| } |
| } |

# **10\\. Provider Comparison Format**

AIX enables side-by-side comparison of multiple providers for the same task. Standardized metadata makes cost and latency comparisons machine-readable.

| { |
| --- |
| "aix": "2.0", |
| "comparison": { |
| "task": { "type": "voice_synthesis" }, |
| "input": { "text": "Hello, this is a comparison test.", "language": "en-US" }, |
| "results": \[ |
| { |
| "provider": "elevenlabs", "model": "eleven_multilingual_v2", |
| "output": { "audio": { "url": "...", "durationMs": 2100 } }, |
| "meta":   { "cost": { "amount": 0.002 }, "latency": { "totalMs": 650 } } |
| }, |
| { |
| "provider": "openai", "model": "tts-1-hd", |
| "output": { "audio": { "url": "...", "durationMs": 2080 } }, |
| "meta":   { "cost": { "amount": 0.003 }, "latency": { "totalMs": 580 } } |
| } |
| \] |
| } |
| } |

# **11\\. Security & Governance**

## **11.1 Security**

AIX does not define authentication — this is provider-specific. AIX messages must never contain API keys, passwords, or personal access tokens. Content filtering signals use a standardized safety object:

| "safety": { |
| --- |
| "flagged": true, |
| "categories": { "violence": false, "sexual": false, "hate": false, |
| "selfHarm": false, "dangerous": true }, |
| "actionTaken": "content_blocked" |
| } |

## **11.2 Versioning**

| Version Level | Trigger | Example |
| --- | --- | --- |
| Major (X.0) | Breaking changes to envelope, required fields, or task type removal | 1.x → 2.0 |
| Minor (2.X) | New task types, new optional fields, additive features | 2.0 → 2.1 |
| Patch (2.0.X) | Clarifications, documentation fixes, no schema changes | 2.0 → 2.0.1 |

## **11.3 Amendment Process**

The naming convention is a controlled document. To add a new key: (1) propose with full four-part hierarchy and plain-English description; (2) run all six validation tests from Section 2.6; (3) check for conflicts with Section 6; (4) add to candidate keys in next minor version draft; (5) upon ratification, move to canonical reference.

| ℹ  Retroactive changes to ratified keys are prohibited. Deprecation is handled by adding a    replacement key and marking the old key \[DEPRECATED\] in the registry — not by renaming it. |
| --- |

## **11.4 Governance**

| Body | Responsibility |
| --- | --- |
| Technical Steering Committee | Specification changes and ratification |
| Provider Council | Implementation feedback and provider registry |
| Community Contributors | Pull requests, issues, and proposals |

# **12\\. Appendix**

## **A. Provider Registry**

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

## **B. Language Codes (BCP-47)**

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

## **C. Media Type Mappings**

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

## **D. References**

https://github.com/modelcontextprotocolhttps://github.com/modelcontextprotocol1\.  Model Context Protocol (MCP) — https://github.com/modelcontextprotocol

https://github.com/edenai/edenai-apishttps://github.com/edenai/edenai-apis2\.  Eden AI APIs — https://github.com/edenai/edenai-apis

https://schema.orghttps://schema.org3\.  Schema.org — https://schema.org

https://json-schema.orghttps://json-schema.org4\.  JSON Schema — https://json-schema.org

https://www.rfc-editor.org/info/bcp47https://www.rfc-editor.org/info/bcp475\.  BCP-47 Language Tags — https://www.rfc-editor.org/info/bcp47

6\.  CrossARM Data Interchange Standard (2014) — Ubertrends LLC (internal)

## **E. Implementation Notes (Non-Normative)**

| ℹ  This appendix is non-normative. Nothing in Appendix E adds requirements to the AIX standard. Material here documents staging patterns, migration sequences, and operational guidance that implementers have found useful. AIX-compliant behavior is fully defined by the normative sections (1–11). When the spec body and this appendix appear to disagree, the spec body governs. A future v2.2+ may extract implementation guidance into a separate AIX Implementation Guide document. |
| --- |

### **E.1 Section 3.5 Media Collections — Single-Primary Subset Pattern**

AIX-compliant implementations MAY ship a single-primary subset of `media\[\]` before supporting the full array. The subset stores one Medium object on the persona (the primary one) and applies Resolution Rule Step 2 (Section 3.5) directly — Steps 1, 3, and 4 become trivial because the collection always has exactly one entry.

The full array is purely additive: a later upgrade adds the array shape and Resolution Steps 1, 3, and 4 without breaking the subset implementation. Templates and Writing Agents that use `{mediumName}` continue to resolve identically across the upgrade.

The Solo Trillion reference implementation uses this staging pattern — single-primary subset at beta launch (May 2026), full array post-beta. The subset is sufficient for an author with one publication; the full array becomes necessary as soon as that author publishes under a second surface (newsletter + podcast, newsletter + platform, etc.).

This pattern is not Section 3.5 specific. Other v2.1 collections (e.g. `allowedModes\[\]` in Section 4.7) admit similar single-entry subset implementations during initial rollout.

# **13\\. Changelog**

## **Version 2.1 (Draft) — April 2026**

Minor version: additive. Introduces Media Collections standard, Agent Behavioral Modes standard, and Soul Document specification.

| Area | Change |
| --- | --- |
| Section 3.5 (NEW) | Media Collections — `media[]` typed collection with `mediumType` controlled vocabulary, four-step template substitution resolution rule, Schema.org alignment, Stranger Test validation, reference implementation, backwards compatibility note. Deprecates non-canonical `{publication_name}` placeholder in favor of `{mediumName}`. |
| Section 3.5 `mediumName` vs `worksFor` | Callout added distinguishing organizational affiliation (`worksFor`) from media surface identity (`mediumName`). Solo operator example: `worksFor = "Ubertrends LLC"`, `mediumName = "Ubertrends"`. |
| Section 3.5 naming rationale | Non-normative aside documenting why `mediumName` was chosen over `publicationName`, `periodicalName`, `seriesName`, `outletName`. |
| Section 3.5 vocabulary | `mediumType` controlled vocabulary: newsletter, blog, podcast, channel, social, platform, publication, community, other. Implementations MUST accept unknown values without error. |
| Section 3.5 resolution | Four-step substitution rule for `{mediumName}`, `{mediumBrand}`, `{mediumTagline}`: request-specified → primary → first → `worksFor` fallback. Rule is normative. |
| Section 3.5 Stranger Test | Restructured as a table; all six `media[]` field keys pass. |
| Section 3.5 backwards compatibility | `{publication_name}` will be removed in v3.0 (strengthened from "during transition" to explicit v3.0 removal). |
| Sections 3.6–3.9 | Renumbered from 3.5–3.8 (Social Media, Contact, Preferences, Conversational Agent) to make room for new 3.5 |
| Section 8.1 author object | `media` array added; clean JSON example replaces one-row table format. `mediumBrand` included in example. |
| Appendix E.1 (NEW) | Single-Primary Subset Pattern — staging guidance for implementations migrating from scalar `mediumName` to full `media[]` array. STO beta reference implementation cited: `persona_fields` storage, `mediumName → worksFor → legalName` cascade, Newsletter `{tagline} Take → {mediumName} Take`. |
| Reference implementation | First production single-primary subset confirmed (STO Beta, 2026-04-25). "Publication Name" display label, `mediumName → worksFor → legalName` cascade. |
| Deprecated placeholder | `{publication_name}` deprecated in favor of `{mediumName}`. Implementations SHOULD migrate; `{publication_name}` removed in v3.0. |
| Section 4.7 (NEW) | Agent Behavioral Modes — `behavioralMode` field, standard mode vocabulary (observer/orchestrator/briefing/off), mode transition rules |
| Section 4.7.5 (NEW) | `agentMode` field in task envelope metadata — receiving agents may calibrate response based on sender mode |
| Section 4.7.6 (NEW) | Soul Documents — persistent agent identity file standard, recommended minimum structure |
| Section 4.7.7 (NEW) | Behavioral modes relationship to MCP, OpenAI Assistants API, and LangChain Agent Types |
| Envelope | `aix` version field updated to "2.1" in behavioral mode examples |

## **Version 2.0 (Draft) — March 2026**

Major version: merges AIX Interchange Format Specification and AIX Naming Convention Standard into a single authoritative document.

| Area | Change |
| --- | --- |
| Document structure | Merged two specs (v1.1 interchange \+ naming convention) into AIX v2.0 |
| Section 3 (NEW) | Persona Field Registry — 40+ fields with Schema.org base or AIX extension designation |
| Section 4 (NEW) | Component & Agent Registry formalized |
| Section 5 (NEW) | Reserved Segment Vocabulary — object types, identifiers, property suffixes |
| Author object | givenName/familyName replace first_name/last_name (Schema.org canonical) |
| Social media fields | linkedinUrl, twitterUrl, facebookUrl etc. — Url suffix required, aix: prefix removed from field keys |
| Phone fields | telephoneMobile, telephoneBusiness — Schema.org base word, qualifier suffix disambiguates |
| Email fields | emailBusiness, emailPersonal — disambiguation extensions |
| Address fields | addressHome and addressBusiness replace Schema.org address — same disambiguation pattern as telephone and email |
| Heritage section | CrossARM 2014 lineage documented in Section 1.3 |
| Naming tests | Six validation tests moved to Section 2.6 (previously Appendix D) |
| Deprecated | writingAgent_template_* | researchAgent_config_passOne/Two/Three_* |

## **Version 1.1 (Draft) — February 2026**

| Area | Change |
| --- | --- |
| content_authoring | New task type added (Section 4.6) |
| author object | first_name, last_name, display_name, title, bio, social_media_profiles |
| voice fields | voice_brand, voice_guidelines, voice_guardrails |
| research_packet | Research Agent → Writing Agent chaining input |
| guardrail_flags | Human review signaling in output |

## **Version 1.0 (Draft) — December 2025**

Initial specification. Core task types: text, voice, image, video, code. Metadata schema with cost/latency tracking. Workflow chaining support. Provider comparison format.

*Apache 2.0 Licensed  ·  aix-standard.org  ·  github.com/aix-standard/aix-spec  ·  ****aix@aix-standard.org****  ·  Copyright 2026 Michael Tchong, Ubertrends LLC*