# Implementation Plan: Add Windows Platform Build

## Context

**Date**: 2026-03-23
**Task**: Enable Windows platform builds in GitHub Actions CI/CD workflow
**Approach**: Solution 1 - Windows + macOS (Dual-Platform)
**Code Signing**: Disabled (unsigned builds)

## Requirements

### User Request
增加windows平台的构建 (Add Windows platform build)

### Requirement Completeness Score: 8/10
- ✅ Goal Clarity: 3/3 - Enable Windows platform build in CI/CD
- ⚠️ Expected Results: 2/3 - Generate Windows artifacts (.exe)
- ✅ Scope Boundaries: 2/2 - Modify GitHub Actions workflow
- ⚠️ Constraints: 1/2 - No code signing (unsigned builds)

### Decision
- Enable Windows builds **without code signing**
- Users will see SmartScreen warnings
- Code signing can be added later if needed

## Solution Design

### Selected Approach
**Solution 1: Windows + macOS (Dual-Platform)**

**Rationale**:
1. Directly addresses user request for Windows builds
2. Low risk - minimal change to working configuration
3. Cost-effective - balances platform coverage with CI costs
4. Windows build infrastructure already exists in workflow
5. Can easily add Linux later if needed

### Alternative Approaches Considered
- **Solution 2**: All three platforms (Linux + macOS + Windows) - Higher CI cost
- **Solution 3**: Gradual rollout with manual triggers - More complex

## Implementation Details

### File Modified
`.github/workflows/build_private_repo.yml`

### Changes Made

#### Line 41: Update OS Matrix Configuration
```yaml
# BEFORE:
os: [macos-latest]

# AFTER:
os: [macos-latest, windows-latest]
```

### Verification Steps

#### ✅ Build Step Configuration (Lines 86-90)
```yaml
- name: Build/release Electron app (Windows)
  if: matrix.os == 'windows-latest'
  run: npm run build:win
  env:
    GH_TOKEN: ${{ secrets.PRIVATE_REPO_PAT }}
```

#### ✅ Release Assets Configuration (Line 126)
```yaml
files: |
  release/*.exe        # Windows installers
  release/*.blockmap   # Update metadata
  release/*.yml        # Update configuration
```

## Expected Behavior

### Build Triggers
Windows builds will be triggered on:
1. **Tag pushes**: `v*.*.*` (e.g., `v1.0.0`)
2. **Manual dispatch**: `workflow_dispatch` event
3. **Scheduled runs**: Daily at UTC 20:00 (Beijing 04:00)

### Build Artifacts
For each Windows build:
- `release/*.exe` - NSIS installer (executable)
- `release/*.exe.blockmap` - Update metadata for electron-updater
- `release/latest.yml` - Auto-update configuration

### Parallel Execution
- macOS and Windows builds run simultaneously
- Estimated total CI time: 15-25 minutes

### User Experience (Unsigned Builds)
⚠️ **Important**: Without code signing, users will encounter:
1. Windows SmartScreen warning: "Windows protected your PC"
2. Required action: Click "More info" → "Run anyway"
3. Warning message: "Unknown publisher"

**To improve**: Purchase code signing certificate (~$200/year) and configure:
- `WIN_CSC_LINK` secret (base64-encoded certificate)
- `WIN_CSC_KEY_PASSWORD` secret (certificate password)

## Testing Strategy

### Post-Deployment Verification
1. **Trigger test build**: Use manual `workflow_dispatch` on test branch
2. **Monitor execution**: Check both macOS and Windows job logs
3. **Verify artifacts**: Confirm `.exe` files appear in release assets
4. **Test installation**: Download and install Windows `.exe` on test machine
5. **Confirm warnings**: Verify SmartScreen warning behavior

### Success Criteria
- ✅ Both macOS and Windows builds complete successfully
- ✅ Windows `.exe` installers generated
- ✅ Artifacts uploaded to GitHub releases
- ✅ No errors in CI logs

## Rollback Plan

If issues occur, revert the change:

```yaml
# In .github/workflows/build_private_repo.yml line 41:
os: [macos-latest]
```

## Future Enhancements

### Optional Improvements
1. **Add Code Signing**
   - Purchase OV/EV certificate from DigiCert, Sectigo, or SSL.com
   - Configure `WIN_CSC_LINK` and `WIN_CSC_KEY_PASSWORD` secrets
   - Eliminates SmartScreen warnings

2. **Enable Linux Builds**
   - Change matrix to: `os: [ubuntu-latest, macos-latest, windows-latest]`
   - Provides complete platform coverage

3. **Optimize CI Costs**
   - Use conditional matrix for different trigger types
   - Run full builds only on tags, limited builds on manual triggers

## Summary

### Changes Summary
| File | Lines | Change Type | Description |
|------|-------|-------------|-------------|
| `.github/workflows/build_private_repo.yml` | 41 | Modification | Add `windows-latest` to OS matrix |

### Impact Analysis
- **Risk Level**: Low (single line change, existing infrastructure)
- **CI Cost Impact**: ~2x (from 1 to 2 parallel jobs)
- **User Impact**: Windows users can now download native builds
- **Maintenance**: No additional maintenance required

### Implementation Status
✅ **COMPLETED** - 2026-03-23

---

*This plan was created following the ZCF Workflow structured development process.*
