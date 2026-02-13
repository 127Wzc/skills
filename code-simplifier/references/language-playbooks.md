# Language playbooks (load only when needed)

Use this file only when the repo has no clear conventions or when the user explicitly asks for language-specific standardization.
Always prefer project-specific lint/format/test rules when they exist.
Applying architecture/pattern changes (e.g. DI, interface/trait boundaries, module reshapes) requires an explicit user decision + approval.

## JavaScript / TypeScript

- **Conventions to detect**: `package.json` scripts, ESLint config, Prettier config, `tsconfig.json`, module system (ESM vs CJS).
- **Safe simplifications**: remove duplication, early returns, split long expressions, avoid nested ternaries, replace “magic numbers” with named constants + comment.
- **Common semantic traps**: `||` vs `??`, truthiness checks, `==` vs `===`, mutating shared objects, async/await ordering.
- **Architecture**: avoid moving files/exports unless requested; if a file is huge, propose an extraction plan behind a decision.
- **Naming**: prefer domain-driven names (e.g. `isUserEligible` over `checkFlag`); default to `camelCase` for vars/fns and `PascalCase` for types/components.
- **Dependency boundaries**: prefer passing dependencies/params over hardcoded global/service calls; only introduce DI patterns behind a decision.
- **Errors**: keep error context (`new Error(msg, { cause })` where available); avoid `catch (e) { return null }` unless it’s an explicit contract.
- **Tests**: identify existing tests before refactoring; if code is hard to test, propose a small refactor toward pure functions (behind a decision if it broadens scope).

## Python

- **Conventions to detect**: `pyproject.toml` (ruff/black/mypy), `setup.cfg`, `requirements*`, `pytest.ini`.
- **Safe simplifications**: flatten control flow, clarify names, extract small helpers, reduce mutation, prefer explicit exceptions and return values.
- **Common semantic traps**: default mutable args, truthiness of empty containers, exception swallowing, iterator exhaustion.
- **Architecture**: prefer local refactors; module/package reshapes require a decision.
- **Naming**: prefer domain-driven names; default to `snake_case` for vars/fns and `PascalCase` for classes.
- **Dependency boundaries**: pass collaborators explicitly (args/objects) instead of importing globals deep inside functions; avoid hidden singletons.
- **Errors**: raise specific exception types; add context in messages; don’t catch broad `Exception` unless re-raising with context.
- **Tests**: if `pytest` exists, keep refactors covered; propose small unit tests for newly-extracted pure helpers.

## Go

- **Conventions to detect**: `go.mod`, `golangci-lint` config, package layout, error-wrapping style.
- **Safe simplifications**: guard clauses, reduce nesting, small functions, consistent error returns, avoid needless interfaces.
- **Common semantic traps**: pointer aliasing, `defer` in loops, `range` variable capture, nil interfaces.
- **Naming**: prefer domain-driven names; keep exported names stable.
- **Dependency boundaries**: at external boundaries, code to small interfaces; avoid depending on concrete clients directly (introduce interfaces behind a decision).
- **Errors**: wrap with context using `%w`; avoid losing context by returning raw errors without operation info.
- **Tests**: favor table-driven tests for pure logic; don’t rewrite package structure unless chosen as a decision.

## Rust

- **Conventions to detect**: `Cargo.toml`, `rustfmt.toml`, clippy settings, workspace layout.
- **Safe simplifications**: reduce nesting with `?`, clarify ownership/borrows, prefer iterators when readable, small helpers.
- **Common semantic traps**: unintended clones, lifetime-driven refactors changing ownership, iterator consumption.
- **Naming**: prefer domain-driven names; default to `snake_case` for vars/fns and `PascalCase` for types/traits.
- **Dependency boundaries**: use traits for external boundaries when it improves testability; keep it behind a decision if it increases diff surface.
- **Errors**: preserve context (e.g. `anyhow::Context` / error variants); avoid collapsing distinct error causes.
- **Tests**: keep refactors local and covered by unit tests where present.

## Java / Kotlin

- **Conventions to detect**: `pom.xml`/`build.gradle*`, formatter/checkstyle/ktlint, package naming/layout.
- **Safe simplifications**: reduce nesting, clarify nullability, extract named methods, keep side effects obvious.
- **Architecture**: refactors that move classes/packages require a decision; prefer minimal diffs.
- **Naming**: prefer domain-driven names; default to `camelCase` vars/methods and `PascalCase` types.
- **Dependency boundaries**: prefer passing collaborators (constructors/params) over hardcoding service calls; interfaces at external boundaries behind a decision.
- **Errors**: preserve context (cause chains); avoid catching overly broad exceptions without rethrowing context.
- **Tests**: identify existing coverage (JUnit/Mockito/Kotest); propose minimal tests for risky refactors.
