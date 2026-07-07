# Mini-PRD: Litigation research + drafting core — extending LQ.AI's fiduciary-grade substrate into litigation work product

> **Status:** Draft — proposed. Forks in §Decisions are **open**; none has been resolved with the maintainer. Acceptance requires an explicit amendment to [PRD §1.6 Out of Scope](../PRD.md#16-out-of-scope-v1), which currently excludes "e-discovery or litigation-specific workflows" from v1 — see Fork 1. Inventory below is grounded in a repo pass of 2026-07-06 (PRD, ADRs 0014–0021, `skills/`, HONEST-STATE); it has **not** yet had the four-front codebase reconciliation the [fiduciary-grade mini-PRD](fiduciary-grade-agentic-legal-work.md) received — treat file-level claims as doc-verified, not code-verified.
> **Effort:** L–XL (phased; Phase 1 is mostly skill authoring + one gate extension; Phase 2 adds a drafting-surface workstream).
> **Contributor profile:** Practicing litigators for the skill corpus (attestation required per [skills/CONTRIBUTING.md](../../skills/CONTRIBUTING.md) — agents do not attest); backend engineers in `api/` (citation engine, autonomous layer) for the filing gate and session template; one `web/` (SvelteKit) contributor for gate surfacing.
> **Mentor:** Maintainer (via PR review); security review per CODEOWNERS wherever the gate or session work touches `api/app/autonomous/**`, the citation/audit surface, or `gateway/**`.

## What this is

Extend LQ.AI from in-house advisory work (contracts, policies, regulatory matters) into the **research and drafting core of litigation practice**: finding and reading authority, verifying that authority is real and still good law, drafting motions and briefs grounded in that authority, and gating the finished draft behind a citation check before it leaves the system.

This is deliberately the *narrow* litigation wedge. It is **not** e-discovery, docket monitoring, deadline calculation, or matter-management workflow — those remain out of scope (see §Out of scope). The claim this mini-PRD makes is that the narrow wedge is nearly free: the fiduciary-grade program ([mini-PRD](fiduciary-grade-agentic-legal-work.md), ADRs [0018](../adr/0018-citation-ledger-and-fiduciary-grade-output.md)–[0021](../adr/0021-content-source-registry-and-free-source-expansion.md)) already builds, for in-house research work, almost exactly the substrate litigation drafting needs. What litigation adds is (a) a domain-specific skill corpus, (b) a stricter output gate, and (c) an honest reckoning with the fact that the stakes are higher — a fabricated citation in a brief is a sanctions event, not an embarrassing memo (the *Mata v. Avianca* pattern the PRD already names in [DE-279](../PRD.md#de-279--case-citation-validation-bluebook-resolution-via-courtlistener)'s context).

## Why the §1.6 exclusion is worth revisiting (and how narrowly)

[PRD §1.6](../PRD.md#16-out-of-scope-v1) excludes "e-discovery or litigation-specific workflows," reasoning that "litigation tools are a separate product category." That reasoning holds for the *workflow* category — review platforms, docketing, calendaring. It has quietly stopped holding for the *analytical* category, because the project has been building litigation-adjacent capability since PR6:

- The shipped case-law research provider and `skills/case-law-research/` skill carry a `litigation` tag today ([PRD §3.6](../PRD.md#36-research)).
- The [skill-authoring guide](../skill-authoring-guide.md) reserves a `litigation` domain tag ("rarely; out of scope for v1").
- [ADR 0019](../adr/0019-transparent-validity-treatment-layer.md)'s derived validity/treatment layer answers a question ("is this still good law?") whose highest-stakes consumer is a litigator, not an in-house advisor.
- [DE-109](../PRD.md#de-109--litigation-hold-support) (litigation hold) already acknowledges litigation-posture deployments.

The proposal: amend §1.6 to exclude "e-discovery and litigation *workflow* tooling (review platforms, docketing, deadline management)" while carving in "litigation research and drafting work product." The analytical-layer boundary the PRD draws against Streamline AI / Checkbox for in-house work is the same boundary drawn here against Relativity / Everlaw / docketing systems for litigation work.

## What LQ.AI already has vs. what's net-new

| Litigation-core element | LQ.AI today | Net-new work |
|---|---|---|
| **Find authority** | **Have.** Governed CourtListener tool-loop (search / get cluster / read opinion / find-in-case / verify citations) shipped in PR6 under ADRs [0014](../adr/0014-gateway-egress-boundary-for-tool-providers.md)/[0015](../adr/0015-governed-tool-calling-model.md); `skills/case-law-research/` ships as a starter skill. | Jurisdiction-aware source expansion rides on [ADR 0021](../adr/0021-content-source-registry-and-free-source-expansion.md)'s content-source registry (GovInfo, Federal Register, etc.) — no litigation-specific egress work. |
| **Verify authority exists / is quoted accurately** | **Partial.** Citation Engine cascade ([PRD §3.3](../PRD.md#33-citation-engine-exact-quote)) verifies KB-document quotes; [ADR 0018](../adr/0018-citation-ledger-and-fiduciary-grade-output.md) D2/D3 extends character-verification to external caselaw via materialized sources; [DE-279](../PRD.md#de-279--case-citation-validation-bluebook-resolution-via-courtlistener) covers Bluebook-citation resolution. | Nothing architectural. WS-L3 consumes these; if DE-279 has not landed when WS-L3 starts, it becomes a hard dependency to schedule, not to design. |
| **Verify authority is still good law** | **Committed, not shipped.** [ADR 0019](../adr/0019-transparent-validity-treatment-layer.md) (Accepted 2026-06-26): treatment derived from the citation graph + LLM-judge, labeled "derived, not editorial," confidence-scored, traceable. | Nothing architectural. The litigation filing gate (WS-L3) makes treatment a *blocking* signal rather than an informational one — a policy layer on top of ADR 0019, not a change to it. |
| **Statement-vs-opinion accuracy** | **Missing, tracked.** [DE-280](../PRD.md#de-280--case-content-accuracy-statement-vs-judicial-opinion) — hardest of the three citation surfaces; explicitly deferred. | None proposed here. WS-L3 marks statements DE-280 would cover as `unverified-content` in the gate report rather than pretending to verify them. Honest labeling over overclaiming. |
| **Research → draft in one governed flow** | **Committed, not shipped.** [ADR 0020](../adr/0020-governed-agentic-legal-matter-sessions.md) (Accepted 2026-06-28): matter-scoped agentic sessions spanning research and drafting, producing ledger-backed output through the fiduciary gate. | WS-L5: a litigation session *template* (research → issue outline → section drafts → gate). Template-level work on ADR 0020's machinery. |
| **Drafting skills** | **Missing.** The 10 starter skills are advisory/transactional. No motion, brief, or written-discovery skill exists in `skills/`. | **WS-L1 — the center of gravity of this mini-PRD.** See below. |
| **Filing-ready output** | **Partial.** DOCX export exists for tabular review; Word add-in plumbing shipped M3 with the in-Word feature surface deferred ([DE-287](../PRD.md#9-deferred-enhancements-and-identified-future-work)). | WS-L4: court-format document assets (caption blocks, line numbering conventions) as *skill reference files*, not an export service — see Fork 3. |
| **Preservation posture** | **Tracked.** [DE-109](../PRD.md#de-109--litigation-hold-support) litigation hold; DE-108 retention. | None. Named here because litigation deployments will raise DE-109's priority; this mini-PRD does not absorb it. |

The shape of the finding mirrors the fiduciary-grade reconciliation: **the platform substrate is built or committed; the litigation gap is skills, one gate policy, and one session template.**

## Decisions needed (open forks — for the maintainer)

Per the project's stop-on-architectural-questions rule, these are put as forks with recommendations, **not** decided:

1. **PRD §1.6 amendment — now or on Phase-1 acceptance?** *Recommend:* amend when this mini-PRD is accepted, with the narrow carve-in wording above. Amending before acceptance overclaims; never amending leaves the PRD contradicting the shipped `litigation`-tagged skill.
2. **Jurisdiction scoping for the skill corpus.** (a) Federal-generic first (FRCP-anchored skills, no local rules); (b) one exemplar state vertical first, proving the jurisdiction-layering pattern (base skill + jurisdiction reference files) end-to-end; (c) both in parallel. *Recommend:* (a) then (b) — federal-generic has the widest contributor and user base and no local-rule maintenance burden; the exemplar state vertical follows as the community-contribution template. The skill format already supports layering via reference files and the `jurisdiction` frontmatter field used by `case-law-research`.
3. **Where court formatting lives.** (a) Reference files inside each drafting skill; (b) a shared formatting service/endpoint. *Recommend:* (a). Formatting requirements are jurisdiction-and-court–specific work product — exactly what the transparency posture says should be inspectable, forkable files, not code. A service is a heavier artifact with no current consumer besides these skills.
4. **How hard is the filing gate?** (a) Blocking: export/delivery refuses while any citation is `not-found` or carries negative treatment, admin-overridable; (b) advisory: prominent report, never blocks. *Recommend:* (a) for skills that declare `lq_ai: { filing_gate: required }`, (b) as the default elsewhere. The attorney remains the responsible filer either way; the gate is a seatbelt, not a substitute for review. **(ADR needed if (a) — it creates a new refusal path analogous to tier-floor refusal and should be specified with the same rigor.)**

## What we'd ship — five workstreams

### WS-L1 — Litigation drafting skill corpus (`skills/`)

The core deliverable. An initial set on the order of: answer/responsive pleading, motion to dismiss (and opposition), summary-judgment motion (and opposition), written discovery (propound + respond), meet-and-confer letter, demand letter. Each skill:

- Follows the existing SKILL.md format and [authoring guide](../skill-authoring-guide.md); declares `tool_usage: [courtlistener]` where it researches, `use_organization_profile: true`, and a `minimum_inference_tier` appropriate to privileged litigation content.
- Ships with at least one worked example and goes through the full attestation path in [skills/CONTRIBUTING.md](../../skills/CONTRIBUTING.md) — practicing-attorney attestation and dual review. **This is the rate-limiting step and should be planned as such**; litigation skills carry more attestation weight than a contract snapshot, because errors surface in front of a tribunal.
- Structures output so every authority-bearing statement is citation-carrying — the skill text itself instructs the model to emit citations the engine can verify, and to mark anything it cannot ground. Skills that produce filings declare the filing-gate frontmatter per Fork 4.

### WS-L2 — Jurisdiction layering pattern (docs + skill conventions)

A documented convention, not a system: base drafting skill + per-jurisdiction reference files (court rules excerpts the contributor is licensed to include, formatting requirements, local-practice notes) + `jurisdiction` frontmatter. Lands as a section in the skill-authoring guide plus one exemplar. Depends on Fork 2.

### WS-L3 — The filing gate (`api/` citation/ledger surface)

A pre-export pass over a finished draft that produces a per-citation report — resolved / quote-verified / paraphrase-verified / not-found / negative-treatment / unverified-content — assembled entirely from existing or committed machinery: the citation cascade (§3.3), materialized external sources (ADR 0018 D2/D3), DE-279 resolution, and the ADR 0019 treatment layer. Net-new: the aggregation, the report artifact (a ledger view scoped to one draft), the gate policy per Fork 4, and the web surface. No new verification science; DE-280-class claims are labeled, not judged.

### WS-L4 — Filing-format output (skill assets + existing export paths)

Per Fork 3 recommendation: formatting reference files within skills, rendered through the existing DOCX export path. The Word add-in in-document surface stays deferred ([DE-287](../PRD.md#9-deferred-enhancements-and-identified-future-work)); when the community picks DE-287 up, litigation drafts inherit it without work here.

### WS-L5 — Litigation matter-session template (ADR 0020 machinery)

A session template for [ADR 0020](../adr/0020-governed-agentic-legal-matter-sessions.md)'s governed matter sessions: intake (posture, jurisdiction, deadline context supplied by the user — the system does not compute deadlines) → research phase (governed CourtListener/source-registry retrieval) → outline → section drafting with inline ledger-backed citations → filing-gate pass → delivery with the gate report attached. Runs under the existing R5→R6→R4 brakes and phase-gated tool grants; no new governance surface.

## Sequencing and dependencies

```
Phase 1 (independent of unshipped substrate):
  WS-L1 federal-generic skill set  ──┐
  WS-L2 jurisdiction convention docs ─┴─ ship on today's main

Phase 2 (depends on fiduciary-grade Phase 1/2 landing):
  WS-L3 filing gate      ← ADR 0018 ledger + DE-279; treatment signal ← ADR 0019
  WS-L5 session template ← ADR 0020

Phase 3:
  WS-L4 beyond skill-asset formatting; exemplar state vertical (Fork 2b)
```

Phase 1 has zero code dependencies — the drafting skills work in ordinary chat with the shipped research tools today, at advisory fidelity. The gate and session template are what upgrade "useful draft" to "fiduciary-grade litigation work product," and they land in the order the fiduciary-grade program lands.

## Out of scope

- **E-discovery** — review, production, privilege logging at scale. Different product category; the §1.6 exclusion stands.
- **Docket monitoring, deadline computation, calendaring.** Deadline math is jurisdiction-rule-driven and malpractice-adjacent; a wrong computed deadline is worse than none. Users supply deadline context; the system never asserts it.
- **PACER/RECAP or commercial-connector integrations as core deliverables.** Operators may connect these via the BYO MCP path (ADR 0014/0015); nothing here depends on them.
- **DE-280 case-content accuracy.** Consumed if and when it lands; not built here.
- **DE-109 litigation hold.** Adjacent; separately tracked.

## Risks and honest caveats

- **Attestation bottleneck.** The corpus is only as good as the attorneys willing to attest it. If contribution stalls, Phase 1 shrinks to two or three exemplar skills rather than shipping unattested drafts — the skill process is not relaxed for schedule.
- **Sanctions-grade stakes.** No gate makes a draft filing-safe; Rule 11-type responsibility stays with the attorney. Every gate report and skill output must say so. The conservative-posture rule applies to the product copy, not just the docs.
- **Jurisdiction sprawl.** Fifty-state coverage is a community outcome, not a roadmap item. The deliverable is the *pattern* (WS-L2) plus exemplars.
- **Treatment layer limits.** ADR 0019 signals are derived and confidence-scored, not editorial. A litigation gate consuming them must surface confidence and the underlying citing cases, never a bare red/green.

## Acceptance criteria (Phase 1)

- PRD §1.6 amended per Fork 1; this mini-PRD linked from PRD §9 with DE numbers assigned to WS-L3/WS-L4/WS-L5.
- At least three litigation drafting skills merged through the full attestation path, each with a worked example, loading cleanly in the skill library with provenance pills.
- Jurisdiction-layering convention documented in the skill-authoring guide with one exemplar.
- Fork 4 resolved; if gate-blocking is chosen, the ADR exists before WS-L3 implementation starts.

## Pointers

- Substrate this builds on: [fiduciary-grade mini-PRD](fiduciary-grade-agentic-legal-work.md); ADRs [0014](../adr/0014-gateway-egress-boundary-for-tool-providers.md), [0015](../adr/0015-governed-tool-calling-model.md), [0016](../adr/0016-transparency-and-governance-invariants.md), [0018](../adr/0018-citation-ledger-and-fiduciary-grade-output.md), [0019](../adr/0019-transparent-validity-treatment-layer.md), [0020](../adr/0020-governed-agentic-legal-matter-sessions.md), [0021](../adr/0021-content-source-registry-and-free-source-expansion.md).
- Citation surfaces taxonomy: [PRD §3.3](../PRD.md#33-citation-engine-exact-quote), [DE-279](../PRD.md#de-279--case-citation-validation-bluebook-resolution-via-courtlistener), [DE-280](../PRD.md#de-280--case-content-accuracy-statement-vs-judicial-opinion), [docs/citation-engine.md](../citation-engine.md).
- Research surface: [PRD §3.6](../PRD.md#36-research); reference skill [`skills/case-law-research/`](../../skills/case-law-research/SKILL.md).
- Skill process: [skills/CONTRIBUTING.md](../../skills/CONTRIBUTING.md), [docs/skill-authoring-guide.md](../skill-authoring-guide.md).

---

*Draft maintained in `docs/proposals/`. On acceptance: resolve the four forks, record them in a §Decisions block per house convention, assign DE numbers, and amend PRD §1.6 in the same PR.*
