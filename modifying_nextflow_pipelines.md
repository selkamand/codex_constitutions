# Minimal AI Constitution for Nextflow DSL2 (Strict Syntax / Parser v2)

**Scope:** All pipeline development and review must target **Nextflow DSL2** running with **strict syntax (syntax parser v2)**.  
Strict syntax is the default in modern Nextflow releases and must be treated as mandatory.

---

# 1. Core Principles

1. **Reproducibility is mandatory**
   - Pin Nextflow version via `manifest.nextflowVersion`.
   - Pin tool versions (container digest, conda lock, or module version).
   - No unversioned dependencies.

2. **Strict syntax v2 compliance is required**
   - Code must parse cleanly with `NXF_SYNTAX_PARSER=v2`.
   - Any construct removed or restricted under strict syntax is forbidden.

3. **Dataflow clarity over Groovy cleverness**
   - Pipelines are dataflow graphs.
   - Processes communicate only through channels.
   - Avoid implicit behavior and side effects.

---

# 2. Absolute Rules (Strict Syntax v2)

The following are **not allowed**:

- `import` statements in pipeline scripts.
- Custom `class` definitions in scripts (move to `lib/` if needed).
- Mixing top-level statements with declarations.
- `for` / `while` loops (use `.map`, `.filter`, `.collect`, etc.).
- `switch` statements (use `if / else if / else`).
- Spread operator (`*list`).
- Assignment inside expressions (`foo(x = 1)`).
- Post/pre increment or decrement in expressions (`x++`, `--y`).
- Implicit environment variable interpolation like `"${PWD}"`  
  → must use `env('PWD')`.
- `include ... addParams(...)` or `include ... params(...)`.

Restricted behavior:

- Variables must be declared with `def` (no typed variable declarations).
- Slashy strings cannot be interpolated.
- Only explicit casts (`as Type`) are allowed.
- Closure parameters must be explicit (avoid implicit `it`).
- Prefer `channel.of()` over `Channel.of()`.

---

# 3. Parameter & Version Policy

1. **Declare minimum Nextflow version**
   ```groovy
   manifest {
       nextflowVersion = '>=25.04.0'
   }
   ```

2. **Do not rely on legacy CLI type coercion**
   - Explicitly validate and coerce params.
   - Use early validation and fail fast.

3. **Validate critical params**
   - Required paths must exist.
   - Enums must be checked against allowed sets.
   - Identifiers must match safe regex.

---

# 4. Workflow Architecture Rules

1. **Use DSL2 modular structure**
   - Processes in modules.
   - Complex chains grouped into subworkflows.
   - No hidden global `params` usage inside modules.

2. **Optional logic must use workflow-level `if`**
   - Conditional chains are implemented as subworkflows.
   - Avoid scattering `when:` unless per-sample gating is required.

3. **Channel contracts are explicit**
   - Treat tuple structure as a formal contract.
   - Use `(meta, ...)` patterns for stability.
   - Reordering must happen in one controlled `.map {}` block.

4. **Processes never “return values”**
   - They emit files or `val` outputs.
   - Downstream logic operates on channels only.

---

# 5. Data Integrity Rules

1. **No reliance on tuple position folklore**
   - Document channel schemas.
   - Avoid long positional tuples where possible.

2. **Single responsibility processes**
   - One logical task per process.
   - No mixed compute + decision logic.

3. **Deterministic outputs**
   - Stable file naming.
   - Explicit `publish` / `output` declarations.
   - No hidden workdir dependencies.

---

# 6. Review Checklist (Mandatory)

Every change must confirm:

- ✅ Strict syntax v2 compliance
- ✅ No banned constructs introduced
- ✅ Channel schema unchanged or clearly version-bumped
- ✅ Tool versions pinned
- ✅ Nextflow minimum version correct
- ✅ Parameter validation updated if needed
- ✅ Optional branches implemented via subworkflow logic

---

# 7. CI Enforcement

CI must:

- Run with `NXF_SYNTAX_PARSER=v2`
- Execute `nextflow lint`
- Execute at least one minimal end-to-end test dataset
- Fail on syntax warnings related to strict mode

---

# 8. Upgrade Policy

When upgrading Nextflow:

1. Enable strict syntax if not already default.
2. Remove deprecated constructs immediately.
3. Bump `manifest.nextflowVersion`.
4. Re-run validation suite before release.

---

# 9. Non-Negotiable Standard

If code:
- Relies on Groovy behavior not guaranteed in strict syntax
- Depends on implicit typing
- Uses removed constructs
- Or violates channel contract clarity

It must be rejected and rewritten.

---

This constitution ensures:
- Forward compatibility with strict syntax
- Stable DSL2 modular pipelines
- Reproducibility across environments
- Minimal technical debt accumulation
- Explicit handling of all syntax v1 → v2 differences
