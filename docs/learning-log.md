# Fuse Learning Log

> Status: normative for Fuse.
>
> This file is the append-only project learning log. It is seeded with
> retrospective entries L001-L022 distilled from earlier Fuse attempts so the
> current rules, audit checklist, and wave plans have concrete historical
> referents. Every new meaningful issue, bug, obstacle, and wave closure is
> fixed or explicitly rescheduled first, then appended here.

## Bug entry format

Every bug entry must use the following structure.

### LNNN — Title

Date: YYYY-MM-DD
Discovered during: Wave / Phase / Task

**Reproducer**:
Minimal case that exposes the problem.

**What was tried first**:
The first failed approach and why it failed.

**Root cause**:
What was actually wrong.

**Spec gap**:
Which part of the language reference was silent, ambiguous, or incomplete.

**Plan gap**:
Which part of the implementation plan failed to schedule or constrain the work.

**Fix**:
What changed.

**Cascading effects**:
What other bugs or design consequences the fix exposed.

**Architectural lesson**:
What invariant or design principle should be carried forward.

**Verification**:
The commands, tests, or fixtures that proved the fix.

## Wave closure entry format

Every wave requires a closure entry before it may be marked complete.
Closure entries use a separate series (`WCxxx`) so they are distinguishable
from bug entries.

### WCxxx — Wave XX Closure

Date: YYYY-MM-DD
Wave: XX — Wave Title

**Proof programs added this wave**:
List each `.fuse` source file committed to `tests/e2e/` along with its
expected output (exit code or stdout). If no proof programs were added,
state why and which wave is responsible for them.

**Stubs retired this wave**:
List each row removed from `STUBS.md`, naming the stub, the task that retired
it, and the `Verify:` command that confirmed its removal.

**Stubs introduced this wave**:
List each row added to `STUBS.md`, naming the stub, the file:line, the
diagnostic it emits, and the wave scheduled to retire it.

**What was harder than planned**:
Honest account of tasks that took longer, required rework, or surfaced
unexpected complexity. If nothing was harder than planned, state so explicitly.

**What the next wave must know**:
State and context the successor contributor or agent needs that is not captured
elsewhere in the foundational documents. Include any latent issues observed but
not fixed, any assumptions carried forward, and any work that was deferred out
of scope.

**Verification**:
The specific commands that prove the wave is complete. These must match the
"Proof of completion" commands in the implementation plan for this wave.

## Entries

### L001 — Lexical Ambiguities Must Be Written Down

Date: 2026-04-19
Discovered during: Retrospective seed from earlier Fuse attempts

**Reproducer**:
Different readers disagreed about raw-string guards, `?.`, and similar edge
cases because the rules existed only as examples.

**What was tried first**:
Rely on parser behavior and golden tests to settle ambiguous tokens.

**Root cause**:
Lexical ambiguities were not specified as explicit contracts.

**Spec gap**:
The language reference implied ambiguity decisions instead of stating them.

**Plan gap**:
Lexer work was scheduled without adversarial ambiguity checks.

**Fix**:
Attempt 6 writes lexical ambiguity contracts into the reference and gives W01
dedicated coverage for them.

**Cascading effects**:
Parser planning now treats ambiguity fixtures as first-class regressions.

**Architectural lesson**:
Ambiguous surface forms must be decided at the lexical boundary, not left for
later passes to infer.

**Verification**:
Cross-check `docs/language-reference/fuse-language-reference.md` §1.10,
`docs/implementation/wave01_lexer.md`, and `docs/audit.md` checklist D1.

### L002 — Stdlib Bodies Must Be Type-Checked

Date: 2026-04-19
Discovered during: Retrospective seed from earlier Fuse attempts

**Reproducer**:
Stdlib signatures appeared valid while unchecked bodies leaked `Unknown` types
 into later passes.

**What was tried first**:
Check stdlib signatures only and assume bodies would follow once user code
worked.

**Root cause**:
The pipeline treated stdlib as declarations instead of ordinary code.

**Spec gap**:
The reference did not make stdlib body checking a hard correctness rule.

**Plan gap**:
Earlier plans failed to bind stdlib body checking to the checker exit criteria.

**Fix**:
Attempt 6 makes stdlib body checking explicit in the rules and W06 exit
criteria.

**Cascading effects**:
Core and hosted stdlib waves now act as semantic stress tests rather than thin
wrappers.

