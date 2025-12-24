# CI Setup Examples

Real-world patterns from todoist-mcp and pydmp repositories.

## Pre-commit Configuration

### Python Project (Minimal)

```yaml
repos:
  - repo: https://github.com/psf/black
    rev: 24.10.0
    hooks:
      - id: black
        args: ["src", "tests"]
        pass_filenames: false

  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.8.4
    hooks:
      - id: ruff
        args: ["--fix", "--exit-non-zero-on-fix"]

  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v6.0.0
    hooks:
      - id: end-of-file-fixer
      - id: trailing-whitespace

  - repo: https://github.com/PyCQA/bandit
    rev: 1.7.9
    hooks:
      - id: bandit
        args: ["-r", "src", "-x", "tests"]
        pass_filenames: false
```

### With Type Checking (mypy)

```yaml
- repo: https://github.com/pre-commit/mirrors-mypy
  rev: v1.14.1
  hooks:
    - id: mypy
      args: ["--config-file=pyproject.toml", "src"]
      pass_filenames: false
      additional_dependencies:
        - types-redis>=4.6.0
        - types-requests>=2.31.0
```

## GitHub Workflows

### Branch Flow Strategy

```
main (development)
  │
  └──▶ release/latest (stable, triggers PyPI publish)
         │
         └──▶ release/1.0.0 (version branch, creates draft release)
         └──▶ release/1.0.1-beta (pre-release)
```

### CI Matrix for Multiple Python Versions

```yaml
jobs:
  build:
    strategy:
      matrix:
        python-version: ["3.12", "3.13"]
    steps:
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
```

### Non-blocking Checks

For checks that shouldn't fail the build but provide useful info:

```yaml
- name: Type check with mypy (non-blocking)
  run: mypy src
  continue-on-error: true

- name: Dependency audit (non-blocking)
  run: pip-audit -s
  continue-on-error: true
```

### Codecov Integration

```yaml
- name: Run pytest with coverage
  run: pytest --cov=mypackage --cov-report=xml

- name: Upload coverage
  uses: codecov/codecov-action@v4
  if: matrix.python-version == '3.12'  # Only upload once
  with:
    file: ./coverage.xml
  env:
    CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

### PyPI Trusted Publishing

No API token needed - uses OIDC:

1. Create `publish` environment in GitHub repo settings
2. Configure PyPI Trusted Publisher with:
   - Owner: your-username
   - Repository: your-repo
   - Workflow: publish.yml
   - Environment: publish

```yaml
jobs:
  publish:
    environment: publish
    permissions:
      id-token: write
    steps:
      - uses: pypa/gh-action-pypi-publish@release/v1
```

### Docker with GitHub Container Registry

```yaml
- name: Log in to GHCR
  uses: docker/login-action@v3
  with:
    registry: ghcr.io
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}

- name: Build and push
  uses: docker/build-push-action@v5
  with:
    push: true
    tags: ghcr.io/${{ github.repository }}:latest
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

### MkDocs with Mike Versioning

Versioning strategy:
- `main` → `dev` alias (rolling, replaced on each push)
- `release/latest` → `latest` alias + version number
- `release/x.y.z` → `x.y.z` (archived)

```yaml
- name: Determine version
  run: |
    if [[ "${{ github.ref_name }}" == "main" ]]; then
      echo "version=dev-${GITHUB_SHA::7}" >> $GITHUB_OUTPUT
      echo "alias=dev" >> $GITHUB_OUTPUT
    elif [[ "${{ github.ref_name }}" == "release/latest" ]]; then
      echo "version=$(python -c 'from mypackage import __version__; print(__version__)')" >> $GITHUB_OUTPUT
      echo "alias=latest" >> $GITHUB_OUTPUT
    fi

- name: Deploy with mike
  run: mike deploy --push --update-aliases "$VERSION" "$ALIAS"
```

## Multi-Language Support

### Python + JavaScript

