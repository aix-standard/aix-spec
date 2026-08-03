**AIX**

**AI Interchange Standard**

*Version 2.0 Draft  ·  March 2026*

| Author: Michael Tchong, Ubertrends LLC License: Apache 2.0 (Open Standard) *AIX is an open standard for naming, exchanging data, and establishing interoperability between generative AI tools, agents, and applications. This document merges the AIX Interchange Format Specification (v1.1) and the AIX Naming Convention Standard into a single authoritative reference.* Heritage: CrossARM (2014) · Ubertrends LLC · Solo Trillion |
| :---- |

| Section | Contents |
| :---- | :---- |
| 1\. Introduction | Problem statement, CrossARM lineage, Schema.org relationship |
| 2\. Core Naming Principles | camelCase, hierarchy, no abbreviations, validation tests |
| 3\. Persona Field Registry | Schema.org base fields \+ AIX extensions for all persona fields |
| 4\. Component & Agent Registry | All registered AIX components |
| 5\. Reserved Segment Vocabulary | Object types, identifiers, property suffixes |
| 6\. Canonical Key Reference | All ratified AIX keys |
| 7\. AIX Envelope & Task Types | Interchange format, task schema, metadata |
| 8\. Task-Specific Schemas | Voice, image, text, video, code, content\_authoring |
| 9\. Workflows & Chaining | Multi-step pipeline schema |
| 10\. Provider Comparison Format | Side-by-side output schema |
| 11\. Security & Governance | Auth, safety, versioning, amendment process |
| 12\. Appendix | Provider list, language codes, media types |
| 13\. Changelog | Version history |

# **1\. Introduction**

## **1.1 The Problem**

The generative AI revolution has produced hundreds of specialized tools — each with its own API format, authentication method, and response structure. This fragmentation creates developer burden through custom integration code for each provider, vendor lock-in when switching providers requires rewriting applications, workflow friction when chaining tools requires format conversion, and no standard way to compare outputs across providers.

A parallel problem exists inside AI agents themselves: without a naming standard, every team invents its own key vocabulary for fields, configuration properties, and persona data. Keys proliferate without governance — some snake\_case, some camelCase, some abbreviated, some redundant — until the system becomes unmaintainable and incompatible with external implementers.

## **1.2 The Solution**

AIX addresses both problems with a single specification. It defines a standard envelope for AI task requests and responses so any AIX-compliant tool can accept standardized inputs, return standardized outputs, be swapped with any other tool in the same category, and chain seamlessly with other AIX tools in workflows. It also defines a canonical naming convention so every field, key, and identifier in the AIX ecosystem is unambiguous, hierarchical, and implementable by developers with no internal context.

## **1.3 Heritage: CrossARM (2014)**

AIX follows the pattern established by CrossARM, a data interchange standard developed by Ubertrends LLC in 2014 for cross-platform marketing data exchange. CrossARM's governing principle was: use Schema.org canonical names where they exist, and invent proprietary extensions for what Schema.org does not cover.

| // CrossARM 2014 — Schema.org base vocabulary |
| :---- |
| givenName, familyName, worksFor, address, email, telephone  (Person) |
| brand, legalName, duns, isicV4                              (Organization) |
|   |
| // CrossARM 2014 — Proprietary extensions (Schema.org had no equivalent) |
| facebookName, twitterName, twitterFollowers, engagementRate |
| uniqueVisits, timeOnSite, bounceRate, influencerRating |

AIX v2.0 formalizes this pattern. Schema.org canonical names are used wherever applicable. Fields that Schema.org does not cover are defined as AIX extensions, noted as such in the registry.

## **1.4 Relationship to Existing Standards**

| Standard | Focus | AIX Relationship |
| :---- | :---- | :---- |
| MCP (Model Context Protocol) | Connecting AI to data sources | Complementary — MCP handles inputs, AIX handles outputs |
| Schema.org | Structured web data vocabulary | Foundation — AIX uses Schema.org canonical names as base layer |
| Eden AI Schemas | Traditional AI APIs (OCR, NLP) | Predecessor — AIX extends into generative AI task types |
| JSON Schema | Data validation | Implementation — AIX uses JSON Schema for field validation |
| CrossARM (2014) | Cross-platform marketing data | Heritage — established the Schema.org base \+ extension pattern |

# **2\. Core Naming Principles**

These principles govern every field name, key, and identifier in the AIX specification and in any AIX-compliant implementation.

## **2.1 camelCase Throughout**

All field names and identifiers use camelCase. No snake\_case, no kebab-case, no PascalCase for field names. This aligns with JSON, JavaScript, and modern API conventions.

