---
description: Create plugin files - Phase 4 generation
allowed-tools: Read, Write, Edit, Bash, Glob
---

You are the BrainDrive Plugin Builder running Phase 4: Build.

## Prerequisites

### Load Context File and Build Plan (CRITICAL - RUN FIRST)

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
TEMPLATE_SOURCE=$(echo "$CONTEXT" | jq -r '.template_source')
BUILD_PLAN_PATH=$(echo "$CONTEXT" | jq -r '.build_plan_path')
CURRENT_PHASE=$(echo "$CONTEXT" | jq -r '.phase')

echo "📋 Loaded Context:"
echo "   Mode: $MODE"
echo "   Plugin Root: $PLUGIN_ROOT"
echo "   Plugin Slug: $PLUGIN_SLUG"
echo "   Module Name: $MODULE_NAME"
echo "   Template Source: $TEMPLATE_SOURCE"
echo "   Build Plan: $BUILD_PLAN_PATH"

# Load build plan
if [ ! -f "$BUILD_PLAN_PATH" ]; then
  echo "❌ ERROR: Build plan not found at $BUILD_PLAN_PATH"
  echo "Please run: /plan"
  exit 1
fi

BUILD_PLAN=$(cat "$BUILD_PLAN_PATH")
echo "✅ Build plan loaded successfully"
```

### Validate Phase and Approval

```bash
if [ "$CURRENT_PHASE" != "plan" ]; then
  echo "⚠️  WARNING: Expected phase 'plan' but found '$CURRENT_PHASE'"
  echo "Have you run /plan and approved it? (yes/no)"
  # Wait for user confirmation
fi
```

- Phase 1-3 must be complete
- User must have approved the plan in Phase 3
- Context file must exist with phase="plan"
- Build plan JSON must exist

## PHASE 4: Build (Generation Only)

### ⚠️ CRITICAL ARCHITECTURE RULES

**BrainDrive plugins MUST use CLASS COMPONENTS - NO HOOKS!**

❌ **DO NOT USE:**
- Functional components with hooks (useState, useEffect, etc.)
- React.FC pattern
- Any React hooks whatsoever

✅ **MUST USE:**
- Class components extending React.Component
- this.state and this.setState() for state management
- Lifecycle methods (componentDidMount, componentDidUpdate, etc.)

**Why:** BrainDrive's Module Federation setup with `eager: true` requires class components. Hooks don't work properly in federated modules. All working BrainDrive plugins (BrainDriveChat, BrainDriveOllama) use class components.

### ⚠️ CRITICAL NAMING RULE

**Module name in lifecycle_manager.py MUST EXACTLY match webpack expose name!**

Example:
- webpack.config.js: `exposes: { "./Alphaone": "./src/index" }`
- lifecycle_manager.py: `"name": "Alphaone"` ✅ CORRECT
- lifecycle_manager.py: `"name": "ComponentAlphaone"` ❌ WRONG - will fail to load!

The module name is used by BrainDrive to locate the federated module. Mismatch = plugin won't load.

### CRITICAL FIRST STEP: Read the Real PluginTemplate

**BEFORE generating ANY files**, you MUST read the actual working PluginTemplate to use as your base:

**If Mode A (repo-aware):**
```bash
# Check if template exists
ls ./BrainDrive/backend/plugins/shared/PluginTemplate/v1.0.0/
```
If found, READ these files from the template:
- `lifecycle_manager.py` - Use this as the base for Step C
- `webpack.config.js` - Use this as the base for Step D
- `package.json` - Use this as the base for Step B
- `tsconfig.json` - Use this as the base for Step E
- `src/index.tsx` - Use this as the base for Step F
- `public/index.html` - Use this for Step J

**If Mode B (standalone):**
```bash
# Use bundled template
ls ./.opencode/reference-pack/PluginTemplate-REAL/
```
READ the same files from this location.

**YOU MUST READ THESE FILES FIRST** - DO NOT use the hardcoded templates below. The templates below are FALLBACK ONLY if the real template cannot be found.

### Safety Check
Before ANY file operation, confirm:
1. Output folder is NOT inside BrainDrive-Core
2. Output folder is the approved location from Phase 3
3. No existing files will be overwritten without permission

### Step A: Create Plugin Folder Structure

```bash
mkdir -p [output-path]/[plugin-slug]
mkdir -p [output-path]/[plugin-slug]/src/components
mkdir -p [output-path]/[plugin-slug]/src/services
mkdir -p [output-path]/[plugin-slug]/src/utils
mkdir -p [output-path]/[plugin-slug]/public
```

### Step B: Generate package.json

**IMPORTANT:** First READ package.json from PluginTemplate, then adapt it to the user's plugin spec.

The template below is FALLBACK ONLY:

```json
{
  "name": "[plugin-slug]",
  "version": "[version]",
  "description": "[description]",
  "main": "dist/remoteEntry.js",
  "author": "[from spec or ask]",
  "license": "MIT",
  "scripts": {
    "build": "webpack --mode production",
    "dev": "webpack --mode development --watch"
  },
  "dependencies": {
    "react": "18.3.1",
    "react-dom": "18.3.1"
  },
  "devDependencies": {
    "@types/react": "^18.2.0",
    "@types/react-dom": "^18.2.0",
    "css-loader": "^6.8.1",
    "html-webpack-plugin": "^5.5.3",
    "style-loader": "^3.3.3",
    "ts-loader": "^9.4.4",
    "typescript": "^5.1.6",
    "webpack": "^5.88.2",
    "webpack-cli": "^5.1.4",
    "webpack-dev-server": "^4.15.1"
  },
  "peerDependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  }
}
```

### Step C: Copy and Adapt lifecycle_manager.py (NO REGENERATION)

**CRITICAL RULE:** lifecycle_manager.py MUST be COPIED from template, NOT regenerated from scratch.

This is the ONLY way to prevent truncation. Claude has a systematic behavior of producing truncated lifecycle managers when generating from scratch.

#### Process:

1. **READ the complete lifecycle_manager.py from template**
   ```bash
   # Read the full template file
   cat "$TEMPLATE_SOURCE/lifecycle_manager.py"
   ```

2. **COPY it entirely to the new plugin**
   ```bash
   # Copy the entire file first
   cp "$TEMPLATE_SOURCE/lifecycle_manager.py" "$PLUGIN_ROOT/lifecycle_manager.py"
   echo "✅ Copied lifecycle_manager.py from template ($(wc -l < "$PLUGIN_ROOT/lifecycle_manager.py") lines)"
   ```

3. **Then apply MINIMAL edits** using sed or targeted replacements:
   - Replace class name: `PluginTemplateLifecycleManager` → `${MODULE_NAME}LifecycleManager`
   - Replace plugin_data fields: name, description, version, scope, plugin_slug
   - Replace module_data fields: name, display_name, description
   - Update required_services based on plugin_type
   - DO NOT rewrite methods, DO NOT regenerate structure

4. **Verify line count after edits**
   ```bash
   LIFECYCLE_LINES=$(wc -l < "$PLUGIN_ROOT/lifecycle_manager.py")
   echo "Lifecycle manager line count: $LIFECYCLE_LINES"

   if [ "$LIFECYCLE_LINES" -lt 900 ]; then
     echo "❌ CRITICAL ERROR: Lifecycle manager is too short ($LIFECYCLE_LINES lines < 900)"
     echo "This indicates the file was truncated or regenerated instead of copied."
     echo "Aborting build. Please report this issue."
     exit 1
   fi
   ```

**WHY THIS MATTERS:**
- Template lifecycle_manager is 1025+ lines
- Claude systematically produces ~400 line versions when generating from scratch
- This is NOT a one-off mistake - it's consistent model behavior
- Copying + minimal edits is the ONLY reliable solution

The fallback template below should NEVER be used (it will trigger the line count validation failure):

```python
"""
[Plugin Display Name] - BrainDrive Plugin
[Description]
"""

