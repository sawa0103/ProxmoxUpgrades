# Installation Instructions for Fixed Version

## What This Fix Does

This update fixes the issue where clicking on a tag folder and then clicking on a VM would cause the VM details panel to go blank and show a console error. 

The fix includes **two critical changes**:
1. **Prevents tag folders from being saved to application state** - Tag folder selections no longer persist across page refreshes
2. **Skips content panel creation for tag folders** - Tag folders now only expand/collapse without trying to load content, preventing the `Cannot read properties of undefined (reading 'replace')` error

## Installation Steps

### 1. Backup Your Current File

Before applying the fix, backup your existing `pvemanagerlib.js`:

```bash
cp /usr/share/pve-manager/js/pvemanagerlib.js /usr/share/pve-manager/js/pvemanagerlib.js.backup-$(date +%Y%m%d)
```

### 2. Upload the Fixed File

Upload the fixed `pvemanagerlib.js` to your Proxmox server:

```bash
scp pvemanagerlib.js root@your-proxmox-ip:/usr/share/pve-manager/js/
```

### 3. Set Correct Permissions

```bash
ssh root@your-proxmox-ip "chmod 644 /usr/share/pve-manager/js/pvemanagerlib.js"
```

### 4. Restart Proxmox Services

```bash
ssh root@your-proxmox-ip "systemctl restart pveproxy && systemctl restart pvedaemon"
```

### 5. Clear Browser Cache

In your browser, press **Ctrl + Shift + R** (or **Cmd + Shift + R** on Mac) to hard refresh and clear cached files.

## Testing the Fix

After installation, verify the fix works:

1. ✅ Click on a tag folder in the tree
2. ✅ Click on a VM within that folder → VM details should display correctly
3. ✅ Click on another VM → Details panel should update properly
4. ✅ Click on different folders then VMs → No blank screens should appear
5. ✅ Refresh your browser → Last selected VM should be restored (not the folder)
6. ✅ Navigate to nodes, storage, etc. → Everything should work as before

## What Changed

**File**: `pvemanagerlib.js`

### Change 1: State Persistence Filter (Line ~16431)

**Before**:
```javascript
me.getSelectionModel().on('select', (_sm, n) => sp.set(stateid, { value: n.data.id }));
```

**After**:
```javascript
me.getSelectionModel().on('select', (_sm, n) => {
    // Only persist state for actual resource nodes, not for tag folders or grouping nodes
    if (n.data.type !== 'tag-folder' && !n.data.groupbyid) {
        sp.set(stateid, { value: n.data.id });
    }
});
```

**Explanation**: Filters out tag folders and grouping nodes before persisting selection state. Only actual resources (VMs, containers, nodes) are saved to state.

### Change 2: Selection Change Handler (Line ~60920)

**Before**:
```javascript
selectionchange: function(sm, selected) {
    if (selected.length <= 0) {
        return;
    }
    let treeNode = selected[0];
    let treeTypeToClass = {
        // ... mapping
    };
    // ... setContent call
```

**After**:
```javascript
selectionchange: function(sm, selected) {
    if (selected.length <= 0) {
        return;
    }
    let treeNode = selected[0];
    
    // Skip content creation for tag folders and grouping nodes
    if (treeNode.data.type === 'tag-folder' || treeNode.data.groupbyid) {
        return;
    }
    
    let treeTypeToClass = {
        // ... mapping
    };
    // ... setContent call
```

**Explanation**: Prevents content panel creation when tag folders or grouping nodes are selected. This stops the error `Cannot read properties of undefined (reading 'replace')` that occurred when trying to load help info for non-existent resources.

## Rollback

If you need to rollback to the previous version:

```bash
ssh root@your-proxmox-ip "cp /usr/share/pve-manager/js/pvemanagerlib.js.backup-YYYYMMDD /usr/share/pve-manager/js/pvemanagerlib.js && systemctl restart pveproxy && systemctl restart pvedaemon"
```

Replace `YYYYMMDD` with your backup date.

## Support

This fix addresses the specific issue where VM details don't display after clicking folders. All other tag-based folder functionality remains the same.
