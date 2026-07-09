# Layout Profiles

Manufacturer-independent layout definitions, keyed by screen resolution and
card-slot count. Introduced in this fork to stop duplicating identical
substitution blocks across devices with the same screen geometry.

## Why this is safe

As of espcontrol v2.5.0, every existing device sharing a given
(`screen_width`, `screen_height`, slot count) combination had **byte-identical**
values for all 31 layout substitutions listed below — this was verified
programmatically against the full device set before extracting these files
(see the fork's commit history for the comparison script). No device's
behavior changes when opting into a profile; the fully-resolved ESPHome
configuration is provably identical before and after (multiset-compared line
by line).

## How a device opts in

In `devices/manifest.json`, set `firmware.package.profileRef` to the profile
name (without `.yaml`), e.g.:

```json
"package": {
  "profileRef": "profile-1280x800-20slots",
  ...
}
```

`scripts/generate_device_slots.py` then:
- Omits the 31 profile keys from the device's own substitutions block
- Inserts `!include ../../profiles/<profileRef>.yaml` as the first package

Devices without `profileRef` are completely unaffected — verified: running
the generator against all four untouched upstream devices produces
byte-identical output before and after this change.

## What's in a profile

Pure layout: screen dimensions, content width, setup-screen button sizing,
font *references* (not the fonts themselves — those stay in each device's
`device/fonts.yaml`), spacing/padding/radius, main-grid gap and top padding,
clock bar geometry, and screensaver/schedule clock brightness defaults.

What's deliberately NOT in a profile (stays device-specific):
- `device_slug`, `firmware_manifest_slug`, `firmware_version`
- Voice assistant chime file URLs
- Cover-art layout (varies more with panel aspect ratio than these 31 keys did)
- Ethernet/network substitutions

## Current profiles

| Profile | Resolution | Slots | Used by |
|---|---|---|---|
| profile-1280x800-20slots | 1280x800 | 20 | waveshare-esp32-p4-10-1, (upstream: jc8012p4a1) |
| profile-720x720-9slots | 720x720 | 9 | esp32-p4-86-lite, (upstream: esp32-p4-86) |
| profile-1024x600-15slots | 1024x600 | 15 | (upstream only: jc1060p470) |
| profile-480x800-6slots | 480x800 | 6 | (upstream only: jc4880p443) |
| profile-480x480-9slots | 480x480 | 9 | (upstream only: guition-esp32-s3-4848s040) |

The three upstream-only profiles exist so a *future* device (e.g. a new
panel with a resolution espcontrol already supports) can reuse them without
re-deriving the values — per Stefan's plan to add "bring your own hardware"
support for new panel purchases, matched against an existing resolution.
The four original upstream devices themselves are NOT wired to
`profileRef` yet (out of scope for this step, see fork discussion) and
continue to carry their own inline substitutions unchanged.
