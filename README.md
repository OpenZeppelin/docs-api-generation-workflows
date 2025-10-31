# API Docs Workflows

Simple GitHub Actions workflows for automating API documentation generation from source repositories to a docs repository.

## How It Works

This system uses two types of workflows:

1. **Source Trigger** (in your source repo) - Detects changes and triggers docs generation
2. **Docs Receiver** (in your docs repo) - Generates and commits documentation

### Flow

```
Source Repo Push/Tag → Trigger Workflow → Docs Repo Workflow → Generate & Commit Docs
```

## Implementation

### Prerequisites

Make sure you have followed the [instructions of adding the API docgen templates](https://github.com/OpenZeppelin/docs#solidity-docgen) before continuting!

### 1. Setup Docs Repository Workflow

Choose the appropriate receiver template:

- `template-docs-receiver-non-versioned.yml` - For non-versioned docs (e.g., `/api`)
- `template-docs-receiver-versioned.yml` - For versioned docs (e.g., `/5.x/api`)

Copy to your docs repo at `.github/workflows/generate-api-docs-{CONTRACT}.yml`

**Configure:**
- `SOURCE_REPO`: Owner/repo name (e.g., `OpenZeppelin/openzeppelin-contracts`)
- `SOURCE_REPO_URL`: Full git URL
- `BASE_OUTPUT_PATH`: Docs output path (e.g., `content/contracts`)
- `USE_VERSIONED_PATHS`: `true` or `false`

### 2. Setup Source Repository Trigger

Choose your trigger type:

- `template-source-trigger-tag-based.yml` - Triggers on git tags (e.g., `release-v5.5`)
- `template-source-trigger-commit-based.yml` - Triggers on commits to a branch

Copy to your source repo at `.github/workflows/trigger-docs-generation.yml`

**Configure:**
- `WORKFLOW_NAME`: Name of the docs workflow (without `.yml`)
- Tag pattern or branch name (depending on template)

### 3. Add Repository Secret

In your source repository settings, add:
- **Secret name**: `DOCS_REPO_TOKEN`
- **Value**: GitHub Personal Access Token with `repo` and `workflow` permissions

## Example

**Source repo** (contracts):
- Push tag `release-v5.5`
- Trigger workflow runs
- Calls docs repo workflow with `tag_name=release-v5.5`

**Docs repo**:
- Receiver workflow runs
- Checks out source at `release-v5.5`
- Generates API docs
- Commits to `content/contracts/5.x/api`

## Manual Triggering

You can also manually trigger docs generation from the docs repository:
1. Go to Actions tab
2. Select your workflow
3. Click "Run workflow"
4. Enter tag name or commit SHA
