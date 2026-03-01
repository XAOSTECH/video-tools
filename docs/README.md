

<!-- Project Shields/Badges -->
<p align="center">
  <a href="https://github.com/XAOSTECH/video-tools/blob/main/LICENSE">
    <img alt="License" src="https://img.shields.io/github/license/XAOSTECH/video-tools?style=flat-square&colour=green">
  </a>
  <a href="https://github.com/XAOSTECH/video-tools/issues">
    <img alt="Issues" src="https://img.shields.io/github/issues/XAOSTECH/video-tools?style=flat-square&logo=github&colour=yellow">
  </a>
  <a href="https://github.com/XAOSTECH/video-tools/stargazers">
    <img alt="Stars" src="https://img.shields.io/github/stars/XAOSTECH/video-tools?style=flat-square&logo=github&colour=gold">
  </a>
  <!-- <img alt="Repo Size" src="https://img.shields.io/github/repo-size/XAOSTECH/video-tools?style=flat-square&logo=files&colour=teal">
  <img alt="Code Size" src="https://img.shields.io/github/languages/code-size/XAOSTECH/video-tools?style=flat-square&logo=files&colour=orange"> -->
  <img alt="Contributors" src="https://img.shields.io/github/contributors/XAOSTECH/video-tools?style=flat-square&logo=github&colour=green">
  <img alt="Maintenance" src="https://img.shields.io/maintenance/yes/2026?style=flat-square">
</p>

# XAOSTECH video-tools

Professional Linux video capture, streaming, and HDR encoding toolset. Integrate low-cost USB capture devices, build P010 10-bit kernel support, and stream 4K HDR to YouTube with GPU acceleration. Some of these produce shims that conflict with secure boot unless enrolled with a MOK.

## 🎯 Project Modules

This monorepo contains standalone, production-ready modules:

### **op-cap** — USB Capture Optimizationg
Linux capture device automation with v4l2loopback, auto-reconnect, and crash recovery.
- ✅ v4l2loopback isolation & feed relay
- ✅ Auto-reconnect on device disconnect
- ✅ OBS crash recovery (auto-restart + auto-resume stream)
- ✅ USB device reset & driver rebind
- ✅ Wayland graphics optimisation
- ✅ HDR passthrough (NV12)

**Quick Start**: `make install && make optimise-drivers`  
**Docs**: [op-cap/docs/README.md](../op-cap/docs/README.md)   
**Note**: v42cl-loopback installation may require MOK enrolment or disabling secure boot

---

### **copt** — Capture & Streaming Automation
FFmpeg-based capture relay with P010 HDR support, HLS streaming, and YouTube integration.
- ✅ NV12 capture (immediate) / P010 10-bit (with kernel patch)
- ✅ GPU encoding (NVIDIA NVENC HEVC Main10)
- ✅ Full HDR metadata (BT.2020, SMPTE2084, mastering display)
- ✅ HLS HTTP PUT relay to YouTube Live
- ✅ Auto-reconnect on source disconnect
- ✅ Bandwidth limiting & performance tuning

**Quick Start**: `export COPT_FFMPEG_BIN=~/.local/bin/ffmpeg && copt-worker stream`  
**Docs**: [copt/docs/README.md](../copt/docs/README.md) | [P010_README.md](P010_README.md)

---

### **P010_for_V4L2** — Kernel HDR Support (Submodule)
Linux kernel patches for native 10-bit P010 video format capture.
- ✅ 3 surgical kernel modifications (~20 lines)
- ✅ DKMS auto-installer
- ✅ Works with standard Linux USB UVC
- ✅ Tested: Ubuntu 24.04, Raspberry Pi

