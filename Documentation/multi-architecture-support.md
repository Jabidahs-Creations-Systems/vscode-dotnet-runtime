# Multi-Architecture Support

This document describes how the .NET Runtime Extension properly handles multiple CPU architectures (x86, x64, ARM64, etc.).

## Overview

The extension supports installation of .NET runtimes and SDKs across multiple architectures:
- **x86** (32-bit Intel/AMD)
- **x64** (64-bit Intel/AMD, also known as AMD64)
- **arm64** (64-bit ARM, also known as AArch64)

## Architecture Detection

### Automatic Detection

The extension uses Node.js's `os.arch()` function to automatically detect the system architecture. This ensures that the correct version of .NET is downloaded and installed for your system.

```typescript
// From DotnetCoreAcquisitionWorker.ts
public static defaultArchitecture(): string {
    return os.arch();
}
```

### Executable Architecture Detection

For existing .NET installations, the extension can detect the architecture of executable files across different platforms:

- **Windows**: PE/COFF format parsing
- **macOS**: Mach-O format parsing
- **Linux**: ELF format parsing

This is implemented in `ExecutableArchitectureDetector.ts`.

## Download URL Construction

When downloading .NET packages, the extension constructs architecture-specific URLs. From `dotnet-install.sh`:

```bash
download_link="$azure_feed/Runtime/$specific_version/dotnet-runtime-$specific_product_version-$osname-$normalized_architecture.tar.gz"
```

The `$normalized_architecture` variable ensures the correct package is downloaded (e.g., `linux-arm64` or `linux-x64`).

## Best Practices for Multi-Architecture Support

When adding features that involve downloading or installing architecture-specific binaries, follow these patterns:

### ✅ DO: Use Architecture Detection

```bash
# Detect architecture
ARCH=$(uname -m)
case $ARCH in
    aarch64|arm64)
        DOWNLOAD_ARCH="arm64"
        ;;
    x86_64|amd64)
        DOWNLOAD_ARCH="x64"
        ;;
    *)
        echo "Unsupported architecture: $ARCH"
        exit 1
        ;;
esac

# Use detected architecture in download URL
curl "https://example.com/downloads/${DOWNLOAD_ARCH}/package.tar.gz"
```

### ✅ DO: Use Docker TARGETARCH for Container Builds

When building Docker images for multiple architectures:

```dockerfile
ARG TARGETARCH
RUN if [ "$TARGETARCH" = "amd64" ]; then \
        curl "https://example.com/downloads/x64/package.deb"; \
    elif [ "$TARGETARCH" = "arm64" ]; then \
        curl "https://example.com/downloads/arm64/package.deb"; \
    fi
```

### ❌ DON'T: Hard-Code Architecture

```bash
# BAD: Always downloads x64 version
curl "https://example.com/downloads/x64/package.tar.gz"

# BAD: Always downloads AMD64 .deb package
curl "https://s3.amazonaws.com/.../ubuntu_64bit/plugin.deb"
```

## Common Architecture Mismatch Issues

### Package Architecture Mismatch

**Problem**: Installing an AMD64 package on an ARM64 system results in:
```
package architecture (amd64) does not match system (arm64)
```

**Solution**: Always detect the system architecture before downloading packages.

### Binary Incompatibility

**Problem**: Running an x64 executable on ARM64 without emulation fails with:
```
cannot execute binary file: Exec format error
```

**Solution**: Ensure downloaded binaries match the system architecture.

## Testing Multi-Architecture Support

### Local Testing

Test architecture detection:
```bash
node -e "console.log('Architecture:', process.arch)"
uname -m
```

### CI/CD Testing

Use GitHub Actions matrix builds to test multiple architectures:

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, macos-latest, windows-latest]
    architecture: [x64, arm64]
```

## References

- [Node.js os.arch() documentation](https://nodejs.org/api/os.html#os_os_arch)
- [Docker Multi-Platform Builds](https://docs.docker.com/build/building/multi-platform/)
- [.NET Supported Architectures](https://learn.microsoft.com/en-us/dotnet/core/install/)

## Related Issue

This document was created in reference to a multi-architecture build failure observed in another project: https://github.com/xpipe-io/xpipe-webtop/actions/runs/20578223451/job/59099982309

The failure demonstrated the importance of proper architecture detection when downloading and installing architecture-specific packages.
