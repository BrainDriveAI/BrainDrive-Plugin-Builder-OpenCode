---
name: validation-checklist
description: BrainDrive plugin validation checklist and common failure modes - use when verifying plugin correctness or debugging issues
---


## Pre-Build Validation

Run these checks **before** starting the build process:

### Plugin Spec Validation

- [ ] Display name is 1-100 characters
- [ ] Display name doesn't contain special characters (no !, @, #, etc.)
- [ ] Plugin slug is lowercase kebab-case (my-plugin-name)
- [ ] Plugin slug contains only alphanumeric and hyphens
- [ ] Version is semver format (X.Y.Z)
- [ ] Version is not pre-release (not 1.0.0-alpha)
- [ ] Description is one sentence, < 100 chars
- [ ] Category is from approved list
- [ ] Icon name exists in BrainDrive icon library
- [ ] Author name is provided
- [ ] Type is one of: frontend, backend, full

### File Structure Validation

- [ ] `package.json` file exists
- [ ] `lifecycle_manager.py` file exists
- [ ] `webpack.config.js` file exists
- [ ] `tsconfig.json` file exists
- [ ] `src/` folder exists
- [ ] `src/index.tsx` exists and is valid
- [ ] Main component file exists (e.g., `src/MyPlugin.tsx`)
- [ ] `public/` folder exists
- [ ] `LICENSE` file exists
- [ ] `README.md` file exists

### Manifest Validation

- [ ] `package.json` name matches plugin slug
- [ ] `package.json` version matches lifecycle_manager.py
- [ ] `lifecycle_manager.py` plugin_slug matches webpack scope
- [ ] webpack.config.js `name` matches plugin_slug (PascalCase)
- [ ] `bundlelocation` is exactly "dist/remoteEntry.js"
- [ ] React and React-DOM are in dependencies
- [ ] No hardcoded API keys or secrets

### Configuration Validation

```json
{
  "package.json": {
    "name": "should-be-kebab-case",
    "version": "should-be-x.y.z",
    "main": "must-be-dist/remoteEntry.js",
    "dependencies": {
      "react": "must-be-^18.2.0-or-compatible",
      "react-dom": "must-be-^18.2.0-or-compatible"
    }
  },
  "lifecycle_manager.py": {
    "plugin_slug": "should-be-PascalCase",
    "version": "must-match-package.json",
    "bundle_location": "must-be-dist/remoteEntry.js"
  },
  "webpack.config.js": {
    "name": "must-match-plugin_slug"
  }
}
```

## Build-Time Validation

After running `npm install` and `npm run build`:

### Dependency Validation

```bash
# ✅ Check that npm install succeeded
npm list react react-dom typescript webpack

# ❌ Common errors:
# - npm ERR! code ERESOLVE
# - npm ERR! peer dep missing
# FIX: Ensure package.json versions are compatible
```

### TypeScript Validation

```bash
# ✅ Check for TS errors
npx tsc --noEmit

# ❌ Common errors:
# - TS2307: Cannot find module
# - TS2688: Cannot find type definitions
# FIX: Check imports and install @types packages
```

### Webpack Build Validation

```bash
# ✅ Check that webpack built successfully
npm run build 2>&1 | grep -i "error\|success"

# ❌ Common errors:
# - Module not found
# - Unexpected token
# - Cannot find loader
# FIX: Check webpack.config.js and imports
```

### Output Validation

```bash
# ✅ Check that dist/ folder has the right files
ls -la dist/

# Must exist:
# - remoteEntry.js (Module Federation entry)
# - main.js (plugin code)

# ❌ If dist/ is empty:
# FIX: Check webpack.config.js output path
```

## Post-Build Validation

After `npm run build`:

### File Existence Checks

```bash
# ✅ Required files must exist
[ -f dist/remoteEntry.js ] || echo "❌ MISSING: dist/remoteEntry.js"
[ -f package.json ] || echo "❌ MISSING: package.json"
[ -f lifecycle_manager.py ] || echo "❌ MISSING: lifecycle_manager.py"
[ -f webpack.config.js ] || echo "❌ MISSING: webpack.config.js"

# ✅ If all pass:
echo "✓ All required files present"
```

