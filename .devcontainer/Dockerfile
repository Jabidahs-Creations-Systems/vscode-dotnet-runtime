# Multi-architecture development container for .NET Runtime Extension
# This Dockerfile demonstrates proper handling of multi-architecture builds
# to avoid the common pitfall of hard-coding architecture-specific downloads

ARG VARIANT=8.0
FROM mcr.microsoft.com/dotnet/sdk:${VARIANT}

# Install development tools
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends \
        curl \
        ca-certificates \
        git \
        sudo \
        procps \
    && apt-get autoremove -y && apt-get clean -y && rm -rf /var/lib/apt/lists/*

# Create non-root user
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

# Example: Architecture-specific package installation
# This demonstrates the CORRECT way to handle multi-architecture builds
# TARGETARCH is automatically populated by Docker BuildKit during multi-platform builds.
# It will be empty when building without buildx or multi-platform mode.
ARG TARGETARCH

# Example of architecture-aware installation
# This is the pattern that prevents errors like:
# "package architecture (amd64) does not match system (arm64)"
RUN echo "Building for architecture: ${TARGETARCH:-auto-detected}" && \
    case "${TARGETARCH}" in \
        amd64) \
            echo "Configuring for AMD64/x86_64 architecture"; \
            # Add AMD64-specific installations here if needed
            ;; \
        arm64) \
            echo "Configuring for ARM64/aarch64 architecture"; \
            # Add ARM64-specific installations here if needed
            ;; \
        "") \
            echo "TARGETARCH not set - likely single-platform build. Architecture will be auto-detected at runtime."; \
            ;; \
        *) \
            echo "Unsupported architecture: ${TARGETARCH}"; \
            exit 1; \
            ;; \
    esac

# Set the default user
USER $USERNAME

# Note: Node.js is installed via dev container features (see devcontainer.json)
# This ensures proper multi-architecture support without manual installation
