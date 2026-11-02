# ESP32-S3 virtual screen (experimental)

This profile runs the normal 320x240 LVGL interface on a bare ESP32-S3 N16R8.
It does not initialize a physical LCD or backlight. Frames are sent over
UART0 to the existing GhostESP Dashboard Mirror and its D-pad uses the
existing `input` commands.

## Hardware and connection

- ESP32-S3 with 16 MB flash and 8 MB octal PSRAM.
- Connect through the USB-to-UART bridge (tested with CH343), at 115200 baud.
- This experimental profile sends Mirror frames to UART0, not the native
  USB Serial/JTAG port. Physical display board profiles were not tested.
- Virtual mode bypasses the physical backlight timeout/wake-lock flow so
  a remote session cannot become stranded behind a hardware wake action.

## Build

Use ESP-IDF 6.1 and its supported Xtensa GCC 15.2 toolchain. In a clean checkout:

```
cp configs/sdkconfig.virtual.esp32s3 sdkconfig.defaults
idf.py set-target esp32s3
idf.py -D CMAKE_NINJA_FORCE_RESPONSE_FILE=ON build
```

The response-file setting avoids Windows command-line length limits.
The profile uses the existing 16 MB A/B partition layout. Back up the full
flash before changing from a generic 4 MB layout; switching layouts overwrites
the old application and coredump regions. No full-chip erase is required.

## Device validation, 2026-11-02

- Full 16 MB original flash backed up locally before loading.
- Firmware compiled and partition size check passed (3,585,728-byte app).
- Bootloader, partition table, OTA data, and app written and hash-verified.
- Device reported `virtual.esp32s3`, ESP-IDF 6.1, and about 8.3 MB free heap.
- Serial Mirror: 320x240, 62 decoded region packets, zero framing/payload errors.
- `input right` and `input select` skipped setup and reached the main menu.
- Reconnected Mirror: 63 decoded region packets, zero framing/payload errors.

These counts are LVGL region packets, not complete frames per second.
Validation used the serial protocol directly and reconstructed its RGB332
image. The live Brave Dashboard session, long-duration stability, other boards,
native USB mirroring, radio operations, and SD hardware remain unverified.
