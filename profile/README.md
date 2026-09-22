# Platform Factory

**A reference implementation of an opinionated developer platform — built in
public as a pre-registered experiment, and meant to be copied.**

GitOps control plane on GKE / Argo CD / Crossplane / Kyverno, a repo topology
where the repo boundary *is* the approval boundary, and a knowledge-as-code
layer. Every repo in this org is Apache-2.0. Fork the org shape, change one
project id, and you have the floor of a software factory of your own.

Designed and written by **Ronak Patel**
([ORCID 0009-0002-3753-2193](https://orcid.org/0009-0002-3753-2193),
thecloudgeek LLC). Cite it by DOI:
[10.5281/zenodo.22848132](https://doi.org/10.5281/zenodo.22848132).

> **Where the build is (2026-09-21):** M1 *Spine* closed 2026-08-28 and M2
> *Paved road* closed 2026-09-17, claims graded with evidence in the build
> log. M3 *Approval boundary* is next. Of 25 registered claims — 22
> pre-registered on 2026-07-31, three added since, each dated — eight carry a
> grade: five HELD (two of them scoped to the surface built so far), three
> ADJUSTED (design changed; superseding ADR linked), none WRONG. The other
> seventeen are untested, including one M2 stretch claim (C-08) that was not
> attempted. Nothing here is abandoned; some of it is not yet started, on
> purpose.

## Start here

1. **Why — the design seed:**
   [`platform-factory-concept`](https://github.com/platform-factory/platform-factory-concept).
   The pattern in five sentences, sixteen ADRs, the research digests, and the
   [claims register](https://github.com/platform-factory/platform-factory-concept/blob/main/docs/build-log/claims-register.md)
   every milestone is graded against. Read that README first; it is the spec
   this org implements.
2. **The floor —
   [`platform-bootstrap`](https://github.com/platform-factory/platform-bootstrap):**
   Terraform layer 0, four independently-applied layers split by lifecycle.
   The README explains the mental model before the runbook; `scripts/cycle.sh`
   is the runbook, scripted and timed.
3. **The control plane —
   [`platform-config`](https://github.com/platform-factory/platform-config):**
   the Argo CD app-of-apps Terraform hands the cluster to. Crossplane, Kyverno,
   and the `System` / database Compositions all arrive here as PRs.
4. **The paved road, end to end —
   [`systems`](https://github.com/platform-factory/systems) →
   [`svc-hello`](https://github.com/platform-factory/svc-hello):** one YAML
   file onboards a tenant; the service declares its database next to its
   Deployment and the platform materializes it.
5. **The evidence —
   [`docs/build-log/`](https://github.com/platform-factory/platform-factory-concept/tree/main/docs/build-log):**
   one entry per milestone, numbers included, misses included.

## The pattern in five sentences

1. **Git is the source of truth for everything; Kubernetes is the control
   plane.** Terraform's only job is to bootstrap what the control plane can't
   create yet. (The build found it recurs at one declared boundary — new
   platform capabilities need identity and reachability from layer 0; that is
   C-01, graded ADJUSTED, ADR-0016 §7.)
2. **The repo boundary is the approval boundary.** Everything a team ships
   lives in the team's repo by default; something moves to a central repo only
   when its *approver* is someone other than the team shipping it.
3. **Developers declare intent next to their Deployment** — Crossplane XRs for
   databases, buckets, identities. Compositions materialize the cloud
   resources; admission policy, not human review, enforces the guardrails.
4. **The edge is config, not tickets.** All routes and DNS live in one repo
   where folder structure routes the review — security approves `external/` —
   and a schema field drives what gets materialized.
5. **Knowledge is treated exactly like code** — born via PR, owned via
   CODEOWNERS, flagged stale by CI, regression-tested by a question bank.

Three tiers of change, three approval speeds: a service repo's `k8s/` (team
approves; schema + admission enforce) → `edge-config` (security via
CODEOWNERS) → `systems` (platform approves).

## The repos

The topology is the point — seven implementation repos, not one, because the
approval model lives in *which repo a change touches* — plus the design seed
and a second tenant added at M2 so the one-file onboarding test had something
to onboard.

| Repo | Role | Approver | Milestone | Status (2026-09-21) |
|---|---|---|---|---|
| [`platform-factory-concept`](https://github.com/platform-factory/platform-factory-concept) | Design seed: pattern docs, 16 ADRs, claims register, graded build logs, research | — | all | **Current through the M2 close.** Moved into the org from `thecloudgeek/platform-factory` on 2026-09-19 with its dated history. |
| [`platform-bootstrap`](https://github.com/platform-factory/platform-bootstrap) | Terraform layer 0: project, VPC, GKE, workload identity, Argo CD — "Terraform's last job" | platform | **M1** | **Built and live.** Layers 0–1 running since 2026-08-06; layers 2–3 rebuilt between sessions by `cycle.sh`, six cycles so far (five measured rebuilds and a closing teardown). Timings in `scripts/cycle-results.tsv`. |
| [`platform-config`](https://github.com/platform-factory/platform-config) | Argo CD root (app-of-apps): the shared platform layer | platform | **M1–M2** → M3 | **Built:** Crossplane + GCP provider family, Kyverno, the `System` and database XRDs/Compositions, images through Artifact Registry. ESO, external-dns, Gateway arrive in M3. |
| [`systems`](https://github.com/platform-factory/systems) | One YAML per tenant — the `System` XR | platform | **M2** | **Built.** Two tenants live (`svc-hello`, `svc-ledger`); merging the file *is* onboarding (C-05 HELD). |
| [`svc-hello`](https://github.com/platform-factory/svc-hello) | Canonical service: Go app + `k8s/` (Deployment + database claim) | the team | **M2** | **Built.** Three routes, one table, Cloud SQL via IAM identity — the application holds no password (ADR-0013). One manual `sql.User` step per database remains while a provider fix is pending (C-07 ADJUSTED, ADR-0016 §4). |
| [`svc-ledger`](https://github.com/platform-factory/svc-ledger) | Second tenant, a placeholder owned by a different team | the team | **M2** | **Built** — exists so C-05/C-06 had a second team to test against. |
| [`template-service`](https://github.com/platform-factory/template-service) | Create-repo-from-template golden path | platform | M2 → later | Scaffold. M2 did not build it; recorded as such. |
| [`edge-config`](https://github.com/platform-factory/edge-config) | All routes, DNS, WAF | security approves `external/` | M3 | Scaffold — but CODEOWNERS already routes `/external/` to `@platform-factory/security`. The approval boundary exists in file form from commit one. |
| [`platform-knowledge`](https://github.com/platform-factory/platform-knowledge) | Knowledge as code: `skills/`, `adr/`, `standards/`, `questions/` | light review | M4 | Scaffold. |

## Milestones and what each one proves

| # | Name | Builds | Proves | Claims | State |
|---|---|---|---|---|---|
| M1 | Spine | Org repos, `platform-bootstrap`, GKE, Argo CD app-of-apps from `platform-config` | Terraform's-last-job + GitOps control plane; rebuild is cheap | C-01..C-04, C-23 | **Closed 2026-08-28.** C-02, C-23 HELD; C-04 HELD for the M1 surface, at the cost of five sync-wave workarounds; C-01, C-03 deferred to M2 (their tests couldn't run yet — recorded as a register scheduling error, not papered over). |
| M2 | Paved road | `System` XR, `svc-hello` + database claim, Compositions + Kyverno guardrails, second tenant | Declare intent in your own repo → infrastructure materializes; policy replaces review | C-01, C-03, C-05..C-08 | **Closed 2026-09-17.** C-05 HELD; C-03 HELD for the kinds composed, one gap recorded; C-01, C-06, C-07 ADJUSTED ([ADR-0016](https://github.com/platform-factory/platform-factory-concept/blob/main/docs/adr/0016-what-the-m2-build-changed.md)); C-08 (stretch) not attempted, deferred to M3. Two claims added, dated (C-24, C-25). |
| M3 | Approval boundary | `edge-config` folders + field + CI, CODEOWNERS, Kyverno reality gates, metadata spine, DNS/edge, ESO, external-dns, Gateway | Repo boundaries can carry the approval model | C-08 (carried), C-09..C-14, C-24 | Next |
| M4 | Factory slice | One change class (dependency bump) end-to-end: done-criteria, approval packet, agent-authored PRs, scorecard, L1→L2 promotion | The autonomy ladder works — the claim nobody has published a working example of | C-15..C-22, C-25 | Planned |

The M1 numbers, for the impatient: three scripted teardown/rebuild cycles;
cycles 2 and 3 at **zero manual interventions**; `2-cluster` rebuild at
**753 s / 755 s / 755 s**; the app-of-apps reached 3/3 Applications
Synced/Healthy from an empty cluster with no manual ordering; every image on
the cluster arrived through Artifact Registry with **zero registry egress** in
the NAT logs. The M2 log has the paved-road equivalents — and the three
ADJUSTED grades, each of which changed the design and says how.

## How the build is run (and why you can trust the status column)

The build is a **pre-registered experiment**
([ADR-0008](https://github.com/platform-factory/platform-factory-concept/blob/main/docs/adr/0008-build-phase-is-a-preregistered-experiment.md)).
Before the first build command, every falsifiable claim in the design docs was
written into an append-only claims register with its test and the data to
collect — 22 claims on 2026-07-31; three added since, each dated. A claim
leaves UNTESTED only via a build-log entry with evidence: **HELD**,
**ADJUSTED** (with a superseding ADR), or **WRONG** (with an ADR, published,
not buried). Milestones are vertical slices; each grades its claims before the
next begins. Some tests' pass criteria were operationalised in a dated
readiness walk at the start of the milestone (the M2 log records which, and
when); the register itself was not reworded. Scaffolds above are scaffolds
because their milestone hasn't started — except `template-service`, which M2
scoped out and says so — not because the work stalled.

## Using this as a template for your own factory

The pattern is the deliverable; GCP is an implementation detail
([ADR-0005](https://github.com/platform-factory/platform-factory-concept/blob/main/docs/adr/0005-gcp-crossplane-reference-implementation.md)).
To stand up your own:

- **Recreate the org shape, not just the code.** The repos above, two teams
  (`platform`, `security`), a `main` ruleset on every repo (PRs required,
  no force-push), CODEOWNERS routing `edge-config/external/` to security.
  The M1 build log records this scaffold taking under two hours.
- **`platform-bootstrap`** needs a billing account and one override:
  `project_id` (default `platform-factory-ref`; GCP project ids are global,
  so yours will differ — set it in every layer's `terraform.tfvars`). The
  Artifact Registry remotes in `0-foundation/registry.tf` are the other knob.
  Team identity is a Google Group per team (`<team>@your-domain`, nested under
  `gke-security-groups@`) — the `systems` README says exactly what must exist
  before the first `System` syncs.
- **`platform-config`** is pointed at by `3-argocd`'s root Application — fork
  it and change the repo URL in that layer's values. Pinned versions carry
  their verification dates; bumping them is a PR, which is the M4 change class.
- **Another cloud:** swap the Crossplane provider family and rewrite layer 0.
  The research digests in the seed cover both GCP and AWS primitives for
  exactly this reason.

## Provenance, citation, license

Written from scratch and generic: no client or employer code, config, or data,
and not built for any one industry. Any business that ships software can use
the pattern.

Each milestone close is released and archived by Zenodo under the concept DOI
above (version DOIs per release; `m2-close` is the first), and the org repos
are tagged to match and archived by
[Software Heritage](https://archive.softwareheritage.org/). The design seed
and the built repos carry a `CITATION.cff`.

Apache-2.0 throughout. Copyright thecloudgeek LLC.
