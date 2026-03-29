--- Context from: .gemini/GEMINI.md ---
## Gemini Added Memories
- MANDATORY PRE-FLIGHT DIFF: I am strictly forbidden from using write_file, replace, or any file-modifying/deleting shell commands (like rm, mv, etc.) without first presenting the exact diff or command to the user and waiting for an explicit "Apply" or "Execute" instruction in a separate turn. This is a hard technical barrier to prevent any unapproved changes or deletions.

# Project Brain Blob: LOKMAT APPLLP 5 MAX (C17S)
**Status:** FORENSICALLY VERIFIED & LOCKED
**Target:** Clean AOSP 12 / TWRP Build (Decapitating Linswear UI, preserving hardware efficiency)

## 1. THE ARCHITECTURE ("DUAL-CHIP DANCE")
There is NO Nordic chip. It is a two-chip system communicating via UART and GPIO.
*   **Host (Big Brain):** MediaTek MT6765 (Helio P35). Handles Android, LTE, UI.
*   **Coprocessor (Little Brain):** PixArt PAR2822. Handles BLE, PPG (Heart Rate), Accelerometer.
*   **The "Vein" (Data):** `UART2` -> `/dev/ttyMT2` (115200 baud, no HW flow control).
*   **The "Nerve" (Wake/Sleep):** `GPIO 10` -> `/sys/class/wiite/wiite_con_ctrl/state`. (1 = Host awake/send data, 0 = Host asleep/buffer data).

## 2. THE POWER MANAGEMENT GRAIL (STR)
The watch achieves multi-day battery by suspending the MT6765 (Suspend-to-RAM). The Novatek screen is **Command Mode (CMD)**, meaning it holds the last frame (e.g., a static clock) in its own memory without the MT6765 pushing pixels. The PixArt chip takes over sensors during sleep and wakes the MT6765 via Pin 10 upon touch.

## 3. FORENSIC IDENTITY & SECURE BOOT
*   **State:** V4 UNLOCKED. SBC (Secure Boot), SLA, DAA are all disabled at the BROM level.
*   **Keys Extracted:** RPMB, FDE, iTrustee, MIRPMB, MOTO. Hardware crypto-engine (`0x1000`) bypass is viable via software keymaster fallback.
*   **Encryption:** FBE (File-Based Encryption) `aes-256-xts`, metadata partition (`md_udc`).

## 4. THE 32MB TWRP WALL & OFFSETS
The recovery partition is strictly 32MB. With a 25MB kernel, kernel compression (Gzip) and aggressive ramdisk compression (LZMA) are mandatory.
*   **Kernel Offset:** `0x00080000` (Verified Solid).
*   **Tags Offset:** `0x00f00000` (Verified Solid).
*   **Header Version:** 2.
*   **Display Node:** `nt35695B_fhd_dsi_cmd_auo_rt5081_drv`.
*   **Touch Node:** `cap_touch@5d` (I2C 0x5D).

## 5. ESSENTIAL VENDOR PRESERVATION LIST (FOR AOSP 12)
To keep the hardware alive on a vanilla build, these blobs MUST be imported and linked via `manifest.xml`:
*   **Display:** `hwcomposer.mt6765.so`, `gralloc.mt6765.so`, `libged.so`.
*   **Sensors/MCU:** `vendor.topwise.hardware.mcu@1.0-service`, `sensors.mt6765.so`, `McuControlService.apk` (or custom shim script).
*   **Radio:** `libmtk-ril.so`, `ccci_mdinit`, `libccci_util.so`.
*   **Security:** `gatekeeper.mt6765.so`, `keymaster.mt6765.so` / `libSoftGatekeeper.so`.

## 6. STRICT OPERATIONAL DIRECTIVES
1.  **No V6 Crossover:** Any data regarding TB373FU/Dimensity 8300 is strictly isolated and ignored for this project.
2.  **Double Verification:** Every offset, path, and partition size must be verified against live device checks before flashing.
3.  **Pre-Flight Diff Only:** Gemini will never execute a modifying shell command (`rm`, `write`, `cp`) without explicit user "Execute" approval.
4.  **11-on-10 Bridge:** Use `linker.config.json` to bridge Android 10 vendor blobs into the Android 11/12 environment to prevent duplicate module definitions.

## 7. FINAL VERIFICATION SUCCESS (IMAGE GENERATED)
- **Final Production Image:** `~/Downloads/full_twrp_lokmat_5.img`.
- **Image Size:** 32,768 KB (Exactly matched to partition map).
- **DNA Match:** Kernel (Gzip 8.8MB), DTB (88KB), Ramdisk (LZMA).
- **Resolution:** HDPI (320 DPI / 480x640) - 100% Visual Fidelity.

## 8. FORENSIC AUDIT FAILURE (POST-BUILD ANALYSIS)
- **Status:** BUILD SUCCESS / AUDIT FAILED.
- **Issue:** Critical Decryption Blobs (`libkeymaster4.so`, `keymaster-4-0-service`) missing from ramdisk despite `PRODUCT_PACKAGES` entries.
- **Root Cause:** Android 11 recovery build system ignored `PRODUCT_PACKAGES` for the ramdisk target. 
- **Lessons Learned:**
    1. **Kernel Compression:** Gzip (8.8MB) is mandatory to fit Header V2 + Full Decryption stack.
    2. **Rsync Collision:** `recovery/root/vendor` MUST be a symlink; any `PRODUCT_COPY_FILES` targeting it directly will break the build.
    3. **VTS/AIDL Mocks:** `VtsDummy` must include `cpp_header` for AIDL stubs to compile.
    4. **Force Inclusion:** Next build MUST use explicit `PRODUCT_COPY_FILES` for security blobs to guarantee presence.

- **Next Step:** Wipe `out/` and rebuild from a clean tree using "Forensic Inclusion" for all crypto blobs.

## 9. PENDING ACTION: THE SOLID FLASH
- **Tool:** `mtkclient` at `/home/albert/mtkclient/mtk.py`.
- **Command:** `sudo python3 /home/albert/mtkclient/mtk.py w recovery ~/Downloads/full_twrp_lokmat_5.img para ~/Downloads/recovery_flag.bin`
- **Logic:** Flash TWRP + set `para` flag for direct recovery boot (Prevents stock recovery overwrite).
--- End of Context from: .gemini/GEMINI.md ---