**Architectural lesson**:
If stdlib is part of the language contract, its bodies must traverse the same
checker path as user code.

**Verification**:
Cross-check `docs/rules.md` Rule 5.2, `docs/implementation/wave06_type_checking.md`,
and `docs/audit.md` checklist D2.

### L003 — Partial Generic Specializations Are Invalid States

Date: 2026-04-19
Discovered during: Retrospective seed from earlier Fuse attempts

**Reproducer**:
Generic functions reached codegen with some type variables substituted and some
still abstract.

**What was tried first**:
Allow partially inferred specializations and hope later passes could complete
them.

**Root cause**:
Monomorphization lacked a hard completeness gate before codegen.

**Spec gap**:
The reference did not clearly forbid partially concrete instantiations.

**Plan gap**:
Generic work was scheduled without a dedicated completeness check in the driver.

**Fix**:
Attempt 6 requires monomorphization to reject partial specializations and names
that check explicitly in W08.

**Cascading effects**:
Driver integration became a first-class part of generics retirement.

**Architectural lesson**:
There is no safe intermediate state where generics are "mostly specialized".

**Verification**:
Cross-check `docs/implementation/wave08_monomorphization.md` and
`docs/audit.md` checklist D3.

### L004 — Borrow Pointers and Raw Pointers Must Stay Distinct

Date: 2026-04-19
Discovered during: Retrospective seed from earlier Fuse attempts

**Reproducer**:
Borrow-derived pointers and raw `Ptr[T]` values followed the same backend path
and lost safety distinctions.

**What was tried first**:
Reuse one pointer lowering path and patch special cases at call sites.

**Root cause**:
Pointer categories were treated as backend convenience instead of architecture.

**Spec gap**:
The representation contract between borrow pointers and raw pointers was too
implicit.

**Plan gap**:
Codegen work did not carry a dedicated pointer-category gate.

**Fix**:
Attempt 6 codifies pointer-category separation in the reference, rules, W17,
and the audit checklist.

**Cascading effects**:
Call-site adaptation and field-access lowering now have explicit invariants.

**Architectural lesson**:
Backend pointer categories are part of the language contract, not cleanup.

**Verification**:
Cross-check `docs/rules.md` Rule 3.10,
`docs/language-reference/fuse-language-reference.md` §57.1, and
`docs/audit.md` checklist D4.

### L005 — Unit Erasure Must Be Total Across One ABI

Date: 2026-04-19
Discovered during: Retrospective seed from earlier Fuse attempts

**Reproducer**:
One path erased `Unit` payloads while another still emitted storage for them,
breaking ABI consistency.

**What was tried first**:
Erase `Unit` opportunistically where it was easiest to codegen.

**Root cause**:
Erasure was treated as a local optimization instead of a whole-ABI invariant.

**Spec gap**:
The reference did not require every participating layout to agree on erasure.

**Plan gap**:
Backend tasks lacked an explicit total-unit-erasure retirement check.

**Fix**:
Attempt 6 makes total unit erasure a representation contract and a W17 gate.

**Cascading effects**:
Constructors, patterns, function pointers, and aggregates now share one rule.

**Architectural lesson**:
ABI erasure choices must be total, not piecemeal.

**Verification**:
Cross-check `docs/language-reference/fuse-language-reference.md` §57.2,
`docs/implementation/wave17_codegen_c11_hardening.md`, and
`docs/audit.md` checklist D5.

### L006 — Divergence Must Be Structural, Not Simulated

Date: 2026-04-19
Discovered during: Retrospective seed from earlier Fuse attempts

**Reproducer**:
Lowering invented fallback values after `return` or panic to satisfy type
expectations.

**What was tried first**:
Patch divergent blocks with placeholder expressions so later passes could keep
going.

**Root cause**:
Divergence was modeled as an expression-value hack instead of control-flow
termination.

**Spec gap**:
The reference did not say strongly enough that divergent paths stay divergent.

**Plan gap**:
MIR and codegen waves lacked a named structural-divergence invariant.

**Fix**:
Attempt 6 makes divergence a structural MIR/codegen contract.

**Cascading effects**:
Sealed-block and post-termination invariants became explicit downstream checks.

**Architectural lesson**:
Divergent control flow must terminate structurally; fake values only hide bugs.

