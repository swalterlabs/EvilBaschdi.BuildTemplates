# EvilBaschdi.BuildTemplates

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg?style=for-the-badge&)](LICENSE)
![Commit Activity](https://img.shields.io/github/commit-activity/m/evilbaschdi/evilbaschdi.buildtemplates?style=for-the-badge&)
![Last Commit](https://img.shields.io/github/last-commit/evilbaschdi/evilbaschdi.buildtemplates?style=for-the-badge&)

## Overview

This repository contains reusable pipeline templates for .NET projects. The templates support building, testing, creating NuGet packages and ZIP archives, and publishing the resulting artifacts.

## Azure DevOps Pipelines

### `buildNPackMultiStage.yml`

Runs the build and publishing process in two stages:

1. **Build** – uses `buildNPack.yml` to restore, build, and test the projects and create NuGet packages.
2. **Deploy** – uses `pushNuget.yml` to publish the packages. This stage runs only after a successful build.

Parameters:

| Parameter                 | Description                                      |
| ------------------------- | ------------------------------------------------ |
| `projects`                | List of project names to build                   |
| `framework`               | Target framework: `net10.0` or `net10.0-windows` |
| `projectsParentDirectory` | Parent directory of the projects                 |
| `testsParentDirectory`    | Parent directory of the test projects            |

### `buildNPack.yml`

Restores, builds, and tests multiple .NET projects. Release packages are created for the `main` branch. On other branches, the branch name is used as the package version suffix. Packages and test results are published as build artifacts.

### `buildNZip.yml`

Restores, builds, and tests a .NET project and publishes it as a ZIP archive. Test results and code coverage reports are also published.

Parameters:

| Parameter     | Description                                      |
| ------------- | ------------------------------------------------ |
| `projectName` | Project name including its project directory     |
| `framework`   | Target framework: `net10.0` or `net10.0-windows` |

### `pushNuget.yml`

Downloads the build artifact and publishes the included NuGet packages to:

- Azure Artifacts
- MyGet
- Codeberg

Credentials are provided through the Azure DevOps variable group `ApiKeys`. Each publishing step is configured to continue if it fails.

### `azureDevOps.yml`

A standalone build and pack template for a single project. It restores, builds, and tests the project, creates a release or preview package depending on the branch, and publishes the NuGet artifact together with a code coverage report.

Parameters:

| Parameter     | Description                                  |
| ------------- | -------------------------------------------- |
| `projectName` | Project name including its project directory |

### `DevOpsAgentCleanUp.yml`

Cleans up a self-hosted Azure DevOps agent. Work directories, local .NET SDK files, and NuGet packages older than 14 days are removed.

## GitHub Actions

The workflows in `.github/workflows` are designed as reusable workflows using `workflow_call`.

### `build-n-pack.yml`

Restores, builds, and tests multiple .NET projects and creates NuGet packages. Release versions are created on `main`; on other branches, the branch name is used as the version suffix. Packages and code coverage files are uploaded as GitHub artifacts.

Important inputs:

| Input                       | Description                                     |
| --------------------------- | ----------------------------------------------- |
| `projects`                  | JSON array of project names                     |
| `framework`                  | Target framework used for testing               |
| `projects-parent-directory` | Parent directory of the projects                |
| `tests-parent-directory`    | Parent directory of the test projects           |
| `artifact-name`             | Package artifact name, default: `Nuget-Package` |
| `enable-codeql`             | Run CodeQL analysis during the build             |

### `build-n-zip.yml`

Builds and tests a single .NET project, creates a ZIP archive, and uploads the archive, test results, and code coverage as artifacts.

### `push-nuget.yml`

Downloads NuGet packages from a GitHub artifact and optionally publishes them to Azure Artifacts, MyGet, and Codeberg. Credentials are passed as GitHub secrets:

- `AZURE_ARTIFACTS_PAT`
- `MYGET_API_KEY`
- `CODEBERG_API_KEY`

When `azure-artifacts-source` is set, `AZURE_ARTIFACTS_PAT` must be available to the calling workflow and have Packaging read/write access to the feed. Azure Artifacts publishing fails explicitly if the PAT is missing or the push fails.

### `build-n-pack-multi-stage.yml`

Combines `build-n-pack.yml` and `push-nuget.yml` into a complete workflow:

```text
build
  -> Create NuGet packages and coverage artifacts
publish
  -> Publish NuGet packages
```

The publish job runs only after a successful build and only for pushes or manual runs with package-relevant changes. Pull requests never publish packages.

Set `enable-codeql` to `true` to run CodeQL analysis as part of the same build. This avoids repeating the .NET compilation for CodeQL. CodeQL-only runs still build and analyze the project without publishing packages.

The multi-stage workflow accepts these additional inputs:

| Input                    | Description                           |
| ------------------------ | ------------------------------------- |
| `azure-artifacts-source` | Optional Azure Artifacts NuGet v3 feed URL |
| `myget-source`           | Optional MyGet NuGet v3 feed URL |
| `enable-codeql`          | Run CodeQL analysis during the build |

### `devops-agent-cleanup.yml`

Cleans up a self-hosted runner. Azure DevOps work directories, .NET SDK files, and stale NuGet packages are removed.

## Usage example

A calling GitHub workflow can use the multi-stage template as follows:

```yaml
jobs:
  build-and-publish:
    uses: swalterlabs/EvilBaschdi.BuildTemplates/.github/workflows/build-n-pack-multi-stage.yml@main
    with:
      projects: '["MyProject"]'
      framework: net10.0
      projects-parent-directory: src/
      tests-parent-directory: tests/
      azure-artifacts-source: https://pkgs.dev.azure.com/your-organization/_packaging/your-feed/nuget/v3/index.json
      myget-source: https://www.myget.org/F/your-feed/api/v3/index.json
      enable-codeql: true
    secrets: inherit
```
