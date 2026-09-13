# Volume Bonus Double-Count Fix + Dead Code Cleanup

**Date:** 2026-09-13
**Scope:** `src/compute/patterns/index.ts`

## Changes

### 1. Removed volume double-counting in `applyConfidenceHierarchy`

**Problem:** `applyConfidenceHierarchy` added a flat `+0.1` volume bonus when `p.volumeConfirmed` was true. However, every context-aware detector already factors volume into its computed confidence via `volumeFactor(volumeRatio)` (single/double patterns) or explicit volume ratio gates (triple patterns). The `volumeConfirmed` flag itself is derived from the same volume ratio the detector already used. This caused volume to be counted twice — once inside the detector's confidence formula, and again as a flat additive bonus on top.

**Fix:** Removed the `volumeBonus` addition from `applyConfidenceHierarchy`. The function now simply applies the confidence floor from `PATTERN_CONFIDENCE_HIERARCHY` via `Math.max`, without an additional volume bonus. Detectors remain the single source of truth for volume's effect on confidence.

### 2. Removed dead `patternDirection` function

**Problem:** `patternDirection` was exported but never imported or called anywhere in the codebase. It also had an incorrect default (`return 'buy'`) for all SMC/structural patterns (mean-reversion, FVG, OB, liquidity-sweep, harmonic, etc.), which would have returned wrong directions for bearish variants of those patterns had it ever been used.

**Fix:** Deleted the function entirely.

## Verification

- `npm run typecheck` — pass
- `npm run test` — 834/834 pass