**Verification**:
Cross-check `docs/rules.md` Rule 3.10,
`docs/implementation/wave15_lowering_and_mir_consolidation.md`,
`docs/implementation/wave17_codegen_c11_hardening.md`, and
`docs/audit.md` checklist D6.

### L007 — Pattern Matching Needs Structured Pattern Nodes

Date: 2026-04-19
Discovered during: Retrospective seed from earlier Fuse attempts

**Reproducer**:
`match` arms were stored as textual descriptions, so exhaustiveness and
lowering could not reason about them structurally.

**What was tried first**:
Carry pattern strings through the frontend and decode them later.

**Root cause**:
Pattern matching lacked a structured semantic IR.

**Spec gap**:
The reference did not require structured pattern nodes early enough.

**Plan gap**:
Pattern matching had no dedicated retirement wave with its own proof program.

**Fix**:
Attempt 6 defines structured pattern nodes in HIR and gives pattern matching
its own wave.

**Cascading effects**:
Exhaustiveness, unreachable-arm detection, and lowering now share the same
pattern model.

**Architectural lesson**:
User-visible pattern semantics cannot be recovered reliably from text later.

**Verification**:
Cross-check `docs/implementation/wave04_hir_and_typetable.md`,
`docs/implementation/wave10_pattern_matching.md`, and
`docs/audit.md` checklist C1.

### L008 — Monomorphization Must Run in the Driver Pipeline

Date: 2026-04-19
Discovered during: Retrospective seed from earlier Fuse attempts

**Reproducer**:
Monomorphization helpers existed but were never called from the real driver
pipeline.

**What was tried first**:
Implement specialization utilities in isolation and plan to wire them later.

**Root cause**:
Cross-cutting generic work was built as a library, not as a pipeline phase.

**Spec gap**:
The reference did not say enough about concrete instantiations reaching codegen
only after driver integration.

**Plan gap**:
Generic work was under-scoped and lacked a dedicated driver integration gate.

**Fix**:
Attempt 6 gives monomorphization its own wave and explicit driver-pipeline
tasks.

**Cascading effects**:
Specialization collection, duplication, call-site rewriting, and codegen skip
logic are now all separately verified.

**Architectural lesson**:
Cross-cutting passes are not real until the driver owns them.

**Verification**:
Cross-check `docs/implementation/wave08_monomorphization.md` and
`docs/audit.md` checklist C2.

### L009 — `?` Must Lower to Branch and Early Return

Date: 2026-04-19
Discovered during: Retrospective seed from earlier Fuse attempts

**Reproducer**:
`expr?` behaved like pass-through lowering instead of branching on `Err` or
`None`.

**What was tried first**:
Type-check `?` and defer its actual branch semantics.

**Root cause**:
Error propagation was treated as syntax sugar instead of control-flow
rewriting.

**Spec gap**:
The lowering behavior of `?` was not pinned down tightly enough.

**Plan gap**:
Error propagation lacked a dedicated wave and proof program.

**Fix**:
Attempt 6 gives `?` a dedicated wave with checker, lowering, and proof tasks.

**Cascading effects**:
`Result` and `Option` propagation now share one explicit lowering contract.

**Architectural lesson**:
User-visible control-flow sugar must lower to explicit control flow, not silent
identity behavior.

**Verification**:
Cross-check `docs/implementation/wave11_error_propagation.md` and
`docs/audit.md` checklist C3.

### L010 — Drop Codegen Must Emit Real Destructor Calls

Date: 2026-04-19
Discovered during: Retrospective seed from earlier Fuse attempts

**Reproducer**:
Generated C contained placeholder comments where destructor calls should have
been emitted.

**What was tried first**:
Track last use in analysis and leave final drop emission for later cleanup.

**Root cause**:
Drop intent and backend emission were not joined by a hard codegen contract.

**Spec gap**:
The reference did not require an observable emitted destructor call.

**Plan gap**:
Ownership and codegen work were split without an end-to-end drop proof.

**Fix**:
Attempt 6 writes the emitted `TypeName_drop(&_lN);` call into the reference and
gives W09 explicit emission tests.

**Cascading effects**:
Drop metadata flow from checker to codegen became an explicit requirement.

**Architectural lesson**:
Deterministic destruction is not implemented until the backend emits the real
call.

**Verification**:
Cross-check `docs/language-reference/fuse-language-reference.md` §14.8,
`docs/implementation/wave09_ownership_and_liveness.md`, and
`docs/audit.md` checklist C4.

