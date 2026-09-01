# 🤖 UiPath Studio XAML CI/CD Pipeline

A **production-ready GitHub Actions CI/CD pipeline** for UiPath automation projects with automated testing, multi-environment deployments, security controls, and comprehensive audit trails.

## 🎯 Features

✅ **Automated Quality Checks**
- Dependency restoration with caching
- Workflow analysis and complexity validation
- XAML linting and code quality enforcement

✅ **Intelligent Packaging**
- Semantic versioning with build numbers
- Package metadata and manifests
- Build artifact retention

✅ **Multi-Environment Deployment**
- Automated: Development (on every commit)
- Manual: Staging (with optional approval)
- Controlled: Production (manual + required approval)

✅ **Security & Compliance**
- GitHub Secrets for credential storage
- Service principal authentication (not user passwords)
- Least-privilege permissions per environment
- Comprehensive audit trails (365-day retention)
- Branch protection and approval gates

✅ **Testing Support**
- Optional test suite execution
- Multiple test suite types (unit, integration, smoke)
- Test artifact collection and reporting

✅ **Observability**
- Detailed workflow logs and step summaries
- PR comments with analysis results
- Deployment records and verification
- GitHub Deployment tracking

## 📁 Pipeline Components

### Workflows (7 Total)

| # | Workflow | Trigger | Purpose |
|---|----------|---------|---------|
| 1️⃣ | Restore Dependencies | Push/PR to main/develop | Restore project dependencies and cache |
| 2️⃣ | UiPath Analyzer | Push/PR when XAML changes | Validate code quality and complexity |
| 3️⃣ | Package Project | Push/PR/main branch | Create distributable .nupkg package |
| 4️⃣ | Execute Tests | Push/PR if tests configured | Run test suites (unit/integration/smoke) |
| 5️⃣ | Deploy to Development | Auto on package success | Deploy to Dev Orchestrator |
| 6️⃣ | Deploy to Staging | Manual with version input | Promote to Staging Orchestrator |
| 7️⃣ | Deploy to Production | Manual with approval | Deploy to Production (requires approval) |

### Documentation

📖 **Complete Documentation** - `UIPATH_PIPELINE_DOCUMENTATION.md`
- Architecture and execution flow
- Detailed workflow explanations
- Setup instructions
- Security best practices
- Customization guide

🚀 **Quick Setup Guide** - `UIPATH_PIPELINE_SETUP.md`
- 15-minute setup process
- Pre-requisite checklist
- Verification steps
- Troubleshooting quick reference

🔑 **Secrets & Variables Template** - `GITHUB_SECRETS_VARIABLES_TEMPLATE.md`
- All required secrets and variables
- How to find your values
- Environment configuration
- Verification checklist

## 🚀 Quick Start

### 1️⃣ Prerequisites (5 minutes)
- ✅ GitHub repository with Actions enabled
- ✅ UiPath project with `UiPathProject/project.json`
- ✅ Access to UiPath Orchestrator

### 2️⃣ Create Service Principals (5 minutes)
- Development: Create service principal for Dev environment
- Staging: Create service principal for Staging environment
- Production: Create service principal for Prod environment

### 3️⃣ Configure GitHub (10 minutes)
- Add 6 secrets (client IDs and secrets for each environment)
- Add 7 variables (Orchestrator URL and tenant paths)
- Create 3 environments (development, staging, production)

### 4️⃣ Enable Branch Protection (2 minutes)
- Require PR reviews
- Require status checks pass
- Require up-to-date branches

### ✅ Done! Workflows are ready to use

👉 **See `UIPATH_PIPELINE_SETUP.md` for step-by-step instructions**

## 📊 Pipeline Execution Flow

### Pull Request Workflow
```
PR Created → Restore Dependencies → Analyzer → Package → Tests → 
  ✓ Can be merged (if all checks pass)
```

### Development Deployment (Automatic)
```
Push to develop → Restore → Analyzer → Package → 
  Auto Deploy to Dev ✓
```

### Staging Promotion (Manual)
```
Manually trigger workflow → Validate version → 
  Download from Dev → Upload to Staging ✓
```

### Production Release (Manual + Approval)
```
Manually trigger workflow → Validate inputs → 
  GitHub approval gate → Deploy to Production ✓ (365-day record)
```

## 🔐 Security Features