```yaml
# .pre-commit-config.yaml
repos:
  # Python
  - repo: https://github.com/psf/black
    rev: 24.10.0
    hooks:
      - id: black

  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.8.4
    hooks:
      - id: ruff

  # JavaScript/TypeScript
  - repo: https://github.com/pre-commit/mirrors-eslint
    rev: v9.0.0
    hooks:
      - id: eslint
        files: \.[jt]sx?$
        additional_dependencies:
          - eslint
          - typescript

  - repo: https://github.com/pre-commit/mirrors-prettier
    rev: v4.0.0
    hooks:
      - id: prettier
```

### CodeQL Multi-Language

```yaml
strategy:
  matrix:
    language: ['python', 'javascript', 'go']
steps:
  - uses: github/codeql-action/init@v3
    with:
      languages: ${{ matrix.language }}
```

## Node.js Examples

### Node.js CI Matrix

```yaml
jobs:
  build:
    strategy:
      matrix:
        node-version: ['20', '22']
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
```

### NPM Publishing with Provenance

NPM provenance links packages to their source repository:

```yaml
permissions:
  id-token: write  # Required for provenance

steps:
  - name: Publish to NPM
    run: npm publish --provenance --access public
    env:
      NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

### Check Version Before Publishing

```yaml
- name: Check if version exists on NPM
  run: |
    PACKAGE_NAME=$(node -p "require('./package.json').name")
    VERSION=$(node -p "require('./package.json').version")
    if npm view "$PACKAGE_NAME@$VERSION" version 2>/dev/null; then
      echo "Version exists, bump first"
      exit 1
    fi
```

### Node.js Pre-commit Hooks

```yaml
repos:
  - repo: https://github.com/pre-commit/mirrors-eslint
    rev: v9.0.0
    hooks:
      - id: eslint
        files: \.[jt]sx?$
        types: [file]
        additional_dependencies:
          - eslint
          - typescript
          - "@typescript-eslint/parser"
          - "@typescript-eslint/eslint-plugin"

  - repo: https://github.com/pre-commit/mirrors-prettier
    rev: v4.0.0
    hooks:
      - id: prettier
        types_or: [javascript, jsx, ts, tsx, json, yaml, markdown, css]
```

### package.json Scripts

```json
{
  "scripts": {
    "build": "tsc",
    "test": "vitest run",
    "test:coverage": "vitest run --coverage",
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "format": "prettier --write .",
    "format:check": "prettier --check .",
    "typecheck": "tsc --noEmit",
    "prepare": "husky"
  }
}
```

## Python Examples

### pyproject.toml Dev Dependencies

```toml
[project.optional-dependencies]
dev = [
    "pytest>=8.0",
    "pytest-cov>=4.0",
    "pytest-asyncio>=0.23",
    "black>=24.0",
    "ruff>=0.8",
    "mypy>=1.0",
    "pre-commit>=3.0",
]

docs = [
    "mkdocs>=1.5",
    "mkdocs-material>=9.4",
    "mkdocstrings[python]>=0.24",
    "mike>=2.0",
]
```

Install:
```bash
pip install -e ".[dev]"
pre-commit install
```

## Monorepo / Multi-Language

### Separate CI Workflows

For projects with both Python and Node.js:

```
.github/workflows/
├── python-ci.yaml     # Triggered by src/python/** changes
├── node-ci.yaml       # Triggered by src/js/** changes
├── publish-pypi.yml   # Python package publishing
└── publish-npm.yml    # NPM package publishing
```

### Path Filtering

```yaml
on:
  push:
    paths:
      - 'packages/python/**'
      - 'pyproject.toml'
  pull_request:
    paths:
      - 'packages/python/**'
      - 'pyproject.toml'
```

### Shared Workflows

```yaml
# .github/workflows/publish.yml
on:
  push:
    branches:
      - release/latest

jobs:
  detect-changes:
    outputs:
      python: ${{ steps.filter.outputs.python }}
      node: ${{ steps.filter.outputs.node }}
    steps:
      - uses: dorny/paths-filter@v3
        id: filter
        with:
          filters: |
            python:
              - 'packages/python/**'
            node:
              - 'packages/node/**'

  publish-pypi:
    needs: detect-changes
    if: needs.detect-changes.outputs.python == 'true'
    # ... PyPI publish steps

  publish-npm:
    needs: detect-changes
    if: needs.detect-changes.outputs.node == 'true'
    # ... NPM publish steps
```