| // ✓ Correct |
| :---- |
| givenName  familyName  jobTitle  telephoneMobile  linkedinUrl |
|   |
| // ✗ Incorrect |
| given\_name    // snake\_case |
| given-name    // kebab-case |
| GivenName     // PascalCase |

| ℹ  Database exception: PostgreSQL column names use snake\_case by convention.    This is layer-appropriate only. API responses, JSON exports, and all    specification field names use camelCase. ORM/serialization layers handle    the translation (e.g., given\_name → givenName). |
| :---- |

## **2.2 Hierarchical Namespace**

Every internal AIX identifier (agent configs, document types, quality passes) follows a strict four-part hierarchy. Each segment narrows scope from broad to specific. Underscores serve exclusively as namespace separators — they are not snake\_case within a segment.

| {component}\_{objectType}\_{identifier}\_{property} |
| :---- |
|   |
| writingAgent\_documentType\_article\_systemPrompt |
| writingAgent\_documentType\_newsletter\_outputRules |
| researchAgent\_qualityPass\_hallucinationSweep\_systemPrompt |
| personaAgent\_voice\_brand |

## **2.3 Schema.org Base \+ AIX Extensions**

For persona data and interchange fields, AIX uses Schema.org canonical property names where they exist. When Schema.org does not have a suitable property — particularly for disambiguation (e.g., business vs. personal phone) or platform-specific fields (e.g., individual social media profiles) — AIX defines extensions, marked 'AIX extension' in the Source column.

| Layer | Example Fields | Rule |
| :---- | :---- | :---- |
| Schema.org canonical | givenName, familyName, jobTitle, legalName, telephone, email, naics, foundingDate | Use as-is, no modification |
| AIX extensions | telephoneMobile, emailBusiness, linkedinUrl, twitterUrl | Only when Schema.org has no equivalent |
| AIX hierarchy keys | writingAgent\_documentType\_article\_systemPrompt | Four-part camelCase hierarchy for agent/config keys |

## **2.4 No Abbreviations**

All segments use full English words. The only exceptions are established industry acronyms registered in the AIX permitted acronym list.

| ✗ Prohibited (abbreviated) | ✓ Required (full word) |
| :---- | :---- |
| liUrl | linkedinUrl |
| sp | systemPrompt |
| or | outputRules |
| docType | documentType |
| qPass | qualityPass |
| ra | researchAgent |

Permitted acronyms: aix, api, url, html, json, uuid, naics

## **2.5 Identifier Names the Format — Properties Define It**

The identifier segment (position 3 in the hierarchy) uses a short, stable handle. The systemPrompt and outputRules properties carry the full behavioral specification. Never encode behavior into the identifier name.

| // ✓ Correct — identifier is a short stable handle |
| :---- |
| writingAgent\_documentType\_newsletter\_systemPrompt |
| writingAgent\_documentType\_newsletter\_outputRules |
|   |
| // ✗ Wrong — behavior encoded into identifier name |
| writingAgent\_documentType\_formattedNewsletterWithCuratedSections\_systemPrompt |

## **2.6 Naming Validation Tests**

Before any new key is ratified, it must pass all six tests. A key that fails any test must be revised.

| Test | Question | Fail Condition |
| :---- | :---- | :---- |
| 1\. Stranger Test | Could a developer with no internal context understand this key from its name alone? | Requires docs or explanation to parse |
| 2\. Hierarchy Test | Does the key follow {component}\_{objectType}\_{identifier}\_{property}? | Missing segment when more specificity exists |
| 3\. Abbreviation Test | Does the key contain any prohibited abbreviated segments? | Any segment not a full word or registered acronym |
| 4\. Case Test | Is every segment in camelCase? | Any snake\_case, kebab-case, or UPPER\_SNAKE within a segment |
| 5\. Redundancy Test | Does any segment restate or mirror an adjacent segment? | e.g. documentType\_document — concept repeated |
| 6\. Conflict Test | Does the key duplicate or closely mirror an existing canonical key? | Duplicate or near-duplicate of a ratified key |

# **3\. Persona Field Registry**

The Persona Field Registry defines every field in the AIX identity vault. Each field is either a Schema.org canonical property (used as-is) or an AIX extension (defined because Schema.org has no equivalent). This registry is the authoritative source for all Persona Vault implementations.

| ℹ  Schema.org canonical fields are used as-is. AIX extension fields are marked 'AIX extension'    in the Source column. The field key is the bare name — e.g., linkedinUrl, not aix:linkedinUrl.    this specification. See Section 1.3 for heritage rationale. |
| :---- |

