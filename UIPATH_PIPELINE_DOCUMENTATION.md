# UiPath Studio XAML CI/CD Pipeline Documentation

## Overview

This documentation describes a production-ready CI/CD pipeline for UiPath Studio XAML automation projects using GitHub Actions. The pipeline follows industry best practices with separation of concerns, environment-based deployments, security controls, and comprehensive validation.

## Pipeline Architecture

### Workflow Execution Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                     GitHub Events Trigger                        │
│  (Push to develop/main, PR, Manual Workflow Dispatch)           │
└──────────────────────────┬──────────────────────────────────────┘
                           │
            ┌──────────────┴──────────────┐
            ▼                             ▼
    ┌───────────────┐             ┌──────────────┐
    │ Dependencies  │             │   Analyzer   │
    │   Restore     │             │ Validation   │
    └───────┬───────┘             └──────┬───────┘
            │                            │
            └──────────────┬─────────────┘
                          ▼
                   ┌─────────────┐
                   │   Package   │
                   │   Project   │
                   └──────┬──────┘
                          │
                   ┌──────┴──────┐
                   ▼             ▼
            ┌────────────┐  ┌─────────┐
            │ Test Suite │  │ Dev env │
            │ (optional) │  │ Deploy  │
            └────────────┘  └────┬────┘
                                 │
                          ┌──────▼──────┐
                          │  Staging    │
                          │  Deployment │
                          │  (Manual)   │
                          └──────┬──────┘
                                 │
                          ┌──────▼──────┐
                          │ Production  │
                          │ Deployment  │
                          │ (Manual+Env)│
                          └─────────────┘
```

## Workflows Overview

### 1. **Restore Dependencies** (`01-dependencies-restore.yml`)

**Purpose**: Validates and restores all UiPath project dependencies.

**When It Runs**:
- Manual trigger via workflow dispatch
- Automatic on push to `main` or `develop` when:
  - `UiPathProject/project.json` changes
  - `UiPathProject/package.json` changes
- Automatic on pull requests to `main`/`develop` with same path filters

**Key Responsibilities**:
- ✓ Install Node.js and UiPath CLI
- ✓ Verify `project.json` exists
- ✓ Restore project dependencies
- ✓ Verify project structure
- ✓ Cache dependencies for faster subsequent builds

**Secrets Required**: None

**Variables Required**: None

**Outputs**:
- Cached dependencies for use in other workflows
- Build logs and verification status

**Why It Matters**:
- Ensures all dependencies are available before analysis and packaging
- Validates project configuration early in the pipeline
- Caches dependencies to speed up subsequent builds

---

### 2. **UiPath Workflow Analyzer** (`02-analyze.yml`)

**Purpose**: Validates XAML workflows using UiPath Workflow Analyzer for code quality, complexity, and maintainability.

**When It Runs**:
- Manual trigger via workflow dispatch
- Automatic on push to `main` or `develop` when:
  - Any `*.xaml` files change in `UiPathProject/`
  - `project.json` changes
- Automatic on pull requests to `main`/`develop` with same filters

**Key Responsibilities**:
- ✓ Restore project dependencies
- ✓ Run UiPath Analyzer with configurable rules
- ✓ Generate analysis report (JSON format)
- ✓ Parse results and fail on critical errors
- ✓ Comment on PRs with summary
- ✓ Upload analysis artifacts

**Analysis Rules**:
- `MaintainabilityIndex`: Code structure and organization
- `SequenceComplexity`: Workflow sequence complexity
- `LowFlowComplexity`: Flow control complexity

**Secrets Required**: None

**Variables Required**: None

**Outputs**:
- `analysis-results.json` - Detailed analyzer report
- PR comments with findings (on pull requests)
- Analysis artifact (30-day retention)

**Failure Conditions**:
- Any ERROR-level issues found
- More than 10 WARNING-level issues

**Why It Matters**:
- Prevents low-quality code from entering main branches
- Catches complexity issues early
- Provides actionable feedback on PRs

---

### 3. **Package UiPath Project** (`03-package.yml`)

**Purpose**: Packages the UiPath project into a distributable `.nupkg` file with version management.

**When It Runs**:
- Manual trigger via workflow dispatch
- Automatic on push to `main`, `develop`, or `staging` when:
  - Any files in `UiPathProject/` change
  - Workflow file changes
- Automatic on pull requests to `main`

**Key Responsibilities**:
- ✓ Restore dependencies
- ✓ Run analyzer (non-blocking)
- ✓ Generate semantic version: `BASE_VERSION.BUILD_NUMBER`
- ✓ Package project into `.nupkg`
- ✓ Verify package integrity
- ✓ Generate deployment manifest
- ✓ Upload artifacts

**Versioning Strategy**:
- Base version: Extracted from `project.json`
- Full version: `BASE_VERSION.GITHUB_RUN_NUMBER`
- Example: `1.2.3.5678` (v1.2.3, run #5678)

**Secrets Required**: None

**Variables Required**: None

**Outputs**:
```
outputs:
  package-path: <full path to .nupkg file>
  package-name: <filename of package>
  package-version: <semantic version used>
