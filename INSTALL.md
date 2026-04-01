# Installation Instructions for Fixed Version

## What This Fix Does

This update fixes the issue where clicking on a tag folder and then clicking on a VM would cause the VM details panel to go blank. The fix prevents tag folders from being saved to application state, allowing proper VM selection after folder navigation.

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
**Line**: ~16431

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

**Explanation**: The fix adds a check to filter out tag folders (`type: 'tag-folder'`) and grouping nodes (`groupbyid`) before persisting selection state. This ensures only actual resources (VMs, containers, nodes) are saved to state, preventing the blank panel issue.

## Rollback

If you need to rollback to the previous version:

```bash
ssh root@your-proxmox-ip "cp /usr/share/pve-manager/js/pvemanagerlib.js.backup-YYYYMMDD /usr/share/pve-manager/js/pvemanagerlib.js && systemctl restart pveproxy && systemctl restart pvedaemon"
```

Replace `YYYYMMDD` with your backup date.

## Support

This fix addresses the specific issue where VM details don't display after clicking folders. All other tag-based folder functionality remains the same.
