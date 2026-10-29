<img src="https://kaeris.dev/icon-512.png" alt="KAERIS" width="72" align="left" style="margin-right:16px" />

# KAERIS i18n Check and Translate

Fail CI when a locale file is missing keys or a translation broke a placeholder. Or translate
the keys you added into 46 languages and open a pull request with the result.

[![test](https://github.com/kaeris-io/kaeris-action/actions/workflows/test.yml/badge.svg)](https://github.com/kaeris-io/kaeris-action/actions/workflows/test.yml)
[![release](https://img.shields.io/github/v/release/kaeris-io/kaeris-action)](https://github.com/kaeris-io/kaeris-action/releases)
[![license](https://img.shields.io/github/license/kaeris-io/kaeris-action)](LICENSE)

```yaml
- uses: kaeris-io/kaeris-action@v2
  with:
    mode: check
    source: locales/en.json
    languages: de,fr,ja
```

- **`check`**: offline, no API call, no key. Fails the job on a missing key, a lost or renamed
  placeholder, number drift or a missing plural form. Optional SARIF report puts each finding on
  the line of the locale file in the pull request.
- **`translate`**: translates only the keys that are new or changed since the last run, writes
  the locale files, and returns `changed` and a Markdown `summary` for the pull request body.
- **13 formats** in both modes: JSON, YAML, `.strings`, `.xcstrings`, `.po`, ARB, Android XML,
  CSV, XLIFF, `.properties`, `.resx`, Fluent and Markdown.

`check` is free. `translate` runs without a key on the free tier (8 languages per run); plans:
Pro $19/month with 12M characters, Team/API $79/month with 60M, Lifetime $79 once with your own
OpenRouter key. See [kaeris.dev/pricing](https://kaeris.dev/pricing.html).

## Usage

### Gate every PR (i18n firewall — no API key)

```yaml
# .github/workflows/i18n-check.yml
name: i18n check
on: [pull_request]
jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: kaeris-io/kaeris-action@v2
        with:
          mode: check
          source: locales/en.json
          languages: es,fr,de,ja
```

A broken or incomplete translation now fails the check and can't merge. Add it as a required status check in branch protection to make it a hard gate.

### Auto-translate new strings into a PR

```yaml
# .github/workflows/i18n-translate.yml
name: i18n translate
on:
  push:
    branches: [main]
    paths: ['locales/en.json']
jobs:
  translate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - id: kaeris
        uses: kaeris-io/kaeris-action@v2
        with:
          mode: translate
          source: locales/en.json
          languages: es,fr,de,ja
          api-key: ${{ secrets.KAERIS_API_KEY }}   # omit for the free demo
      - if: steps.kaeris.outputs.changed == 'true'
        uses: peter-evans/create-pull-request@v6
        with:
          title: "i18n: update translations"
          body: ${{ steps.kaeris.outputs.summary }}
          branch: i18n/update
```

## Inputs

| Input | Description | Default |
|---|---|---|
| `mode` | `translate` (write locale files) or `check` (i18n firewall, no API call) | `translate` |
| `source` | Source strings file, e.g. `locales/en.json` | — (required) |
| `languages` | Comma-separated target languages, e.g. `es,fr,de,ja` | — (required) |
| `out` | Directory for translated / target locale files | source's dir |
| `api-key` | KAERIS API key, e.g. the free key from [kaeris.dev/docs/key](https://kaeris.dev/docs/key) (no account). Omit for the free demo. Unused in `check`. | — |
| `openrouter-key` | OpenRouter key for the Lifetime (BYOK) plan. Unused in `check`. | — |
| `only-new` | `translate`: only translate keys missing/changed since the last run (uses `kaeris.lock`) | `true` |
| `tone` | `translate`: `neutral`, `formal`, or `casual` | — (`neutral`) |
| `icu` | `translate`: hint that strings use ICU MessageFormat (plurals/select) so it's preserved | `false` |
| `strict` | `check`: also fail on extra/stale keys not in the source | `false` |
| `sarif-file` | `check`: also write a SARIF 2.1.0 report to this path (see below) | — |
| `api-url` | API base URL | `https://kaeris.dev` |

## Outputs

| Output | Description |
|---|---|
| `changed` | `"true"` if `translate` wrote/updated any locale content — open a PR only when this is true |
| `summary` | Markdown summary (keys translated, per-language matrix, placeholder/overflow counts) — use as a PR body |
| `missing` | Total missing-key count across all target languages |
| `placeholder-issues` | Total placeholder-mismatch count (should be `0`) |
| `check-exit-code` | Exit code of `kaeris check` (0 clean, 1 problem, 2 bad usage) |
| `translate-exit-code` | Exit code of `kaeris translate` (0 clean, 3 some languages fell back to the source); non-zero also fails the step |

## Findings inside the pull request (SARIF)

`check` can emit a SARIF 2.1.0 report. Upload it and every finding lands as an annotation on
the exact line of the locale file — in the diff, next to the human reviewer's comments —
instead of hiding in the job log.

```yaml
- uses: kaeris-io/kaeris-action@v2
  with:
    mode: check
    source: locales/en.json
    languages: de,fr,es
    sarif-file: kaeris.sarif

- uses: github/codeql-action/upload-sarif@v3
  if: always()            # upload the report even when the gate failed — that is when it matters
  with:
    sarif_file: kaeris.sarif
```

The report is written even on a clean run (an empty result set), because `upload-sarif`
fails the workflow if the file is missing, and "nothing was wrong" must not look like "the
step is broken".

Note on visibility: code-scanning uploads are free on public repositories; on private ones
they need GitHub Advanced Security. Without it the gate still works exactly as before — the
job fails and the log lists every finding — you just do not get the inline annotations.

## What it protects against

The deterministic checks (no API needed) catch the bugs that actually break apps: lost or renamed placeholders, number drift (`Delete 5 files` → `Delete 50 files`), the ICU/CLDR plural categories a target language requires, and UI-overflow risk. Every number in the PR summary comes from a real `kaeris check --json` run — nothing is fabricated.

## Upgrading from v1

Change `@v1` to `@v2`. Inputs and outputs are the same. v2 installs `kaeris>=0.2.39`
(v1 installs `>=0.2.12`), and the Marketplace name is now "KAERIS i18n Check and Translate".
The `v1` tag stays where it is.

## Links

- Website and docs: [kaeris.dev](https://kaeris.dev), [GitHub Actions recipe](https://kaeris.dev/ci-github-actions.html)
- CLI: `pip install kaeris` ([PyPI](https://pypi.org/project/kaeris/)); the check alone, without Python: `npx kaeris check` ([npm](https://www.npmjs.com/package/kaeris))
- MCP server for Claude, Cursor and other MCP clients: `pip install kaeris-mcp`
- Questions and ideas: [Discussions](https://github.com/kaeris-io/kaeris-cli/discussions). Security: [SECURITY.md](SECURITY.md)

## License

MIT — see [LICENSE](./LICENSE).
