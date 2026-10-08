# Repository Guide

## Project

Maigret is a Python 3.10+ command-line and web application for checking usernames across sites. The Python package is in `maigret/`; tests are in `tests/`; site definitions and settings are under `maigret/resources/`; user and developer documentation is in `README.md`, `CONTRIBUTING.md`, and `docs/source/`.

## Development

- Install the project and development dependencies with `poetry install --with dev`.
- Run the test suite with `make test` or `poetry run pytest tests`.
- Run linting and type checks with `make lint` (flake8 and mypy).
- Format Python code with `make format` (Black).
- Follow the existing Python style: PEP 8, 4-space indentation, and standard-library / third-party / local import groups.

## Contribution Guidance

- Keep changes focused and follow existing patterns in the package and tests.
- Make surgical edits to `maigret/resources/data.json`; do not load and re-dump the whole file, since that reformats unrelated entries.
- When changing `.rst` documentation under `docs/source/`, update the translation catalogs as described in `CONTRIBUTING.md`.
- For site-database changes, follow the validation and metadata-generation workflow in `CONTRIBUTING.md`; the repository's `.githooks` include the relevant pre-commit automation.
