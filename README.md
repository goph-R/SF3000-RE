Absolutely — here is an updated **INTRO.md** fully adjusted to the **new discoveries**, especially:

* **The SF3000 is a MIPS32 (little-endian) system**, not ARM.
* **Custom code execution confirmed** using a static MIPS binary.
* **Writable SD filesystem**.
* **Launcher mechanism is replaceable/extendable**.

You can paste this directly into your GitHub repo.

---

# SF3000-RE

Reverse Engineering notes and development findings for the **SF3000 retro handheld console**.

## 🎯 Purpose

This repository documents the internals of the SF3000 handheld:

* its Linux-based operating system
* the runtime environment (rootfs + cubegm launcher)
* emulator architecture
* and most importantly:
  **how to run custom MIPS binaries on the device**

All information is for educational and reverse-engineering purposes.

---

# 🧩 System Overview

The SF3000 is a **MIPS32 (MIPS32r2) little-endian** handheld running a lightweight Linux environment built with **Buildroot**.

### `/etc/os-release`

```
NAME=Buildroot
VERSION=2021.05-rc2-65-g51882090d2
ID=buildroot
VERSION_ID=2021.05-rc2
PRETTY_NAME="Buildroot 2021.05-rc2"
```

### Key characteristics

* **CPU:** MIPS32 (LSB / little-endian, MIPS32r2 ABI)
* **Kernel:** Linux 4.4.x
* **Libc:** glibc
* **Dynamic Loader:** `/lib/ld.so.1`
* **Root filesystem:** located on the SD card under `rootfs/`
* **Game launcher environment:** located under `cubegm/`
* **Primary UI:** proprietary LVGL-based binary (`hcprojector`)
* **Game launcher:** `/mnt/sdcard/cubegm/usr/bin/icube`

---

# 📂 Filesystem Structure

The SD card contains several key directories:

```
rootfs/       → Actual system root loaded by kernel (Buildroot)
cubegm/       → Game launcher environment + emulators
roms/ etc.    → User ROM files and media
```

### Important mount point (runtime):

On the device, the SD card is mounted at:

```
/mnt/sdcard
```

Thus:

* `F:\cubegm` on Windows → `/mnt/sdcard/cubegm/` at runtime
* `F:\rootfs` → `/mnt/sdcard/rootfs`

---

# 🕹 Emulator Architecture

Inside `cubegm/cores/` you find all emulator cores as `.so` files:

```
libemu_snes9x.so
libemu_mgba.so
libemu_nes.so
libemu_pce.so
...
```

They are all **MIPS32 shared objects**, dynamically linked with `/lib/ld.so.1`.

The launcher binary:

```
/mnt/sdcard/cubegm/usr/bin/icube
```

loads the appropriate core based on XML config files:

* `cores/config.xml`
* `cores/filelist.xml`

---

# 🎨 UI + Graphics Stack

Two major subsystems exist:

### 1. **System UI (boot menu)**

* Implemented in `/usr/bin/hcprojector`
* Uses **LVGL** (LittlevGL)
* Runs outside the cubegm environment

### 2. **Game UI**

* Implemented by `icube`
* Uses **DirectFB** libraries (present inside cubegm/usr/lib)

There is **no SDL, EGL, or GLES** in the stock system.

---

# 🧪 Custom Code Execution (Working!)

We achieved **full code execution on the SF3000** by replacing the launcher binary with our own MIPS32 static binary.

### Steps & findings:

1. Built a static MIPS test program using:

   ```
   mipsel-linux-gnu-gcc -static
   ```

2. Replaced:

   ```
   cubegm/usr/bin/icube
   ```

   with our own test binary.

3. Device booted → did not launch UI (expected), but:

   * The custom binary **ran successfully**
   * Created a file on the SD card root:

     ```
     test_log.txt
     ```
   * Confirmed the filesystem is writable

This proves:

* Custom MIPS code **executes cleanly on hardware**
* MIPS32 static binaries require **no additional libs**
* The `icube` binary is a valid hook for launching homebrew

---

# 🧠 What We Know Now

| Feature                 | Status                                      |
| ----------------------- | ------------------------------------------- |
| CPU Architecture        | **MIPS32 LSB (MIPS32r2)**                   |
| Writes to SD card       | **Working**                                 |
| Running custom binaries | **Working**                                 |
| Static binaries         | **Work reliably**                           |
| Dynamic binaries        | Must use `/lib/ld.so.1` and the correct ABI |
| SDL2                    | Not included (must be cross-compiled)       |
| OpenGL/EGL              | Not present                                 |
| DirectFB                | Included in cubegm environment              |
| LVGL                    | Used by system UI                           |

---

# 🚀 Development Path Forward

Now that execution works, potential next steps include:

### ✔ Running custom helpers or apps in parallel with `icube`

Modify `icube_start.sh` to run your program first, then start the UI.

### ✔ Framebuffer or DirectFB graphics

Draw directly to `/dev/fb0` or link against DirectFB from cubegm.

### ✔ SDL2 (fbdev backend)

Cross-compile SDL2 for MIPS as a static library.

### ✔ DOSBox / DOSBox Pure

Possible with SDL2 (no JIT on MIPS → interpreter mode, but playable for DOS).

### ✔ LÖVE (Love2D)

Possible but involves building LuaJIT (which is tricky on MIPS), or fallback to Lua 5.1 with non-JIT interpreter.

---

# 🧯 Safety Tips

* Always back up the **entire SD card** (Win32DiskImager recommended)
* Keep a copy of the original `icube` binary
* Use static builds for early testing
* Never delete or modify the kernel partition (not on SD anyway)

---

# 📌 Summary

The SF3000 is a fully reverse-engineerable little MIPS-based Linux handheld.
We now have:

* A confirmed build toolchain
* A confirmed execution hook
* Working SD writes
* Control over the launcher

This repo aims to document all findings and help others run homebrew, custom emulators, or fully replace the launcher.
