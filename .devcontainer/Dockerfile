# Multi-architecture development container for .NET Runtime Extension
# This Dockerfile demonstrates proper handling of multi-architecture builds
# to avoid the common pitfall of hard-coding architecture-specific downloads

ARG VARIANT=8.0
FROM mcr.microsoft.com/dotnet/sdk:${VARIANT}

# Install Node.js - using NodeSource which provides multi-arch packages
ARG NODE_VERSION=20
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends \
        curl \
        ca-certificates \
        gnupg \
    && mkdir -p /etc/apt/keyrings \
    && curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg \
    && echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_${NODE_VERSION}.x nodistro main" | tee /etc/apt/sources.list.d/nodesource.list \
    && apt-get update \
    && apt-get -y install --no-install-recommends nodejs \
    && apt-get autoremove -y && apt-get clean -y && rm -rf /var/lib/apt/lists/*

# Install development tools
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends \
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
# Use TARGETARCH which Docker automatically sets during multi-platform builds
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
        *) \
            echo "Architecture: ${TARGETARCH} - using default configuration"; \
            ;; \
    esac

# Set the default user
USER $USERNAME

# Display architecture information for verification
RUN echo "Container architecture: $(uname -m)" && \
    echo "Node.js architecture: $(node -e 'console.log(process.arch)')" && \
    echo ".NET architecture: $(dotnet --info | grep RID || echo 'N/A')"
