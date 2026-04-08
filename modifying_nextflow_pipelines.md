# Minimal AI Constitution for Nextflow DSL2  
## Strict Syntax v2 • Modular • Reproducible • Side-Effect Controlled

---

# 0. Scope

All pipeline development and review must target:

- **Nextflow DSL2**
- **Strict syntax (parser v2)**
- Modern stable Nextflow (≥ 25.04.0 unless otherwise pinned)

Strict syntax is mandatory and assumed enabled in CI.

---

# 1. Core Principles

## 1.1 Reproducibility is mandatory
- `manifest.nextflowVersion` must be pinned.
- Containers must use immutable digests (not floating tags).
- Conda environments must be lock-resolved.
- No unpinned tool versions.

## 1.2 Strict syntax v2 compliance is required
- Code must parse cleanly with `NXF_SYNTAX_PARSER=v2`.
- Deprecated constructs are forbidden.
- No backward-compatibility hacks.

## 1.3 Processes are pure functions
A process:
- Takes inputs via channels
- Produces declared outputs
- Has **no external side effects**
- Does not manage final output layout

## 1.4 Workflow controls orchestration and publishing
- All data routing, conditional logic, and publishing decisions occur at workflow scope.
- Modules must be location-agnostic and reusable.

---

# 2. Absolute Syntax Rules (Strict Parser v2)

The following are **forbidden**:

- `import` statements in pipeline scripts
- Custom `class` definitions in scripts
- `for` / `while` loops (use channel operators)
- `switch` statements
- Spread operator (`*list`)
- Assignment inside expressions
- `x++` / `--y`
- Implicit closure parameter `it`
- Slashy string interpolation
- `include ... addParams(...)`
- Mixing top-level logic with declarations
- Implicit Groovy behavior relied upon for channel operations
- Implicit type coercion assumptions
- Use of `Channel.<operators>` - should be `channel.<operators>` (lowercase)

Variables must:
- Be declared using `def`
- Use explicit casts (`as Type`) when required

---

# 3. Parameter & Validation Policy

## 3.1 Version pinning required

```groovy
manifest {
    nextflowVersion = '>=25.04.0'
}
```

## 3.2 All critical params must be validated

Required:
- Existence checks for file paths
- Regex validation for identifiers
- Enum validation for allowed values
- Explicit type coercion

Fail fast. Never defer parameter errors into process runtime.

---

# 4. Workflow Architecture Rules

## 4.1 DSL2 modular structure required
- Processes live in modules when reusable
- Subworkflows encapsulate multi-step logic
- No hidden global `params` usage inside modules

## 4.2 Channel contracts are formal APIs
- Tuple schemas must be documented
- Reordering must occur in one controlled `.map {}` block
- Avoid long positional tuples without schema comment

## 4.3 Optional logic lives at workflow scope
- Use `if` at workflow level
- Avoid per-process `when:` unless strictly per-sample

## 4.4 Processes never return values
- They emit declared outputs only
- All downstream logic operates on channels

---

# 5. Publishing & Output Contract (Non-Negotiable)

## 5.1 publishDir is forbidden inside processes

The following is permanently banned:

```groovy
process X {
    publishDir ...
}
```

Rationale:
- Breaks modularity
- Hard-codes output paths
- Introduces hidden side effects
- Violates functional purity

## 5.2 Processes must be side-effect free

Inside `script:` blocks, the following are forbidden:

- `cp` / `mv` / `rsync` to `${params.outdir}`
- Writing outside task working directory
- Referring to global output paths

A process may only write:
- Declared output files
- Within its task work directory

## 5.3 Publishing must occur centrally

Allowed patterns:

### Pattern A — Workflow output block

```groovy
output {
    qc_results {
        path "${params.outdir}/qc"
        mode 'copy'
    }
}
```

### Pattern B — workflow `publish:` block

```groovy
workflow {
    publish:
    qc = qc_ch
}
```

### Pattern C — config-driven publishing

In `nextflow.config`:

```groovy
process {
  withName: QC_ALIGNED_BAM {
    publishDir = "${params.outdir}/qc"
  }
}
```

(If config-driven publishing is used, it must be centralized and documented.)

---

# 6. Deterministic Output Rules

- File names must be stable and reproducible.
- No random suffixes.
- No reliance on workdir naming.
- Directories should not be emitted unless unavoidable.
- Prefer emitting explicit file lists or manifests.

---

# 7. Review Checklist (Mandatory)

Every change must confirm:

- ✅ Strict syntax v2 compliance
- ✅ No `publishDir` in any process
- ✅ No external side-effects in `script:` blocks
- ✅ Publishing occurs only at workflow/config scope
- ✅ Channel schemas preserved or versioned
- ✅ Tool versions pinned
- ✅ Required params validated
- ✅ No forbidden Groovy constructs
- ✅ No reliance on implicit coercion
- ✅ A `test` profile exists
- ✅ At least one supported runtime profile exists
- ✅ CI executes `nextflow run . -profile <runtime>,test`
- ✅ Runtime detection logic is config-aware and does not invent profiles