### Bundle Integrity Checks

```bash
# ✅ Check remoteEntry.js is valid JavaScript
node -c dist/remoteEntry.js && echo "✓ remoteEntry.js is valid JS"

# ❌ If error: check webpack.config.js Module Federation settings

# ✅ Check bundle size is reasonable
du -sh dist/
# Typical: 100KB - 500KB

# ❌ If > 2MB: check for large dependencies
find dist/ -size +500k
```

### Manifest Consistency Checks

```bash
# Extract and verify versions match
PACKAGE_VER=$(grep '"version"' package.json | head -1 | cut -d'"' -f4)
LIFECYCLE_VER=$(grep '"version"' lifecycle_manager.py | head -1 | cut -d'"' -f2)

if [ "$PACKAGE_VER" = "$LIFECYCLE_VER" ]; then
  echo "✓ Versions match: $PACKAGE_VER"
else
  echo "❌ VERSION MISMATCH: package=$PACKAGE_VER lifecycle=$LIFECYCLE_VER"
fi

# Extract and verify slugs match
PACKAGE_NAME=$(grep '"name"' package.json | head -1 | cut -d'"' -f4)
PACKAGE_SLUG=$(echo $PACKAGE_NAME | sed 's/-//g' | tr 'a-z' 'A-Z' | cut -c1-1)$(echo $PACKAGE_NAME | cut -d'-' -f1 | cut -c2- | tr 'a-z' 'A-Z')

LIFECYCLE_SLUG=$(grep "plugin_slug" lifecycle_manager.py | head -1 | cut -d'"' -f2)

echo "Package name: $PACKAGE_NAME"
echo "Lifecycle slug: $LIFECYCLE_SLUG"
```

## Common Failure Modes

### Failure Mode 1: Module Not Found

**Error:**
```
Module not found: Error: Can't resolve 'react'
```

**Cause:** React not in dependencies

**Fix:**
```bash
npm install react react-dom
# Update package.json
```

### Failure Mode 2: TypeScript Compilation Error

**Error:**
```
error TS2307: Cannot find module './services/api'
```

**Cause:** Incorrect import path

**Fix:**
```tsx
// Check import path matches file location
// If file is src/services/api.ts:
import { ApiService } from './services/api';  // ✓ Correct
import { ApiService } from '../services/api';  // ✗ Wrong path
```

### Failure Mode 3: Webpack Build Fails

**Error:**
```
ERROR in ./src/index.tsx
Module parse failed: Unexpected token
```

**Cause:** Loader not configured, or syntax error

**Fix:**
```javascript
// webpack.config.js must have ts-loader
module: {
  rules: [
    {
      test: /\.tsx?$/,
      use: 'ts-loader',
      exclude: /node_modules/
    }
  ]
}
```

### Failure Mode 4: remoteEntry.js Not Generated

**Error:**
```
dist/remoteEntry.js NOT FOUND
```

**Cause:** ModuleFederationPlugin not in webpack config

**Fix:**
```javascript
// webpack.config.js
const { ModuleFederationPlugin } = require('webpack').container;

plugins: [
  new ModuleFederationPlugin({
    name: 'YourPlugin',
    filename: 'remoteEntry.js',  // Must be exactly this
    exposes: {
      './index': './src/index.tsx'
    }
  })
]
```

### Failure Mode 5: Version Mismatch

**Error:**
```
Plugin won't load: Version mismatch detected
package.json: 1.0.0
lifecycle_manager.py: 1.0.1
```

**Cause:** Forgot to update both files

**Fix:**
```bash
# Update package.json
"version": "1.0.1"

# Update lifecycle_manager.py
"version": "1.0.1"

# Rebuild
npm run build
```

### Failure Mode 6: Plugin Slug Inconsistency

**Error:**
```
Plugin won't register: scope mismatch
```

**Cause:** Slugs don't match across files

**Fix:**
```
package.json:          "name": "my-plugin"
lifecycle_manager.py:  "plugin_slug": "MyPlugin"
webpack.config.js:     name: 'MyPlugin'

All three must represent the same plugin!
- package.json: kebab-case (my-plugin)
- lifecycle_manager.py: PascalCase (MyPlugin)
- webpack: PascalCase (MyPlugin)
```

