---
name: optimizing-dockerfiles
description: Use when creating Dockerfiles - offers to create both simple and optimized versions, then compares them to find the best approach
---

# Optimizing Dockerfiles

## When to Use This Skill

Use this skill when:
- User asks you to create a Dockerfile
- User asks you to optimize an existing Dockerfile
- User asks about Docker image size, build speed, or layer caching

## The Process

### 1. Initial Question

When a user asks you to create a Dockerfile, ALWAYS ask:

"Would you like me to create an optimized version using multi-stage builds and layer caching strategies? I can create both a simple version and an optimized version, then compare them to see which works better for your use case."

If they say no, create just a simple, straightforward Dockerfile.

If they say yes, proceed with the comparison approach.

### 2. Create Both Versions

Create two Dockerfiles:

**Dockerfile** (Simple version):
- Single-stage build
- Straightforward approach
- Apply basic best practices:
  - Use `--no-install-recommends` with apt-get
  - Clean up in the same RUN command (all in one layer)
  - Remove package manager caches
  - Order commands from least to most frequently changing

**Dockerfile.optimized** (Multi-stage version):
- Multi-stage build separating concerns
- Dependencies stage (changes rarely)
- Application stage (changes frequently)
- Selective copying to minimize final image size
- Layer separation for maximum cache reuse

### 3. Key Principles for Docker Optimization

Apply these learnings from real-world experience:

**Layer Caching:**
- Docker reuses layers when inputs haven't changed
- Order matters: put stable layers first, changing layers last
- Typical order:
  1. Base image
  2. System dependencies (apt packages)
  3. Language runtime dependencies (npm install, pip install)
  4. Application code
  5. Configuration files

**Cleanup Strategy:**
- Always clean up in the SAME RUN command that creates the mess
- Example: `RUN apt-get update && apt-get install -y foo && rm -rf /var/lib/apt/lists/*`
- DON'T do cleanup in a separate RUN command (creates a new layer, doesn't reduce size)

**Multi-stage Benefits:**
- Separate build-time dependencies from runtime dependencies
- Copy only what's needed to final image
- Keep build tools out of final image for security
- Works best when you have:
  - Dependencies that rarely change
  - Application code that changes frequently
  - Build tools not needed at runtime

**When Multi-stage DOESN'T Help:**
- Everything changes together (like copying a pre-built binary)
- No build step required
- No separation between dependencies and application code
- Single executable with no external dependencies

### 4. Compare the Versions

After creating both versions, build and compare them:

```bash
# Build both
docker build -f Dockerfile -t app-simple .
docker build -f Dockerfile.optimized -t app-optimized .

# Compare sizes
docker images | grep -E "(app-simple|app-optimized)"

# Compare layer counts and sizes
docker history app-simple --human=true
docker history app-optimized --human=true

# Test rebuild times (simulating code change)
touch <most-frequently-changed-file>
time docker build -f Dockerfile -t app-simple .
time docker build -f Dockerfile.optimized -t app-optimized .

# Check what's actually in the images
docker run --rm app-simple dpkg -l | wc -l
docker run --rm app-optimized dpkg -l | wc -l
```

### 5. Analysis and Recommendation

Analyze the results and provide a clear recommendation:

**Compare:**
- Image size (MB)
- Layer count
- Rebuild time with cache
- Package count (security surface)
- Complexity (maintainability)

**Recommend the version that:**
- Is smaller OR has a good reason for being larger
- Builds faster on typical changes
- Has fewer unnecessary packages
- Is simpler to maintain (if benefits are marginal)

**Be honest:** If the "optimized" version is actually worse (larger, slower, more complex with no benefits), recommend the simple version and explain why.

## Example Scenarios

### Scenario 1: Application with Separate Dependencies
**Best approach:** Multi-stage with separated dependency installation

```dockerfile
FROM node:20 AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:20-slim AS runtime
COPY --from=deps /app/node_modules ./node_modules
COPY . .
CMD ["node", "index.js"]
```

**Why:** Dependencies change rarely, code changes frequently. Cache reuse is high.

### Scenario 2: Pre-built Binary
**Best approach:** Single-stage

```dockerfile
FROM ubuntu:24.04
RUN apt-get update && \
    apt-get install -y --no-install-recommends ca-certificates && \
    rm -rf /var/lib/apt/lists/*
COPY my-binary /usr/local/bin/
CMD ["my-binary"]
```

**Why:** Everything changes together. Multi-stage adds complexity without benefit.

### Scenario 3: Compiled Application (Java, Go, Rust)
**Best approach:** Multi-stage with build stage

```dockerfile
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /app
COPY pom.xml ./
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package -DskipTests

FROM eclipse-temurin:21-jre-jammy
COPY --from=build /app/target/*.jar app.jar
CMD ["java", "-jar", "app.jar"]
```

**Why:** Build tools (maven, full JDK) not needed in runtime. Final image much smaller.

## Anti-Patterns to Avoid

**❌ Don't cleanup in separate layer:**
```dockerfile
RUN apt-get update
RUN apt-get install -y foo
RUN rm -rf /var/lib/apt/lists/*  # This doesn't reduce image size!
```

**✅ Do cleanup in same layer:**
```dockerfile
RUN apt-get update && \
    apt-get install -y foo && \
    rm -rf /var/lib/apt/lists/*
```

**❌ Don't copy everything then delete:**
```dockerfile
COPY . .
RUN rm -rf tests/ docs/  # Files still in layer!
```

**✅ Do use .dockerignore:**
```
tests/
docs/
.git/
*.log
```

**❌ Don't use multi-stage when it doesn't help:**
```dockerfile
# Pointless multi-stage
FROM ubuntu AS base
RUN apt-get update && apt-get install -y nodejs

FROM ubuntu AS runtime
COPY --from=base /usr/bin/node /usr/bin/node
# Now you need to copy ALL dependencies manually!
```

## Summary

1. **Always offer** to create both versions when making a Dockerfile
2. **Actually compare** them with real measurements
3. **Recommend honestly** based on data, not assumptions
4. **Explain the tradeoffs** clearly to the user
5. **Simpler is better** when optimization doesn't provide clear benefits

The goal is not to always use multi-stage builds, but to **choose the right approach** for each situation based on actual measurements.
