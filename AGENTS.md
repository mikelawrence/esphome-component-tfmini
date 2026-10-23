# AI Collaboration

First read the [ESPHome AI Collaboration Guide](https://github.com/esphome/esphome/blob/dev/AGENTS.md), then follow these repository-specific instructions.

## Pull Requests

### Description

```markdown
Use a format similar to this...

# What does this implement/fix?

Describe the changes.

## Types of changes

- [] Code quality improvements to existing code
- [] Bugfix
- [] New feature
- [] Breaking change

## Tests

- [] Linting/Validation
- [] Local Builds
```

## Local Builds and testing

Run commands from the repository root in PowerShell. Docker Desktop must be installed and running; the build and validation scripts run ESPHome in Docker Compose containers.

### Validate and auto-fix

Run:

```powershell
.\.devcontainer\validate.ps1
```

This command applies supported automatic fixes, then runs strict YAML linting, C/C++ formatting checks, Ruff formatting and lint, flake8, pylint, pyupgrade, and component convention checks. Fixes are applied before the checks, so a separate fix or validation run is unnecessary. Docker Desktop must be installed and running.

### Build test configurations

In VS Code, use **Terminal > Run Build Task** and select **ML ESPHome: Build ESPHome test configs**. This is the default Build task and runs component validation first; compilation starts only if validation succeeds. The build task compiles every `.yaml` and `.yml` file under `test/` with ESPHome `stable`, `beta`, and `dev` images.

To run compilation by itself, run:

```powershell
.\.devcontainer\build.ps1
```

The build script temporarily rewrites matching GitHub external-component sources to use this checkout's `components/` directory, leaving the original test files unchanged. It reads the component's GitHub owner and repository from the `origin` remote; if there is no GitHub `origin`, provide it explicitly with `-ComponentSource owner/repository`. Pass `-Versions @("dev")` to select ESPHome tags, or `-Image ghcr.io/esphome/esphome:<tag>` to use one custom image. The VS Code task **ML ESPHome: Validate ESPHome component** runs just the auto-fix and validation step.

There is no separate unit-test runner configured in this repository. Treat compiling the test configurations as the integration/build test. If auto-fix, validation, or compilation fails, report the specific failure rather than treating it as passed.
