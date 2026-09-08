# Pixel 5 (`redfin` / `redbull`) Custom Server & Networking Kernel

A specialized downstream Linux 4.19 kernel fork tailored for the **Google Pixel 5 (`redfin`, Snapdragon 765G / SM7250)**.

While standard Android kernels intentionally trim out networking, virtualization, and peripheral drivers for power efficiency and mobile isolation, this kernel reinstates server-grade features to transform the Pixel 5 into a **compact, battery-backed, high-performance headless Linux server, edge network appliance, and IoT router**.

---

## 🚀 Key Highlights & Features Beyond Stock Android

### 1. Advanced Networking & Multi-WAN Routing
Stock Android kernels strip out multi-routing and non-standard netfilter targets. This kernel restores complete Linux network capabilities:
- **Multipath Policy Routing (`CONFIG_IP_ROUTE_MULTIPATH=y`, `CONFIG_IP_MULTIPLE_TABLES=y`, `CONFIG_IP_ADVANCED_ROUTER=y`)**: Native ECMP (Equal-Cost Multi-Path) routing and multiple routing tables, enabling load balancing and failover across multiple cellular PDNs (`rmnet*`) or concurrent Wi-Fi / Ethernet connections.
- **Transparent Proxy Support (`CONFIG_NETFILTER_XT_TARGET_TPROXY=y`, `CONFIG_NETFILTER_XT_MATCH_SOCKET=y`, `CONFIG_NETFILTER_XT_MATCH_OWNER=y`)**: Enables high-efficiency TPROXY transparent proxying (sing-box, Clash, Xray) without the overhead and MTU penalty of userspace tun interfaces.
- **Google BBR Congestion Control (`CONFIG_TCP_CONG_BBR=y`, `CONFIG_DEFAULT_TCP_CONG="bbr"`, `CONFIG_NET_SCH_FQ=y`)**: Replaces default Cubic with BBR v1 and Fair Queuing (FQ) packet scheduler, optimizing throughput and lowering latency on lossy or congested mobile networks.
- **In-Tree WireGuard (`CONFIG_WIREGUARD=y`)**: Native kernel-level WireGuard driver v1.0.0 for wire-speed VPN performance with significantly lower CPU and battery drain compared to userspace implementations (`wireguard-go`).

### 2. Containerization & Linux Namespace Isolation
Provides the foundational kernel subsystems required to run Linux containers (Docker, Podman, LXC) and Linux rootfs environments (Debian, Ubuntu, Alpine chroots):
- **User Namespaces (`CONFIG_USER_NS=y`)**: Unlocks rootless container execution and unprivileged user mappings.
- **Full Namespaces (`CONFIG_NAMESPACES=y`, `CONFIG_PID_NS=y`, `CONFIG_IPC_NS=y`, `CONFIG_NET_NS=y`, `CONFIG_UTS_NS=y`)**: Complete process and network namespace isolation.
- **Full Control Groups (Cgroups)**: `CONFIG_MEMCG=y` (memory controller), `CONFIG_MEMCG_SWAP=y`, `CONFIG_CGROUP_DEVICE=y`, `CONFIG_CGROUP_FREEZER=y`, `CONFIG_CGROUP_BPF=y`, `CONFIG_BLK_CGROUP=y`, and `CONFIG_CPUSETS=y`.
- **Filesystem & Virtual Networking**:
  - `CONFIG_OVERLAY_FS=y`: Essential for Docker storage drivers and layered container images.
  - `CONFIG_VETH=y`: Virtual Ethernet pairs for container-to-host networking.
  - `CONFIG_BRIDGE=y` & `CONFIG_BRIDGE_NETFILTER=y`: Kernel bridge for container networks and NAT.
  - `CONFIG_TUN=y` & `CONFIG_FUSE_FS=y`: TUN/TAP devices and user-space filesystems (rclone, sshfs).

### 3. Peripheral Connectivity & USB Drivers
Turn your Pixel 5 into a hardware gateway with plug-and-play USB peripheral support:
- **USB-to-Serial Adapters (`CONFIG_USB_SERIAL=y`)**:
  - `CONFIG_USB_SERIAL_CP210X=y` (Silicon Labs CP2102/CP2104)
  - `CONFIG_USB_SERIAL_FTDI_SIO=y` (FTDI FT232R)
  - `CONFIG_USB_SERIAL_CH341=y` (WCH CH340 / CH341)
  - `CONFIG_USB_SERIAL_PL2303=y` (Prolific PL2303)
  - Direct connection to microcontrollers (ESP32, Arduino, STM32), serial consoles, and external USB LTE modems via USB-OTG.
