# Phase 1 Integration Issues and Fixes

**Date:** November 5, 2025  
**Status:** Critical Issue Identified and Fixed

---

## Critical Issue: Command Registration Failure

### Problem Statement

After Phase 1 integration, Kilocode extension activated but all button commands failed with:

```
command 'kilo-code.plusButtonClicked' not found
command 'kilo-code.mcpButtonClicked' not found
command 'kilo-code.marketplaceButtonClicked' not found
command 'kilo-code.historyButtonClicked' not found
command 'kilo-code.settingsButtonClicked' not found
command 'kilo-code.popoutButtonClicked' not found
```

### Root Cause Analysis

**The Problem:**
The integration script copied `/kilocode/src/dist/*` contents directly to `/vscode/extensions/kilo-code/`, flattening the directory structure. However, `package.json` still had:

```json
{
	"main": "./dist/extension.js"
}
```

**File Structure Mismatch:**

- **Expected by package.json:** `/kilo-code/dist/extension.js`
- **Actual location:** `/kilo-code/extension.js`

**Result:** VS Code couldn't find the extension entry point, so activate() never ran, and commands were never registered.

### The Fix

**Updated:** `/vscodium-unicorn/integrate_kilocode.sh`

**Change:**

```bash
jq '(.publisher = "vscodium")
    | (.main = "./extension.js")  # ← ADDED THIS LINE
    | (.dependencies = {})
    ...' "${TARGET_DIR}/package.json"
```

**Effect:** Now package.json correctly points to `./extension.js` in the root of the extension folder.

---

## Lessons Learned

### 1. Package Manifest Integrity

**Principle:** When restructuring extension files, always update package.json paths to match.

**Fields to Check:**

- `main` - Extension entry point
- `contributes.iconFonts[].src` - Icon font paths
- `contributes.themes[].path` - Theme file paths
- `contributes.languages[].configuration` - Language config paths
- `contributes.grammars[].path` - Grammar file paths

### 2. Testing Built-in Extensions

**Development vs Built-in:**

- Development: Uses relative paths from workspace
- Built-in: Uses paths from `/resources/app/extensions/`

**Critical Verification:**

- [ ] Extension activates (check VSCodium console)
- [ ] Commands execute (test UI buttons)
- [ ] Webviews load (check for blank panels)
- [ ] Settings apply (verify config changes)
- [ ] No console errors (monitor main process log)

### 3. Integration Script Validation

**Checklist for Future Changes:**

```bash
# After integration script runs:
1. Check extension.js exists at expected location
2. Verify package.json main field points correctly
3. Confirm webview resources at expected paths
4. Test command registration in VSCodium
```

---

## Prevention Strategy

### Pre-Integration Checklist

Before running integrate_kilocode.sh:

- [ ] Kilocode builds successfully
- [ ] VSIX packages without errors
- [ ] Test extension in regular VS Code
- [ ] Verify no hardcoded path assumptions

### Post-Integration Validation

After integration script completes:

```bash
# Verify critical files
ls -la vscode/extensions/kilo-code/extension.js
cat vscode/extensions/kilo-code/package.json | jq '.main, .publisher'

# Expected output:
# "./extension.js"
# "vscodium"
```

### Runtime Testing

After VSCodium build:

```bash
# Launch and test
./VSCode-linux-x64/codium

# Check console for errors
# Test these commands work:
# - kilo-code.plusButtonClicked
# - kilo-code.settingsButtonClicked
# - kilo-code.historyButtonClicked
```

---

## Related Issues to Watch

### Issue #1: Webview Resource Loading

**Status:** Under investigation  
**Symptoms:** "Blocked vscode-webview request" warnings

**Potential Causes:**

- Content Security Policy (CSP) mismatch
- Resource URI resolution (need `webview.asWebviewUri()`)
- Base path assumptions

**Next Steps:**

- Test if webview actually loads despite warnings
- Check CSP headers in webview HTML
- Verify resource paths use proper URI conversion

### Issue #2: Configuration Storage

**Status:** Not yet tested  
**Concern:** Built-in extensions may have different settings scope

**To Verify:**

- Settings persist across restarts
- Workspace vs user settings work correctly
- No permission errors writing config

---

## Phase 2 Preparation: Renaming Concerns

### Comprehensive Renaming Required

When rebranding to "Zenlix" in Phase 2, these items MUST all be renamed consistently:

#### 1. Extension Identity (package.json)