import json
import logging
import datetime
import os
import shutil
import asyncio
import uuid
from pathlib import Path
from typing import Dict, Any, Optional, List
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import text
import structlog

logger = structlog.get_logger()

try:
    # Try to import from the BrainDrive system first (when running in production)
    from app.plugins.base_lifecycle_manager import BaseLifecycleManager
    logger.info("[Plugin]: Using BaseLifecycleManager from app.plugins")
except ImportError:
    try:
        # Fallback for development/testing inside BrainDrive repo
        import sys
        current_dir = os.path.dirname(os.path.abspath(__file__))
        backend_path = os.path.join(current_dir, "..", "..", "..", "..", "app", "plugins")
        backend_path = os.path.abspath(backend_path)

        if os.path.exists(backend_path):
            if backend_path not in sys.path:
                sys.path.insert(0, backend_path)
            from base_lifecycle_manager import BaseLifecycleManager
            logger.info(f"[Plugin]: Using BaseLifecycleManager from: {backend_path}")
        else:
            # Minimal implementation for remote installations
            logger.warning(f"[Plugin]: BaseLifecycleManager not found, using minimal implementation")
            from abc import ABC, abstractmethod
            from datetime import datetime
            from pathlib import Path
            from typing import Set

            class BaseLifecycleManager(ABC):
                """Minimal base class for remote installations"""
                def __init__(self, plugin_slug: str, version: str, shared_storage_path: Path):
                    self.plugin_slug = plugin_slug
                    self.version = version
                    self.shared_path = shared_storage_path
                    self.active_users: Set[str] = set()
                    self.instance_id = f"{plugin_slug}_{version}"
                    self.created_at = datetime.now()
                    self.last_used = datetime.now()

                async def install_for_user(self, user_id: str, db, shared_plugin_path: Path):
                    if user_id in self.active_users:
                        return {'success': False, 'error': 'Plugin already installed for user'}
                    result = await self._perform_user_installation(user_id, db, shared_plugin_path)
                    if result['success']:
                        self.active_users.add(user_id)
                        self.last_used = datetime.now()
                    return result

                async def uninstall_for_user(self, user_id: str, db):
                    if user_id not in self.active_users:
                        return {'success': False, 'error': 'Plugin not installed for user'}
                    result = await self._perform_user_uninstallation(user_id, db)
                    if result['success']:
                        self.active_users.discard(user_id)
                        self.last_used = datetime.now()
                    return result

                @abstractmethod
                async def get_plugin_metadata(self): pass
                @abstractmethod
                async def get_module_metadata(self): pass
                @abstractmethod
                async def _perform_user_installation(self, user_id, db, shared_plugin_path): pass
                @abstractmethod
                async def _perform_user_uninstallation(self, user_id, db): pass

            logger.info("[Plugin]: Using minimal BaseLifecycleManager implementation")

    except ImportError as e:
        logger.error(f"[Plugin]: Failed to import BaseLifecycleManager: {e}")
        raise ImportError("[Plugin] requires BaseLifecycleManager")

