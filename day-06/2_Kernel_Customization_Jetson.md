# Kernel customization on Jetson

## 🎯 Goal
- Build a custom Linux kernel for a Jetson device (e.g., enable WireGuard) from NVIDIA BSP sources and install the kernel + modules.

This document provides a concise, numbered workflow, corrected commands and troubleshooting steps for common permission and build issues.

---

## ⚙️ Prerequisites
- Jetson device with JetPack installed (or a Linux host with the BSP sources).  
- sudo access and basic build tools.  
- Enough disk space (it can take up to 10-20 GB) and optionally swap for building.

Install required build tools:

```bash
sudo apt update
sudo apt install -y build-essential bc libncurses5-dev flex bison libssl-dev libelf-dev
```

---

## 1) Download BSP / public sources
1. Download the NVIDIA BSP / public sources for your JetPack version from:
	https://developer.nvidia.com/embedded/jetson-linux-archive
2. Copy the downloaded `public_sources.tbz2` to a working directory on the Jetson or build host.

3. Extract public sources (use `-j` for bzip2-compressed `.tbz2`):

```bash
tar -xjf public_sources.tbz2
```

4. Change into the extracted source folder:

```bash
cd Linux_for_Tegra/source
```

5. Inside `source/public` extract the kernel sources (name may vary; e.g. `kernel_src.tbz2`):

```bash
cd public
tar -xjf kernel_src.tbz2
# This should create a kernel/ or kernel-*/ tree under this directory
ls -la
```

6. Locate the kernel tree (e.g., `kernel/kernel-5.10`) and note the path for the build steps.

---

## 2) Prepare an out-of-tree build directory
Build out-of-tree to keep sources clean:

```bash
# from Linux_for_Tegra/source (or where you placed the sources)
mkdir -p kernel_out
export TEGRA_KERNEL_OUT=$PWD/kernel_out
```

---

## 3) Generate default config and customize (menuconfig)
1. Change to the kernel source directory (adjust the path as found earlier):

```bash
cd public/kernel/kernel-5.10/
```

2. Optional: clean the output area before starting:

```bash
make ARCH=arm64 O=$TEGRA_KERNEL_OUT mrproper
```

3. Generate the default Tegra config and open `menuconfig`:

```bash
make ARCH=arm64 O=$TEGRA_KERNEL_OUT tegra_defconfig
make ARCH=arm64 O=$TEGRA_KERNEL_OUT menuconfig
```

In `menuconfig` to enable WireGuard as a module:

- Navigate: Device Drivers -> Network device support -> Network core driver support -> WireGuard secure network tunnel
- Press `M` to build as module (shows `<M>`), then `Esc` twice and save when prompted.

---

## 4) Build kernel, device trees (DTBs) and modules
Build the kernel image, DTBs and modules (may take a while):

```bash
make ARCH=arm64 O=$TEGRA_KERNEL_OUT -j$(nproc)
```

After a successful build, back up current kernel and modules:

```bash
sudo cp /boot/Image /boot/Image.backup
sudo cp -r /lib/modules/$(uname -r) /lib/modules/$(uname -r).backup
```

Install new modules and copy kernel + DTBs:

```bash
sudo make ARCH=arm64 O=$TEGRA_KERNEL_OUT modules_install
sudo cp $TEGRA_KERNEL_OUT/arch/arm64/boot/Image /boot/Image
sudo cp $TEGRA_KERNEL_OUT/arch/arm64/boot/dts/nvidia/* /boot/
# If your board requires special flash/bootloader steps, follow NVIDIA's instructions
```

Reboot to load the new kernel:

```bash
sudo reboot
```

After boot, verify module availability:

```bash
sudo modprobe wireguard
lsmod | grep wireguard
```

---

## Troubleshooting: permission errors during clean/config
If you encounter `Permission denied` errors when running `make mrproper` or `make tegra_defconfig` (e.g., on `net/wireguard/modules.order`), fix ownership and try again:

```bash
# go to extracted public sources
cd <path-to-extracted-public-sources>/Linux_for_Tegra/source/public

# set ownership to your user
sudo chown -R $USER:$USER .

# ensure the output directory is writable
sudo chown -R $USER:$USER $TEGRA_KERNEL_OUT

# then clean and re-run
cd kernel/kernel-5.10/
make ARCH=arm64 O=$TEGRA_KERNEL_OUT mrproper
make ARCH=arm64 O=$TEGRA_KERNEL_OUT tegra_defconfig
```

If the error persists, check for immutable attributes or files created by root earlier; `lsattr` can help.

---

## Verification
- Print kernel version after reboot:

```bash
uname -a
```

- Check module loaded or check dmesg for errors:

```bash
sudomodprobe wireguard || dmesg | tail -n 50
lsmod | grep wireguard
```

---

## Quick reference commands
```bash
# extract public sources
tar -xjf public_sources.tbz2
cd Linux_for_Tegra/source/public
tar -xjf kernel_src.tbz2

# prepare build
mkdir -p kernel_out
export TEGRA_KERNEL_OUT=$PWD/kernel_out
cd kernel/kernel-5.10/

# generate config & customize
make ARCH=arm64 O=$TEGRA_KERNEL_OUT tegra_defconfig
make ARCH=arm64 O=$TEGRA_KERNEL_OUT menuconfig

# build & install
make ARCH=arm64 O=$TEGRA_KERNEL_OUT -j$(nproc)
sudo make ARCH=arm64 O=$TEGRA_KERNEL_OUT modules_install
sudo cp $TEGRA_KERNEL_OUT/arch/arm64/boot/Image /boot/Image
sudo cp $TEGRA_KERNEL_OUT/arch/arm64/boot/dts/nvidia/* /boot/
sudo reboot
```

---

## Notes & references
- Builds are slow on-device; consider cross-compiling on x86 for faster iteration. 
- Kernel and DTB installation may be Jetson-model specific — consult the NVIDIA Jetson Linux Developer Guide for board-specific steps.  
- References: NVIDIA Jetson Linux Developer Guide, kernel build documentation.

