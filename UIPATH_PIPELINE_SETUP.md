# UiPath CI/CD Pipeline - Quick Setup Guide

## 📋 Pre-Requisites Checklist

- [ ] GitHub repository with Actions enabled
- [ ] UiPath Project with `project.json` in `UiPathProject/` directory
- [ ] Access to UiPath Orchestrator (cloud instance or on-premises)
- [ ] Ability to create service principals in Orchestrator
- [ ] Repository Settings access

## 🚀 Quick Setup (15-20 minutes)

### Step 1: Create Service Principals in UiPath Orchestrator (per environment)

For each environment (Dev, Staging, Production):

1. **Login to UiPath Orchestrator**
2. **Navigate**: Admin → [Your Tenant] → External Applications
3. **Create Application** (repeat for each environment):
   - Name: `github-actions-dev` (or `-staging`, `-prod`)
   - Type: Service User
   - Folder: Select appropriate environment folder
4. **Copy credentials**:
   - Client ID: `<SAVE FOR STEP 3>`
   - Client Secret: `<SAVE FOR STEP 3>`

### Step 2: Add GitHub Secrets (via Settings)

**Path**: Repository → Settings → Secrets and variables → Secrets

Add the following secrets:

```
UIPATH_DEV_CLIENT_ID = [From Step 1]
UIPATH_DEV_CLIENT_SECRET = [From Step 1]

UIPATH_STAGING_CLIENT_ID = [From Step 1]
UIPATH_STAGING_CLIENT_SECRET = [From Step 1]

UIPATH_PROD_CLIENT_ID = [From Step 1]
UIPATH_PROD_CLIENT_SECRET = [From Step 1]
```

### Step 3: Add GitHub Variables (via Settings)

**Path**: Repository → Settings → Secrets and variables → Variables

Add the following variables:

```
ORCHESTRATOR_URL = https://your-orchestrator-instance.uipath.com

UIPATH_DEV_TENANT = tenant/DefaultFolder/dev
UIPATH_DEV_FEED_ID = [Optional - your dev feed ID]

UIPATH_STAGING_TENANT = tenant/DefaultFolder/staging
UIPATH_STAGING_FEED_ID = [Optional - your staging feed ID]

UIPATH_PROD_TENANT = tenant/DefaultFolder/prod
UIPATH_PROD_FEED_ID = [Optional - your prod feed ID]
```

### Step 4: Configure GitHub Environments

**Path**: Repository → Settings → Environments

#### Create Development Environment:
- Name: `development`
- Click "Create environment"
- Deployment branches: `develop`

#### Create Staging Environment:
- Name: `staging`
- Click "Create environment"
- Add required reviewers: 1+ (recommended)
- Deployment branches: `main`

#### Create Production Environment:
- Name: `production`
- Click "Create environment"
- Add required reviewers: 2-3 (recommended)
- Restrict deployments to: None (manual only)

### Step 5: Verify Workflows Are Loaded

**Path**: Repository → Actions

You should see:
- ✅ Restore Dependencies
- ✅ UiPath Workflow Analyzer
- ✅ Package UiPath Project
- ✅ Execute Tests
- ✅ Deploy to Development
- ✅ Deploy to Staging
- ✅ Deploy to Production

## ✅ Verification Steps

### 1. Trigger Initial Build

```bash
git push origin develop
```

**Check**: GitHub Actions → Restore Dependencies workflow should run

### 2. Check Analyzer Validation

Create a test pull request:

```bash
git checkout -b test-pr
echo "# Test" >> README.md
git add README.md
git commit -m "test: verify analyzer"
git push origin test-pr
```

**Check**: 
- PR should trigger Analyzer
- Should show results in PR comments
- Packaging should happen if no errors

### 3. Verify Package Creation

After successful packaging run:

```bash
# Navigate to: Actions → Package UiPath Project → Latest Run
# Check: Artifacts should show "uipath-package-*"
```

### 4. Verify Development Deployment

On successful package:

**Check**: Actions → Deploy to Development should auto-trigger