| Feature | Implementation |
|---------|-----------------|
| **Credential Storage** | GitHub Secrets (never in code) |
| **Authentication** | Service principals (not user passwords) |
| **Permissions** | Least-privilege per environment |
| **Approval Gates** | GitHub Environment protection rules |
| **Audit Trail** | 365-day retention for production |
| **Secret Masking** | Automatic in GitHub Actions logs |
| **Version Pinning** | All action versions pinned |
| **Approval Notes** | Mandatory for production deployments |

## 📋 Required GitHub Configuration

### Secrets (6 total)
```
UIPATH_DEV_CLIENT_ID
UIPATH_DEV_CLIENT_SECRET
UIPATH_STAGING_CLIENT_ID
UIPATH_STAGING_CLIENT_SECRET
UIPATH_PROD_CLIENT_ID
UIPATH_PROD_CLIENT_SECRET
```

### Variables (7 total)
```
ORCHESTRATOR_URL
UIPATH_DEV_TENANT
UIPATH_DEV_FEED_ID (optional)
UIPATH_STAGING_TENANT
UIPATH_STAGING_FEED_ID (optional)
UIPATH_PROD_TENANT
UIPATH_PROD_FEED_ID (optional)
```

### Environments (3 total)
- `development` - Auto-deploy on success
- `staging` - Manual deployment, optional approval
- `production` - Manual only, requires 2+ approvers

👉 **See `GITHUB_SECRETS_VARIABLES_TEMPLATE.md` for detailed values**

## 📁 Project Structure

```
your-uipath-repo/
├── .github/
│   └── workflows/
│       ├── 01-dependencies-restore.yml      (Restore dependencies)
│       ├── 02-analyze.yml                   (Quality validation)
│       ├── 03-package.yml                   (Create .nupkg)
│       ├── 04-test.yml                      (Run tests)
│       ├── 05-deploy-dev.yml                (Auto to Dev)
│       ├── 06-deploy-staging.yml            (Manual to Staging)
│       └── 07-deploy-production.yml         (Manual to Prod)
├── UiPathProject/
│   ├── project.json                         (Project config)
│   ├── Main.xaml                            (Main workflow)
│   └── ...
├── tests/                                   (Optional)
│   ├── run-tests.sh
│   └── test-config.json
├── UIPATH_PIPELINE_DOCUMENTATION.md         (Full docs)
├── UIPATH_PIPELINE_SETUP.md                 (Setup guide)
├── GITHUB_SECRETS_VARIABLES_TEMPLATE.md     (Secrets template)
└── README.md                                (This file)
```

## ⚙️ Customization

### Change Deployment Folders
Edit `env.DEPLOYMENT_FOLDER` in deployment workflows

### Add New Environment
1. Copy `07-deploy-production.yml` → `08-deploy-uat.yml`
2. Create secrets: `UIPATH_UAT_CLIENT_ID`, `UIPATH_UAT_CLIENT_SECRET`
3. Create variables: `UIPATH_UAT_TENANT`, `UIPATH_UAT_FEED_ID`
4. Create GitHub Environment: `uat`

### Update UiPath CLI Version
Change `UIPATH_CLI_VERSION` environment variable in workflows

### Adjust Analyzer Rules
Modify `ANALYSIS_RULES` in `02-analyze.yml`

### Customize Version Strategy
Update version generation in `03-package.yml`

👉 **See `UIPATH_PIPELINE_DOCUMENTATION.md` Customization section for details**

## 🔍 Monitoring & Troubleshooting

### View Workflow Status
```
GitHub → Actions → Select workflow → View runs
```

### Download Build Artifacts
```
Actions → [Workflow Run] → Artifacts → Download
```

### View Detailed Logs
```
Actions → [Workflow Run] → [Job] → [Step Name]
```

### Common Issues & Solutions

| Issue | Solution |
|-------|----------|
| **CLI not found** | Ensure Node.js v20+ in workflow |
| **project.json not found** | Verify file exists in `UiPathProject/` |
| **Auth failed** | Check secrets are set, verify service principal perms |
| **Workflow not running** | Check branch filters, verify correct branch |
| **Package not found** | Check `PROJECT_PATH`, verify project structure |
| **Deployment hangs** | Check environment approval rules |

👉 **See `UIPATH_PIPELINE_SETUP.md` Troubleshooting section for more**

## 📚 Documentation Guide

| Document | Purpose | Time |
|----------|---------|------|
| `UIPATH_PIPELINE_DOCUMENTATION.md` | Comprehensive reference guide | 15-20 min read |
| `UIPATH_PIPELINE_SETUP.md` | Quick start and setup guide | 5 min read |
| `GITHUB_SECRETS_VARIABLES_TEMPLATE.md` | Secrets/variables checklist | Reference |
| `README.md` | This file - Overview | 5 min read |

