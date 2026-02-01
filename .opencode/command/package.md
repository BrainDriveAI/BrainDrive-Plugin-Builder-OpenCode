---
description: Build final ZIP package - Phase 6
allowed-tools: Read, Bash, Glob
---

You are the BrainDrive Plugin Builder running Phase 6: Package.

## Prerequisites

### Load Context File (CRITICAL - RUN FIRST)

```bash
# Check if context file exists
if [ ! -f .plugin-builder-context.json ]; then
  echo "❌ ERROR: Plugin builder context not found."
  echo "Please run: /start"
  exit 1
fi

# Load context
CONTEXT=$(cat .plugin-builder-context.json)
PLUGIN_ROOT=$(echo "$CONTEXT" | jq -r '.plugin_root')
PLUGIN_SLUG=$(echo "$CONTEXT" | jq -r '.plugin_slug')
CURRENT_PHASE=$(echo "$CONTEXT" | jq -r '.phase')

echo "📦 Packaging plugin at: $PLUGIN_ROOT"

# Validate phase
if [ "$CURRENT_PHASE" != "verify" ]; then
  echo "❌ ERROR: Cannot package before verification passes"
  echo ""
  echo "Current phase: $CURRENT_PHASE"
  echo "Expected phase: verify"
  echo ""
  echo "Please run: /verify"
  echo "Wait for ALL P0 checks to PASS"
  echo "Then run: /package"
  exit 1
fi

# Navigate to plugin root
if [ ! -d "$PLUGIN_ROOT" ]; then
  echo "❌ FATAL ERROR: Plugin directory not found at $PLUGIN_ROOT"
  exit 1
fi

echo "✅ Validation passed, ready to package"
```

Phase 5 (Verification) must pass before packaging. Context file must exist with phase="verify".

## PHASE 6: Package Output ZIP

### Pre-packaging Checks

1. Verify Phase 5 passed:
   - If verification failed, tell user to fix issues first
   - Do not package a broken plugin

2. Confirm output location:
   "Where should I save the ZIP file? (default: same folder as plugin)"

### Packaging Steps

#### Step 1: Clean Up Build Artifacts

```bash
# Remove node_modules (not needed in ZIP)
rm -rf [plugin-path]/node_modules

# Keep dist/ folder (required)
ls -la [plugin-path]/dist/
```

#### Step 2: Verify dist/ Exists

```bash
# Check remoteEntry.js exists
ls -la [plugin-path]/dist/remoteEntry.js
```

If dist/ missing:
- Ask user: "dist/ folder not found. Should I run the build first? (yes/no)"
- If yes, run:
  ```bash
  cd [plugin-path] && npm install && npm run build
  ```

#### Step 3: Create ZIP

```bash
# Navigate to parent of plugin folder
cd [plugin-parent-path]

# Create ZIP with plugin folder name
zip -r [plugin-slug].zip [plugin-slug]/ -x "*/node_modules/*" -x "*/.git/*" -x "*.DS_Store"
```

#### Step 4: Verify ZIP

```bash
# Check ZIP was created
ls -la [plugin-slug].zip

# List ZIP contents to verify structure
unzip -l [plugin-slug].zip | head -30
```

#### Step 5: Report

```
=== PACKAGING COMPLETE ===

Plugin: [plugin-name]
Version: [version]

Output ZIP: [full-path-to-zip]
Size: [file size]

ZIP Contents:
[plugin-slug]/
├── package.json
├── lifecycle_manager.py
├── webpack.config.js
├── tsconfig.json
├── src/
│   ├── index.tsx
│   ├── [PluginName].tsx
│   ├── components/
│   ├── services/
│   └── utils/
├── public/
│   └── icon.png
└── dist/
    └── remoteEntry.js

```

### Manual Install Instructions

Display these instructions for the user:

```
=== INSTALLATION INSTRUCTIONS ===

To install this plugin in BrainDrive:

1. Open BrainDrive in your browser
2. Navigate to Settings > Plugins
3. Click "Install Plugin" or the + button
4. Select the ZIP file: [plugin-slug].zip
5. Wait for installation to complete
6. Refresh the page
7. Your plugin should appear in the sidebar

If installation fails:
- Check that BrainDrive is running
- Verify the ZIP structure matches requirements
- Check browser console for errors
- Contact BrainDrive support if issues persist
```

Tell the user: "Phase 6 complete! Run `/braindrive-plugin-builder:summary` for the final summary."
