# Fleet: one project model, one repository interface

Architecture consolidation and reuse review • 14 October 2026

**Recommendation for refinement, not an implementation decision.** Review covers all 34 accessible fleet-47 repositories, related skills, current task records and a primary-source market survey. No repository was changed, no environment restarted, and no new runtime acceptance test was run.

## 1. The direction

Build a small Rust project engine around **jj-owned complete snapshots**, with a single continuous repository interface for agents. Keep Sem as the primary code-navigation and semantic-change provider. Make Astrometrics the connection between approved intent, tasks, actual code and verification evidence.

This is a greenfield integration boundary with selective reuse, not a rewrite of every Fleet application. Existing Fleet work already contains useful record rules, frontier logic, schemas, execution adapters, environment definitions and review concepts. The principal problem is that these pieces describe different generations of the system and do not yet share one accepted-state boundary.

The clean map is:

1. **Judgment:** Pocock skills guide discovery, design, implementation and retrospective. Humans decide architectural tradeoffs and required acceptance.
2. **Project engine:** one Rust interface owns records, task eligibility, claims, submissions, acceptance and revision-aware navigation requests.
3. **Meaning and evidence:** Astrometrics joins specification units to tasks, Sem code entities, tests and review outcomes. These are revision-specific relationships.
4. **Persistence:** jj owns canonical source/record snapshots and history. Git is its initial object backing and exchange path. Query indexes are disposable.
5. **Coordination:** independent workspaces create candidate changes. A shared acceptance authority admits validated results. Iroh and Tandem are candidates at transport/store seams, not competing task authorities.
6. **Execution and presentation:** Fleet-Dispatch/Herdr, FleetHooks, bridge/devenv, CLI/MCP, review tools and visual clients consume the same core contracts.

Weave assists merges. Sem explains code. Astrometrics explains traceability. None of them silently approves work.

## 2. What the consolidation uncovered

### Valuable existing work

- **RepoSync has real Rust code:** a deep Engine interface, record authoring, session handling, a ready frontier, claim/completion behavior, jj subprocess integration and a SQLite projection. Reuse these ideas and tested scenarios selectively.
- **Astrometrics has a substantive binding design:** stable requirement/assertion units, implementation/verification tags and reviewed bindings. Its checker is not yet a completed product. Some move/rename and jj assumptions need revision.
- **The skills pathway is already recognizable**, but Yazzz96/agents is behind the inspected Pocock upstream and lacks implement-spec, pr and retro at its current pin.
- **bridge and the helpers contain reusable environment machinery.** The helpers' package/binary names explain references to Fleet.Stacks; it is not evidence of another accessible standalone repository.
- **Some projects are simply placeholders.** They should appear in the map without becoming invented dependencies.

### Conflicting generations must be reconciled

RepoSync's August design makes records authoritative and databases disposable. Later work proposes a shared live project with no offline actions. October planning selects Omnigraph as authority. The current direction returns canonical ownership to jj and permits provisional offline work. These are materially different contracts.

Keep the history of those decisions, then explicitly supersede the affected authority and concurrency rules in the new specification. Do not merge every old proposal into one larger architecture.

The implementation gap is concrete: current RepoSync lifecycle writes replace multiple Markdown files sequentially and snapshot afterward. A per-file atomic rename is not an atomic domain action. A claim can involve both a ticket and its session record. The new boundary must prepare the complete candidate state before publication.

Also, source presence, a closed documentation bead and a successful historical test are different evidence. The new map preserves those distinctions.

### In-flight work that must not be lost

