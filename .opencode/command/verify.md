---
description: Run verification checks - Phase 5 validation
allowed-tools: Read, Glob, Grep, Bash
---

You are the BrainDrive Plugin Builder running Phase 5: Verification.

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
MODULE_NAME=$(echo "$CONTEXT" | jq -r '.module_name')
BUILD_PLAN_PATH=$(echo "$CONTEXT" | jq -r '.build_plan_path')
CURRENT_PHASE=$(echo "$CONTEXT" | jq -r '.phase')

echo "📋 Verifying plugin at: $PLUGIN_ROOT"

# Verify plugin_root exists
if [ ! -d "$PLUGIN_ROOT" ]; then
  echo "❌ FATAL ERROR: Plugin directory not found at $PLUGIN_ROOT"
  echo ""
  echo "Expected directory: $PLUGIN_ROOT"
  echo "This means /build was not run or failed."
  echo ""
  echo "Recommended fix:"
  echo "  1. Run: /build"
  echo "  2. Wait for build to complete"
  echo "  3. Then run: /verify"
  exit 1
fi

# Navigate to plugin root
cd "$PLUGIN_ROOT" || exit 1
echo "✅ Plugin directory found and entered"
```

Phase 4 (Build) must be complete. Context file must exist with phase="build".

## PHASE 5: Verification (Hard-Gated Enforcement)

This is NOT a checklist - this is a GATE SYSTEM.

**FAIL = Workflow stops immediately**
**WARN = Allowed to continue**

### Validation Results System

```bash
# Initialize validation tracking
FAIL_COUNT=0
WARN_COUNT=0
VALIDATION_ERRORS=()
VALIDATION_WARNINGS=()
```

### P0 FAIL Checks (BLOCKING)

These checks MUST pass. Any failure stops the workflow immediately.

#### 1. Context File Validation (P0 FAIL)

```bash
echo "▶ Validating context file..."

# Check context exists
if [ ! -f ../../.plugin-builder-context.json ]; then
  FAIL_COUNT=$((FAIL_COUNT + 1))
  VALIDATION_ERRORS+=("❌ FAIL: Context file missing at repository root")
  VALIDATION_ERRORS+=("")
  VALIDATION_ERRORS+=("   Expected: /.plugin-builder-context.json")
  VALIDATION_ERRORS+=("   Cause: Context file was deleted or never created")
  VALIDATION_ERRORS+=("   Fix: Run /start to create a new build session")
  VALIDATION_ERRORS+=("")
else
  # Validate context matches current plugin_root
  CONTEXT_ROOT=$(jq -r '.plugin_root' ../../.plugin-builder-context.json)
  ACTUAL_ROOT=$(basename $(dirname $(pwd)))/$(basename $(pwd))

  if [ "$CONTEXT_ROOT" != "$ACTUAL_ROOT" ]; then
    FAIL_COUNT=$((FAIL_COUNT + 1))
    VALIDATION_ERRORS+=("❌ FAIL: Context file plugin_root mismatch")
    VALIDATION_ERRORS+=("")
    VALIDATION_ERRORS+=("   Expected root: $CONTEXT_ROOT")
    VALIDATION_ERRORS+=("   Actual root: $ACTUAL_ROOT")
    VALIDATION_ERRORS+=("   Cause: Verification running in wrong directory")
    VALIDATION_ERRORS+=("   Fix: cd to repository root and run /verify again")
    VALIDATION_ERRORS+=("")
  else
    echo "   ✅ PASS: Context file valid and matches plugin_root"
  fi
fi
```

#### 2. Structural Checks (P0 FAIL)

```bash
echo "▶ Validating file structure..."

REQUIRED_FILES=(
  "package.json"
  "lifecycle_manager.py"
  "webpack.config.js"
  "tsconfig.json"
  "src/index.tsx"
  "src/$MODULE_NAME.tsx"
)

for file in "${REQUIRED_FILES[@]}"; do
  if [ ! -f "$file" ]; then
    FAIL_COUNT=$((FAIL_COUNT + 1))
    VALIDATION_ERRORS+=("❌ FAIL: Required file missing: $file")
    VALIDATION_ERRORS+=("")
    VALIDATION_ERRORS+=("   Expected path: $PLUGIN_ROOT/$file")
    VALIDATION_ERRORS+=("   Cause: File was not created during /build")
    VALIDATION_ERRORS+=("   Fix: Run /build again or manually create this file")
    VALIDATION_ERRORS+=("")
  else
    echo "   ✅ PASS: $file exists"
  fi
