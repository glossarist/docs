# GLOSSARIST SOTA ROADMAP — Repo Split, 1-Week Agent Sprint, JSI Positioning

Written 2026-11-27. Successor to `GLOSSARIST-SERVER-PLAN.md` (v1, shipped) and `REGISTER-EDITOR-SPEC.md` (editor floor).
Strategic goal: as OIML, present the Ribose-run Glossarist platform to the JSI roundtable (BIPM, CIE, IEC, ISO, DCC) as the **already-live SOTA system** for QI digital and semantic content — and make Ribose the hosting home for every party's vocabulary.

---

## 0. The conviction argument (why this wins)

The JSI community's own agenda is a wall of differently-typed registers with no shared infrastructure:

| Party | Their register | What they lack today |
|---|---|---|
| BIPM | SI Reference Point / SIDF (units, constants; Turtle + PIDs) | No API, no SPARQL, no MCP, no editing procedure surface |
| CIE | e-ILV (lighting vocabulary) | Static PDF/website; **their data is already a Glossarist dataset (cie-eilv)** |
| IEC | IEV / Electropedia, IEC CDD | Nascent Open Data API; CDD closed at cdd.iec.ch (OpenCDD is the open engine) |
| ISO | OBP, ISO/IEC SMART info model, ISO 80000 (isq-smart is ours) | OBP closed/paywalled; no dereferenceable concept IRIs |
| DCC parties | refType register (calibration references) | No register backend at all |
| Part 2 | VC / Recognized Entities / UNTP | **No register engine with verifiable-credential binding exists anywhere** |

Our live production deployment answers their three hands-on assessment questions **today**:

