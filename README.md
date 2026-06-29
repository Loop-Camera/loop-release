# loop-release

Public distribution point for **Loop Camera** firmware images.

This repo holds no source — only published firmware builds, as
[GitHub Releases](../../releases). The [Loop app](https://github.com/garagehq/loop-app-beta)
polls the **latest** release here to discover and pull over-the-air (OTA)
updates for the camera. Source lives in the private
[`loop-firmware`](https://github.com/Loop-Camera/loop-firmware) repo.

## How a release is produced

When a pull request is merged into `main` in `loop-firmware`, its
`Build & Publish Release` workflow:

1. builds the ESP32-S3 camera firmware (PlatformIO),
2. builds the ESP32-P4 main firmware (ESP-IDF) with the S3 image **embedded**
   (for OTA-over-SPI of the cameras),
3. generates a changelog from the merged PR, and
4. publishes a Release **here** with the image assets attached.

## Release assets

Each release attaches:

| Asset | Purpose |
| --- | --- |
| `loop.bin` | the P4 application image — **this is what the app OTAs** |
| `loop-merged.bin` | full-flash image (bootloader + partitions + app) for fresh devices |
| `loop-bootloader.bin` | 2nd-stage bootloader (manual flashing) |
| `loop-partition-table.bin` | partition table (manual flashing) |
| `latest.json` | machine-readable manifest the app reads |

## How the app consumes it

The "latest" release is always reachable at a stable, unauthenticated URL
(this repo is public), so the app needs no token:

```
GET https://github.com/Loop-Camera/loop-release/releases/latest/download/latest.json
```

```jsonc
{
  "version": "1.0.0",        // firmware semver (from esp32p4/CMakeLists.txt)
  "tag": "v1.0.0.42",        // unique release tag (version + CI run number)
  "bin": "loop.bin",
  "sha256": "…",             // verify the download before flashing
  "size": 4012345,
  "url":  "https://github.com/Loop-Camera/loop-release/releases/download/v1.0.0.42/loop.bin",
  "notes_url": "https://github.com/Loop-Camera/loop-release/releases/tag/v1.0.0.42"
}
```

The app compares the device's reported firmware version (read over the device
API) against `version`/`tag`; if they differ, it downloads `url`, verifies
`sha256`, and streams it to the camera in chunked OTA. The GitHub Releases API
(`GET /repos/Loop-Camera/loop-release/releases/latest`) is an equivalent
source for richer metadata (notes, published date).