## **3.1 Essentials**

| AIX Field Key | Source | Description | Example Value |
| :---- | :---- | :---- | :---- |
| givenName | Schema.org | Legal first name | Michael |
| familyName | Schema.org | Legal last name | Tchong |
| preferredName | AIX extension | Preferred name or nickname if different from legal name | Mike |
| jobTitle | Schema.org | Professional title | Founder & CEO |
| worksFor | Schema.org | Primary organization name | Ubertrends LLC |
| industry | AIX extension | Industry or sector (Schema.org has no direct property) | Technology Consulting |
| addressHome | AIX extension | Home or personal display location (e.g., City, State) | Las Vegas, NV |
| timezone | AIX extension | IANA timezone identifier | America/Los\_Angeles |

## **3.2 Professional**

| AIX Field Key | Source | Description |
| :---- | :---- | :---- |
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
| :---- | :---- | :---- |
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
| :---- | :---- | :---- |
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
| :---- |

## **3.5 Social Media**

Schema.org's sameAs property accepts an array of profile URLs but provides no per-platform disambiguation. AIX defines individual extension fields for each platform so agents can reference specific profiles by name without parsing URL arrays.

| AIX Field Key | Source | Description | Format |
| :---- | :---- | :---- | :---- |
| linkedinUrl | AIX extension | LinkedIn profile URL | https://linkedin.com/in/handle |
| twitterUrl | AIX extension | X/Twitter profile URL | https://x.com/handle |
| facebookUrl | AIX extension | Facebook profile or page URL | https://facebook.com/handle |
| instagramUrl | AIX extension | Instagram profile URL | https://instagram.com/handle |
| youtubeUrl | AIX extension | YouTube channel URL | https://youtube.com/@handle |
| tiktokUrl | AIX extension | TikTok profile URL | https://tiktok.com/@handle |
| substackUrl | AIX extension | Substack publication URL | https://name.substack.com |

| ℹ  Naming pattern: {platform}Url  e.g., linkedinUrl, twitterUrl, youtubeUrl    The Url suffix is required. Without it, 'linkedin' is ambiguous — it could be a    handle string, a URL, or a boolean. The Url suffix passes the Stranger Test.    Schema.org equivalent: sameAs (array) — insufficient for per-platform agent use. |
| :---- |

## **3.6 Contact (Email)**

Schema.org has a single email property. AIX extensions disambiguate by context.

| AIX Field Key | Source | Description |
| :---- | :---- | :---- |
| email | Schema.org | Primary email address |
| emailBusiness | AIX extension | Business email address |
| emailPersonal | AIX extension | Personal email address |

## **3.7 Preferences**

| AIX Field Key | Source | Description |
| :---- | :---- | :---- |
| commStyle | AIX extension | Preferred communication style with this agent |
| checkinPreference | AIX extension | How frequently and in what format to receive check-ins |
| interests | AIX extension | Personal and professional interests |
| offLimits | AIX extension | Topics or subjects to avoid in generated content |
| responseFormat | AIX extension | Preferred response format (prose, bullets, structured, etc.) |

## **3.8 Conversational Agent**

Fields describing the conversational agent assigned to this identity vault. The default name is Rosie — a user-configurable display name stored as a value, not a key.

| AIX Field Key | Source | Description |
| :---- | :---- | :---- |
| conversationalAgentName | AIX extension | Display name of the conversational agent. Default: Rosie |
| conversationalAgentVoiceId | AIX extension | TTS voice identifier for the conversational agent (e.g., ElevenLabs voice ID) |
| conversationalAgentSystemPrompt | AIX extension | System prompt defining the conversational agent persona |

# **4\. Component & Agent Registry**

Components are the top-level namespace segment in all AIX hierarchy keys. New components require a spec amendment before use.

| Component Key | Full Name | Description |
| :---- | :---- | :---- |
| researchAgent | Research Agent | Web research, source synthesis, hallucination detection |
| writingAgent | Writing Agent | Content generation, document type management, output formatting |
| personaAgent | Persona Agent | Identity vault, voice, guidelines, guardrails |
| conversationalAgent | Conversational Agent | Conversational assistant (Telegram \+ web) — default name: Rosie |
| orchestra | Orchestra | Platform-level orchestration, agent chaining, workflow config |
| aix | AIX Standard | Specification-level fields (version, schema, envelope) |

# **5\. Reserved Segment Vocabulary**

## **5.1 Object Type Segments (Position 2\)**

