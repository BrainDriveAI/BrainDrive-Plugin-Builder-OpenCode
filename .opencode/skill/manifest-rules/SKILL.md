---
name: manifest-rules
description: BrainDrive plugin manifest and naming rules - use when validating package.json, naming conventions, or version requirements
---


## Plugin Manifest Overview

The "manifest" is the contract between your plugin and BrainDrive. It's defined in two places:
1. **package.json** (npm metadata)
2. **lifecycle_manager.py** (plugin_data dictionary)

## package.json Rules

### Required Fields

```json
{
  "name": "plugin-name-kebab-case",
  "version": "1.0.0",
  "description": "Short description of plugin",
  "main": "dist/remoteEntry.js",
  "scripts": {
    "build": "webpack --mode production",
    "dev": "webpack --mode development --watch",
    "start": "webpack serve --mode development"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "typescript": "^5.1.6",
    "webpack": "^5.88.2",
    "@types/react": "^18.2.0"
  },
  "peerDependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  }
}
```

### Field Details

| Field | Type | Rules | Example |
|-------|------|-------|---------|
| `name` | string | Must be kebab-case, no spaces, lowercase | `my-awesome-plugin` |
| `version` | string | MUST be semantic versioning (X.Y.Z) | `1.0.0` or `0.1.0` |
| `description` | string | One sentence, < 100 chars | `"Manage daily tasks and reminders"` |
| `main` | string | MUST be exactly `"dist/remoteEntry.js"` | `"dist/remoteEntry.js"` |
| `keywords` | array | Should include "braindrive" and "plugin" | `["braindrive", "plugin", "tasks"]` |
| `author` | string | Name or email | `"John Doe"` or `"john@example.com"` |
| `license` | string | Recommended "MIT" | `"MIT"` |

### Dependency Rules

- **React/React-DOM:** Must be `^18.2.0` or compatible
- **Peer Dependencies:** React and React-DOM should be listed as peer deps
- **No Node.js built-ins:** Don't use `fs`, `path`, etc. in frontend code
- **Third-party packages:** OK to add (lodash, axios, etc.)

### Script Rules

These MUST exist and MUST NOT change:
- `build` → webpack production build
- `dev` → webpack watch mode (optional but recommended)
- `start` → webpack dev server (optional but recommended)

## lifecycle_manager.py Rules

### Plugin Data Dictionary

```python
self.plugin_data = {
    # Identity
    "name": "PluginTemplate",                    # Display name
    "plugin_slug": "PluginTemplate",             # PascalCase slug
    "scope": "PluginTemplate",                   # Module Federation scope
    "version": "1.0.0",                          # MUST match package.json
    "type": "frontend",                          # or "backend" or "full"
    
    # Description
    "description": "Short description",
    "long_description": "Longer description",
    "category": "template",                      # Choose appropriate category
    "icon": "Puzzle",                            # BrainDrive icon name
    
    # Technical
    "bundle_method": "webpack",                  # Must be "webpack"
    "bundle_location": "dist/remoteEntry.js",   # MUST match webpack output
    "compatibility": "1.0.0",                    # BrainDrive compatibility
    "is_local": False,                           # Usually False
    "official": False,                           # True only if official
    
    # Source/Updates
    "source_type": "github",                     # Where plugin code is hosted
    "source_url": "https://github.com/...",      # Link to repo
    "update_check_url": "https://api.github.com/repos/.../releases/latest",
    
    # Author
    "author": "BrainDrive",                      # Your name/org
}
```

### Field Details

| Field | Type | Rules | Example |
|-------|------|-------|---------|
| `name` | string | Display name (can have spaces) | `"My Awesome Plugin"` |
| `plugin_slug` | string | PascalCase, no spaces or special chars | `"MyAwesomePlugin"` |
| `scope` | string | MUST match webpack.config.js scope | `"MyAwesomePlugin"` |
| `version` | string | MUST be semver and match package.json | `"1.0.0"` |
| `type` | string | "frontend", "backend", or "full" | `"frontend"` |
| `description` | string | One-liner | `"Manage tasks and reminders"` |
| `category` | string | Choose from approved list | `"productivity"`, `"chat"`, `"tools"` |
| `icon` | string | BrainDrive icon name | `"Puzzle"`, `"Settings"`, `"Zap"` |
| `bundle_location` | string | MUST be exactly `"dist/remoteEntry.js"` | `"dist/remoteEntry.js"` |
| `compatibility` | string | BrainDrive version (use "1.0.0") | `"1.0.0"` |

## Naming Rules

### Plugin Slug (kebab-case in package.json)

**Format:** Lowercase, hyphens, no spaces, no special characters

