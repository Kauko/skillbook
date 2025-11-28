---
name: optimizing-dockerfiles
description: Use when user wants to create a Dockerfile, reduce image size, improve build caching, or optimize container builds.
requires:
  tools: [docker]
  skills: []
---

# Optimizing Dockerfiles

## Prerequisites

```bash
command -v docker >/dev/null || { echo "Install Docker Desktop or docker CLI"; exit 1; }
```

## Simple Version

```dockerfile
FROM clojure:temurin-21-tools-deps-alpine

WORKDIR /app
COPY deps.edn ./
RUN clojure -P

COPY . .
RUN clojure -T:build uber

CMD ["java", "-jar", "target/app.jar"]
```

## Optimized Version (Multi-Stage)

```dockerfile
# Stage 1: Dependencies (cached)
FROM clojure:temurin-21-tools-deps-alpine AS deps
WORKDIR /app
COPY deps.edn ./
RUN clojure -P

# Stage 2: Build
FROM deps AS build
COPY . .
RUN clojure -T:build uber

# Stage 3: Runtime (minimal)
FROM eclipse-temurin:21-jre-alpine AS runtime
WORKDIR /app
COPY --from=build /app/target/app.jar ./app.jar
EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```

## Key Principles

### Layer Ordering
```dockerfile
# GOOD: Least → most frequently changed
COPY deps.edn ./       # Changes rarely
RUN clojure -P         # Cached when deps.edn unchanged
COPY src/ ./src/       # Changes frequently
```

### Multi-Stage Benefits
1. **Smaller images**: Runtime image has no build tools
2. **Better caching**: Deps stage cached separately
3. **Security**: Fewer packages in production image

### Cleanup in Same Layer
```dockerfile
# GOOD
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*

# BAD (cleanup in separate layer doesn't reduce size)
RUN apt-get update && apt-get install -y curl
RUN rm -rf /var/lib/apt/lists/*
```

## Compare Results

```bash
# Build both
docker build -t app:simple -f Dockerfile .
docker build -t app:optimized -f Dockerfile.optimized .

# Compare sizes
docker images | grep app
```

## Clojure-Specific Tips

**Cache dependencies:**
```dockerfile
COPY deps.edn ./
RUN clojure -P -M:dev:test  # Download all deps
```

**Use Alpine base:**
```dockerfile
FROM clojure:temurin-21-tools-deps-alpine  # vs debian
```

**JRE for runtime:**
```dockerfile
FROM eclipse-temurin:21-jre-alpine  # Not JDK
```

## Success Criteria

- [ ] Dockerfile builds successfully
- [ ] Multi-stage build produces smaller runtime image
- [ ] `docker images | grep app` shows size reduction vs simple build