class [PluginName]LifecycleManager(BaseLifecycleManager):
    def __init__(self, plugins_base_dir: str = None):
        """Initialize the lifecycle manager"""
        # Plugin-specific data
        self.plugin_data = {
            "name": "[Plugin Display Name]",
            "description": "[description]",
            "version": "[version]",
            "type": "frontend",
            "icon": "[icon-name]",
            "category": "[category]",
            "official": False,
            "author": "[author-name]",
            "compatibility": "1.0.0",
            "scope": "[PluginNamePascalCase]",  # Must match webpack name (PascalCase)
            "bundle_method": "webpack",
            "bundle_location": "dist/remoteEntry.js",
            "is_local": False,
            "long_description": "[long_description]",
            "plugin_slug": "[PluginNamePascalCase]",  # PascalCase, matches scope
            "source_type": "local",
            "source_url": f"file://{os.path.abspath(__file__)}",
            "update_check_url": "",
            "last_update_check": None,
            "update_available": False,
            "latest_version": None,
            "installation_type": "local",
            "permissions": ["storage.read", "storage.write", "api.access"]
        }

        # Modules exported by the plugin
        self.module_data = [
            {
                "name": "[PluginNamePascalCase]Module",  # CRITICAL: Must match webpack expose name EXACTLY! Always add "Module" suffix to differ from plugin_slug
                "display_name": "[Plugin Display Name] Module",    # Human-readable name
                "description": "[description]",
                "icon": "[icon-name]",
                "category": "[category]",
                "priority": 1,
                "props": {},
                "config_fields": {},
                "messages": {"sends": [], "receives": []},
                "required_services": {
                    "api": {
                        "methods": ["get", "post"],
                        "version": "1.0.0"
                    },
                    "theme": {
                        "methods": ["getCurrentTheme"],
                        "version": "1.0.0"
                    },
                    "settings": {
                        "methods": ["getSetting", "setSetting"],
                        "version": "1.0.0"
                    }
                },
                "dependencies": [],
                "layout": {
                    "minWidth": 4,
                    "minHeight": 3,
                    "defaultWidth": 6,
                    "defaultHeight": 4
                },
                "tags": ["[category]"]
            }
        ]

        # Initialize base class with required parameters
        if plugins_base_dir:
            shared_path = Path(plugins_base_dir) / "shared" / self.plugin_data['plugin_slug'] / f"v{self.plugin_data['version']}"
        else:
            shared_path = Path(__file__).parent

        super().__init__(
            plugin_slug=self.plugin_data['plugin_slug'],
            version=self.plugin_data['version'],
            shared_storage_path=shared_path
        )

    @property
    def PLUGIN_DATA(self):
        """Compatibility property for remote installer validation"""
        return self.plugin_data

    async def get_plugin_metadata(self) -> Dict[str, Any]:
        """Return plugin metadata"""
        return self.plugin_data

    async def get_module_metadata(self) -> list:
        """Return module metadata"""
        return self.module_data

    async def _perform_user_installation(self, user_id: str, db: AsyncSession, shared_plugin_path: Path) -> Dict[str, Any]:
        """Perform user-specific installation using shared plugin path"""
        try:
            # 1. Create database records first
            db_result = await self._create_database_records(user_id, db)
            if not db_result['success']:
                return db_result

            # 2. Create plugin page SEPARATELY
            page_result = await self._create_plugin_page(user_id, db, db_result['modules_created'])
            if not page_result.get('success'):
                # ROLLBACK: Delete plugin records if page creation fails
                plugin_id = db_result.get('plugin_id')
                if plugin_id:
                    await self._delete_database_records(user_id, plugin_id, db)
                return page_result

            logger.info(f"[Plugin]: User installation completed for {user_id}")
            return {
                'success': True,
                'plugin_id': db_result['plugin_id'],
                'plugin_slug': self.plugin_data['plugin_slug'],
                'plugin_name': self.plugin_data['name'],
                'modules_created': db_result['modules_created'],
                'page_id': page_result.get('page_id'),
                'page_created': page_result.get('created', False)
            }

        except Exception as e:
            logger.error(f"[Plugin]: User installation failed for {user_id}: {e}")
            return {'success': False, 'error': str(e)}

    async def _perform_user_uninstallation(self, user_id: str, db: AsyncSession) -> Dict[str, Any]:
        """Perform user-specific uninstallation"""
        try:
            # Check if plugin exists for user
            existing_check = await self._check_existing_plugin(user_id, db)
            if not existing_check['exists']:
                return {'success': False, 'error': 'Plugin not found for user'}

            plugin_id = existing_check['plugin_id']

            # Delete plugin page first
            page_result = await self._delete_plugin_page(user_id, db)
            if not page_result.get('success'):
                return page_result

            # Delete database records
            delete_result = await self._delete_database_records(user_id, plugin_id, db)
            if not delete_result['success']:
                return delete_result

            logger.info(f"[Plugin]: User uninstallation completed for {user_id}")
            return {
                'success': True,
                'plugin_id': plugin_id,
                'deleted_modules': delete_result['deleted_modules'],
                'page_deleted': page_result.get('deleted_rows', 0) > 0
            }

        except Exception as e:
            logger.error(f"[Plugin]: User uninstallation failed for {user_id}: {e}")
            return {'success': False, 'error': str(e)}

    async def _check_existing_plugin(self, user_id: str, db: AsyncSession) -> Dict[str, Any]:
        """Check if plugin already exists for user"""
        try:
            check_stmt = text("""
            SELECT id FROM plugin
            WHERE user_id = :user_id AND plugin_slug = :plugin_slug
            """)

            result = await db.execute(check_stmt, {
                "user_id": user_id,
                "plugin_slug": self.plugin_data['plugin_slug']
            })

            existing_plugin = result.fetchone()

            if existing_plugin:
                return {'exists': True, 'plugin_id': existing_plugin[0]}
            else:
                return {'exists': False}

        except Exception as e:
            return {'exists': False, 'error': str(e)}

    async def _create_plugin_page(self, user_id: str, db: AsyncSession, modules_created: List[str]) -> Dict[str, Any]:
        """Create a page in BrainDrive for this plugin

        This automatically creates a page where users can access the plugin.
        The page will have the plugin's module(s) pre-configured in the layout.
        """
        try:
            # Check if page already exists
            plugin_route = f"{self.plugin_data['plugin_slug']}"
            check_stmt = text("""
                SELECT id FROM pages
                WHERE creator_id = :user_id AND route = :route
            """)
            existing_result = await db.execute(check_stmt, {
                "user_id": user_id,
                "route": plugin_route
            })
            existing = existing_result.fetchone()

            if existing:
                existing_page_id = existing.id if hasattr(existing, "id") else existing[0]
                logger.info(f"[Plugin]: Page already exists for {user_id}", page_id=existing_page_id)
                return {"success": True, "page_id": existing_page_id, "created": False}

            # Find the module ID
            module_id = None
            for mid in modules_created:
                if mid.endswith(f"_{self.module_data[0]['name']}"):
                    module_id = mid
                    break

            if not module_id:
                # Fallback query
                module_stmt = text("""
                    SELECT id FROM module
                    WHERE user_id = :user_id AND plugin_id = :plugin_id AND name = :name
                """)
                plugin_id = f"{user_id}_{self.plugin_data['plugin_slug']}"
                module_result = await db.execute(module_stmt, {
                    "user_id": user_id,
                    "plugin_id": plugin_id,
                    "name": self.module_data[0]['name']
                })
                module_row = module_result.fetchone()
                if module_row:
                    module_id = module_row.id if hasattr(module_row, "id") else module_row[0]

            if not module_id:
                logger.error(f"[Plugin]: Failed to resolve module ID for {user_id}")
                return {"success": False, "error": "Unable to resolve module ID"}

            # Create page content with responsive layouts
            timestamp_ms = int(datetime.datetime.utcnow().timestamp() * 1000)
            layout_id = f"{self.plugin_data['plugin_slug']}_{module_id}_{timestamp_ms}"

            content = {
                "layouts": {
                    "desktop": [{
                        "i": layout_id,
                        "x": 0,
                        "y": 0,
                        "w": self.module_data[0].get('layout', {}).get('defaultWidth', 12),
                        "h": self.module_data[0].get('layout', {}).get('defaultHeight', 8),
                        "pluginId": self.plugin_data["plugin_slug"],
                        "args": {
                            "moduleId": module_id,
                            "displayName": self.module_data[0]['display_name']
                        }
                    }],
                    "tablet": [{
                        "i": layout_id,
                        "x": 0,
                        "y": 0,
                        "w": min(self.module_data[0].get('layout', {}).get('defaultWidth', 12), 8),
                        "h": self.module_data[0].get('layout', {}).get('defaultHeight', 8),
                        "pluginId": self.plugin_data["plugin_slug"],
                        "args": {
                            "moduleId": module_id,
                            "displayName": self.module_data[0]['display_name']
                        }
                    }],
                    "mobile": [{
                        "i": layout_id,
                        "x": 0,
                        "y": 0,
                        "w": 4,
                        "h": min(self.module_data[0].get('layout', {}).get('defaultHeight', 8), 6),
                        "pluginId": self.plugin_data["plugin_slug"],
                        "args": {
                            "moduleId": module_id,
                            "displayName": self.module_data[0]['display_name']
                        }
                    }]
                },
                "modules": {}
            }

            # Insert page with correct schema
            page_id = uuid.uuid4().hex
            now = datetime.datetime.utcnow().strftime("%Y-%m-%d %H:%M:%S")

            insert_stmt = text("""
                INSERT INTO pages (
                    id, name, route, content, creator_id,
                    created_at, updated_at, is_published, publish_date
                ) VALUES (
                    :id, :name, :route, :content, :creator_id,
                    :created_at, :updated_at, :is_published, :publish_date
                )
            """)

            await db.execute(insert_stmt, {
                "id": page_id,
                "name": self.plugin_data['name'],
                "route": plugin_route,
                "content": json.dumps(content),
                "creator_id": user_id,
                "created_at": now,
                "updated_at": now,
                "is_published": 1,
                "publish_date": now
            })

            await db.commit()
            logger.info(f"[Plugin]: Created page for {user_id}", page_id=page_id)
            return {"success": True, "page_id": page_id, "created": True}

        except Exception as e:
            await db.rollback()
            logger.error(f"[Plugin]: Failed to create page for {user_id}: {e}")
            return {"success": False, "error": str(e)}

    async def _delete_plugin_page(self, user_id: str, db: AsyncSession) -> Dict[str, Any]:
        """Delete the plugin page"""
        try:
            plugin_route = f"{self.plugin_data['plugin_slug']}"
            delete_stmt = text("""
                DELETE FROM pages
                WHERE creator_id = :user_id AND route = :route
            """)
            result = await db.execute(delete_stmt, {
                "user_id": user_id,
                "route": plugin_route
            })
            await db.commit()
            logger.info(f"[Plugin]: Deleted page for {user_id}", deleted_rows=result.rowcount)
            return {"success": True, "deleted_rows": result.rowcount}

        except Exception as e:
            await db.rollback()
            logger.error(f"[Plugin]: Failed to delete page for {user_id}: {e}")
            return {"success": False, "error": str(e)}

    async def _create_database_records(self, user_id: str, db: AsyncSession) -> Dict[str, Any]:
        """Create plugin and module records in database"""
        try:
            # Check if plugin already exists
            existing_check = await self._check_existing_plugin(user_id, db)
            if existing_check['exists']:
                return {'success': False, 'error': 'Plugin already exists for user'}

            # Create plugin record
            plugin_id = f"{user_id}_{self.plugin_data['plugin_slug']}"
            current_time = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")

            plugin_stmt = text("""
            INSERT INTO plugin
            (id, name, description, version, type, enabled, icon, category, status,
            official, author, last_updated, compatibility, downloads, scope,
            bundle_method, bundle_location, is_local, long_description,
            config_fields, messages, dependencies, created_at, updated_at, user_id, plugin_slug,
            source_type, source_url, update_check_url, last_update_check, update_available,
            latest_version, installation_type, permissions)
            VALUES
            (:id, :name, :description, :version, :type, :enabled, :icon, :category,
            :status, :official, :author, :last_updated, :compatibility, :downloads,
            :scope, :bundle_method, :bundle_location, :is_local, :long_description,
            :config_fields, :messages, :dependencies, :created_at, :updated_at, :user_id, :plugin_slug,
            :source_type, :source_url, :update_check_url, :last_update_check, :update_available,
            :latest_version, :installation_type, :permissions)
            """)

            await db.execute(plugin_stmt, {
                "id": plugin_id,
                "name": self.plugin_data["name"],
                "description": self.plugin_data["description"],
                "version": self.plugin_data["version"],
                "type": self.plugin_data["type"],
                "enabled": True,
                "icon": self.plugin_data.get("icon"),
                "category": self.plugin_data.get("category"),
                "status": "activated",
                "official": self.plugin_data.get("official", False),
                "author": self.plugin_data.get("author"),
                "last_updated": current_time,
                "compatibility": self.plugin_data.get("compatibility", "1.0.0"),
                "downloads": 0,
                "scope": self.plugin_data.get("scope"),
                "bundle_method": self.plugin_data.get("bundle_method"),
                "bundle_location": self.plugin_data.get("bundle_location"),
                "is_local": self.plugin_data.get("is_local", False),
                "long_description": self.plugin_data.get("long_description"),
                "config_fields": None,
                "messages": None,
                "dependencies": None,
                "created_at": current_time,
                "updated_at": current_time,
                "user_id": user_id,
                "plugin_slug": self.plugin_data.get("plugin_slug"),
                "source_type": self.plugin_data.get("source_type"),
                "source_url": self.plugin_data.get("source_url"),
                "update_check_url": self.plugin_data.get("update_check_url"),
                "last_update_check": self.plugin_data.get("last_update_check"),
                "update_available": self.plugin_data.get("update_available", False),
                "latest_version": self.plugin_data.get("latest_version"),
                "installation_type": self.plugin_data.get("installation_type"),
                "permissions": json.dumps(self.plugin_data.get("permissions", []))
            })

            # Create module records
            modules_created = []
            for module_data in self.module_data:
                module_id = f"{user_id}_{module_data['name']}"

                module_stmt = text("""
                INSERT INTO module
                (id, plugin_id, name, display_name, description, icon, category,
                enabled, priority, props, config_fields, messages, required_services,
                dependencies, layout, tags, created_at, updated_at, user_id)
                VALUES
                (:id, :plugin_id, :name, :display_name, :description, :icon, :category,
                :enabled, :priority, :props, :config_fields, :messages, :required_services,
                :dependencies, :layout, :tags, :created_at, :updated_at, :user_id)
                """)

                await db.execute(module_stmt, {
                    "id": module_id,
                    "plugin_id": plugin_id,
                    "name": module_data["name"],
                    "display_name": module_data.get("display_name"),
                    "description": module_data.get("description"),
                    "icon": module_data.get("icon"),
                    "category": module_data.get("category"),
                    "enabled": True,
                    "priority": module_data.get("priority", 1),
                    "props": json.dumps(module_data.get("props", {})),
                    "config_fields": json.dumps(module_data.get("config_fields", {})),
                    "messages": json.dumps(module_data.get("messages", {})),
                    "required_services": json.dumps(module_data.get("required_services", {})),
                    "dependencies": json.dumps(module_data.get("dependencies", [])),
                    "layout": json.dumps(module_data.get("layout", {})),
                    "tags": json.dumps(module_data.get("tags", [])),
                    "created_at": current_time,
                    "updated_at": current_time,
                    "user_id": user_id
                })

                modules_created.append(module_id)

            await db.commit()

            # Verify
            verify_query = text("SELECT id FROM plugin WHERE id = :plugin_id AND user_id = :user_id")
            verify_result = await db.execute(verify_query, {'plugin_id': plugin_id, 'user_id': user_id})
            verify_row = verify_result.fetchone()

            if verify_row:
                logger.info(f"[Plugin]: Created records for {plugin_id}")
            else:
                return {'success': False, 'error': 'Plugin creation verification failed'}

            return {'success': True, 'plugin_id': plugin_id, 'modules_created': modules_created}

        except Exception as e:
            logger.error(f"[Plugin]: Error creating database records: {e}")
            await db.rollback()
            return {'success': False, 'error': str(e)}

    async def _delete_database_records(self, user_id: str, plugin_id: str, db: AsyncSession) -> Dict[str, Any]:
        """Delete plugin and module records from database"""
        try:
            # Delete modules first (foreign key constraint)
            module_stmt = text("""
            DELETE FROM module
            WHERE user_id = :user_id AND plugin_id = :plugin_id
            """)

            module_result = await db.execute(module_stmt, {
                "user_id": user_id,
                "plugin_id": plugin_id
            })

            deleted_modules = module_result.rowcount

            # Delete plugin
            plugin_stmt = text("""
            DELETE FROM plugin
            WHERE user_id = :user_id AND id = :plugin_id
            """)

            plugin_result = await db.execute(plugin_stmt, {
                "user_id": user_id,
                "plugin_id": plugin_id
            })

            if plugin_result.rowcount == 0:
                await db.rollback()
                return {'success': False, 'error': 'Plugin not found'}

            await db.commit()

            logger.info(f"[Plugin]: Deleted records for {plugin_id}")
            return {'success': True, 'deleted_modules': deleted_modules}

        except Exception as e:
            logger.error(f"[Plugin]: Error deleting records: {e}")
            await db.rollback()
            return {'success': False, 'error': str(e)}

    async def _copy_plugin_files_impl(self, user_id: str, target_dir: Path, update: bool = False) -> Dict[str, Any]:
        """Copy plugin files from source to target directory"""
        try:
            import shutil
            source_dir = Path(__file__).parent
            copied_files = []

            # Define files and directories to exclude
            exclude_patterns = {
                'node_modules',
                'package-lock.json',
                '.git',
                '.gitignore',
                '__pycache__',
                '*.pyc',
                '.DS_Store',
                'Thumbs.db'
            }

            def should_copy(path: Path) -> bool:
                """Check if a file/directory should be copied"""
                for part in path.parts:
                    if part in exclude_patterns:
                        return False
                for pattern in exclude_patterns:
                    if '*' in pattern and path.name.endswith(pattern.replace('*', '')):
                        return False
                return True

            # Copy all files and directories recursively
            for item in source_dir.rglob('*'):
                # Skip the lifecycle_manager.py file itself
                if item.name == 'lifecycle_manager.py' and item == Path(__file__):
                    continue

                relative_path = item.relative_to(source_dir)

                if not should_copy(relative_path):
                    continue

                target_path = target_dir / relative_path

                try:
                    if item.is_file():
                        target_path.parent.mkdir(parents=True, exist_ok=True)
                        if update and target_path.exists():
                            target_path.unlink()
                        shutil.copy2(item, target_path)
                        copied_files.append(str(relative_path))
                    elif item.is_dir():
                        target_path.mkdir(parents=True, exist_ok=True)
                except Exception as e:
                    continue

            # Copy lifecycle_manager.py itself
            lifecycle_manager_source = source_dir / 'lifecycle_manager.py'
            lifecycle_manager_target = target_dir / 'lifecycle_manager.py'
            if lifecycle_manager_source.exists():
                lifecycle_manager_target.parent.mkdir(parents=True, exist_ok=True)
                if update and lifecycle_manager_target.exists():
                    lifecycle_manager_target.unlink()
                shutil.copy2(lifecycle_manager_source, lifecycle_manager_target)
                copied_files.append('lifecycle_manager.py')

            return {'success': True, 'copied_files': copied_files}

        except Exception as e:
            return {'success': False, 'error': str(e)}

    async def install_plugin(self, user_id: str, db: AsyncSession) -> Dict[str, Any]:
        """Install plugin for specific user"""
        try:
            # Check if plugin already exists
            existing_check = await self._check_existing_plugin(user_id, db)
            if existing_check['exists']:
                return {
                    'success': False,
                    'error': 'Plugin already installed for user',
                    'plugin_id': existing_check['plugin_id']
                }

            # Create shared directory
            shared_path = self.shared_path
            shared_path.mkdir(parents=True, exist_ok=True)

            # Copy plugin files to shared directory FIRST
            copy_result = await self._copy_plugin_files_impl(user_id, shared_path)
            if not copy_result['success']:
                return copy_result

            # Install for user (creates database records)
            result = await self.install_for_user(user_id, db, shared_path)

            if result.get('success'):
                result.update({
                    'shared_path': str(shared_path),
                    'files_copied': copy_result.get('copied_files', [])
                })
                await db.commit()
                return result
            else:
                await db.rollback()
                return result

        except Exception as e:
            await db.rollback()
            return {'success': False, 'error': str(e)}

    async def delete_plugin(self, user_id: str, db: AsyncSession) -> Dict[str, Any]:
        """Delete plugin for user"""
        try:
            # Check if plugin exists
            existing_check = await self._check_existing_plugin(user_id, db)
            if not existing_check['exists']:
                return {'success': False, 'error': 'Plugin not found for user'}

            plugin_id = existing_check['plugin_id']

            # Delete database records
            delete_result = await self._delete_database_records(user_id, plugin_id, db)
            if delete_result['success']:
                await db.commit()

            return delete_result

        except Exception as e:
            await db.rollback()
            return {'success': False, 'error': str(e)}