The current bridge beads snapshot is newer than the Android draft PRs. It records .1 and .4 closed, .2 and .10 in progress, and explicitly retires the older custom hash/promotion design. The paired PRs remain draft and unmerged. Newer simplified exporter, provider mapping and Cargo/Clippy profile work is described in task notes but is not present at the inspected main/PR heads. Preserve it as a local-source capture gap, not as missing work or a completed integration. [Current bridge records](https://github.com/fleet-47/bridge/blob/eecb6ee6b8a5694138d994d8edca72691600a29f/state.jsonl)

RepoSync's separate core-cycle epic has three closed document-delivery tasks while most runtime slices remain open. Its embedded October 12 specifications are valuable acceptance requirements, but the Omnigraph-authority choice needs supersession under this direction. [RepoSync records](https://github.com/fleet-47/Fleet.RepoSync/blob/47daef873b2a78507f5a4ac69f03d34c876c832a/state.jsonl)

Fleet.Supplies also contains copied tool sources, and ConfigManager/Qontinuum copies have diverged. Select one maintained source before porting code. bridge's lock points to an older helper commit than helper main, so auditing the latest helper does not prove it is the deployed package. These are source-of-truth and version-selection issues, not reasons to build another tracker.

## 3. A lifecycle that connects intent to delivery

Use the current Pocock pathway as a branching workflow:

**Wayfinder when needed → design/grilling → to-spec → to-tickets → implement or implement-spec → TDD and two-axis review → authorized delivery → retro.**

Wayfinder clears design uncertainty; it is not mandatory ceremony for a small fix. Its decision graph differs from the implementation task graph. implement-spec can coordinate a ready frontier, while each implementer works from a bounded task and fresh context. Review checks both coding standards and the originating specification.

Use an OpenSpec-like distinction between **proposed change** and **effective specification**. A proposal, design, requirements/scenarios and tasks can evolve together, but acceptance explicitly selects the effective revision. A checked task box or valid document schema is not evidence that the software works.

Recommended initial document strategy: keep one Fleet-compatible canonical record set inside jj snapshots; adopt OpenSpec's change/delta concepts and support import/export only where useful. Evaluate ZhangHanDong/agent-spec as a mechanical contract/checking adapter. Do not maintain Fleet records, an OpenSpec tree and an agent-spec database as three editable authorities.

The trace to expose to an agent is:

**Decision → accepted requirement/scenario revision → task → code/test binding → exact candidate revision → check evidence → review decision → delivered revision.**

An agent should be able to ask which requirement a function implements, which tasks are affected by a change, what evidence is stale and why a task is blocked through the same project interface. Sem supplies code observations; Astrometrics supplies the domain joins and coverage explanation.

Distinguish **write freshness** from **evidence freshness**. The accepted-head precondition prevents races. Evidence remains tied to the specific requirement, code, test, configuration and tool inputs that were examined. An unrelated claim or newly added review receipt can advance the repository without invalidating unchanged inputs. A receipt points to the tested candidate; the later acceptance snapshot contains that receipt and proves its relevant inputs still match. Use existing jj references and the chosen test runner's input model, not a new whole-repository checksum scheme.

Keep separate meanings for governance, execution progress and current evidence. An accepted feature can later regress. A resolved design question is not an implemented task. A delivered task can have stale bindings after a refactor without erasing its historical review.

## 4. The graph model and jj ownership

There are four distinct graph meanings:

- **Decision DAG:** which design questions must be resolved first.
- **Execution DAG:** which accepted task outcomes unblock later work.
- **Semantic relationship graph:** specifications, code, tests, evidence and typed relationships; this whole graph need not be acyclic.
- **Revision DAG:** jj changes and ancestry. jj's operation history additionally records repository-state operations.

Store stable-ID records and typed edges in each complete canonical tree. Use existing Rust graph algorithms such as petgraph for traversal, reachability and cycle checks. Library node indices are not permanent domain identities. jj does not infer that a task dependency means commit ancestry.

One logical domain action should construct its complete candidate records and source changes, validate the applicable invariants, and publish through one selected jj transaction owner. Incomplete/conflicted work may exist as a candidate; it must not become accepted state accidentally. A query states the exact immutable revision it represents. A mutable jj change ID alone is insufficient for evidence identity.

Use one engine-owned **accepted project reference**, represented by a jj bookmark, distinct from candidate workspace heads. Checking the expected accepted revision, validating ownership/domain rules and advancing that reference must be one serialized admission decision. jj can preserve concurrent repository operations; that alone does not choose one accepted task owner. A compare-then-publish check outside this boundary is insufficient. Remote clients may publish candidates only through their allowed scope, not bypass the accepted-reference rule.

Atomic publication concerns the accepted tree/reference, not simultaneous replacement of every working-copy file. A crash may leave a checkout or query index stale. Recover those views against the accepted revision without treating their partially materialized files as new accepted truth. Ordinary workspace snapshots may retain draft edits; they do not automatically advance acceptance.

**Use jj's existing revision/tree/blob identities and producer-native evidence identifiers.** No new per-file hashing framework, parallel event history or universal integrity registry is proposed. A SQLite or graph index may accelerate queries if it can always be rebuilt from the selected snapshot and its schema version.

Undo changes selected active state while preserving jj's recovery history. If task A is removed, unrelated work B can survive. If C depends on A, the semantic layer must report that dependency and require a deliberate repair/removal policy. Fetching or restarting must not replay an independent append-only log that resurrects A.

Local undo does not erase remote copies or automatically retract an already published result. Correction of shared accepted state is itself an explicit publication. This design is not a promise of permanent historical erasure.

Selective domain undo must choose the appropriate jj rewrite/revert and then revalidate the domain graph. Restoring an entire older repository operation is a different action and can discard later unrelated work; it must not be presented as the selective behavior promised here.

## 5. One continuous VCS interface

The strongest facade candidate found is **vcs-core/vcs-jj in ZelAnton/vcs-toolkit-rs**. It already provides a Rust Git/jj interface, with native jj access where the common interface is insufficient. That is worth evaluating instead of writing another broad CLI wrapper.

However, its transaction helper executes multiple visible CLI commands and may subsequently restore the operation state. That is not the required all-or-nothing logical publication under concurrent writers. Its rollback safeguards do not make it a distributed transaction manager.

**Preferred direction:** one jj-native repository interface inside the Rust engine, with jj-lib as the leading candidate for the canonical transaction implementation. Evaluate vcs-core for compatible peripheral repository/forge operations. Do not let vcs-jj, jj-lib and Tandem independently mutate canonical state.

This is still an integration decision to prove. jj-lib gives the needed lower-level transaction seams but brings version/API coupling. A facade does not automatically preserve jj change IDs, operation history, conflicted trees or workspace semantics. Git-only gix/git2 and read-only introspection libraries solve smaller problems. Other surveyed facades include asyncjj (CLI/UI plumbing), endringer (introspection), onevcs (interface-only at the inspected pin) and libvctrl (a custom-VCS SDK). None replaces the required domain-acceptance boundary.

Sem's inspected jj adapter resolves a revset by running jj and taking the first Git SHA, with fallback to the original reference on resolution failure. It is useful integration, not a full jj facade. Fleet must resolve unambiguous baseline/target inputs through its repository interface and pass pinned revisions to Sem, handling conflicts explicitly. A failed resolution must not silently select a different comparison.

## 6. Concurrency without a second truth store

Each agent gets a workspace and a specific plan/requirement revision. Online claims are admitted by the shared authority. When disconnected, work and provisional claims can continue locally, but exclusive ownership cannot be promised. Reconnection retains both attempts and resolves conflicts explicitly.

The current owner/attempt and expected accepted revision are checked when submitting or completing. Old claims cannot overwrite newer ownership. Execution finishing is not the same as a task being accepted, so Dispatch's terminal job state must not directly unblock domain tasks.

Reverting historical domain content must not revive an old worker's permission. An accepted correction that affects ownership must invalidate or replace the affected attempt in that same transition. Preserve the historical claim for explanation, not as a reusable authorization token.

Iroh can carry authenticated peer communication and immutable data. Its sync components do not by themselves define task ownership or jj publication. Exact component selection remains open; “irohsync” should not conceal a second canonical document history.

Tandem is a relevant existing jj store integration. Its current main is substantially newer than its latest inspected release and uses an HTTPS host with S3-compatible durable storage. Its prepared graph publication is promising, but adopting it also adopts meaningful deployment and API responsibilities. A domain acceptance seam, supported offline behavior and compatibility with the chosen jj version must be demonstrated first. Combining Tandem and Iroh is not a verified drop-in configuration.

Offline candidates require a real local jj write path; Tandem's immutable read cache is not evidence of offline mutation support. Sem also cannot be assumed to open a Tandem store through an unrelated stock jj executable. Feed it pinned tree materializations or its parser/graph library seam and qualify that integration.

Weave remains a structural merge backstop. Its advisory coordination state is not the exclusive task-claim authority. After a merge, validate the actual resulting graph and code; previous evidence is reused only when its relevant inputs still match.

For cross-repository work, record the exact component revisions in a versioned integration manifest. Do not promise an atomic transaction across independent repositories or push mutable working directories through a file-sync service.

Set the publication privacy boundary before enabling automatic sync. Raw transcripts, tool output, command arguments, receipts and caches can contain secrets. Keep protected originals outside replicated trees and publish only the explicitly permitted metadata/artifacts. Authenticated transport does not make every recipient authorized for every payload. A pre-publication secret check is preventive; an Actions scan after a push can detect a leak but cannot undo the disclosure. The precise transcript retention/export policy needs ratification because older specs requested complete project-versioned transcripts.

## 7. The proposed build plan

### Phase 0 — Ratify one design

Reconcile the authority/concurrency contradictions, select one canonical specification representation, define the continuous repository interface and choose one small existing Fleet feature as the proving case. Preserve prior IDs, decisions and task history. Review the missing upstream skills deliberately rather than replacing local adaptations wholesale.

Exit: one accepted specification, a source-backed reuse map, explicit deferred choices and a small execution DAG. No mass repository migration.

### Phase 1 — Prove the local transaction boundary

Implement one vertical slice: accepted requirement → task → claim → code/record candidate → evidence → review → accepted snapshot. Prove jj-backed complete-action publication and disposable query rebuilding before adding distributed services.

Exit: interruption between every persistence stage cannot yield a falsely successful partial action; undo preserves unrelated work and detects dependent invalidity; restart does not resurrect removed state.

### Phase 2 — Prove semantic traceability

Use Sem for navigation and changes, Astrometrics for spec/code/test bindings, and a narrowly scoped agent-spec adapter if it earns reuse. Configure --review-mode strict: pending_review, skip and unavailable must not pass a required gate. Fleet still owns approver identity and the binding to exact reviewed inputs. Keep unsupported languages and ambiguous bindings visible. Add Inspect only as optional evidence with its license and coverage understood.

Exit: change a requirement, rename/move a symbol and alter a test; the interface reports exactly which links and evidence need review. No false pass from an empty report or exit code alone.

### Phase 3 — Prove concurrent workspaces

Add connected ownership, provisional offline attempts, explicit integration and Weave-assisted merging. Evaluate Tandem/Iroh at their actual seams rather than constructing a new synchronization framework.

Exit: two connected claims yield at most one accepted owner; disconnected work survives; stale submissions cannot complete newer attempts; retries do not duplicate accepted outcomes; merged code and graph are checked together.

### Phase 4 — Dogfood, then migrate selectively

Connect FleetHooks and Dispatch, use bridge/devenv's existing checks and standard caches, and run one real multi-agent task through human review and delivery. Only then rehearse imports from beads and legacy records on copies, reconcile IDs/content/history and switch one authority.

Exit: one complete observed cycle, an honest retrospective, an export/recovery demonstration and a clear cutover/reconciliation route. Existing repositories and task stores remain untouched until that decision.

## 8. Failure cases that must shape the design

- Crash after writing one of two related records: no accepted partial task/session state.
- Undo, restart and fetch: removed active records stay removed; unrelated later work survives.
- Clean text merge creates a dangling dependency or cycle: semantic acceptance rejects it.
- Spec/code changes after review: earlier acceptance remains historical, not silently current.
- Offline duplicate work: both attempts retained; no fabricated exclusive claim.
- Timeout after publication: reconcile the actual outcome before retry.
- Weave produces clean code: tests and requirement bindings still need validation.
- Tool missing, traversal truncated or language unsupported: report incomplete evidence, not absence or success.
- Transcript includes credentials/private material: do not distribute raw originals to every repository clone. Decide access and retention separately.
- Native Android tool builds: field evidence for one binary does not prove the entire server/toolchain stack works there.

## 9. Decisions for our next refinement

1. **Persistence/interface:** agree that jj owns canonical snapshots and that the first proof is a single jj-native transaction boundary; use facade libraries where they preserve those semantics.
2. **Specification contract:** choose the Fleet-compatible canonical document grammar, with OpenSpec-style change lifecycle and a bounded agent-spec adapter, rather than parallel editable stores.
3. **First proving scope:** one repository, two workspaces and one real feature, before a shared-store deployment or Fleet-wide migration.

These are recommendations, not requests to approve a large build in one go. The remaining backend and transport choices should be settled by the bounded proofs above.

## 10. Market shortlist and reuse decisions

- **Matt Pocock skills:** adopt the branching judgment pathway, with a reviewed upstream pin and explicit Fleet adaptations. The current local collection is missing the final orchestration/PR/retro pieces. [Current router](https://github.com/mattpocock/skills/blob/49dd158d1076134a641b33efb035946536778336/skills/engineering/ask-matt/SKILL.md)
- **OpenSpec 1.14.1:** use the proposal/design/requirements/tasks and delta-to-effective-spec model. Its implementation is TypeScript; its artifact DAG orders documents, not exclusive worker claims. No Rust SDK was found. Oracle’s agent-spec is a different agent-runtime layer; iging/agent-spec packages documents/skills. Neither is the Rust contract candidate recommended here. [Source](https://github.com/Fission-AI/OpenSpec/tree/9111a7654d7800391459431fff4eaf66e33a3d2e)
- **ZhangHanDong/agent-spec 1.4.0:** strongest new Rust contract candidate. Actual requirement-plan, code-binding and lifecycle code exists. Start through CLI/JSON because the top-level package has no public library target. Require strict review mode: its default auto mode can treat pending review as passing. This still does not authenticate Fleet's human approval. [Gateway](https://github.com/ZhangHanDong/agent-spec/blob/2483398da89bd392186849bb865aae969667ac0a/src/spec_gateway/lifecycle.rs)
- **rust-atlas and the agent-spec code-provider kit:** reusable Rust libraries and small provider seams. Evaluate only for demonstrated Sem coverage gaps; do not build two full code indexes by default. No agent-navigation performance gain was verified in this review. [Provider kit](https://github.com/ZhangHanDong/agent-spec/tree/2483398da89bd392186849bb865aae969667ac0a/crates/code-graph-provider)
- **vcs-core 0.10 / vcs-jj 0.12:** useful existing facade and native jj command surface; CLI rollback is not atomic concurrent domain publication. Qualify at a narrow seam. [Facade guide](https://docs.rs/vcs-core/0.10.0/vcs_core/guide/), [jj guide](https://docs.rs/vcs-jj/0.12.0/vcs_jj/guide/)
- **jj-lib:** leading canonical transaction implementation candidate. Reuse its object/history/view machinery; account for upstream API coupling. [Architecture](https://www.jj-vcs.dev/latest/technical/architecture/)
- **Sem:** primary navigation/semantic-change provider. Native entity IDs, hashes and language coverage are observations at a revision, not permanent requirement IDs or acceptance. Its jj helper alone is not a repository facade. [Source](https://github.com/Ataraxy-Labs/sem)
- **Ataraxy Weave:** reuse structural merge core/driver under Fleet's publication owner. Its newer land/check flow is relevant prior art, but per-file merge verification does not prove cross-file behavior. Do not add a second publisher or use advisory claims as exclusive ownership. [Source](https://github.com/Ataraxy-Labs/weave)
- **Inspect:** optional local review triage. Its inspected license is FSL-1.1-ALv2, not the same permissive license as Sem/Weave; assess the intended redistribution/use before embedding. Embedded Sem versions also differ across these tools. [License](https://github.com/Ataraxy-Labs/inspect/blob/main/LICENSE.md)
- **jj-tandem:** substantial shared-store prior art, with a headless repository seam. Current main has a different architecture from the inspected v0.4.0 release. Qualify admission, durability and client/version behavior before adopting it; do not assume an Iroh transport replacement is supported. [Architecture](https://github.com/laulauland/tandem/blob/main/ARCHITECTURE.md)
- **Iroh:** candidate authenticated transport and immutable-object movement, not task acceptance or full jj synchronization by itself. Select actual components after the data/publication protocol is settled. [Official project](https://github.com/n0-computer/iroh)
- **petgraph:** use standard traversal/DAG algorithms for domain rules rather than inventing graph machinery. Its acyclic wrapper can reject cycle-creating edges; domain validation still owns relationship meaning. [Acyclic API](https://docs.rs/petgraph/0.8.3/petgraph/acyclic/struct.Acyclic.html)
- **beads-rs / jjj:** useful task UX, import and design references, but not unchanged solutions for this undo requirement. delightful-ai/beads-rs makes notes append-only; inspected doug/jjj writes local metadata outside normal snapshots and batches jj commits at push. [beads-rs contract](https://github.com/delightful-ai/beads-rs/blob/main/SPEC.md), [jjj publication boundary](https://github.com/doug/jjj/blob/main/src/commands/push.rs)
- **Dolt, Automerge and Loro:** real versioned-data alternatives, but they introduce their own history/merge semantics. Do not adopt them as another canonical state machine alongside jj. Loro/Automerge may be useful for optional collaborative UI drafts if a later requirement justifies it. [Dolt remotes](https://www.dolthub.com/docs/sql-reference/version-control/remotes/), [Automerge model](https://automerge.org/docs/reference/concepts/), [Loro versioning](https://www.loro.dev/docs/api/js)


## 11. The complete repository map

All 34 accessible fleet-47 repositories are accounted for below. “Source” or “application” means inspected implementation exists, not that current runtime acceptance passed. These dispositions are recommendations; no repository is being deleted, merged or published. Pins are exact inspected default-branch commits; qontinuum uses v1. Active non-default work is noted separately.

| Repository | Observed stage | Consolidation disposition | Head |
|---|---|---|---|
| [agentchattr](https://github.com/fleet-47/agentchattr) | Python app | Reuse chat/review UX and identity routing. Replace independent JSON/JSONL domain writers if used as a client. | [4892cf1b](https://github.com/fleet-47/agentchattr/tree/4892cf1b0af975e53dc253f34b072f8c6e284bb8) |
| [Astrometrics](https://github.com/fleet-47/Astrometrics) | Design + visual demo | Keep requirement/assertion and binding contract; build the actual checker over Sem and canonical revisions. | [3b8a5766](https://github.com/fleet-47/Astrometrics/tree/3b8a57668f0a9607a2f07113599dab2643eceff8) |
| [bridge](https://github.com/fleet-47/bridge) | Nix source + drafts | Keep toolchains/gates. Reconcile newer task-recorded local simplifications before reusing Android drafts. | [5c336ddb](https://github.com/fleet-47/bridge/tree/5c336ddb3473562131cda6337a9813d9f6ff9281) |
| [comment-review](https://github.com/fleet-47/comment-review) | Rust utility | Useful sidecar/drift patterns. Default strips comments; do not use as automatic semantic ingestion. | [f19a7e6d](https://github.com/fleet-47/comment-review/tree/f19a7e6d28964354d6d9333bbb753c4eeb081a8d) |
| [ConfigManager](https://github.com/fleet-47/ConfigManager) | Rust/TS application | Keep useful native config adapters separately. Qontinuum realignment incomplete; live writes not jj transactions. | [4a352c83](https://github.com/fleet-47/ConfigManager/tree/4a352c834454520c22a81c15ca089f7cb4e5ef01) |
| [data-core](https://github.com/fleet-47/data-core) | TypeScript prototype | Port useful pure graph transformations and views only. Its file writer is not a whole-action transaction. | [f31f482e](https://github.com/fleet-47/data-core/tree/f31f482ede6bbd7d50e3674fed0730a217cc7411) |
| [demo-repository](https://github.com/fleet-47/demo-repository) | GitHub example | Exclude from product architecture; retain as demonstration history. | [cc438feb](https://github.com/fleet-47/demo-repository/tree/cc438feb09b655a9c568ea3db465d40c27c4e23b) |
| [doGroups](https://github.com/fleet-47/doGroups) | Scaffold | No application source discovered; defer. | [80265c50](https://github.com/fleet-47/doGroups/tree/80265c50fadc4c531571e552e94fedb0dd08ef0d) |
| [fleet-configs](https://github.com/fleet-47/fleet-configs) | Parked reference | Upstream-surface corpus and extraction ideas; README says not a working tool. Not device config history. | [f3cac0a3](https://github.com/fleet-47/fleet-configs/tree/f3cac0a3bddd8d4e8a0fe169db9e3199bd5191d1) |
| [fleet-dev](https://github.com/fleet-47/fleet-dev) | Request only | Future prompt/status consumer of common repository/project interface. | [69e78469](https://github.com/fleet-47/fleet-dev/tree/69e78469e646df21ea3eba6c1299a454c64f7116) |
| [Fleet-Dispatch](https://github.com/fleet-47/Fleet-Dispatch) | Partial Rust runtime | Reuse execution/Herdr adapters; accepted prerequisites and claims stay in the core. Public verbs/authz integration incomplete. | [341db292](https://github.com/fleet-47/Fleet-Dispatch/tree/341db2929ff379682fbea15a7f2e8247468518df) |
| [Fleet.Configs](https://github.com/fleet-47/Fleet.Configs) | Scaffold | Intended standalone generated-config/device-history destination. Distinct from lowercase fleet-configs. | [284477ac](https://github.com/fleet-47/Fleet.Configs/tree/284477ac058ce3b23eea9629392b851a09335271) |
| [Fleet.Devenv](https://github.com/fleet-47/Fleet.Devenv) | Design/research | Reuse shared spec/binding contracts. No executable environment builder to consolidate. | [461687c3](https://github.com/fleet-47/Fleet.Devenv/tree/461687c3c807456299788f7f4ac0fb013b8bb230) |
| [Fleet.Flake-Helpers](https://github.com/fleet-47/Fleet.Flake-Helpers) | Rust source + drafts | Reuse declaration/binding helpers. Fleet.Stacks alias; reject wholesale import of superseded native controller. | [69f39a16](https://github.com/fleet-47/Fleet.Flake-Helpers/tree/69f39a161ca4c63225893016af348f629a7e7a4f) |
| [Fleet.Formations](https://github.com/fleet-47/Fleet.Formations) | Legacy Nix scaffold | Historical composition ideas only; bridge is the current environment baseline. | [5b431e2a](https://github.com/fleet-47/Fleet.Formations/tree/5b431e2a9a8c97983d6457fef2c1e41c42d30852) |
| [Fleet.RepoSync](https://github.com/fleet-47/Fleet.RepoSync) | Partial Rust core | Primary domain reuse candidate. Replace per-file writes then snapshots with complete-action jj publication. | [23e523b6](https://github.com/fleet-47/Fleet.RepoSync/tree/23e523b6d545d31c6f5c4ddfc2afe22f0563d0b4) |
| [Fleet.Scripts](https://github.com/fleet-47/Fleet.Scripts) | Scaffold | No executable scripts to adopt; preserve history rather than inventing a core module. | [7f84ea03](https://github.com/fleet-47/Fleet.Scripts/tree/7f84ea0362c4bb13bfc8b3d4e1f984a01b05a0f0) |
| [Fleet.Supplies](https://github.com/fleet-47/Fleet.Supplies) | Legacy catalogue | Reconcile package facts and copied sources with bridge/standalone owners. No second software authority. | [52436f66](https://github.com/fleet-47/Fleet.Supplies/tree/52436f66e5216cf41da29617c8281af0bd7e5777) |
| [Fleet.Terax](https://github.com/fleet-47/Fleet.Terax) | Partial native editor | Optional editor/operator client. Adapt its Git write path; preserve IPC/view patterns, not another authority. | [c14c3430](https://github.com/fleet-47/Fleet.Terax/tree/c14c34309aea92b9c2d14ba6e383e0706fd568e9) |
| [FleetCmd](https://github.com/fleet-47/FleetCmd) | Rust mount app | Peripheral NFS/mount UI, overlapping fleetctl. Select maintained source; not a task engine. | [8e797ca4](https://github.com/fleet-47/FleetCmd/tree/8e797ca4b8bdbc6bbc217a7b672d19627947c581) |
| [fleetctl](https://github.com/fleet-47/fleetctl) | Rust mount app | Keep peripheral. macOS validation gap; source explicitly rejects Android. | [aaeec209](https://github.com/fleet-47/fleetctl/tree/aaeec20961c050db49da4baa5dafa86c9f8a4940) |
| [FleetHooks](https://github.com/fleet-47/FleetHooks) | Partial Rust integration | Keep harness normalization and capability reporting; reconcile ruled policy/config changes against actual source. | [2af757d7](https://github.com/fleet-47/FleetHooks/tree/2af757d75861ab00ac283378902bddc11be6f40e) |
| [holobridge](https://github.com/fleet-47/holobridge) | Rust media relay | Keep independent; media/VM networking is outside spec/task/history core. | [a7ebfae6](https://github.com/fleet-47/holobridge/tree/a7ebfae636622a7b6c9d8f2db612f2ac9073ab23) |
| [MarkdownEditor](https://github.com/fleet-47/MarkdownEditor) | Empty repository | No source or design to consolidate. | Empty |
| [maxwells-demon-rs](https://github.com/fleet-47/maxwells-demon-rs) | Rust/Swift application | Use as real binding/check fixture and deep-module/acceptance example, not a core dependency. | [fc4e3952](https://github.com/fleet-47/maxwells-demon-rs/tree/fc4e3952e420c40aa8e77e84d52bd8d24f5f2b05) |
| [mess-hall](https://github.com/fleet-47/mess-hall) | Design only | Optional high-fidelity clarification/review UI for Wayfinder/grilling; no server implementation. | [4d6b5965](https://github.com/fleet-47/mess-hall/tree/4d6b5965628c28c71a83cfcd7507fe36f892543b) |
| [nix-on-droid](https://github.com/fleet-47/nix-on-droid) | Empty initial commit | Paused native Bionic research, no implementation to port. Not upstream Nix-on-Droid. | [d527f032](https://github.com/fleet-47/nix-on-droid/tree/d527f032abd372f88e98c92244525637016a8b7c) |
| [ObsidianMods](https://github.com/fleet-47/ObsidianMods) | Scaffold | No implemented Obsidian adapter discovered; defer. | [ed26fd19](https://github.com/fleet-47/ObsidianMods/tree/ed26fd19d22679c0cf28b2a13245e6cd7ce5bef3) |
| [pattern-buffer](https://github.com/fleet-47/pattern-buffer) | Scaffold | No storage engine discovered; its name is not evidence of persistence capability. | [b3085ffc](https://github.com/fleet-47/pattern-buffer/tree/b3085ffc569ce17c5163082eef8d81828dad1e91) |
| [qontinuum](https://github.com/fleet-47/qontinuum) | Rust/TS application | Default branch v1; reconcile divergent ConfigManager/Supplies copies before choosing source. Reuse evidence patterns. | [094b8568](https://github.com/fleet-47/qontinuum/tree/094b8568816b8d0e3873b32f198fcbd53296b9de) |
| [RustMem](https://github.com/fleet-47/RustMem) | Placeholder + design | Defer memory services; no retain/recall engine to integrate today. | [7ea717c5](https://github.com/fleet-47/RustMem/tree/7ea717c5eae0be9f76b9085793173068555b3785) |
| [tabularis-secscan](https://github.com/fleet-47/tabularis-secscan) | Reference staging | Vendored docs, MCP/query tasks untouched. Peripheral security research, not a core dependency. | [847aa835](https://github.com/fleet-47/tabularis-secscan/tree/847aa835d80a6bdefc620a6a8dd7693663995c3c) |
| [TeloxideBot](https://github.com/fleet-47/TeloxideBot) | Scaffold | No bot transport implemented; future optional client. | [4535775c](https://github.com/fleet-47/TeloxideBot/tree/4535775cd519b45742b05c0fadf14e4f8b0d25ea) |
| [xattr-deck](https://github.com/fleet-47/xattr-deck) | Rust core modules | xattr/Finder/ACL helpers exist; main is empty. Peripheral utility, not stable record identity. | [82e1cc1c](https://github.com/fleet-47/xattr-deck/tree/82e1cc1c4d684838c423ed30b7905f63304c2855) |

### Reuse boundaries that deserve explicit attention

- Dispatch has real runtime adapters, but several public commands remain unimplemented and the newer authorization crate is not wired into its engine/CLI. A terminal job, including a failed one, can satisfy its operational dependency rule. Adapt that boundary; it is not proof of task acceptance.
- FleetHooks source still defaults missing/empty configuration and uses older boolean policies despite newer ruled changes in its backlog. Reconcile source and decisions before claiming the new policy is enforced.
- Astrometrics currently excludes jj from its checker scope and gives moved/renamed definitions new identities, while its charter asks for persistent semantic anchors. Revise the contract explicitly; do not claim those requirements already agree.
- data-core's graph is a teaching/content model with containment constraints. Reuse its pure transformations and relationship distinctions, not its learner-owned mastery state or sequential writer.
- ConfigManager, qontinuum and Supplies contain overlapping/divergent copies. FleetCmd/fleetctl also overlap. A clean map should reduce duplicate ownership before introducing another shared crate.
- Keep live process supervision, connections and credentials out of canonical source history. Dispatch operational stores may persist useful runtime state without becoming task/spec truth. The “disposable database” rule applies to derived project queries, not blindly to every process supervisor.

## 12. Coverage, limitations and source register

### What was actually checked

All 34 accessible repositories were enumerated. Reviews inspected default-branch trees and pinned relevant source, instructions, manifests, designs and handoffs. Central semantic/execution/environment projects received deeper operational-path review; small placeholders were verified directly. Relevant branch/PR collections, bridge and RepoSync beads stores, draft receipts and selected Actions evidence were compared. The market review used current primary repositories, source, release metadata and API documentation.

This is a broad architecture/source audit, not a line-by-line audit of all vendored code or a runtime certification. No builds, tests, installations, deployments, local machine inspection or repository mutations were performed. Selected historical checks are evidence of their documented scope only. GitHub license metadata was absent for many Fleet repos; dependency/vendored rights still need review before any public greenfield publication.

### Known blind spots

Newer bridge exporter/provider/Cargo changes are recorded as local work but not present in the inspected GitHub heads. Other unpushed jj changes, external symlink targets, active jobs and installed binary/configuration state were not inspected. Archived or unrelated vendor/UI/media code was inventoried but not exhaustively re-audited. These are explicit coverage gaps, not a finding that no work exists.

Before implementing a successor, capture/reconcile the newer local work in the selected environment with permission to resume it. The stopped environment remained stopped during this review.

### Reproducibility anchors

- [RepoSync current code and historical design](https://github.com/fleet-47/Fleet.RepoSync/tree/23e523b6d545d31c6f5c4ddfc2afe22f0563d0b4); [current core-cycle task contract](https://github.com/fleet-47/Fleet.RepoSync/blob/47daef873b2a78507f5a4ac69f03d34c876c832a/state.jsonl).
- [Astrometrics binding specification](https://github.com/fleet-47/Astrometrics/blob/3b8a57668f0a9607a2f07113599dab2643eceff8/.fleet/docs/specifications/MAPSPC1-Binding-Checker.md); [Dispatch engine](https://github.com/fleet-47/Fleet-Dispatch/blob/341db2929ff379682fbea15a7f2e8247468518df/crates/dispatch-engine/src/lib.rs); [FleetHooks configuration](https://github.com/fleet-47/FleetHooks/blob/2af757d75861ab00ac283378902bddc11be6f40e/crates/fleethooks/src/config.rs).
- [Bridge main](https://github.com/fleet-47/bridge/tree/5c336ddb3473562131cda6337a9813d9f6ff9281); [newer canonical task state](https://github.com/fleet-47/bridge/blob/eecb6ee6b8a5694138d994d8edca72691600a29f/state.jsonl); [bridge draft PR 2](https://github.com/fleet-47/bridge/pull/2); [helper draft PR 1](https://github.com/fleet-47/Fleet.Flake-Helpers/pull/1).
- [Local skills collection](https://github.com/Yazzz96/agents/tree/0f88e9ea4c69542534c2d070e2d89b354001bc1b); [Pocock upstream](https://github.com/mattpocock/skills/tree/49dd158d1076134a641b33efb035946536778336).
- [OpenSpec snapshot](https://github.com/Fission-AI/OpenSpec/tree/9111a7654d7800391459431fff4eaf66e33a3d2e); [Rust agent-spec snapshot](https://github.com/ZhangHanDong/agent-spec/tree/2483398da89bd392186849bb865aae969667ac0a).
- [vcs-toolkit snapshot](https://github.com/ZelAnton/vcs-toolkit-rs/tree/fb88014f2da25d801ec46758593f520792434499); [CLI rollback implementation](https://github.com/ZelAnton/vcs-toolkit-rs/blob/fb88014f2da25d801ec46758593f520792434499/crates/jj/src/transaction.rs); [jj native transaction source](https://github.com/jj-vcs/jj/blob/v0.45.1/lib/src/transaction.rs).
- [Sem snapshot](https://github.com/Ataraxy-Labs/sem/tree/6cb178dc209dc1109b46c8c6ed8173c5209d91b5); [Weave snapshot](https://github.com/Ataraxy-Labs/weave/tree/928650289c2e3adb378472432345a7bd34323ca3); [Inspect snapshot/license](https://github.com/Ataraxy-Labs/inspect/blob/90a8a5dd3a15c39e59062e2518171615fecff5c3/LICENSE.md).
- [Tandem current architecture pin](https://github.com/laulauland/tandem/blob/d0cd683dbfefc990e12cec7af4e46764deb63598/ARCHITECTURE.md); [reliability contract](https://github.com/laulauland/tandem/blob/d0cd683dbfefc990e12cec7af4e46764deb63598/docs/reliability.md); [iroh-docs pin](https://github.com/n0-computer/iroh-docs/tree/8cfeacb087b4b195b1930683aa4448e990da0659).

At review time jj 0.46.0 was released, while Tandem main embeds 0.45.1 and vcs-toolkit's inspected real-jj CI matrix covers older versions. Sem primary is 0.27, Weave main depends on sem-core 0.25 and Inspect on 0.5.3. The newest package versions are not automatically a compatible integration set. Pin a coherent proof configuration and test upgrades deliberately.

“Irohsync” was interpreted as the current Iroh ecosystem. The historical iroh-sync line was renamed into iroh-docs; no exact active project named irohsync was verified. If a different project was intended, its link could materially change the transport comparison.
