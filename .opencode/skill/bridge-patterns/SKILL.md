---
name: bridge-patterns
description: BrainDrive bridge patterns for plugin communication - use when implementing API calls, settings, theme, events, or plugin state
---


## What are Bridges?

Bridges are communication channels between your plugin (frontend React component) and BrainDrive's core services. They let your plugin:
- Fetch conversation/message data
- Store settings
- Access theme colors
- Listen to events
- Persist plugin state

## Bridge Types

### 1. API Bridge (HTTP)

**Purpose:** Call BrainDrive backend endpoints

**How it works:**
```tsx
// Your plugin component
const apiUrl = props.apiUrl;  // BrainDrive passes this

// Make a request
const response = await fetch(`${apiUrl}/api/v1/conversations`, {
  headers: {
    'Authorization': `Bearer ${authToken}`,
    'Content-Type': 'application/json'
  }
});

const conversations = await response.json();
```

**Available Endpoints:**
```
GET    /api/v1/conversations          # List conversations
GET    /api/v1/conversations/{id}     # Get conversation details
GET    /api/v1/messages               # List messages
POST   /api/v1/messages               # Send message
GET    /api/v1/users/{id}             # Get user info
GET    /api/v1/settings               # Get app settings
```

**Authentication:**
- BrainDrive provides auth token in props or via localStorage
- Include in `Authorization: Bearer {token}` header
- Verify CORS headers are set correctly

**Example Service:**
```tsx
// services/api.ts
export const ApiService = {
  async getConversations(apiUrl: string): Promise<Conversation[]> {
    try {
      const response = await fetch(`${apiUrl}/api/v1/conversations`);
      if (!response.ok) throw new Error(`API error: ${response.status}`);
      return await response.json();
    } catch (error) {
      console.error('Failed to fetch conversations:', error);
      throw error;
    }
  },
  
  async createMessage(apiUrl: string, message: Message): Promise<Message> {
    const response = await fetch(`${apiUrl}/api/v1/messages`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(message)
    });
    return await response.json();
  }
};
```

### 2. Settings Bridge

**Purpose:** Store and retrieve plugin-specific user settings

**Defined in:** lifecycle_manager.py as settings schema

```python
# lifecycle_manager.py
"settings_schema": {
    "theme": {
        "type": "select",
        "label": "Theme",
        "options": ["light", "dark", "auto"],
        "default": "auto"
    },
    "notifications_enabled": {
        "type": "boolean",
        "label": "Enable notifications",
        "default": True
    },
    "max_items": {
        "type": "number",
        "label": "Maximum items to display",
        "default": 50,
        "min": 1,
        "max": 1000
    }
}
```

**Access in Component:**
```tsx
interface Props {
  settings?: {
    theme: string;
    notifications_enabled: boolean;
    max_items: number;
  };
  onSettingsChange?: (settings: Record<string, any>) => void;
}

export const MyPlugin: React.FC<Props> = ({ settings, onSettingsChange }) => {
  const handleThemeChange = (theme: string) => {
    onSettingsChange?.({ ...settings, theme });
  };
  
  return (
    <div>
      <select value={settings?.theme} onChange={(e) => handleThemeChange(e.target.value)}>
        <option value="light">Light</option>
        <option value="dark">Dark</option>
        <option value="auto">Auto</option>
      </select>
    </div>
  );
};
```

**Schema Field Types:**
- `text` - Single line input
- `textarea` - Multi-line text
- `number` - Numeric input with min/max
- `boolean` - Toggle/checkbox
- `select` - Dropdown with options
- `email` - Email validation
- `password` - Hidden text input
- `url` - URL validation

### 3. Theme Bridge

**Purpose:** Access BrainDrive's theme (colors, fonts) for consistent UI

**How it works:**
```tsx
// BrainDrive injects CSS variables
export const MyPlugin: React.FC = () => {
  return (
    <div className="plugin-container">
      {/* These use theme colors */}
      <p style={{ color: 'var(--primary-color)' }}>Primary text</p>
      <p style={{ color: 'var(--secondary-color)' }}>Secondary text</p>
    </div>
  );
};
```

**Available CSS Variables:**
```css
--primary-color: #2563eb;
--secondary-color: #475569;
--accent-color: #f59e0b;
--background-color: #ffffff;
--surface-color: #f8fafc;
--border-color: #e2e8f0;
--success-color: #10b981;
--warning-color: #f59e0b;
--error-color: #ef4444;
--text-primary: #1f2937;
--text-secondary: #6b7280;
--text-disabled: #d1d5db;
```

**Or receive as props:**
```tsx
interface Props {
  theme?: {
    primaryColor: string;
    secondaryColor: string;
    accentColor: string;
    isDark: boolean;
  };
}

export const MyPlugin: React.FC<Props> = ({ theme }) => {
  return (
    <div style={{ 
      backgroundColor: theme?.isDark ? '#1f2937' : '#ffffff',
      color: theme?.isDark ? '#f3f4f6' : '#1f2937'
    }}>
      Themed content
    </div>
  );
};
```

### 4. Event Bridge

**Purpose:** Subscribe to BrainDrive events

**Concept:** Listen to events in your component

```tsx
import { useEffect, useState } from 'react';

export const MyPlugin: React.FC = () => {
  const [lastEvent, setLastEvent] = useState<string>('');
  
  useEffect(() => {
    // Listen to conversation events
    const handler = (event: CustomEvent) => {
      setLastEvent(event.detail.type);
    };
    
    window.addEventListener('braindrive:conversation-created', handler);
    window.addEventListener('braindrive:conversation-deleted', handler);
    window.addEventListener('braindrive:message-received', handler);
    
    return () => {
      window.removeEventListener('braindrive:conversation-created', handler);
      window.removeEventListener('braindrive:conversation-deleted', handler);
      window.removeEventListener('braindrive:message-received', handler);
    };
  }, []);
  
  return <div>Last event: {lastEvent}</div>;
};
```

