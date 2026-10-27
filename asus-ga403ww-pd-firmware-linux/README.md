# ASUS ROG Zephyrus G14 GA403WW — ITE USB-C PD firmware update from Linux

**Tested: 27 October 2026 · Single device · CachyOS · Independent community report**

> [!CAUTION]
> **EXPERIMENTAL FIRMWARE UPDATE — USE AT YOUR OWN RISK.** This is an observed successful procedure on **one** GA403WW, **not** an ASUS-, ITE-, LVFS- or fwupd-approved update method. Incorrect, incompatible or interrupted firmware updates can disable USB-C charging/Power Delivery, require professional repair, or leave a machine unable to boot. **There is no guaranteed recovery method.** Read and independently verify **everything** before considering an update.
>
> All material is provided **AS IS**, without express or implied warranties. To the maximum extent allowed by applicable law, the author disclaims responsibility for device damage, loss of data or use, or other consequences of applying this information. This notice cannot override applicable law or create absolute immunity from liability. **The decision and risk remain with the person performing the update.**

## What actually worked

On an ASUS ROG Zephyrus G14 **GA403WW** (RTX 5080 Laptop), the ASUS **PD_FW 0002** Windows download contained an ITE UEFI FMP capsule. Extracting it and passing it to `fwupdtool install-blob` updated the device's ESRT firmware version from **1** to **53 (0x35)** after a reboot. No Windows or Wine used.

The machine booted, and `nvidia-smi` could access the RTX 5080. This **does not** demonstrate improved PD/charging performance, a solution for NVIDIA GPU Xid 79 / PCIe link loss, or long-term stability.

### The update was genuinely nerve-racking

The progress indicator advanced **extremely slowly** around **6–10%**, with long sequences of repeated or 0.1% incremental readings. The owner was seriously worried that the update had stalled and the laptop might be bricked. It eventually reached 100%, asked for a reboot, and the installed firmware version subsequently read 53. **We did not measure the total duration**; the displayed percentages are **not evidence of physical flash-write duration** (some steps may be staging/processing). Slow progress alone does not establish failure. **Do not interrupt an active firmware update on the basis of an apparently stalled progress bar.**

## Tested setup and immutable identifiers

| Item | Observation |
|---|---|
| Model | ASUS GA403WW |
| OS | CachyOS / Arch Linux |
| BIOS at PD update | `GA403WW.312` |
| Kernel | `7.3.0-5-cachyos` |
| fwupd | `2.1.9-1.1` |
| fwupd-efi | `1.9-1` |
| ESRT and FMP image GUID | `3da51d25-cfe1-3d63-a93d-756e483c67db` |
| ESRT version before → after | **1 → 53** |
| Plugin | `uefi_capsule`, flags included `updatable`, `require-ac`, `needs-reboot` |
| Official ASUS package | **PD_FW Firmware 0002** (2025-10-29) |

