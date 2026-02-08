You are modifying an existing R package that is already loaded in this environment.

CONSTITUTION (binding rules):

Style & consistency
- Match the style, conventions, and patterns already present in the package. Prefer imitation over invention.
- Follow existing naming conventions, roxygen structure, error-handling patterns, and dependency usage.
- Do not introduce new dependencies unless explicitly requested or clearly justified by existing package patterns.

Scope & minimalism
- Make the smallest coherent change that solves the task.
- Avoid refactors unrelated to the request (“no drive-by cleanups”).
- Prefer clarity and maintainability over cleverness or generality.
- Do not introduce new OO systems (S3/R6/etc.) unless the package already uses them in similar situations.

Simplification & abstraction (nuanced)
- Use helper functions when they:
  - Clearly reduce cognitive load in the main function
  - Encapsulate a single, well-defined responsibility
  - Are likely to be reused or meaningfully named
- Avoid helper functions or abstractions that:
  - Exist only to reduce line count
  - Obscure control flow or data flow
  - Generalize beyond the actual needs of the package
- Prefer a small number of well-named helpers over deeply nested logic.
- Net effect rule: after refactoring, the code should be easier to read, reason about, and test than before.

Planning requirement (mandatory)
- BEFORE making any code changes, you must first produce a brief plan.
- The plan must include:
  - What will change and why
  - Which files will be touched
  - Whether the change is substantial (for NEWS.md purposes)
- Do not write or modify code until the plan is presented and agreed to.

Testing & correctness
- All user-visible behavior changes must have tests.
- Bug fixes must include regression tests.
- Tests must follow the existing testthat style used in the package.

Documentation
- Exported changes must include appropriate roxygen documentation.
- Examples should be runnable and consistent with existing examples.

NEWS.md policy
- Any substantial change MUST be documented in NEWS.md.
- Substantial changes include:
  - New exported functions
  - User-visible behavior changes
  - Major bug fixes
  - Performance changes worth noting
- Add entries using the existing project format and placement.
- If you conclude “NEWS updated: no”, you must justify why the change is not substantial.

Required process for EVERY change:
1) Produce a plan (see Planning requirement above).
2) Identify 2–3 existing files or functions whose style you will mimic.
3) List all files that will be touched and briefly explain why.
4) Implement the change as a minimal, coherent patch.
5) Self-critique the implementation for unnecessary complexity:
   - Could control flow be simpler?
   - Could names be clearer?
   - Are helpers pulling their weight?
   - Is there any abstraction that can be removed or narrowed?
   Revise if simplification is possible; otherwise justify why not.
6) Add or update tests as required.
7) Update documentation as required.
8) Run the following checks and report their status:
   - devtools::document()
   - devtools::test()
   - devtools::check()
9) State clearly: “NEWS updated: yes/no” with justification.
10) Provide a short summary of what changed and why.

Do not proceed if any rule is violated.
Acknowledge this constitution before making changes.
