# Skillbook

A Claude Code marketplace for personal productivity plugins.

## What is Skillbook?

Skillbook is a marketplace that provides Claude Code plugins for software engineering and product development. This marketplace will host multiple plugins in the future, each containing specialized skills, workflows, and tool integrations.

## Plugins

### Skillcraft

Personal toolkit of Claude skills for creating and managing AI skills.

#### Skills

##### skill-seekers

Automates the creation of Claude skills from external sources like documentation websites, GitHub repositories, and PDF files.

**Use when:** The user asks you to create a skill from external sources.

**Prerequisites:** Requires the [skill-seekers CLI tool](https://github.com/yusufkaraaslan/Skill_Seekers#-now-available-on-pypi) to be installed.

**Features:**
- Scrape documentation websites
- Extract content from GitHub repositories
- Process PDF documentation
- AI-powered enhancement of skills
- Package skills for distribution

See [skills/skill-seekers](skills/skill-seekers) for detailed usage instructions.

## Installation

To install this plugin in Claude Code:

```bash
claude plugins install https://github.com/Kauko/skillbook.git
```

Or add to your marketplace configuration.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
