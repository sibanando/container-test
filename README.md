# Container Test CI/CD Workflow

This repository uses a GitHub Actions CI/CD workflow that dynamically builds, tests, and publishes Python container images using official Python images from Docker Hub.

## Overview

The workflow is defined in `.github/workflows/ci.yml` and performs the following:

1. **Build & Test**: Runs tests inside official Python container images for each version specified in `python-versions.json`.
2. **Publish**: Re-tags and pushes the base Python images to GitHub Container Registry (GHCR) on every push to `main`.

## Configuration

### Python Versions

Python versions are managed in `python-versions.json` at the repository root:

```json
["3.10", "3.11"]
```

Modify this file to add or remove Python versions. The workflow will automatically:
- Run tests against each version in the list.
- Publish images tagged as `ghcr.io/<owner>/<repo>:py<version>` for each version.

Example to test Python 3.9, 3.10, 3.11:

```json
["3.9", "3.10", "3.11"]
```

## Workflow Triggers

The workflow runs automatically on:

- **Push to `main`**: Runs tests and publishes images to GHCR.
- **Pull requests to `main`**: Runs tests only (no publish).
- **Manual trigger** (`workflow_dispatch`): Allows manual runs from the Actions UI.

## Jobs

### 1. build-and-test

- **Trigger**: Always runs on push/PR/dispatch
- **Steps**:
  1. Checks out the repository
  2. Reads Python versions from `python-versions.json`
  3. For each version:
     - Pulls the official `python:<version>-slim` image
     - Installs dependencies from `requirements.txt` (if present)
     - Runs `pytest` if tests are found, otherwise skips

### 2. publish-image

- **Trigger**: Only runs on push to `main` (after `build-and-test` succeeds)
- **Steps**:
  1. Checks out the repository
  2. Logs in to GitHub Container Registry (GHCR) using `GITHUB_TOKEN`
  3. For each version in `python-versions.json`:
     - Pulls the official `python:<version>-slim` image
     - Tags it as `ghcr.io/<owner>/<repo>:py<version>`
     - Pushes to GHCR

## Requirements

### Repository Setup

- **File**: `python-versions.json` — **Required** at the repository root. The workflow will fail if missing.
- **File**: `requirements.txt` — Optional. If present, dependencies will be installed during test runs.
- **Directory**: `tests/` — Optional. If present, `pytest` will run tests from this directory.

### GitHub Setup

1. **Permissions**: The workflow requires `packages: write` permission to push images to GHCR. This is set in the workflow file.
2. **GITHUB_TOKEN**: Uses GitHub's built-in `GITHUB_TOKEN` for authentication. No additional secrets needed.
3. **Package Settings**: Make sure your repository has packages enabled (usually enabled by default).

## Image Tags

Published images are tagged and pushed to GHCR as:

```
ghcr.io/<owner>/<repo>:py3.10
ghcr.io/<owner>/<repo>:py3.11
ghcr.io/<owner>/<repo>:py3.9
... (per your python-versions.json)
```

Example pull command:

```bash
docker pull ghcr.io/sibanando/container-test:py3.11
docker run -it ghcr.io/sibanando/container-test:py3.11 python --version
```

## Running Tests Locally

To test locally with the same Python versions:

```bash
# Test with Python 3.10
docker run --rm -v $(pwd):/src -w /src python:3.10-slim bash -c \
  "pip install -r requirements.txt && pytest"

# Test with Python 3.11
docker run --rm -v $(pwd):/src -w /src python:3.11-slim bash -c \
  "pip install -r requirements.txt && pytest"
```

Or run directly with your local Python:

```bash
pip install -r requirements.txt
pytest
```

## Workflow File Reference

- **File**: `.github/workflows/ci.yml`
- **Name**: "CI/CD - Build, Test, Publish (using container images)"
- **Base Images**: Official Python images from Docker Hub (`python:<version>-slim`)
- **Container Registry**: GitHub Container Registry (GHCR)

## Troubleshooting

### Workflow fails with "python-versions.json not found"

- Ensure `python-versions.json` exists at the repository root.
- Commit and push the file to the branch you're working on.
- Re-run the workflow from the Actions tab.

### GHCR push fails with authentication error

- Verify that `permissions.packages: write` is set in the workflow (already configured).
- Check that your repository has packages enabled.
- Ensure you're pushing to the correct GHCR URL: `ghcr.io/<owner>/<repo>:py<version>`

### Tests don't run

- Ensure `requirements.txt` exists with all test dependencies (e.g., `pytest`).
- Create a `tests/` directory or name test files with `test_` prefix.
- Check the workflow logs in the Actions tab for detailed error messages.

### No tests found — skipping pytest

This is normal if your repository has no test files. Add tests or a `tests/` directory to enable automated testing.

## Example: Adding a New Python Version

1. Edit `python-versions.json`:

```json
["3.9", "3.10", "3.11", "3.12"]
```

2. Commit and push:

```bash
git add python-versions.json
git commit -m "ci: add Python 3.12 to tested versions"
git push origin main
```

3. The workflow will automatically run tests and publish images for Python 3.9, 3.10, 3.11, and 3.12.

## Quick Start

1. **Ensure `python-versions.json` exists** in the repository root.
2. **Create `requirements.txt`** with your dependencies (optional but recommended).
3. **Add tests** in a `tests/` directory (optional but recommended).
4. **Push to `main`** — the workflow will automatically run tests and publish images.

## Security Notes

- The workflow uses GitHub's built-in `GITHUB_TOKEN` for GHCR authentication.
- `GITHUB_TOKEN` is scoped per workflow run and has the minimum required permissions.
- If you prefer a separate personal access token (PAT), you can store it as a repository secret and update the workflow to use it.

## Support

For issues or questions about the workflow, check:
- The Actions tab in your repository for workflow logs.
- The `.github/workflows/ci.yml` file for workflow configuration.
- The `python-versions.json` file for the list of tested Python versions.
