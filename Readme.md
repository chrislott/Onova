# Onova

[![Build](https://img.shields.io/appveyor/ci/Tyrrrz/Onova/master.svg)](https://ci.appveyor.com/project/Tyrrrz/Onova)
[![Tests](https://img.shields.io/appveyor/tests/Tyrrrz/Onova/master.svg)](https://ci.appveyor.com/project/Tyrrrz/Onova)
[![NuGet](https://img.shields.io/nuget/v/Onova.svg)](https://nuget.org/packages/Onova)
[![NuGet](https://img.shields.io/nuget/dt/Onova.svg)](https://nuget.org/packages/Onova)

Onova is a library that provides a simple but extensible interface to perform auto-updates in applications. It was designed primarily for open source projects that distribute their releases using archive files instead of installers, but can be configured to support almost any setup. Acquired updates are applied in place via an external process, so there are no launchers, release files or special directories.

## Download

- Using NuGet: `Install-Package Onova`
- [Continuous integration](https://ci.appveyor.com/project/Tyrrrz/Onova)

## Features

- Minimal configuration
- Supported resolvers:
  - `LocalPackageResolver` - file system
  - `GithubPackageResolver` - GitHub releases
  - `WebPackageResolver` - web version manifest
- Supported extractors:
  - `ZipPackageExtractor` - zip archives
- Can be extended with custom resolvers and extractors
- Progress reporting and cancellation
- Can apply packages of any version, not necessarily latest
- In-place update via an external executable
- Automatically prompts for elevated privileges if necessary
- Fully self-contained
- Targets .NET Framework 4.6+

## Workflow

### Package resolving

Packages and their versions are resolved using an implementation of `IPackageResolver`. Currently there are 3 built-in implementations:

#### `LocalPackageResolver` 

This implementation looks for files in the given directory using a predefined search pattern (default is `*.onv`). Package versions are extracted from file names, e.g. file named `MyProject-v2.1.5.onv` corresponds to package version `2.1.5`.

#### `GithubPackageResolver`

This implementation looks for assets with predefined name (default is `Package.onv`) in releases of the given GitHub repository. Package versions are extracted from release names, e.g. release named `v1.0` corresponds to package version `1.0`.

#### `WebPackageResolver`

This implementation requests a version manifest using given URL. The manifest should contain a list of package versions and their URLs, separated by space, one line per package. E.g.:
```
1.0 https://my.server.com/1.0.zip
2.0 https://my.server.com/2.0.zip
```

> **Note:** Packages whose versions could not be extracted will not be seen by the resolvers. Also, if there are multiple packages with the same version, only one of them will be available.

### Package extraction

Downloaded packages are extracted using an implementation of `IPackageExtractor`. Currently there is 1 built-in implementation:

#### `ZipPackageExtractor`

This implementation treats packages as zip archives.

## Usage

##### Basic usage example

```c#
// Set up the manager to look for packages in given directory and treat them as zips
var resolver = new LocalPackageResolver("c:\\test\\packages");
var extractor = new ZipPackageExtractor();
var manager = new UpdateManager(resolver, extractor);

// Check for new version and perform full update if available
await manager.CheckPerformUpdateAsync();
```

##### Handling intermediate steps manually

```c#
// Check for updates
var result = await manager.CheckForUpdatesAsync();
if (result.CanUpdate)
{
    // Prepare package so that it can be applied later
    // (supports progress reporting and cancellation in overloads)
    await manager.PreparePackageAsync(result.LastPackageVersion);

    // Launch updater that will apply package and restart application
    // (restart can be enabled/disabled in overloads)
    await manager.ApplyPackageAsync(result.LastPackageVersion);

    // External updater will wait until the application exits
    Environment.Exit(0);
}
```

## Libraries used

- [Newtonsoft.Json](https://github.com/JamesNK/Newtonsoft.Json)
- [Moq](https://github.com/Moq/moq4)
- [NUnit](https://github.com/nunit/nunit)
- [CliWrap](https://github.com/Tyrrrz/CliWrap)
- [Mono.Cecil](https://github.com/jbevain/cecil)