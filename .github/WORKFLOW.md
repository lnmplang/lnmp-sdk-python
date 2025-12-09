# Python SDK Workflow Organization

## Overview

The LNMP Python SDK has two main GitHub Actions workflows for continuous integration and release automation.

## Workflow Structure

### 1. **ci.yml** - Continuous Integration

**Triggers:**
- Push to `main` or `develop` branches
- Pull requests to `main`
- Manual workflow dispatch

**Jobs:**

#### Quality Check
- Runs `cargo fmt`, `cargo clippy`, and Rust unit tests
- Ensures code quality before proceeding

#### Python Tests
- Matrix strategy across Python 3.9-3.12
- Multi-platform: Linux, macOS, Windows
- Runs pytest with coverage reporting
- Uploads coverage to Codecov

#### Build Wheels (Manual Only)
- Triggered via `workflow_dispatch`
- Builds native wheels for:
  - Linux (x86_64)
  - macOS (x86_64, arm64)
  - Windows (x86_64)
- Uploads wheels as artifacts

#### Publish to PyPI (Manual Only)
- Triggered via `workflow_dispatch`
- Downloads all platform wheels
- Publishes to PyPI using trusted publisher (OIDC)

#### Benchmarks
- Runs on pushes to `main` only
- Executes performance benchmarks
- Uploads results as artifacts

---

### 2. **release.yml** - Automated Releases

**Triggers:**
- Push tags matching `v*` (e.g., `v0.5.7`)

**Jobs:**

#### Test
- Matrix testing across Python 3.9-3.12 and all platforms
- Ensures quality before release

#### Build Wheels
- Builds wheels for all supported platforms:
  - Linux x86_64
  - macOS universal2 (x86_64 + arm64)
  - Windows x86_64
- Uploads as artifacts

#### Build Source Distribution
- Creates source distribution (sdist)
- Uploads as artifact

#### Publish to PyPI
- Downloads all wheels and sdist
- Publishes to PyPI using trusted publisher
- Only runs for version tags (not pre-releases)

#### Create GitHub Release
- Creates GitHub release with changelog
- Attaches all wheels and sdist
- Marks pre-releases appropriately

## Environment Variables

Both workflows use:
- `CARGO_TERM_COLOR=always` - Colored Cargo output
- `RUST_BACKTRACE=1` - Full backtraces on errors
- `PYTHONPATH=$PWD:$PYTHONPATH` - For local package imports

## PyPI Publishing Setup

### Trusted Publisher (Recommended) ✅

Configure at https://pypi.org/manage/account/publishing/

**Settings:**
- **Owner**: Your GitHub username or organization
- **Repository**: `lnmp-sdk-python`
- **Workflow**: `release.yml`
- **Environment**: `pypi`

No manual tokens required with trusted publishers!

### Alternative: Manual Token

If trusted publisher isn't available:
1. Generate PyPI API token at https://pypi.org/manage/account/token/
2. Add as GitHub secret: `PYPI_TOKEN`
3. Update workflow to use token-based authentication

## Release Process

### Creating a Release

```bash
# 1. Update version in pyproject.toml and Cargo.toml
# 2. Commit changes
git add pyproject.toml Cargo.toml
git commit -m "chore: bump version to 0.5.7"

# 3. Create and push tag
git tag v0.5.7
git push origin main
git push origin v0.5.7

# 4. GitHub Actions automatically:
#    - Runs tests
#    - Builds wheels for all platforms
#    - Publishes to PyPI
#    - Creates GitHub Release
```

### Pre-release

```bash
# Use pre-release tag format
git tag v0.5.7-beta.1
git push origin v0.5.7-beta.1

# Workflow will create GitHub release but skip PyPI
```

## Required GitHub Secrets

- None required if using PyPI trusted publisher
- `PYPI_TOKEN` - Only if using manual token authentication

## Permissions

Workflows require:
- `contents: write` - For creating GitHub releases
- `id-token: write` - For PyPI trusted publishing