### L011 — Closures Require Lifting and Environment Structs

Date: 2026-04-19
Discovered during: Retrospective seed from earlier Fuse attempts

**Reproducer**:
Closure expressions lowered to `constUnit()` or skipped liveness over captured
bodies entirely.

**What was tried first**:
Treat closures as a thin frontend surface and postpone real closure lowering.

**Root cause**:
The compiler had no explicit closure representation contract.

**Spec gap**:
The reference did not force lifted functions plus environment structs.

**Plan gap**:
Closures were not isolated as a dedicated user-visible feature wave.

**Fix**:
Attempt 6 specifies closure capture analysis, escape classification, lifting,
and callable traits explicitly and schedules them to W12.

**Cascading effects**:
Concurrency checks and ownership analysis now depend on shared closure
metadata instead of ad hoc inspection.

**Architectural lesson**:
Anonymous functions need a concrete lowered representation to be real.

**Verification**:
Cross-check `docs/language-reference/fuse-language-reference.md` §15.7,
`docs/implementation/wave12_closures_and_callable_traits.md`, and
`docs/audit.md` checklist C5.

### L012 — `spawn` and Channels Require Real Runtime Calls

Date: 2026-04-19
Discovered during: Retrospective seed from earlier Fuse attempts

**Reproducer**:
Thread and channel syntax type-checked but lowered to synchronous calls or no
observable runtime behavior.

**What was tried first**:
Ship concurrency semantics at the checker first and defer runtime bridging.

**Root cause**:
The runtime ABI and lowering boundary for concurrency were not treated as part
of the same user-visible promise.

**Spec gap**:
The reference did not make the runtime calls and observable proofs explicit
enough.

**Plan gap**:
Concurrency semantics and runtime wiring were under-connected in scheduling.

**Fix**:
Attempt 6 splits checker semantics (W07) from runtime ABI (W16) but keeps both
explicit and proof-backed.

**Cascading effects**:
`spawn`, channels, join, and `Shared[T]` now have both checker and runtime
validation paths.

**Architectural lesson**:
Concurrency is not delivered by type rules alone; it needs observable runtime
behavior.

**Verification**:
Cross-check `docs/implementation/wave07_concurrency_semantics.md`,
`docs/implementation/wave16_runtime_abi.md`, and `docs/audit.md` checklist C6.

### L013 — Structural Sign-Off Without Executable Proofs Fails Late

Date: 2026-04-19
Discovered during: Retrospective seed from earlier Fuse attempts

**Reproducer**:
Waves closed with green unit tests and plausible docs while critical features
still failed when exercised end to end.

**What was tried first**:
Use structural exit criteria and package-local tests as evidence of completion.

**Root cause**:
No one forced the feature to run as a user-visible program before sign-off.

**Spec gap**:
The reference examples were not tied tightly enough to executable proof
programs.

**Plan gap**:
The implementation plan did not require proof programs early enough.

**Fix**:
Attempt 6 starts end-to-end proof ownership at W05 and makes executable proofs
mandatory for user-visible features.

**Cascading effects**:
Audit checklist B exists specifically to block ceremonial sign-off.

**Architectural lesson**:
If a feature has not run end to end, it is still unproven.

**Verification**:
Cross-check `docs/implementation/wave05_minimal_end_to_end_spine.md`,
`docs/audit.md` checklist B, and Rule 6 proof requirements in `docs/rules.md`.

### L014 — Proof Programs Must Be Adversarial, Not Ceremonial

Date: 2026-04-19
Discovered during: Retrospective seed from earlier Fuse attempts

**Reproducer**:
Proof fixtures existed, but they would still pass if the feature were reverted
to a stub.

**What was tried first**:
Treat any running example as sufficient proof.

**Root cause**:
Proof programs were not designed to fail under the historical failure mode.

**Spec gap**:
The relationship between examples, proofs, and adversarial coverage was too
weak.

**Plan gap**:
Wave closure criteria did not require proofs to actually press the feature.

**Fix**:
Attempt 6 makes proof coverage adversarial and wires it into audit B3/B5.

**Cascading effects**:
Proof registries now record expected behavior that would fail under a stub.

**Architectural lesson**:
An example that cannot falsify a stub is documentation, not proof.