done
```

#### 3. Lifecycle Manager Line Count (P0 FAIL)

```bash
echo "▶ Validating lifecycle_manager.py line count..."

if [ -f "lifecycle_manager.py" ]; then
  LIFECYCLE_LINES=$(wc -l < lifecycle_manager.py)
  echo "   Line count: $LIFECYCLE_LINES"

  if [ "$LIFECYCLE_LINES" -lt 900 ]; then
    FAIL_COUNT=$((FAIL_COUNT + 1))
    VALIDATION_ERRORS+=("❌ FAIL: Lifecycle manager is too short")
    VALIDATION_ERRORS+=("")
    VALIDATION_ERRORS+=("   Actual lines: $LIFECYCLE_LINES")
    VALIDATION_ERRORS+=("   Minimum required: 900 lines")
    VALIDATION_ERRORS+=("   Cause: File was regenerated from scratch instead of copied from template")
    VALIDATION_ERRORS+=("   This is a CRITICAL architecture violation!")
    VALIDATION_ERRORS+=("")
    VALIDATION_ERRORS+=("   Fix:")
    VALIDATION_ERRORS+=("     1. Delete $PLUGIN_ROOT/lifecycle_manager.py")
    VALIDATION_ERRORS+=("     2. Manually copy from template:")
    VALIDATION_ERRORS+=("        cp $TEMPLATE_SOURCE/lifecycle_manager.py $PLUGIN_ROOT/")
    VALIDATION_ERRORS+=("     3. Edit only: class name, plugin_data, module_data")
    VALIDATION_ERRORS+=("     4. Do NOT regenerate methods or structure")
    VALIDATION_ERRORS+=("     5. Run /verify again")
    VALIDATION_ERRORS+=("")
  else
    echo "   ✅ PASS: Lifecycle manager has $LIFECYCLE_LINES lines (>= 900)"
  fi
fi
```

#### 4. Manifest Validation (P0 FAIL)

```bash
echo "▶ Validating package.json..."

if [ ! -f "package.json" ]; then
  # Already caught in structural checks
  :
else
  # Validate JSON is parseable
  if ! jq . package.json > /dev/null 2>&1; then
    FAIL_COUNT=$((FAIL_COUNT + 1))
    VALIDATION_ERRORS+=("❌ FAIL: package.json is not valid JSON")
    VALIDATION_ERRORS+=("")
    VALIDATION_ERRORS+=("   File: $PLUGIN_ROOT/package.json")
    VALIDATION_ERRORS+=("   Cause: Syntax error in JSON")
    VALIDATION_ERRORS+=("   Fix: Run 'cat package.json | jq .' to see the exact error")
    VALIDATION_ERRORS+=("        Then fix the JSON syntax and run /verify again")
    VALIDATION_ERRORS+=("")
  else
    # Validate required fields
    PKG_NAME=$(jq -r '.name' package.json)
    PKG_VERSION=$(jq -r '.version' package.json)
    PKG_MAIN=$(jq -r '.main' package.json)

    if [ "$PKG_NAME" != "$PLUGIN_SLUG" ]; then
      FAIL_COUNT=$((FAIL_COUNT + 1))
      VALIDATION_ERRORS+=("❌ FAIL: package.json name mismatch")
      VALIDATION_ERRORS+=("")
      VALIDATION_ERRORS+=("   Expected: $PLUGIN_SLUG")
      VALIDATION_ERRORS+=("   Actual: $PKG_NAME")
      VALIDATION_ERRORS+=("   File: $PLUGIN_ROOT/package.json:2")
      VALIDATION_ERRORS+=("   Fix: Change \"name\": \"$PKG_NAME\" to \"name\": \"$PLUGIN_SLUG\"")
      VALIDATION_ERRORS+=("")
    fi

    if [ "$PKG_MAIN" != "dist/remoteEntry.js" ]; then
      FAIL_COUNT=$((FAIL_COUNT + 1))
      VALIDATION_ERRORS+=("❌ FAIL: package.json main field incorrect")
      VALIDATION_ERRORS+=("")
      VALIDATION_ERRORS+=("   Expected: dist/remoteEntry.js")
      VALIDATION_ERRORS+=("   Actual: $PKG_MAIN")
      VALIDATION_ERRORS+=("   File: $PLUGIN_ROOT/package.json")
      VALIDATION_ERRORS+=("   Fix: Change \"main\": \"$PKG_MAIN\" to \"main\": \"dist/remoteEntry.js\"")
      VALIDATION_ERRORS+=("")
    fi

    # Validate React dependencies exact version
    REACT_VERSION=$(jq -r '.dependencies.react' package.json)
    if [ "$REACT_VERSION" != "18.3.1" ]; then
      FAIL_COUNT=$((FAIL_COUNT + 1))
      VALIDATION_ERRORS+=("❌ FAIL: React version must be exactly 18.3.1")
      VALIDATION_ERRORS+=("")
      VALIDATION_ERRORS+=("   Expected: 18.3.1")
      VALIDATION_ERRORS+=("   Actual: $REACT_VERSION")
      VALIDATION_ERRORS+=("   Cause: BrainDrive requires exact React 18.3.1 match")
      VALIDATION_ERRORS+=("   File: $PLUGIN_ROOT/package.json")
      VALIDATION_ERRORS+=("   Fix: Change \"react\": \"$REACT_VERSION\" to \"react\": \"18.3.1\"")
      VALIDATION_ERRORS+=("         Also change \"react-dom\": \"...\" to \"react-dom\": \"18.3.1\"")
      VALIDATION_ERRORS+=("")
    else
      echo "   ✅ PASS: package.json name, main, and React version correct"
    fi
  fi