```

**Artifacts**:
- `uipath-package-{VERSION}` containing:
  - `.nupkg` file
  - `manifest.json` (metadata)

**Why It Matters**:
- Creates distributable, versioned packages
- Ensures consistent naming and metadata
- Provides package for deployment to orchestrators

---

### 4. **Execute Tests** (`04-test.yml`)

**Purpose**: Runs test suites if configured in the project.

**When It Runs**:
- Manual trigger via workflow dispatch (select test suite)
- Automatic on push to `main` or `develop` when:
  - `UiPathProject/` changes
  - Test files change
- Automatic on pull requests to `main`

**Supported Test Suites**:
- `all` - All test suites
- `unit` - Unit tests only
- `integration` - Integration tests
- `smoke` - Smoke/regression tests

**Prerequisites**:
- Test configuration: `tests/run-tests.sh` or `tests/test-config.json`

**Key Responsibilities**:
- ✓ Check if test configuration exists
- ✓ Install CLI and dependencies
- ✓ Execute selected test suite
- ✓ Capture test results
- ✓ Upload artifacts
- ✓ Comment on PRs with execution status

**Secrets Required**: None

**Variables Required**: None

**Artifacts**:
- `test-results` - Test output and reports (30-day retention)

**Note**: Workflow gracefully skips if no test configuration is found.

**Why It Matters**:
- Validates automation logic before deployment
- Catches regressions early
- Provides audit trail of test execution

---

### 5. **Deploy to Development** (`05-deploy-dev.yml`)

**Purpose**: Deploys packages to the Development Orchestrator for testing.

**When It Runs**:
- Automatic on successful `03-package.yml` completion on `develop` branch
- Manual trigger via workflow dispatch
- Triggered by package workflow artifacts

**Environment**: `development`
- **GitHub Environment**: Configured as `development` for protection rules
- **Orchestrator Deployment Folder**: `DefaultFolder` (customize as needed)

**Key Responsibilities**:
- ✓ Download package from artifacts
- ✓ Validate credentials
- ✓ Authenticate with Development Orchestrator
- ✓ Upload package to development feed
- ✓ Verify deployment
- ✓ Generate deployment record

**Secrets Required**:
| Secret | Purpose |
|--------|---------|
| `UIPATH_DEV_CLIENT_ID` | Service principal client ID for Dev |
| `UIPATH_DEV_CLIENT_SECRET` | Service principal secret for Dev |

**Variables Required**:
| Variable | Purpose |
|----------|---------|
| `UIPATH_DEV_TENANT` | Dev tenant identifier |
| `UIPATH_DEV_FEED_ID` | Feed ID for package uploads (optional) |
| `ORCHESTRATOR_URL` | Orchestrator base URL |

**Artifacts**:
- `deployment-record-development` (90-day retention)

**Why It Matters**:
- First deployment validates package in running environment
- Development is low-risk for testing
- Deployment records provide audit trail

---

### 6. **Deploy to Staging** (`06-deploy-staging.yml`)

**Purpose**: Promotes validated packages from Development to Staging for pre-production testing.

**When It Runs**:
- Manual trigger only (via workflow dispatch)
- Can be triggered by workflow run completion notification
- Requires explicit package version input

**Environment**: `staging`
- **GitHub Environment**: Configured as `staging` with approval rules (recommended)
- **Orchestrator Deployment Folder**: `StagingFolder` (customize as needed)

**Input Parameters**:
- `package-version` (required) - Semantic version to promote (e.g., `1.2.3.5678`)

**Key Responsibilities**:
- ✓ Request approval if triggered by workflow run
- ✓ Validate package version format
- ✓ Download package from Dev feed
- ✓ Upload to Staging feed
- ✓ Create promotion record
- ✓ Generate deployment summary

**Secrets Required**:
| Secret | Purpose |
|--------|---------|
| `UIPATH_STAGING_CLIENT_ID` | Service principal client ID for Staging |
| `UIPATH_STAGING_CLIENT_SECRET` | Service principal secret for Staging |

**Variables Required**:
| Variable | Purpose |
|----------|---------|
| `UIPATH_STAGING_TENANT` | Staging tenant identifier |
| `UIPATH_STAGING_FEED_ID` | Feed ID for package uploads (optional) |
| `ORCHESTRATOR_URL` | Orchestrator base URL |

**Promotion Flow**:
1. Request approval (if workflow run triggered)
2. Validate version format
3. Download from Development feed
4. Upload to Staging feed
5. Generate deployment record

**Why It Matters**:
- Manual promotion control ensures deliberate staging deployment
- Staging mirrors production for realistic testing
- Approval gates prevent accidental deployments

---

### 7. **Deploy to Production** (`07-deploy-production.yml`)

**Purpose**: Controlled, audited deployment to Production Orchestrator with mandatory approval.

**When It Runs**:
- **Manual only** - workflow dispatch with required inputs
- No automatic triggers
- Requires explicit approval inputs

**Environment**: `production`
- **GitHub Environment**: Must be configured with required reviewers
- **Orchestrator Deployment Folder**: `ProductionFolder` (customize as needed)

**Required Input Parameters**:
| Parameter | Description | Validation |
|-----------|-------------|-----------|
| `package-version` | Semantic version to deploy | Format: X.Y.Z or X.Y.Z.N |
| `approval-notes` | Reason and approval notes | Required, min 1 char |

**Key Responsibilities**:
- ✓ Validate package version format
- ✓ Require approval notes
- ✓ Create deployment request record
- ✓ Authenticate with Production Orchestrator
- ✓ Download from Staging feed
- ✓ Upload to Production feed
- ✓ Verify deployment
- ✓ Generate comprehensive audit record
- ✓ Create GitHub Deployment record

**Secrets Required**:
| Secret | Purpose |
|--------|---------|
| `UIPATH_PROD_CLIENT_ID` | Service principal client ID for Production |
| `UIPATH_PROD_CLIENT_SECRET` | Service principal secret for Production |

**Variables Required**:
| Variable | Purpose |
|----------|---------|
| `UIPATH_PROD_TENANT` | Production tenant identifier |
| `UIPATH_PROD_FEED_ID` | Feed ID for package uploads (optional) |
| `ORCHESTRATOR_URL` | Orchestrator base URL |

**Approval Gates**:
- GitHub Environment protection rules (recommended 2+ reviewers)
- Mandatory approval notes input
- Version format validation
- All secrets/variables must be present

**Deployment Record** (365-day retention):
```json
{
  "environment": "production",
  "packageVersion": "X.Y.Z.N",
  "deployedAt": "ISO-8601 timestamp",
  "deployedBy": "GitHub actor",
  "approvalNotes": "Deployment reason",
  "commitSha": "git commit hash",
  "runId": "GitHub run ID"
}
```

**Why It Matters**:
- **Strict Controls**: Mandatory approval and approval notes
- **Audit Trail**: Comprehensive records retained 365 days
- **Version Safety**: Semantic version validation
- **Low Risk**: Only manual triggering, no automatic deployments

---

## Setup Instructions

### 1. Create GitHub Secrets

All secrets must be added to repository Settings > Secrets and variables:

#### Development Secrets:
```
UIPATH_DEV_CLIENT_ID = "<client-id-for-dev>"
UIPATH_DEV_CLIENT_SECRET = "<client-secret-for-dev>"
```

#### Staging Secrets:
```
UIPATH_STAGING_CLIENT_ID = "<client-id-for-staging>"
UIPATH_STAGING_CLIENT_SECRET = "<client-secret-for-staging>"
```

#### Production Secrets:
```
UIPATH_PROD_CLIENT_ID = "<client-id-for-prod>"
UIPATH_PROD_CLIENT_SECRET = "<client-secret-for-prod>"
```

#### Optional for Code Review:
```
ANTHROPIC_API_KEY = "<your-anthropic-api-key>"  # For Claude PR reviews
```

### 2. Create GitHub Variables

Repository Settings > Secrets and variables > Variables:

#### Orchestrator Configuration:
```
ORCHESTRATOR_URL = "https://your-orchestrator-instance.uipath.com"
```

#### Development Environment:
```
UIPATH_DEV_TENANT = "tenant/folder/dev"
UIPATH_DEV_FEED_ID = "feed-id-for-dev"  # Optional
```

#### Staging Environment:
```
UIPATH_STAGING_TENANT = "tenant/folder/staging"
UIPATH_STAGING_FEED_ID = "feed-id-for-staging"  # Optional
```

#### Production Environment:
```
UIPATH_PROD_TENANT = "tenant/folder/prod"
UIPATH_PROD_FEED_ID = "feed-id-for-prod"  # Optional
```

### 3. Configure GitHub Environments

**Settings > Environments**:

#### Development Environment:
- **Name**: `development`
- **Deployment branches**: `develop`
- **Environment secrets**: (inherit from repo)

#### Staging Environment:
- **Name**: `staging`
- **Deployment branches**: `main`
- **Environment secrets**: (inherit from repo)
- **Required reviewers**: 1+ (recommended)

#### Production Environment:
- **Name**: `production`
- **Deployment branches**: None (manual only)
- **Environment secrets**: (inherit from repo)
- **Required reviewers**: 2+ (strongly recommended)
- **Restrict deployments to**: Production branch protection rules

### 4. Create UiPath Service Principals

For each environment (Dev, Staging, Prod), create a service principal in UiPath Orchestrator:

**Steps**:
1. Orchestrator → Admin → Tenants → [Your Tenant] → External Applications
2. Create Application
3. Configure:
   - **User Type**: Service User
   - **Folder**: Appropriate folder for environment
   - **Permissions**: 
     - Package Upload/Download
     - Process Execution
     - Release Management (if needed)
4. Generate credentials
5. Store Client ID and Secret in GitHub Secrets

### 5. Update Project Configuration

**Ensure `UiPathProject/project.json` exists** with:
```json
{
  "name": "YourProjectName",
  "version": "1.0.0",
  ...
}
```

**Create `tests/` directory** if test execution is needed:
```
tests/
├── run-tests.sh
└── test-config.json
```

### 6. Enable Branch Protection

**Settings > Branches > Branch protection rules** for `main`:
- Require pull request reviews (2+)
- Require status checks to pass:
  - Analyzer must succeed
  - Package must build successfully
- Require branches to be up to date
- Dismiss stale PR approvals on push
- Allow force pushes: No
- Allow deletions: No

---

## Execution Order and Dependencies

### Pull Request Workflow
1. **Restore Dependencies** (automatic)
2. **Analyzer** (automatic, parallel with restore)
3. **Package** (automatic, on successful analysis)
4. **Tests** (automatic if configured)
5. **Manual Code Review** (GitHub required)

→ **Result**: PR can be merged if all checks pass

### Development Branch Deployment
1. **Restore Dependencies** (automatic on push)
2. **Analyzer** (automatic)
3. **Package** (automatic)
4. **Tests** (automatic if configured)
5. **Deploy to Development** (automatic on package success)

→ **Result**: Package automatically deployed to Dev environment

### Staging Promotion
1. Manual trigger: `Deploy to Staging` workflow
2. Input: Package version (e.g., `1.2.3.5678`)
3. Validation: Version format check
4. Promotion: Download from Dev feed → Upload to Staging feed
5. Optional: Approval gate (recommended)

→ **Result**: Validated package available in Staging

### Production Deployment
1. Manual trigger: `Deploy to Production` workflow
2. Required Inputs:
   - Package version (e.g., `1.2.3.5678`)
   - Approval notes (reason for deployment)
3. Validation:
   - Version format
   - Credentials present
   - Approval gate (GitHub Environment rules)
4. Deployment:
   - Download from Staging feed
   - Upload to Production feed
   - Verify in production tenant
5. Record: 365-day audit trail

→ **Result**: Package live in Production with full audit trail

---

## Security Best Practices Implemented

✓ **Credential Management**:
- All credentials stored in GitHub Secrets
- Never logged or exposed
- Environment-specific secrets
- Service principal authentication (not user credentials)

✓ **Least Privilege**:
- Service principals scoped to specific folders
- Minimal required permissions per environment
- Environment-specific access controls

✓ **Workflow Security**:
- Explicit `permissions:` blocks for each job
- No unnecessary access to repository
- Token scoped to required operations

✓ **Approval Gates**:
- Manual-only production deployments
- Mandatory approval notes
- GitHub Environment protection rules
- Reviewer requirements (recommended 2+ for prod)

✓ **Audit Trail**:
- Comprehensive deployment records
- 365-day retention for production
- Git commit SHA and actor tracking
- Deployment timestamps

✓ **Action Versions**:
- All actions pinned to specific versions
- No floating `@latest` tags
- Reduces risk of action compromises

✓ **Secrets Masking**:
- GitHub Actions automatically masks secrets in logs
- Scripts use environment variables, not inline values
- Credentials never printed to console

---

## Monitoring and Troubleshooting

### Workflow Status
- **GitHub Actions Tab**: View all workflow runs
- **Artifacts**: Download build outputs and reports
- **Logs**: Check detailed step-by-step logs

### Common Issues

#### Package Not Found After Packaging
- Ensure `UiPathProject/project.json` exists
- Check `PROJECT_PATH` variable in workflow
- Verify UiPath CLI version compatibility

#### Authentication Failures
- Verify secrets are correctly set
- Check service principal still has required permissions
- Confirm orchestrator URL is correct
- Test credentials manually if needed

#### Analyzer Finding Too Many Warnings
- Update project to address issues
- Consider raising threshold in workflow if appropriate
- Review XAML structure and complexity

#### Deployment to Wrong Tenant
- Verify `UIPATH_*_TENANT` variables
- Confirm service principal folder assignment
- Check feed ID (if applicable)

### Debug Steps
1. Check workflow logs in GitHub Actions tab
2. Verify all secrets/variables are set
3. Confirm branch/path filters match trigger
4. Test manually with UiPath CLI:
   ```bash
   uip rpa restore ./UiPathProject
   uip rpa analyze ./UiPathProject
   uip rpa pack ./UiPathProject --output ./output
   ```

---

## Customization Guide

### Changing Deployment Folders
Edit deployment workflows to specify different folders:
```yaml
env:
  DEPLOYMENT_FOLDER: "CustomFolder/SubFolder"