| Object Segment | Usage | Example |
| :---- | :---- | :---- |
| documentType | Output format variants for the Writing Agent | writingAgent\_documentType\_article\_systemPrompt |
| qualityPass | Error-check and verification passes (any agent) | researchAgent\_qualityPass\_hallucinationSweep\_systemPrompt |
| config | General agent operational configuration | researchAgent\_config\_default\_systemPrompt |
| voice | Author identity and brand voice data | personaAgent\_voice\_brand |
| output | Generation output schema fields | writingAgent\_output\_draft\_headline |
| input | Generation input schema fields | writingAgent\_input\_contentBrief\_format |
| schema | JSON schema definitions | aix\_schema\_contentAuthoring\_input |
| meta | Metadata (cost, latency, tokens) | aix\_meta\_cost\_amount |

## **5.2 Identifier Segments (Position 3\)**

### **Document Type Identifiers**

| Identifier | Format Description |
| :---- | :---- |
| article | Default long-form editorial article |
| fieldGuide | Solo Trillion Field Guide practitioner reference format |
| meetingBrief | Pre-meeting intelligence brief — condensed, actionable |
| executiveBrief | Executive summary format |
| competitiveAnalysis | Competitive analysis structured comparison report |
| newsletter | Newsletter edition |
| linkedinPost | LinkedIn post |

### **Quality Pass Identifiers**

| Identifier | Description |
| :---- | :---- |
| researchBuild | Initial data gathering and source synthesis pass |
| factCheck | Verify all claims against cited sources |
| hallucinationSweep | Detect and flag fabricated or unsupported content |
| complianceEdit | Apply guardrails, voice rules, and editorial standards |
| qualityScore | Final quality assessment and confidence scoring |
| optimizationPass | Refine output for performance, cost, or length targets |

### **Voice Identifiers (under personaAgent\_voice)**

| Identifier | Description |
| :---- | :---- |
| brand | Author brand voice DNA — tone, vocabulary, sentence rhythm, signature phrases |
| guidelines | Editorial writing structure and style rules |
| guardrails | Hard stops, non-negotiables, topics requiring human review |

## **5.3 Property Segments (Position 4\)**

| Property Suffix | Value Type | Description |
| :---- | :---- | :---- |
| systemPrompt | string | LLM system prompt for the given context |
| outputRules | string | Formatting and structure rules for generated output |
| config | object | Configuration object (when used as terminal node) |
| schema | object | JSON Schema definition |
| template | object | Complete template bundle (prompt \+ rules \+ metadata) |
| label | string | Human-readable display label |
| description | string | Human-readable description |
| version | string | Semantic version string |
| enabled | boolean | Feature flag |

# **6\. Canonical Key Reference**

The master reference for all currently ratified AIX hierarchy keys. Keys marked \* were revised in v2.0. Keys marked \*\* are newly introduced in v2.0.

| Canonical Key | Type | Ver | Description |
| :---- | :---- | :---- | :---- |
| personaAgent\_voice\_brand | string | 1.1 | Author brand voice DNA |
| personaAgent\_voice\_guidelines | string | 1.1 | Editorial writing guidelines |
| personaAgent\_voice\_guardrails | string | 1.1 | Hard stops and non-negotiables |
| writingAgent\_documentType\_article\_systemPrompt | string | 2.0\* | Article system prompt |
| writingAgent\_documentType\_article\_outputRules | string | 2.0\* | Article output rules |
| writingAgent\_documentType\_fieldGuide\_systemPrompt | string | 2.0\* | Field Guide system prompt |
| writingAgent\_documentType\_fieldGuide\_outputRules | string | 2.0\* | Field Guide output rules |
| writingAgent\_documentType\_meetingBrief\_systemPrompt | string | 2.0\*\* | Meeting Brief system prompt |
| writingAgent\_documentType\_meetingBrief\_outputRules | string | 2.0\*\* | Meeting Brief output rules |
| writingAgent\_documentType\_executiveBrief\_systemPrompt | string | 2.0\* | Executive Brief system prompt |
| writingAgent\_documentType\_executiveBrief\_outputRules | string | 2.0\* | Executive Brief output rules |
| writingAgent\_documentType\_competitiveAnalysis\_systemPrompt | string | 2.0\* | Competitive Analysis system prompt |
| writingAgent\_documentType\_competitiveAnalysis\_outputRules | string | 2.0\* | Competitive Analysis output rules |
| writingAgent\_documentType\_newsletter\_systemPrompt | string | 2.0\* | Newsletter system prompt |
| writingAgent\_documentType\_newsletter\_outputRules | string | 2.0\* | Newsletter output rules |
| writingAgent\_documentType\_linkedinPost\_systemPrompt | string | 2.0\* | LinkedIn Post system prompt |
| writingAgent\_documentType\_linkedinPost\_outputRules | string | 2.0\* | LinkedIn Post output rules |
| writingAgent\_config\_default\_systemPrompt | string | 2.0\* | Default Writing Agent system prompt |
| writingAgent\_config\_default\_outputRules | string | 2.0\* | Default output rules |
| researchAgent\_config\_default\_systemPrompt | string | 1.2 | Research Agent default system prompt |
| researchAgent\_qualityPass\_researchBuild\_systemPrompt | string | 2.0\*\* | Pass 1: Research Build prompt |
| researchAgent\_qualityPass\_factCheck\_systemPrompt | string | 2.0\*\* | Pass 2: Fact Check prompt |
| researchAgent\_qualityPass\_hallucinationSweep\_systemPrompt | string | 2.0\*\* | Pass 3: Hallucination Sweep prompt |
| researchAgent\_qualityPass\_complianceEdit\_systemPrompt | string | 2.0\*\* | Pass 4: Compliance Edit prompt |
| researchAgent\_qualityPass\_qualityScore\_systemPrompt | string | 2.0\*\* | Pass 5: Quality Score prompt |