```

### Step D: Generate Webpack Config

**IMPORTANT:** First READ webpack.config.js from PluginTemplate, then adapt it.

The template below is FALLBACK ONLY:

**webpack.config.js:**
```javascript
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const { ModuleFederationPlugin } = require("webpack").container;
const deps = require("./package.json").dependencies;

const PLUGIN_NAME = "[PluginNamePascalCase]";
const PLUGIN_PORT = 3003;

module.exports = {
  mode: "production",
  entry: "./src/index",
  output: {
    path: path.resolve(__dirname, 'dist'),
    publicPath: "auto",
    clean: true,
    library: {
      type: 'var',
      name: PLUGIN_NAME
    }
  },
  resolve: {
    extensions: [".tsx", ".ts", ".js"],
  },
  module: {
    rules: [
      {
        test: /\.(ts|tsx)$/,
        use: "ts-loader",
        exclude: /node_modules/,
      },
      {
        test: /\.css$/,
        use: [
          'style-loader',
          'css-loader'
        ]
      }
    ],
  },
  plugins: [
    new ModuleFederationPlugin({
      name: PLUGIN_NAME,
      library: { type: "var", name: PLUGIN_NAME },
      filename: "remoteEntry.js",
      exposes: {
        [`./${PLUGIN_NAME}`]: "./src/index"
      },
      shared: {
        react: {
          singleton: true,
          requiredVersion: deps.react,
          eager: true
        },
        "react-dom": {
          singleton: true,
          requiredVersion: deps["react-dom"],
          eager: true
        }
      }
    }),
    new HtmlWebpackPlugin({
      template: "./public/index.html",
    }),
  ],
  devServer: {
    port: PLUGIN_PORT,
    static: {
      directory: path.join(__dirname, "public"),
    },
    hot: true,
  },
};
```

### Step E: Generate TypeScript Config

**IMPORTANT:** First READ tsconfig.json from PluginTemplate, then adapt it.

The template below is FALLBACK ONLY:

**tsconfig.json:**
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM"],
    "jsx": "react",
    "module": "ESNext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "allowJs": true,
    "noEmit": true,
    "isolatedModules": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### Step F: Generate Source Files

**IMPORTANT:** First READ src/index.tsx from PluginTemplate to understand the export pattern and development mode setup, then adapt it.

The template below is FALLBACK ONLY:

**src/index.tsx:**
```typescript
import React from 'react';
import [PluginNamePascalCase] from './[PluginNamePascalCase]';