fi
```

#### 3. Lifecycle Manager Checks
Read lifecycle_manager.py and verify:

```python
Required elements:
- Class inherits from BaseLifecycleManager
- self.plugin_data dictionary with: name, description, version, type, icon, category, scope, bundle_method, bundle_location, plugin_slug
- self.module_data list with at least one entry
- get_plugin_metadata() method
- get_module_metadata() method
- _perform_user_installation() method
- _perform_user_uninstallation() method
- scope in plugin_data must be PascalCase
- bundle_location must be "dist/remoteEntry.js"
```

Check for:
- ✅ Inherits BaseLifecycleManager
- ✅ plugin_data exists and has all required keys
- ✅ module_data list exists and is non-empty
- ✅ scope matches webpack name (PascalCase)
- ✅ bundle_location is "dist/remoteEntry.js"
- ✅ All required methods defined

#### 4. Webpack & TypeScript Checks
Verify configuration:

```bash
# Check webpack and TypeScript configs
cat [plugin-path]/webpack.config.js
cat [plugin-path]/tsconfig.json
```

Validate webpack.config.js:
- ✅ Has ModuleFederationPlugin
- ✅ name matches lifecycle_manager scope (PascalCase)
- ✅ filename is "remoteEntry.js"
- ✅ exposes component correctly
- ✅ shared has react and react-dom as singletons

Validate tsconfig.json:
- ✅ target is ES2020 or higher
- ✅ jsx is "react"
- ✅ includes src/**/*

#### 5. Source File Checks
Verify source structure:

```bash
# Check source files
ls -la [plugin-path]/src/
ls -la [plugin-path]/src/components/
```

Validate:
- ✅ src/index.tsx exports main component
- ✅ src/[PluginName].tsx exists and is valid React
- ✅ src/types.ts has PluginProps interface
- ✅ src/components/ has necessary components

#### 5. TypeScript Configuration (P0 FAIL)

```bash
echo "▶ Validating tsconfig.json..."

if [ ! -f "tsconfig.json" ]; then
  # Already caught in structural checks
  :
else
  if ! jq . tsconfig.json > /dev/null 2>&1; then
    FAIL_COUNT=$((FAIL_COUNT + 1))
    VALIDATION_ERRORS+=("❌ FAIL: tsconfig.json is not valid JSON")
    VALIDATION_ERRORS+=("")
    VALIDATION_ERRORS+=("   File: $PLUGIN_ROOT/tsconfig.json")
    VALIDATION_ERRORS+=("   Fix: Run 'cat tsconfig.json | jq .' to see the exact error")
    VALIDATION_ERRORS+=("")
  else
    echo "   ✅ PASS: tsconfig.json is valid JSON"
  fi
fi
```

#### 6. Build Compilation Check (P0 FAIL)

```bash
echo "▶ Running build to verify compilation..."