```json
{
  "name": "kilo-code" → "zenlix",
  "publisher": "vscodium" → "zenlix",
  "displayName": "Kilo Code" → "Zenlix"
}
```

#### 2. Command IDs (119 occurrences found)

```
kilo-code.plusButtonClicked → zenlix.plusButtonClicked
kilo-code.mcpButtonClicked → zenlix.mcpButtonClicked
... (117 more commands)
```

**Files Affected:**

- `package.json` (99 commands)
- `src/services/ghost/index.ts` (7 commands)
- `src/core/webview/webviewMessageHandler.ts` (3 commands)
- ... (8 more files)

#### 3. String References (2473 occurrences found)

**Categories:**

- Variable names: `kilocode`, `KiloCode`, `KILOCODE`
- File paths: `.kilocode/`, `/kilocode/`
- URLs: `https://kilocode.ai`
- Comments: "Kilo Code does..."
- Log messages: "KiloCode initialized"

**Strategy:** Not all need renaming (e.g., historical comments OK), but functional references must change.

#### 4. Configuration Keys

**Pattern:** `vscode.workspace.getConfiguration('kilo-code')`

**Must become:** `vscode.workspace.getConfiguration('zenlix')`

**Impact:** User settings will reset unless migrated.

#### 5. File/Folder Names

**Examples:**

- `.kilocode/` → `.zenlix/`
- `kilocode-config.json` → `zenlix-config.json`
- `KILOCODE_API_KEY` → `ZENLIX_API_KEY`

---

## Automated Renaming Tools

### Grep Search Patterns

**Find all command IDs:**

```bash
grep -r "kilo-code\." src/ --include="*.ts" --include="*.json"
```

**Find configuration keys:**

```bash
grep -r "getConfiguration.*kilo" src/ --include="*.ts"
```

**Find string literals:**

```bash
grep -ri "kilocode" src/ --include="*.ts" --include="*.tsx"
```

### Replacement Strategy

**Safe Approach:**

1. Create rename script with all find/replace pairs
2. Run script on copy of codebase
3. Build and test thoroughly
4. Compare with original to verify changes
5. Apply to actual codebase

**High-Risk Changes:**

- Command IDs (break all UI buttons if wrong)
- Configuration keys (lose user settings)
- File paths (break file operations)

**Low-Risk Changes:**

- Comments
- Display strings
- Documentation

---

## Recommended Phase 2 Workflow

### Step 1: Inventory

- Generate complete list of all items to rename
- Categorize by risk level (high/medium/low)
- Create verification test for each category

### Step 2: Rename Script

```bash
#!/bin/bash
# phase2-rename.sh

# High-risk: Command IDs
find src -name "*.ts" -exec sed -i 's/kilo-code\./zenlix./g' {} +

# High-risk: Extension name
sed -i 's/"kilo-code"/"zenlix"/g' src/package.json

# Medium-risk: Config keys
find src -name "*.ts" -exec sed -i "s/getConfiguration('kilo-code')/getConfiguration('zenlix')/g" {} +

# ... (more replacements)
```

### Step 3: Verification

- Run TypeScript compiler (catch reference errors)
- Run all tests
- Build VSIX and test in VS Code
- Manual UI testing (all buttons work)
- Settings migration test

### Step 4: Integration

- Rebuild with new name
- Update integrate_kilocode.sh (rename to integrate_zenlix.sh?)
- Test in VSCodium build
- Document breaking changes for users

---

## Testing Checklist for Phase 2

### Pre-Rename

- [ ] All existing tests pass
- [ ] Extension activates correctly
- [ ] All commands work in UI
- [ ] Settings load properly

### Post-Rename

- [ ] TypeScript compiles without errors
- [ ] All tests still pass (update test names)
- [ ] Extension activates with new name
- [ ] Commands work with new IDs
- [ ] Settings persist (or migration works)
- [ ] No "kilo-code" references in UI
- [ ] VSCodium build succeeds
- [ ] Runtime testing shows no errors

---

## Documentation Updates Needed

### For Phase 2

**Update these files:**

- `/kilocode/.docs/` - All references to name/branding
- `/integration/phase2-zenlix-rebrand/` - Actual rename process docs
- `README.md` - Project name and branding
- `CONTRIBUTING.md` - Command examples
- User guides - All screenshots and examples

---

**Last Updated:** November 5, 2025  
**Next Review:** Before Phase 2 implementation begins
