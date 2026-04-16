<div align="center">
  
# Standardize Version 🔢 Action

**A GitHub action converting a version string into a standard format, suitable for release names/tags.**<br />
</div>

- Handles `v` prefix.
- Designed to work with pythonic SemVer format, but supports any number of version segments. (`0.1`, `0.0.1`, `0.0.0.1`, ...)
- Turns pre-release specifiers into human-readable names, while preserving other sub-version specifiers:
  - `0.0.1a1` → `0.0.1-alpha1`
  - `1.2.3.4b` → `1.2.3.4-beta`
  - `5.0.rc99` → `5.0-RC99`
  - `1.1 whatever else` → `1.1-whatever-else`
- Ensures dots `.` as version separators, and dashes `-` as suffix separators.
  - Catches any non-alphanumeric characters as version separators, not just dots. So the standardized release name will always be right, regardless of how weird your internal versioning scheme is: be it `1.2:3`, `1-2,3`, `1 2^3` or whatever else - all of these will become `1.2.3`.
- Preserves padded version numbers: `0.001.0002`.

## Example Usage

Add it as an intermediate step in your workflow job:
```yaml
# Get a raw version from the source code - example for static version:
- name: "➡️ Extract version from 'pyproject.toml'"
  id: toml-version
  uses: SebRollen/toml-action@main
  with:
    file: 'pyproject.toml'
    field: 'project.version'

# Turn it to standard form:
- name: "🔢 Standardize Version"
  id: standard-version
  uses: Lex-DRL/standardize-version-action@main
  with:
    version: '${{ steps.toml-version.outputs.value }}'

# Use it - here, just print it to the log:
- name: "Echo standardized version"
  run: |
    echo 'The standard version: ${{ steps.standard-version.outputs.v }}' &&
    echo 'Is pre-release: ${{ steps.standard-version.outputs.is-pre }}'
```

## Inputs

- `version` - The raw version string to normalize: `0.1` / `v.1.2.3a1` / `v5`.
- [optional] `python-version` - You could specify the python version to use inside the action run. Defaults to the latest python 3.
- [optional] `skip-python-setup` - If you already have a `setup-python` action in the caller, set this input to anything except for `false` or empty string - to disable a (redundant) python setup.
  - Makes `python-version` irrelevant.

## Outputs
- `v` - Full version + "v" prefix - recommended for 🏷️ release name/tag: `v0.1` / `v1.2.3-alpha1`.
- `full` - Full version without "v" prefix: `0.1` / `1.2.3-alpha1`
- `number` - Only the main numeric version without suffix: `0.1` / `1.2.3`
- `suffix` - The optional last sub-version suffix/specifier. Example: `""` / `alpha1`.
- `is-pre` - Is the version detected as pre-release (alpha/beta/RC): `false` / `true`

## Log

For the ease of debug, the action itself also prints all the outputs into the log, like this:
```
🧩 Parsed version: 'v.1.1.6a1 b1'
   ├ 🏷️ 'v'      → v1.1.6-alpha1-beta1
   ├ full        →  1.1.6-alpha1-beta1
   ├ number      →  1.1.6
   ├ suffix      → 'alpha1-beta1'
   └ is-pre      →  true
✅ Outputs set
```
