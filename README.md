# BrainDrive Plugin Builder

**By BrainDrive**

Create production-ready BrainDrive plugins through an interactive guided workflow.

## What is this?

A development tool that generates complete BrainDrive plugins. Answer questions about your plugin, and get production-ready code with all necessary files, lifecycle management, and packaging.

**What you get:**
- Complete plugin structure
- Full lifecycle manager (900+ lines)
- React/TypeScript frontend
- Webpack build configuration
- Ready-to-install ZIP package

## Requirements

- [OpenCode CLI](https://opencode.ai) installed
- Node.js 18+ and npm
- Python 3.8+

## Installation

```bash
# Navigate to an empty working directory
cd ~/my-workspace/

# Clone this repository
git clone https://github.com/BrainDriveAI/BrainDrive-Plugin-Builder-OpenCode.git

# Navigate into it
cd BrainDrive-Plugin-Builder-OpenCode

# Start OpenCode
opencode
```

The `.opencode` folder is automatically included as a hidden directory with all necessary configuration.

## Quick Start

```bash
# Inside the cloned directory
opencode

# Type this to begin:
/start
```

Answer the questions, and your plugin will be built in the `plugin-test/` directory.

## How it works

1. **Run `/start`** - Initialize and begin the interview
2. **Answer questions** - Plugin name, features, UI pages, etc.
3. **Review plan** - The builder shows what it will create
4. **Build automatically** - All files are generated
5. **Get your plugin** - Find it in `plugin-test/your-plugin-name/`

The builder creates a `plugin-test/` directory in your workspace where all generated plugins are saved.

## Available Commands

### Core Commands

| Command | Description |
|---------|-------------|
| `/start` | Initialize and start plugin creation interview |
| `/build` | Generate all plugin files |
| `/verify` | Run validation checks |
| `/package` | Create installation ZIP |
| `/summary` | Show completion summary |

### Utility Commands

| Command | Description |
|---------|-------------|
| `/spec` | Display current plugin specification |
| `/mode` | Show operating mode (A or B) |
| `/plan` | Show detailed implementation plan |
| `/map` | Run compatibility mapping |
| `/help` | Show all commands |
| `/reset` | Clear state and start over |

## Command Usage

### `/start`
Initializes the builder and begins the interactive interview. You'll be asked:
- Plugin name and description
- Target users
- Core features and workflow
- UI pages needed
- Settings requirements
- Data storage needs
- External API integrations

### `/build`
Creates all plugin files based on your specification:
- `package.json` - NPM configuration
- `lifecycle_manager.py` - Plugin lifecycle handler
- `webpack.config.js` - Build configuration
- `tsconfig.json` - TypeScript config
- `src/` - All React components and code

Output: `plugin-test/your-plugin-name/`

### `/verify`
Runs comprehensive validation:
- File structure checks
- Manifest field validation
- Build configuration syntax
- TypeScript compilation readiness
- BrainDrive compatibility
- Naming convention compliance

### `/package`
Creates the final installation package:
- Validates all checks passed
- Creates ZIP with proper structure
- Names: `your-plugin-name-version.zip`

Output: `plugin-test/your-plugin-name.zip`

### `/summary`
Shows what was built, installation instructions, and next steps.

## Output Structure

All generated plugins are saved in the `plugin-test/` directory:

```
plugin-test/
└── your-plugin-name/
    ├── package.json
    ├── lifecycle_manager.py
    ├── webpack.config.js
    ├── tsconfig.json
    ├── src/
    │   ├── index.tsx
    │   ├── YourPluginModule.tsx
    │   └── components/
    ├── public/
    └── your-plugin-name-1.0.0.zip    # Ready to install
```

## Installing in BrainDrive

1. Find the ZIP in `plugin-test/your-plugin-name-version.zip`
2. Open BrainDrive → Plugin Manager
3. Click "Install Plugin"
4. Select the ZIP file
5. Done!

## Operating Modes

**Mode A (Repo-Aware)**
- Detects if BrainDrive-Core is in workspace
- Uses actual PluginTemplate for maximum accuracy

**Mode B (Standalone)**
- Uses bundled reference pack
- Works in any empty directory

Check mode with `/mode` command.

## Example Session

```bash
$ cd BrainDrive-Plugin-Builder-OpenCode
$ opencode

> /start

# Answer interview questions
Plugin name: Task Manager
Description: Manage tasks
...

# Plugin is built automatically
✓ Files created in plugin-test/task-manager/

> /verify
✓ All checks passed

> /package
✓ task-manager-1.0.0.zip created

> /summary
Plugin ready to install!
```

## License

MIT License - Copyright (c) 2026 BrainDrive

---

**Ready to build?** Clone the repo, run `opencode`, and type `/start`
