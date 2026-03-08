# Reusable CI/CD Workflows

Collection of reusable GitHub Actions workflows for Terraform projects. These workflows provide standardized CI/CD pipelines for linting, security scanning, and releasing Terraform modules.

## Table of Contents

- [Overview](#overview)
- [Workflows](#workflows)
  - [Terraform Lint](#terraform-lint)
  - [Checkov Security Scan](#checkov-security-scan)
  - [Terraform Release](#terraform-release)
- [Usage Examples](#usage-examples)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

## Overview

This repository contains reusable GitHub Actions workflows designed to standardize CI/CD processes across Terraform projects. Each workflow is designed to be called from other repositories using the `workflow_call` trigger.

**Benefits:**
- Centralized workflow management
- Consistent CI/CD practices across all Terraform modules
- Easy updates - fix once, apply everywhere
- Reduced duplication in repository workflows

## Workflows

### Terraform Lint

**File:** `.github/workflows/terraform-lint.yaml`

Comprehensive Terraform code quality checks including formatting, linting, and validation.

#### What It Does

1. **Terraform Format Check** - Ensures code follows Terraform formatting standards
2. **TFLint** - Static analysis for Terraform code to catch errors and enforce best practices
3. **Terraform Validate** - Validates Terraform configuration syntax and internal consistency

#### Execution Order

The workflow runs three parallel jobs:

**Job 1: terraform-fmt**
- Checks if all `.tf` files are properly formatted
- Runs `terraform fmt -check -recursive`
- Fails if any files need formatting

**Job 2: tflint**
- Sets up Terraform (required for module resolution)
- Runs `terraform init -backend=false` to download modules
- Initializes TFLint plugins with `tflint --init`
- Runs `tflint --recursive --format compact`
- Checks for errors, warnings, and best practice violations

**Job 3: terraform-validate**
- Finds all directories containing `.tf` files
- Runs `terraform init` and `terraform validate` in each directory
- Skips directories that aren't root modules
- Reports validation failures

#### Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `terraform_version` | Terraform version to use | No | `1.14.3` |
| `working_directory` | Working directory for Terraform commands | No | `.` |
| `tflint_version` | TFLint version to use | No | `latest` |

#### Usage

```yaml
name: Terraform Lint

on:
  push:
    paths:
      - '**.tf'
      - '**.tfvars'
  pull_request:
    branches:
      - main

jobs:
  lint:
    uses: islamelkadi/reusable-cicd-workflows/.github/workflows/terraform-lint.yaml@main
    with:
      terraform_version: '1.14.3'
      tflint_version: 'latest'
      working_directory: '.'
```

#### Requirements

- Repository must have a `.tflint.hcl` configuration file
- Terraform code must be in the specified `working_directory`
- For modules with dependencies, ensure they're accessible (public repos or with proper credentials)

---

### Checkov Security Scan

**File:** `.github/workflows/checkov-scan.yaml`

Security and compliance scanning for Terraform code using Checkov.

#### What It Does

1. Scans Terraform code for security vulnerabilities
2. Checks compliance with industry standards (CIS, PCI-DSS, HIPAA, etc.)
3. Identifies misconfigurations and security risks
4. Provides detailed reports with remediation guidance

#### Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `working_directory` | Directory to scan | No | `.` |
| `config_file` | Path to Checkov config file | No | `.checkov.yml` |

#### Usage

```yaml
name: Security Scan

on:
  push:
    paths:
      - '**.tf'
  pull_request:
    branches:
      - main

jobs:
  security:
    uses: islamelkadi/reusable-cicd-workflows/.github/workflows/checkov-scan.yaml@main
    with:
      working_directory: '.'
      config_file: '.checkov.yml'
```

#### Configuration

Create a `.checkov.yml` file in your repository to customize scanning:

```yaml
framework:
  - terraform
skip-check:
  - CKV_AWS_123  # Skip specific checks with justification
quiet: false
compact: true
```

---

### Terraform Release

**File:** `.github/workflows/terraform-release.yaml`

Automated semantic versioning and GitHub releases using semantic-release.

#### What It Does

1. Analyzes commit messages to determine version bump (major, minor, patch)
2. Generates changelog from commit history
3. Creates Git tags
4. Publishes GitHub releases
5. Updates version documentation

#### Commit Message Format

Uses [Conventional Commits](https://www.conventionalcommits.org/):

- `feat:` - New feature (minor version bump)
- `fix:` - Bug fix (patch version bump)
- `docs:` - Documentation changes (no version bump)
- `chore:` - Maintenance tasks (no version bump)
- `BREAKING CHANGE:` - Breaking change (major version bump)

**Examples:**
```
feat: add support for VPC endpoints
fix: correct IAM policy attachment
docs: update README with new examples
chore: update dependencies
```

#### Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `node_version` | Node.js version for semantic-release | No | `lts/*` |

#### Secrets

| Secret | Description | Required |
|--------|-------------|----------|
| `token` | GitHub token with repo permissions | Yes |

#### Usage

```yaml
name: Release

on:
  push:
    branches:
      - main

jobs:
  release:
    uses: islamelkadi/reusable-cicd-workflows/.github/workflows/terraform-release.yaml@main
    with:
      node_version: 'lts/*'
    secrets:
      token: ${{ secrets.GITHUB_TOKEN }}
```

#### Requirements

- Repository must have a `.releaserc.json` configuration file
- Commits must follow Conventional Commits format
- `GITHUB_TOKEN` must have write permissions for releases

---

## Usage Examples

### Complete CI/CD Pipeline

Combine all workflows for a complete CI/CD pipeline:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  lint:
    uses: islamelkadi/reusable-cicd-workflows/.github/workflows/terraform-lint.yaml@main
    with:
      terraform_version: '1.14.3'
      tflint_version: 'latest'

  security:
    uses: islamelkadi/reusable-cicd-workflows/.github/workflows/checkov-scan.yaml@main
    with:
      config_file: '.checkov.yml'

  release:
    needs: [lint, security]
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    uses: islamelkadi/reusable-cicd-workflows/.github/workflows/terraform-release.yaml@main
    secrets:
      token: ${{ secrets.GITHUB_TOKEN }}
```

### Multi-Directory Project

For projects with multiple Terraform directories:

```yaml
jobs:
  lint-root:
    uses: islamelkadi/reusable-cicd-workflows/.github/workflows/terraform-lint.yaml@main
    with:
      working_directory: '.'

  lint-modules:
    uses: islamelkadi/reusable-cicd-workflows/.github/workflows/terraform-lint.yaml@main
    with:
      working_directory: 'modules'
```

---

## Best Practices

### 1. Pin Workflow Versions

Use specific tags or commit SHAs instead of `@main` for production:

```yaml
uses: islamelkadi/reusable-cicd-workflows/.github/workflows/terraform-lint.yaml@v1.0.0
```

### 2. Configure TFLint

Create a `.tflint.hcl` file in your repository:

```hcl
plugin "aws" {
  enabled = true
  version = "0.35.0"
  source  = "github.com/terraform-linters/tflint-ruleset-aws"
}

rule "terraform_naming_convention" {
  enabled = true
}
```

### 3. Configure Checkov

Create a `.checkov.yml` file to customize security scanning:

```yaml
framework:
  - terraform
skip-check:
  - CKV_AWS_123  # Document why checks are skipped
quiet: false
```

### 4. Use Conventional Commits

Follow the Conventional Commits specification for automatic versioning:

```
<type>(<scope>): <subject>

<body>

<footer>
```

### 5. Protect Main Branch

Configure branch protection rules:
- Require status checks to pass (lint, security)
- Require pull request reviews
- Require linear history

---

## Troubleshooting

### TFLint Fails with "Module not found"

**Problem:** TFLint can't find Terraform modules.

**Solution:** The workflow now runs `terraform init` before `tflint --init` to download modules. Ensure your modules are accessible (public repos or with proper credentials).

### Terraform Validate Fails

**Problem:** Validation fails in subdirectories.

**Solution:** The workflow automatically skips directories that aren't root modules. Check that your Terraform configuration is valid.

### Checkov Reports Too Many Failures

**Problem:** Too many security findings to fix immediately.

**Solution:** Use `.checkov.yml` to skip specific checks with documented justification:

```yaml
skip-check:
  - CKV_AWS_123  # Justification: Dev environment only
```

### Release Not Triggered

**Problem:** No release is created after merging to main.

**Solution:** 
- Ensure commits follow Conventional Commits format
- Check that `GITHUB_TOKEN` has write permissions
- Verify `.releaserc.json` configuration

---

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test with a sample Terraform project
5. Submit a pull request

---

## License

See [LICENSE](LICENSE) file.