| ℹ  \* Keys marked 2.0\* replace v1.2\* keys. The v1.2 keys used 'template' as the object    segment and 'standardArticle/preMeeting' as identifiers. These are deprecated.    \*\* Keys marked 2.0\*\* are newly introduced in this version.    Deprecated: writingAgent\_template\_\*  researchAgent\_config\_passOne/passTwo/passThree\_\*    Deprecation policy: old keys remain in registry marked \[DEPRECATED\]. Do not use for new work. |
| :---- |

# **7\. AIX Envelope & Task Types**

## **7.1 The AIX Envelope**

Every AIX message is wrapped in an envelope. The aix field carries the specification version. The task, input, output, and meta fields are always present; error is included only on failure.

| { |
| :---- |
|   "aix": "2.0", |
|   "id": "uuid-v4", |
|   "timestamp": "ISO-8601", |
|   "task": { "type": "...", "provider": "...", "model": "..." }, |
|   "input": { }, |
|   "output": { }, |
|   "meta": { "cost": { }, "latency": { }, "tokens": { } }, |
|   "error": { } |
| } |

## **7.2 Task Type Registry**

| Category | Task Type | Description |
| :---- | :---- | :---- |
| Text | text\_generation | Generate text from prompts |
| Text | text\_completion | Complete partial text |
| Text | text\_summarization | Summarize longer text |
| Text | text\_translation | Translate between languages |
| Voice | voice\_synthesis | Generate speech from text |
| Voice | voice\_cloning | Clone a voice from samples |
| Voice | speech\_to\_text | Transcribe audio to text |
| Image | image\_generation | Generate images from prompts |
| Image | image\_editing | Modify existing images |
| Image | image\_upscaling | Increase image resolution |
| Video | video\_generation | Generate video from prompts |
| Video | video\_editing | Modify existing video |
| Video | image\_to\_video | Animate a static image |
| Code | code\_generation | Generate code from description |
| Code | code\_completion | Complete partial code |
| Code | code\_review | Analyze and review code |
| Music | music\_generation | Generate music / audio |
| 3D | model\_generation | Generate 3D models |
| Content | content\_authoring | Generate authored content with voice, guidelines, and guardrails |

## **7.3 Metadata Object**

The metadata object captures operational information enabling comparison, cost tracking, and debugging across providers.

| "meta": { |
| :---- |
|   "cost":    { "amount": 0.018, "currency": "USD", "unit": "per\_request" }, |
|   "latency": { "total\_ms": 8400, "processing\_ms": 8100, "network\_ms": 300 }, |
|   "tokens":  { "input": 4200, "output": 1850, "total": 6050 }, |
|   "model\_version": "claude-sonnet-4-6", |
|   "request\_id": "req\_abc123", |
|   "region": "us-east-1", |
|   "retention": { "input\_stored": false, "output\_stored": true, "retention\_days": 30 } |
| } |

## **7.4 Error Object**

| "error": { |
| :---- |
|   "code": "rate\_limited",       // invalid\_input | rate\_limited | quota\_exceeded | |
|                                  // content\_filtered | model\_unavailable | timeout | |
|                                  // provider\_error | unknown |
|   "message": "Rate limit exceeded for this model.", |
|   "retryable": true, |
|   "retry\_after\_ms": 5000, |
|   "provider\_error": { }         // Raw error object from provider |
| } |

# **8\. Task-Specific Schemas**

## **8.1 content\_authoring**

