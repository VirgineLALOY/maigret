# Copilot instructions for Maigret

## Project overview

Maigret is a Python 3.10+ CLI and web app for checking usernames across many sites. The main package is `maigret/`, tests live under `tests/`, site definitions are in `maigret/resources/data.json`, and user/developer docs are in `README.md`, `CONTRIBUTING.md`, and `docs/source/`.

## Setup and commands

Install dependencies for local development:

```bash
poetry install --with dev
```

Activate the repo hooks once after cloning:

```bash
git config --local core.hooksPath .githooks/
```

Core repo commands:

```bash
make format     # black
make lint       # flake8 + mypy
make test       # pytest with coverage
```

Run a single test file or test selection without running the full suite:

```bash
pytest tests/test_sites.py -q
pytest tests/test_checking.py -k "self_check" -q
```

## High-level architecture

- `maigret/__main__.py` and `maigret/maigret.py` are the CLI entrypoints. They parse arguments, configure runtime settings, and orchestrate the full scan/report flow.
- `maigret/checking.py` contains the async search logic, per-site validation, self-check diagnostics, and detection helpers. This is the heart of the username-check execution loop.
- `maigret/sites.py` defines the in-memory site model (`MaigretSite`, `MaigretDatabase`) and is the layer that turns `maigret/resources/data.json` entries into executable site checks.
- `maigret/resources/data.json` is the canonical site database. It is huge and highly structured; changes should be surgical and preserve formatting.
- `maigret/report.py` assembles output formats (HTML, CSV, JSON, TXT, XMind, PDF, etc.) after a search completes.
- `maigret/web/app.py` provides the web interface, while `maigret/ai.py` covers optional AI analysis.
- `maigret/settings.py` and `maigret/resources/settings.json` hold configuration defaults for retries, recursion, extraction, and other runtime behavior.

This repo is data-driven: most behavior for a site is defined by the site entry in `data.json`, not by custom code paths scattered across the package.

## Important conventions and project-specific rules

### Site database edits

- `maigret/resources/data.json` is a large generated-style config file. Do not read it into a Python dict and dump it back out.
- Preserve the existing formatting and make line-level/surgical edits only.
- When changing site entries, use the project’s validation flow rather than broad refactors.

### Site validation workflows

- For a disabled site being re-enabled, validate with two independent checks:

```bash
maigret --self-check --site "SiteName" --use-disabled-sites
maigret noonewouldeverusethis7 --site "SiteName" --use-disabled-sites --print-not-found
```

- `--self-check` alone is not sufficient because it only exercises the known claimed/unclaimed examples in that entry.
- `--use-disabled-sites` is required when testing a currently disabled site; otherwise the site is silently skipped.

### Site metadata rules

- `checkType` is the most important modeling decision. Use it based on actual site behavior: `message`, `status_code`, or `response_url`.
- `urlProbe` is preferred for clean public APIs when the profile page is not the most stable signal.
- `presenseStrs` / `absenceStrs` should be narrow and specific; avoid generic markup or boilerplate text that causes false positives.
- If a site is blocked by anti-bot protections, prefer a correct `protection` tag and/or a site-specific activation flow, not a blanket disabled entry.

### Docs and translations

If you change any `.rst` file under `docs/source/`, refresh the translation catalogs so locales do not silently fall back to English:

```bash
cd docs
make intl-update LANG=zh_CN
make intl-update LANG=fr
```

### Git hooks and commit workflow

The repo uses a local git hook path:

```bash
git config --local core.hooksPath .githooks/
```

This hook regenerates `maigret/resources/db_meta.json` and `sites.md` for site-database changes and stages those generated files with the commit. Run `git commit` from inside the repo so the hook can fire.

### Code style

- Python code follows PEP 8.
- Indentation is 4 spaces.
- Imports are grouped standard library, third-party, then project-local.
- Naming conventions are explicit in the repo: `snake_case` for functions/variables, `CamelCase` for classes, `UPPER_CASE` for constants.

## When making changes

- Keep patches focused and consistent with adjacent code.
- Favor the smallest change that matches the existing patterns in the package or tests.
- For new site additions, prefer the project-supported `--submit` workflow and only fall back to manual edits when needed.
- If a change touches site metadata or docs, validate the relevant commands before finalizing the PR.