- **USB Ethernet Adapters**:
  - `CONFIG_USB_RTL8152=y` (Realtek RTL8152 / RTL8153 Gigabit Ethernet)
  - `CONFIG_USB_USBNET=y` (Generic USB Networking)
  - `CONFIG_USB_NET_AX8817X=y` & `CONFIG_USB_NET_AX88179_178A=y` (ASIX Gigabit)
  - `CONFIG_USB_NET_CDCETHER=y` & `CONFIG_USB_NET_CDC_NCM=y` (CDC Ethernet & NCM)
- **USB Gadget Mode**: Built-in Android ConfigFS USB Ethernet gadgets (`CONFIG_USB_CONFIGFS_NCM`, `CONFIG_USB_CONFIGFS_ECM`, `CONFIG_USB_CONFIGFS_RNDIS`), allowing the Pixel 5 to serve as a high-speed network device when plugged into a PC or router.

### 4. Stealth & Security Layer
- **KernelSU Integrated**: Native kernel-level root solution.
- **SuSFS v2.2.0**: Integrated kernel filesystem hiding (`SUS_PATH`, `SUS_MAP`) for seamless root stealth and tamper resistance.

### 5. CPU Governors
- Supports `CONFIG_CPU_FREQ_GOV_PERFORMANCE=y`, `CONFIG_CPU_FREQ_GOV_ONDEMAND=y`, and `CONFIG_CPU_FREQ_GOV_SCHEDUTIL=y` (default EAS governor), enabling userspace scaling selection via sysfs depending on whether low latency or thermal longevity is desired.

---

## 🛠️ Automated CI/CD Build via GitHub Actions

This repository includes a completely automated GitHub Actions workflow (`.github/workflows/build-redbull-ksu.yml`). You do not need to set up a local Linux cross-compilation environment on your PC.

### Triggering a Build:
1. Push any commit to `feature/*` or the default branch, or trigger manually from the **Actions** tab on GitHub (**"Build redbull kernel + boot.img (KSU enabled)"** -> **Run workflow**).
2. The runner will compile the kernel with Clang/LLVM and package the build outputs into an artifact zip named `redbull-ksu-kernel`.

### Artifacts Produced:
- **`AnyKernel3-redbull-server-ksu.zip`**: **(Recommended)** Flashable zip package compatible with KernelSU App, Magisk, or custom recovery. It extracts the device's current boot partition, replaces the kernel binary (`Image.lz4`), and repacks it while preserving existing ramdisks, AVB signatures, and touchscreen/Wi-Fi calibrations.
- **`Image.lz4` / `Image.gz` / `Image`**: Raw compressed and uncompressed kernel binaries.
- **`modules-ksu.tar.gz`**: Compiled in-tree kernel modules.
- **`redbull_defconfig.ksu`**: The exact `.config` used for the build.

---

## 📲 Installation / Flashing Guide

### Method 1: Flashing via KernelSU / Magisk App (Easiest)
1. Download `AnyKernel3-redbull-server-ksu.zip` from your GitHub Actions run artifacts.
2. Transfer the zip file to your Pixel 5.
3. Open the **KernelSU App** (or Magisk App) on your phone.
4. Go to the **Modules** tab -> **Install from storage** -> Select `AnyKernel3-redbull-server-ksu.zip`.
5. Wait for the flashing process to complete, then tap **Reboot**.

### Method 2: Flashing via Recovery (TWRP / OrangeFox)
1. Boot your Pixel 5 into recovery mode.
2. Select **Install** -> Choose `AnyKernel3-redbull-server-ksu.zip`.
3. Swipe to confirm flash and reboot system.

---

## 🔍 Verification After Boot

Verify that the custom features are active via adb shell or terminal:

```bash
# Verify kernel version and build date
uname -r

# Verify TCP congestion control algorithm (should report bbr or cubic)
cat /proc/sys/net/ipv4/tcp_congestion_control
cat /proc/sys/net/ipv4/tcp_available_congestion_control

# Verify in-tree WireGuard
dmesg | grep -i wireguard

# Verify OverlayFS for Docker / Chroot
cat /proc/filesystems | grep overlay

# Verify USB serial driver availability
ls -l /sys/bus/usb-serial/drivers/
```

---

## 📜 Target Hardware Information
- **Device**: Google Pixel 5
- **Codename**: `redfin` (platform `redbull`, shared with Pixel 4a 5G `bramble` and Pixel 5a `barbet`)
- **Chipset**: Qualcomm Snapdragon 765G (`SM7250`)
- **Kernel Base**: Android Linux Kernel 4.19 LTS (`4.19.325`)
- **Defconfig**: `arch/arm64/configs/redbull_defconfig`
