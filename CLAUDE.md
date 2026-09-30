# esp32-storage

The firmware to flash on a spare ESP32 before shelving it, and a bench tool
once it's taken out again. It contains the Wi-Fi setup AP, web UI, OTA,
Files, Hardware (I2C/GPIO), Serial monitor, Log, Clock, and the serial
console (`gw wifi …`, `gw password reset`). See README.md.

It's a thin app on top of [garnet_web](../garnet_web) and
[garnet_settings](../garnet_settings), which are symlinked from `../`. Almost
all behaviour lives there; this repo is `src/main.cpp` (the Board group,
Ethernet-from-variant, the serial status line) plus `platformio.ini`
(the per-chip tool sets). Library changes belong in garnet_web, including
anything about the web UI, the tools or the serial console.

## Rules

- **Size is the constraint.** Every env must fit `default.csv`'s 1.25 MB OTA
  slot (1,310,720 bytes), measured on the `.bin`, not PlatformIO's
  percentage: segment padding makes the `.bin` bigger. The C5 has ~1 KB
  left and the C6 ~4 KB. A new tool means choosing what a chip leaves out
  (`[tools]` sets), not just enabling it.
- **Keep `default.csv`.** OTA can't change the partition table, so the
  firmware installed later must use the same table; the standard one is
  the least surprising.
- **Touch no pins** until the user asks (the tools do that), except
  Ethernet on boards whose variant defines it.
- Same pinned pioarduino release as garnet_web's examples (see
  garnet_web/CLAUDE.md for why), and the same PlatformIO lib_ignore
  warning: never add `lib_ignore`, since pioarduino rewrites the shared
  framework build script.
- The ESP32-P4 env is Ethernet-only until its C6 is reflashed (README).

## Verifying a change

```
pio run                                   # all envs, then check .bin sizes:
for e in esp32 esp32s2 esp32s3 esp32c3 esp32c5 esp32c6 esp32p4; do
  stat -f "%z $e" .pio/build/$e/firmware.bin; done
```

Flash one board and check the serial status line
(`[storage] ESP32 setup AP  id … http://192.168.4.1/`) and `gw status`.
