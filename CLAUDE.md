# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Rust library for loading and working with glTF 2.0 files (a 3D asset transmission format). The crate provides rustic utilities for parsing glTF JSON, loading binary data, and iterating over 3D scene data.

## Project Structure

This is a Cargo workspace with three crates:

- **gltf** (main crate): High-level API for loading and working with glTF assets
- **gltf-json**: Low-level (de)serialization of glTF JSON structures
- **gltf-derive**: Procedural macros used by gltf-json

The main crate wraps the JSON structures from gltf-json with convenient iterator-based APIs and provides import functionality for loading buffers and images.

## Common Commands

### Build and Test
```bash
# Build the project
cargo build

# Build with all features enabled
cargo build --all-features

# Run all tests (requires glTF-Sample-Assets repo cloned in project root)
cargo test

# Run a specific test
cargo test import_sample_models
cargo test roundtrip_binary_gltf

# Check compilation without building
cargo check
```

### Running Examples
```bash
# Display glTF JSON structure
cargo run --example gltf-display path/to/asset.gltf

# Export glTF JSON
cargo run --example gltf-export

# Roundtrip glTF JSON (deserialize + serialize)
cargo run --example gltf-roundtrip path/to/asset.gltf

# Visualize scene hierarchy (requires "extensions" and "names" features)
cargo run --example gltf-tree path/to/asset.gltf
```

### Linting and Formatting
```bash
# Format code (uses rustfmt.toml config)
cargo fmt

# Run clippy (uses clippy.toml config)
cargo clippy

# Run clippy with all features
cargo clippy --all-features
```

### Documentation
```bash
# Build and open documentation
cargo doc --open

# Build docs with all features (as docs.rs does)
cargo doc --all-features
```

## Development Setup

### Test Data
Running integration tests requires cloning the glTF sample models repository:
```bash
git clone https://github.com/KhronosGroup/glTF-Sample-Assets.git
```
The test suite in `tests/import_sample_models.rs` expects this at `glTF-Sample-Assets/Models/`.

## Architecture

### Core Types and Data Flow

1. **Loading**: `Gltf::open()` or `gltf::import()` parse files and load data
   - `Gltf` struct contains a `Document` (JSON wrapper) and optional binary blob
   - `Document` wraps `json::Root` and provides high-level iterators

2. **Document Structure**: The `Document` provides iterators for accessing:
   - Scenes, nodes (scene hierarchy as a tree)
   - Meshes, primitives, materials
   - Accessors (typed views into buffer data)
   - Buffers, buffer views, images
   - Animations, skins, cameras
   - Extensions (KHR_lights_punctual, KHR_materials_*, etc.)

3. **Data Access Pattern**: All types follow a consistent pattern:
   - JSON types in `gltf-json` crate (low-level, direct serialization)
   - Wrapper types in main crate that hold a reference to `Document` and index
   - Iterator types that yield wrapper types
   - Example: `Document::meshes()` returns `iter::Meshes` which yields `Mesh` wrappers

4. **Feature Flags**: The crate uses extensive feature flags for:
   - `import`: File loading and buffer/image import (enabled by default)
   - `names`, `extras`: Include optional glTF fields
   - `KHR_*` and `EXT_*`: glTF extension support
   - Each extension feature flag propagates to the gltf-json dependency

### Key Modules

- `accessor`: Reading typed vertex data from buffers with normalization support
- `binary`: Binary glTF (.glb) format parsing
- `import`: File system import with buffer and image loading
- `mesh`: Mesh primitives, attributes, and iteration utilities
- `material`: PBR materials and extensions
- `scene`: Node hierarchy and transformations
- `animation`, `skin`: Animation and skeletal animation data

### Normalization System

The `Normalize` trait (in lib.rs) handles type conversions between glTF data types (i8, u8, i16, u16, f32) with proper value range mapping. This is used extensively in accessor utilities.

## Code Style

- Minimum Rust version: 1.61
- Uses Rust 2021 edition
- Code must be formatted with rustfmt (checked in CI)
- Clippy configuration in `clippy.toml` allows the identifier "glTF"
- Documentation required (`#![deny(missing_docs)]`)
- Uses `#[cfg_attr(docsrs, doc(cfg(feature = "...")))]` for feature-gated docs

## Contribution Notes

- Submit PRs to the `master` branch
- Update CHANGELOG.md under "Unreleased" section
- Follow the Rust API guidelines
- Dual-licensed MIT or Apache-2.0
