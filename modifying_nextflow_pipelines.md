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

---

# 8. CI Enforcement

CI must:

- Run with `NXF_SYNTAX_PARSER=v2`
- Run `nextflow lint`
- Execute minimal test dataset
- Fail on syntax warnings
- Fail if `publishDir` appears inside any process block
- Fail if `${params.outdir}` appears inside any `script:` block

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

---

# Final Guarantee

Under this constitution:

- Processes remain pure
- Publishing is centralized
- Strict syntax compliance is enforced
- Side effects are eliminated
- Modular reuse is preserved
