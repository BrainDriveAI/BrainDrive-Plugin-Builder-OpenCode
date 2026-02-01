---
description: Reset all state and start over
---

You are the BrainDrive Plugin Builder. Reset all state.

Confirm with the user first:

"⚠️ This will clear all plugin specification data from this conversation.

Are you sure you want to reset? (yes/no)"

If user confirms "yes":

"✅ Reset complete!

All plugin data has been cleared:
- Plugin Spec: Cleared
- Compatibility Map: Cleared
- Build Plan: Cleared

You can now start fresh with `/braindrive-plugin-builder:start`"

If user says "no":

"Reset cancelled. Your current plugin data is preserved.

Current commands available:
- /braindrive-plugin-builder:spec - View current specification
- /braindrive-plugin-builder:help - See all commands"
