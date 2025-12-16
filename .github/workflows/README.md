# CI/CD Pipeline Documentation

This repository uses GitHub Actions for automated CI/CD processes including linting, testing, packaging, and deployment of Helm charts to GitHub Pages.

## Workflows

### 1. CI/CD Pipeline (`.github/workflows/ci-cd.yml`)

**Triggers:**
- Push to `main`/`master` branches
- Pull requests to `main`/`master` branches
- Published releases

**Jobs:**

#### Lint Job
- Installs Helm and helm-docs
- Validates and regenerates documentation
- Lints all Helm charts using `helm lint`
- Validates rendered Kubernetes templates with kubeval

#### Package Job
- Packages all Helm charts into `.tgz` files
- Uploads packages as GitHub artifacts

#### Deploy Job
- Deploys packaged charts to GitHub Pages
- Generates `index.yaml` for Helm repository
- Only runs on main branch or releases

#### Release Job
- Attaches packaged charts to GitHub releases
- Only runs on release events

### 2. Chart Release (`.github/workflows/release.yml`)

**Triggers:**
- Tag pushes (e.g., `linkding-*`, `postgres-*`, `v*`)

**Jobs:**

#### Test Job
- Tests charts against multiple Kubernetes versions (1.25-1.28)
- Validates with different value combinations
- Uses kubeconform for strict validation

#### Security Scan Job
- Scans container images for vulnerabilities using Trivy
- Reports HIGH and CRITICAL severity issues
- Skips local/localhost images

#### Release Job
- Uses Helm Chart Releaser to publish charts
- Updates chart repository on GitHub Pages

### 3. Dependency Update (`.github/workflows/dependency-update.yml`)

**Triggers:**
- Weekly schedule (Mondays at 2 AM UTC)
- Manual workflow dispatch

**Jobs:**

#### Update Dependencies Job
- Checks for Helm chart dependency updates
- Identifies charts using 'latest' tags
- Creates pull requests with updates

## Repository Structure for CI/CD

```
.github/
├── workflows/
│   ├── ci-cd.yml              # Main CI/CD pipeline
│   ├── release.yml            # Chart releases and testing
│   └── dependency-update.yml  # Automated dependency updates
```

## GitHub Pages Deployment

The charts are automatically deployed to GitHub Pages as a Helm repository:

- **URL**: `https://{username}.github.io/{repository-name}`
- **Index**: `index.yaml` is automatically generated
- **Charts**: All `.tgz` packages are published

## Usage

### Adding New Charts

1. Create chart in `charts/{chart-name}/`
2. Ensure `Chart.yaml` has proper metadata
3. The pipeline will automatically detect and include the new chart

### Manual Chart Release

1. Tag the release: `git tag linkding-1.0.0`
2. Push the tag: `git push origin linkding-1.0.0`
3. The release workflow will automatically:
   - Test the chart
   - Package it
   - Create a GitHub release
   - Deploy to GitHub Pages

### Local Testing

Before pushing, you can run the same checks locally:

```bash
# Install dependencies
helm plugin install https://github.com/norwoodj/helm-docs
helm plugin install https://github.com/quintush/helm-unittest

# Lint charts
for chart in charts/*/; do
  helm lint "$chart"
done

# Test templates
helm template linkding ./charts/linkding | kubeval
helm template postgres ./charts/postgres | kubeval

# Generate docs
helm-docs
```

## Configuration

### Required Secrets

- `GITHUB_TOKEN`: Automatically provided by GitHub Actions

### Environment Variables

- `HELM_VERSION`: Controls the Helm version used (default: "3.14.0")

### Permissions

The workflows require the following permissions:
- `contents: read` - For checkout and release operations
- `pages: write` - For GitHub Pages deployment
- `id-token: write` - For GitHub Pages deployment

## Troubleshooting

### Common Issues

1. **Documentation drift**: Run `helm-docs` locally and commit changes
2. **Template validation failures**: Check Kubernetes API compatibility
3. **Image scan failures**: Update to more secure image versions
4. **GitHub Pages deployment**: Ensure repository has Pages enabled

### Debugging

- Check the Actions tab in GitHub for detailed logs
- Use `workflow_dispatch` to manually trigger workflows
- Local testing helps identify issues before pushing