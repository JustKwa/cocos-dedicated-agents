# Cocos Dedicated Agents

A collection of AI agent configurations for Cocos Creator game development. These agents provide specialized assistance for working with Cocos Creator projects, including code generation, debugging, and technical guidance.

## What This Is

This project contains pre-configured AI agent settings that help you work with Cocos Creator game projects. Think of it as a set of specialized AI assistants, each with different expertise (coding, debugging, architecture guidance, etc.).

## Setup

### Prerequisites

- [OpenCode CLI](https://opencode.ai) installed
- Node.js and Bun (for running the agent system)
- `uv` installed. [Installation Guide](https://docs.astral.sh/uv/getting-started/installation/)

### Quick Command (For Tech-Savvy Users)

```bash
mkdir -p ~/<path-to-your-project>/.opencode
cp -r ~/<path-to-cocos-dedicated-agents>/* ~/<path-to-your-project>/.opencode/
```

### Manual Setup (Step-by-Step)

1. **Open File Manager**
2. Navigate to cocos-dedicated-agents folder
3. Select and copy all files in this folder
4. Navigate to your project folder
5. Create a new folder called `.opencode` (note the dot at the start)
6. Paste the copied files into this new `.opencode` folder

## What's Inside

- **cocos-director**: Manages project workflows and orchestrates tasks
- **cocos-sage**: Provides technical expertise and guidance on Cocos Creator
- **explorer**: Helps navigate and understand codebase structure
- **fixer**: Assists with debugging and fixing code issues
- **oracle**: Offers strategic technical advice and architecture guidance

## Usage

Once set up in `~/<path-to-your-project>/.opencode`, these agents will be available when you run OpenCode in any Cocos Creator project directory.
