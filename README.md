# .github

## Zonit Organization - Shared GitHub Actions Workflows & Tools

This repository contains reusable GitHub Actions workflows and PowerShell tools for the Zonit organization. These workflows can be called from any repository in the organization to standardize CI/CD processes.

> **🚀 Quick Start:** Use our [setup script](#-quick-setup-script) to configure a new repository in seconds!

---

## 📚 Table of Contents

- [Reusable Workflows](#-reusable-workflows)
  - [Update NuGet Packages](#1-update-nuget-packages)
  - [Publish NuGet Packages](#2-publish-nuget-packages)
  - [Check and Publish NuGet](#3-check-and-publish-nuget)
  - [Update .NET Version](#4-update-net-version)
  - [Update .NET Version with Packages](#5-update-net-version-with-packages-new)
- [PowerShell Tools](#-powershell-tools)
- [Quick Setup Script](#-quick-setup-script)
- [Setup Guide](#-setup-guide-for-new-repositories)
- [Best Practices](#-best-practices)
- [Documentation](#-documentation)

---

## 🔄 Reusable Workflows

### 1. Update NuGet Packages

**File:** `.github/workflows/reusable-update-nugets.yml`

Automatically updates NuGet packages to their latest compatible versions within the same major version.

#### Usage in your repository:

```yaml
name: Update NuGet Packages

on:
  schedule:
    - cron: '0 2 * * 1'  # Every Monday at 2 AM
  workflow_dispatch:      # Manual trigger

jobs:
  update-nugets:
    uses: Zonit/.github/.github/workflows/reusable-update-nugets.yml@main
    permissions:
      contents: write
      pull-requests: write
    with:
      root-dir: '.'  # Optional, defaults to '.'
```

#### Features:
- ✅ Updates packages in `Directory.Packages.props` and conditional `.csproj` references
- ✅ Maintains major version compatibility
- ✅ Creates a pull request with detailed change log
- ✅ Automatically adds `automated-task` label

---

### 2. Publish NuGet Packages

**File:** `.github/workflows/reusable-publish-nuget.yml`

Builds, packs, and publishes NuGet packages to nuget.org.

#### Usage in your repository:

```yaml
name: Publish NuGet Package

on:
  push:
    tags:
      - 'v*'  # Triggers on version tags like v1.0.0

jobs:
  publish:
    uses: Zonit/.github/.github/workflows/reusable-publish-nuget.yml@main
    secrets:
      NUGET_API_KEY: ${{ secrets.NUGET_API_KEY }}
    with:
      source_directory: 'Source'  # Optional, defaults to 'Source'
```

#### Features:
- ✅ Auto-detects required .NET SDK versions (.NET 6, 7, 8, 9, 10)
- ✅ Multi-targeting support
- ✅ Builds and publishes all projects in the source directory
- ✅ Publishes both packages (.nupkg) and symbols (.snupkg)

#### Required Secret:
- `NUGET_API_KEY` - Your nuget.org API key

---

### 3. Check and Publish NuGet

**File:** `.github/workflows/reusable-check-nuget-and-publish.yml`

Compares local package versions with published versions on nuget.org and triggers publish if local is newer.

#### Usage in your repository:

```yaml
name: Check and Publish NuGet

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  check-publish:
    uses: Zonit/.github/.github/workflows/reusable-check-nuget-and-publish.yml@main
    secrets:
      GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

#### Features:
- ✅ Scans all `.csproj` files in `Source` directory
- ✅ Compares versions with nuget.org
- ✅ Triggers publish workflow only when needed

---

### 4. Update .NET Version

**File:** `.github/workflows/reusable-update-dotnet-version.yml`

Updates .NET framework version across all project files in the repository.

#### Usage in your repository:

```yaml
name: Update .NET Version

on:
  workflow_dispatch:
    inputs:
      from-version:
        description: 'Current .NET version'
        required: true
        default: '9.0'
      to-version:
        description: 'Target .NET version'
        required: true
        default: '10.0'

jobs:
  update-dotnet:
    uses: Zonit/.github/.github/workflows/reusable-update-dotnet-version.yml@main
    permissions:
      contents: write
      pull-requests: write
    with:
      from-version: ${{ github.event.inputs.from-version }}
      to-version: ${{ github.event.inputs.to-version }}
      create-pr: true  # Optional, defaults to true
```

#### Features:
- ✅ Updates `<TargetFramework>` and `<TargetFrameworks>`
- ✅ Updates conditional package references
- ✅ Updates `global.json` SDK version
- ✅ Creates detailed PR with all changes
- ✅ Handles multi-targeting scenarios

---

### 5. Update .NET Version with Packages (NEW! 🎯)

**File:** `.github/workflows/reusable-update-dotnet-with-packages.yml`

**NEW SYSTEM**: Intelligently updates .NET framework version(s) AND automatically updates all NuGet packages to latest compatible versions using conditional package management.

#### Usage in your repository:

```yaml
name: Update .NET Framework

on:
  workflow_dispatch:
    inputs:
      target-frameworks:
        description: 'Target frameworks (e.g., "net9.0" or "net8.0,net9.0" for multi-targeting)'
        required: true
        default: 'net9.0'

jobs:
  update:
    uses: Zonit/.github/.github/workflows/reusable-update-dotnet-with-packages.yml@main
    permissions:
      contents: write
      pull-requests: write
    with:
      target-frameworks: ${{ github.event.inputs.target-frameworks }}
```

#### Features:
- ✅ **Smart .NET Detection** - Automatically detects if repository contains .NET projects
- ✅ **Multi-Targeting Support** - Single framework or multiple (e.g., net8.0,net9.0)
- ✅ **Conditional Package Management** - Creates separate ItemGroups per framework version
- ✅ **Latest Compatible Versions** - Fetches the latest NuGet version compatible with each framework
- ✅ **Central Package Management** - Uses Directory.Packages.props with conditions
- ✅ **Automatic PR Creation** - Creates detailed PR for manual verification
- ✅ **Modular Design** - Reusable PowerShell modules for future scheduled updates

#### Example: Single Framework Update

Input: `net9.0`

Creates:
```xml
<!-- Directory.Packages.props -->
<ItemGroup Condition="'$(TargetFramework)' == 'net9.0'">
  <PackageVersion Include="Microsoft.Extensions.Hosting" Version="9.0.1" />
</ItemGroup>
```

#### Example: Multi-Targeting

Input: `net8.0,net9.0`

Creates:
```xml
<!-- Directory.Packages.props -->
<ItemGroup Condition="'$(TargetFramework)' == 'net8.0'">
  <PackageVersion Include="Microsoft.Extensions.Hosting" Version="8.0.1" />
</ItemGroup>

<ItemGroup Condition="'$(TargetFramework)' == 'net9.0'">
  <PackageVersion Include="Microsoft.Extensions.Hosting" Version="9.0.1" />
</ItemGroup>
```

📖 **For complete documentation, see:** [.NET Update System Guide](docs/DOTNET_UPDATE_SYSTEM.md)

---

## 🛠️ PowerShell Tools

These tools are downloaded and executed by the workflows, but can also be run locally.

### update-dotnet-simple.ps1

**NEW SYSTEM**: Updates .NET framework version(s), ensures Directory.Packages.props exists, and updates all package versions to latest.

```powershell
# Download and run locally
curl -fsSL https://raw.githubusercontent.com/Zonit/.github/refs/heads/master/tools/update-dotnet-simple.ps1 -o update-dotnet.ps1

# Single framework
pwsh ./update-dotnet.ps1 -TargetFrameworks @("net9.0")

# Multi-targeting
pwsh ./update-dotnet.ps1 -TargetFrameworks @("net8.0", "net9.0")
```

**Features:**
- ✅ Updates target frameworks in .csproj files
- ✅ Creates Directory.Packages.props if missing (preserves existing)
- ✅ Adds missing packages from projects to Directory.Packages.props
- ✅ **NEW**: Automatically updates all package versions to latest compatible versions
- ✅ Removes package versions from individual projects (Central Package Management)
- ✅ Framework-aware version selection (e.g., 9.x.x for net9.0, 8.x.x for net8.0)
- ✅ **SAFE**: Never overwrites existing Directory.Packages.props structure

**How it works (8 steps):**
1. Detect .NET projects
2. Scan for project files
3. Update target frameworks in .csproj
4. Remove package versions from projects (enables CPM)
5. Collect all package references
6. Fetch latest versions for each framework
7. **Create or verify Directory.Packages.props** (no overwriting!)
8. **Add missing packages + update all versions to latest**

### update-package-versions.ps1

**NEW**: Standalone tool to update package versions in existing Directory.Packages.props files without changing frameworks.

```powershell
# Download and run locally
curl -fsSL https://raw.githubusercontent.com/Zonit/.github/refs/heads/master/tools/update-package-versions.ps1 -o update-packages.ps1

# Update packages
pwsh ./update-packages.ps1

# Dry run (preview changes without applying)
pwsh ./update-packages.ps1 -WhatIf
```

**Use Cases:**
- 🔄 Scheduled package updates without framework changes
- 🔍 Preview what would be updated (WhatIf mode)
- 🎯 Quick security updates
- 📦 Maintenance updates for existing projects

**Features:**
- ✅ Updates all packages in Directory.Packages.props
- ✅ Framework-aware: updates net8.0 packages to 8.x.x, net9.0 to 9.x.x, etc.
- ✅ Handles conditional and unconditional ItemGroups
- ✅ Rate-limited NuGet API queries
- ✅ Detailed progress and summary reporting

---

## ⚡ Quick Setup

To use these workflows in your repository, simply reference them in your workflow files. See [Example Workflows](docs/EXAMPLE_WORKFLOWS.md) for ready-to-use templates.

### Example: Add Maintenance Workflow

Create `.github/workflows/maintenance.yml` in your repository:

```yaml
name: Maintenance

on:
  schedule:
    - cron: '0 2 * * 1'  # Weekly updates
  workflow_dispatch:

jobs:
  update-packages:
    uses: Zonit/.github/.github/workflows/reusable-update-nugets.yml@master
    permissions:
      contents: write
      pull-requests: write
  update-dotnet:
    uses: Zonit/.github/.github/workflows/reusable-update-dotnet-version.yml@master
    permissions:
      contents: write
      pull-requests: write
    with:
      from-version: '9.0'
      to-version: '10.0'
    if: github.event_name == 'workflow_dispatch'  # Only on manual trigger
```

For organization-wide updates, see [Organization-Wide Update Guide](docs/ORG_WIDE_UPDATE_GUIDE.md).

---

## 🚀 Setup Guide for New Repositories

### Quick Start

1. **Create workflow file** in your repository at `.github/workflows/maintenance.yml`:

```yaml
name: Maintenance

on:
  schedule:
    - cron: '0 2 * * 1'  # Weekly NuGet updates
  workflow_dispatch:

jobs:
  update-nugets:
    uses: Zonit/.github/.github/workflows/reusable-update-nugets.yml@main
    permissions:
      contents: write
      pull-requests: write

  update-dotnet:
    uses: Zonit/.github/.github/workflows/reusable-update-dotnet-version.yml@main
    permissions:
      contents: write
      pull-requests: write
    with:
      from-version: '9.0'
      to-version: '10.0'
    if: github.event_name == 'workflow_dispatch'  # Only on manual trigger
```

2. **Add secrets** to your repository:
   - `NUGET_API_KEY` (if publishing packages)

3. **Test the workflow**:
   - Go to Actions tab in your repository
   - Find "Maintenance" workflow
   - Click "Run workflow"

---

## 📖 Documentation

### Main Documentation
- **[README.md](README.md)** - This file, overview of all workflows and tools
- **[Example Workflows](docs/EXAMPLE_WORKFLOWS.md)** - Copy-paste ready workflow examples
- **[Organization-Wide Update Guide](docs/ORG_WIDE_UPDATE_GUIDE.md)** - Complete guide for updating all repos at once
- **[Package Update System](docs/PACKAGE_UPDATE_SYSTEM.md)** - **NEW**: Detailed guide for automated package version updates

### Quick Links
- [How to update NuGet packages](#1-update-nuget-packages)
- [How to publish NuGet packages](#2-publish-nuget-packages)
- [How to update .NET version](#4-update-net-version)
- [How to update package versions only](docs/PACKAGE_UPDATE_SYSTEM.md)
- [Example workflows for your repository](docs/EXAMPLE_WORKFLOWS.md)
- [Quick setup for new repositories](#-quick-setup-script)

### External Resources
- [GitHub Actions Reusable Workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
- [NuGet Package Publishing](https://docs.microsoft.com/nuget/nuget-org/publish-a-package)
- [.NET CLI Reference](https://docs.microsoft.com/dotnet/core/tools/)

---

## 📝 License

This repository is maintained by Zonit organization.

---

## 🆘 Support

For issues or questions:
- Open an issue in this repository
- Contact the DevOps team

---

## 📊 Workflow Status

Track the status of automated workflows across repositories:

| Workflow | Purpose | Frequency |
|----------|---------|-----------|
| Update NuGet | Keep packages up to date | Weekly |
| Publish NuGet | Release packages | On version tags |
| Check & Publish | Auto-publish new versions | On main push |
| Update .NET | Upgrade framework versions | Manual/On-demand |

---

**Last Updated:** 2024
**Maintained by:** Zonit DevOps Team
