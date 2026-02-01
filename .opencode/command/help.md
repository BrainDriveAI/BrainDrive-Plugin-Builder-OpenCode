---
description: Show all available BrainDrive Plugin Builder commands
---

You are the BrainDrive Plugin Builder. Display the help information:

```
╔══════════════════════════════════════════════════════════════╗
║           BRAINDRIVE PLUGIN BUILDER - HELP                   ║
╚══════════════════════════════════════════════════════════════╝

WORKFLOW COMMANDS (run in order)
────────────────────────────────

/braindrive-plugin-builder:start
  Phase 0-1: Introduction and plugin interview
  Collects all information needed for your plugin

/braindrive-plugin-builder:map
  Phase 2: Compatibility mapping
  Determines BrainDrive plugin requirements

/braindrive-plugin-builder:plan
  Phase 3: File-by-file plan
  Shows exactly what will be created

/braindrive-plugin-builder:build
  Phase 4: Generate plugin files
  Creates all plugin files (requires approval)

/braindrive-plugin-builder:verify
  Phase 5: Verification checks
  Validates the plugin structure and code

/braindrive-plugin-builder:package
  Phase 6: Create ZIP package
  Packages plugin for installation

/braindrive-plugin-builder:summary
  Phase 7: Final summary
  Shows what was built and how to use it

UTILITY COMMANDS
────────────────

/braindrive-plugin-builder:help
  Show this help message

/braindrive-plugin-builder:spec
  Display current plugin specification

/braindrive-plugin-builder:reset
  Clear all state and start over

/braindrive-plugin-builder:mode
  Show current operating mode (A or B)

TYPICAL WORKFLOW
────────────────

1. /braindrive-plugin-builder:start  → Answer interview questions
2. /braindrive-plugin-builder:map    → See compatibility requirements
3. /braindrive-plugin-builder:plan   → Review and approve plan
4. /braindrive-plugin-builder:build  → Generate files
5. /braindrive-plugin-builder:verify → Check for issues
6. /braindrive-plugin-builder:package → Create ZIP
7. /braindrive-plugin-builder:summary → See final report

SCOPE LIMITATIONS
─────────────────
This builder ONLY creates BrainDrive plugins.
It cannot build web apps, scripts, or other projects.

SAFETY RULES
────────────
• Never deletes files without permission
• Never modifies BrainDrive-Core
• All output goes to a new folder only
• Shows plan before any file changes

Need help? Just ask!
```
