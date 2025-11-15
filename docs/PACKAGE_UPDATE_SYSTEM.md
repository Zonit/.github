# Package Version Update System

## Overview

The Package Version Update system provides automated tools to keep your NuGet packages up-to-date with the latest compatible versions. The system is framework-aware and respects Central Package Management practices.

## Architecture

### Modules

#### 1. PackageVersionUpdater.psm1

Core module responsible for updating package versions in `Directory.Packages.props` files.

**Functions:**
- `Get-LatestMajorNuGetVersion` - Queries NuGet.org for latest version matching framework major version
- `Update-PackageVersionsInProps` - Updates all packages in a Directory.Packages.props file
- `Update-PackageVersionsInProject` - Updates conditional packages in .csproj files
- `Update-AllPackageVersions` - Bulk updates across entire repository

**Smart Version Selection:**
- For `net8.0` packages: selects latest `8.x.x` version
- For `net9.0` packages: selects latest `9.x.x` version
- For `net10.0` packages: selects latest `10.x.x` version
- For framework-agnostic packages: selects latest stable version

### Scripts

#### update-package-versions.ps1

Standalone script for updating packages without changing frameworks.

**Usage:**
```powershell
# Update all packages
pwsh ./update-package-versions.ps1

# Preview changes without applying
pwsh ./update-package-versions.ps1 -WhatIf
```

**Parameters:**
- `-RootDirectory` - Root directory to scan (default: current directory)
- `-WhatIf` - Preview mode, shows what would be updated without making changes

#### update-dotnet-simple.ps1

Complete framework and package update script (8 steps):
1. Detect .NET projects
2. Scan for project files
3. Update target frameworks
4. Remove package versions from projects
5. Collect package references
6. Fetch latest versions for frameworks
7. **Create Directory.Packages.props if missing (never overwrites existing)**
8. **Add missing packages + update all versions to latest**

**Key Improvement:** Step 7 now **preserves** existing Directory.Packages.props files instead of overwriting them. Step 8 intelligently:
- Adds any missing packages from your projects
- Updates all package versions to latest compatible versions
- Respects framework conditions and structure

This ensures that:
- ? Existing package configurations are never lost
- ? Manual customizations are preserved
- ? Only versions are updated, not structure
- ? Missing packages are added automatically

## How It Works

### Step 1: Parse Directory.Packages.props

The system scans for `<ItemGroup>` elements with `Condition` attributes:

```xml
<ItemGroup Condition="'$(TargetFramework)' == 'net9.0'">
  <PackageVersion Include="Microsoft.Extensions.Hosting" Version="9.0.0" />
</ItemGroup>
```

### Step 2: Extract Framework Version

From the condition, extracts the major framework version:
- `'$(TargetFramework)' == 'net9.0'` ? major version = `9`

### Step 3: Query NuGet API

Queries NuGet.org API for the package:
```
https://api.nuget.org/v3-flatcontainer/[package-id]/index.json
```

Filters versions:
- Only stable releases (no `-preview`, `-rc`, etc.)
- Versions matching major framework version (e.g., `9.x.x` for net9.0)

### Step 4: Update Version

Replaces the `Version` attribute with the latest compatible version:

```xml
<!-- Before -->
<PackageVersion Include="Microsoft.Extensions.Hosting" Version="9.0.0" />

<!-- After -->
<PackageVersion Include="Microsoft.Extensions.Hosting" Version="9.0.5" />
```

### Step 5: Save Changes

Writes updated XML back to file with UTF-8 encoding (no BOM).

## Example Scenarios

### Scenario 1: Multi-Targeting Project

**Directory.Packages.props:**
```xml
<Project>
  <ItemGroup Condition="'$(TargetFramework)' == 'net8.0'">
    <PackageVersion Include="Microsoft.Extensions.Hosting" Version="8.0.0" />
    <PackageVersion Include="Serilog" Version="3.1.0" />
  </ItemGroup>
  
  <ItemGroup Condition="'$(TargetFramework)' == 'net9.0'">
    <PackageVersion Include="Microsoft.Extensions.Hosting" Version="9.0.0" />
    <PackageVersion Include="Serilog" Version="3.1.0" />
  </ItemGroup>
  
  <ItemGroup>
    <PackageVersion Include="Newtonsoft.Json" Version="13.0.1" />
  </ItemGroup>
</Project>
```

**After running update-package-versions.ps1:**
```xml
<Project>
  <ItemGroup Condition="'$(TargetFramework)' == 'net8.0'">
    <PackageVersion Include="Microsoft.Extensions.Hosting" Version="8.0.2" />
    <PackageVersion Include="Serilog" Version="3.1.1" />
  </ItemGroup>
  
  <ItemGroup Condition="'$(TargetFramework)' == 'net9.0'">
    <PackageVersion Include="Microsoft.Extensions.Hosting" Version="9.0.5" />
    <PackageVersion Include="Serilog" Version="3.1.1" />
  </ItemGroup>
  
  <ItemGroup>
    <PackageVersion Include="Newtonsoft.Json" Version="13.0.3" />
  </ItemGroup>
</Project>
```

**What happened:**
- `Microsoft.Extensions.Hosting` for net8.0: `8.0.0` ? `8.0.2` (latest 8.x)
- `Microsoft.Extensions.Hosting` for net9.0: `9.0.0` ? `9.0.5` (latest 9.x)
- `Serilog`: `3.1.0` ? `3.1.1` (same for both frameworks)
- `Newtonsoft.Json`: `13.0.1` ? `13.0.3` (framework-agnostic, latest stable)