**Available Events:**
- `braindrive:conversation-created` - New conversation
- `braindrive:conversation-updated` - Conversation modified
- `braindrive:conversation-deleted` - Conversation removed
- `braindrive:message-received` - New message
- `braindrive:message-updated` - Message modified
- `braindrive:user-settings-changed` - User settings changed
- `braindrive:theme-changed` - Theme switched
- `braindrive:plugin-activated` - Your plugin loaded
- `braindrive:plugin-deactivated` - Your plugin unloaded

### 5. Plugin State Bridge

**Purpose:** Persist plugin state across sessions

**Local Storage Approach:**
```tsx
const KEY = 'my-plugin:state';

// Save state
const saveState = (state: any) => {
  localStorage.setItem(KEY, JSON.stringify(state));
};

// Load state
const loadState = () => {
  const stored = localStorage.getItem(KEY);
  return stored ? JSON.parse(stored) : null;
};

// Use in component
export const MyPlugin: React.FC = () => {
  const [state, setState] = useState(() => loadState());
  
  const handleUpdate = (newState: any) => {
    setState(newState);
    saveState(newState);
  };
  
  return <div>State: {JSON.stringify(state)}</div>;
};
```

**Backend State (if using lifecycle_manager):**
```python
# In lifecycle_manager.py
async def save_plugin_state(self, user_id: str, state_key: str, state_value: any, db):
    """Save plugin state to database"""
    async with db.begin() as conn:
        await conn.execute(text("""
            INSERT INTO plugin_state (user_id, state_key, state_value)
            VALUES (:user_id, :key, :value)
            ON CONFLICT (user_id, state_key) DO UPDATE SET state_value = :value
        """), {
            "user_id": user_id,
            "key": state_key,
            "value": state_value
        })

async def load_plugin_state(self, user_id: str, state_key: str, db):
    """Load plugin state from database"""
    result = await db.fetch_one(text("""
        SELECT state_value FROM plugin_state
        WHERE user_id = :user_id AND state_key = :key
    """), {
        "user_id": user_id,
        "key": state_key
    })
    return result['state_value'] if result else None
```

### 6. Log Bridge

**Purpose:** Write logs to BrainDrive's log system

```tsx
// Log to BrainDrive
const logToPlugin = (level: string, message: string, data?: any) => {
  window.dispatchEvent(new CustomEvent('braindrive:plugin-log', {
    detail: { level, message, data, plugin: 'MyPlugin' }
  }));
};

// Use in component
logToPlugin('info', 'Plugin initialized');
logToPlugin('error', 'Failed to load data', { reason: 'network error' });
logToPlugin('warn', 'High resource usage detected');
```

## Complete Bridge Example

Here's a real plugin using all bridges:

```tsx
// MyPlugin.tsx
import React, { useEffect, useState } from 'react';
import { ApiService } from './services/api';

interface Props {
  apiUrl?: string;
  settings?: {
    theme: string;
    itemsPerPage: number;
  };
  onSettingsChange?: (settings: any) => void;
  theme?: {
    primaryColor: string;
    isDark: boolean;
  };
}

export const MyPlugin: React.FC<Props> = ({
  apiUrl,
  settings,
  onSettingsChange,
  theme
}) => {
  const [conversations, setConversations] = useState<any[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  
  // API Bridge: Fetch conversations
  useEffect(() => {
    if (!apiUrl) return;
    
    const fetchData = async () => {
      setLoading(true);
      try {
        const data = await ApiService.getConversations(apiUrl);
        setConversations(data);
        setError(null);
      } catch (err) {
        setError(String(err));
      } finally {
        setLoading(false);
      }
    };
    
    fetchData();
  }, [apiUrl]);
  
  // Event Bridge: Listen to new messages
  useEffect(() => {
    const handleMessage = (event: CustomEvent) => {
      setConversations(prev => [...prev, event.detail]);
    };
    
    window.addEventListener('braindrive:message-received', handleMessage);
    return () => window.removeEventListener('braindrive:message-received', handleMessage);
  }, []);
  
  // Theme Bridge: Use theme colors
  const containerStyle = {
    backgroundColor: theme?.isDark ? '#1f2937' : '#ffffff',
    color: theme?.isDark ? '#f3f4f6' : '#1f2937',
    padding: '1rem',
    borderRadius: '0.5rem',
    borderColor: theme?.primaryColor
  };
  
  return (
    <div style={containerStyle}>
      {/* Settings Bridge */}
      <div>
        <label>Items per page:</label>
        <input 
          type="number"
          value={settings?.itemsPerPage}
          onChange={(e) => onSettingsChange?.({
            ...settings,
            itemsPerPage: parseInt(e.target.value)
          })}
        />
      </div>
      
      {/* Display data */}
      {loading && <p>Loading...</p>}
      {error && <p style={{ color: 'red' }}>Error: {error}</p>}
      {conversations.map(conv => (
        <div key={conv.id}>{conv.title}</div>
      ))}
    </div>
  );
};

export default MyPlugin;
```

## Best Practices

✅ **DO:**
- Handle API errors gracefully
- Store sensitive data server-side, not localStorage
- Use theme variables for consistent UI
- Subscribe/unsubscribe from events properly
- Validate all settings values

❌ **DON'T:**
- Hardcode colors or theme values
- Make API calls in render
- Forget to clean up event listeners
- Store API tokens in localStorage
- Assume settings values are always present

---

Bridges are your plugin's interface to BrainDrive. Master them, and your plugin becomes a seamless part of the ecosystem.
