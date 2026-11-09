# Repository Guide

This repository contains reusable Azure DevOps and GitHub Actions pipeline templates for .NET projects. See [README.md](README.md) for the public template catalog, inputs, and usage examples.

## GitHub Actions

- Workflows in `.github/workflows` are reusable workflows, generally exposed through `workflow_call`.
- `build-n-pack.yml` restores, builds, tests, packs projects, and uploads NuGet packages and coverage as artifacts.
- `build-n-pack-multi-stage.yml` combines the build and publishing workflows. Pull requests do not publish; package publishing follows successful package-relevant pushes or manual runs. CodeQL can also trigger a build.
- `push-nuget.yml` downloads the package artifact and can publish to Azure Artifacts, MyGet, and Codeberg.
- Publishing credentials are caller-provided secrets: `AZURE_ARTIFACTS_PAT`, `MYGET_API_KEY`, and `CODEBERG_API_KEY`. Azure Artifacts publishing is skipped without a source URL and fails if a configured source has no PAT. MyGet and Codeberg skip when their token is missing; their steps use `continue-on-error`.
- Keep reusable workflow inputs, secrets, artifact names, and calling-workflow wiring consistent when changing these workflows.

## Azure DevOps

Root-level YAML templates provide build, test, pack, ZIP, publishing, and agent-cleanup workflows. Use the README for their parameters and intended use.
