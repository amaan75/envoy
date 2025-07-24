# Envoy Docker Image Build Process

This document describes how to execute all the scripts needed to build Envoy Docker images in the `ci` folder.

## Quick Start

To build Envoy Docker images, execute these two main scripts in the `ci` directory:

```bash
# Step 1: Build Envoy binary
./ci/run_envoy_docker.sh './ci/do_ci.sh release.server_only'

# Step 2: Build Docker images  
./ci/docker_ci.sh
```

## Alternative Execution Methods

We've provided several wrapper scripts for your convenience:

1. **`build_images.sh`** - Simple execution of core build commands
2. **`execute_build_scripts.sh`** - Detailed execution with explanations
3. **`build_envoy_images_demo.sh`** - Comprehensive demonstration and diagnostics

## Key Build Scripts

### 1. Binary Build Scripts
- **`ci/run_envoy_docker.sh`** - Docker wrapper for building binaries
- **`ci/do_ci.sh`** - Main CI script with various build targets
- **`ci/build_setup.sh`** - Build environment setup

### 2. Docker Image Build Scripts  
- **`ci/docker_ci.sh`** - Main Docker image builder
- **`ci/Dockerfile-envoy`** - Multi-stage Dockerfile for Envoy images
- **`ci/envoy_build_sha.sh`** - Build container SHA management

## Build Targets

### Binary Build Targets
```bash
# Development build (fast)
./ci/run_envoy_docker.sh './ci/do_ci.sh dev'

# Release build (optimized)
./ci/run_envoy_docker.sh './ci/do_ci.sh release.server_only'

# Debug build
./ci/run_envoy_docker.sh './ci/do_ci.sh debug.server_only'

# With testing
./ci/run_envoy_docker.sh './ci/do_ci.sh release'
```

### Docker Image Variants Built
The `docker_ci.sh` script builds 7 image variants:

1. **`envoyproxy/envoy`** - Standard Envoy image
2. **`envoyproxy/envoy-debug`** - Debug version with symbols
3. **`envoyproxy/envoy-contrib`** - With contrib extensions
4. **`envoyproxy/envoy-contrib-debug`** - Contrib with debug symbols
5. **`envoyproxy/envoy-distroless`** - Minimal distroless image
6. **`envoyproxy/envoy-google-vrp`** - Google VRP variant
7. **`envoyproxy/envoy-tools`** - With additional tools

## System Requirements

### Required Tools
- Docker (with buildx support)
- Bazel 7.6.0
- Python 3.x
- skopeo (for image pushing)

### Network Requirements
The build process requires internet access to:
- `releases.bazel.build` - For downloading Bazel
- Docker Hub - For base images  
- Various package repositories - For dependencies

### Platform Support
- Linux x86_64 (amd64)
- Linux ARM64 (aarch64)
- Windows (limited support)

## Environment Variables

### Common Variables
```bash
# Build directory (default: /tmp/envoy-docker-build)
export ENVOY_DOCKER_BUILD_DIR=/path/to/build

# Docker options
export ENVOY_DOCKER_OPTIONS="--dns 8.8.8.8"

# Force pull latest images
export ENVOY_DOCKER_PULL=true

# Load images locally instead of pushing
export DOCKER_LOAD_IMAGES=1

# Dry run mode (shows commands without executing)
export DOCKER_CI_DRYRUN=true
```

### Build Configuration
```bash
# C++ standard library (libc++ or libstdc++)
export ENVOY_STDLIB=libc++

# Build architecture
export ENVOY_BUILD_ARCH=x86_64

# Bazel options
export BAZEL_BUILD_EXTRA_OPTIONS="--config=my-config"
```

## Directory Structure

After successful build:
```
linux/
├── amd64/
│   └── release.tar.zst    # x86_64 binary
└── arm64/
    └── release.tar.zst    # ARM64 binary

build_images/              # Docker image archives
├── envoy.tar
├── envoy-debug.tar
├── envoy-contrib.tar
├── envoy-contrib-debug.tar
├── envoy-distroless.tar
├── envoy-google-vrp.tar
└── envoy-tools.tar
```

## Troubleshooting

### Network Issues
If you encounter DNS resolution problems:

```bash
# Try different DNS servers
export ENVOY_DOCKER_OPTIONS="--dns 8.8.8.8 --dns 8.8.4.4"

# Use host networking
export ENVOY_DOCKER_OPTIONS="--network host"

# Configure proxy if needed
export http_proxy=http://your-proxy:port
export https_proxy=http://your-proxy:port
```

### Build Issues
```bash
# Clean build cache
rm -rf /tmp/envoy-docker-build

# Use different build image
export IMAGE_NAME=envoyproxy/envoy-build-ubuntu

# Enable verbose output
export BAZEL_BUILD_EXTRA_OPTIONS="--verbose_failures"
```

## Version Information

- **Envoy Version**: 1.34.4
- **Bazel Version**: 7.6.0
- **Base Image**: envoyproxy/envoy-build-ubuntu

## Notes

- The build process uses multi-stage Docker builds
- Cross-platform builds are supported via Docker buildx
- Images can be pushed to registries or saved locally
- Debug images include symbols for debugging
- Contrib images include experimental extensions

For more detailed information, see:
- `ci/README.md` - Comprehensive CI documentation
- `DEVELOPER.md` - Developer guide
- `CONTRIBUTING.md` - Contribution guidelines