1. *Do the identifiers fulfil FAIR?* → dereferenceable `/id` IRIs with 303 conneg, w3id permalinks (Phase 0), DCAT catalog, PROV-O provenance, honest coverage reporting.
2. *How are physical units represented?* → units/quantities as item classes with IEC 62720 IRDI + QUDT/OM typing (plugin #2 — sibling data already modeled in unitsdb/isq-smart).
3. *How are the vocabularies linked?* → typed cross-register relations, one SPARQL endpoint joining concepts ↔ units ↔ sources across tenants, OWL imports (GLOSS ⊃ SKOS/SKOS-XL/ISO 25964/PROV-O).

Positioning line: **complementary, not competitive** — parties keep data ownership and governance (git repos as SSOT, FERIN procedure); we provide the serving, querying, and procedure layer at edge cost none of their JVM stacks can match. OIML is the reference tenant; every other party gets their own vocabulary live in days, not quarters.

Evidence pack: NLnet/NGI0 funding (breviter, 2021–22, public page), Ribose's ISO/TC 12 & IEC/TC 25 service (isq-smart), production OIML deployment, npm-published provenance-signed packages.

---

## 1. Current state (what's live, 2026-11-27)

- **glossarist-server** `vocab.oimlsmart.org` + workers.dev: `/v1` REST (registers/editions/concepts/designations/relations/lineage/citations/exports), conneg everywhere (JSON-LD default, Turtle, TBX, CSV, JSON), ETag/304/Vary/CORS/Problem-Details, HEAD-correct; `/sparql` (roll-our-own engine, 69% of defined read surface, cost-limited); `/mcp` (6 tools, protocol 2025-06-18); `/id/*` 303s; `/dcat`, `/prov`, `/ontologies` (+ GLOSS OWL served, SHACL publish pending); 45k-file static browser streamed from R2 per build id; 25-case golden suite (incl. French + refusals + 304); 11 registers / 8,177 concepts ingested (OIML VIM/VIML all editions).
- **glossarist-sparql** 0.1.2 + **glossarist-mcp** 0.1.2 on npm (trusted publishing, provenance-signed, release via `v*` tags).
- Ship state: 45k-object upload completing; final golden + evidence closeout on master issue #17 pending.

---

## 2. Repo-by-repo roadmap

### 2.1 glossarist-server — the platform
Now: single-tenant API + engine + site. Target: multi-tenant base register engine (generic items) + editorial workflow + conformance surface.
- [ ] **P0** SHACL publish (`/ontologies/gloss-shacl`, conneg) + build-time validation gate (aggregates pass GLOSS shapes before deploy)
- [ ] **P0** observability (Workers Logs/Traces; Issues→webhook)
- [ ] **P0** canonical IRI (owner decision) + 301s + w3id permalink
- [ ] **P1** `/v1/search?semantic=true` (Vectorize namespaces per language; build-time embeddings); cross-lingual demo
- [ ] **P1** AI Search trial ("ask the register" endpoint over R2)
- [ ] **P2** editorial workflow: FERIN register-spec config in D1; roles + OAuth (Workers OAuth Provider ↔ id.oimlsmart.org); proposal lifecycle = PR round-trip (DOs for review queues, Workflows for vote windows/sweeps, Email Service notifications); proposal API (`/v1/proposals`)
- [ ] **P2** editor UI served by the worker (glossary-scoped; desktop 36-feature parity floor per `REGISTER-EDITOR-SPEC.md` + this roadmap §4)
- [ ] **P3** generic-items core (non-breaking migration from `concepts` table; item-class registry; per-class ontologies)
- [ ] **P3** plugin #2: units/quantities/symbols tenant (unitsdb + isq-smart TermEntry ingest; QUDT/OM + IEC 62720 IRDI typing)
- [ ] **P3** `/conformance`: machine-readable cc/rc self-checks + API-response SHACL shapes as artifacts
- [ ] **P4** federation: DCAT harvest endpoint hardening, cross-tenant SPARQL, per-tenant domains via Workers for Platforms + DO Facets (at SDO #2)
- [ ] **P4** plugin #4 (2027): recognized entities + verifiable credentials (ML-DSA proof suites; VC as source type)

### 2.2 glossarist-sparql — the engine
- [ ] **P1** close read-surface: subqueries, `BIND`, `MINUS`, `EXISTS/NOT EXISTS` → ≥85% of the 26-item denominator
- [ ] **P2** aggregates + GROUP BY/HAVING; property paths stretch
- [ ] **P1** perf bench suite (publish vs. the QLever-set expectations); keep cost limits
- [ ] release via tag (trusted publishing active)

### 2.3 glossarist-mcp — agent surface
- [ ] **P1** protocol bump: support spec 2026-07-28 (MCP SDK v2 semantics); keep stateless design
- [ ] **P1** new tool: `search_concepts_semantic`
- [ ] **P2** OAuth-ready (issuer config); server-card/metadata for registries
- [ ] **P4** WriteGuard-style governance when write tools appear (recognized entities)

### 2.4 glossarist-js — SDK
- [ ] **P0** owner decision: language-tag form (#157) then implement
- [ ] **P1** semantic-search client (Vectorize query + score + provenance types)
- [ ] **P3** emitOutputSet for new item classes (units/quantities projection)

### 2.5 glossarist-ruby — authoring gem/CLI
- [ ] **P2** proposal→PR bridge CLI (open a formatted proposal PR against a dataset repo from editor state)
- [ ] **P3** units/quantities ingest helpers (reuse unitsdb gem patterns)
- [ ] (TBX/GCR exports already here — server reuses)

### 2.6 concept-model — SSOT
- [ ] **P0** SHACL publishing pipeline (shapes → artifact → served + golden-gate)
- [ ] **P3** core/base ontology module split: register/item/edition/status core (item-class-agnostic) vs `gloss:` module (concept classes) — enables plugins without GLOSS pollution
- [ ] **P3** units/quantity model (from unitsdb; TermEntry alignment with sdu-smart)
- [ ] **P4** VC binding model extension (credential as source/attestation type)

### 2.7 concept-browser — the read layer
- [ ] **P1** relation graph view (d3 neighborhood, local/global, SVG/PNG export — demonstrator-validated)
- [ ] **P1** curation queues (worklists from SHACL/validation + missing-translation detection)
- [ ] **P1** edition-diff view (legacy numbering; VIM 1984→2012 once lineage populated)
- [ ] **P2** `<glossarist-term>` embeddable web component (edge-served widget)
- [ ] **P2** tree/facet navigation (precomputed closures in static JSON); multilingual UX to Skosmos-3 bar
- [ ] keep output-tree parity checklist as the gate

### 2.8 glossarist/docs — specs
- [ ] **P0** GLOSS 1.0 spec completion (Metanorma) + SKOS profile via `prof:`
- [ ] **P1** cookbook: semantic-search recipes (live-verified)
- [ ] **P2** whitepaper: "The register platform for the Quality Infrastructure" (SOTA matrix + JSI positioning + evidence)
- [ ] **P3** conformance doc: ISO 19135 Ed 2 cc/rc mapping of the server

### 2.9 glossarist.org — positioning
- [ ] **P1** platform page: live tenants, API/SPARQL/MCP badges, evidence links
- [ ] **P2** "host with us" page for SDOs (cost story, data-ownership model, FERIN conformance)

### 2.10 oimlsmart/vocab — tenant #1
- [ ] **P0** canonical IRI re-ingest + 301s
- [ ] **P1** populate edition supersession/lineage for VIM (turns lineage coverage from honest-0 into demonstrated)
- [ ] **P1** re-build → re-ingest → golden (locks the graph/diff features' data)

### 2.11 Tenants for JSI (new seeds, same pattern as cie-eilv)
- [ ] **P1** CIE ILV tenant (we hold cie-eilv) — live demo URL for Peter/Diana
- [ ] **P1** units demo tenant (unitsdb seed through plugin #2)
- [ ] **P2** IEV subset tenant (iev-data in hand)
- [ ] **P2** SIDF crosswalk demo: BIPM units served with their PIDs preserved (outreach artifact, not a mirror claim)

### 2.12 Reference-only (no work, cited): paneron (architecture precedent), ferin.org (procedure model + schema), opencdd (federation target: CDDAL→JSON-LD bridges), glossarist-desktop (parity floor evidence), e-ILV demonstrator (community UX signals).

---

## 3. The 1-week agent sprint (parallel tracks)

Ten tracks, each agent-assignable, each with acceptance criteria. One human gate per track (merge + deploy).

| Track | Repos | Days 1–2 | Days 3–5 | Days 6–7 | Acceptance |
|---|---|---|---|---|---|
| **A Platform** | glossarist-server | SHACL publish + gate; observability; canonical IRI prep | w3id + 301s; `/conformance` skeleton; federation DCAT hardening | generic-items groundwork behind flags | golden green; shapes served; w3id resolves |
| **B Search** | server, glossarist-js | Vectorize namespaces + build-time embeddings (eng/fra) | all 5 languages; `/v1/search?semantic=true`; AI Search trial | cross-lingual demo + evidence | semantic hits ≥ keyword baseline; cross-lingual demo recorded |
| **C MCP** | glossarist-mcp, server | protocol 2026-07-28 support | `search_concepts_semantic`; server card | OAuth-ready config | MCP golden cases green on new protocol |
| **D Engine** | glossarist-sparql | subqueries + BIND | EXISTS/NOT EXISTS + MINUS | bench suite + release | coverage ≥85%; bench published |
| **E Browser** | concept-browser | graph view | curation queues | edition-diff + `<glossarist-term>` | features demonstrable on production data |
| **F SSOT** | concept-model | SHACL pipeline | core-ontology module split | units/quantity model draft | shapes published; core module importable |
| **G Editor** | server (UI), REGISTER-EDITOR-SPEC | spec refresh (36-floor + FERIN config schema) | CR model + designation/definition editing MVP | review diff + proposal PR round-trip skeleton | CR open→merge on a scratch dataset repo |
| **H Docs** | glossarist/docs, glossarist.org | GLOSS spec + SKOS profile | whitepaper draft | platform page + one-pager | spec builds; whitepaper reviewed by owner |
| **I Tenants** | oimlsmart/vocab, tenant seeds | canonical IRI + lineage populate | CIE ILV tenant live | units demo tenant | three tenant URLs live with API+IRIs |
| **J Positioning** | presentations, docs | JSI one-pager | demo script (hands-on session answers) | BIPM SIDF crosswalk outreach draft | owner-approved drafts |

**Parallelization rules** (per standing process): branch → PR → CI green → merge; explicit paths; no AI attribution; tests before features; blocked → file + move on. Tracks are file-disjoint by repo, so agents don't collide; A and B touch glossarist-server in disjoint routes (`/ontologies/*`, `/conformance` vs `/v1/search`) — sequence their server-side PRs.

**What deliberately does NOT fit week 1** (and is scheduled, not dropped): full editor parity (weeks 3–8 per plan §2.1 P2 — G delivers the CR skeleton only), plugin #4 VC (2027), multi-tenant onboarding (at SDO #2), SPARQL property paths/aggregates (D delivers the read-surface core).

## 4. Incremental strategy — glossary users first, always shippable

**Wave 1 (days 1–2) — pure user value, zero schema change:** semantic search (eng/fra first), graph view, SHACL publication, observability, canonical IRI prep. Current users get a visibly better product every day; nothing breaks; golden suite grows after each merge.

**Wave 2 (days 3–5) — additive surfaces:** MCP v2 protocol, `/conformance`, federation DCAT hardening, curation queues, all-language semantic search, w3id cut-over, tenant #2 (CIE) live. All new routes/namespaces; the v1 API untouched.

**Wave 3 (days 6–7) — generalization groundwork behind flags:** generic-items core scaffold, units tenant seed, editor CR skeleton against a scratch repo. No production exposure until Wave 4.

**Wave 4 (weeks 2+):** editorial workflow full build (FERIN config → DO review queues → PR round-trip → editor parity sprints), plugin #3 federation, multi-tenant runbook, VC spike. Every wave ends with: golden green on both origins + evidence update + (when packages change) tag release.

Rule inherited from v1: **every deploy is gated by the golden suite; every feature lands with tests first; every claim in the JSI material is backed by a live URL.**

## 5. JSI / BIPM playbook (how the conviction actually happens)

1. **Never argue capability — hand over live URLs.** "Your vocabulary, as a tenant: [CIE ILV demo] — API, SPARQL, dereferenceable IRIs, working today." We already hold the data for CIE (cie-eilv), IEC IEV (iev-data), ISO 80000 (isq-smart family), OIML (production).
2. **Answer the agenda's own hands-on questions with our tenants** (identifiers FAIR / units representation / cross-vocabulary links) — the roundtable format literally hands us the demo slot.
3. **The cross-tenant SPARQL moment**: one query joining OIML VIM concepts to units typed with IEC 62720 IRDIs and SIDF crosswalks — this is the sentence-ender for "why host with Ribose".
4. **BIPM specifically**: complementary posture. SIDF keeps its PIDs, governance, and repo (git SSOT); we serve it: conneg, SPARQL, MCP, `/id`, editorial procedure, browser — the layer BIPM has not built. Offer the crosswalk tenant first (Track J), hosting second.
5. **Governance credibility**: FERIN-conformant procedure (schema-checked config), ISO 19135 cc/rc self-checks, honest coverage reporting — the "already SOTA" claim is evidenced, not asserted.
6. **Ops credibility**: edge Workers vs JVM farms; production OIML precedent; NLnet/NGI0 + Ribose ISO/TC 12 & IEC/TC 25 track record.
7. **Artifacts**: whitepaper (Track H), JSI one-pager + demo script (Track J), refreshed WG deck (presentations/ pattern), live tenant URLs.

## 6. Decision ledger (owner gates)

| Decision | Needed by |
|---|---|
| Canonical IRI base (github.io vs www.oimlsmart.org vs vocab.oimlsmart.org) | Wave 1 |
| Base-platform naming (glossarist-server vs paneron-server brand) | before editor/public positioning |
| Language-tag form (glossarist-js#157) | Wave 1 |
| NLnet follow-up submission (deadline 2027-01-03) | Week 2 |
| VC / recognized-entities commitment | Q1 2027 |
| OpenCDD federation MOU | Q1 2027 |
| First external SDO tenant | after tenant demo |
