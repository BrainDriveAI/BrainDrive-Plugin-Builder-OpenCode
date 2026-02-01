---
name: plugin-anatomy
description: Understanding BrainDrive plugin structure and architecture - use when explaining what a BrainDrive plugin is, how it works, or when planning plugin structure
---


## What is a BrainDrive Plugin?

A BrainDrive plugin is a modular software package that extends BrainDrive's functionality. Users install it via the Plugin Installer UI, and it runs inside the BrainDrive application.

### Not a Standalone App
- NOT a web server
- NOT a desktop application
- NOT a library you import elsewhere
- IS a self-contained package that BrainDrive loads dynamically

## Plugin Architecture

### High-Level Structure
```
BrainDrive Application
├── Core Backend (Python)
├── Core Frontend (React)
├── Plugin Lifecycle Manager
├── Plugin 1 (Your Plugin)
│   ├── Frontend Bundle (Module Federation)
│   ├── Settings Schema
│   ├── Backend Hooks (optional)
│   └── Lifecycle Manager
├── Plugin 2
└── Plugin N
```

### The Three Layers

#### Layer 1: Frontend Bundle
- **Technology:** React + TypeScript
- **Build:** Webpack with Module Federation (exposes remoteEntry.js)
- **Loading:** BrainDrive loads remoteEntry.js at runtime
- **Export:** Main component and other shareable modules
- **Output:** `dist/remoteEntry.js`

#### Layer 2: Backend (Optional)
- **Technology:** Python (Flask/async)
- **Purpose:** API endpoints, data processing, external integrations
- **Lifecycle Manager:** Handles install/uninstall per user
- **Database Access:** Via BrainDrive ORM if needed
- **File:** `lifecycle_manager.py`

#### Layer 3: Metadata & Registration
- **Plugin Manifest:** `package.json` (name, version, scope, etc.)
- **Lifecycle Data:** `lifecycle_manager.py` (plugin_data, modules)
- **Settings Schema:** JSON schema for user settings
- **UI Registration:** Which pages/routes to mount

## Folder Structure (Real PluginTemplate)

```
PluginTemplate/
├── v1.0.0/                          # Version folder
│   ├── package.json                 # npm metadata
│   ├── tsconfig.json               # TypeScript config
│   ├── webpack.config.js           # Build config (Module Federation)
│   ├── lifecycle_manager.py        # Backend lifecycle hooks
│   ├── src/
│   │   ├── index.tsx               # Webpack entry point (exports factory)
│   │   ├── PluginTemplate.tsx      # Main component
│   │   ├── PluginTemplate.css      # Styles
│   │   ├── types.ts                # TypeScript interfaces
│   │   ├── components/
│   │   │   ├── ErrorBoundary.tsx
│   │   │   ├── ErrorDisplay.tsx
│   │   │   └── LoadingSpinner.tsx
│   │   ├── services/
│   │   │   ├── api.ts              # BrainDrive API bridge
│   │   │   └── errorHandler.ts
│   │   └── utils/
│   │       ├── validators.ts
│   │       └── formatters.ts
│   ├── public/
│   │   └── icon.png                # Plugin icon (optional)
│   ├── dist/                       # Build output (generated)
│   │   ├── remoteEntry.js         # Module Federation entry
│   │   └── ...other chunks...
│   ├── README.md                   # Developer guide
│   └── DEVELOPMENT.md              # Setup instructions
```

## Key Concepts

### Module Federation
- **What:** Webpack feature that allows loading code at runtime
- **Why:** BrainDrive loads plugins dynamically without bundling them into Core
- **How:** Plugin exposes modules via `remoteEntry.js`, BrainDrive loads and executes
- **Output file:** Must be `dist/remoteEntry.js`

### Plugin Scope
- **Definition:** A unique namespace for the plugin (e.g., "MyPlugin")
- **Location:** In webpack.config.js and lifecycle_manager.py
- **Purpose:** Prevents namespace collisions between plugins
- **Format:** PascalCase (e.g., "DataDashboard", "NotesIntegration")

### Lifecycle Manager
- **File:** `lifecycle_manager.py`
- **Purpose:** Handles install/uninstall/update per user
- **Inherits from:** `BaseLifecycleManager` class
- **Provides:** Plugin metadata, module definitions, user installation logic

### Plugin Metadata (package.json)
```json
{
  "name": "plugin-template",
  "version": "1.0.0",
  "description": "What this plugin does",
  "main": "dist/remoteEntry.js",
  "scripts": {
    "build": "webpack --mode production",
    "dev": "webpack --mode development --watch",
    "start": "webpack serve --mode development"
  }
}
```

