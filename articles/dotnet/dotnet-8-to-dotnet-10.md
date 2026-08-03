---
layout: page
title: What I Learned Upgrading a Production Service from .NET 8 to .NET 10
permalink: /articles/dotnet/dotnet-8-to-dotnet-10/
---

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

## Npgsql 10 and GSS-API encryption

If you're using Npgsql with PostgreSQL, be aware that Npgsql 10 defaults to `Prefer` for GSS-API session encryption. In some Linux environments without Kerberos installed, this may generate errors when attempting to get GSSAPI credentials.

When running in Kubernetes, you might not notice this issue until you try to check your application logs and state—that's when you'll see errors related to `libgssapi`:

```
Unable to load shared library 'libgssapi_krb5.so.2'
```

You can fix this by explicitly setting `GSS Encryption Mode=Disable` in your connection string if you're not using GSS-API:

```csharp
"Host=myserver;Database=mydb;Username=myuser;Password=mypass;GSS Encryption Mode=Disable"
```

See the [Npgsql Security documentation](https://www.npgsql.org/doc/security.html) for more details.

## Docker image changes: `ADDUSER` removed from bookworm-slim

When upgrading to .NET 10, I discovered that the `adduser` command was removed from the `bookworm-slim` base images. This breaks Dockerfiles that create non-root users with `adduser`.

If your Dockerfile uses:

```dockerfile
RUN adduser --disabled-password --gecos "" appuser
```

You have a few options:

1. **Switch to `noble` or `azurelinux` images** - These images still include the `adduser` command
2. **Use `useradd` instead** - The lower-level command is still available:
   ```dockerfile
   RUN groupadd --gid 1653 appuser && \
       useradd --uid 1653 --gid 1653 --create-home appuser
   ```
   **Important:** If you don't specify `--uid` and `--gid` explicitly, it will default to UID/GID 1638, which may not match your expected permissions.
3. **Use chiseled images** - `azurelinux/chiseled` images include non-root user support built-in

For more context on this change, see the [GitHub discussion](https://github.com/dotnet/docs/issues/40441).

## Key lesson

A successful build does not guarantee a successful upgrade.

Testing and monitoring are essential parts of the migration process.