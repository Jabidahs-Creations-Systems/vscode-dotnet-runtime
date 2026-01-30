# Dev Container for Multi-Architecture Development

This development container is configured to support multi-architecture development and demonstrates best practices for handling different CPU architectures.

## Features

- **Multi-Architecture Support**: Built to run on both AMD64 (x86_64) and ARM64 (aarch64) systems
- **.NET SDK**: Pre-installed .NET SDK for building and testing the extension
- **Node.js**: Includes Node.js for VS Code extension development
- **Architecture Detection**: Demonstrates proper use of Docker's TARGETARCH variable

## Architecture-Aware Configuration

The Dockerfile shows the correct pattern for handling architecture-specific installations:

```dockerfile
ARG TARGETARCH
RUN case "${TARGETARCH}" in \
        amd64) \
            # AMD64-specific installations
            ;; \
        arm64) \
            # ARM64-specific installations
            ;; \
    esac
```

This approach ensures that:
1. Packages are downloaded for the correct architecture
2. No hard-coded architecture assumptions are made
3. The container works on both Intel/AMD and ARM-based systems

## Building for Multiple Architectures

To build this container for multiple architectures:

```bash
docker buildx build --platform linux/amd64,linux/arm64 -t vscode-dotnet-dev .
```

## Common Pitfalls Avoided

This configuration avoids common mistakes such as:

❌ **Hard-coding architecture in download URLs**:
```bash
# WRONG - always downloads AMD64
curl https://example.com/ubuntu_64bit/package.deb
```

✅ **Using architecture detection**:
```bash
# CORRECT - downloads for current architecture
if [ "$TARGETARCH" = "amd64" ]; then
    curl https://example.com/ubuntu_64bit/package.deb
elif [ "$TARGETARCH" = "arm64" ]; then
    curl https://example.com/ubuntu_arm64/package.deb
fi
```

## Reference

See [multi-architecture-support.md](../Documentation/multi-architecture-support.md) for more details on how this repository handles multiple architectures.

## Related Issue

This dev container configuration was created in response to multi-architecture build failures, such as: https://github.com/xpipe-io/xpipe-webtop/actions/runs/20578223451/job/59099982309