```
✅ VALID
- my-plugin
- task-manager
- notes-integration
- data-dashboard-pro

❌ INVALID
- MyPlugin (has capitals)
- my_plugin (uses underscore)
- my plugin (has space)
- my-plugin! (has special char)
- my-plugin-1 (if "1" not semantic version)
```

### Plugin Slug (PascalCase in lifecycle_manager.py)

**Format:** Capitalize each word, no spaces or hyphens

```
✅ VALID
- MyPlugin
- TaskManager
- NotesIntegration
- DataDashboardPro

❌ INVALID
- myPlugin (starts lowercase)
- my-plugin (has hyphens)
- my_plugin (has underscores)
- MYPLUGIN (all caps)
```

### Module Scope (Must Match Exactly)

Webpack scope MUST match the PascalCase slug in lifecycle_manager:

```javascript
// webpack.config.js
const { ModuleFederationPlugin } = require('webpack').container;

new ModuleFederationPlugin({
  name: 'MyAwesomePlugin',      // Must match plugin_slug
  filename: 'remoteEntry.js',
  // ...
})
```

```python
# lifecycle_manager.py
self.plugin_data = {
    "plugin_slug": "MyAwesomePlugin",   # Must match webpack name
    "scope": "MyAwesomePlugin",         # Must match webpack name
    # ...
}
```

## Version Rules

### Semantic Versioning

Format: `MAJOR.MINOR.PATCH` (e.g., `1.0.0`)

| Component | Increment When | Example |
|-----------|----------------|---------|
| MAJOR | Breaking changes | 1.0.0 → 2.0.0 |
| MINOR | New features (backward compatible) | 1.0.0 → 1.1.0 |
| PATCH | Bug fixes | 1.0.0 → 1.0.1 |

### Rules
- Cannot use `1.0`, `1`, or `latest` (must be X.Y.Z format)
- Cannot use `v1.0.0` (no "v" prefix in package.json/lifecycle_manager)
- First version is usually `0.1.0` or `1.0.0`
- Increment in all three places:
  - `package.json` → `"version": "1.0.0"`
  - `lifecycle_manager.py` → `"version": "1.0.0"`
  - `webpack.config.js` comments (optional)

## Icon Rules

Choose from BrainDrive's icon library (Feather icons):

```
Popular choices:
- "Puzzle" (for plugins, extensibility)
- "Box" (for integrations, containers)
- "Zap" (for performance, energy, shortcuts)
- "BarChart3" (for analytics, dashboards)
- "MessageSquare" (for communication, chat)
- "Settings" (for configuration, preferences)
- "FileText" (for documents, notes)
- "CheckCircle" (for tasks, completion)
- "Clock" (for scheduling, time)
- "Users" (for teams, collaboration)
```

## Category Rules

Approved categories:

```
- analytics
- automation
- chat
- collaboration
- content
- data
- dev-tools
- education
- finance
- health
- integration
- knowledge
- marketing
- media
- navigation
- notifications
- organization
- personalization
- productivity
- research
- security
- social
- storage
- template
- tools
- training
- utility
- workspace
```

## Consistency Rules (MUST MATCH)

### package.json ↔ lifecycle_manager.py

```
package.json                              lifecycle_manager.py
├── "name": "my-plugin"         MUST correspond to    plugin_slug: "MyPlugin"
├── "version": "1.0.0"          MUST MATCH           "version": "1.0.0"
└── "description": "..."        MUST MATCH           "description": "..."
```

### webpack.config.js ↔ lifecycle_manager.py

```
webpack.config.js                         lifecycle_manager.py
├── new ModuleFederationPlugin({
│   name: 'MyPlugin'             MUST MATCH           "scope": "MyPlugin"
│   filename: 'remoteEntry.js'   MUST MATCH           "bundle_location": "dist/remoteEntry.js"
```

## Validation Checklist

Before building, verify ALL of these:

- [ ] `package.json` name is kebab-case (lowercase with hyphens)
- [ ] `package.json` version is X.Y.Z semver
- [ ] `lifecycle_manager.py` plugin_slug is PascalCase
- [ ] `lifecycle_manager.py` version matches package.json
- [ ] webpack.config.js scope matches plugin_slug
- [ ] `bundle_location` is exactly `"dist/remoteEntry.js"`
- [ ] Icon name exists in BrainDrive icon library
- [ ] Category is from approved list
- [ ] Author name is filled
- [ ] Description is one sentence, < 100 chars
- [ ] No special characters in slugs (only alphanumeric + hyphens/case)

---

**GOLDEN RULE:** If slugs, versions, or scope don't match exactly, BrainDrive won't find or load your plugin. Always triple-check consistency.