**Install**: `sudo copt/scripts/setup-p010-support.sh`  
**Credits**: [@awawa-dev](https://github.com/awawa-dev) — [upstream repo](https://github.com/awawa-dev/P010_for_V4L2)  [(licence)](https://github.com/awawa-dev/P010_for_V4L2?tab=MIT-1-ov-file)  
**Detailed Guide**: [P010 Kernel Support](../copt/docs/P010_KERNEL_SUPPORT.md)  
**Note**: Installation requires MOK enrolment or disabling secure boot

---

## 🚀 Quick Start

### Scenario 1: Stable OBS Capture (op-cap only)
```bash
cd op-cap
make install
make optimise-drivers
# Launch OBS, add /dev/video10 as source
# v42cl-loopback (video10) requires MOK enrolment or disabling secure boot
```

### Scenario 2: YouTube HDR Streaming (op-cap + copt)
```bash
# Install op-cap
cd op-cap && make install && make optimise-drivers

# Install copt & custom FFmpeg
cd ../copt
bash scripts/build-ffmpeg-hevc.sh
export COPT_FFMPEG_BIN=~/.local/bin/ffmpeg

# Start streaming
copt-worker stream
```

### Scenario 3: Full 10-bit Native P010 (all with kernel)
```bash
# Setup P010 kernel support (requires reboot and MOK enrolment or disabling secure boot)
sudo copt/scripts/setup-p010-support.sh

# After reboot, verify
v4l2-ctl -d /dev/video0 --list-formats-ext | grep -i p010

# Use P010 for capture
export COPT_USB_INPUT_FORMAT=p010
copt-worker stream
```

---

## 📚 Comprehensive Guides

### For OBS USB Capture Users
👉 **[op-cap README](../op-cap/docs/README.md)** — Device setup, crash recovery, troubleshooting

### For HDR Streaming to YouTube
👉 **[P010_README.md](P010_README.md)** — Complete HDR pipeline, NV12 vs P010, verification checklist

### For Advanced Configuration
- [P010 Kernel Support Deep Dive](../copt/docs/P010_KERNEL_SUPPORT.md)
- [Device Hardware Info & Accuracy](../op-cap/docs/DRIVER_INFO.md)
- [Buffer Corruption Diagnosis](../op-cap/docs/BUFFER_CORRUPTION.md)
- [USB Stability & Recovery](../op-cap/docs/USB_STABILITY.md)
- [Wayland Graphics Optimisation](../op-cap/docs/WAYLAND_OPTIMISATION.md)

---

## ✨ Key Features

### Crash Recovery (op-cap)
- Automatic OBS restart on USB device crash
- Stream resumption (if streaming before crash)
- Exponential backoff (3 retry limit)
- 3-second recovery timeout

### HDR Streaming (copt)
- **4K @ 30fps** native P010 capture
- **NVIDIA NVENC** GPU encoding (HEVC Main10)
- **Full HDR metadata** (colour space, tone curve, mastering display)
- **YouTube recognition** verified working

### USB Stability (op-cap)
- Auto-reconnect on device disconnect
- Device health monitoring
- USB reset & driver rebind
- Power optimisation (disable autosuspend)

---

## 🔧 Module Integration

```
HDMI Source
    ↓
op-cap (USB Capture + OBS)
    ↓ [/dev/video10 loopback]
copt (FFmpeg relay)
    ↓ [P010 10-bit HEVC Main10]
YouTube Live HLS
    ↓
YouTube recognises as HDR ✓
```

---

## 📋 System Requirements

### Minimum
- Linux kernel 5.10+
- NVIDIA GPU with NVENC (RTX 2060+) or compatible AMD/Intel
- USB 3.0+ capture device (UGREEN 25173, Elgato HD60 S+, etc.)
- 8GB RAM, 4 CPU cores

### Recommended
- Ubuntu 24.04 or later / Debian 12+
- NVIDIA RTX 2060 SUPER or newer
- Desktop environment: GNOME/KDE

### Optional
- P010_for_V4L2 kernel patches (for native 10-bit capture)
- Custom FFmpeg build with NVENC (included script)

---

## 🏗️ Architecture

```
video-tools/
├── op-cap/                          # USB capture & OBS stability
│   ├── scripts/
│   │   ├── obs-safe-launch.sh      # ← Crash recovery wrapper
│   │   ├── auto_reconnect.sh       # ← Device monitoring
│   │   ├── optimise_drivers.sh     # ← GPU driver fixes
│   │   └── maintenance.sh          # ← Interactive recovery tools
│   ├── lib/                        # Core libraries
│   └── docs/
│       ├── README.md              # Main op-cap guide
│       ├── DRIVER_INFO.md         # Hardware specs
│       └── USB_STABILITY.md       # Troubleshooting
│
├── copt/                            # Capture & streaming relay
│   ├── scripts/
│   │   ├── build-ffmpeg-hevc.sh   # Custom FFmpeg builder
│   │   ├── setup-p010-support.sh  # ← Kernel patch installer
│   │   └── hls-upload-relay.sh    # YouTube HLS uploader
│   ├── lib/
│   │   └── ffmpeg.sh              # HEVC Main10 encoder
│   └── docs/
│       ├── README.md
│       └── P010_KERNEL_SUPPORT.md  # Kernel patching guide
│
├── P010_for_V4L2/                   # Submodule: Kernel patches
│   ├── p010.patch                  # 3 kernel modifications
│   ├── dkms-installer.sh          # DKMS installer
│   └── README.md                   # Upstream docs
│
└── docs/
    ├── README.md                    # ← You are here
    ├── P010_README.md              # Complete HDR guide
    └── [standard files]
```

---

## 🔄 Latest Updates

### op-cap (OBS Safety & Stability)
- **Crash Recovery** — Auto-restart OBS + resume stream on disconnect
- **--no-device** flag — Launch OBS without device requirement
- **--no-loopback** option — Direct device mode (skip v4l2loopback)
- **Stream Detection** — Monitors logs to detect active streams before crash
- **3-second Recovery** — Fast restart after device failure

### copt (HDR Streaming)
- **Full HDR Metadata** — BT.2020 colorspace + SMPTE2084 tone curve
- **Mastering Display Info** — Content light level integration
- **HLS Playlist Upload** — Fixed m3u8 relay to YouTube every 3-5 seconds
- **Bandwidth Control** — Limit streaming bitrate dynamically

---

## 🙏 Credits & Attribution

### Original Authors
- **XAOSTECH** — Integration, testing, documentation
- **@awawa-dev** — [P010_for_V4L2](https://github.com/awawa-dev/P010_for_V4L2) kernel patches, HyperHDR research

### Standards & References
- USB UVC 1.0+ specification
- HEVC/H.265 Main10 profile (HDR10)
- ITU-R BT.2100 (HDR colour space)
- SMPTE ST 2084 (PQ tone curve)
- YouTube HLS Live streaming API

---

## 📄 License

All modules distributed under **GPL-3.0 License**. See individual `LICENSE` files in each module.

---

## 💬 Support & Contributing

- 📧 **Issues**: [GitHub Issues](https://github.com/XAOSTECH/video-tools/issues)
- 🤝 **Contributing**: See [CONTRIBUTING.md](CONTRIBUTING.md)
- 📖 **Code of Conduct**: [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)

---

## 🚀 Getting Help

**OBS keeps crashing?** → [op-cap README crash recovery section](../op-cap/docs/README.md#obs-setup)

**YouTube not recognising HDR?** → [P010_README.md verification checklist](P010_README.md#verify-youtube-hdr-recognition)

**Device not detected?** → [op-cap troubleshooting](../op-cap/docs/README.md#troubleshooting)

**Building FFmpeg fails?** → [copt FFmpeg builder](../copt/scripts/build-ffmpeg-hevc.sh) — check script comments for common issues

---

<p align="center">
  <b>Made with ❤️ by XAOSTECH</b><br>
  <a href="https://github.com/XAOSTECH/video-tools">GitHub</a> • 
  <a href="https://github.com/XAOSTECH/video-tools/releases">Releases</a> •
  <a href="https://github.com/XAOSTECH/video-tools/issues">Issues</a>
</p>

**Status**: Production Ready ✅  
**Last Updated**: Feb 2026  
**Tested**: Ubuntu 24.04, UGREEN 25173, NVIDIA RTX 2060 SUPER, YouTube Live