# Check if node_modules exists
if [ ! -d "node_modules" ]; then
  echo "   📦 Installing dependencies first..."
  npm install > /tmp/npm-install.log 2>&1

  if [ $? -ne 0 ]; then
    FAIL_COUNT=$((FAIL_COUNT + 1))
    VALIDATION_ERRORS+=("❌ FAIL: npm install failed")
    VALIDATION_ERRORS+=("")
    VALIDATION_ERRORS+=("   Directory: $PLUGIN_ROOT")
    VALIDATION_ERRORS+=("   Log: /tmp/npm-install.log")
    VALIDATION_ERRORS+=("   Fix: Review the log file:")
    VALIDATION_ERRORS+=("        cat /tmp/npm-install.log")
    VALIDATION_ERRORS+=("        Fix any package.json errors and run /verify again")
    VALIDATION_ERRORS+=("")
  else
    echo "   ✅ Dependencies installed"
  fi
fi

# Run build
echo "   🔨 Running npm run build..."
npm run build > /tmp/npm-build.log 2>&1
BUILD_EXIT_CODE=$?

if [ $BUILD_EXIT_CODE -ne 0 ]; then
  FAIL_COUNT=$((FAIL_COUNT + 1))
  VALIDATION_ERRORS+=("❌ FAIL: npm run build failed")
  VALIDATION_ERRORS+=("")
  VALIDATION_ERRORS+=("   Exit code: $BUILD_EXIT_CODE")
  VALIDATION_ERRORS+=("   Directory: $PLUGIN_ROOT")
  VALIDATION_ERRORS+=("   Build log: /tmp/npm-build.log")
  VALIDATION_ERRORS+=("")
  VALIDATION_ERRORS+=("   Common causes:")
  VALIDATION_ERRORS+=("     - TypeScript errors in source files")
  VALIDATION_ERRORS+=("     - Missing imports")
  VALIDATION_ERRORS+=("     - Webpack configuration errors")
  VALIDATION_ERRORS+=("")
  VALIDATION_ERRORS+=("   Fix:")
  VALIDATION_ERRORS+=("     1. Review build log: cat /tmp/npm-build.log")
  VALIDATION_ERRORS+=("     2. Fix TypeScript/webpack errors")
  VALIDATION_ERRORS+=("     3. Run /verify again")
  VALIDATION_ERRORS+=("")

  # Show last 20 lines of build log
  VALIDATION_ERRORS+=("   Last 20 lines of build output:")
  VALIDATION_ERRORS+=("   $(tail -20 /tmp/npm-build.log)")
  VALIDATION_ERRORS+=("")
else
  # Check dist/remoteEntry.js exists and is non-empty
  if [ ! -f "dist/remoteEntry.js" ]; then
    FAIL_COUNT=$((FAIL_COUNT + 1))
    VALIDATION_ERRORS+=("❌ FAIL: dist/remoteEntry.js not created by build")
    VALIDATION_ERRORS+=("")
    VALIDATION_ERRORS+=("   Expected: $PLUGIN_ROOT/dist/remoteEntry.js")
    VALIDATION_ERRORS+=("   Cause: Webpack build completed but didn't output bundle")
    VALIDATION_ERRORS+=("   Fix: Check webpack.config.js output.filename setting")
    VALIDATION_ERRORS+=("")
  else
    BUNDLE_SIZE=$(wc -c < "dist/remoteEntry.js")
    if [ "$BUNDLE_SIZE" -eq 0 ]; then
      FAIL_COUNT=$((FAIL_COUNT + 1))
      VALIDATION_ERRORS+=("❌ FAIL: dist/remoteEntry.js is empty")
      VALIDATION_ERRORS+=("")
      VALIDATION_ERRORS+=("   File: $PLUGIN_ROOT/dist/remoteEntry.js")
      VALIDATION_ERRORS+=("   Size: 0 bytes")
      VALIDATION_ERRORS+=("   Cause: Build produced empty bundle")
      VALIDATION_ERRORS+=("   Fix: Review webpack configuration and rebuild")
      VALIDATION_ERRORS+=("")
    else
      echo "   ✅ PASS: Build succeeded, dist/remoteEntry.js created ($BUNDLE_SIZE bytes)"
    fi
  fi
fi
```

#### 7. Import/Reference Check
Scan for obvious errors:

```bash
# Check for undefined imports
grep -r "from '\./" [plugin-path]/src/
```

Verify:
- ✅ No imports from non-existent files
- ✅ No obvious syntax errors
- ✅ All imported files exist

### P1 WARN Checks (Non-Blocking)

```bash
echo ""
echo "▶ Running P1 WARN checks (non-blocking)..."