The content\_authoring task type enables AI-generated content carrying full author identity, editorial voice, structural guidelines, and hard guardrails. Designed for human-in-the-loop publishing workflows where brand consistency and claim integrity are non-negotiable.

### **Input: author object**

| "author": { |
| :---- |
|   "givenName":   "Michael",           // Schema.org canonical (replaces first\_name in v1.1) |
|   "familyName":  "Tchong",            // Schema.org canonical (replaces last\_name in v1.1) |
|   "displayName": "Michael Tchong",// Pen name or byline if different from legal name |
|   "jobTitle":    "Founder, Solo Trillion", |
|   "bio":     "Short author bio for attribution", |
|   "socialProfiles": { |
|     "linkedinUrl":  "https://linkedin.com/in/michaeltchong", |
|     "twitterUrl":   "https://x.com/michaeltchong", |
|     "youtubeUrl":   "https://youtube.com/@michaeltchong", |
|     "url":              "https://ubertrends.com"   // Schema.org — primary website |
|   } |
| } |

### **Input: voice, research, brief**

| "voiceBrand":       "...",  // Author brand voice DNA: tone, vocabulary, sentence rhythm |
| :---- |
| "voiceGuidelines":  "...",  // Structural/editorial rules: formats, sections, citations |
| "voiceGuardrails":  "...",  // Hard stops: topics to avoid, claims needing sourcing |
|   |
| "researchPacket": { |
|   "topic":           "...", |
|   "thesis":          "...", |
|   "narrativeAngle":  "...", |
|   "keyFindings":     \[ { "finding": "...", "sourceUrl": "...", "confidence": 8, "isContrarian": true } \], |
|   "traceabilityTable": \[ { "finding": "...", "sourceUrl": "...", "supportingPassage": "...", "confidence": 8 } \], |
|   "auditSummary":    { "auditPassed": true, "hallucinationsFound": 0 } |
| }, |
|   |
| "contentBrief": { |
|   "format":             "article",  // article | newsletter | linkedinPost | executiveBrief | opEd |
|   "toneOverride":       "contrarian",  // contrarian | analytical | explainer | inspirational |
|   "targetLength":       "standard\_1200",  // short\_600 | standard\_1200 | deepDive\_2000 |
|   "targetPublication":  "Solo Trillion", |
|   "selectedFindings":   \[ \]  // Subset of keyFindings to emphasize; omit to use all |
| } |

### **Output**

| "output": { |
| :---- |
|   "draft": { |
|     "headlineOptions": \[ "...", "...", "..." \],  // 3 variants ranked by contrarian strength |
|     "lede":    "...", |
|     "body":    "...",     // Full article in markdown |
|     "sections": \[ { "heading": "...", "content": "..." } \], |
|     "cta":     "...", |
|     "wordCount": 1247, |
|     "citations": \[ { "text": "...", "url": "...", "title": "..." } \] |
|   }, |
|   "guardrailFlags": \[ { "flag": "...", "location": "...", "recommendation": "..." } \], |
|   "finishReason": "complete"  // complete | guardrailHalt | length | contentFilter |
| } |

## **8.2 voice\_synthesis**

| "input": { |
| :---- |
|   "text": "Welcome to the future of AI interchange standards.", |
|   "voice": "aria",  "language": "en-US",  "speed": 1.0, |
|   "emotion": "neutral",  "outputFormat": "mp3",  "sampleRate": 24000 |
| }, |
| "output": { |
|   "audio": { "url": "...", "format": "mp3", "sampleRate": 24000, |
|              "channels": 1, "durationMs": 3200, "fileSizeBytes": 51200 }, |
|   "wordTimestamps": \[ { "word": "Welcome", "startMs": 0, "endMs": 320 } \] |
| } |

## **8.3 image\_generation**

| "input": { |
| :---- |
|   "prompt": "A futuristic city at sunset, cyberpunk style", |
|   "negativePrompt": "blurry, low quality", |
|   "width": 1024,  "height": 1024, |
|   "style": "photorealistic",  "numOutputs": 1, |
|   "outputFormat": "png",  "seed": 42 |
| }, |
| "output": { |
|   "images": \[ { "url": "...", "format": "png", "width": 1024, "height": 1024, |
|                 "fileSizeBytes": 1048576, "revisedPrompt": "..." } \], |
|   "nsfwDetected": false |
| } |

## **8.4 text\_generation**

