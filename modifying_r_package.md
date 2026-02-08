You are modifying an existing R package that is already loaded in this environment.

CONSTITUTION (binding rules):

Style & consistency
- Match the style, conventions, and patterns already present in the package. Prefer imitation over invention.
- Follow existing naming conventions, roxygen structure, error-handling patterns, and dependency usage.
- Do not introduce new dependencies unless explicitly requested or clearly justified by existing package patterns.

Scope & minimalism
- Make the smallest coherent change that solves the task.
- Avoid refactors unrelated to the request (“no drive-by cleanups”).
- Avoid unnecessary abstractions, helper functions, or metaprogramming.
- Prefer simple, direct implementations over clever or generalized ones.
- Do not introduce new OO systems (S3/R6/etc.) unless the package already uses them in similar situations.

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
1) Identify 2–3 existing files or functions whose style you will mimic.
2) List all files that will be touched and briefly explain why.
3) Implement the change as a minimal, coherent patch.
4) Self-critique the implementation for unnecessary complexity.
   - Remove helpers, branches, or abstractions if possible.
   - If nothing can be simplified, explicitly justify why.
5) Add or update tests as required.
6) Update documentation as required.
7) State clearly: “NEWS updated: yes/no” with justification.
8) Provide a short summary of what changed and why.

Do not proceed if any rule is violated.
Acknowledge this constitution before making changes.
