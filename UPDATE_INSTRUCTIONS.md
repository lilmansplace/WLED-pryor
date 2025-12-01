# Updating Custom WLED Builds for New Releases

This guide explains how to sync your custom WLED fork when a new stable release is published upstream.

## Prerequisites

- Git installed locally
- Your fork cloned: `git clone https://github.com/lilmansplace/WLED-pryor.git`
- Upstream remote configured (should already be set up)

## Step 1: Check Current Version
```bash
cd WLED-pryor
git checkout custom-builds
git describe --tags
```

This shows your current base version (e.g., `v0.15.2-2-g1a2b3c4`).

## Step 2: Fetch Latest Release from Upstream
```bash
# Add upstream remote if not already added
git remote add upstream https://github.com/Aircoookie/WLED.git

# Fetch all tags and branches from upstream
git fetch upstream --tags

# List the latest stable releases
git tag -l "v0.*" | sort -V | tail -5
```

Look for the latest stable release tag (e.g., `v0.15.3`, `v0.16.0`). Avoid beta/RC tags unless you want bleeding edge.

## Step 3: Rebase Your Custom Branch

**⚠️ Important:** This rewrites history, so make sure you're on the `custom-builds` branch.
```bash
# Make sure you're on the custom-builds branch
git checkout custom-builds

# Rebase onto the new release (replace v0.16.0 with actual version)
git rebase v0.16.0
```

### If There Are Conflicts

If you see merge conflicts (unlikely with just `platformio_override.ini`):
```bash
# Check which files have conflicts
git status

# Edit the conflicted files, then:
git add <fixed-file>
git rebase --continue
```

If things go wrong, you can abort:
```bash
git rebase --abort
```

## Step 4: Push Updated Branch

Since rebase rewrites history, you need to force push:
```bash
git push origin custom-builds --force
```

**⚠️ Warning:** `--force` is necessary but overwrites remote history. Only use this on your custom branch, never on `main`.

## Step 5: GitHub Actions Build

Once pushed, GitHub Actions will automatically:
1. Build `WLED_SAL001S` (ESP8266)
2. Build `WLED_drchop_corner_lamp` (ESP32-C3)

Monitor the build at: https://github.com/lilmansplace/WLED-pryor/actions

## Step 6: Download Firmware

1. Go to the Actions tab
2. Click on the latest successful workflow run
3. Scroll to **Artifacts** section
4. Download:
   - `firmware-WLED_SAL001S`
   - `firmware-WLED_drchop_corner_lamp`

The `.bin` files inside are ready to flash!

## Troubleshooting

### Build Fails After Update

If the build fails after updating to a new version:

1. **Check the workflow run logs** for specific errors
2. **Common issues:**
   - PlatformIO configuration changes
   - Deprecated variables in `platformio_override.ini`
   - New dependencies or build flags

3. **Compare with upstream:**
```bash
   # See what changed in platformio.ini
   git diff v0.15.2 v0.16.0 -- platformio.ini
```

4. **Update your override file** if needed and commit:
```bash
   git add platformio_override.ini
   git commit -m "Update override for v0.16.0 compatibility"
   git push
```

### Verify the Rebase Worked
```bash
# This should show the new version
git describe --tags

# This should show your custom commit(s) on top of the new release
git log --oneline -5
```

## Quick Reference Commands
```bash
# Complete update in one go (after verifying new version):
cd WLED-pryor
git checkout custom-builds
git fetch upstream --tags
git rebase v0.16.0  # Replace with actual new version
git push origin custom-builds --force
```

## Maintaining Multiple Device Configs

Your current setup builds for:
- **WLED_SAL001S**: ESP8266 NodeMCU with rotary encoder usermod
- **WLED_drchop_corner_lamp**: ESP32-C3 corner lamp

To add more devices, edit `platformio_override.ini` and add new `[env:device_name]` sections. They'll automatically build via GitHub Actions.

## Notes

- This workflow keeps your `main` branch clean and untouched
- All customizations are in the `custom-builds` branch
- Only `platformio_override.ini` should have changes (easy to maintain)
- GitHub Actions builds on every push to `custom-builds`

## Support

For WLED issues: https://github.com/Aircoookie/WLED/issues  
For build/workflow issues: Check this repo's Actions tab or create an issue
