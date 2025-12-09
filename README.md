# SF3000-RE

Reverse Engineering notes and findings about the SF3000 handheld console.

## 🎯 Goal

This project documents the internal software structure of the **SF3000 retro handheld**, with the aim of understanding how the system works, how games are launched, and how to run custom software (e.g. DOSBox Pure, SDL apps, etc.).

This is **not** a hacking firmware replacement project, but a research project focused on learning and documenting.

---

## 🧩 Hardware / OS Summary

The SF3000 uses:

* **Buildroot Linux**
* **Linux Kernel 4.4**
* **glibc (not musl)**
* **aarch64 architecture**
* custom vendor UI built with **LVGL**
* secondary game launcher environment called **cubegm**

### `/etc/os-release`

```
NAME=Buildroot
VERSION=2021.05-rc2-65-g51882090d2
ID=buildroot
VERSION_ID=2021.05-rc2
PRETTY_NAME="Buildroot 2021.05-rc2"
```

So the OS is basically a small embedded Linux distribution built using Buildroot.

---

## 🧠 Software Architecture (important!)

### Boot UI

* Main UI process is `/usr/bin/hcprojector`
* It uses **LVGL** for the GUI
* Started from `/etc/cubeapp_start.sh`
* If the UI dies, a secondary launcher starts

### Game Launcher

* Located under the SD card folder: `/cubegm`
* This acts like a “mini rootfs”:

  * emulator cores under `/cubegm/cores`
  * DirectFB libs under `/cubegm/usr/lib`
  * launcher binaries under `/cubegm/usr/bin`

### Emulators

* Each console emulator is a shared library like:

  ```
  libemu_snes9x.so
  libemu_mgba.so
  libemu_mame2000.so
  ...
  ```
* Launcher (`icube`) loads them dynamically

### Frontend

* Game launcher binary:

  ```
  cubegm/usr/bin/icube
  ```
* Likely responsible for reading config + loading libemu_*.so

---

## 🧵 Important technologies inside

### UI:

* **LVGL** (visible through exported symbols in hcprojector)

### Rendering:

* **DirectFB**, not SDL/OpenGL
* No visible GLES/EGL system libraries
* Means emulators are statically compiled with their own dependencies

---

## 📁 Configuration files

* `/cubegm/cores/config.xml`
  Maps emulator names to `.so` files

* `/cubegm/cores/filelist.xml`
  Maps game ROM file names to cores

These XML files define how the launcher picks a core for each game.

---

## ❓ Can we run custom software?

**Yes, almost certainly.**

### Methods:

* Modify `/cubegm/icube_start.sh`
* Replace a system emulator you don’t need with your own launcher
* Add your own script/binary and launch it before/after `icube`

Because everything important is on the SD card, this is very mod-friendly.

---

## 🚧 What is missing

* No shared SDL or GLES libraries
* No RetroArch
* Emulators seem to be fully vendor-provided
* Custom software must bring its own libs (static linking)

For example:

* DOSBox Pure will need SDL2 compiled for aarch64
* LÖVE (Love2D) needs SDL2 + Lua (static ideally)

---

## 🔨 Building your own stuff

Best approach:

* Use Windows + WSL2
* Install aarch64 cross-compiler
* Build a small “hello world”
* Launch via script inside `/cubegm`

Once that works, move toward SDL/DOSBox/LÖVE.

---

## 🧯 Safe Backup

Recommended:

* Create a full disk image of the SD card using Win32DiskImager
* Also copy the directories manually

This protects you against boot failures if you experiment.

---

## 🚀 Next Steps (planned)

* Cross-compile aarch64 “hello world”
* Insert into `icube_start.sh`
* Verify execution
* Build SDL2 statically
* Build DOSBox Pure / LÖVE dynamically after that

---

## ✨ Purpose of this repo

To document:

* system structure
* binaries
* config
* launch scripts
* reverse engineering progress
* potential for custom development

All info here is for educational purposes.

---

Enjoy hacking! 😄
