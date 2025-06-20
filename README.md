# Dotnet Repository Publish

![Latest Release](https://img.shields.io/github/v/release/p6m-actions/dotnet-repository-publish?style=flat-square&label=Latest%20Release&color=blue)

## Description

A GitHub Action that packages and publishes .NET libraries to configured NuGet repositories. This action works in conjunction with the `dotnet-repository-login` action to authenticate with and publish to multiple NuGet sources including private repositories and NuGet.org.

### Key Features

- **Multi-project support**: Automatically discovers packable projects or accepts specific project paths
- **Multiple repository publishing**: Publishes to one or more configured NuGet repositories
- **Smart project discovery**: Identifies packable projects based on `IsPackable` or `PackageId` properties
- **Version management**: Supports custom package versioning or uses project-defined versions
- **Symbol package support**: Optionally includes symbol packages (.snupkg) for debugging
- **Dry run capability**: Test publishing workflow without actually publishing packages
- **Duplicate handling**: Configurable behavior for handling existing package versions
- **Comprehensive logging**: Detailed output with timestamps and status indicators

## Usage

This action should be used after building your .NET solution and configuring repository authentication with `dotnet-repository-login`.

```yaml
- name: Login to NuGet repositories
  uses: p6m-actions/dotnet-repository-login@v1
  with:
    credentials: |
      nuget-org=${{ secrets.NUGET_API_KEY }}
      private-repo=${{ secrets.PRIVATE_REPO_TOKEN }}

- name: Publish packages
  uses: p6m-actions/dotnet-repository-publish@v1
  with:
    solution-path: './src'
    configuration: 'Release'
    package-version: '1.2.3'
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `solution-path` | Path to the solution file or directory containing projects to publish | No | `.` |
| `configuration` | Build configuration (Debug or Release) | No | `Release` |
| `projects` | Specific project paths to publish (one per line). If not specified, all packable projects will be published | No | _(empty)_ |
| `package-version` | Version to assign to the packages. If not specified, uses project version | No | _(empty)_ |
| `repositories` | Repository names to publish to (one per line). Must match names configured in dotnet-repository-login. If not specified, publishes to all configured repositories | No | _(empty)_ |
| `skip-duplicate` | Skip publishing if package version already exists | No | `true` |
| `include-symbols` | Include symbol packages (.snupkg) | No | `false` |
| `dry-run` | Perform a dry run without actually publishing | No | `false` |
| `verbosity` | MSBuild verbosity level | No | `minimal` |

## Outputs

| Output | Description |
|--------|-------------|
| `published-packages` | List of successfully published packages with their versions |
| `published-count` | Number of packages successfully published |
| `skipped-packages` | List of packages that were skipped (e.g., due to duplicates) |

## Examples

### Basic Usage

Publish all packable projects to all configured repositories:

```yaml
- name: Publish NuGet packages
  uses: p6m-actions/dotnet-repository-publish@v1
  with:
    solution-path: './src'
    configuration: 'Release'
```

### Publish Specific Projects

Publish only specific projects:

```yaml
- name: Publish specific packages
  uses: p6m-actions/dotnet-repository-publish@v1
  with:
    projects: |
      ./src/MyLibrary.Core/MyLibrary.Core.csproj
      ./src/MyLibrary.Client/MyLibrary.Client.csproj
    package-version: '2.1.0'
```

### Publish to Specific Repositories

Publish to only specific repositories (must be configured via `dotnet-repository-login`):

```yaml
- name: Publish to private repository only
  uses: p6m-actions/dotnet-repository-publish@v1
  with:
    repositories: |
      private-repo
    configuration: 'Release'
```

### Complete Workflow Example

A complete workflow showing build, login, and publish:

```yaml
name: Publish NuGet Packages

on:
  push:
    tags:
      - 'v*'

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Build solution
        uses: p6m-actions/dotnet-build@v1
        with:
          solution-path: './src'
          configuration: 'Release'
          run-tests: true

      - name: Login to repositories
        uses: p6m-actions/dotnet-repository-login@v1
        with:
          repository-urls: |
            private-repo=https://nuget.example.com/v3/index.json
          credentials: |
            nuget-org=${{ secrets.NUGET_API_KEY }}
            private-repo=${{ secrets.PRIVATE_REPO_TOKEN }}

      - name: Publish packages
        uses: p6m-actions/dotnet-repository-publish@v1
        with:
          solution-path: './src'
          configuration: 'Release'
          package-version: ${{ github.ref_name }}
          include-symbols: true
          skip-duplicate: true
        id: publish

      - name: Summary
        run: |
          echo "Published ${{ steps.publish.outputs.published-count }} packages"
          echo "Packages: ${{ steps.publish.outputs.published-packages }}"
```

### Dry Run Example

Test the publishing process without actually publishing:

```yaml
- name: Test publish process
  uses: p6m-actions/dotnet-repository-publish@v1
  with:
    solution-path: './src'
    dry-run: true
    verbosity: 'detailed'
```

### Symbol Packages Example

Publish packages with symbol packages for debugging:

```yaml
- name: Publish with symbols
  uses: p6m-actions/dotnet-repository-publish@v1
  with:
    solution-path: './src'
    include-symbols: true
    configuration: 'Release'
```

## Prerequisites

1. **Repository Authentication**: Use `dotnet-repository-login` action first to configure authentication
2. **Built Solution**: Ensure your solution is built before running this action
3. **Packable Projects**: Projects must have packaging metadata (PackageId, Version, etc.) or be explicitly marked as packable

## Project Configuration

For projects to be automatically discovered, they should have one of the following in their `.csproj` file:

```xml
<PropertyGroup>
  <IsPackable>true</IsPackable>
  <PackageId>MyLibrary.Core</PackageId>
  <Description>My library description</Description>
  <Authors>Your Name</Authors>
</PropertyGroup>
```

Or use a `Directory.Build.props` file in your solution root:

```xml
<Project>
  <PropertyGroup>
    <PackageAuthor>Your Name</PackageAuthor>
    <PackageLicenseExpression>MIT</PackageLicenseExpression>
    <RepositoryUrl>https://github.com/yourusername/yourrepo</RepositoryUrl>
  </PropertyGroup>
</Project>
```