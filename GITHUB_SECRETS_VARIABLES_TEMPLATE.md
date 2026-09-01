# GitHub Secrets and Variables Template

This file contains all the secrets and variables required for the UiPath CI/CD pipeline. Use this as a checklist when setting up your repository.

## 📝 Instructions

1. **Do NOT commit this file** - Keep it locally for reference only
2. Add each secret/variable via: Repository → Settings → Secrets and variables
3. Copy the exact names (case-sensitive)
4. Replace placeholder values with your actual values

---

## 🔐 GitHub Secrets

Navigate to: **Settings → Secrets and variables → Secrets → New repository secret**

### Development Environment Secrets

```
Name: UIPATH_DEV_CLIENT_ID
Value: [Your Dev Environment Service Principal Client ID]
```

```
Name: UIPATH_DEV_CLIENT_SECRET
Value: [Your Dev Environment Service Principal Secret]
```

### Staging Environment Secrets

```
Name: UIPATH_STAGING_CLIENT_ID
Value: [Your Staging Environment Service Principal Client ID]
```

```
Name: UIPATH_STAGING_CLIENT_SECRET
Value: [Your Staging Environment Service Principal Secret]
```

### Production Environment Secrets

```
Name: UIPATH_PROD_CLIENT_ID
Value: [Your Production Environment Service Principal Client ID]
```

```
Name: UIPATH_PROD_CLIENT_SECRET
Value: [Your Production Environment Service Principal Secret]
```

### Optional: Code Review Secrets

```
Name: ANTHROPIC_API_KEY
Value: [Your Anthropic API Key for Claude PR reviews]
```

---

## 📋 GitHub Variables

Navigate to: **Settings → Secrets and variables → Variables → New repository variable**

### Orchestrator Configuration

```
Name: ORCHESTRATOR_URL
Value: https://your-orchestrator-instance.uipath.com
Example: https://cloud.uipath.com
Example: https://orchestrator.company.com
```

### Development Environment Variables

```
Name: UIPATH_DEV_TENANT
Value: your-tenant-id/DefaultFolder/dev
Example: acmecorp/DefaultFolder/dev
Description: Tenant path where Dev packages are uploaded
```

```
Name: UIPATH_DEV_FEED_ID
Value: [Optional] your-dev-feed-id
Description: Feed ID for package uploads (leave empty to use default)
Example: arn:aws:feed:us-east-1:123456789:feed/dev-packages
```

### Staging Environment Variables

```
Name: UIPATH_STAGING_TENANT
Value: your-tenant-id/DefaultFolder/staging
Example: acmecorp/DefaultFolder/staging
Description: Tenant path where Staging packages are uploaded
```

```
Name: UIPATH_STAGING_FEED_ID
Value: [Optional] your-staging-feed-id
Description: Feed ID for package uploads (leave empty to use default)
Example: arn:aws:feed:us-east-1:123456789:feed/staging-packages
```

### Production Environment Variables

```
Name: UIPATH_PROD_TENANT
Value: your-tenant-id/DefaultFolder/prod
Example: acmecorp/DefaultFolder/prod
Description: Tenant path where Production packages are uploaded
```

```
Name: UIPATH_PROD_FEED_ID
Value: [Optional] your-prod-feed-id
Description: Feed ID for package uploads (leave empty to use default)
Example: arn:aws:feed:us-east-1:123456789:feed/prod-packages
```

---

## 📋 GitHub Environments Configuration

Navigate to: **Settings → Environments**

### Development Environment

**Name**: `development`

**Deployment branches**: `develop` (optional - only for workflow_run trigger)

**Environment variables**: None required (inherits from repository)

**Environment secrets**: None required (inherits from repository)

**Protection rules**: (Optional)
- Add required reviewers: 1
- Add deployment branches: `develop`

---

### Staging Environment

**Name**: `staging`

**Deployment branches**: `main` (optional)

**Environment variables**: None required (inherits from repository)

**Environment secrets**: None required (inherits from repository)

**Protection rules**: (Recommended)
- ☑ Required reviewers: 1-2
- ☑ Deployment branches: `main`
- ☐ Restrict deployments to: (leave empty for all branches)

---

### Production Environment

**Name**: `production`

**Deployment branches**: None (manual trigger only)

**Environment variables**: None required (inherits from repository)

**Environment secrets**: None required (inherits from repository)

**Protection rules**: (Strongly Recommended)
- ☑ Required reviewers: 2-3 (recommend at least 2)
- ☑ Dismiss stale pull request approvals when new commits are pushed
- ☐ Deployment branches: (leave empty)
- ☐ Restrict deployments to: (leave empty)

---

## 🔍 How to Find Your Values

### Finding Service Principal Credentials

1. **Login to UiPath Orchestrator**
2. **Navigate to**: Admin → [Your Tenant Name] → External Applications
3. **Find or Create** your service principal application
4. **Copy**:
   - Application ID → Use as `CLIENT_ID`
   - Application Secret → Use as `CLIENT_SECRET`

### Finding Tenant Path

1. **Orchestrator** → Click your tenant name (top-left)
2. **Current path** shows your tenant/folder structure
3. Use format: `tenant-name/folder-path`

### Finding Orchestrator URL

The URL in your browser address bar:
```
https://your-orchestrator-instance.uipath.com
```

Use everything before `/ui/` path

### Finding Feed ID (Optional)

1. **Orchestrator** → Feeds (Admin section)
2. **Feed details** contain Feed ID
3. If not using custom feeds, leave empty

---

## ✅ Verification Checklist

After adding all secrets and variables:

- [ ] 6 secrets added (dev client id + secret, staging client id + secret, prod client id + secret)
- [ ] 7 variables added (orchestrator URL + 2 each for dev/staging/prod tenant + feed id)
- [ ] 3 environments created (development, staging, production)
- [ ] Production environment has 2+ required reviewers
- [ ] All values match your Orchestrator configuration

---

## 🧪 Quick Test

Run a test workflow to verify configuration:

```bash
# Push to develop branch
git push origin develop

# Should trigger:
# 1. Restore Dependencies
# 2. Analyzer
# 3. Package

# Check: GitHub Actions → Workflows should complete successfully
```

If workflows fail:
1. Check action logs for specific error
2. Verify secret/variable names (case-sensitive)
3. Verify values are correct (no extra spaces/quotes)
4. Confirm service principal has required permissions

---

## 🚨 Security Best Practices

**DO:**
- ✅ Store secrets in GitHub Secrets (never in code)
- ✅ Use service principals (not personal accounts)
- ✅ Rotate secrets regularly (quarterly recommended)
- ✅ Review who has access to secrets (repo Settings)
- ✅ Use different credentials per environment
- ✅ Enable branch protection on main

**DON'T:**
- ❌ Commit secrets to git repository
- ❌ Hardcode values in workflows
- ❌ Share secrets in chat/email
- ❌ Use production credentials in development
- ❌ Reuse credentials across environments

---

## 📞 Support

| Question | Answer |
|----------|--------|
| **Where do I add secrets?** | Settings → Secrets and variables → Secrets |
| **Where do I add variables?** | Settings → Secrets and variables → Variables |
| **Are variable values visible?** | Yes, variables are public. Secrets are masked. |
| **Can I use secrets in variables?** | No, reference secrets only in workflow files. |
| **How often should I rotate?** | Recommend quarterly, or if compromised. |
| **Can I test with dummy values?** | Yes, workflows will fail with clear messages if invalid. |

---

**Template Last Updated**: 2026-09-01  
**Keep This File Secure** - Do not commit to repository