Original package: [ASUS GA403WW BIOS and firmware downloads](https://www.asus.com/supportonly/ga403ww/helpdesk_bios/)

- ASUS Windows installer SHA-256: `6D1FE14BC8D82CCE838714AC88E135C14894B386819178AE0397F6A64D5B1C58`
- Embedded archive: `.rsrc/ZIP/103`
- Extracted capsule: `GA403WW-PD_20-0002.cap`
- Capsule SHA-256: `75c9cf4bc53881e770f8311bd23c53152008013625e021053a104dc470f29cd9`
- Capsule size: **484053 bytes**
- Generic FMP capsule GUID: `6dcbd5ed-e82d-4c44-bda1-7194199ad92a`
- FMP image type GUID: `3da51d25-cfe1-3d63-a93d-756e483c67db`
- Firmware INF version: `0x00000035` (53 decimal)

**Do not redistribute ASUS's firmware binary.** This repository documents extraction from the official source.

## Procedure as performed (advanced users; potentially destructive)

**STOP unless:** the exact laptop model is GA403WW, the two hashes match, the capsule's inner FMP image GUID matches the device's ESRT GUID, the firmware is older than target version 53, and fwupd specifically exposes the matching device as updatable. A matching GUID alone is not a universal safety guarantee. On any discrepancy, do not flash. Attach the **original ASUS AC adapter**, charge the battery adequately, save work, and do not bypass warnings with `--force`.

### 1. Check exact model and target version (read-only)

~~~bash
cat /sys/class/dmi/id/product_name
sudo bash -c '
for d in /sys/firmware/efi/esrt/entries/entry*; do
  [[ -r "$d/fw_class" ]] || continue
  [[ $(cat "$d/fw_class") == "3da51d25-cfe1-3d63-a93d-756e483c67db" ]] || continue
  for f in fw_class fw_version last_attempt_version last_attempt_status; do
    printf "%-24s " "$f"; cat "$d/$f"
  done
'
~~~

The test machine returned firmware version **1** before the update. If the target GUID is missing or the model differs, **STOP**.

### 2. Verify ASUS package and extract capsule

Install `7z`, `sha256sum`, `jq`, `fwupd` and `fwupd-efi` first. Set the correct path to your **own** ASUS download (some browsers add suffixes such as `(1)`).

~~~bash
FW="$HOME/Downloads/ASUS_GA403WW_0002_PD_FW_Update.exe"
WORK="$HOME/GA403WW-PD-analysis"
mkdir -p "$WORK/payload"

printf '%s  %s\n' \
  '6D1FE14BC8D82CCE838714AC88E135C14894B386819178AE0397F6A64D5B1C58' "$FW" |
  sha256sum -c -

# Do not proceed if checksum verification failed.
7z x -t'*' -y -o"$WORK" "$FW"
7z e -y -o"$WORK/payload" "$WORK/.rsrc/ZIP/103" GA403WW-PD_20-0002.cap

CAP="$WORK/payload/GA403WW-PD_20-0002.cap"
printf '%s  %s\n' \
  '75c9cf4bc53881e770f8311bd23c53152008013625e021053a104dc470f29cd9' "$CAP" |
  sha256sum -c -
~~~

**Each checksum must return OK.** The Windows INF names the target as ITE PD; `FirmwareVersion=0x35`; the capsule has one FMP image with the same ESRT GUID. The extraction steps do **not** run the ASUS Windows executable.

### 3. Confirm target device ID (read-only)

~~~bash
GUID='3da51d25-cfe1-3d63-a93d-756e483c67db'
sudo fwupdtool get-devices --json |
  jq --arg guid "$GUID" \
  '.Devices[] | select((.Guid // []) | index($guid)) |
   {Name, DeviceId, Guid, Version, VersionRaw, Plugin, Flags}'
~~~

Device ID on **this tested unit**: `e1f9cf857139cd45618af7d948c73cc7dd90d537`. **Do not assume this identifier will be the same on another unit.** This machine reported firmware version 1, plugin `uefi_capsule` and an `updatable` flag.

### 4. Schedule firmware update (RISK: FLASH)

**Only if you have independently completed and verified all checks above and accept the risk.**

~~~bash
sudo fwupdtool install-blob "$CAP" '<YOUR-VERIFIED-DEVICE-ID>'
~~~

Replace `<YOUR-VERIFIED-DEVICE-ID>` with the specific ID for the matching GUID found on **your** machine. Do not use `--force`. This uses a manually extracted capsule rather than a normal LVFS metadata-backed update. The test command reached 100% and requested reboot. Reboot normally with the ASUS adapter connected; never interrupt firmware installation.

### 5. Verify after reboot

Repeat the ESRT command from step 1. The tested unit reported:

~~~text
fw_class                 3da51d25-cfe1-3d63-a93d-756e483c67db
fw_version               53
last_attempt_version     0
last_attempt_status      0
~~~

`fwupdmgr get-history` printed `No history`. The **independently observed ESRT version change (1 → 53)**, not the empty history or last-attempt values, is the main evidence of success.

## Limits of this report

- **Only one GA403WW** observed; no guarantee for other laptops or GA403 variants.
- No published ASUS changelog listing concrete PD firmware fixes.
- No comprehensive USB-C charging/PD functional test conducted.
- NVIDIA Xid 79 and PCIe link-down occurred **before** BIOS/PD maintenance; whether either update changes suspend/GPU reliability is **unknown**.
- BIOS 312 was installed when the PD firmware was updated; it is **not proven to be required**.
- `fwupdmgr get-updates` did not provide this PD update through LVFS.
- AI was used to help draft this report based on **actual terminal output supplied by the device owner**. The assistant did not access or flash the owner's hardware directly.

### Links

- [Official ASUS download](https://www.asus.com/supportonly/ga403ww/helpdesk_bios/)
- [fwupd](https://github.com/fwupd/fwupd)
- [asusctl / rog-control-center](https://github.com/OpenGamingCollective/asusctl)

*This is a test report, not a recommendation to flash firmware. Do not claim vendor support or copy the proprietary ASUS binaries.*