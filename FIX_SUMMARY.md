# Proxmox Tag Folder VM Display Fix - Summary

## Issue Description

When using the tag-based folder organization feature:
1. User clicks on a tag folder in the tree
2. User then clicks on a VM within that folder  
3. **Result**: VM details panel goes blank
4. **Console Error**: `Uncaught TypeError: Cannot read properties of undefined (reading 'replace')`

## Root Causes

### Cause 1: Invalid State Persistence
- Tag folders (ID format: `tag/A-Production`) were being saved to application state
- These synthetic IDs don't correspond to actual resources
- State restoration failed when trying to load content for these IDs

### Cause 2: Content Panel Creation for Non-Resources
- The `selectionchange` event handler tried to create content panels for tag folders
- Tag folders lack the required metadata (like help info)
- Calling `setOnlineHelp` on undefined data caused the error

## Fixes Applied

### Fix 1: State Persistence Filter (Line ~16431)
**Location**: `PVE.tree.ResourceTree` initialization

```javascript
// BEFORE
me.getSelectionModel().on('select', (_sm, n) => sp.set(stateid, { value: n.data.id }));

// AFTER  
me.getSelectionModel().on('select', (_sm, n) => {
    // Only persist state for actual resource nodes, not for tag folders or grouping nodes
    if (n.data.type !== 'tag-folder' && !n.data.groupbyid) {
        sp.set(stateid, { value: n.data.id });
    }
});
```

**Effect**: Tag folder selections are no longer saved to state. Only actual resources persist.

### Fix 2: Selection Change Handler Guard (Line ~60920)
**Location**: `PVE.StdWorkspace` resource tree configuration

```javascript
// BEFORE
selectionchange: function(sm, selected) {
    if (selected.length <= 0) {
        return;
    }
    let treeNode = selected[0];
    // ... proceed to create content

// AFTER
selectionchange: function(sm, selected) {
    if (selected.length <= 0) {
        return;
    }
    let treeNode = selected[0];
    
    // Skip content creation for tag folders and grouping nodes
    if (treeNode.data.type === 'tag-folder' || treeNode.data.groupbyid) {
        return;
    }
    
    // ... proceed to create content
```

**Effect**: Content panel creation is skipped for tag folders, preventing the undefined error.

## Expected Behavior After Fix

✅ Click on tag folder → Folder expands/collapses (no error)
✅ Click on VM → VM details panel displays correctly  
✅ Click on another VM → Details update properly
✅ Refresh browser → Last selected VM (not folder) is restored
✅ No console errors
✅ All other tree navigation works normally

## Testing Checklist

- [ ] Click folder then VM - verify VM details show
- [ ] Click multiple VMs in sequence - verify navigation works
- [ ] Click folder multiple times - verify no errors  
- [ ] Refresh browser after VM selection - verify restoration
- [ ] Test nested folders (if applicable)
- [ ] Verify node navigation still works
- [ ] Verify storage navigation still works
- [ ] Check browser console - no errors

## Files Modified

- `pvemanagerlib.js` - Two changes applied
- `README.md` - Updated with fix information
- `INSTALL.md` - Updated installation instructions
- `FIX_SUMMARY.md` - This file (new)

## Version

- Original Version: 1.0 (March 2025)
- Fixed Version: 1.1 (April 2026)
- Status: Ready for deployment

## Deployment

See `INSTALL.md` for detailed deployment instructions.
