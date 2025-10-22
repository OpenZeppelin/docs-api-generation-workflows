# API Documentation Workflow Templates

This repository contains **template workflows** for automatically generating API documentation when contract repositories are updated.

## Quick Start

### Step 1: Choose Your Templates

Pick the templates based on your deployment strategy:

#### For Tag-Based Deployments (Versioned Releases)
- **Source repo**: `template-source-trigger-tag-based.yml`
- **Docs repo**: `template-docs-receiver-versioned.yml`
- **Use when**: You release versioned tags like `v5.1.0`, `release-v6.0`, etc.
- **Output**: `content/contracts/5.x/api`, `content/contracts/6.x/api`, etc.

#### For Commit-Based Deployments (Continuous)
- **Source repo**: `template-source-trigger-commit-based.yml`
- **Docs repo**: `template-docs-receiver-non-versioned.yml`
- **Use when**: You deploy on every commit to main branch
- **Output**: `content/your-contract/api`

### Step 2: Setup Source Repository

1. Copy the appropriate source trigger template
2. Save as `.github/workflows/trigger-docs-generation.yml` in your source repo
3. Update the `{WORKFLOW_NAME}` placeholder with your docs workflow name
4. Customize tag/branch patterns if needed
5. Add `DOCS_REPO_TOKEN` secret to your repository

### Step 3: Setup Docs Repository

1. Copy the appropriate docs receiver template
2. Save as `.github/workflows/generate-api-docs-{YOUR_CONTRACT}.yml`
3. Replace all `{YOUR_CONTRACT_NAME}` and `{YOUR_CONTRACT}` placeholders
4. Configure the environment variables:
   ```yaml
   env:
     SOURCE_REPO: 'OpenZeppelin/your-contract'
     SOURCE_REPO_URL: 'https://github.com/OpenZeppelin/your-contract.git'
     BASE_OUTPUT_PATH: 'content/your-path'
     USE_VERSIONED_PATHS: 'true'  # or 'false'
   ```

## Template Files

| File | Purpose | Location | Description |
|------|---------|----------|-------------|
| `template-source-trigger-tag-based.yml` | Source trigger | Source repos | Triggers on version tags |
| `template-source-trigger-commit-based.yml` | Source trigger | Source repos | Triggers on main branch commits |
| `template-docs-receiver-versioned.yml` | Docs generator | Docs repo | Generates versioned docs (5.x/api) |
| `template-docs-receiver-non-versioned.yml` | Docs generator | Docs repo | Generates non-versioned docs (api) |

## Example: Adding OpenZeppelin Contracts

### In Source Repo (openzeppelin-contracts)

Create `.github/workflows/trigger-docs-generation.yml`:

```yaml
# Copy from template-source-trigger-tag-based.yml
# Change line 46:
gh workflow run generate-api-docs-openzeppelin-contracts.yml \
```

### In Docs Repo

Create `.github/workflows/generate-api-docs-openzeppelin-contracts.yml`:

```yaml
# Copy from template-docs-receiver-versioned.yml
name: Generate API Docs - OpenZeppelin Contracts

env:
  SOURCE_REPO: 'OpenZeppelin/openzeppelin-contracts'
  SOURCE_REPO_URL: 'https://github.com/OpenZeppelin/openzeppelin-contracts.git'
  BASE_OUTPUT_PATH: 'content/contracts'
  USE_VERSIONED_PATHS: 'true'
```

**Result**: When you push `release-v5.1.0`, docs are generated at `content/contracts/5.x/api`

## Example: Adding Community Contracts

### In Source Repo (community-contracts)

Create `.github/workflows/trigger-docs-generation.yml`:

```yaml
# Copy from template-source-trigger-commit-based.yml
# Change line 43:
gh workflow run generate-api-docs-community-contracts.yml \
```

### In Docs Repo

Create `.github/workflows/generate-api-docs-community-contracts.yml`:

```yaml
# Copy from template-docs-receiver-non-versioned.yml
name: Generate API Docs - Community Contracts

env:
  SOURCE_REPO: 'OpenZeppelin/community-contracts'
  SOURCE_REPO_URL: 'https://github.com/OpenZeppelin/community-contracts.git'
  BASE_OUTPUT_PATH: 'content/community-contracts'
  USE_VERSIONED_PATHS: 'false'
```

**Result**: When you push to `main`, docs are generated at `content/community-contracts/api`

