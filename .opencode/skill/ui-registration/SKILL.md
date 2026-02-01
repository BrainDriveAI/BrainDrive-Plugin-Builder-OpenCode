---
name: ui-registration
description: BrainDrive UI page registration patterns - use when adding pages, routes, or modules to plugins
---


## What is UI Page Registration?

UI page registration is how your plugin tells BrainDrive where to display your component. Without registration, BrainDrive doesn't know where to mount your React component.

## Registration Concepts

### Route/Page Definition
Each UI page needs:
- **Page slug:** Unique identifier (e.g., `dashboard`, `settings`)
- **Component name:** What to render (must be exported from your bundle)
- **Route:** URL path or menu location
- **Title:** Display name in UI
- **Icon:** What icon to show in menus

### Module vs Component
- **Module:** The webpack Module Federation chunk (e.g., `./Dashboard`)
- **Component:** The React component inside the module (e.g., `DashboardComponent`)

## Registration Methods

### Method 1: In lifecycle_manager.py (Recommended)

Define pages in the `modules` metadata:

```python
# lifecycle_manager.py
self.modules = [
    {
        "module_id": "main",
        "name": "Main Dashboard",
        "role": "frontend",
        "description": "Main plugin UI",
        "pages": [
            {
                "slug": "dashboard",
                "title": "Dashboard",
                "route": "/plugin/my-plugin/dashboard",
                "icon": "BarChart3",
                "component": "DashboardPage",
                "description": "Main dashboard view"
            },
            {
                "slug": "settings",
                "title": "Settings",
                "route": "/plugin/my-plugin/settings",
                "icon": "Settings",
                "component": "SettingsPage",
                "description": "Plugin configuration"
            }
        ],
        "component": {
            "bundlelocation": "dist/remoteEntry.js",
            "scope": "MyPlugin",
            "module": "./index",
            "container": "MY_PLUGIN_CONTAINER"
        }
    }
]
```

### Method 2: In Your Component (Declarative)

Export metadata from your component:

```tsx
// src/MyPlugin.tsx
export const MyPlugin: React.FC = () => {
  return <div>Main plugin UI</div>;
};

// Register pages
MyPlugin.pages = [
  {
    slug: 'dashboard',
    title: 'Dashboard',
    component: 'DashboardPage',
    route: '/plugin/my-plugin/dashboard'
  },
  {
    slug: 'settings',
    title: 'Settings',
    component: 'SettingsPage',
    route: '/plugin/my-plugin/settings'
  }
];
```

## Complete Registration Example

### With Multiple Pages

```python
# lifecycle_manager.py
self.modules = [
    {
        "module_id": "main",
        "name": "Complete Plugin",
        "role": "frontend",
        "description": "Full-featured plugin",
        "pages": [
            {
                "slug": "home",
                "title": "Home",
                "route": "/plugin/task-manager/home",
                "icon": "Home",
                "component": "HomePage",
                "description": "Main home page",
                "default": True,  # Show this page first
                "public": False   # Requires authentication
            },
            {
                "slug": "tasks",
                "title": "My Tasks",
                "route": "/plugin/task-manager/tasks",
                "icon": "CheckCircle",
                "component": "TasksPage",
                "description": "View and manage tasks",
                "default": False,
                "public": False
            },
            {
                "slug": "analytics",
                "title": "Analytics",
                "route": "/plugin/task-manager/analytics",
                "icon": "BarChart3",
                "component": "AnalyticsPage",
                "description": "Task completion analytics",
                "default": False,
                "public": False
            },
            {
                "slug": "settings",
                "title": "Settings",
                "route": "/plugin/task-manager/settings",
                "icon": "Settings",
                "component": "SettingsPage",
                "description": "Plugin configuration",
                "default": False,
                "public": False
            }
        ],
        "component": {
            "bundlelocation": "dist/remoteEntry.js",
            "scope": "TaskManager",
            "module": "./index",
            "container": "TASK_MANAGER_CONTAINER"
        }
    }
]
```

### Component Structure to Match Registration

Your component file must export each registered component:

```tsx
// src/index.tsx
export const HomePage = lazy(() => import('./pages/HomePage'));
export const TasksPage = lazy(() => import('./pages/TasksPage'));
export const AnalyticsPage = lazy(() => import('./pages/AnalyticsPage'));
export const SettingsPage = lazy(() => import('./pages/SettingsPage'));

// Entry point for Module Federation
export default () => HomePage;

// Or use a router
export const TaskManager = () => {
  const [currentPage] = useSearchParams();
  
  switch(currentPage.get('slug')) {
    case 'tasks': return <TasksPage />;
    case 'analytics': return <AnalyticsPage />;
    case 'settings': return <SettingsPage />;
    default: return <HomePage />;
  }
};

export default TaskManager;
```

## Page Definition Fields

