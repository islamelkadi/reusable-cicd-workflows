# Reusable CI/CD Workflows

Collection of reusable GitHub Actions workflows for Terraform projects.

## Workflows

### Checkov Security Scan
Security scanning for Terraform code using Checkov.

**Usage:**
```yaml
jobs:
  security:
    uses: your-org/reusable-cicd-workflows/.github/workflows/checkov-scan.yaml@main
    with:
      working_directory: '.'  # optional
      config_file: '.checkov.yml'  # optional
```

### Terraform Lint
Runs `terraform fmt`, `tflint`, and `terraform validate`.

**Usage:**
```yaml
jobs:
  lint:
    uses: your-org/reusable-cicd-workflows/.github/workflows/terraform-lint.yaml@main
    with:
      terraform_version: '1.14.3'  # optional
      working_directory: '.'  # optional
      tflint_version: 'latest'  # optional
```

### Terraform Release
Automated semantic versioning and releases using semantic-release.

**Usage:**
```yaml
jobs:
  release:
    uses: your-org/reusable-cicd-workflows/.github/workflows/terraform-release.yaml@main
    with:
      node_version: 'lts/*'  # optional
    secrets:
      token: ${{ secrets.GITHUB_TOKEN }}
```

## License

See [LICENSE](LICENSE) file.
