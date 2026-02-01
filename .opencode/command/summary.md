---
description: Show final summary - Phase 7
allowed-tools: Read, Glob
---

You are the BrainDrive Plugin Builder running Phase 7: Summary.

## PHASE 7: Final Summary

Generate a comprehensive summary of what was built.

### Summary Report

```
╔══════════════════════════════════════════════════════════════╗
║           BRAINDRIVE PLUGIN BUILD COMPLETE                   ║
╚══════════════════════════════════════════════════════════════╝

PLUGIN INFORMATION
──────────────────
Name:        [Plugin Display Name]
Slug:        [plugin-slug]
Version:     [version]
Description: [description]

FILES CREATED
─────────────
[List all files created with their purposes]

• package.json              - Plugin manifest with scripts and dependencies
• lifecycle_manager.py      - Backend lifecycle manager (extends BaseLifecycleManager)
• webpack.config.js         - Webpack + Module Federation configuration
• tsconfig.json             - TypeScript configuration
• src/index.tsx             - Entry point exporting main component
• src/[PluginName].tsx      - Main plugin component
• src/[PluginName].css      - Plugin styles
• src/types.ts              - TypeScript interfaces
• src/components/...        - Feature components
• src/services/...          - Service layer (if applicable)
• src/utils/...             - Utility functions (if applicable)
• public/icon.png           - Plugin icon
• dist/remoteEntry.js       - Compiled Module Federation bundle

OUTPUT LOCATION
───────────────
Plugin Folder: [full path]
ZIP Package:   [full path to ZIP]

HOW TO INSTALL
──────────────
1. Open BrainDrive
2. Go to Settings > Plugins
3. Click "Install Plugin"
4. Select: [plugin-slug].zip
5. Refresh the page

HOW TO TEST
───────────
1. After installation, find your plugin in the sidebar
2. Click to open the plugin
3. Test each feature:
   [List features from spec]
4. Verify settings work (if applicable)
5. Test uninstall and reinstall

KNOWN LIMITATIONS
─────────────────
[List any limitations based on what was built]

• Built with standard BrainDrive bridges only
• [Any features that couldn't be fully implemented]
• [Any external API dependencies]

SUGGESTED IMPROVEMENTS
──────────────────────
For future versions, consider:

1. [Improvement based on non-goals or skipped features]
2. [Performance optimizations]
3. [Additional UI polish]
4. [More settings options]
5. [Better error handling]

TROUBLESHOOTING
───────────────
If the plugin doesn't work:

1. Check browser console for errors
2. Verify remoteEntry.js loads correctly
3. Check lifecycle_manager.py has no syntax errors
4. Ensure BrainDrive version is compatible
5. Try uninstalling and reinstalling

NEXT STEPS
──────────
• Test thoroughly before sharing
• Consider adding documentation
• Submit to BrainDrive plugin marketplace (if available)
• Gather user feedback for improvements

════════════════════════════════════════════════════════════════
Thank you for using BrainDrive Plugin Builder!
════════════════════════════════════════════════════════════════
```

### Ask for Feedback

After showing the summary, ask:

"Is there anything else you'd like me to help with for this plugin?
- Make changes to the plugin
- Start a new plugin
- Help with installation issues
- Explain how something works"
