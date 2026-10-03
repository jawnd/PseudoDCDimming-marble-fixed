# PseudoDCDimming marble fix

This fork fixes the live-brightness detection path on Xiaomi `marble` / model `23049RAD8C` (Android 13, SDK 33).

## Root cause

The device's `LocalDisplayAdapter.BacklightAdapter` invokes the HDR-aware overload:

`setBacklight(float sdrBacklight, float sdrNits, float backlight, float nits, boolean galleryHdrBoost, float galleryHdrFactor)`

The upstream module only hooked the four-argument overload (and its HyperOS branch looked for a five-argument overload), so the UI stayed at `NaN` and no software gain was applied.

## Fix

`XposedInit` now selects the six-argument overload first, then the five-argument overload, and finally the original four-argument overload. The existing argument rewrite leaves the HDR flags/factor untouched.

## Device validation

- Device: `23049RAD8C` / `marble`
- Android: 13 / SDK 33
- Display path: `useSurfaceControl=true`
- Module: LSPosed, enabled for `android`
- Verified behavior: minimum hardware brightness set to 50%; at the system brightness minimum, the hardware request remains at the threshold and the software gain reduces output below it. Camera observation confirmed DC-like behavior.
