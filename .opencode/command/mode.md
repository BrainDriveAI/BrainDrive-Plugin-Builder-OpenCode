---
description: Show current operating mode (A or B)
allowed-tools: Bash, Glob
---

You are the BrainDrive Plugin Builder. Detect and show the current operating mode.

### Mode Detection

**IMPORTANT:** Search ONLY in current directory. Do NOT search outside.

```bash
# Search ONLY in current directory
find . -maxdepth 4 -name "BrainDrive" -type d 2>/dev/null | head -1
find . -maxdepth 5 -name "PluginTemplate" -type d 2>/dev/null | head -1
```

### Display Mode

**If PluginTemplate found in ./BrainDrive/backend/plugins/shared/PluginTemplate:**

```
╔══════════════════════════════════════════════════════════════╗
║                    MODE A: REPO-AWARE                        ║
╚══════════════════════════════════════════════════════════════╝

✅ BrainDrive folder detected in current directory
✅ PluginTemplate found at: ./BrainDrive/backend/plugins/shared/PluginTemplate/v1.0.0/

In this mode, the builder will:
• Read the actual PluginTemplate structure from local BrainDrive
• Use real lifecycle_manager.py as reference
• Match exact webpack configuration from template
• Ensure 100% compatibility with your BrainDrive version

This provides the most accurate plugin generation.

Template Source: ./BrainDrive/backend/plugins/shared/PluginTemplate/v1.0.0/
```

**If PluginTemplate NOT found:**

```
╔══════════════════════════════════════════════════════════════╗
║                   MODE B: STANDALONE                         ║
╚══════════════════════════════════════════════════════════════╝

ℹ️ BrainDrive folder not found in current directory

In this mode, the builder will:
• Use the bundled Reference Pack
• Reference: ./.opencode/reference-pack/PluginTemplate-REAL/
• Apply official BrainDrive plugin patterns
• Generate standard-compliant plugins

This mode is fully functional and creates plugins compatible
with standard BrainDrive installations.

Reference Source: ./.opencode/reference-pack/PluginTemplate-REAL/

To enable Mode A:
• Copy your BrainDrive folder to the current project root
• The builder will automatically detect it and switch to Mode A
```
