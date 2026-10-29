# Contributing

Bug reports, format requests and pull requests are welcome.

## Before you start

- For a bug, open an issue with the bug form: version, command, source file, actual output.
- For anything larger than a small fix, open an issue or a discussion first, so we agree on
  the approach before you write the code.
- Security problems go to support@kaeris.dev, not to issues. See [SECURITY.md](SECURITY.md).

## Development

```bash
git clone https://github.com/kaeris-io/kaeris-action
cd kaeris-action
# The action is a composite step around the kaeris CLI.
# .github/workflows/test.yml runs it on the fixtures in test/fixtures.
pip install kaeris
kaeris check --source test/fixtures/good/en.json --langs de,fr --out test/fixtures/good
```

## Pull requests

- One change per pull request, with a test that fails without it.
- Keep the existing style. No new runtime dependencies without discussing it first.
- Update the README when a flag, input or output changes.
- Never commit API keys, tokens or real customer strings.

By contributing you agree that your work is released under the MIT License of this repository.
