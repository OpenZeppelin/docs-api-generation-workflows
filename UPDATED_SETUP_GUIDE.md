# Updated API Documentation Automation Setup Guide

Thanks to [this excellent Medium article](https://medium.com/hostspaceng/triggering-workflows-in-another-repository-with-github-actions-4f581f8e0ceb), there's a **much simpler** approach using `workflow_dispatch` and the GitHub CLI!

## 🎯 **Recommended Approach: workflow_dispatch + gh CLI**

This is now the **simplest and most reliable** method:

### How It Works
1. Tag is created in source repository (e.g., `openzeppelin-contracts`)
2. Source repo workflow triggers immediately 
3. Uses `gh workflow run` to trigger workflow in docs repository
4. Docs repository generates and commits API documentation

### Benefits
- ✅ **No webhooks needed**
- ✅ **No repository_dispatch complexity**
- ✅ **Direct workflow-to-workflow communication**
- ✅ **Immediate triggering (no polling)**
- ✅ **Built-in GitHub authentication**
- ✅ **Easy to test and debug**

## 🚀 **Quick Setup (Recommended)**

### Step 1: Add to Source Repositories

Add this workflow to each source repository (e.g., `openzeppelin-contracts`):

```yaml
# .github/workflows/trigger-docs-generation.yml
# (See simple-source-trigger.yml in the files)
```

### Step 2: Add to Docs Repository  

Add this workflow to your docs repository:

```yaml
# .github/workflows/generate-api-docs.yml
# (See simple-docs-receiver.yml in the files)
```

### Step 3: Create GitHub Token

1. Go to GitHub Settings → Developer settings → Personal access tokens
2. Create a token with `repo` scope
3. Add it as a secret named `DOCS_REPO_TOKEN` in each **source repository**

### Step 4: Test

1. Go to your docs repo → Actions → "Generate API Documentation"
2. Click "Run workflow" and test with manual inputs
3. Then create a test tag in a source repo to test the automation

## 🔄 **Alternative Approaches**

### Option 1: Repository Webhooks (More Complex)
The webhook approach from my original solution - use if you can't modify source repositories.

### Option 2: Scheduled Polling (Fallback)
Poll for new tags every hour - use if other approaches don't work.

### Option 3: Manual Triggers Only
Just use manual workflow runs - good for infrequent releases.

## 📊 **Comparison**

| Approach | Setup Complexity | Reliability | Speed | Source Repo Changes |
|----------|-----------------|-------------|-------|-------------------|
| **workflow_dispatch + gh CLI** | ⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | Required |
| Repository Webhooks | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | Not Required |
| Scheduled Polling | ⭐⭐ | ⭐⭐⭐ | ⭐⭐ | Not Required |
| Manual Only | ⭐ | ⭐⭐⭐⭐⭐ | ⭐ | Not Required |

## 🔧 **Configuration**

### Token Permissions
The `DOCS_REPO_TOKEN` needs:
- `repo` scope (to trigger workflows in the docs repository)
- Access to the docs repository

### Customizing Repositories
Edit the source workflow to change the target:
```yaml
gh workflow run generate-api-docs.yml \
  --repo YOUR_ORG/docs \  # Change this line
```

### Customizing Output Paths
The docs workflow automatically determines paths based on:
- Repository name (contracts vs contracts-upgradeable)
- Tag version (v5.x, v6.x, etc.)

## 🧪 **Testing**

### Test the docs workflow manually:
1. Go to docs repo → Actions → "Generate API Documentation"
2. Run with test parameters

### Test the full automation:
1. Create a test tag in source repository: `git tag v5.99.0 && git push origin v5.99.0`
2. Check Actions tab in both repositories
3. Verify docs are generated and committed

### Debug workflow runs:
- Check Actions tab for detailed logs
- Verify token permissions
- Ensure workflow file names match the `gh workflow run` command

## 🔒 **Security Notes**

- Token should have minimal required permissions
- Consider using GitHub Apps for organization-wide deployment
- Repository secrets are only accessible to workflows in that repository
- The docs repository workflow validates inputs

## 🎯 **Why This Is Better**

1. **Simpler**: No webhook configuration needed
2. **More reliable**: Direct GitHub API calls vs webhook delivery
3. **Easier to debug**: All logs in GitHub Actions
4. **Immediate**: No polling delays
5. **Standard**: Uses built-in GitHub features

This approach eliminates the complexity of webhooks while maintaining immediate triggering. It's the best of both worlds!

---

## 📁 **File Structure**

```
source-repository/
└── .github/workflows/
    └── trigger-docs-generation.yml

docs-repository/
├── .github/workflows/
│   └── generate-api-docs.yml
├── scripts/
│   └── generate-api-docs.js (your existing script)
└── content/contracts/
    └── (generated docs will go here)
```

The beauty of this approach is its simplicity - it's exactly what GitHub Actions was designed for!
