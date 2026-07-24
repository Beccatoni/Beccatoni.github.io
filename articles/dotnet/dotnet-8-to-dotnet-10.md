---
layout: default
title: What I Learned Upgrading a Production Service from .NET 8 to .NET 10
---

# What I Learned Upgrading a Production Service from .NET 8 to .NET 10

Recently, I had the opportunity to upgrade a backend service from .NET 8 to .NET 10.

At first, the task appeared simple: update the target framework, restore the packages, and run the tests.

However, I learned that a runtime upgrade involves much more than changing one line in a project file.

## Why upgrade?

.NET runtime upgrades help applications stay aligned with supported frameworks, security updates, performance improvements, and newer library versions.

## Updating the target framework

Before:

```xml
<TargetFramework>net8.0</TargetFramework>
```
After:

```xml
<TargetFramework>net10.0</TargetFramework>
```

The framework change was the easiest part. Most of the work involved validating everything around it.


## What I checked

- NuGet package compatibility
- Unit tests
- Integration tests
- Docker builds
- CI/CD pipelines
- Application logs
- Runtime metrics

## Key lesson

A successful build does not guarantee a successful upgrade.

Testing and monitoring are essential parts of the migration process.