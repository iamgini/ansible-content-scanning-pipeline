# Ansible Pipeline Templates

Reusable GitHub Actions workflows for Ansible content scanning, security checks, and collection publishing.

## Overview

This repository provides pre-built CI/CD pipeline templates that can be integrated into your Ansible collection or playbook repositories. These workflows help automate quality checks, security scans, and publishing processes.

## Available Workflows

### 1. Scan Pipeline (`scan-pipeline.yml`)

Runs security and quality checks on your Ansible content.

**Features:**
- **Secret Detection**: Scans for exposed credentials and sensitive data
- **Linting**: Validates Ansible syntax and best practices
- **SAST**: Static Application Security Testing
- **Tagging**: Automated version tagging
- **Signing**: Artifact signing for verified releases

**All checks run in parallel** after initial setup for faster execution.

### 2. Publish Pipeline (`publish-pipeline.yml`)

Builds and publishes Ansible collections to distribution platforms.

**Features:**
- Automated collection building from `galaxy.yml`
- Publish to **Ansible Galaxy** (public)
- Publish to **Private Automation Hub** (enterprise)
- Artifact caching and reuse

## Usage

### Basic Integration

Add these workflows to your Ansible repository by creating a workflow file in `.github/workflows/`:

**Example: `ci.yml`**
```yaml
name: CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:

jobs:
  scan:
    uses: gmadappa/ansible-pipeline-templates/.github/workflows/scan-pipeline.yml@main
```

**Example: `publish.yml`**
```yaml
name: Publish Collection

on:
  push:
    tags:
      - 'v*'

jobs:
  publish:
    uses: gmadappa/ansible-pipeline-templates/.github/workflows/publish-pipeline.yml@main
    with:
      publish_to_galaxy: true
      publish_to_pah: false
    secrets:
      ANSIBLE_GALAXY_TOKEN: ${{ secrets.ANSIBLE_GALAXY_TOKEN }}
```

### Configuration Options

#### Scan Pipeline Inputs

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `run_secret_detection` | boolean | `true` | Enable secret detection |
| `run_lint` | boolean | `true` | Enable linting checks |
| `run_sast` | boolean | `true` | Enable SAST scanning |
| `run_tagging` | boolean | `true` | Enable automated tagging |
| `run_sign` | boolean | `true` | Enable artifact signing |

#### Publish Pipeline Inputs

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `publish_to_galaxy` | boolean | `true` | Publish to Ansible Galaxy |
| `publish_to_pah` | boolean | `false` | Publish to Private Automation Hub |
| `pah_server` | string | `''` | Private Automation Hub server URL |

#### Publish Pipeline Secrets

| Secret | Required | Description |
|--------|----------|-------------|
| `ANSIBLE_GALAXY_TOKEN` | Conditional | API token for Ansible Galaxy (required if publishing to Galaxy) |
| `PAH_TOKEN` | Conditional | API token for Private Automation Hub (required if publishing to PAH) |

### Advanced Examples

**Scan only on pull requests, skip signing:**
```yaml
jobs:
  scan:
    uses: gmadappa/ansible-pipeline-templates/.github/workflows/scan-pipeline.yml@main
    with:
      run_sign: false
```

**Publish to both Galaxy and Private Hub:**
```yaml
jobs:
  publish:
    uses: gmadappa/ansible-pipeline-templates/.github/workflows/publish-pipeline.yml@main
    with:
      publish_to_galaxy: true
      publish_to_pah: true
      pah_server: https://automation-hub.example.com/api/galaxy/content/published/
    secrets:
      ANSIBLE_GALAXY_TOKEN: ${{ secrets.ANSIBLE_GALAXY_TOKEN }}
      PAH_TOKEN: ${{ secrets.PAH_TOKEN }}
```

**Combined workflow:**
```yaml
name: Full CI/CD Pipeline

on:
  push:
    branches: [main]
    tags: ['v*']
  pull_request:

jobs:
  scan:
    uses: gmadappa/ansible-pipeline-templates/.github/workflows/scan-pipeline.yml@main
  
  publish:
    needs: scan
    if: startsWith(github.ref, 'refs/tags/v')
    uses: gmadappa/ansible-pipeline-templates/.github/workflows/publish-pipeline.yml@main
    with:
      publish_to_galaxy: true
    secrets:
      ANSIBLE_GALAXY_TOKEN: ${{ secrets.ANSIBLE_GALAXY_TOKEN }}
```

## Requirements

### For Scan Pipeline
- Repository must have Ansible content (playbooks, roles, or collections)

### For Publish Pipeline
- Repository must contain a valid `galaxy.yml` file at the root
- Required secrets configured in repository settings
- For Private Automation Hub: server URL and API token

## Contributing

Improvements and additional pipeline templates are welcome! Please submit pull requests with:
- Clear description of the change
- Example usage in the PR description
- Updated README if adding new features

## License

This project is open source. Use these templates freely in your Ansible projects.

## Related Resources

- [Ansible Galaxy](https://galaxy.ansible.com/)
- [Ansible Documentation](https://docs.ansible.com/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