**Verification**:
Cross-check `docs/audit.md` checklist B1-B5 and the proof-program requirements
in `docs/implementation-plan.md`.

### L015 — Cross-Cutting Features Need Their Own Waves

Date: 2026-04-19
Discovered during: Retrospective seed from earlier Fuse attempts

**Reproducer**:
Features spanning checking, lowering, codegen, and proofs were hidden inside
one phase of an unrelated wave.

**What was tried first**:
Treat cross-cutting work as an implementation detail inside a larger milestone.

**Root cause**:
Planning optimized for milestone count instead of honest retirement boundaries.

**Spec gap**:
The reference named large features, but the plan did not always give them their
own closure points.

**Plan gap**:
Generic and similar cross-cutting work lacked dedicated waves.

**Fix**:
Attempt 6 gives historically dangerous features their own waves and proof
programs.

**Cascading effects**:
Audit checklist E now blocks folding cross-cutting features into umbrella
phases.

**Architectural lesson**:
When a feature spans the whole pipeline, the plan must expose that fact.

**Verification**:
Cross-check `docs/implementation-plan.md` working principle 8 and
`docs/audit.md` checklist E.

### L016 — Phase Names Must Expose Delivered Semantics

Date: 2026-04-19
Discovered during: Retrospective seed from earlier Fuse attempts

**Reproducer**:
Phases named "misc" or "cleanup" hid real unscheduled public-surface work.

**What was tried first**:
Use umbrella phase names and rely on commit history for specifics.

**Root cause**:
Phase names were treated as labels instead of delivery contracts.

**Spec gap**:
The implementation tree did not insist on semantically meaningful phase names.

**Plan gap**:
Reviewers could not tell what a phase retired by reading the plan alone.

**Fix**:
Attempt 6 requires phase names to expose the semantic contract being delivered.

**Cascading effects**:
Wave docs now have narrower review boundaries and less scope hiding.

**Architectural lesson**:
Ambiguous phase names create ambiguous accountability.

**Verification**:
Cross-check `docs/implementation-plan.md` working principle 17 and
`docs/implementation/phase-model.md` phase naming rules.

### L017 — Diagnostics Structure Is a Correctness Contract

Date: 2026-04-19
Discovered during: Retrospective seed from earlier Fuse attempts

**Reproducer**:
Errors surfaced without spans, explanations, or fix suggestions, making real
defects harder to diagnose than they should have been.

**What was tried first**:
Treat diagnostic quality as polish for a later tooling pass.

**Root cause**:
Diagnostics were not treated as part of the language promise.

**Spec gap**:
The rules did not enforce structured diagnostics strongly enough.

**Plan gap**:
No wave owned a diagnostic-quality audit across existing compiler paths.

**Fix**:
Attempt 6 turns Rule 6.17 into a correctness constraint and gives W18 a
dedicated compliance audit.

**Cascading effects**:
Later waves must preserve suggestions and spans at the CLI and LSP boundary.

**Architectural lesson**:
Bad diagnostics are not a UI defect alone; they are a correctness defect in a
developer-facing compiler.

**Verification**:
Cross-check `docs/implementation-plan.md` working principle 13,
`docs/implementation/wave18_cli_and_diagnostics.md`, and
`docs/audit.md` checklist F.

### L018 — Self-Hosting Cannot Retire Missing Language Semantics

Date: 2026-04-19
Discovered during: Retrospective seed from earlier Fuse attempts

**Reproducer**:
There was pressure to declare the language "done enough" and let self-hosting
waves finish missing semantics later.

**What was tried first**:
Treat self-hosting as proof of feature completeness.

**Root cause**:
Implementation-path milestones were confused with language-semantic retirement.

**Spec gap**:
The reference-to-wave mapping did not enforce one honest retirement wave per
feature.

**Plan gap**:
Later implementation-path waves could absorb unfinished semantics by drift.

**Fix**:
Attempt 6 states that all language and stdlib semantics must already be retired
before self-hosting begins.

**Cascading effects**:
W24 became a dedicated stub-clearance gate before W25.

**Architectural lesson**:
Self-hosting proves an implementation path, not the honesty of earlier feature
retirements.

**Verification**:
Cross-check `docs/rules.md` Rule 2.6 and
`docs/implementation/wave24_stub_clearance_gate.md`.

### L019 — Baseline Stdlib Scope Must Be Named Explicitly

