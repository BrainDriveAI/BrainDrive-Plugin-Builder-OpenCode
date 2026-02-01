---
description: Generate file-by-file implementation plan - Phase 3
allowed-tools: Read, Glob, Grep, Write
---

You are the BrainDrive Plugin Builder running Phase 3: Planning.

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
MODE=$(echo "$CONTEXT" | jq -r '.mode')
WORKSPACE_DIR=$(echo "$CONTEXT" | jq -r '.workspace_dir')
PLUGIN_ROOT=$(echo "$CONTEXT" | jq -r '.plugin_root')
PLUGIN_SLUG=$(echo "$CONTEXT" | jq -r '.plugin_slug')
MODULE_NAME=$(echo "$CONTEXT" | jq -r '.module_name')
PLUGIN_TYPE=$(echo "$CONTEXT" | jq -r '.plugin_type')
CURRENT_PHASE=$(echo "$CONTEXT" | jq -r '.phase')

echo "📋 Loaded Context:"
echo "   Mode: $MODE"
echo "   Workspace: $WORKSPACE_DIR"
echo "   Plugin Root: $PLUGIN_ROOT"
echo "   Plugin Slug: $PLUGIN_SLUG"
echo "   Module Name: $MODULE_NAME"
echo "   Plugin Type: $PLUGIN_TYPE"
echo "   Current Phase: $CURRENT_PHASE"
```

### Validate Phase

```bash
if [ "$CURRENT_PHASE" != "map" ]; then
  echo "⚠️  WARNING: Expected phase 'map' but found '$CURRENT_PHASE'"
  echo "Have you run /map yet? (yes/no)"
  # Wait for user confirmation
fi
```

- Phase 1 (Plugin Spec) must be complete
- Phase 2 (Compatibility Map) must be complete
- Context file must exist with phase="map"

## PHASE 3: File-by-File Plan

Based on the Plugin Spec and Compatibility Map, create an explicit plan.

### Step A: Determine Output Location

**DO NOT ask user for location.** Use the plugin_root from context file.

```bash
# Output location is already determined from context
echo "📁 Plugin will be created at: $PLUGIN_ROOT"

# Verify parent directory exists or will be created
PARENT_DIR=$(dirname "$PLUGIN_ROOT")
echo "   Parent directory: $PARENT_DIR"

# Safety check for Mode A: ensure not in backend/shared_plugins
if [ "$MODE" = "A" ] && [[ "$PLUGIN_ROOT" == *"backend/shared_plugins"* ]]; then
  echo "❌ SAFETY CHECK FAILED!"
  echo ""
  echo "Plugin root is inside backend/shared_plugins, which is FORBIDDEN."
  echo "This would overwrite an installed plugin."
  echo ""
  echo "Expected workspace: plugin-build/"
  echo "Actual workspace: $PLUGIN_ROOT"
  echo ""
  echo "Please restart /start to fix workspace configuration."
  exit 1
fi

echo "✅ Workspace location validated"
```

### Step B: Generate File Plan

Create a detailed plan listing every file:

```
=== BUILD PLAN FOR [PLUGIN_NAME] ===

Output Folder: [path]/[plugin-slug]/

Files to Create:

1. package.json
   - Role: Plugin manifest with build scripts and dependencies
   - Contents: name, version, description, main: "dist/remoteEntry.js", webpack scripts

2. lifecycle_manager.py
   - Role: Backend lifecycle hooks (inherits BaseLifecycleManager)
   - Contents: plugin_data dict, module_data list, get_plugin_metadata(), get_module_metadata()

3. webpack.config.js
   - Role: Webpack + Module Federation configuration
   - Contents: Entry point, output to dist/, Module Federation with scope/exposes

4. tsconfig.json
   - Role: TypeScript configuration
   - Contents: ES2020, React JSX, module resolution

5. src/index.tsx
   - Role: Entry point that exports main component
   - Contents: Import and re-export main component