### Plugin Data (lifecycle_manager.py)
```python
self.plugin_data = {
    "name": "PluginTemplate",
    "description": "Description here",
    "version": "1.0.0",
    "type": "frontend",                # or "backend" or "full"
    "icon": "Puzzle",                  # BrainDrive icon name
    "category": "template",
    "plugin_slug": "PluginTemplate",
    "scope": "PluginTemplate",         # Must match webpack config
    "bundle_method": "webpack",
    "bundle_location": "dist/remoteEntry.js",
    "long_description": "...",
    "author": "...",
    "compatibility": "1.0.0",
}
```

## Component Export Pattern

### How BrainDrive Loads Your Plugin

1. **BrainDrive fetches:** `dist/remoteEntry.js`
2. **BrainDrive calls:** `window[scope].init(shareScope, initScope)`
3. **BrainDrive gets factory:** `window[scope].default()`
4. **BrainDrive renders:** Main React component

### Your src/index.tsx Must Export Factory Function

```tsx
// This is what BrainDrive calls
export default () => PluginTemplate;

// OR with async loading
export default async () => {
  const { default: PluginTemplate } = await import('./PluginTemplate');
  return PluginTemplate;
};
```

### Your Main Component (PluginTemplate.tsx)

```tsx
import React from 'react';

interface Props {
  // BrainDrive passes these props
  apiUrl?: string;
  userId?: string;
  settings?: Record<string, any>;
}

export const PluginTemplate: React.FC<Props> = (props) => {
  return (
    <div className="plugin-container">
      <h1>Plugin Loaded!</h1>
      <p>Plugin can now render any UI.</p>
    </div>
  );
};

export default PluginTemplate;
```

## Common Plugin Types

### Type 1: UI-Only Plugin
- **Frontend:** React component with UI
- **Backend:** None
- **Storage:** Uses BrainDrive's plugin state storage
- **Example:** Dashboard, note viewer, task manager

### Type 2: UI + Backend
- **Frontend:** React component with UI
- **Backend:** Python endpoints
- **Storage:** Database tables via lifecycle_manager
- **Example:** Chat plugin, data sync plugin, external API integrator

### Type 3: Headless Backend Plugin
- **Frontend:** None (or minimal settings UI)
- **Backend:** Python service with async tasks
- **Storage:** Database tables, plugin state
- **Example:** Scheduler, background sync, notification service

### Type 4: Settings Plugin
- **Frontend:** Settings form only
- **Backend:** Optional config management
- **Storage:** Plugin settings
- **Example:** Theme customizer, integration config manager

## Bridges Available

Plugins can communicate with BrainDrive via "bridges":

### API Bridge
- **What:** HTTP endpoint to call BrainDrive backend APIs
- **Usage:** Fetch conversations, messages, user data
- **Configuration:** API URL + authentication

### Settings Bridge
- **What:** Store and retrieve plugin-specific settings
- **Usage:** Per-user plugin configuration
- **Format:** JSON schema + instance values

### Theme Bridge
- **What:** Access to BrainDrive theme (colors, fonts)
- **Usage:** Consistent UI appearance
- **Provides:** CSS variables or theme object

### Event Bridge
- **What:** Subscribe to BrainDrive events
- **Usage:** React to app events (conversations opened, etc.)
- **Examples:** onConversationCreated, onMessageReceived

### Plugin State Bridge
- **What:** Persistent key-value storage
- **Usage:** Save plugin state across sessions
- **Access:** Per-user and global scope

## Typical User Journey

1. **User downloads plugin ZIP** from plugin marketplace or creator
2. **User opens BrainDrive**
3. **User navigates to Settings > Plugins > Install**
4. **User selects ZIP file**
5. **BrainDrive installs plugin** (runs lifecycle_manager.py:install_for_user)
6. **Plugin appears in UI** (pages registered, settings available)
7. **User configures plugin** (if settings defined)
8. **User uses plugin** (interacts with component)
9. **Plugin persists data** (settings, state, backend data)
10. **User uninstalls** (cleanup via uninstall_for_user)

## Plugin Versioning

- **Semver format:** MAJOR.MINOR.PATCH (e.g., 1.0.0)
- **Stored in:** package.json and lifecycle_manager.py
- **Must match** in both files
- **Increments:**
  - MAJOR: Breaking changes
  - MINOR: New features (backward compatible)
  - PATCH: Bug fixes

---

This is the anatomical foundation. When building a plugin, every file and concept above must be present or considered.
