---
summary: "GitHub Actions CI pipeline: how to run workflows, understand test matrix, and debug failures"
read_when:
  - Setting up CI for the repository
  - Running GitHub Actions manually
  - Debugging CI failures
---

# Continuous Integration (CI)

OpenClaw uses GitHub Actions for continuous integration testing across multiple platforms and runtimes.

## Quick Start

### Automatic Triggers

The CI pipeline automatically runs on:
- **Push to main branch**: Full test suite across all platforms
- **Push to copilot/** branches**: Full test suite for Copilot workspace branches
- **Pull Requests**: Complete test matrix including macOS app builds

### Manual Triggers

You can manually trigger CI workflows from the GitHub UI:

1. Go to **Actions** tab in the repository
2. Select the workflow (e.g., "CI")
3. Click **Run workflow** button
4. Select the branch to run on
5. Click **Run workflow**

## Workflow Overview

### Main CI Workflow (`.github/workflows/ci.yml`)

The primary CI workflow includes multiple job types:

#### 1. Installation Check (`install-check`)
- Validates dependency installation
- Tests with frozen lockfile
- Runs on: Ubuntu (Blacksmith runner)

#### 2. Checks (`checks`)
- **Matrix dimensions**:
  - Runtime: Node.js 22.x, Bun (latest)
  - Tasks: lint, test, build, protocol check, format check
- **Commands**:
  - `pnpm lint` - Code linting with oxlint
  - `pnpm test` - Unit and integration tests
  - `pnpm build` - TypeScript compilation
  - `pnpm protocol:check` - Protocol schema validation
  - `pnpm format` - Code formatting check with oxfmt
- Runs on: Ubuntu (Blacksmith runner)

#### 3. Windows Checks (`checks-windows`)
- **Matrix dimensions**:
  - Runtime: Node.js 22.x only
  - Tasks: lint, test, build, protocol check
- **Environment**:
  - `NODE_OPTIONS: --max-old-space-size=4096`
  - `CLAWDBOT_TEST_WORKERS: 1`
- Runs on: Windows Server 2025 (Blacksmith runner)

#### 4. macOS Checks (`checks-macos`)
- **Tasks**: test suite only
- **Trigger**: Pull requests only
- Runs on: macOS latest (GitHub-hosted)

#### 5. macOS App (`macos-app`)
- **Tasks**:
  - Swift lint (swiftlint)
  - Swift format check (swiftformat)
  - Swift build (release configuration)
  - Swift tests with coverage
- **Toolchain**: Xcode 26.1
- **Trigger**: Pull requests only
- Runs on: macOS latest (GitHub-hosted)

#### 6. iOS (`ios`)
- **Status**: Currently disabled (`if: false`)
- **Note**: iOS CI is skipped for now; enable by removing the `if: false` condition

#### 7. Android (`android`)
- **Tasks**:
  - Unit tests: `./gradlew :app:testDebugUnitTest`
  - Debug build: `./gradlew :app:assembleDebug`
- **Requirements**:
  - Java 21 (Temurin)
  - Android SDK (platform-tools, android-36, build-tools 36.0.0)
  - Gradle 8.11.1
- Runs on: Ubuntu (Blacksmith runner)

#### 8. Secrets Scanning (`secrets`)
- **Tool**: detect-secrets 1.5.0
- **Config**: `.secrets.baseline`
- **Baseline**: Pre-approved secrets are tracked in baseline file
- Runs on: Ubuntu (Blacksmith runner)

### Workflow Sanity (`.github/workflows/workflow-sanity.yml`)

- **Purpose**: Validates workflow files don't contain tabs
- **Trigger**: All pushes and pull requests
- **Can be triggered manually** via workflow_dispatch
- Runs on: Ubuntu (GitHub-hosted)

## Understanding Test Failures

### Common Failure Types

1. **Submodule checkout failures**
   - All jobs retry submodule init up to 5 times
   - Check network connectivity if failing consistently

2. **pnpm installation failures**
   - Jobs retry corepack prepare up to 3 times
   - Frozen lockfile enforced (`--frozen-lockfile`)

3. **Test failures**
   - Check the specific runtime (Node vs Bun)
   - Windows has limited test workers (`CLAWDBOT_TEST_WORKERS: 1`)
   - macOS tests have higher memory limit (`NODE_OPTIONS: --max-old-space-size=4096`)

4. **Build failures**
   - Swift builds retry up to 3 times on macOS
   - Check Xcode version compatibility (requires Xcode 26.1)

5. **Secret scanning failures**
   - New secrets detected in code
   - Update `.secrets.baseline` if secrets are false positives
   - See: `docs/gateway/security.md#secret-scanning-detect-secrets`

## Local Development

Before pushing, run the same checks locally:

```bash
# Full gate (recommended before push)
pnpm lint && pnpm build && pnpm test

# Individual checks
pnpm lint              # Code linting
pnpm format            # Format checking
pnpm build             # TypeScript build
pnpm test              # Unit tests
pnpm test:coverage     # With coverage
pnpm protocol:check    # Protocol schema validation

# Swift checks (macOS only)
swiftlint --config .swiftlint.yml
swiftformat --lint apps/macos/Sources --config .swiftformat
swift build --package-path apps/macos --configuration release
swift test --package-path apps/macos

# Android checks
cd apps/android
./gradlew :app:testDebugUnitTest
./gradlew :app:assembleDebug
```

## Runner Configuration

### Blacksmith Runners
- Custom high-performance runners
- Used for: Ubuntu and Windows jobs
- Configuration: `blacksmith-4vcpu-ubuntu-2404`, `blacksmith-4vcpu-windows-2025`

### GitHub-Hosted Runners
- Standard GitHub runners
- Used for: macOS jobs, workflow sanity
- Configuration: `macos-latest`, `ubuntu-latest`

## Branch Filtering

CI runs on:
- `main` branch (all commits)
- `copilot/**` branches (Copilot workspace branches)
- All pull requests (regardless of target branch)

To modify branch filters, edit the `on.push.branches` section in `.github/workflows/ci.yml`.

## Advanced: Adding New Checks

To add a new check to the CI matrix:

1. Edit `.github/workflows/ci.yml`
2. Add to the `matrix.include` array under the appropriate job
3. Example:
   ```yaml
   - runtime: node
     task: new-check
     command: pnpm new-check
   ```
4. Test locally first
5. Validate with actionlint: `actionlint .github/workflows/ci.yml`

## Troubleshooting

### Workflow not triggering
- Check branch filters in `on.push.branches`
- Ensure workflow file has no syntax errors
- Check repository Actions settings are enabled

### Manual trigger not available
- Ensure workflow has `workflow_dispatch:` trigger
- Check you have repository permissions

### Tests timing out
- Check `NODE_OPTIONS: --max-old-space-size=4096` is set
- Review test worker configuration
- Consider splitting long-running tests

### Secrets baseline conflicts
- Run `detect-secrets scan --baseline .secrets.baseline` locally
- Review new secrets and add to baseline if approved
- Commit updated baseline file

## Related Documentation

- [Testing Guide](../testing.md) - Detailed testing documentation
- [Contributing Guide](../../CONTRIBUTING.md) - How to contribute
- [Release Process](./RELEASING.md) - Release workflow

## Workflow Validation

All workflow files are validated for:
- No tab characters (enforced by workflow-sanity.yml)
- Valid YAML syntax
- Valid GitHub Actions syntax

Use `actionlint` locally to validate workflows before pushing:

```bash
# Install actionlint
curl -fsSL https://raw.githubusercontent.com/rhysd/actionlint/main/scripts/download-actionlint.bash | bash -s -- latest /usr/local/bin

# Validate workflow
actionlint .github/workflows/ci.yml
```