// Simple export - NO development rendering, NO ReactDOM
export default [PluginNamePascalCase];

// Version info for BrainDrive
export const version = '[version]';
```

**src/[PluginNamePascalCase].tsx:**

**MANDATORY THEME BRIDGE:** For UI plugins (ui_only, ui_plus_service), theme service MUST be included and used.

```typescript
import React from 'react';
import './[PluginNamePascalCase].css';

interface PluginProps {
  services?: {
    api?: any;
    theme?: any;
    settings?: any;
    event?: any;
    pageContext?: any;
  };
  // Add plugin-specific props
}

interface PluginState {
  currentTheme: 'light' | 'dark';
  // Add plugin-specific state
}

// CRITICAL: Use CLASS COMPONENT - NO HOOKS!
class [PluginNamePascalCase] extends React.Component<PluginProps, PluginState> {
  private themeCleanup?: () => void;

  constructor(props: PluginProps) {
    super(props);
    this.state = {
      currentTheme: props.services?.theme?.getCurrentTheme() || 'light',
      // Initialize state here
    };
  }

  componentDidMount() {
    // MANDATORY: Subscribe to theme changes
    const { theme } = this.props.services || {};
    if (theme) {
      this.themeCleanup = theme.addThemeChangeListener((newTheme: 'light' | 'dark') => {
        this.setState({ currentTheme: newTheme });
      });
    }

    // Other initialization logic
  }