# Check for theme bridge usage (warn only for UI plugins)
PLUGIN_TYPE=$(jq -r '.plugin_type' ../../.plugin-builder-context.json)
if [ "$PLUGIN_TYPE" = "ui_only" ] || [ "$PLUGIN_TYPE" = "ui_plus_service" ]; then
  if ! grep -q "theme" "src/$MODULE_NAME.tsx"; then
    WARN_COUNT=$((WARN_COUNT + 1))
    VALIDATION_WARNINGS+=("⚠️  WARN: Theme service not detected in main component")
    VALIDATION_WARNINGS+=("   File: src/$MODULE_NAME.tsx")
    VALIDATION_WARNINGS+=("   Recommendation: Add theme bridge for dark mode support")
    VALIDATION_WARNINGS+=("")
  fi
fi

# Check for error boundary
if [ ! -f "src/components/ErrorBoundary.tsx" ]; then
  WARN_COUNT=$((WARN_COUNT + 1))
  VALIDATION_WARNINGS+=("⚠️  WARN: ErrorBoundary component missing")
  VALIDATION_WARNINGS+=("   Recommended: src/components/ErrorBoundary.tsx")
  VALIDATION_WARNINGS+=("")
fi
```

### Final Validation Report

```bash
echo ""
echo "=================================================================="
echo "                  VERIFICATION REPORT"
echo "=================================================================="
echo ""
echo "Plugin: $PLUGIN_SLUG"
echo "Module Name: $MODULE_NAME"
echo "Date: $(date -u +"%Y-%m-%d %H:%M:%S UTC")"
echo ""
echo "P0 FAIL Checks: $([ $FAIL_COUNT -eq 0 ] && echo "✅ ALL PASSED" || echo "❌ $FAIL_COUNT FAILED")"
echo "P1 WARN Checks: $([ $WARN_COUNT -eq 0 ] && echo "✅ NO WARNINGS" || echo "⚠️  $WARN_COUNT WARNINGS")"
echo ""

# Display all errors
if [ $FAIL_COUNT -gt 0 ]; then
  echo "=================================================================="
  echo "                    ❌ BLOCKING FAILURES"
  echo "=================================================================="
  echo ""
  for error in "${VALIDATION_ERRORS[@]}"; do
    echo "$error"
  done
  echo ""
  echo "=================================================================="
  echo "                  ❌ VERIFICATION FAILED"
  echo "=================================================================="
  echo ""
  echo "WORKFLOW STOPPED: $FAIL_COUNT P0 failures detected."
  echo ""
  echo "You MUST fix all P0 failures before continuing to /package."
  echo "Each error above includes:"
  echo "  - Exact file path"
  echo "  - Root cause"
  echo "  - Prescriptive fix steps"
  echo ""
  echo "After fixing, run /verify again."
  echo ""
  exit 1
fi

# Display warnings
if [ $WARN_COUNT -gt 0 ]; then
  echo "=================================================================="
  echo "                     ⚠️  WARNINGS"
  echo "=================================================================="
  echo ""
  for warning in "${VALIDATION_WARNINGS[@]}"; do
    echo "$warning"
  done
  echo ""
fi

# All passed
echo "=================================================================="
echo "                   ✅ VERIFICATION PASSED"
echo "=================================================================="
echo ""
echo "All P0 checks passed. Plugin is ready for packaging."
echo ""

# Update context
jq '.phase = "verify" | .last_updated_at = (now | todate)' ../../.plugin-builder-context.json > ../../.plugin-builder-context.tmp
mv ../../.plugin-builder-context.tmp ../../.plugin-builder-context.json

# Update lifecycle_manager_line_count in context
if [ -f "lifecycle_manager.py" ]; then
  LIFECYCLE_LINES=$(wc -l < lifecycle_manager.py)
  jq --arg lines "$LIFECYCLE_LINES" \
     '.lifecycle_manager_line_count = ($lines | tonumber)' \
     ../../.plugin-builder-context.json > ../../.plugin-builder-context.tmp
  mv ../../.plugin-builder-context.tmp ../../.plugin-builder-context.json
fi

echo "✅ Context updated: phase = verify"
echo ""
echo "Next step: Run /package to create the distributable ZIP"
```
