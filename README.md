# Pseudo DC Dimming

[简体中文](README.zh.md) | [下载已适配版本](https://github.com/jawnd/PseudoDCDimming-marble-fixed/releases/latest)

Enable an alternative dimming mode (likely DC-like) on low brightness for some OLED displays by using software brightness gain.

Requires Android 12+ and an Xposed-compatible framework such as LSPosed.

## Redmi Note 12 Turbo / marble compatibility

This repository contains a fix for the Redmi Note 12 Turbo, whose device code is `marble` and model is commonly reported as `23049RAD8C`.

On the tested Android 13 / SDK 33 ROM, the display service did not call the four-argument backlight method expected by the upstream module. It called the HDR-aware six-argument overload instead. Because that overload was not hooked, the module could not receive live brightness callbacks: the UI remained `NaN`, the minimum hardware brightness was not locked, and the software dimming gain was never applied.

The fixed build hooks the six-argument overload first, then falls back to the five- and four-argument overloads used by other Android/vendor framework versions. The HDR flags and factor are passed through unchanged; only the SDR/backlight values used by pseudo-DC dimming are rewritten.

The fix was validated on a rooted Redmi Note 12 Turbo with LSPosed enabled for the `android` process. With the minimum hardware brightness set to 50%, lowering the Android brightness to its minimum kept the hardware request at the 50% threshold while the software gain reduced the visible output below that threshold. A second phone camera confirmed the expected DC-like low-brightness behavior.

If your ROM has a different framework method signature, the same overload-selection logic may need to be adapted. Check `PATCH_NOTES.md` before changing the hook.

## Download and install

[Download the latest fixed release](https://github.com/jawnd/PseudoDCDimming-marble-fixed/releases/latest)

1. Install the APK in the release page.
2. Enable the module in LSPosed and enable it for the `android` process.
3. Reboot the phone.
4. Open the module and choose an acceptable minimum hardware brightness. On the tested Redmi Note 12 Turbo, 50% was used.
5. Use the system brightness slider below that threshold. The module uses Android's software gain to provide the lower visible brightness.

## How it works

By limiting the minimum hardware brightness and scaling down the output signal through a degamma-gain-regamma transform, the actual display brightness matches the expected brightness while maintaining a higher PWM frequency and duty cycle.

[details](details.md)

## Limitations

* It relies heavily on the manufacturer's calibration of screen brightness controls and response curves. If the manufacturer uses different response curves during calibration, or the brightness control has non-linear behavior, display quality may be affected after enabling the module;
* It may conflict with other color transform functions;
* It may conflict with HDR content;
* The Redmi Note 12 Turbo fix was verified on the stated Android 13 / SDK 33 display stack. Other ROM versions may expose a different framework method signature.

## Configuration and verification

Use the high-speed shutter mode of a camera to amplify the stroboscopic effect, or use professional instruments to measure PWM frequency and duty cycle. Choose an acceptable value as the minimum hardware brightness. On the tested Redmi Note 12 Turbo, the hardware minimum was set to 50%, and the system brightness was then lowered to its minimum to verify pseudo-DC operation.

The detailed root cause and patch are documented in [PATCH_NOTES.md](PATCH_NOTES.md).

## Acknowledgements

Inspired by [ztc1997/FakeDCBacklight](https://github.com/ztc1997/FakeDCBacklight). This project additionally implements immediate application as well as stabilizing the brightness before and after enabling.
