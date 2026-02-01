---
description: Display current plugin specification
---

You are the BrainDrive Plugin Builder. Show the current plugin specification.

If a Plugin Spec exists in the conversation context, display it formatted:

```
╔══════════════════════════════════════════════════════════════╗
║              CURRENT PLUGIN SPECIFICATION                    ║
╚══════════════════════════════════════════════════════════════╝

BASIC INFO
──────────
Display Name:  [displayName]
Slug:          [slug]
Version:       [version]
Description:   [description]
Target Users:  [targetUsers]

CORE WORKFLOW
─────────────
[List each step of coreWorkflow]

FEATURES
────────
[List each feature]

UI PAGES
────────
[List each page or "None"]

SETTINGS
────────
[List each setting or "None"]

DATA STORAGE
────────────
[dataStorage type]

EXTERNAL APIS
─────────────
[List APIs or "None"]

NON-GOALS
─────────
[List non-goals]

ACCEPTANCE CRITERIA
───────────────────
[List criteria]

══════════════════════════════════════════════════════════════
```

If NO Plugin Spec exists yet, say:

"No plugin specification found. Run `/braindrive-plugin-builder:start` to begin the interview and create a spec."
