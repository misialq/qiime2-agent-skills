# QIIME 2 Test Patterns

Use this file after inspecting the target repo. These examples come from `q2-annotate` and should anchor the default shape of new tests.

## Patterns to Copy

### `TestPluginBase` scaffold

- Subclass from `qiime2.plugin.testing.TestPluginBase`.
- Set `package = "...tests"` on the class.
- Use `self.get_data_path(...)` to locate fixtures.

Examples:

- `q2_annotate/prodigal/tests/test_prodigal.py`
- `q2_annotate/kaiju/tests/test_classification.py`
- `q2_annotate/tests/test_utils.py`
- `q2_annotate/kraken2/tests/test_filter.py`

### Reusable fixture files in `tests/data`

- Store reusable sequence files, tables, reports, metadata, JSON, and expected outputs under the package-local `tests/data/` directory.
- Reuse existing fixtures before adding new ones.
- When new coverage needs new inputs, add stable files to `tests/data/` instead of building strings, archives, or random payloads inside the test body.
- Use temporary directories only for outputs or staging areas created by the code under test.

Examples:

- `q2_annotate/prodigal/tests/data/`
- `q2_annotate/kaiju/tests/data/`
- `q2_annotate/kraken2/tests/data/`
- `q2_annotate/busco/tests/data/`

### Patch external boundaries

- Patch command execution for CLI-backed code.
- Patch network calls for download helpers.
- Patch helper functions or plugin actions when the test should isolate orchestration logic.
- Patch at the lookup site used by the implementation module.

Examples:

- `@patch("subprocess.run")` in `q2_annotate/prodigal/tests/test_prodigal.py`
- `@patch("q2_annotate.kraken2.database.run_command")` in `q2_annotate/kraken2/tests/test_database.py`
- `@patch("requests.get")` in `q2_annotate/kraken2/tests/test_database.py`
- `@patch("q2_annotate.kraken2.collapse.q2templates.render")` in `q2_annotate/kraken2/tests/test_collapse.py`

### Assert exact behavior

- Assert exact command lists with `assert_called_once_with`.
- Assert multiple calls with `assert_has_calls`.
- Assert output object types with `assertIsInstance`.
- Assert dataframes with `pandas.testing.assert_frame_equal` or similar helpers.
- Assert error paths with `assertRaisesRegex`.
- Assert materialized files with explicit `os.path.exists(...)` checks when the behavior is filesystem-oriented.

Examples:

- Command assertions: `q2_annotate/prodigal/tests/test_prodigal.py`
- Error-path assertions: `q2_annotate/kraken2/tests/test_database.py`
- Dataframe assertions: `q2_annotate/kaiju/tests/test_classification.py`, `q2_annotate/tests/test_utils.py`
- File existence assertions: `q2_annotate/kraken2/tests/test_filter.py`

### Reuse plugin wiring instead of faking it

- Use `self.plugin.methods[...]` or `self.plugin.pipelines[...]` when the registered plugin entrypoint is what matters.
- Mock downstream actions or context methods to isolate pipeline orchestration.

Examples:

- `self.plugin.pipelines["multiply_tables"]` in `q2_annotate/tests/test_utils.py`
- `qiime2.plugins.annotate` action mocking in `q2_annotate/kaiju/tests/test_classification.py`

## Patterns to Avoid

- Do not write plain pytest function tests when the surrounding package uses `TestPluginBase`.
- Do not create inline input fixtures when a reusable file under `tests/data/` would work.
- Do not let unit tests hit real executables, remote services, or external databases.
- Do not patch the wrong import path. Patch the name looked up by the module under test.
- Do not broaden scope into end-to-end plugin behavior if the request is for unit coverage.

## Legacy Exceptions

Some older tests in `q2-annotate` create temporary content in `setUp`, such as ad hoc tar archives in `q2_annotate/kraken2/tests/test_database.py`. Treat those as legacy exceptions, not the default pattern. Prefer reusable fixture files unless the code path is inherently about temporary output generation.

## Execution Defaults

- Default environment: `moshpit-dev`
- Default command: `conda run -n moshpit-dev python -m pytest path/to/test_file.py`
- Narrow iteration first, widen to package or full-suite runs only after the target tests pass.