| "input": { |
| :---- |
|   "prompt": "...",  "systemPrompt": "...", |
|   "messages": \[ { "role": "user", "content": "..." } \], |
|   "maxTokens": 2000,  "temperature": 1.0,  "stream": false, |
|   "responseFormat": "markdown"  // text | json | markdown |
| }, |
| "output": { |
|   "text": "...", |
|   "finishReason": "complete",  // complete | length | stopSequence | contentFilter |
|   "citations": \[ { "text": "...", "url": "...", "title": "..." } \] |
| } |

## **8.5 video\_generation**

| "input": { |
| :---- |
|   "prompt": "...", |
|   "image": { "url": "..." },    // for image\_to\_video |
|   "durationSeconds": 10,  "aspectRatio": "16:9", |
|   "resolution": "1080p",  "fps": 24, |
|   "motionIntensity": "medium",  "cameraMotion": "static" |
| }, |
| "output": { |
|   "video": { "url": "...", "format": "mp4", "width": 1920, "height": 1080, |
|              "durationMs": 10000, "fps": 24, "fileSizeBytes": 8400000 }, |
|   "thumbnail": { "url": "...", "timestampMs": 500 } |
| } |

## **8.6 code\_generation**

| "input": { |
| :---- |
|   "prompt": "...",  "language": "python", |
|   "context": "...",  "framework": "fastapi", |
|   "includeTests": false,  "includeComments": true, |
|   "styleGuide": "pep8" |
| }, |
| "output": { |
|   "code": "...",  "language": "python",  "explanation": "...", |
|   "files": \[ { "filename": "main.py", "content": "...", "language": "python" } \], |
|   "dependencies": \[ { "name": "fastapi", "version": "^0.110.0" } \] |
| } |

# **9\. Workflows & Chaining**

AIX supports chaining multiple tasks in a workflow. The output of one step becomes the input of the next via ${step\_id.output.field} variable substitution.

| { |
| :---- |
|   "aix": "2.0", |
|   "workflow": { |
|     "id": "uuid", |
|     "name": "Research → Write → Synthesize Voice", |
|     "steps": \[ |
|       { |
|         "id": "step\_research", |
|         "task": { "type": "text\_generation", "provider": "anthropic" }, |
|         "input": { "prompt": "Research AI agents replacing knowledge workers" } |
|       }, |
|       { |
|         "id": "step\_article", |
|         "task": { "type": "content\_authoring", "provider": "anthropic" }, |
|         "input": { |
|           "researchPacket": "${step\_research.output.text}", |
|           "contentBrief": { "format": "article", "toneOverride": "contrarian" } |
|         } |
|       }, |
|       { |
|         "id": "step\_audio", |
|         "task": { "type": "voice\_synthesis", "provider": "elevenlabs" }, |
|         "input": { |
|           "text": "${step\_article.output.draft.lede}", |
|           "voice": "aria" |
|         } |
|       } |
|     \] |
|   } |
| } |

# **10\. Provider Comparison Format**

AIX enables side-by-side comparison of multiple providers for the same task. Standardized metadata makes cost and latency comparisons machine-readable.

| { |
| :---- |
|   "aix": "2.0", |
|   "comparison": { |
|     "task": { "type": "voice\_synthesis" }, |
|     "input": { "text": "Hello, this is a comparison test.", "language": "en-US" }, |
|     "results": \[ |
|       { |
|         "provider": "elevenlabs", "model": "eleven\_multilingual\_v2", |
|         "output": { "audio": { "url": "...", "durationMs": 2100 } }, |
|         "meta":   { "cost": { "amount": 0.002 }, "latency": { "totalMs": 650 } } |
|       }, |
|       { |
|         "provider": "openai", "model": "tts-1-hd", |
|         "output": { "audio": { "url": "...", "durationMs": 2080 } }, |
|         "meta":   { "cost": { "amount": 0.003 }, "latency": { "totalMs": 580 } } |
|       } |
|     \] |
|   } |
| } |

# **11\. Security & Governance**

## **11.1 Security**

AIX does not define authentication — this is provider-specific. AIX messages must never contain API keys, passwords, or personal access tokens. Content filtering signals use a standardized safety object:

| "safety": { |
| :---- |
|   "flagged": true, |
|   "categories": { "violence": false, "sexual": false, "hate": false, |
|                   "selfHarm": false, "dangerous": true }, |
|   "actionTaken": "content\_blocked" |
| } |

## **11.2 Versioning**

| Version Level | Trigger | Example |
| :---- | :---- | :---- |
| Major (X.0) | Breaking changes to envelope, required fields, or task type removal | 1.x → 2.0 |
| Minor (2.X) | New task types, new optional fields, additive features | 2.0 → 2.1 |
| Patch (2.0.X) | Clarifications, documentation fixes, no schema changes | 2.0 → 2.0.1 |

