# Universal SafetyNet Fix

This is a universal fix for SafetyNet on devices with hardware attestation and unlocked bootloaders. It defeats both hardware attestation and the new SafetyNet ctsProfile updates released on January 12, 2021. The only requirement is that you can pass basic attestation, which requires a valid combination of device and model names, build fingerprints, and security patch levels. Passing basic attestation is out-of-scope for this module.

No device-specific features (such as the new Pixel-exclusive Google Assistant design or screen-off voice match) will be lost with this fix.

MagiskHide is required if the device is rooted.

Android 8.0, 8.1, 9, 10, and 11 are supported. Heavy OEM skins are not officially supported, but it may work on them depending on your luck and the particular ROM in question.

## How does it work?

In order to enforce SafetyNet security, Google Play Services is now
using hardware attestation for ctsProfile validation in all cases, even
when basic attestation is selected. The SafetyNet API response from GMS
will report that basic attestation was used, but under the hood,
hardware attestation is always used regardless of the reported state.
This results in SafetyNet failing to pass due to TrustZone reporting an
unlocked bootloader (and a partially invalidated root of trust) in the
key attestation result.

We can still take advantage of the fact that this usage of hardware
attestation is opportunistic - that is, it falls back to basic
attestation if key attestation fails to run - and prevent GMS from using
key attestation at the framework level. This causes it to gracefully
fall back to basic attestation and pass SafetyNet with an unlocked
bootloader.

Key attestation is still available for other apps, as there are valid
uses for it that do not involve SafetyNet.

The "not implemented" error code from keymaster is used to simulate the
most realistic failure condition to evade detection, i.e. an old device
that lacks support for key attestation.

## ROM integration

Ideally, this workaround should be incorporated in ROMs instead of overriding part of the ROM in a Magisk module.

There are 2 options for:

- Blocking GMS in the framework, which is more portable across Android versions and typically less intrusive for ROMs to integrate
- Blocking GMS in the native keystore service, which is slightly more future-proof and  but may require forking another repository

You only need **one** of the workarounds on the ROM side. Adding both is redundant.

Commits for the framework version of the workaround:

- [Android 11](https://github.com/ProtonAOSP/android_frameworks_base/commit/7f7a9b19c8293c09dfee12bec75ff17225c6710e)

Commits for the native version of the workaround that modifies the C++ keystore service in system/security:

- [Android 11](https://github.com/ProtonAOSP/android_system_security/commit/15633a3d29bf727b83083f2c49d906c16527d389)
- [Android 10](https://github.com/ProtonAOSP/android_system_security/commit/qt)
- [Android 9](https://github.com/ProtonAOSP/android_system_security/commit/pi)
- [Android 8.1](https://github.com/ProtonAOSP/android_system_security/commit/oc)