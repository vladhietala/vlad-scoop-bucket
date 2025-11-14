# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Scoop bucket repository for Windows package management. Scoop is a command-line installer for Windows that uses JSON manifests to define how applications should be installed. This bucket contains custom app manifests for packages not available in the main Scoop repository.

## Architecture

### Manifest Structure

App manifests are JSON files located in `bucket/` directory. Each manifest defines:
- Version information
- Download URLs (with architecture-specific variants: 64bit, 32bit, arm64)
- SHA hashes for integrity verification
- Binary paths for command-line access
- Auto-update configuration with `checkver` (version checking) and `autoupdate` (automatic updates)

Key manifest components:
- `checkver`: Defines how to check for new versions (typically GitHub releases or regex patterns)
- `autoupdate`: Template URLs and hash sources for automatic version updates
- `architecture`: Platform-specific download URLs and extraction settings
- `bin`: Executable files to add to PATH

### Automation Scripts

All scripts in `bin/` are thin wrappers that delegate to Scoop core functionality:

- **checkver.ps1**: Check for new versions of apps in the bucket
- **auto-pr.ps1**: Automatically create pull requests for version updates (upstream: `vladhietala/vlad-scoop-bucket:main`)
- **formatjson.ps1**: Format JSON manifests according to Scoop standards
- **checkurls.ps1**: Verify download URLs are accessible
- **checkhashes.ps1**: Verify file hashes match manifest definitions
- **test.ps1**: Run Pester tests to validate manifests

All scripts require `$env:SCOOP_HOME` to be set (automatically detected via `scoop prefix scoop`).

### CI/CD Workflows

- **ci.yml**: Runs tests on all pull requests and pushes using both Windows PowerShell and PowerShell Core
- **excavator.yml**: Scheduled job (every 4 hours) that automatically checks for updates and creates PRs

## Common Commands

### Testing Manifests

```powershell
# Run all tests (requires Pester 5.2.0+ and BuildHelpers 2.0.1+)
.\bin\test.ps1

# Check for version updates
.\bin\checkver.ps1

# Verify all download URLs are accessible
.\bin\checkurls.ps1

# Verify file hashes
.\bin\checkhashes.ps1

# Format JSON manifests
.\bin\formatjson.ps1
```

### Development Workflow

```powershell
# Add this bucket locally for testing
scoop bucket add vlad-bucket .

# Install an app from this bucket
scoop install vlad-bucket/<app-name>

# Test installation without actually installing
scoop install --no-cache vlad-bucket/<app-name>

# Uninstall for testing
scoop uninstall <app-name>
```

### Creating/Updating Manifests

When creating or updating manifests:

1. **Version updates**: Modify `version` field and update all architecture-specific URLs and hashes
2. **Hash generation**: Use `scoop download <url>` or manually calculate SHA256/SHA512
3. **Testing**: Always run `.\bin\test.ps1` before committing
4. **Auto-update**: Ensure `checkver` and `autoupdate` sections are properly configured for automated future updates

### Manifest Variables

Common variables used in `autoupdate` sections:
- `$version`: Full version string (e.g., "1.42.8")
- `$cleanVersion`: Version with dots removed (e.g., "1428" or "10060")
- `$baseurl`: Base URL from the download link

## Repository-Specific Notes

- The upstream repository is configured as `vladhietala/vlad-scoop-bucket:main` in `bin/auto-pr.ps1`
- Tests require Scoop core to be available and will checkout `ScoopInstaller/Scoop` for validation
- The Excavator workflow handles automated version checking and updates via GitHub Actions
