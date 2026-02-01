---
name: lifecycle-manager
description: BrainDrive plugin lifecycle manager rules - use when creating or validating lifecycle_manager.py, handling install/uninstall, or database operations
---


## What is the Lifecycle Manager?

The **lifecycle_manager.py** file handles what happens when a user installs, updates, or uninstalls your plugin. It's the backend "glue" between your plugin and BrainDrive.

### Key Responsibility
- **Install:** Called when user installs plugin (per-user)
- **Update:** Called when plugin version changes
- **Uninstall:** Called when user removes plugin
- **Metadata:** Declares plugin name, version, features
- **Database:** Creates/manages tables if needed

## File Location and Structure

```
your-plugin/
├── lifecycle_manager.py        # THIS FILE
├── package.json
├── webpack.config.js
└── src/
```

## Base Class: BaseLifecycleManager

Your lifecycle_manager.py must inherit from `BaseLifecycleManager`:

```python
from app.plugins.base_lifecycle_manager import BaseLifecycleManager

class YourPluginLifecycleManager(BaseLifecycleManager):
    def __init__(self, plugins_base_dir: str = None):
        # Define plugin metadata
        self.plugin_data = { ... }
        # Define modules
        self.modules = { ... }
    
    async def get_plugin_metadata(self):
        return self.plugin_data
    
    async def get_module_metadata(self):
        return self.modules
    
    async def _perform_user_installation(self, user_id, db, shared_plugin_path):
        # What happens when user installs
        pass
    
    async def _perform_user_uninstallation(self, user_id, db):
        # What happens when user uninstalls
        pass
```

## Required Methods

### 1. __init__(self, plugins_base_dir)

Initializes the lifecycle manager. Must set:

```python
def __init__(self, plugins_base_dir: str = None):
    self.plugin_data = {
        "name": "Your Plugin Name",
        "plugin_slug": "YourPluginName",
        "scope": "YourPluginName",
        "version": "1.0.0",
        "type": "frontend",
        "description": "What your plugin does",
        "bundle_location": "dist/remoteEntry.js",
        # ... other metadata
    }
    
    self.modules = [
        {
            "module_id": "main",
            "name": "Main Module",
            "role": "frontend",
            "description": "Main plugin interface",
            "component": {
                "bundlelocation": "dist/remoteEntry.js",
                "scope": "YourPluginName",
                "module": "./index",
                "container": "YOUR_CONTAINER_ID"
            }
        }
    ]
    
    super().__init__(
        plugin_slug="YourPluginName",
        version="1.0.0",
        shared_storage_path=Path(plugins_base_dir)
    )
```

### 2. async get_plugin_metadata(self)

Returns the plugin_data dictionary:

```python
async def get_plugin_metadata(self):
    return self.plugin_data
```

### 3. async get_module_metadata(self)

Returns list of module definitions:

```python
async def get_module_metadata(self):
    return self.modules
```

### 4. async _perform_user_installation(self, user_id, db, shared_plugin_path)

Called when user installs plugin. Return success/error:

```python
async def _perform_user_installation(self, user_id: str, db, shared_plugin_path: Path):
    try:
        # Create database tables if needed
        if not await self._tables_exist(db):
            await self._create_tables(db)
        
        # Initialize user-specific data
        await self._init_user_data(user_id, db)
        
        return {
            'success': True,
            'message': 'Plugin installed successfully'
        }
    except Exception as e:
        return {
            'success': False,
            'error': f'Installation failed: {str(e)}'
        }
```

### 5. async _perform_user_uninstallation(self, user_id, db)

Called when user uninstalls plugin. Clean up data:

```python
async def _perform_user_uninstallation(self, user_id: str, db):
    try:
        # Clean up user-specific data
        await self._cleanup_user_data(user_id, db)
        
        # Don't delete shared tables (other users may need them)
        
        return {
            'success': True,
            'message': 'Plugin uninstalled successfully'
        }
    except Exception as e:
        return {
            'success': False,
            'error': f'Uninstallation failed: {str(e)}'
        }
```

## plugin_data Dictionary

### Minimal Example (Frontend-Only Plugin)

```python
self.plugin_data = {
    "name": "Task Manager",
    "description": "Manage your daily tasks",
    "version": "1.0.0",
    "type": "frontend",
    "icon": "CheckCircle",
    "category": "productivity",
    "plugin_slug": "TaskManager",
    "scope": "TaskManager",
    "bundle_method": "webpack",
    "bundle_location": "dist/remoteEntry.js",
    "is_local": False,
    "official": False,
    "author": "Your Name",
    "compatibility": "1.0.0",
}
```

### Complete Example (With All Fields)

```python
self.plugin_data = {
    # Identity
    "name": "Advanced Plugin",
    "plugin_slug": "AdvancedPlugin",
    "scope": "AdvancedPlugin",
    "version": "1.2.0",
    "type": "full",  # "frontend", "backend", or "full"
    
    # Description
    "description": "An advanced plugin with all features",
    "long_description": "This is a longer description that explains what the plugin does in detail.",
    "category": "productivity",
    "icon": "Zap",
    
    # Technical
    "bundle_method": "webpack",
    "bundle_location": "dist/remoteEntry.js",
    "compatibility": "1.0.0",
    "is_local": False,
    "official": False,
    
    # Source
    "source_type": "github",
    "source_url": "https://github.com/user/advanced-plugin",
    "update_check_url": "https://api.github.com/repos/user/advanced-plugin/releases/latest",
    
    # Author
    "author": "John Doe",
    "license": "MIT",
    "documentation_url": "https://docs.example.com/plugin",
}
```

## modules List

