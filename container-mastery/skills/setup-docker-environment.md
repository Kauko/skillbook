---
name: setup-docker-environment
description: Use when creating new software projects - adds Claude Code Docker setup as git submodule for containerized development
---

# Setup Docker Environment

## When to Use This Skill

Use this skill when:
- User asks you to create a new software project or repository
- User asks to add Docker support to an existing project
- User mentions wanting to run Claude Code in a container
- User asks about containerized development setup

## The Process

### 1. Check if Project is a Git Repository

Before adding the Docker setup, verify the project is initialized as a git repository:

```bash
# Check if .git directory exists
if [ -d .git ]; then
    echo "Git repository found"
else
    echo "Initializing git repository..."
    git init
fi
```

### 2. Add Docker Setup as Git Submodule

Add the claude-docker-setup repository as a git submodule in the `.docker` directory:

```bash
git submodule add git@github.com:teemukaukoranta/claude-docker-setup.git .docker
```

**Why use `.docker` directory:**
- Keeps Docker files organized and separate from project files
- No pollution of project root with Docker configuration
- Git tracks the exact version of the Docker setup
- Easy to update independently of project code

### 3. Configure the Environment

Run the setup script to create environment configuration:

```bash
cd .docker
./setup-docker.sh
```

This auto-detects macOS settings (UID 501, GID 20) and creates a `.env` file.

### 4. Add API Key

The user needs to add their Anthropic API key to the `.env` file:

```bash
echo "ANTHROPIC_API_KEY=your-key-here" >> .docker/.env
```

**Important:** Tell the user they need to add their actual API key.

### 5. Update .gitignore

Add the submodule's `.env` file to the project's `.gitignore` to prevent committing secrets:

```bash
# Add to .gitignore
echo "" >> .gitignore
echo "# Claude Docker environment" >> .gitignore
echo ".docker/.env" >> .gitignore
```

**Why:**
- The `.docker` directory itself is tracked (as a submodule reference)
- But `.docker/.env` contains secrets and should not be committed
- The submodule has its own `.env.example` for reference

### 6. Build the Docker Image

Build the Claude Code Docker image:

```bash
docker-compose -f .docker/docker-compose.yml build
```

This creates a container with:
- Node 20.18.1
- Claude Code v2.0.50
- M1 Mac optimization (ARM64)
- Security hardening

### 7. Provide Usage Instructions

Tell the user how to run Claude Code in Docker:

**Interactive session:**
```bash
docker-compose -f .docker/docker-compose.yml run --rm claude
```

**One-off task:**
```bash
docker-compose -f .docker/docker-compose.yml run --rm task "analyze this codebase"
```

**Manual exploration:**
```bash
docker-compose -f .docker/docker-compose.yml run --rm bash
```

### 8. Optional: Create Convenience Aliases

Suggest creating aliases for easier usage:

**For Fish shell** (`~/.config/fish/config.fish`):
```fish
alias claude-docker="docker-compose -f .docker/docker-compose.yml run --rm claude"
alias claude-docker-build="docker-compose -f .docker/docker-compose.yml build"
```

**For Bash/Zsh** (`~/.bashrc` or `~/.zshrc`):
```bash
alias claude-docker="docker-compose -f .docker/docker-compose.yml run --rm claude"
alias claude-docker-build="docker-compose -f .docker/docker-compose.yml build"
```

## Docker Setup Features

The containerized Claude Code environment includes:

**Security:**
- Container isolation
- User mapping (runs as your UID/GID to prevent permission issues)
- Minimal Linux capabilities
- Resource limits (4 CPU, 4GB RAM)
- Read-only plugin directory

**Performance:**
- M1/M2/M3 optimized (ARM64 platform)
- `:delegated` volume mounts for macOS
- Layer caching for fast rebuilds

**Access:**
- Full read-write access to project directory
- Read-only access to `~/.claude` plugins and configuration
- Git configuration for making commits inside container

## Updating the Docker Setup

When improvements are made to the Docker setup, update the submodule:

```bash
git submodule update --remote .docker
cd .docker
./setup-docker.sh  # Regenerate .env if needed
docker-compose -f .docker/docker-compose.yml build --no-cache

# Commit the update
git add .docker
git commit -m "Update Docker setup to latest version"
```

## Troubleshooting

### Submodule Already Exists
If the submodule addition fails because it already exists:
```bash
git submodule update --init .docker
```

### Permission Issues
Run the setup script to regenerate `.env` with correct UID/GID:
```bash
cd .docker && ./setup-docker.sh
```

### API Key Not Set
Check that `ANTHROPIC_API_KEY` is set in `.docker/.env`:
```bash
cat .docker/.env | grep ANTHROPIC_API_KEY
```

### Docker Build Fails
Try a clean rebuild:
```bash
docker-compose -f .docker/docker-compose.yml build --no-cache
```

## Complete Example

Here's what the complete setup looks like:

```bash
# 1. Create or navigate to project
cd ~/code/my-project
git init

# 2. Add Docker setup
git submodule add git@github.com:teemukaukoranta/claude-docker-setup.git .docker
cd .docker
./setup-docker.sh

# 3. Add API key
echo "ANTHROPIC_API_KEY=sk-ant-..." >> .env

# 4. Update .gitignore
echo ".docker/.env" >> ../.gitignore

# 5. Build and run
cd ..
docker-compose -f .docker/docker-compose.yml build
docker-compose -f .docker/docker-compose.yml run --rm claude
```

## Project Structure After Setup

```
my-project/
├── .git/
├── .gitignore              # Contains .docker/.env
├── .docker/                # Submodule → claude-docker-setup
│   ├── Dockerfile
│   ├── docker-compose.yml
│   ├── .dockerignore
│   ├── setup-docker.sh
│   ├── .env.example
│   ├── .env                # User's secrets (gitignored)
│   └── README.docker.md
├── src/                    # Your project files
└── README.md
```

## Key Principles

1. **Use Git Submodules** - Version-locked, updateable, no file duplication
2. **Separate Configuration** - `.docker/` directory keeps things organized
3. **Protect Secrets** - Always gitignore `.docker/.env`
4. **Provide Clear Instructions** - Tell user about API key and usage commands
5. **Offer Aliases** - Make the long docker-compose commands easier to use

## Summary

When setting up a new project:
1. ✅ Verify git repository exists (or create one)
2. ✅ Add claude-docker-setup as submodule to `.docker/`
3. ✅ Run setup script
4. ✅ Instruct user to add API key
5. ✅ Update `.gitignore` to exclude `.docker/.env`
6. ✅ Build Docker image
7. ✅ Provide usage commands and optional aliases
8. ✅ Explain how to update the setup later

This gives the user a production-ready, secure, containerized Claude Code environment that's easy to maintain and update.