Date: 2026-04-19
Discovered during: Retrospective seed from earlier Fuse attempts

**Reproducer**:
Core and hosted standard-library promises were described in broad families,
making it easy to forget specific modules.

**What was tried first**:
Use labels like "core foundations" or "hosted utilities" without enumerating
the public modules.

**Root cause**:
Stdlib scope was treated as implied rather than explicit.

**Spec gap**:
The reference named more modules than the plan clearly owned.

**Plan gap**:
Wave ownership tables were too coarse to prevent omission drift.

**Fix**:
Attempt 6 enumerates the baseline stdlib module set in the reference and the
implementation plan.

**Cascading effects**:
W20 and W22 now retire named module surfaces instead of broad umbrellas.

**Architectural lesson**:
If a standard-library module is promised publicly, it must be scheduled by
name.

**Verification**:
Cross-check `docs/implementation-plan.md` stdlib ownership tables and the
module-guide indexes under `docs/language-reference/`.

### L020 — P00, Implementation, and PCL Need Separate Commits

Date: 2026-04-19
Discovered during: Retrospective seed from earlier Fuse attempts

**Reproducer**:
Stub audit, implementation work, and wave closure landed in one commit, making
it impossible to audit the temporal order honestly.

**What was tried first**:
Collapse wave work into a single "done" commit for convenience.

**Root cause**:
Wave governance artifacts were treated as bookkeeping instead of auditable
state transitions.

**Spec gap**:
The rules did not require a separate commit sequence for P00, implementation,
and PCL.

**Plan gap**:
Wave closure discipline was under-specified for multi-phase delivery.

**Fix**:
Attempt 6 requires separate commits for P00, implementation, and PCL from W07
forward.

**Cascading effects**:
`.claude/current-wave.json` and `STUBS.md` updates are now part of auditable
phase transitions.

**Architectural lesson**:
Governance state must change in reviewable steps, not be reconstructed from one
final diff.

**Verification**:
Cross-check `docs/rules.md` commit-sequencing rule and the audit checklist's
closure-separation expectations.

### L021 — Baseline Libraries Cannot Be Deferred to `stdlib/ext`

Date: 2026-04-19
Discovered during: Retrospective seed from earlier Fuse attempts

**Reproducer**:
Application-facing baseline modules risked being relabeled as optional
extensions when scheduling got tight.

**What was tried first**:
Treat `stdlib/ext` as a release valve for modules that still felt desirable but
not immediately convenient.

**Root cause**:
The baseline/optional boundary was not protected by explicit wave ownership.

**Spec gap**:
The reference named baseline modules without a sufficiently guarded placement
rule.

**Plan gap**:
Earlier planning left room to shrink the promised baseline silently.

**Fix**:
Attempt 6 ties baseline hosted modules to W22 and reserves `stdlib/ext` for
explicitly admitted optional work in W29.

**Cascading effects**:
Repository-layout and W29 now treat `ext` as governed optional scope rather
than overflow storage.

**Architectural lesson**:
Optional namespaces must not become escape hatches for promised baseline work.

**Verification**:
Cross-check `docs/repository-layout.md`, `docs/implementation/wave22_stdlib_hosted.md`,
and `docs/implementation/wave29_targets_and_native_expansion.md`.

### L022 — Developer-Experience Surfaces Need Dedicated Waves

Date: 2026-04-19
Discovered during: Retrospective seed from earlier Fuse attempts

**Reproducer**:
Language-server, incremental compile, package tooling, and user docs were at
risk of being treated as nice-to-have cleanup after core compiler work.

**What was tried first**:
Assume DX could be filled in once the compiler was "basically done".

**Root cause**:
Developer experience was not planned as a first-class delivery axis.

**Spec gap**:
The project promise mentioned usability but did not bind it to schedule and
proof.

**Plan gap**:
No dedicated waves protected editor tooling, package management, or docs from
deferral.

**Fix**:
Attempt 6 dedicates waves to diagnostics, LSP, package management, and
ecosystem documentation.

**Cascading effects**:
DX is now part of completion criteria rather than a postscript to compiler
correctness.

**Architectural lesson**:
A modern systems language ships tooling and docs as part of the product, not as
afterthoughts.

**Verification**:
Cross-check `docs/implementation-plan.md` working principles 11 and 14 and the
dedicated W18, W19, W23, and W30 wave docs.