  componentWillUnmount() {
    // MANDATORY: Cleanup theme listener
    if (this.themeCleanup) {
      this.themeCleanup();
    }

    // Other cleanup logic
  }

  render() {
    const { currentTheme } = this.state;

    return (
      <div
        className={`[plugin-slug]-container theme-${currentTheme}`}
        data-theme={currentTheme}
      >
        {/* Plugin UI */}
      </div>
    );
  }
}

export default [PluginNamePascalCase];
```

**CSS must include theme-aware styles:**
```css
/* Light theme (default) */
.[plugin-slug]-container.theme-light {
  background-color: var(--bg-primary, #ffffff);
  color: var(--text-primary, #000000);
}

/* Dark theme */
.[plugin-slug]-container.theme-dark {
  background-color: var(--bg-primary, #1a1a1a);
  color: var(--text-primary, #ffffff);
}
```

**src/[PluginNamePascalCase].css:**
Generate base styles for the plugin.

**src/types.ts:**
```typescript
export interface PluginProps {
  services?: {
    api?: any;
    theme?: any;
    settings?: any;
    event?: any;
    pageContext?: any;
  };
}

// Add plugin-specific types based on spec
```

### Step G: Generate Feature Components

**CRITICAL:** ALL components MUST be CLASS COMPONENTS - NO HOOKS!

For each feature in the spec, create appropriate components in `src/components/`.

**Example Component Pattern:**
```typescript
import React from 'react';

interface ComponentProps {
  // Props here
}

interface ComponentState {
  // State here
}

// ALWAYS use class components!
class MyComponent extends React.Component<ComponentProps, ComponentState> {
  constructor(props: ComponentProps) {
    super(props);
    this.state = {
      // Initialize state
    };
  }

  // Event handlers as class methods
  handleClick = () => {
    this.setState({ /* update state */ });
  }

  render() {
    return (
      <div>{/* Component UI */}</div>
    );
  }
}

export default MyComponent;
```

**src/components/ErrorBoundary.tsx:**
Create error boundary component (class component with componentDidCatch).

**src/components/LoadingSpinner.tsx:**
Create loading spinner component (class component).

**src/components/ErrorDisplay.tsx:**
Create error display component (class component).

### Step H: Generate Services (if needed)

If plugin needs API interactions or business logic:

**src/services/PluginService.ts:**
Create service layer for plugin functionality.

### Step I: Generate Utils (if needed)

**src/utils/errorHandling.ts:**
Create error handling utilities.

### Step J: Add Public Assets

**IMPORTANT:** First READ public/index.html from PluginTemplate, then adapt it.

The template below is FALLBACK ONLY:

**public/index.html:**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>[Plugin Display Name] - Development</title>
    <style>
        body {
            margin: 0;
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen',
                'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue',
                sans-serif;
            -webkit-font-smoothing: antialiased;
            -moz-osx-font-smoothing: grayscale;
            background-color: #f5f5f5;
            padding: 20px;
        }

        .dev-container {
            max-width: 800px;
            margin: 0 auto;
            background: white;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            padding: 20px;
        }

        .dev-header {
            text-align: center;
            margin-bottom: 30px;
            padding-bottom: 20px;
            border-bottom: 1px solid #eee;
        }

        .dev-header h1 {
            color: #333;
            margin: 0 0 10px 0;
        }

        .dev-header p {
            color: #666;
            margin: 0;
        }

        #root {
            min-height: 400px;
        }
    </style>
</head>
<body>
    <div class="dev-container">
        <div class="dev-header">
            <h1>🧩 [Plugin Display Name]</h1>
            <p>Development Environment</p>
        </div>
        <div id="root"></div>
    </div>
</body>
</html>
```

**public/icon.png:**
Copy or create plugin icon (should be provided in spec or use placeholder).

### Step K: Progress Updates

After each major file creation, report:
- ✅ Created: [filename]
- Purpose: [what it does]

### Stopping Points

STOP and ask permission if:
- Any file already exists
- Output path seems wrong
- Missing information from spec

Tell the user: "Phase 4 complete! Run `/braindrive-plugin-builder:verify` to continue to Phase 5 (Verification)."