## ✅ Deployment Checklist

- [ ] All 7 workflows created and visible in Actions
- [ ] 6 secrets configured and verified
- [ ] 7 variables configured and verified
- [ ] 3 GitHub Environments created
- [ ] Production environment has 2+ reviewers
- [ ] Branch protection enabled on `main`
- [ ] Test workflow triggered successfully
- [ ] Development deployment automatic and working
- [ ] Staging promotion manual and working
- [ ] Production deployment with approval working

## 🎯 Using the Pipeline

### Daily Development
```bash
# Create feature branch
git checkout -b feature/my-automation
# ... make changes ...
git commit -m "feat: add new automation"

# Create PR
git push origin feature/my-automation
# → Auto runs: Restore → Analyzer → Package → Tests

# Merge to develop after approval
# → Auto deploys to Development environment
```

### Promote to Staging
```
GitHub → Actions → Deploy to Staging → Run workflow
Input: Package version (e.g., 1.2.3.5678)
```

### Release to Production (with Approval)
```
GitHub → Actions → Deploy to Production → Run workflow
Inputs:
  - Package version: 1.2.3.5678
  - Approval notes: "Ready for production release"
Wait for reviewers to approve
→ Deploys to Production with full audit trail
```

## 📊 What You Get

- ✅ **7 production-ready workflows**
- ✅ **Automated dependency management**
- ✅ **Code quality validation**
- ✅ **Semantic versioning**
- ✅ **Multi-environment orchestration**
- ✅ **Security best practices**
- ✅ **Comprehensive audit trails**
- ✅ **Deployment records (365 days)**
- ✅ **PR integration**
- ✅ **Full documentation**

## 🚨 Security Reminders

- 🔒 **Never commit secrets** - Use GitHub Secrets
- 🔑 **Use service principals** - Not personal accounts
- 🛡️ **Enable approvals** - Especially for production
- 🔄 **Rotate credentials** - Quarterly recommended
- 📝 **Keep audit logs** - 365 days for production
- 👥 **Limit access** - Use least-privilege permissions

## 📞 Support Resources

- 📖 [UiPath CLI Documentation](https://docs.uipath.com/automation-suite/latest/user-guide/uipath-cli)
- 🔐 [GitHub Actions Security Guide](https://docs.github.com/en/actions/security-guides)
- 🤖 [UiPath Orchestrator Docs](https://docs.uipath.com/orchestrator/latest/user-guide)
- 👤 [Service Principal Setup](https://docs.uipath.com/automation-suite/latest/user-guide/orchestrator/managing-access-and-authentication)

## 🎓 Next Steps

1. **Read Setup Guide**: `UIPATH_PIPELINE_SETUP.md` (15 minutes)
2. **Configure Secrets**: Add GitHub secrets and variables (10 minutes)
3. **Create Environments**: Set up GitHub environments (5 minutes)
4. **Test Pipeline**: Push to develop and verify workflows (5 minutes)
5. **Enable Production**: Configure production approval rules
6. **Read Full Docs**: Reference `UIPATH_PIPELINE_DOCUMENTATION.md` as needed

## 📊 Pipeline Statistics

- **Total Workflows**: 7
- **Automation Coverage**: ~95% of typical deployment process
- **Manual Steps**: 2 (Staging & Production promotions)
- **Approval Gates**: Production deployment
- **Audit Trail**: 365 days for production
- **Average Setup Time**: 30-45 minutes

## ✨ Highlights

🚀 **Zero-Touch Development** - Push to develop = auto-deployed to Dev  
🔒 **Enterprise Security** - Secrets, least-privilege, audit trails  
✅ **Quality First** - Analysis and validation before packaging  
📊 **Full Observability** - Logs, artifacts, deployment records  
🎯 **Multi-Environment** - Dev → Staging → Prod promotion flow  
🛡️ **Controlled Releases** - Manual approval for production  

---

## 📝 License & Usage

This CI/CD pipeline is provided as-is for use with UiPath automation projects. Customize for your organization's requirements.

---

**Pipeline Version**: 1.0.0  
**Last Updated**: 2026-09-01  
**Status**: ✅ Production Ready

👉 **Start Setup**: See `UIPATH_PIPELINE_SETUP.md`  
📖 **Full Reference**: See `UIPATH_PIPELINE_DOCUMENTATION.md`
