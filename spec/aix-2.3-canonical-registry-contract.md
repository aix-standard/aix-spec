# AIX 2.3 — Canonical Registry Contract

**Spec file:** `docs/aix-standard/aix-2.3-canonical-registry-contract.md` (internal authoring draft — public `aix-standard.org` publication is a downstream workstream, out of scope here).
**Status:** Ratified 2026-08-02 (Claude AI D3, Michael). Prior state, for the record: Gstack round 2 verified Folds 1-3 and Decisions 1-2 clean; the final pre-ratification fold was an enforcement-honesty fix to §5.1/§7/§8, downgrading the claim that obligations are uniformly mechanically enforceable. Obligations are explicitly split into three enforcement modalities (pre-read gates, mechanical downstream-use on system-controlled data, and advisory-only downstream-use once a value enters an LLM agent's free-text context) — §5.1 and §8's payload-embedding path states plainly that modality-3 enforcement rests on agent compliance plus post-hoc audit, not a technical block. Derivative-value tracking (CD1) and grant-downgrade-via-independent-re-read (CD2) are named as acknowledged limitations, not fixed with new mechanism. **Two D3 ratification notes recorded (§16):** (1) the per-enforcement-point modality declaration is named as a candidate for a future PATCH revision, not required for this ratification; (2) the B3 citation-gap follow-up was ordered same-day and is now resolved — §1.2's own citation.
**Author:** Nimbalyst (2026-07-04, revised same day).
**Anchor:** `main @ a815c2c` — bumped from `939be67`, confirmed merged (clean fast-forward, branch cleaned up) via three independent ref sources (`git rev-parse main`, `git rev-parse origin/main`, `git ls-remote origin`). `a815c2c` is frontend-only (`WritingModule.jsx` fetch-race fix) and touches nothing this spec cites — no re-verification of this spec's own citations was needed for the bump.
**Scope:** an open-standard contract for how canonical registries expose scoped, capability-gated, auditable reads to agents — sibling to AIX 2.2's field-key naming discipline, not a replacement. STO's Persona serves as a **partial** reference implementation (see §13 for exactly what it does and doesn't yet demonstrate).
**Explicit scope exclusions:** does NOT design Persona-Editorial's content itself (Stage-1-track work, see §1.2/§13). Does NOT design any domain namespace's content (Trademark schema, Investment schema, Health schema — separate future workstreams). Does NOT redesign block-first refactor Stage 1 (already ratified — this spec conforms to it, doesn't override it). Does NOT design the `aix-standard.org` public-publication process (downstream after ratification).

---

## §0 — Overview and positioning

AIX 2.3 generalizes a pattern already latent, but never made explicit, in this codebase: a **canonical registry** — a scoped, versioned, single-source-of-truth store that agents read from at dispatch time rather than caching or pinning locally. The motivating failure modes (per-agent pinning drift, per-file library divergence, disputes over which copy of a "canonical" fact is authoritative) are exactly the shape of defect this session's own work surfaced twice already: the press-release dispatch bug (D-29, format-key drift across five enumeration surfaces) and, independently, the block-first architecture refactor's own verification pass (`writing_system_prompts`/`writing_output_rules`'s `prompt_key`/`rules_key` naming, which doesn't mechanically derive from the canonical `format_key` at all — `docs/specs/writing-block-first-architecture-refactor.md` §3.2). AIX 2.3 is not solving a hypothetical; it's naming a defect class this codebase has independently produced at least three times.

**A fourth, freshly-discovered instance, found during this spec's own verification pass:** Persona's backend service module defines a field-key constant, `ESSENTIALS_KEYS` (`solo-trillion-agents/persona/app/persona_agent.py:25-29`), using Schema.org-canonical names — `givenName`, `familyName`, `preferredName`, `jobTitle`, `legalName`, `industry`, `addressHome`, `telephoneMobile`, `timezone`, `emailBusiness` (10 fields). The frontend-facing schema documentation (`docs/persona-field-schema.md:13-14`, dated "Migrated: March 4-5, 2026... aligned") instead lists an "essentials" category using **pre-rename** names — `firstName`, `lastName`, `company` (9 fields) — even though `docs/aix-standard/AIX-Field-Key-Master-List.md:183` explicitly documents the rename this doc failed to pick up: *"March 2026 | firstName/lastName | givenName/familyName | Schema.org canonical per AIX 2.0 Section 3.1."* Confirmed further at the migration layer: `migrations/20260328_aix_field_key_renames.sql:11-19` is the actual rename migration, executing `UPDATE public.persona_fields SET field_key = 'givenName' WHERE field_key = 'firstName'` and five sibling renames — the backend code and this migration agree; the frontend-facing schema doc is the stale artifact. **This is not corrected in this spec** (out of scope — it's Persona's own documentation hygiene, not this contract) but it is cited here deliberately: it is live, first-party evidence that even the reference implementation this spec cites has already drifted internally.

**Reference implementation, precisely scoped (see §13 for full detail):** STO's Persona (`app.solotrillion.ai/persona`, shipping in this repo at `solo-trillion-agents/persona/` + `solo-trillion-frontend/src/components/persona/`) demonstrates the **tier axis** in production — a real, per-field, three-way classification (§3) — but does **not** currently demonstrate the capability axis, namespace/scope concept, grant lifetime, or resource-scoped grants this spec proposes (§2, §4, §5, §8). Those are net-new design surface this spec introduces, using Persona's tier system as the closest existing analog, not as proof the full pattern already ships.

**External precedent:** Airtable's Personal Access Token model — orthogonal capabilities (`data.records:read`, `schema.bases:read`), base-scoped grants, per-token revocation. This is domain knowledge about an external product, not a claim about this codebase; cited as a design precedent for §2/§5, not grep-verified against any file here.

**Sibling standard:** AIX 2.2 (`docs/aix-standard/AIX-Standard-v2.2.md`) is a naming-and-envelope standard — camelCase, hierarchical keys, a six-test validation gate. Verified directly across the entire 1154-line document: it has no access-control, tier, or capability concept for its own registries — `records:read`, "Public tier," and "persistent grant" appear nowhere in it. What it DOES already carry is a real, if undeveloped, capability-string precedent and an explicit forward-pointer to a future addendum that this spec fulfills. §11 has the full account, including a discrepancy this revision pass had to resolve directly.

### §0.1 — Open-standard positioning

Spec authoring targets external adoption from the start: STO as reference implementation, `aix-standard.org` as the eventual public spec site, other agent platforms encouraged to implement independently. Publication of the public-facing version is an explicit downstream workstream, not this spec's scope — this document is the internal-canonical draft that later becomes the `aix-standard.org` publication.

**Gate on publication claims, per Decision 2 (two-tier conformance, §10):** the spec itself may publish now — the contract text doesn't need a running full-conformance system to exist first. But conformance CLAIMS, and specifically "Solo Trillion/Persona is THE reference implementation" language (unqualified), do NOT publish until a full-conformance reference actually exists somewhere in this codebase. Today, Persona satisfies only **incubation-profile** conformance (§10, §13) — the shape is declared correctly, but handoff (§8), audit (§6), lifetime-expiry (§4), and denial paths aren't demonstrated end-to-end anywhere. This is an explicit gate on the `aix-standard.org` publication workstream (itself already out of scope for implementation here) — the gating CONDITION belongs in this spec even though executing the publication doesn't.

**"Universal Markdown" comparison — closed.** The dispatch draws a parallel to "Universal Markdown work" as an analogous open-contract/reference-implementation growth shape. Confirmed real: `docs/canvas-composition-layer-v1.3.md` carries a full section, `# **Universal Markdown via AIX**` (`:564`), naming the exact problem ("Claude, ChatGPT, Gemini, and Perplexity each use different Markdown conventions... The iA Writer developer acknowledged the problem in 2015. Ten years later: no solution," `:568-570`), a canonical-writer-rules table (`:576-589`) implemented at `solo-trillion-relay/app/services/canvas_markdown_writer.py` (`:591`, the same module the block-first architecture spec's revision pass independently verified `render_block_markdown` lives in), and a public validator at `solotrillion.io`, explicitly framed as "the developer adoption trojan horse" (`:598`, Phase D scope, post-Phase-C). Earlier revisions of the same document (`v1.0:190`, `v1.1:359`, `v1.2:529`) carry the same section, and `docs/product-reference-v1.0.md:374,376-378` restates the same framing independently. This is the exact strategic shape §0.1 describes — real, well-documented, and directly analogous — not Michael's uncorroborated characterization.

---

## §1 — Canonical-registry namespace definition

A namespace is:
- A typed canonical store, single source of truth for a domain of ground facts.
- Supabase-backed or equivalent RDBMS-backed, with row-level security.
- Versioned in place (`version` column, `updated_at` timestamp) — no per-agent pinned copies allowed by contract.
- Read at dispatch time, not cached across sessions by agents, by contract.
- Scope-boundaried — agents declare needed namespaces at registration; they cannot silently drift to new ones (see §7).

**STO's initial namespace set, with current status:**

| Namespace | Status |
|---|---|
| Persona-Identity | **Shipping**, tier-gated only today (§13) — not yet namespace/capability-scoped per this contract |
| Persona-Editorial | **Stage 1 addition** (flipped from "proposed" — see §1.2, §12, §13) |
| Legal-Structure | Proposed (Ubertrends LLC entity data) |
| Trademark-Filings | Proposed, for a future Trademark agent |
| Investment-Thesis | Proposed, for financial agents responding to the forthcoming financial-AI product class |
| Health-Record | Proposed, privacy-tier-critical example |
| Airtable-per-base | Proposed — each of Michael's six Airtable databases as a scoped namespace via MCP |
| `format_registry` / `block_schemas` | **Not yet built** — Stage 1 of the block-first refactor (§12). These become AIX 2.3 canonical registries once implemented; this is a forward-looking conformance target for that implementation dispatch, not a retrofit of already-shipped code. |
| Mobile-conducting client namespace access | See §iOS — the mobile client is a CONSUMER profile across all namespaces above, not a namespace itself. |

**Extension pattern for adding a namespace:** registration protocol, RLS setup, agent-scope-declaration re-negotiation (see §7 — adding a namespace to an already-registered agent's scope requires the same ratification as initial registration, not a silent grant).

### §1.1 — Reserved template namespace (fill-in-the-blank addition protocol)

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

**`obligations: []` added per Decision 1** — the fifth orthogonal axis (§5.1). Empty by default; a namespace declares which obligation types it supports/requires the same way it narrows `capabilities`/`tiers`/`lifetimes_supported` in step 4 of the addition protocol below.

Addition protocol: (1) clone the template, (2) rename `namespace_id`, (3) fill `resource_scopes` with the namespace's actual resource-class boundaries, (4) narrow `capabilities`/`tiers`/`lifetimes_supported` from the full defaults if the domain requires tighter constraints (e.g., Health narrowing `lifetimes_supported` to `["session", "single-read"]` only), (5) register per §7.

**This spec does not attempt to prove the four axes (capability, tier, lifetime, resource-scope) are complete for every conceivable namespace, and doesn't need to.** §2's namespace-specific-capability clause (`emergency:read` for Health, e.g.) is the explicit escape valve for whatever a domain needs that the four standard axes don't express — a namespace author adds a namespace-specific capability rather than the contract growing a fifth orthogonal axis per domain. The point of the reserved template is to make ADDING a namespace cheap enough that arguing over axis-completeness stops being the bottleneck — not to guarantee no namespace will ever need the escape valve.

### §1.2 — Persona-Editorial: Stage 1 addition, not a future proposal

**Status flip from the prior draft:** Persona-Editorial moves from "proposed, not designed here" to a Stage 1 addition, migrating whatever document currently governs Rosie's editorial content rules into Supabase namespace form, read by Rosie at dispatch time alongside `format_registry` (§12).

**Citation gap — RESOLVED (B3 follow-up, ordered at ratification, 2026-08-02).** The prior citation gap (below, for the record) flagged this source as unlocatable after an extensive multi-angle search. It has since appeared in the repo and was read in full this pass, not assumed from filename alone: `docs/reference/Canonical_Agent_Content_Rules_v4.md` (291 lines, 18 sections, v4, dated 2026-07-02). Title match is exact — "Canonical Agent Content Rules" (`:1`) against the dispatch's own "the Canonical Agent Content Rules Markdown doc." Scope match is exact — the file's own §1 states it "governs writing, formatting, verification, terminology, article structure, and content handoffs" (`:12`), precisely this section's "editorial content rules" domain. **Confirmed: this is the Persona-Editorial source document.** Stage 1's namespace migration (above) migrates THIS file's content into Supabase form — the gap that blocked acting on it is closed.

**Original citation-gap finding, preserved for the record rather than deleted:** the dispatch names this source as "the Canonical Agent Content Rules Markdown doc." An extensive, multi-angle search of this repo at the time — exact phrase match, synonym search ("editorial rules," "content rules," "agent content standards"), filename patterns (`*rules*.md` repo-wide), and direct reads of `solo-trillion-agents/rosie/Soul.md` and `system_prompt.py` for an equivalent inline concept — found no file matching this description under any name then constructible. The closest adjacent artifact, `docs/specs/rosie-editorial-voice-exhibit.md`, is a different thing entirely (a single verbatim transcript exhibit of one dispatch's editorial framing, not a rules document). The file above did not yet exist in the repo at that point in the session — the gap was genuine, not a missed search.

**The retirement obligation, regardless of where the source document lives:** whatever currently governs Rosie's editorial content rules must be RETIRED in the same motion that lands the Persona-Editorial namespace — not left as parallel surface alongside the new namespace. This is precisely the class of problem D-34's acceptance-criteria checklist (`docs/specs/rosie-roadmap.md` §8, logged this session for the block-first refactor) exists to catch: shipping new positive surface without retiring the old surface it replaces produces more parallel infrastructure, not consolidation. Persona-Editorial's Stage 1 landing should carry its own equivalent one-line acceptance criterion — "the source document is deleted or explicitly marked deprecated-and-empty in the same commit" — not merely "the namespace now also exists."

---

## §2 — Capability axis (orthogonal to tier)

Enumerated capabilities per namespace type:
- `records:read` — read records.
- `records:write` — write records directly (privilege-escalating, most gated).
- `schema:read` — read the shape of records (Airtable's `schema.bases:read` precedent).
- `proposals:write` — submit a proposed change to a review queue that a user or approver-agent ratifies before landing (the pattern that lets a financial agent "rebalance" without a silent direct write).
- `stream:read` — read canonical content as part of an active dispatch's LLM prompt, without persistent caching; short-lived and session-bound implicitly.
- Namespace-specific capabilities are allowed for special domains, but narrower than the prior draft framed them — see the correction below.

Contract requires namespaces to enumerate their supported capabilities explicitly. Agents cannot request capabilities a namespace doesn't declare.

**Corrected per Decision 1: the namespace-specific-capability escape valve is narrower than the prior draft implied.** Capability strings are for WHAT can be done — read/write/schema mechanics only. The prior draft's example, Health carrying `emergency:read` "distinct from `records:read`," smuggled an unstated CONSTRAINT (implicitly: emergency access requires justification, review, or some other guardrail) into a capability string with no declared semantics for what made it different from plain `records:read` beyond its name. That constraint belongs on the new Obligations axis (§5.1), not encoded as an ad hoc capability string. **Re-specified:** Health's emergency-access case is `capability: records:read` + `obligation: post-hoc-audit-required` (or an equivalent obligation type) — the capability stays the same, ordinary `records:read`; the obligation carries the actual constraint (must be justified within N hours or auto-flagged). This keeps capability strings mechanically simple and pushes all constraint-shaped semantics through one typed, enforced axis instead of accumulating implicit meaning inside capability names.

**Grounding:** this is genuinely new design surface, not a codification of an existing pattern. The closest precedent in this codebase is AIX 2.2's `permissionScopes` field (§11) — a capability-shaped string list with no enforcement mechanism defined anywhere in AIX 2.2 itself. §2's enumerated capability list is this spec's proposal for what that undeveloped string list should actually mean and how it should be checked.

---

## §3 — Tier axis (orthogonal to capability)

**This axis has real, verified, shipping prior art in Persona — the one part of this contract with a genuine production reference.**

Verified directly: Persona implements a per-field, three-way tier classification, with **two parallel naming schemes deliberately mapped to each other**:
- DB column values (`permission_tier`, per `docs/persona-field-schema.md:114-123`, and confirmed at the schema layer: `migrations/001_baseline.sql:100-101`'s `CHECK (permission_tier IN ('public', 'trusted', 'private'))`): `public` / `trusted` / `private`.
- UI-facing tier names (`solo-trillion-frontend/src/data/personaData.js:108-112`): `Public` / `Professional` / `Sensitive`.
- Explicit bidirectional mapping (`solo-trillion-frontend/src/components/persona/PersonaModule.jsx:22-33`):
  ```js
  const backendToFrontendTier = { public: 'public', trusted: 'professional', private: 'sensitive' };
  const frontendToBackendTier = { public: 'public', professional: 'trusted', sensitive: 'private' };
  ```

Tier is assigned **per field**, not per category — confirmed at `PersonaModule.jsx:64` (`tier: backendToFrontendTier[field.permission_tier] || field.permission_tier`), and the tier-picker UI (`solo-trillion-frontend/src/components/persona/permissions/TierDropdown.jsx:4`) hard-codes exactly the three values: `const tiers = ['public', 'professional', 'sensitive'];`.

Read-time gating is a simple hierarchy comparison, confirmed at `personaData.js:184-187`:
```js
function shouldIncludeField(fieldTier, filterTier) {
  const hierarchy = { public: 0, professional: 1, sensitive: 2 };
  return hierarchy[fieldTier] <= hierarchy[filterTier];
}
```

**There is no fixed "25 Public / 15 Professional / 12 Sensitive" field count anywhere in this codebase.** Tier is a per-row, user-editable value — "how many fields are currently at each tier" is account state, not a schema constant. **Real, confirmed category totals** (`docs/persona-field-schema.md:11-36`): 50 fields across 7 categories. The "Essentials" category has **9 fields** in this doc's (stale, pre-rename) naming; the backend's `persona_agent.py`'s `ESSENTIALS_KEYS` constant has 10, using current canonical names (§0).

**`ESSENTIALS_KEYS` usage — resolved.** Repo-wide grep for `ESSENTIALS_KEYS` returns exactly one hit: its own definition (`persona_agent.py:25-29`). This is decisive, not ambiguous: its sibling constants in the same file, `SOCIAL_KEYS` and `VOICE_KEYS`, ARE actively consumed elsewhere in the same file (`SOCIAL_KEYS` at `:144`, filtering `AuthorIdentity.socialMediaProfiles`; `VOICE_KEYS` at `:156`, as a Supabase `.in_("field_key", ...)` query filter) — confirming the file's pattern is to define-then-use these roster constants. `ESSENTIALS_KEYS` breaks that pattern: it is defined and never consumed, anywhere in this repo, including via the module's `from persona.app.persona_agent import *` wildcard re-export (`solo-trillion-relay/app/services/persona_service.py:2`, confirmed no downstream file references `ESSENTIALS_KEYS` by name). **It is a currently-unreferenced constant** — likely superseded scaffolding, not live code.

---

## §4 — Grant lifetime (orthogonal to capability and tier)

- **Persistent** — grant survives across sessions until explicitly revoked.
- **Session-only** — revoked at end of the dispatch that requested it.
- **Single-read** — revoked immediately after one successful read.

Namespaces can require minimum lifetime constraints (Health might require session-only-or-less for all its capabilities). Agents cannot request longer lifetimes than a namespace allows.

**Grounding:** Persona demonstrates **none** of these three today. Tier assignment (§3) is a persistent field property with no lifetime concept at all. This entire axis is net-new design surface this spec introduces.

---

## §5 — Resource-scoped grants within namespaces

Grants scoped to specific resources, following the Airtable-PAT precedent (grant the `Toolhacker` workspace, not all workspaces; grant specific bases within a workspace, not all bases — external-product domain knowledge, not grep-verified against this repo). For non-Airtable-shaped namespaces, resource scoping means:
- Per-record-class scoping (a Trademark-Filings grant restricted to USPTO records, not international).
- Per-record scoping (an Investment-Thesis grant restricted to public-market positions, not private-company).
- Per-field-set scoping (a Persona-Identity grant restricted to the Essentials category, not addresses or biographical fields).

Grant is fully specified by the tuple: `(namespace, resource-scope, capability-set, tier-max, lifetime, obligations[])` — extended per Decision 1 (§5.1) with the fifth element; the first five stay exactly as originally specified.

**Grounding:** Persona's existing category grouping (essentials/professional/social/financial/organization/preferences/conversationalAgent, §3) provides UI STRUCTURE a per-field-set grant could map onto — but today, access is gated by TIER alone, not by category. Implementing category-scoped grants is new work.

---

### §5.1 — Obligations axis (fifth orthogonal concept, Decision 1)

**Not a peer to capability/tier/lifetime/resource-scope — those four stay exactly as specified above, unchanged.** Obligations are a structured, typed constraint layer attached to a grant, enforced actively rather than checked as bare string-presence. Where a capability answers "what can be done" and a tier/lifetime/resource-scope answer "how much, how long, over what," an obligation answers "under what continuing condition."

**Initial obligation types, enumerated:**
- `purpose-limitation` — grant valid only for a declared purpose string, checked against namespace-declared allowed purposes.
- `consent-required` — grant inactive until an explicit consent record exists.
- `human-approval-required` — grant inactive until a human (not an approver-agent) explicitly signs off.
- `redact-before-persist` — any downstream persistence of the read value must redact/mask it first.
- `session-only-use` — narrower than the lifetime axis: constrains USE within a session even if the grant's lifetime is technically persistent (e.g., a persistent grant that nonetheless obligates the reading agent not to carry the value's USE past the current session, distinct from whether the GRANT itself survives).
- `post-hoc-audit-required` — access (e.g., an emergency read) must be justified within N hours or gets auto-flagged.
- `no-further-forward` — **added during this pass, not in Decision 1's original enumeration; named explicitly, not smuggled in.** §8's redesign (below) needs a way to mark a value that crossed into a handoff payload as usable-once but not re-forwardable to a third agent — none of the six types above express this, and §8's own text (Fold 3) names it directly ("a value from a Sensitive-tier field carries `no-further-forward` or `redact-before-persist` as appropriate"). Adding it here is the honest resolution: Decision 1's list and Fold 3's own worked example didn't match, and the gap needed a real type, not a workaround.

**Interaction with §2's escape valve:** namespace-specific capabilities (§2) are for mechanics only (read/write/schema); anything constraint-shaped that used to get folded into a bespoke capability string (Health's `emergency:read`, corrected above) now expresses as capability + obligation instead.

**Enforcement, not just declaration:** an obligation attached to a grant must be actively checked at the point(s) its type implies — `consent-required`/`human-approval-required` gate BEFORE the read executes; `redact-before-persist`/`session-only-use`/`no-further-forward` gate at every point the value is about to be written, forwarded, or used past its stated bound; `post-hoc-audit-required` gates on a timer, independent of any single read/write event. This is why obligations are a typed list, not a free-text note — each type has a specific enforcement point, unlike a capability string's implicit, unstated semantics.

**Enforcement-boundary honesty fix, per Gstack/Codex's converged round-2 finding: "mechanically checkable" does not hold uniformly across all obligation types, and the prior paragraph overstated it.** Obligations bundle three distinct enforcement modalities, and every obligation type must declare which one it operates under, not leave it implicit: **(1) pre-read gates** (`consent-required`, `human-approval-required`) — genuinely mechanical, since the system controls whether the read executes at all. **(2) downstream-use constraints on structured data still inside system-controlled storage/transport** (e.g., `redact-before-persist` applied to a value that stays inside a database/API boundary, never exposed to a free-text context) — mechanical, since the system controls the write path. **(3) downstream-use constraints on a value once it has entered an LLM agent's free-text context** (`session-only-use`, `no-further-forward` on a value embedded in a handoff payload/prompt/brief) — **ADVISORY ONLY, not mechanically enforceable by the system once the value is inside model-generated text.** An LLM agent holding a `no-further-forward` value in its context can paraphrase it into a new brief for a third agent with no technical checkpoint catching it. **This is a known, current limitation of LLM-mediated handoffs, not a gap this spec claims to close:** the contract specifies the obligation and its intended constraint; enforcement of modality (3) rests on agent compliance plus post-hoc audit (§6), not a technical block. Stating this plainly is the fix — not inventing new enforcement machinery to make modality (3) mechanical, which isn't achievable for free-text-embedded values in the current state of the art.

**Derivative values (CD1), same modality-3 limitation, extended:** obligations are understood to travel with the underlying FACT a value conveys — including paraphrases, summaries, or extracts that substantively restate it, not just the exact original string. But detecting that a piece of agent-generated text IS such a derivative is itself modality-3 (advisory), potentially aided by post-hoc audit or similarity-review, never a mechanical pattern-match against the original value-instance. "Value-instance" in this section's earlier framing should not be read as implying exact-string tracking — it names what the obligation conceptually travels with, not a claim that the system can technically detect every restatement of it.

---

## §6 — Audit contract

What every canonical read logs: agent identity (agent-id, dispatch-id if applicable), namespace read, resource(s) accessed, capability exercised, tier of data accessed (max tier touched during the read), grant type (persistent/session/single-read), timestamp, result (success, denied-by-tier, denied-by-capability, denied-by-scope, denied-by-lifetime, denied-by-obligation), **and, per Decision 1, obligation-fulfillment state** — was each attached obligation satisfied (consent recorded, human approval obtained, post-hoc justification filed within the required window, redaction confirmed before persistence, etc.), not just whether the grant/access itself succeeded. Retention policy: 90 days minimum. Query interface must expose a readable audit trail to the user.

**Grounding:** Persona has a real UI tab called "Access Log" (`PersonaModule.jsx:19`, rendering `solo-trillion-frontend/src/components/persona/log/AccessLog.jsx`), which is the shape §6 needs. **But it is currently mock data, not a live audit trail** — `AccessLog.jsx` imports `mockAccessLog`, a hardcoded array (`personaData.js:114-123`), not an API call, and its own UI text says so: *"Track all data access requests (Prototype data shown)"* (`AccessLog.jsx:44`).

**A real, if partial, audit primitive DOES already exist, confirmed this pass — not the same thing as the Access Log UI, and previously unverified.** The `conversations` table carries a `persona_fields_accessed TEXT[]` column (`migrations/001_baseline.sql:146`), and it is genuinely live: written by `solo-trillion-relay/app/routers/conversations.py:240,250,319,329` and `solo-trillion-relay/app/routers/agent.py:678,688`, sourced from `personaContextUsed`/`context.get("fields_accessed", [])`. This tracks WHICH persona fields were read per conversation turn — a real, running mechanism — but it does NOT carry capability, tier-touched, grant-type, or result fields, so it satisfies only a fraction of §6's audit contract, and it is architecturally separate from the (mock) Access Log UI, which doesn't currently read from it. **Wiring the Access Log UI to `conversations.persona_fields_accessed` (extended with the missing §6 fields) is a smaller, more concrete lift than building an audit system from nothing** — named explicitly in §13.

---

## §7 — Grant issuance discipline

Explicit-at-agent-registration. No blanket-future-namespace grants. Adding a namespace to an agent's scope after initial registration requires user (or authorized-approver) ratification. Grants are user-revocable per-grant, not just per-agent. Bulk-grant patterns are explicitly discouraged — an agent requesting "all namespaces" is non-compliant with this contract.

**Grant-downgrade (CD2), acknowledged as a limitation, not silently fixed:** if a namespace issues two grants of differing strictness to the same underlying resource, an agent holding the LAXER grant can legitimately access that resource with fewer obligations than a stricter grant would carry for the same fact — including by simply re-reading under its own laxer grant instead of relying on a stricter obligation-bearing copy handed to it by another agent (§8). This is legitimate, not a bypass, since it's a genuinely different, independently-issued grant. **Recommended, not mandated** (a namespace may have a legitimate reason to issue grants of differing strictness for the same resource): namespaces that want to prevent this should declare a minimum obligation FLOOR per tier, applying to ALL grants issued at that tier for that resource — not a per-instance patch. Where a namespace does not declare such a floor, grant-downgrade-via-independent-re-read is a known, accepted possibility under this contract, not a defect this spec claims to prevent by default.

---

## §8 — Cross-agent handoff contract (redesigned this pass, Fold 3 — closes a real security hole)

**The prior draft's first handoff option had a gap Gstack caught: once a canonical-fact value crosses from A's grant-scope into B's handoff payload, nothing stopped B from persisting or reusing it outside any grant of its own.** Pasted into B's brief, the value sits outside the grant/revocation/lifetime system entirely — revoking A's grant, or letting A's session-only lifetime expire, does nothing to a copy B already has embedded in its own context. This redesign fixes it using the Obligations axis (§5.1) rather than inventing a separate taint-tracking mechanism.

**When agent A hands a task to agent B (e.g., Rosie handing a writing dispatch to the Writing Agent), agent B does NOT inherit agent A's scopes — unchanged from the prior draft.** Two mechanisms, now clearly differentiated by tier and by what travels with the value:

**1. Payload-embedding — Public-tier values only.** A reads a value under its own grant and embeds the raw value directly into B's handoff payload. **Restricted to Public-tier data.** The value carries forward, attached to it (not merely at the original read), whatever obligations were attached to A's read grant — e.g., a value read under a `session-only-use` obligation carries that obligation into B's context, not just into A's own use of it. B is REQUIRED to treat an obligation-bearing embedded value as bound by its attached obligations — `session-only-use` bars persistence past B's current session; `redact-before-persist` requires masking before any write B makes; `no-further-forward` (§5.1, added this pass) bars B from embedding that same value into a further handoff to C. **Per §5.1's enforcement-boundary fix: this is squarely modality (3) — advisory, not mechanically enforced.** Once the value is embedded in B's payload/prompt, it has entered B's free-text context, and the system has no technical checkpoint that catches B paraphrasing it forward or persisting it against its obligation. This section specifies the constraint and requires it be declared and carried; it does not claim the system can mechanically block B from violating it — that gap is closed only by agent compliance plus post-hoc audit (§6), consistent with §5.1's honesty fix, not a stronger guarantee unique to the handoff case.

**2. Grant-forwarding transaction — required for Professional and Sensitive-tier values.** A requests a grant-forwarding transaction that gives B its own scoped, time-limited, capability-limited grant derived from A's grants — never broader than A's own grants, and audited as a distinct grant issuance (§6, including its own obligation set). **Resolving the dispatch's own open question explicitly, since it was phrased two ways ("non-Sensitive-tier" vs. "Professional/Sensitive... required"):** this spec adopts the more conservative reading — Professional-tier values require grant-forwarding, not just Sensitive-tier, since Persona's own tier definition for Professional ("released to agents you specifically approve," §3) already implies a per-agent approval step that payload-embedding's looser obligation-carrying model doesn't provide. Only Public-tier values may cross via the lighter payload-embedding path.

**What distinguishes B's independently-granted access from an obligation-bearing embedded value, addressed directly:** if B separately holds its OWN grant to the same underlying namespace/resource (obtained through B's own registration, not through A's handoff), B's use of that independently-read copy is governed by B's OWN grant and its OWN obligations — not by whatever obligation traveled with a DIFFERENT copy A happened to embed in a payload. The obligation attaches to the specific value-instance that crossed via the handoff, not to the underlying canonical fact itself. This means an agent that wants unrestricted use of a fact should read it under its own grant rather than relying on a forwarded copy — payload-embedding is deliberately the more constrained path, not a shortcut around a namespace's own grant discipline.

This is what prevents sub-agent spawn from being implicit privilege escalation, and it is what makes the Rosie-as-conductor model formally auditable — now including what happens to a VALUE after it crosses an agent boundary, not only what happens at the moment of the original read.

**Grounding, re-verified this pass, not assumed unchanged:** the existing Rosie→Writing-Agent handoff mechanism, the `<!--DISPATCH:-->` HTML-comment tag convention (`solo-trillion-relay/app/services/system_prompt.py:892`: `'<!--DISPATCH:{"action":"writing_agent","topic":"...","format_key":"...","source_count":5,"brief":"..."}-->'`; parsed via `DISPATCH_RE = /<!--DISPATCH:(.*?)-->/s` at `solo-trillion-frontend/src/lib/dispatch.js:18`), carries a plain JSON payload (`action`, `topic`, `format_key`, `source_count`, `brief`) with **no scope, grant, or obligation concept anywhere** — confirmed fresh, not restated from the prior draft's citation. §8's redesign names a real, concrete gap in an existing, shipping mechanism: today, ANY value Rosie chooses to embed in `brief` crosses with zero obligation-tracking of any kind, which is exactly the unconstrained-payload-embedding shape this section's redesign closes.

---

## §iOS — Mobile conducting-and-reviewing client profile

**Scope, explicit: a mobile CONDUCTING-AND-REVIEWING client, not a coding client.** This section specifies what an AIX-conformant mobile client exposes, so any implementation (STO's own, or a third party's) builds to one shape rather than renegotiating platform-by-platform. Full mobile CODING access is explicitly out of scope here — see the closing note below and §14 item #4.

**Streaming-artifact receipt.** A mobile client must be able to receive a canonical read's or a dispatch's output as it streams, not only as a completed poll result — the same SSE/streaming discipline the block-first canvas pipeline already uses server-side (`docs/specs/writing-block-first-architecture-refactor.md` §5, §8's progressive-disclosure pattern), extended to a mobile transport.

**Push-notification schema.** A minimal, namespace-agnostic event: draft-complete (or equivalent dispatch-complete signal) → notify. The schema must carry enough for the client to deep-link directly to the artifact (namespace, resource-id, dispatch-id) without a follow-up poll to resolve "what finished."

**Canonical-read-from-mobile: the same dispatch-time read discipline as desktop.** A mobile client is a consumer like any other under §1 — it reads namespaces at dispatch/render time, it does not locally pin or cache canonical facts across sessions. This is a hard constraint carried over from §1's namespace definition, not a mobile-specific exception.

**Background-dispatch state model.** Mobile platforms suspend background execution more aggressively than desktop. A conformant client's dispatch-state model must survive suspension — a dispatch fired, then the app backgrounded, then resumed, must reflect the CURRENT state (including if it completed while backgrounded, per the push-notification schema above) rather than the state at backgrounding time.

**Why this is a profile, not a mobile-specific contract fork:** network variability, background suspension, touch input, and limited concurrent tasks are real, distinct constraints from desktop's assumptions — but they constrain HOW a client implements streaming-receipt/push/read-discipline/background-state, not WHAT those four things mean. Any AIX-conformant client (mobile or otherwise) implements the same four; a mobile client's constraints shape its implementation, not the contract.

**Cross-reference §12/§14:** the block-first architecture refactor's Stage 1 (and specifically the Rosie iPad conducting experience) is the forcing function that makes this section concrete rather than speculative — see §14 item #3 for the timing decision and its rationale.

---

## §9 — RLS enforcement mechanics

Reference-implementation guidance for external implementers: Supabase RLS policies as the mechanism, JWT-scoped tokens carrying agent identity and current grants, database-layer enforcement (not application-layer) so bypass would require compromising the RDBMS. Cross-store federation for canonical stores held in different systems (e.g., Airtable via MCP).

**Core Persona identity table's RLS — resolved, not open.** `persona_fields` — the actual identity/fields table, not the `persona_books` table cited in the prior draft — is created and RLS-enabled at `migrations/001_baseline.sql:91-127`. Full policy set, quoted directly:
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
**All four policies are ownership-scoped only (`auth.uid() = user_id`) — none are tier-aware.** `permission_tier` (§3) is enforced purely as a `CHECK` constraint on the column value (`migrations/001_baseline.sql:100-101`); no RLS policy references it. **This means tier-based access control happens entirely at the application layer today (the frontend's `shouldIncludeField` hierarchy check, §3), not at the database layer** — any query that bypasses the application's filtering logic (a raw Supabase client call, for instance) would return ALL of a user's own fields regardless of tier, since RLS only gates by row ownership. §9's reference-implementation guidance (database-layer, not application-layer, enforcement) is therefore a genuine gap relative to Persona's current posture, not something Persona already demonstrates — named explicitly in §13.

---

## §10 — Compliance checklist for canonical-registry implementations — two named tiers, per Decision 2

**Replacing the prior single conformance bar with two explicitly-named tiers, per Michael's ruling (Decision 2 — Gstack recommended a `/plan-ceo-review` pass for this exact question; Michael ruled directly this session, serving that function, so no separate CEO-review pass is pending):**

**Incubation-profile conformance** — the shape is declared correctly: namespace registration; capability enumeration; tier system (three tiers or a documented equivalent) correctly declared. This is a lower, achievable bar, and it's what Persona satisfies today (§13).

**Full conformance** — every incubation-profile item PLUS all remaining §10 checklist items demonstrated end-to-end in a running system, not just declared in a spec: grant lifetime support (all three — persistent, session, single-read) actually enforced; resource-scoped grants actually gating access; an audit log with §6's required fields (including obligation-fulfillment state, Decision 1) actually populated by real reads, not mock data; explicit-grant-issuance discipline actually preventing blanket-future grants; cross-agent handoff contract (§8) actually implemented, including the Obligations-based redesign; obligation declaration and enforcement (Decision 1, §5.1) actually checked at runtime for any namespace that declares obligations; and denial paths (denied-by-tier, denied-by-capability, denied-by-scope, denied-by-lifetime, denied-by-obligation) actually exercised and logged, not just enumerated as possible states.

**Note, per verification:** by this two-tier split, Persona today satisfies **incubation-profile conformance only** — its tier system is correctly declared (three tiers, per-field), but even that one demonstrated axis has a real gap (tier enforcement is application-layer, not database-layer, §9). Every other line item is either partially present (Access Log UI shape without a live backend, §6; a partial field-access log in `conversations.persona_fields_accessed`) or entirely absent (capability enumeration, grant lifetime, resource scoping, handoff contract, obligations). Persona does not satisfy full conformance — §13 has the itemized delta, now explicitly framed against this two-tier split rather than a single pass/fail bar.

---

## §11 — Relationship to AIX 2.2

**The forward-pointer's EXISTENCE stands, confirmed real, not a fabrication — but its COMMIT ATTRIBUTION in the prior draft was wrong, and this pass corrects it with the rigor that was missing before.** Re-grepped directly against the live repo file, `docs/aix-standard/AIX-Standard-v2.2.md` (dot, confirmed as the only AIX-2.2-named file in `docs/` — no underscore-named variant exists in this repo):
```
$ grep -in "tier" docs/aix-standard/AIX-Standard-v2.2.md
490:> **UMA 3.0 UI Primitive Contract Meta-Framework:** §4.8 defines module-level manifests. For UI primitive
contracts — the shared components consumed by modules — see `docs/aix/uma-3.0-contract-meta-framework.md`.
The meta-framework extends §4.8 to primitive-level AIX contracts and ratifies new canonical keys:
`umaModule_primitive_*` (primitive contract metadata), `umaModule_telemetry_usageEvent_*` (telemetry event
payloads), `umaModule_billing_transactionEvent_*` (billing event payloads), `umaModule_tier_capability_*`
(tier-gated capability flags). These keys are addenda to v2.2; they will be formally registered in a future
minor version.
```
Identical result for `grep -in "capabilit"` — same single line. The referenced file, `docs/aix/uma-3.0-contract-meta-framework.md`, genuinely exists.

**The commit attribution: `git log -p -L 488,492:docs/aix-standard/AIX-Standard-v2.2.md` (the full line-range history, not a single-snapshot blame) shows exactly two commits ever touching these lines** — `2faf9ce` (2026-05-24, the file's own creation, adding only the preceding "Slug-to-camelCase rule" paragraph, no UMA 3.0 text yet) and `e2d65fb` (2026-06-02, `docs: add UMA 3.0 spec foundation — meta-framework, implementation guide, AIX cross-ref`), whose diff adds exactly the forward-pointer paragraph quoted above. **`e2d65fb` is the true, sole origin — not `c0694a3`, which the prior draft cited.** Independently confirmed why `c0694a3` was mistaken for the origin: `git show c0694a3` is a real commit (`docs(rename): PR-1 product rename sweep "Solo Trillion Orchestra" -> "Solo Trillion"`, Jun 17 2026) that touches `AIX-Standard-v2.2.md`, but its diff shows four unrelated one-word swaps at lines 254, 288, 430, and 1066 — nowhere near line 490. A commit that happens to touch the same FILE, at a LATER date, was mistaken for the commit that introduced this specific PARAGRAPH; a plain single-point `git blame`-style check (rather than the full line-range history `git log -L` traces) would surface whichever commit most recently touched line 490's general vicinity, which can differ from whichever commit actually authored that content if an unrelated later edit lands nearby. **This line has now been mis-cited in some form across three passes this session — the pointer's existence has been correct throughout; only the commit attribution was wrong, and this pass is the first to verify it against the full line-range history rather than a single-point check.** No fourth unverified pass: the fix above is reproducible by anyone re-running `git log -p -L 488,492:docs/aix-standard/AIX-Standard-v2.2.md` against this exact repo state.

**The underlying §11 argument was never in question, and isn't affected by which commit gets cited** — everything else in this section's original framing stands unchanged:

- Grepped across the entire live document: `records:read`, "Public" (as an access label), and "persistent grant" appear **zero times**. AIX 2.2 has no access-control, tiering, or capability-scoping concept whatsoever for any of its registries.
- **§2.6 "Naming Validation Tests"** (`AIX-Standard-v2.2.md:149-160`) — the six-test gate (Stranger, Hierarchy, Abbreviation, Case, Redundancy, Conflict).
- **§4.8 "Module Manifest Standard"** (`:443-491`) — `{moduleType}Module_{moduleId}_{property}` key hierarchy (`:451-457`), four ratified module types (`publishingModule`, `researchModule`, `writingModule`, `personaModule`, `:460-463`), and a manifest schema example (`:473-486`) carrying a `permissionScopes` field (`:483`: `["persona:read", "writing:generate", "publishing:send"]`) with no enforcement semantics defined anywhere else in the document.
- **The forward-pointer at line 490** (quoted above, confirmed real, now correctly attributed to `e2d65fb`): AIX 2.3 is the future minor version's substantive content this paragraph names — this spec fulfills a promissory note AIX 2.2 already made, rather than inventing capability/tier concepts unprompted. This framing does not depend solely on the forward-pointer either: even considered alone, the undeveloped `permissionScopes` field would be sufficient grounding for "AIX 2.3 specifies what an existing-but-undeveloped AIX 2.2 field means" — the forward-pointer strengthens this section, but was never load-bearing for its core argument.

**Migration path for AIX 2.2-only implementations:** naming discipline (§2 of AIX 2.2) and access-control discipline (this spec) are orthogonal concerns — a field's camelCase, hierarchical, six-test-validated key name never changes when its namespace gains tier/capability/grant metadata. Migration is additive, never a naming rewrite.

---

## §12 — Relationship to block-first architecture refactor (`ae73aca`)

Block-first's `format_registry` and `block_schemas` (`docs/specs/writing-block-first-architecture-refactor.md` §3, ratified via D3) ARE AIX 2.3 canonical registries under this contract, per §1's namespace table. Stage 1 implementation (not yet started) should be authored with AIX 2.3 conformance in mind from the start.

**Persona-Editorial is now IN Stage 1 scope** (§1.2, flipped from a future proposal) — authored on the separate Stage-1 implementation track, not by this spec, but tracked here as a real cross-spec dependency.

**The Rosie iPad conducting experience is compressed INTO Stage 1** (§14 item #3) — voice input as a first-class input mode, push-on-completion, background dispatch survival, and a streamed-artifact review mode, all per §iOS's contract. Bastille Day (July 14) is the forcing function for this timing decision.

**Concrete, scheduled follow-up:** once this spec ratifies, `docs/specs/writing-block-first-architecture-refactor.md` §3 should be updated to explicitly cite AIX 2.3 as its governing contract, and the Stage 1 implementation authoring pass should verify `format_registry`'s table shape satisfies at minimum §1's namespace-definition requirements, ideally the full §10 compliance checklist.

**Fold 2 (this pass): checked whether this section, §iOS, or §14 need re-pointing from block-first to `docs/specs/document-workflow-architecture.md` (authored this session, draft status confirmed — untracked, `**Status:** draft — spec-ready for Gstack /plan-eng-review`, not yet ratified). They do not.** Grepped this entire spec for "universal canvas"/"single enumeration"/"single registry" framing — no such citation exists anywhere in this file's current text; §12 cites block-first for the `format_registry`/`block_schemas` TABLES themselves (still block-first's own architecture, correctly cited), and §14/§iOS cite block-first for Stage 1/3 TIMING (the Rosie iPad conducting experience, the canvas palette UX build) — not for a "universal-canvas correction" framing this file never actually claimed to source from anywhere. **Worth stating for completeness, not as a correction:** `document-workflow-architecture.md`'s own reconciliation section explicitly defers to block-first for registry/storage/canvas design (its items 1-3), reserving its own scope for the generation-output-contract and frontend fetch-race concerns (its items 4-5) — meaning even if this section HAD cited block-first for universal-canvas, that citation would still be correct per the newer document's own stated stance, not something to redirect. **§1.2 (Persona-Editorial's Stage-1-addition status) is unaffected either way** — it was never a candidate for re-pointing, since it's governed by block-first's own implementation sequencing regardless of document-workflow-architecture.md's existence, exactly as instructed.

---

## §13 — Relationship to Persona: incubation-profile reference, not "the reference implementation" unqualified (Decision 2)

**Explicit marking, per Decision 2's two-tier conformance split (§10): Persona is an incubation-profile reference only.** Prior drafts of this spec (and this section's own prior title) referred to Persona as "the reference implementation" without qualification — that framing is corrected here. Persona correctly declares the SHAPE of one axis (tier) but does not demonstrate handoff, audit, lifetime-expiry, or denial paths end-to-end anywhere in a running system. Calling it "the reference implementation" unqualified would overstate what it actually proves, and would be exactly the kind of claim §0.1's new publication gate exists to prevent from reaching `aix-standard.org`.

**What Persona covers today, verified as real and shipping:**
- A genuine per-field, three-tier classification (`public`/`trusted`/`private` DB, `Public`/`Professional`/`Sensitive` UI), with an explicit bidirectional mapping and a working hierarchy-based read-gate (§3) — but enforced at the APPLICATION layer only; the core `persona_fields` table's RLS is ownership-scoped, not tier-aware (§9, resolved this pass).
- A "Data Interchange" tab (`DataInterchange.jsx`) that is real but export-only: tier-simulator, single-category selection, AIX JSON / Canonical Markdown output, Copy/Download. Import is a separate tab entirely.
- An "Access Log" tab with the right UI shape but mock data (§6) — not yet wired to any live source, including the genuinely-live `conversations.persona_fields_accessed` column that already tracks per-turn field access and would be the natural backend to wire it to.

**What Persona does NOT yet have, needed for full AIX 2.3 conformance:**
- **No namespace/capability axis at all.** Access today is tier-gated only — any consumer clearing a tier gets every field at or below it; no per-agent scoped grant mechanism exists.
- **No grant lifetime concept.** Tier assignment is persistent-only; no session-scoped or single-read grant exists.
- **No resource-scoped grants beyond the tier hierarchy.** The category structure is a natural fit but doesn't currently gate access.
- **No live audit backend wired to the Access Log UI** — though `conversations.persona_fields_accessed` (§6) is a real, running, partial primitive that makes this a smaller lift than building from nothing.
- **No cross-agent handoff protocol** (§8).
- **Database-layer tier enforcement is absent** — RLS is ownership-scoped only (§9); a raw client bypassing the application's tier filter would see all of a user's own fields regardless of tier. This is a genuine security-posture gap for any consumer that reads `persona_fields` outside the sanctioned frontend/API path, not just a conformance nicety.
- **Persona-Editorial's actual source document is unlocated** (§1.2) — needs Michael's pointer before its namespace-migration work can proceed on the Stage 1 track.

Persona-extension to close this delta is its own future workstream, not this spec's scope — but the delta itself is the concrete input to whoever scopes that workstream.

---

## §14 — Answers from Michael's structured design-question pass

**Note on count:** the dispatch's header states seven ratified decisions; only six are enumerated across its body (§iOS=#2, §1.1=#6, §1.2=#5, and the three below, #1/#3/#4). This section records exactly the six given — it does not invent a seventh to force the count to match, and flags the discrepancy rather than silently padding it.

- **#1 — iPad viewport.** A Stage 3 canvas-UX constraint (palette placement, drag-affordance density, block-overlay layout, inline-edit keyboard behavior) — not a Stage 1 concern. Deferred to when Stage 3's canvas palette/drag-to-reshape UX (`docs/specs/writing-block-first-architecture-refactor.md` §9 Stage 3) is actually built.
- **#2 — mobile conducting-and-reviewing client profile.** See §iOS. Not a coding client (#4).
- **#3 — the Rosie iPad experience is compressed INTO block-first Stage 1.** Voice input first-class, push-on-completion, background dispatch, streamed-artifact review mode — all per §iOS's contract. **Bastille Day (July 14) is the forcing function** for this timing decision; cross-ref §12.
- **#4 — CC/coding-agent mobile access is shelved, not designed here.** Rationale: few Solo Trillion users code. Future form, when coding agents integrate on mobile: a split chat/coding-artifact view with CC live — recorded as future scope, explicitly out of this contract.
- **#5 — Persona-Editorial status flip.** See §1.2, §12, §13.
- **#6 — reserved TBD template namespace.** See §1.1.

---

## §15 — W2 verifications (G1-G6 discipline)

**Every citation in this spec resolves to a live file:line confirmation or a named, current [BLOCKED] — no deferred/unverified markers remain, per the explicit correction this revision pass was dispatched to apply.** The prior draft's "outage interrupted me, named as open" framing for two items is retracted as a pattern; both were closed directly in this pass, and a genuine repo-access outage encountered mid-revision was treated as pause-and-retry (confirmed recovered within one message, then verification continued), not grounds to ship a citation as permanently open.

- **A1/§11 forward-pointer** — RESOLVED as real, not fabricated, WITH a corrected commit attribution (fold 1, this pass). Reproduced directly against the live file: `grep -in "tier" docs/aix-standard/AIX-Standard-v2.2.md` and `grep -in "capabilit" docs/aix-standard/AIX-Standard-v2.2.md` both return exactly line 490, quoted verbatim in §11. The prior pass's commit citation (`c0694a3`) was wrong — verified this time via the FULL line-range history, `git log -p -L 488,492:docs/aix-standard/AIX-Standard-v2.2.md`, which shows exactly two commits ever touching these lines: `2faf9ce` (file creation, no UMA 3.0 text) and `e2d65fb` (2026-06-02, adds the forward-pointer paragraph verbatim). `c0694a3` is a real but unrelated later commit (a Jun 17 product-rename sweep) that happens to touch the same FILE at four other, unrelated lines (254, 288, 430, 1066) — confirmed via `git show c0694a3`'s own diff, none of which are near line 490. **`e2d65fb` is the correct, sole attribution.** Confirmed the referenced file `docs/aix/uma-3.0-contract-meta-framework.md` exists. Confirmed the live repo has no `AIX-Standard-v2_2.md` (underscore) file anywhere — only `AIX-Standard-v2.2.md` (dot) — resolving the filename-discrepancy question an earlier correction request raised.
- **A2** — not applicable; conditional on A1 finding the pointer didn't exist, which it does. §11 retains the promissory-note framing, with the added, more resilient note that the argument doesn't depend on the forward-pointer alone.
- **A3/Universal Markdown** — RESOLVED as real. `docs/canvas-composition-layer-v1.3.md:564-608` (and matching sections in v1.0/v1.1/v1.2, plus independent corroboration in `docs/product-reference-v1.0.md:374-378`) — full citations in §0.1.
- **A4/core Persona RLS** — RESOLVED, decisively. `persona_fields` table + all four RLS policies read in full from `migrations/001_baseline.sql:91-127` (quoted verbatim in §9). All ownership-scoped (`auth.uid() = user_id`); none reference `permission_tier`. Tier enforcement confirmed application-layer only.
- **A5/`ESSENTIALS_KEYS`** — RESOLVED. Confirmed unreferenced anywhere in the repo (one grep hit, its own definition), in contrast to sibling constants `SOCIAL_KEYS`/`VOICE_KEYS` in the same file, which ARE consumed (`persona_agent.py:144,156`). Full reasoning in §3.
- **New finding, not previously flagged: `conversations.persona_fields_accessed`** (§6) — a real, live, partial audit primitive, confirmed via direct reads of `migrations/001_baseline.sql:146` (column definition) and its four write-sites in `conversations.py`/`agent.py`. Surfaced during this pass's RLS/audit re-verification, not searched for specifically by the dispatch — closes part of what would otherwise have been an open §6/§13 gap.
- **B3/Persona-Editorial source document** — **RESOLVED at ratification, 2026-08-02, see §1.2's own citation.** At the time of this pass, genuinely could not be located. Retry attempts, all against the live repo: exact-phrase grep (`"canonical agent content rules"`, `"content.rules"`), synonym grep (`"editorial rules"`, `"agent content standards"`), filename search (`*rules*.md` repo-wide, zero hits), direct reads of `solo-trillion-agents/rosie/Soul.md` and `system_prompt.py` for an equivalent inline concept. This is reported as `[BLOCKED — no file matching "Canonical Agent Content Rules Markdown doc" found under any searched name, phrase, or filename pattern in this repo; the closest adjacent artifact, docs/specs/rosie-editorial-voice-exhibit.md, is a different document (a single dispatch's editorial-voice transcript, not a rules document)]` — a genuine, current inaccessibility (the file does not appear to exist here under any name tried), not a paused/deferred verification.
- **All other citations** (block-first spec cross-references, AIX 2.2's §2.6/§4.8/§3.10 structure, Airtable PAT model as external domain knowledge) — carried forward unchanged from the prior pass; re-confirmed HEAD unchanged (`939be67`) before citing the block-first anchor again.
- **Fold 2 (§12/§iOS/§14 cross-references)** — checked, not blindly re-pointed. Grepped this entire file for "universal canvas"/"single enumeration"/"single registry" framing: zero matches outside the unrelated §0.1 "Universal Markdown" comparison. §12/§14 cite block-first for the `format_registry`/`block_schemas` tables and for Stage 1/3 timing — both still accurate, neither is the "universal-canvas correction" citation the dispatch described. `docs/specs/document-workflow-architecture.md`'s status confirmed live (`grep "^\*\*Status"` on that file: "draft — spec-ready for Gstack /plan-eng-review") — cited as draft, not settled, per the dispatch's own instruction. §1.2 confirmed untouched, exactly as instructed.
- **Fold 3 (§8 handoff redesign)** — the DISPATCH mechanism re-verified fresh, not assumed unchanged from the prior citation: `solo-trillion-relay/app/services/system_prompt.py:892` (payload shape) and `solo-trillion-frontend/src/lib/dispatch.js:18` (`DISPATCH_RE` parser) both read directly this pass. Confirmed still a plain JSON payload with zero scope/grant/obligation concept — the security hole's premise holds.
- **Decision 1 (Obligations axis)** — no code to verify (this is new contract design, not a description of existing code); the one verification-relevant addition, `no-further-forward` as a 7th obligation type not in the dispatch's own enumerated six, is named explicitly as an addition in §5.1 rather than silently folded into the list Decision 1 specified.
- **Decision 2 (incubation-profile framing)** — grounded in citations already verified earlier in this document (§9's RLS finding, §6's mock-Access-Log finding, §13's itemized delta) — no new code verification needed, this decision reframes existing, already-cited facts under a new two-tier structure rather than introducing new claims.
- **Known, out-of-scope-for-this-pass staleness, flagged not fixed — CLOSED at ratification, 2026-08-02, see §1.2.** §1.2/§16's "Canonical Agent Content Rules Markdown doc — could not be located" finding predates a file, `docs/reference/Canonical_Agent_Content_Rules_v4.md`, that has since appeared in this repo (observed in git status during adjacent work this session, not independently re-verified for content in THIS pass, since it's outside folds 1-3/decisions 1-2's scope). Left as-is per this dispatch's precise scope at the time — flagged here so it doesn't get silently treated as still-accurate by a future reader, and named as a discrete follow-up rather than fixed inline. **That follow-up (B3) has now run — content confirmed, citation resolved in §1.2.**
- **Final fold (enforcement-honesty fix, this pass)** — no new code citations needed; this is a claim-scope correction against the spec's own already-authored §5.1/§8 text, not a description of external code. Confirmed `git rev-parse HEAD` directly before finalizing the anchor note: HEAD is `a815c2c`, verified via `git log -1` to be `fix(writing): decouple legacy-history fetch from drafts effect (Test 1 regression)` — matching the WritingModule.jsx fix flagged as claimed-but-unlanded during the document-workflow-architecture pass; it has since actually landed. Citation held at `939be67` per explicit instruction, not silently bumped.

---

## §16 — Report to Michael

**Fold 1 — the citation error this pass fixes was genuinely mine, not the reviewing side's, and it's worth being straight about that rather than reflexively defending prior work.** The forward-pointer's EXISTENCE at `docs/aix-standard/AIX-Standard-v2.2.md:490` has been correct across every pass — it's real, verbatim, and was never fabricated. But the COMMIT ATTRIBUTION I gave it in the prior revision (`c0694a3`) was wrong, caught this time by Gstack/Codex's fresh re-verification. Re-running the check against the FULL line-range history (`git log -p -L`, not a single-point blame) shows the true, sole origin is `e2d65fb` (Jun 2) — `c0694a3` is a real but unrelated Jun 17 commit that happens to touch the same file at four other lines, nowhere near 490. This is a different kind of correction than the one two passes ago, where the party checking my work was the one working from a stale, out-of-repo copy — this time the error was mine, introduced while I was correctly defending the pointer's existence but citing its origin from a less rigorous check than the argument deserved. Named plainly per the explicit instruction not to let this line take a fourth unverified pass: the fix above is fully reproducible against this exact repo state by anyone who runs the same `git log -L` command.

**Six of the seven "ratified decisions" were enumerated in the dispatch body; I authored exactly those six** (§iOS, the reserved-template namespace, Persona-Editorial's status flip, and the three §14 answers) rather than inventing a seventh to make the count match — flagged in §14 rather than silently smoothed over.

**Historical record — this citation did not resolve at the time this paragraph was written; it has since been closed (B3 follow-up, 2026-08-02, see §1.2).** "The Canonical Agent Content Rules Markdown doc" that Persona-Editorial (§1.2) is supposed to migrate could not be found anywhere in this repo after an extensive multi-angle search at that point. It has since appeared — `docs/reference/Canonical_Agent_Content_Rules_v4.md` — confirmed by direct read to be the source document (§1.2 has the full match reasoning). Everything else about §1.2 (the status flip, the retirement obligation, the D-34 cross-reference) already stood regardless of where the source document turned out to be; it now has its citation too.

**One new, useful finding this pass surfaced that wasn't in the original dispatch's ask:** `conversations.persona_fields_accessed` (§6) is a real, live, currently-written column already tracking per-turn persona-field access — a genuine, if partial, audit primitive that makes wiring the (currently mock) Access Log UI to something real a smaller lift than building an audit system from scratch.

**Also resolved, decisively:** the core `persona_fields` table's RLS is ownership-scoped only, with zero tier-awareness at the database layer — meaning tier enforcement today is entirely an application-layer filter, not a database-layer guarantee. This is a real security-posture gap worth Persona's own team knowing about independent of this spec's ratification, since it means any code path that queries `persona_fields` directly (bypassing the frontend's `shouldIncludeField` check) would see every field a user has, regardless of tier.

**Fold 2 — checked precisely, nothing re-pointed.** §12/§iOS/§14's block-first citations turned out to already be accurate for what they actually claim (registry tables, Stage 1/3 timing) — none of them currently make the "universal-canvas correction" claim the fold's premise described, so there was nothing to redirect. Named this precisely rather than force a re-point that the file's own text didn't need, and confirmed `document-workflow-architecture.md`'s draft (not settled) status before citing its existence at all. §1.2 left untouched, exactly as instructed — it was never a candidate for re-pointing.

**Fold 3 — the security hole is real and now closed.** The prior draft's payload-embedding option let a canonical-fact value cross agent boundaries with nothing constraining B's later use of it — grant revocation or lifetime expiry on A's side did nothing to a copy already sitting in B's context. Fixed using the new Obligations axis (Decision 1) rather than a separate taint mechanism: values crossing via payload-embedding now carry their originating grant's obligations forward, Public-tier only is permitted to use this lighter path, and Professional/Sensitive-tier values are routed through the existing grant-forwarding transaction instead. One judgment call worth flagging: the dispatch phrased the tier-restriction question two ways ("non-Sensitive-tier" vs. "Professional/Sensitive... required") — I resolved it toward the more conservative reading (Professional requires grant-forwarding too, not just Sensitive), defended by Persona's own Professional-tier definition already implying a per-agent approval step. Also added a 7th obligation type, `no-further-forward`, not in Decision 1's original six — Fold 3's own worked example named it directly and none of the original six covered "usable once, not re-forwardable," so the gap needed a real type rather than a workaround.

**Decisions 1 and 2 — authored directly, as instructed, not treated as open.** The Obligations axis lands as §5.1 (no cascading renumber — the dispatch's own naming already specified the sub-section number), with downstream updates to §1.1's template, §2's Health example, §5's grant tuple, §6's audit field, and §10's checklist. The incubation-profile/full-conformance split replaces §10's single bar with two named tiers, gates `aix-standard.org` publication claims in §0.1, and re-titles §13 to mark Persona as incubation-profile only rather than "the reference implementation" unqualified.

**One thing outside this pass's scope, flagged rather than silently left stale — that follow-up dispatch has now run (B3, 2026-08-02).** §1.2/§16's earlier finding that "the Canonical Agent Content Rules Markdown doc" couldn't be located anywhere in this repo predated a file — `docs/reference/Canonical_Agent_Content_Rules_v4.md` — that has since appeared. It was read in full and confirmed as the source document; §1.2 carries the resolved citation and match reasoning. Named here rather than deleted, so the record shows the gap was real, flagged, and closed — not silently never-having-existed.

**Ratification recorded, 2026-08-02 (Claude AI D3, Michael) — docs-only finalization, no further design changes.** Two notes from D3 at ratification, recorded here rather than left implicit: (1) the enforcement-modality split landed in the final pre-ratification fold (§5.1/§7/§8, three modalities: pre-read gates, mechanical system-controlled downstream-use, advisory-only free-text-context downstream-use) is coarser than it could be — D3 named a per-enforcement-point modality declaration (stating which modality applies at each individual gate/read-path, not just per obligation-type category) as a real improvement, but explicitly a PATCH-revision candidate, not a blocker to this ratification. (2) The B3 citation-gap follow-up (§1.2) was ordered as a same-day action item at ratification, executed above — `docs/reference/Canonical_Agent_Content_Rules_v4.md` confirmed as the Persona-Editorial source document, closing the last open item this spec carried into ratification.

**Final fold, this pass — an honesty fix, not a redesign, exactly as scoped.** Gstack/Codex converged on the same gap: §5.1/§8 implied obligations are uniformly mechanically enforceable, which isn't true once a value crosses into an LLM agent's free-text context. Fixed by naming three enforcement modalities explicitly — pre-read gates and system-controlled downstream-use are genuinely mechanical; downstream-use on a value already embedded in a prompt/brief is advisory only, resting on agent compliance plus post-hoc audit. Derivative values (CD1) and grant-downgrade-via-independent-re-read (CD2) are named as acknowledged, current limitations — I did not invent new obligation types or enforcement machinery to paper over either, per the explicit discipline instruction not to. This downgrades the spec's claim to match what's actually achievable in an LLM-mediated system, which is a more honest and more defensible contract than the version that implied uniform mechanical enforcement.

**Anchor note — confirmed, bumped.** `a815c2c` is now confirmed merged, not just observed via `git rev-parse HEAD` in my own working copy: CC verified via three independent ref sources (`git rev-parse main`, `git rev-parse origin/main`, `git ls-remote origin`) — clean fast-forward, branch cleaned up. The anchor is bumped to `a815c2c` accordingly; no re-verification of this spec's own citations was needed, since `a815c2c` is frontend-only (`WritingModule.jsx`) and touches nothing this spec cites.

**Anchor at authoring time:** `main @ a815c2c`. No commit — presented for Claude AI D3 ratification. Gstack's round-2 verdict already states the spec clears on this fold alone.
