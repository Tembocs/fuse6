# Wave 00: Governance and Phase Model

> Part of the [Fuse implementation plan](../implementation-plan.md).


Goal: establish the repository, module, build, test, tooling, documentation,
and governance foundations required for disciplined compiler work. Seed
`STUBS.md` with the initial stub inventory covering every unimplemented
language feature.

Entry criterion: none.

State on entry: only the foundational docs exist. The repository may already
contain pre-bootstrap coordination artifacts (`STUBS.md`,
`.claude/current-wave.json`) in seed form, but there is no Go module, no CI,
and no compiler packages.

Exit criteria:

- the project-owned build roundtrip succeeds from a clean checkout
- `go test ./...` succeeds on the initial package set
- CI runs on every push and PR for Linux, macOS, and Windows
- the foundational docs exist and are readable
- `STUBS.md` exists with a seeded Active stubs table covering every
  user-visible language feature not yet implemented, and an empty Stub
  history section
- `.claude/current-wave.json` names the active wave for coordination

Proof of completion:

```
go run tools/checkartifacts/main.go -build-roundtrip
go test ./...
go run tools/checkstubs/main.go
go run tools/checkgov/main.go -current-wave
```

## Phase 00: Stub Audit [W00-P00-STUB-AUDIT]

- Task 01: Install bootstrap `tools/checkstubs` validator
  [W00-P00-T01-CHECKSTUBS-BOOTSTRAP]
  Currently: stub-audit verification cannot rely on a tool that does not yet
  exist.
  DoD: a minimal `tools/checkstubs` validator exists and can verify the
  pre-bootstrap seed state as well as the first code-backed STUBS inventory.
  Verify: `go test ./tools/checkstubs/... -v`
- Task 02: Initialize or refresh STUBS.md seed state [W00-P00-T02-INIT-STUBS]
  Currently: the repository may contain only the pre-bootstrap seed state.
  DoD: STUBS.md exists. If the repository is still docs-only, it says so
  explicitly. Once the Wave 00 package skeleton exists, the Active stubs table
  is refreshed to the first code-backed inventory before W00 closes.
  Verify: `go run tools/checkstubs/main.go -audit-seed`

## Phase 01: Repository Initialization [W00-P01-REPO-INIT]

- Task 01: Create repository skeleton [W00-P01-T01-REPO-SKELETON]
  Verify: `go run tools/checklayout/main.go`
- Task 02: Add foundational docs [W00-P01-T02-FOUNDATIONAL-DOCS]
  Verify: `go run tools/checkdocs/main.go -foundational`
- Task 03: Artifact policy [W00-P01-T03-ARTIFACT-POLICY]
  Verify: `go run tools/checkartifacts/main.go`

## Phase 02: Go Module and Build Scaffold [W00-P02-GO-MODULE]

- Task 01: Initialize Go module [W00-P02-T01-GO-MOD]
  Verify: `go build ./...`
- Task 02: Create package stubs [W00-P02-T02-PACKAGE-STUBS]
  Verify: `go build ./compiler/...`
- Task 03: Create Stage 1 CLI stub [W00-P02-T03-CLI-STUB]
  Verify: `go test ./cmd/fuse/... -run TestCliStub -v`

## Phase 03: Build and CI Baseline [W00-P03-BUILD-CI]

- Task 01: Author Makefile targets [W00-P03-T01-MAKEFILE]
  Verify: `go run tools/checkartifacts/main.go -build-roundtrip`
- Task 02: Add CI matrix [W00-P03-T02-CI-MATRIX]
  Verify: `go run tools/checkci/main.go`
- Task 03: Add golden harness [W00-P03-T03-GOLDEN-HARNESS]
  Verify: `go test ./tools/goldens/... -run TestGoldenStability -count=2 -v`

## Phase 04: Governance Artifacts [W00-P04-GOVERNANCE]

- Task 01: Initialize `.claude/current-wave.json`
  [W00-P04-T01-CURRENT-WAVE-FILE]
  Verify: `go run tools/checkgov/main.go -current-wave`
- Task 02: Document phase model [W00-P04-T02-PHASE-MODEL-DOC]
  Verify: `go run tools/checkdocs/main.go -phase-model`

## Wave Closure Phase [W00-PCL-WAVE-CLOSURE]

- Task 01: Update STUBS.md stub history [W00-PCL-T01-HISTORY]
  Verify: `go run tools/checkstubs/main.go -history-current-wave W00`
- Task 02: Write WC000 learning-log entry [W00-PCL-T02-CLOSURE-LOG]
  Verify: `go run tools/checkgov/main.go -wc-entry WC000`

