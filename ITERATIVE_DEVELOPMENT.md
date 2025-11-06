# Iterative Development Workflow for VSCodium Integration

**Purpose:** Quick rebuild and test cycle for Kilocode changes without rebuilding entire VSCodium

---

## Quick Development Cycle

### 1. Make Changes to Kilocode

Edit files in `/home/josh/Documents/windsurfinfo/kilocode/src/`

### 2. Build Extension

```bash
cd /home/josh/Documents/windsurfinfo/kilocode
pnpm build
```

**Output:**

- `src/dist/extension.js` and related files
- Build time: ~2 minutes

### 3. Create Test VSIX (Optional)

```bash
cd /home/josh/Documents/windsurfinfo/kilocode/src
pnpm vsix
```

**Output:** `../bin/kilo-code-test-4.115.0.vsix`

### 4. Re-integrate into VSCodium Build

```bash
cd /home/josh/Documents/windsurfinfo/vscodium-unicorn
./integrate_kilocode.sh
```

**What this does:**

- Removes old `vscode/extensions/kilo-code/`
- Copies fresh build from kilocode
- Normalizes package.json (publisher → vscodium)
- Takes ~5 seconds

### 5. Rebuild VSCodium (Incremental)

```bash
cd /home/josh/Documents/windsurfinfo/vscodium-unicorn
./dev/build.sh -s
```

**Note:** `-s` flag skips source download for faster builds (~10-15 min vs 30+ min)

### 6. Test the Build

```bash
cd /home/josh/Documents/windsurfinfo/vscodium-unicorn/VSCode-linux-x64
sudo chown root:root chrome-sandbox && sudo chmod 4755 chrome-sandbox
./codium
```

---

## Even Faster: Development Mode (Extension Testing)

If you just want to test Kilocode changes without full VSCodium integration:

### 1. Build Kilocode

```bash
cd /home/josh/Documents/windsurfinfo/kilocode
pnpm build
```

### 2. Test in Regular VS Code

```bash
cd /home/josh/Documents/windsurfinfo/kilocode
code --extensionDevelopmentPath=$(pwd)/src
```

**Benefits:**

- Instant reload with F5
- Debug with breakpoints
- See console output
- No VSCodium rebuild needed

### 3. When Ready, Integrate into VSCodium

Follow steps 4-6 from Quick Development Cycle above.

---

## Troubleshooting

### Webview "Blocked" Warnings

**Symptoms:** Console shows `Blocked vscode-webview request ...`

**Diagnosis:**

- These are CSP (Content Security Policy) warnings
- Common in VSCodium due to stricter security
- Usually don't break functionality

**Check if Actually Broken:**

1. Open Kilocode sidebar
2. Try creating a task
3. Check if chat panel renders
4. If UI works, warnings are harmless

**If Webview is Actually Broken:**

```bash
# Verify webview assets were copied
ls -la /home/josh/Documents/windsurfinfo/vscodium-unicorn/VSCode-linux-x64/resources/app/extensions/kilo-code/webview-ui/build/

# Should see:
# - index.html
# - assets/ folder with JS chunks
```

### Extension Not Loading

**Check:**

```bash
# Verify extension exists
ls -la /home/josh/Documents/windsurfinfo/vscodium-unicorn/VSCode-linux-x64/resources/app/extensions/kilo-code/

# Verify package.json
cat /home/josh/Documents/windsurfinfo/vscodium-unicorn/VSCode-linux-x64/resources/app/extensions/kilo-code/package.json | jq '.publisher, .name'

# Should output:
# "vscodium"
# "kilo-code"
```

### Build Fails with ENOSPC

**Issue:** Out of disk space

**Fix:**

```bash
# Check available space
df -h /home/josh/Documents/windsurfinfo

# Need at least 13GB free
# Clean up if needed
```

---

## File Locations Reference

| What                  | Location                                                                                                  |
| --------------------- | --------------------------------------------------------------------------------------------------------- |
| Kilocode Source       | `/home/josh/Documents/windsurfinfo/kilocode/src/`                                                         |
| Kilocode Build Output | `/home/josh/Documents/windsurfinfo/kilocode/src/dist/`                                                    |
| Test VSIX             | `/home/josh/Documents/windsurfinfo/kilocode/bin/kilo-code-test-4.115.0.vsix`                              |
| Integration Script    | `/home/josh/Documents/windsurfinfo/vscodium-unicorn/integrate_kilocode.sh`                                |
| VSCodium Build        | `/home/josh/Documents/windsurfinfo/vscodium-unicorn/VSCode-linux-x64/`                                    |
| Integrated Extension  | `/home/josh/Documents/windsurfinfo/vscodium-unicorn/VSCode-linux-x64/resources/app/extensions/kilo-code/` |

---

## Best Practices

1. **Always build Kilocode before integrating**

    ```bash
    cd kilocode && pnpm build
    ```

2. **Use incremental VSCodium builds** (`-s` flag)

    - Saves 15+ minutes per build
    - Only rebuilds changed components

3. **Test in VS Code first** when possible

    - Faster iteration
    - Better debugging
    - No full rebuild needed

4. **Verify integration before full rebuild**

    ```bash
    ./integrate_kilocode.sh
    # Check for "✓ Kilo Code integrated successfully"
    ```

5. **Keep test VSIX separate**
    - Output now named `kilo-code-test-4.115.0.vsix`
    - Won't conflict with production builds

---

## Timeline Reference

| Action                        | Duration       |
| ----------------------------- | -------------- |
| Kilocode build                | ~2 minutes     |
| Create VSIX                   | ~10 seconds    |
| Integration script            | ~5 seconds     |
| VSCodium incremental build    | ~10-15 minutes |
| VSCodium full build           | ~30 minutes    |
| VS Code dev mode (no rebuild) | Instant        |

---

**Last Updated:** November 5, 2025