### Scenario 2: Single Framework Project

**Directory.Packages.props:**
```xml
<Project>
  <ItemGroup Condition="'$(TargetFramework)' == 'net9.0'">
    <PackageVersion Include="Microsoft.Extensions.DependencyInjection" Version="9.0.0" />
    <PackageVersion Include="Microsoft.Extensions.Logging" Version="9.0.0" />
  </ItemGroup>
</Project>
```

**After update:**
```xml
<Project>
  <ItemGroup Condition="'$(TargetFramework)' == 'net9.0'">
    <PackageVersion Include="Microsoft.Extensions.DependencyInjection" Version="9.0.5" />
    <PackageVersion Include="Microsoft.Extensions.Logging" Version="9.0.5" />
  </ItemGroup>
</Project>
```

## Integration with Workflows

### Manual Update Workflow

Create `.github/workflows/update-packages.yml`:

```yaml
name: Update Package Versions

on:
  workflow_dispatch:  # Manual trigger
  schedule:
    - cron: '0 3 * * 1'  # Weekly, Monday 3 AM

jobs:
  update:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Download update script
        run: |
          mkdir -p tools/modules
          curl -fsSL https://raw.githubusercontent.com/Zonit/.github/refs/heads/master/tools/update-package-versions.ps1 \
            -o tools/update-package-versions.ps1
          curl -fsSL https://raw.githubusercontent.com/Zonit/.github/refs/heads/master/tools/modules/PackageVersionUpdater.psm1 \
            -o tools/modules/PackageVersionUpdater.psm1
      
      - name: Update package versions
        run: pwsh ./tools/update-package-versions.ps1
      
      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v5
        with:
          commit-message: 'chore: update NuGet package versions'
          title: 'Update NuGet Package Versions'
          body: |
            Automated package version updates to latest compatible versions.
            
            **What changed:**
            - Updated all packages in Directory.Packages.props
            - Framework-aware version selection
            
            **Next steps:**
            1. Review changes
            2. Run `dotnet restore`
            3. Run `dotnet build`
            4. Run tests
          branch: auto/update-packages
          delete-branch: true
```

### Using in Organization-Wide Updates

The Package Version Updater is automatically integrated into the organization-wide .NET update workflow (`org-wide-dotnet-update.yml`). When you run the organization update, it:

1. Updates target frameworks in all projects
2. Creates/updates Directory.Packages.props structure
3. **Automatically updates all package versions to latest compatible versions**
4. Creates PRs for each repository

No additional configuration needed!

## Rate Limiting & Best Practices

### NuGet API Rate Limits

The scripts implement a 100ms delay between API calls to respect NuGet.org rate limits:

```powershell
Start-Sleep -Milliseconds 100
```

For large repositories with many packages, this ensures stable operation.

### Best Practices

1. **Always test locally first:**
   ```powershell
   pwsh ./update-package-versions.ps1 -WhatIf
   ```

2. **Review changes before committing:**
   ```bash
   git diff Directory.Packages.props
   ```

3. **Test after updates:**
   ```bash
   dotnet restore
   dotnet build
   dotnet test
   ```

4. **Schedule updates during low-activity periods:**
   - Use cron: `0 3 * * 1` (Monday 3 AM)

5. **Enable PR auto-merge for minor updates:**
   - Configure branch protection rules
   - Require status checks to pass

## Troubleshooting

### Package Update Fails

**Problem:** Script reports "ERROR" for a package

**Causes:**
- Package doesn't exist on NuGet.org
- Network issues
- Invalid package ID
- Package was unpublished

**Solution:**
1. Verify package exists: https://www.nuget.org/packages/[PackageId]
2. Check spelling of package ID
3. Retry with better network connection
4. Remove deprecated packages manually

### No Updates Applied

**Problem:** Script runs but no changes are made

**Possible reasons:**
1. All packages already at latest versions
2. No Directory.Packages.props found
3. Wrong framework versions in conditions

**Solution:**
- Check output: "Already up to date" vs "ERROR"
- Verify Directory.Packages.props exists
- Check condition syntax: `'$(TargetFramework)' == 'net9.0'`

### Breaking Changes After Update

**Problem:** Build fails after package updates

**Solution:**
1. Review the PR diff carefully
2. Check package release notes for breaking changes
3. Use `git bisect` to identify problematic update
4. Pin problematic package to previous version temporarily
5. Update code to accommodate breaking changes

## Future Enhancements

### Planned Features

1. **Prerelease Support:**
   - Add `-IncludePrerelease` parameter
   - Support for preview versions (net10.0-preview, etc.)

2. **Version Constraints:**
   - Support for max version constraints
   - Allow excluding specific major versions

3. **Dependency Analysis:**
   - Check for dependency conflicts
   - Validate compatibility before updating

4. **Rollback Support:**
   - Automatic rollback on build failure
   - Version history tracking

5. **Smart Update Scheduling:**
   - Update only security patches by default
   - Major/minor updates on separate schedule

## Contributing

To improve the Package Version Update system:

1. Fork the `.github` repository
2. Create a feature branch
3. Add/modify modules in `tools/modules/`
4. Test with real repositories
5. Submit a pull request

**Testing Guidelines:**
- Test with single-framework projects
- Test with multi-targeting projects
- Test with empty Directory.Packages.props
- Test with large number of packages (50+)
- Test error handling (invalid packages, network issues)

## Support

For issues or questions:
- Open an issue in the `.github` repository
- Tag with `package-updates` label
- Provide sample Directory.Packages.props if applicable

---

**Last Updated:** 2024
**Maintained by:** Zonit DevOps Team