## Configuration Guide

### Source Trigger Templates

**TODO Items to Replace:**
1. `{WORKFLOW_NAME}` - The docs workflow filename (without extension)
2. Tag/branch patterns (if using custom patterns)
3. Docs repo name (if not `OpenZeppelin/docs`)

### Docs Receiver Templates

**TODO Items to Replace:**
1. `{YOUR_CONTRACT_NAME}` - Display name for the workflow
2. `{YOUR_CONTRACT}` - Repository name in environment variables
3. `{YOUR_PATH}` - Output path for documentation
4. Default `tag_name` value
5. Default `trigger_type` value

### Environment Variables

| Variable | Required | Description | Example |
|----------|----------|-------------|---------|
| `SOURCE_REPO` | Yes | Source repository name | `OpenZeppelin/openzeppelin-contracts` |
| `SOURCE_REPO_URL` | Yes | Full git clone URL | `https://github.com/OpenZeppelin/openzeppelin-contracts.git` |
| `BASE_OUTPUT_PATH` | Yes | Base directory for docs | `content/contracts` |
| `USE_VERSIONED_PATHS` | Yes | Use version-based paths | `'true'` or `'false'` |

### Path Resolution Logic

The docs workflow automatically determines the output path:

**Non-Versioned** (`USE_VERSIONED_PATHS: 'false'`):
- Always: `{BASE_OUTPUT_PATH}/api`
- Example: `content/community-contracts/api`

**Versioned - Commits** (`USE_VERSIONED_PATHS: 'true'`, `trigger_type: 'commit'`):
- Always: `{BASE_OUTPUT_PATH}/latest/api`
- Example: `content/contracts/latest/api`

**Versioned - Tags** (`USE_VERSIONED_PATHS: 'true'`, `trigger_type: 'tag'`):
- Pattern: `{BASE_OUTPUT_PATH}/{MAJOR_VERSION}.x/api`
- Example: `v5.1.0` → `content/contracts/5.x/api`

## Required Secrets

### In Source Repositories

Add a secret named `DOCS_REPO_TOKEN`:
1. Generate a GitHub Personal Access Token with `workflow` permission
2. Add it to your source repository secrets
3. Name it exactly `DOCS_REPO_TOKEN`

## Customization

### Custom Tag Patterns

Default pattern is `release-v*`. To customize:

1. Update line 16 in source trigger template:
   ```yaml
   - 'v*'  # Matches v1.0.0, v2.0.0, etc.
   ```

2. Update the regex check on line ~28:
   ```yaml
   if [[ "${{ github.ref_name }}" =~ ^v ]]; then
   ```

### Custom Branch Names

Default branch is `main`. To customize:

1. Update line 17 in commit-based trigger:
   ```yaml
   - 'develop'  # Or 'master', etc.
   ```

2. Update the check on line ~26:
   ```yaml
   if [ "${{ github.ref }}" == "refs/heads/develop" ]; then
   ```

### Custom Version Extraction

Default regex extracts major version from tags like `v5.1.0` → `5`.

To customize, update line ~77 in versioned receiver template:
```yaml
if [[ "$TAG_NAME" =~ v([0-9]+) ]]; then
```

## Troubleshooting

### Workflow Not Triggering

1. Check that `DOCS_REPO_TOKEN` secret exists in source repo
2. Verify the token has `workflow` permission
3. Confirm the workflow name matches between source and docs repos
4. Check that tag/branch patterns match your conventions

### Wrong Output Path

1. Verify `BASE_OUTPUT_PATH` is correct
2. Check `USE_VERSIONED_PATHS` setting
3. For versioned paths, ensure tag format matches regex pattern
4. Review workflow logs for path resolution details

### Documentation Not Updating

1. Check if `node scripts/generate-api-docs.js` exists in docs repo
2. Verify script parameters are correct
3. Check workflow logs for generation errors
4. Ensure source repo URL is accessible

## Benefits

- **No central mapping**: Each contract has its own dedicated workflows
- **Clear configuration**: All settings in one place per contract
- **Easy to understand**: Template-based approach with clear TODOs
- **Flexible paths**: Each contract can have unique output structure
- **Independent versioning**: Each contract manages its own versioning
- **Type-safe**: No JSON configuration that could have typos

## Additional Documentation

See `PER_CONTRACT_SETUP.md` for detailed setup instructions and examples.