### Failure Mode 7: Missing React Peer Dependency

**Error:**
```
npm ERR! code ERESOLVE
npm ERR! ERESOLVE could not resolve dependency peer
```

**Cause:** React version conflict

**Fix:**
```json
{
  "peerDependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  }
}
```

### Failure Mode 8: ZIPfile Too Large

**Error:**
```
ZIP file is 50MB - too large to upload
```

**Cause:** node_modules included in ZIP

**Fix:**
```bash
# Never ZIP node_modules
rm -rf node_modules/
zip -r my-plugin-1.0.0.zip my-plugin/

# Users will run npm install when building
```

### Failure Mode 9: Component Not Exported

**Error:**
```
DashboardPage component not found in bundle
```

**Cause:** Component registered but not exported

**Fix:**
```tsx
// src/index.tsx
export { DashboardPage } from './pages/Dashboard';

// In lifecycle_manager.py
"component": "DashboardPage"  // Must match export name
```

### Failure Mode 10: No Default Page

**Error:**
```
BrainDrive doesn't know which page to load first
```

**Cause:** No page marked as default

**Fix:**
```python
"pages": [
    {
        "slug": "dashboard",
        "title": "Dashboard",
        ...,
        "default": True  # Add this
    },
    { ... }
]
```

## Runtime Validation Tests

After packaging and before release:

### Test 1: ZIP Extraction

```bash
# Create temp dir
mkdir test_plugin
cd test_plugin

# Extract ZIP
unzip ../my-plugin-1.0.0.zip

# Verify structure
ls -la my-plugin/
# Should show: package.json, lifecycle_manager.py, etc.
```

### Test 2: Mock Installation

```bash
cd my-plugin

# Install dependencies
npm install

# Build
npm run build

# Check dist/
ls -la dist/remoteEntry.js
```

### Test 3: Lifecycle Manager Import

```bash
# If Python available
python3 -c "import sys; sys.path.insert(0, '.'); from lifecycle_manager import YourPluginLifecycleManager; print('✓ Lifecycle manager imports successfully')"
```

### Test 4: Bundle Analysis

```bash
# Check what's in remoteEntry.js
file dist/remoteEntry.js

# Should output: JavaScript source

# Check if it's minified (as expected in production)
head -c 100 dist/remoteEntry.js | grep -q "function\|const\|var" && echo "✓ Appears to be minified"
```

## Pre-Release Checklist

Before releasing your plugin:

```
SPEC VALIDATION
- [ ] Plugin name is clear and memorable
- [ ] Description is one sentence and accurate
- [ ] Version follows semver
- [ ] Icon matches plugin purpose
- [ ] Category is appropriate

TECHNICAL VALIDATION
- [ ] All required files exist
- [ ] npm install succeeds
- [ ] npm run build succeeds
- [ ] dist/remoteEntry.js exists and is > 1KB
- [ ] All versions match across files
- [ ] All slugs match across files

FEATURE VALIDATION
- [ ] Main component renders without errors
- [ ] Settings load if defined
- [ ] API calls use correct endpoints
- [ ] No hardcoded secrets or API keys
- [ ] Error handling is present

DOCUMENTATION
- [ ] README.md has installation instructions
- [ ] README.md lists features
- [ ] README.md has troubleshooting section
- [ ] LICENSE file is present
- [ ] CHANGELOG.md documents version history

PACKAGING
- [ ] ZIP file created successfully
- [ ] ZIP file is < 2MB (or documented reason for size)
- [ ] ZIP contains dist/ folder
- [ ] ZIP doesn't contain node_modules
- [ ] ZIP filename matches plugin-version pattern

FINAL CHECKS
- [ ] Plugin installs without errors
- [ ] Plugin loads in BrainDrive
- [ ] All pages render correctly
- [ ] Settings can be modified
- [ ] Plugin uninstalls cleanly
```

---

Validation is your safety net. The more thorough you are here, the fewer user issues you'll face.
