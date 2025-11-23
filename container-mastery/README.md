# container-mastery

Create and optimize Dockerfiles using layer caching best practices.

## Overview

This plugin helps you create optimized Docker images by applying proven patterns from production experience. Instead of blindly following "best practices," it creates both simple and optimized versions of Dockerfiles, compares them with actual measurements, and recommends the best approach for your specific use case.

## Skills

### `container-mastery:optimizing-dockerfiles`

Use when creating or optimizing Dockerfiles. This skill:

1. **Offers both approaches** - Creates a simple single-stage Dockerfile AND a multi-stage optimized version
2. **Measures actual results** - Compares image size, build time, layer count, and security surface
3. **Recommends honestly** - Tells you which version is actually better based on data, not assumptions
4. **Explains tradeoffs** - Helps you understand when optimization helps and when it doesn't

#### When to Use

- Creating a new Dockerfile
- Optimizing an existing Dockerfile
- Questions about Docker image size, build speed, or layer caching

#### Key Principles

Based on real-world experience with Docker optimization:

**Layer Caching:**
- Order layers from least to most frequently changing
- Separate dependencies (stable) from application code (changes frequently)
- Docker reuses unchanged layers to speed up builds

**Cleanup Strategy:**
- Always clean up in the SAME RUN command that creates the mess
- Don't clean up in separate layers (doesn't reduce image size)

**Multi-stage Builds:**
- Separate build-time from runtime dependencies
- Keep build tools out of final image
- Only beneficial when you have actual separation between dependencies and code

**Honest Evaluation:**
- Sometimes the "simple" version is actually better
- Optimization should provide measurable benefits
- Complexity has a cost in maintainability

## Examples

### Example 1: Node.js Application (Multi-stage wins)

**Simple version:** 800MB with development dependencies
**Optimized version:** 200MB with only production dependencies
**Winner:** Optimized (4x smaller, better cache reuse)

### Example 2: Pre-built Binary (Simple wins)

**Simple version:** 397MB, straightforward
**Optimized version:** 420MB, unnecessarily complex
**Winner:** Simple (smaller and clearer)

### Example 3: Compiled Go Application (Multi-stage wins)

**Simple version:** 850MB with full Go toolchain
**Optimized version:** 15MB with just the binary
**Winner:** Optimized (56x smaller!)

## Installation

This plugin is part of the Skillbook marketplace. To use it:

```bash
# Navigate to your project
cd your-project/

# The skill will be available when you run Claude Code in this directory
```

## Usage

Just ask Claude to create a Dockerfile:

```
Create a Dockerfile for my Node.js application
```

Claude will ask:

> Would you like me to create an optimized version using multi-stage builds and layer caching strategies? I can create both a simple version and an optimized version, then compare them to see which works better for your use case.

Say yes, and Claude will:
1. Create both versions
2. Build and compare them
3. Recommend the best approach with measurements
4. Explain the tradeoffs

## Background

This plugin is based on learnings from:
- Real-world experiments building containerized development environments
- Production experience with Docker optimization patterns
- Analysis of layer caching behavior and multi-stage builds

The key insight: **optimization should be based on measurements, not assumptions.** Sometimes the "optimized" approach is actually worse for your specific use case.

## License

MIT
