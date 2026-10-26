# DataSynth

Deterministic synthetic datasets for demos, fixtures, and bounded load tests.

> Planning repository — documentation and Git initialization only. No application, package, solution, database, tests, CI workflow, or runnable demo has been implemented yet. Everything below describes the proposed implementation, unless explicitly marked as verified.

## Goals

Generate repeatable structured data with explicit constraints and controlled anomalies so portfolio projects can demonstrate meaningful behavior without copying real personal records.

## Target users

Developers writing integration tests, preparing demos, or measuring performance with reproducible datasets.

## MVP scope

- Seeded generators for constrained records, timestamps, identifiers, and numeric distributions.
- Composable schemas with field dependencies and validation.
- Telemetry series with trends, noise, gaps, and out-of-order arrival profiles.
- Streaming JSONL and CSV export with metadata describing seed, generator version, and schema.
- CLI profiles for general telemetry, trace fixtures, and job payloads.

## Stretch goals

- Correlated multivariate series and configurable burst processes.
- Scenario packs for fault injection and regression datasets.
- Cross-language reproducibility only after a fully specified portable generator contract.

## Proposed tech stack

C# / .NET 10 library and CLI, System.Text.Json, xUnit with statistical/property tests, BenchmarkDotNet. Avoid heavy data-science dependencies for the initial release.

Select compatible stable dependency versions during milestone 1, record them in project files, and pin the SDK/toolchain after the compatibility spike. No packages or development workloads are installed by this documentation task.

## Architecture

Core owns seeded random streams and primitive distributions. Schema generators compose validated field dependencies; scenario profiles apply domain-neutral patterns. Writers stream records with cancellation and bounded buffers. CLI parses profiles and delegates to the same public library. Separate random streams by named component so adding a field need not perturb unrelated generators.

## Proposed folder structure

Only README.md and .gitignore exist initially, alongside Git metadata. The following tree is a plan; create directories when their implementation begins.

```text
DataSynth/
  README.md
  .gitignore
  DataSynth.slnx
  src/DataSynth/
  src/DataSynth.Cli/
  tests/DataSynth.UnitTests/
  tests/DataSynth.ScenarioTests/
  benchmarks/DataSynth.Benchmarks/
  samples/profiles/
  docs/reproducibility/
```

## Data models and invariants

GenerationProfile(Id, Version, Seed, Count, Schema); FieldSpec(Name, Type, Distribution, Constraints); AnomalyProfile(GapRate, DuplicateRate, OutOfOrderWindow); DatasetManifest(GeneratorVersion, PrngId, Seed, ProfileHash, RecordCount, OutputHash). Declare time origin and numeric formatting rather than using wall-clock defaults.

## Proposed API and contracts

Proposed API: Dataset.Generate<T>(profile, seed) returns IAsyncEnumerable<T>; TelemetryScenario.Generate(spec) yields sample records; DatasetWriter.WriteJsonLinesAsync(records, stream, cancellationToken) streams output. CLI proposal: generate --profile <json> --seed <n> --count <n> --output <path>.

Version persisted/exported formats from their first implementation. Validate at boundaries and return structured errors rather than leaking internal exceptions. The contracts above remain proposals until backed by code and contract tests.

## Implementation milestones

1. Specify PRNG, seed splitting, output formats, and reproducibility guarantees.
2. Implement primitives, constrained schemas, and streaming writers.
3. Add telemetry/trace/job profiles and anomaly manifests.
4. Publish benchmark results and package/CLI artifacts with stable fixture checksums.

Each milestone should end with a small demonstrable result, a written acceptance check, and tests for its important failure modes. Work through a vertical slice before expanding the stretch scope.

## Testing and CI plan

Use golden fixtures for identical version/seed/profile combinations, constraint/property tests, and distribution checks with documented tolerances. Test cancellation, output escaping, large counts with bounded memory, invalid cyclic field dependencies, and CSV round trips. CI builds/tests on Windows/Linux and compares fixture hashes across platforms before claiming portability.

CI is planned, not configured. Once a solution exists, add .github/workflows/ci.yml with read-only default permissions, explicit toolchain setup, dependency restore, build, and the checks above. Keep ordinary CI offline from real services and hardware. Store test reports as artifacts; do not publish packages, create releases, or deploy from the first workflow. Record hardware/manual results separately from automated passes.

## Setup and development

### Current checkout

Open this README in a Markdown viewer. Git is initialized locally; no build or run step exists yet. Check the working tree with `git status --short --branch`. Documentation files start untracked; no initial commit or remote is created.

### Prerequisites and first implementation

After scaffolding, create a dependency-light library and CLI. Define one small profile with a fixed timestamp origin; generate it twice and compare manifests/hashes. Keep generated large datasets under ignored data/generated and commit only intentionally small fixtures.

Install only the dependencies needed for the selected milestone. Add a checked-in sample configuration with harmless defaults; keep local databases, recordings, generated datasets, credentials, and device identifiers outside version control. Pick a license before public distribution; no license is implied by this plan.

### Future commands

These commands are examples for after the referenced projects/files have been created; they will not build this documentation-only repository.

```powershell
dotnet restore DataSynth.slnx
dotnet test DataSynth.slnx
dotnet run --project src/DataSynth.Cli -- generate --profile samples/profiles/telemetry.json --seed 42 --count 1000 --output data/generated/telemetry.jsonl
```

## Supporting libraries and boundaries

Supplies fixtures for SignalBench, TraceNest, PulseQueue, FlowForge, and AutoDiag's simulator. Domain-specific semantic validation remains in consumers; DataSynth should not depend on those applications.

Consume stable library versions through local package artifacts initially. Avoid absolute ProjectReference paths into sibling repositories and avoid Git submodules for the MVP. Keep a consumer's first vertical slice runnable using a small local interface or fixture while a supporting package is under development.

## Realistic limitations

Synthetic records are not automatically representative or anonymous if profiles are fitted to real data. Reproducibility is guaranteed only within a documented generator/profile version and supported environment matrix. It is not a cryptographically secure random generator or a production traffic attack tool.

Scope and performance targets are hypotheses until measured. Document reproducible dataset sizes, hardware, dependency versions, and failure cases alongside any benchmark claim. Do not present planned capabilities as shipped features.

## Documentation provenance

Prepared 2026-10-23. The previous chat supplied the project names and the AutoDiag platform change, but its detailed assistant responses were unavailable content references. The user confirmed the seven proposed application themes in this session; individual designs here are newly proposed plans, not a recovered transcript.

