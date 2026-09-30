# M5Stack NanoC6 Aliro NFC Reader

Experimental Aliro-compatible NFC reader and Matter door-lock reference for the
[M5Stack NanoC6](https://docs.m5stack.com/) and
[M5Stack Unit NFC](https://docs.m5stack.com/en/unit/Unit_NFC).

This project adapts Espressif's `esp-matter` door-lock example for the NanoC6,
provisions Apple Home Key credentials through Matter, and attributes a successful
Aliro NFC unlock to the corresponding Matter user and credential.

## What works

- Matter commissioning over Thread
- Apple Home Key provisioning through Apple Home
- Aliro NFC standard and fast transactions
- M5Stack Unit NFC over I2C using its ST25R3916 controller
- Matter user and endpoint-credential persistence
- Aliro credential attribution without logging raw keys
- A Matter `LockOperation` event containing the resolved `userIndex`, credential
  type, and credential index
- Automatic relocking from the upstream example

The tested credential was resolved as an Aliro evictable endpoint key. The
implementation also handles non-evictable endpoint keys.

## Tested stack

| Component | Tested version |
| --- | --- |
| Hardware | M5Stack NanoC6 + M5Stack Unit NFC |
| ESP-IDF | `v6.0.2` |
| esp-matter | `59574f3fb62fddc690b4127fd3a4a43cfce4245e` |
| connectedhomeip submodule | `539342f32d5f4dc93761c2f9325afe29270068f1` |
| esp-aliro NFC component | `db6bc8ea93854b730b9163d2472dfa7bff86a280` |
| esp_aliro_lib | `^1.2.0` |

The reference build uses GPIO 2 for SDA, GPIO 1 for SCL, GPIO 9 for the button,
and GPIO 20 for the RGB LED.

## Repository layout

- `patches/0001-nanoc6-aliro-credential-attribution.patch` contains the
  `esp-matter` door-lock changes.
- `config/sdkconfig.defaults.nanoc6_aliro_nfc` contains the tested ESP32-C6,
  Thread, console, board, and NFC configuration.
- `SECURITY.md` documents the security boundary and production caveats.

No firmware image, flash dump, Matter fabric data, PIN, private key, persistent
Aliro key, network address, or commissioned-device state is included.

## Build

Install ESP-IDF 6.0.2 and the prerequisites required by `esp-matter`, then:

```bash
git clone --recursive https://github.com/espressif/esp-matter.git
cd esp-matter
git checkout 59574f3fb62fddc690b4127fd3a4a43cfce4245e
git submodule update --init --recursive

git apply /path/to/m5stack-nanoc6-aliro-reader/patches/0001-nanoc6-aliro-credential-attribution.patch
cp /path/to/m5stack-nanoc6-aliro-reader/config/sdkconfig.defaults.nanoc6_aliro_nfc \
  examples/door_lock/sdkconfig.defaults.nanoc6_aliro_nfc

source /path/to/esp-idf-v6.0.2/export.sh
cd examples/door_lock
idf.py -D SDKCONFIG_DEFAULTS=sdkconfig.defaults.nanoc6_aliro_nfc set-target esp32c6
idf.py build
idf.py -p /dev/your-device-port flash monitor
```

The ESP-IDF component manager downloads the declared Aliro and M5Stack NFC
dependencies during configuration.

## Commissioning and validation

1. Flash the firmware and use the Matter onboarding information printed by the
   example to add the lock to Apple Home.
2. Allow Apple Home to provision Home Key credentials.
3. Present the iPhone or Apple Watch to the NFC reader.
4. Confirm that the serial log reports a successful Aliro transaction and a
   resolved Matter user and credential index.

Expected log shape, with example-only identifiers:

```text
Aliro NFC transaction completed successfully
Aliro credential resolved: user=<user-index> credential-type=<type> credential-index=<index>
Unlock event: source=aliro user=<user-index> credential-type=<type> credential-index=<index>
```

Raw credential data is deliberately not printed.

The diagnostic console can inspect capacity and user-to-credential references:

```text
matter esp dl status
matter esp dl users
```

## Attribution design

The Aliro SDK stores the public access-credential key associated with a fast
transaction in its NVS-backed persistent-key cache. After a successful
transaction, the patch locates the most recently used cache entry, reads only
its public key, and compares it in memory with the enabled Matter users'
provisioned Aliro endpoint credentials. A successful match is attached to the
Matter `LockOperation` event.

This avoids exposing credential material in logs while making the event useful
to a Matter controller or higher-level automation system.

## Important limitations

- This is a reference implementation, not a production access-control product.
- The attribution code currently relies on the internal NVS schema used by the
  tested `esp_aliro_lib` version. Revalidate it after any SDK upgrade.
- The upstream door-lock example simulates actuator movement. Add independently
  reviewed hardware, safety, and failure handling before controlling a real door.
- Home Assistant and alarm-system integration are intentionally outside this
  repository.
- Aliro controller behavior and credential layout may vary across ecosystems and
  future software releases.

## License and attribution

Licensed under Apache License 2.0. The patch modifies Espressif's `esp-matter`
door-lock example. See `NOTICE` and the upstream project licenses for details.
