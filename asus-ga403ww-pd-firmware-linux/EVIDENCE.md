## Verified data (27 October 2026)

ASUS GA403WW, CachyOS, BIOS GA403WW.312, kernel 7.3.0-5-cachyos, fwupd 2.1.9-1.1, fwupd-efi 1.9-1.

- Target ESRT GUID: `3da51d25-cfe1-3d63-a93d-756e483c67db`
- Firmware before: **1**; after reboot: **53**
- ASUS EXE SHA256: `6D1FE14BC8D82CCE838714AC88E135C14894B386819178AE0397F6A64D5B1C58`
- Extracted CAP SHA256: `75c9cf4bc53881e770f8311bd23c53152008013625e021053a104dc470f29cd9`
- CAP FMP image GUID matches ESRT GUID; INF firmware version 0x35
- `fwupdmgr get-history`: `No history`; ESRT last_attempt_version 0 and last_attempt_status 0
- `fwupdtool install-blob` reached 100% and requested reboot; after reboot version read 53
- Progress appeared almost stalled near 6–10%; this caused serious concern. Exact duration not measured.
- GPU responds to nvidia-smi after reboot; no evidence of cured PCIe/Xid 79 issue.

Single-device observation only. No comprehensive PD functional testing or ASUS approval. Data supplied from the owner's actual terminal, writing assisted by AI.
