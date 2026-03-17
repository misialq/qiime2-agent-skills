---
name: qiime2-unit-test-writer
description: Write or update unit tests for QIIME 2 plugin code. Use when Codex needs to add tests for a module, method, function, pipeline, helper, or regression in a QIIME 2 plugin repo, especially when the repo expects TestPluginBase, reusable fixtures under tests/data, patched external APIs or command-line tools, and targeted pytest runs in a conda environment such as moshpit-dev.
---

# QIIME 2 Unit Test Writer

## Overview

Add or update unit tests for QIIME 2 plugin code by mirroring the local repo's test style instead of writing generic pytest tests. Inspect the nearest existing tests first, then use [references/qiime2-test-patterns.md](references/qiime2-test-patterns.md) to match the conventions captured from `q2-annotate`.

## Workflow

1. Inspect the code under test, the nearest existing `test_*.py` files, the package's `tests/data/` directory, and the repo's test command before editing.
2. Prefer extending an existing package-specific test module. Create a new `test_*.py` only when there is no natural home for the new coverage.
3. Subclass test classes from `qiime2.plugin.testing.TestPluginBase`. Set `package = "<package_path>.tests"` so `self.get_data_path(...)` resolves fixtures.
4. Reuse or add stable fixture files under the matching `tests/data/` directory. Treat reusable files as the default for inputs and expected outputs.
5. Patch every external boundary:
   - Patch command execution such as `subprocess.run` or the repo's command wrapper.
   - Patch network or remote API calls such as `requests.get`.
   - Patch plugin actions, renderers, copy helpers, or filesystem side effects when the code under test delegates to them.
   - Patch at the lookup site used by the module under test, not at an arbitrary import path.
6. Assert behavior precisely. Prefer explicit command-list assertions, `assert_has_calls`, output type checks, dataframe equality checks, existence checks for materialized files, and `assertRaisesRegex` for failure paths.
7. Keep tests narrow. Cover the requested behavior, one or more edge cases, and at least one failure path when the code wraps an external tool or remote call.
8. Run the narrowest relevant pytest selection in the `moshpit-dev` conda environment unless the user specifies a different environment.

## Rules

- Use `TestPluginBase` for unit tests in this style, even for helper-focused tests, unless the user explicitly asks to follow a different house style.
- Use `self.get_data_path(...)` instead of hard-coding fixture paths.
- Prefer `setUp` or `setUpClass` for shared fixture loading and plugin-action mocks.
- Use `self.plugin.methods[...]` or `self.plugin.pipelines[...]` when testing registered plugin actions instead of reimplementing plugin wiring.
- Never create input fixture content on the fly inside a test when a reusable file in `tests/data/` can represent it.
- Use temporary directories only for outputs, staging, or code paths that inherently require a temporary working directory.
- Never hit the network or execute real external tools in unit tests.
- Preserve the module's local naming, import style, and assertion style when it already matches the surrounding repo.

## Test Execution

- Default iteration command: `conda run -n moshpit-dev python -m pytest path/to/test_file.py`
- Single-test command: `conda run -n moshpit-dev python -m pytest path/to/test_file.py::TestClass::test_name`
- If the repo exposes `py.test`, `pytest`, or `make test`, still prefer a targeted direct pytest invocation while iterating.

## References

- Read [references/qiime2-test-patterns.md](references/qiime2-test-patterns.md) before writing tests. It points at concrete `q2-annotate` examples and highlights patterns to copy versus legacy patterns to avoid.