---

# 8. CI Enforcement

CI must:

- Run with `NXF_SYNTAX_PARSER=v2`
- Run `nextflow lint`
- Execute a minimal test dataset
- Fail on syntax warnings
- Fail if `publishDir` appears inside any process block
- Fail if `${params.outdir}` appears inside any `script:` block

## 8.1 Test execution profile policy

Pipeline tests must execute the pipeline from the repository root using:

```bash
nextflow run . -profile <runtime>,test
```

Where:

- `<runtime>` is a real execution profile such as `docker`, `singularity`, `apptainer`, `podman`, `conda`, or another explicitly supported runtime profile
- `test` is the dedicated lightweight test profile
- The runtime profile must be combined with `test`, not used instead of it

Examples:

```bash
nextflow run . -profile docker,test
nextflow run . -profile singularity,test
nextflow run . -profile apptainer,test
```

## 8.2 Runtime profile discovery must be automatic

CI must not hard-code exactly one runtime unless the repository supports only one.

Instead, CI/test logic must inspect repository configuration and determine which runtime profiles are actually available. Discovery must examine, as applicable:

- `nextflow.config`
- included config files
- `conf/*.config`
- profile blocks declared in the pipeline
- profile documentation if used as the canonical support declaration

The test runner must:

1. Detect supported runtime profiles
2. Prefer containerized runtimes over host execution
3. Run at least one valid `<runtime>,test` combination
4. Run multiple runtime tests when the project explicitly supports multiple runtimes and CI resources allow it

## 8.3 Required runtime selection order

When multiple supported runtime profiles are present, CI should prefer them in this order unless the repository documents a different required precedence:

1. `docker`
2. `singularity`
3. `apptainer`
4. `podman`
5. `conda`

If none of the above are defined, CI must fail with a configuration error unless the constitution explicitly permits another named runtime profile.

## 8.4 The `test` profile is mandatory

Every pipeline must define a `test` profile.

The `test` profile must:

- use a minimal deterministic dataset
- minimize runtime and resource usage
- avoid network dependence where possible
- preserve full pipeline validity
- be compatible with each supported runtime profile unless explicitly documented otherwise

A pipeline is non-compliant if it requires ad hoc CLI flags in CI instead of a proper `test` profile.

## 8.5 Profile composition rules

Profiles must be composable.

Required:

- runtime concerns belong in runtime profiles
- test-input/resource overrides belong in the `test` profile
- orchestration/test logic must not be duplicated across runtimes

Forbidden:

- separate one-off profiles like `docker_test`, `singularity_test`, unless unavoidable and justified
- test logic embedded directly into runtime profiles
- CI commands that bypass the standard profile contract

Preferred pattern:

```groovy
profiles {
    docker {
        docker.enabled = true
    }
    singularity {
        singularity.enabled = true
    }
    test {
        params.input = 'tests/data/*'
        params.max_samples = 2
    }
}
```

## 8.6 Test command discovery must be config-aware

CI wrappers/scripts may inspect config to choose valid commands, but they must not silently guess unsupported profile names.

Allowed behavior:

- parse declared profile names
- verify that `test` exists
- verify that at least one supported runtime exists
- construct commands of the form `nextflow run . -profile <runtime>,test`

Forbidden behavior:

- running `nextflow run . -profile test` alone when a runtime profile exists
- inventing undeclared profile names
- falling back to ambient local tools as a substitute for a declared runtime profile

## 8.7 Minimum CI matrix expectation

At minimum, CI must execute one successful profile combination:

```bash
nextflow run . -profile <detected-runtime>,test
```

If the repository explicitly claims support for multiple runtimes, CI should validate more than one:

```bash
nextflow run . -profile docker,test
nextflow run . -profile singularity,test
```

A claimed runtime that is never exercised in CI should be treated as lower trust and may be removed from support documentation.

---

# 9. Upgrade Policy

When upgrading Nextflow:

1. Confirm strict syntax default behavior
2. Remove deprecated constructs immediately
3. Bump `manifest.nextflowVersion`
4. Run validation suite
5. Verify publishing still centralized

---

# 10. Non-Negotiable Standard

Code must be rejected if it:

- Uses `publishDir` inside processes
- Introduces filesystem side effects
- Relies on implicit Groovy behavior
- Breaks channel contract clarity
- Violates strict syntax v2 constraints
- Mixes orchestration and execution concerns
- Lacks a `test` profile
- Cannot be tested via `nextflow run . -profile <runtime>,test`
- Declares runtime support that CI never validates

---

# Final Guarantee

Under this constitution:

- Processes remain pure
- Publishing is centralized
- Strict syntax compliance is enforced
- Side effects are eliminated
- Modular reuse is preserved