Defines what UI modules your plugin exposes:

### Minimal (One Module)

```python
self.modules = [
    {
        "module_id": "main",
        "name": "Main Module",
        "role": "frontend",
        "description": "Main plugin UI",
        "component": {
            "bundlelocation": "dist/remoteEntry.js",
            "scope": "YourPluginName",
            "module": "./index",
            "container": "YOUR_UNIQUE_CONTAINER_ID"
        }
    }
]
```

### Multiple Modules

```python
self.modules = [
    {
        "module_id": "dashboard",
        "name": "Dashboard View",
        "role": "frontend",
        "description": "Main dashboard",
        "component": {
            "bundlelocation": "dist/remoteEntry.js",
            "scope": "YourPluginName",
            "module": "./Dashboard",
            "container": "DASHBOARD_CONTAINER"
        }
    },
    {
        "module_id": "settings",
        "name": "Settings View",
        "role": "frontend",
        "description": "Plugin settings and configuration",
        "component": {
            "bundlelocation": "dist/remoteEntry.js",
            "scope": "YourPluginName",
            "module": "./Settings",
            "container": "SETTINGS_CONTAINER"
        }
    }
]
```

## Database Operations

### Creating Tables on Install

```python
async def _create_tables(self, db):
    """Create plugin-specific database tables"""
    async with db.begin() as conn:
        await conn.execute(text("""
            CREATE TABLE IF NOT EXISTS plugin_tasks (
                id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
                user_id VARCHAR(255) NOT NULL,
                title VARCHAR(255) NOT NULL,
                description TEXT,
                completed BOOLEAN DEFAULT FALSE,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                CONSTRAINT unique_user_task UNIQUE (user_id, title)
            )
        """))
```

### Checking if Tables Exist

```python
async def _tables_exist(self, db):
    """Check if plugin tables are already created"""
    async with db.connect() as conn:
        result = await conn.execute(text("""
            SELECT EXISTS (
                SELECT 1 FROM information_schema.tables 
                WHERE table_name = 'plugin_tasks'
            )
        """))
        return result.scalar()
```

### Initializing User Data

```python
async def _init_user_data(self, user_id: str, db):
    """Initialize user-specific plugin data"""
    async with db.begin() as conn:
        # Insert default user settings
        await conn.execute(text("""
            INSERT INTO plugin_user_settings (user_id, setting_key, setting_value)
            VALUES (:user_id, :key, :value)
        """), {
            "user_id": user_id,
            "key": "theme",
            "value": "light"
        })
```

### Cleaning Up User Data

```python
async def _cleanup_user_data(self, user_id: str, db):
    """Clean up user-specific data on uninstall"""
    async with db.begin() as conn:
        # Delete user's tasks
        await conn.execute(text("""
            DELETE FROM plugin_tasks
            WHERE user_id = :user_id
        """), {"user_id": user_id})
        
        # Delete user's settings
        await conn.execute(text("""
            DELETE FROM plugin_user_settings
            WHERE user_id = :user_id
        """), {"user_id": user_id})
```

## Error Handling

Always wrap operations in try/except and return structured response:

```python
async def _perform_user_installation(self, user_id: str, db, shared_plugin_path: Path):
    try:
        # Your code here
        logger.info(f"Installing plugin for user {user_id}")
        
        if not await self._tables_exist(db):
            await self._create_tables(db)
        
        return {
            'success': True,
            'message': 'Plugin installed successfully',
            'data': {'user_id': user_id}
        }
    
    except ValueError as e:
        logger.error(f"Validation error during installation: {e}")
        return {
            'success': False,
            'error': f'Validation failed: {str(e)}'
        }
    
    except Exception as e:
        logger.error(f"Installation failed: {e}")
        return {
            'success': False,
            'error': f'Installation failed: {str(e)}'
        }
```

## Logging

Use structlog for consistent logging:

```python
import structlog

logger = structlog.get_logger()

# Use like:
logger.info("plugin_installed", user_id=user_id, plugin_name=self.plugin_data['name'])
logger.error("installation_failed", error=str(e), user_id=user_id)
```

## Testing Your Lifecycle Manager

Test all three lifecycle methods:

```python
# Simulate installation
result = await manager._perform_user_installation("user_123", db, plugin_path)
assert result['success'] == True

# Verify data was created
tasks = await db.fetch("SELECT * FROM plugin_tasks WHERE user_id = 'user_123'")
assert len(tasks) > 0

# Simulate uninstallation
result = await manager._perform_user_uninstallation("user_123", db)
assert result['success'] == True

# Verify data was cleaned
tasks = await db.fetch("SELECT * FROM plugin_tasks WHERE user_id = 'user_123'")
assert len(tasks) == 0
```

## Common Mistakes

❌ **Wrong:** Deleting shared tables on uninstall
```python
# DON'T DO THIS - other users need this!
await conn.execute(text("DROP TABLE plugin_tasks"))
```

✅ **Right:** Only delete user-specific data
```python
# DO THIS - only clean up this user's data
await conn.execute(text("DELETE FROM plugin_tasks WHERE user_id = ?"), (user_id,))
```

❌ **Wrong:** Not matching scope to webpack config
```python
# In lifecycle_manager.py
"scope": "MyPlugin"
```
```javascript
// In webpack.config.js
name: 'DifferentScope'  // WRONG!
```

✅ **Right:** Match exactly
```python
# Both must be
"scope": "MyPlugin"
```
```javascript
name: 'MyPlugin'
```

---

The lifecycle_manager.py is where your plugin becomes **multi-user aware**. Get it right, and your plugin works for every BrainDrive user seamlessly.