```

### Adding More Environments
1. Create new workflow: `08-deploy-uat.yml` (copy production template)
2. Create secrets: `UIPATH_UAT_CLIENT_ID`, etc.
3. Create variables: `UIPATH_UAT_TENANT`, etc.
4. Add GitHub Environment: `uat`
5. Add approval rule if needed

### Customizing Analyzer Rules
Update `ANALYSIS_RULES` in `02-analyze.yml`:
```yaml
env:
  ANALYSIS_RULES: "MaintainabilityIndex,SequenceComplexity,LowFlowComplexity,VBNetCompatibility"
```

### Updating CLI Version
Change `UIPATH_CLI_VERSION` in any workflow:
```yaml
env:
  UIPATH_CLI_VERSION: "1.1.0"  # Update version
```

### Adjusting Version Strategy
Modify version generation in `03-package.yml`:
```bash
# Current: BASE_VERSION.BUILD_NUMBER
# Alternative: BASE_VERSION-build.BUILD_NUMBER
PACKAGE_VERSION="${BASE_VERSION}-build.${GITHUB_RUN_NUMBER}"
```

---

## Support and References

- **UiPath CLI Documentation**: [UiPath CLI Docs](https://docs.uipath.com/automation-suite/latest/user-guide/uipath-cli)
- **GitHub Actions Best Practices**: [GitHub Actions Security](https://docs.github.com/en/actions/security-guides)
- **UiPath Orchestrator**: [Orchestrator Admin Guide](https://docs.uipath.com/orchestrator/latest/user-guide/orchestrator)
- **Service Principals**: [UiPath Service Principal Setup](https://docs.uipath.com/automation-suite/latest/user-guide/orchestrator/managing-access-and-authentication)

---

## Glossary

- **Orchestrator**: UiPath cloud platform for managing automation processes
- **Package (.nupkg)**: Distributable automation project file
- **Service Principal**: Non-human identity for API authentication
- **Feed**: Orchestrator component for managing package versions
- **Tenant**: Isolated workspace in Orchestrator
- **Folder**: Organizational unit within a tenant
- **CLI**: Command-line interface for UiPath operations
- **XAML**: Extensible Application Markup Language (UiPath workflow format)

---

**Last Updated**: 2026-09-01  
**Pipeline Version**: 1.0.0  
**Maintained By**: Platform Engineering Team