## **11.3 Amendment Process**

The naming convention is a controlled document. To add a new key: (1) propose with full four-part hierarchy and plain-English description; (2) run all six validation tests from Section 2.6; (3) check for conflicts with Section 6; (4) add to candidate keys in next minor version draft; (5) upon ratification, move to canonical reference.

| ℹ  Retroactive changes to ratified keys are prohibited. Deprecation is handled by adding a    replacement key and marking the old key \[DEPRECATED\] in the registry — not by renaming it. |
| :---- |

## **11.4 Governance**

| Body | Responsibility |
| :---- | :---- |
| Technical Steering Committee | Specification changes and ratification |
| Provider Council | Implementation feedback and provider registry |
| Community Contributors | Pull requests, issues, and proposals |

# **12\. Appendix**

## **A. Provider Registry**

| Slug | Company | Task Types |
| :---- | :---- | :---- |
| anthropic | Anthropic | text\_generation, text\_completion, content\_authoring |
| openai | OpenAI | text\_generation, image\_generation, voice\_synthesis, speech\_to\_text, code\_generation |
| google | Google | text\_generation, image\_generation, speech\_to\_text |
| elevenlabs | ElevenLabs | voice\_synthesis, voice\_cloning, speech\_to\_text |
| playht | PlayHT | voice\_synthesis, voice\_cloning |
| runway | Runway | video\_generation, image\_to\_video, image\_editing |
| midjourney | Midjourney | image\_generation |
| stability | Stability AI | image\_generation, image\_editing, image\_upscaling |
| pika | Pika Labs | video\_generation, image\_to\_video |
| replicate | Replicate | (various hosted models) |
| huggingface | Hugging Face | (various hosted models) |

## **B. Language Codes (BCP-47)**

| Code | Language |
| :---- | :---- |
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
| :---- | :---- |
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

1\.  Model Context Protocol (MCP) — https://github.com/modelcontextprotocol

2\.  Eden AI APIs — https://github.com/edenai/edenai-apis

3\.  Schema.org — https://schema.org

4\.  JSON Schema — https://json-schema.org

5\.  BCP-47 Language Tags — https://www.rfc-editor.org/info/bcp47

6\.  CrossARM Data Interchange Standard (2014) — Ubertrends LLC (internal)

# **13\. Changelog**

## **Version 2.0 (Draft) — March 2026**

Major version: merges AIX Interchange Format Specification and AIX Naming Convention Standard into a single authoritative document.

| Area | Change |
| :---- | :---- |
| Document structure | Merged two specs (v1.1 interchange \+ naming convention) into AIX v2.0 |
| Section 3 (NEW) | Persona Field Registry — 40+ fields with Schema.org base or AIX extension designation |
| Section 4 (NEW) | Component & Agent Registry formalized |
| Section 5 (NEW) | Reserved Segment Vocabulary — object types, identifiers, property suffixes |
| Author object | givenName/familyName replace first\_name/last\_name (Schema.org canonical) |
| Social media fields | linkedinUrl, twitterUrl, facebookUrl etc. — Url suffix required, aix: prefix removed from field keys |
| Phone fields | telephoneMobile, telephoneBusiness — Schema.org base word, qualifier suffix disambiguates |
| Email fields | emailBusiness, emailPersonal — disambiguation extensions |
| Address fields | addressHome and addressBusiness replace Schema.org address — same disambiguation pattern as telephone and email |
| Heritage section | CrossARM 2014 lineage documented in Section 1.3 |
| Naming tests | Six validation tests moved to Section 2.6 (previously Appendix D) |
| Deprecated | writingAgent\_template\_\*  |  researchAgent\_config\_passOne/Two/Three\_\* |

## **Version 1.1 (Draft) — February 2026**

| Area | Change |
| :---- | :---- |
| content\_authoring | New task type added (Section 4.6) |
| author object | first\_name, last\_name, display\_name, title, bio, social\_media\_profiles |
| voice fields | voice\_brand, voice\_guidelines, voice\_guardrails |
| research\_packet | Research Agent → Writing Agent chaining input |
| guardrail\_flags | Human review signaling in output |

## **Version 1.0 (Draft) — December 2025**

Initial specification. Core task types: text, voice, image, video, code. Metadata schema with cost/latency tracking. Workflow chaining support. Provider comparison format.

*Apache 2.0 Licensed  ·  aix-standard.org  ·  github.com/aix-standard/aix-spec  ·  aix@aix-standard.org  ·  Copyright 2026 Michael Tchong, Ubertrends LLC*