| Field | Type | Required | Example |
|-------|------|----------|---------|
| `slug` | string | ✅ | `"dashboard"` |
| `title` | string | ✅ | `"Dashboard"` |
| `route` | string | ✅ | `"/plugin/my-plugin/dashboard"` |
| `icon` | string | ✅ | `"BarChart3"` |
| `component` | string | ✅ | `"DashboardPage"` |
| `description` | string | ❌ | `"Main dashboard view"` |
| `default` | boolean | ❌ | `true` (show first) |
| `public` | boolean | ❌ | `false` (requires auth) |
| `children` | array | ❌ | Sub-pages/nested routes |

## Route Naming Convention

Routes should follow this pattern:

```
/plugin/{plugin-slug}/{page-slug}
```

Examples:
- `/plugin/task-manager/home`
- `/plugin/task-manager/tasks`
- `/plugin/task-manager/settings`
- `/plugin/data-dashboard/analytics`
- `/plugin/data-dashboard/export`

## Icon Selection

Choose from BrainDrive's icon library (Feather icons):

```python
# Navigation
"Home", "Menu", "ChevronRight", "ChevronDown"

# Content
"FileText", "Image", "MessageSquare", "Bell"

# Actions
"Plus", "Edit", "Trash2", "Download", "Upload"

# Status
"CheckCircle", "AlertCircle", "Info", "HelpCircle"

# Data
"BarChart3", "LineChart", "Pie Chart", "TrendingUp"

# Organization
"Folder", "Archive", "Tag", "Filter"

# Settings
"Settings", "Sliders", "Tool", "Zap"

# Users
"Users", "User", "UserCheck", "UserX"

# Calendar
"Calendar", "Clock", "Clock3"
```

## Nested/Sub-Pages

For hierarchical structures:

```python
{
    "slug": "settings",
    "title": "Settings",
    "route": "/plugin/my-plugin/settings",
    "icon": "Settings",
    "component": "SettingsLayout",
    "children": [
        {
            "slug": "general",
            "title": "General",
            "route": "/plugin/my-plugin/settings/general",
            "component": "GeneralSettings"
        },
        {
            "slug": "notifications",
            "title": "Notifications",
            "route": "/plugin/my-plugin/settings/notifications",
            "component": "NotificationSettings"
        },
        {
            "slug": "integrations",
            "title": "Integrations",
            "route": "/plugin/my-plugin/settings/integrations",
            "component": "IntegrationSettings"
        }
    ]
}
```

## Context Menu Registration

Register plugin actions in context menus:

```python
{
    "module_id": "context-menu",
    "name": "Context Actions",
    "role": "plugin",
    "actions": [
        {
            "slug": "add-to-plugin",
            "title": "Add to My Plugin",
            "context": ["conversation", "message"],
            "handler": "addToPlugin"
        },
        {
            "slug": "share-via-plugin",
            "title": "Share via Plugin",
            "context": ["conversation"],
            "handler": "shareViaPlugin"
        }
    ]
}
```

## Dynamic Page Registration

For plugin-generated pages:

```tsx
// During plugin initialization
export const MyPlugin: React.FC = () => {
  const [pages, setPages] = useState([
    { slug: 'default', title: 'Default', component: 'DefaultPage' }
  ]);
  
  const addPage = (pageConfig) => {
    setPages([...pages, pageConfig]);
    // Notify BrainDrive of new page
    window.dispatchEvent(new CustomEvent('braindrive:pages-updated', {
      detail: pages
    }));
  };
  
  return <div>Dynamic pages setup</div>;
};
```

## Validation Checklist for Pages

- [ ] Every page `slug` is unique within module
- [ ] Every page `component` is exported from your bundle
- [ ] Route follows `/plugin/{slug}/{page-slug}` pattern
- [ ] Icon name exists in icon library
- [ ] At least one page has `default: true`
- [ ] `component` field matches exported component name
- [ ] All nested `children` have `component` fields
- [ ] No circular dependencies between pages
- [ ] Public pages are stateless or preload data

## Common Mistakes

❌ **Component name doesn't match export:**
```python
# In lifecycle_manager
"component": "DashboardPage"

# But exported as
export const Dashboard = ...  // WRONG! Names don't match
```

✅ **Fix: Match names exactly**
```tsx
export const DashboardPage = ...  // CORRECT!
```

❌ **Routes don't follow pattern:**
```python
"route": "/my-plugin/dashboard"  # Missing /plugin/
```

✅ **Fix: Use correct pattern**
```python
"route": "/plugin/my-plugin/dashboard"
```

❌ **No default page:**
```python
"pages": [
    { "slug": "settings", ... },
    { "slug": "about", ... }
    # BrainDrive doesn't know what to show!
]
```

✅ **Fix: Set one as default**
```python
"pages": [
    { "slug": "dashboard", ..., "default": True },
    { "slug": "settings", ... },
    { "slug": "about", ... }
]
```

---

**Golden Rule:** Every component you register must be exported from your bundle, and every route must follow BrainDrive's naming convention.