### 5. Test Staging Deployment

```bash
# Actions → Deploy to Staging → Run workflow
# Input package version: [from package workflow]
```

### 6. Test Production Deployment (Caution!)

```bash
# Actions → Deploy to Production → Run workflow
# Inputs:
#   - package-version: X.Y.Z.N
#   - approval-notes: "Testing production deployment"
```

## 📊 Monitoring Workflows

### View All Runs
```
GitHub → Actions → Filter by workflow name
```

### Download Artifacts
```
GitHub → Actions → [Workflow Run] → Artifacts
```

### Check Detailed Logs
```
GitHub → Actions → [Workflow Run] → [Job Name] → [Step Name]
```

## 🔍 Troubleshooting Quick Reference

| Issue | Solution |
|-------|----------|
| **"No UiPath CLI found"** | Node.js v20+ required, check runner |
| **"project.json not found"** | Ensure file exists in `UiPathProject/` |
| **"Authentication failed"** | Verify secrets are set correctly, check service principal permissions |
| **"Package not found"** | Check `PROJECT_PATH` env var, verify project structure |
| **"Workflow not running"** | Check branch filters, verify push/PR targets correct branch |
| **"Deployment hangs"** | Check environment approval rules, ensure reviewers available |

## 📁 Expected Project Structure

```
your-uipath-repo/
├── .github/
│   └── workflows/
│       ├── 01-dependencies-restore.yml
│       ├── 02-analyze.yml
│       ├── 03-package.yml
│       ├── 04-test.yml
│       ├── 05-deploy-dev.yml
│       ├── 06-deploy-staging.yml
│       └── 07-deploy-production.yml
├── UiPathProject/
│   ├── project.json
│   ├── package.json
│   ├── Main.xaml
│   └── ... other XAML files
├── tests/ (optional)
│   ├── run-tests.sh
│   └── test-config.json
├── UIPATH_PIPELINE_DOCUMENTATION.md
├── UIPATH_PIPELINE_SETUP.md
└── README.md
```

## 🎯 Common Workflows

### Daily Development
1. Create feature branch from `develop`
2. Commit changes
3. Create PR → Auto runs analyzer & package
4. Merge to `develop` → Auto deploys to Dev

### Promote to Staging
```
Actions → Deploy to Staging → Run workflow
Input: package version from package workflow run
```

### Release to Production (Requires Approval)
```
Actions → Deploy to Production → Run workflow
Inputs:
  - package-version: X.Y.Z.N
  - approval-notes: "[reason for release]"
Wait for reviewers to approve
```

## 🔐 Security Checklist

- [ ] Service principals created (not personal accounts)
- [ ] Secrets stored in GitHub (not in code)
- [ ] Environment approval rules configured
- [ ] Branch protection rules enabled on `main`
- [ ] Production environment requires 2+ reviewers
- [ ] Orchestrator URLs stored as variables (not hardcoded)
- [ ] CLI versions pinned to specific versions
- [ ] Test against UAT before production deployment

## 📚 Next Steps

1. **Read Full Documentation**: See `UIPATH_PIPELINE_DOCUMENTATION.md`
2. **Customize for Your Environment**:
   - Update orchestrator URL
   - Adjust folder names
   - Configure analyzer rules
3. **Set Up Monitoring**: Configure Slack/Teams notifications (optional)
4. **Document Your Process**: Update README with team-specific instructions
5. **Test All Environments**: Verify each deployment path works

## ❓ Need Help?

| Resource | Link |
|----------|------|
| UiPath CLI Docs | https://docs.uipath.com/automation-suite/latest/user-guide/uipath-cli |
| GitHub Actions | https://docs.github.com/en/actions |
| UiPath Orchestrator | https://docs.uipath.com/orchestrator/latest/user-guide |
| Service Principals | https://docs.uipath.com/automation-suite/latest/user-guide/orchestrator/managing-access-and-authentication |

---

**Setup Status**: Ready to Deploy  
**Last Updated**: 2026-09-01