6. src/[PluginName].tsx
   - Role: Main plugin component
   - Contents: [Based on user's features]

7. src/[PluginName].css
   - Role: Plugin styles
   - Contents: Base styles for the plugin

8. src/types.ts
   - Role: TypeScript interfaces
   - Contents: PluginProps and plugin-specific types

9. src/components/ErrorBoundary.tsx
   - Role: Error handling component
   - Contents: React error boundary

10. src/components/LoadingSpinner.tsx
    - Role: Loading state component
    - Contents: Loading spinner UI

11. src/components/ErrorDisplay.tsx
    - Role: Error display component
    - Contents: Error message display

12. src/components/[FeatureComponent].tsx (per feature)
    - Role: Feature-specific components
    - Contents: [One file per major feature]

13. src/services/PluginService.ts (if needed)
    - Role: Business logic and API calls
    - Contents: Service layer for plugin functionality

14. src/utils/errorHandling.ts (if needed)
    - Role: Error handling utilities
    - Contents: Error handling helper functions

15. public/icon.png
    - Role: Plugin icon
    - Contents: Plugin icon image

Build Steps:
1. Create folder structure (src/, public/, src/components/, src/services/, src/utils/)
2. Generate all source files
3. Install dependencies (npm install)
4. Build with webpack (npm run build)
5. Verify dist/remoteEntry.js exists

Verification Steps:
1. Check all required files exist
2. Validate package.json schema
3. Verify lifecycle_manager.py has required functions
4. Confirm frontend builds successfully
5. Test remoteEntry.js is valid

Packaging Steps:
1. Remove node_modules/ folder
2. Keep dist/ folder with remoteEntry.js
3. ZIP entire plugin folder
4. Output: [plugin-slug].zip
```

### Step C: No-Core Fallback Plan

If Mode B (Standalone):
```
Standalone Mode Plan:
- Using bundled Reference Pack templates
- All patterns based on official PluginTemplate
- No live Core validation available
- Will use built-in validation rules
```

### Step D: Generate build-plan.json Artifact

Create an explicit build plan file that Phase 4 must follow:

```bash
# Create plugin directory if it doesn't exist yet (for build-plan.json storage)
mkdir -p "$PLUGIN_ROOT"

# Generate build-plan.json
cat > "$PLUGIN_ROOT/build-plan.json" <<EOF
{
  "plugin_slug": "$PLUGIN_SLUG",
  "module_name": "$MODULE_NAME",
  "plugin_type": "$PLUGIN_TYPE",
  "template_source": "$(jq -r '.template_source' .plugin-builder-context.json)",
  "lifecycle_manager_copy_required": true,
  "theme_bridge_required": $([ "$PLUGIN_TYPE" = "ui_only" ] || [ "$PLUGIN_TYPE" = "ui_plus_service" ] && echo "true" || echo "false"),
  "files_to_create": [
    {
      "path": "package.json",
      "source": "template",
      "required": true,
      "modifications": ["name", "version", "description"]
    },
    {
      "path": "lifecycle_manager.py",
      "source": "copy_from_template",
      "required": true,
      "modifications": ["plugin_data", "module_data"],
      "min_line_count": 900
    },
    {
      "path": "webpack.config.js",
      "source": "template",
      "required": true,
      "modifications": ["PLUGIN_NAME", "PLUGIN_PORT"]
    },
    {
      "path": "tsconfig.json",
      "source": "copy_from_template",
      "required": true,
      "modifications": []
    },
    {
      "path": "src/index.tsx",
      "source": "template",
      "required": true,
      "modifications": ["component_name"]
    },
    {
      "path": "src/$MODULE_NAME.tsx",
      "source": "generate",
      "required": true,
      "component_type": "class",
      "bridges": ["api", "theme", "settings", "event", "pageContext"]
    },
    {
      "path": "src/$MODULE_NAME.css",
      "source": "generate",
      "required": true
    },
    {
      "path": "src/types.ts",
      "source": "template",
      "required": true
    },
    {
      "path": "src/components/ErrorBoundary.tsx",
      "source": "copy_from_template",
      "required": true
    },
    {
      "path": "src/components/LoadingSpinner.tsx",
      "source": "copy_from_template",
      "required": true
    },
    {
      "path": "src/components/ErrorDisplay.tsx",
      "source": "copy_from_template",
      "required": true
    },
    {
      "path": "public/index.html",
      "source": "template",
      "required": true,
      "modifications": ["plugin_display_name"]
    }
  ],
  "bridges_to_include": {
    "api": true,
    "theme": true,
    "settings": true,
    "event": $([ "$PLUGIN_TYPE" = "ui_only" ] || [ "$PLUGIN_TYPE" = "ui_plus_service" ] && echo "true" || echo "false"),
    "pageContext": $([ "$PLUGIN_TYPE" = "ui_only" ] || [ "$PLUGIN_TYPE" = "ui_plus_service" ] && echo "true" || echo "false")
  },
  "success_criteria": [
    "All files in files_to_create exist",
    "lifecycle_manager.py has minimum 900 lines",
    "Theme bridge is included",
    "Module name in lifecycle_manager matches webpack",
    "npm run build succeeds",
    "dist/remoteEntry.js exists and is non-empty"
  ],
  "created_at": "$(date -u +"%Y-%m-%dT%H:%M:%SZ")"
}
EOF

echo "✅ Build plan created at: $PLUGIN_ROOT/build-plan.json"
cat "$PLUGIN_ROOT/build-plan.json" | jq .
```

### Step E: Update Context File

```bash
# Update context with build plan path
jq --arg plan_path "$PLUGIN_ROOT/build-plan.json" \
   '.build_plan_path = $plan_path | .phase = "plan" | .last_updated_at = (now | todate)' \
   .plugin-builder-context.json > .plugin-builder-context.tmp
mv .plugin-builder-context.tmp .plugin-builder-context.json

echo "✅ Context updated: phase = plan, build_plan_path set"
```

### Step F: Request Approval

Display the complete plan and ask:

"This is the build plan. Please review:
- [X] files will be created
- Output location: $PLUGIN_ROOT
- Build commands will run: npm install, npm run build
- Theme bridge: REQUIRED (auto-included)
- Lifecycle manager: COPIED from template (not regenerated)
- Module name: $MODULE_NAME (verified different from slug)

Do you approve this plan? (yes/edit/cancel)"

**IMPORTANT:** Do NOT proceed to build without explicit "yes" approval.

Tell the user: "Phase 3 complete! Build plan artifact saved at: $PLUGIN_ROOT/build-plan.json

Run `/build` to continue to Phase 4 (Build)."
