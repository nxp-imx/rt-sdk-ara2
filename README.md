<div align="center">

# rt-sdk-ara2 ⚡

[![License badge](https://img.shields.io/badge/License-Proprietary-red)](./LICENSE.txt)
[![Platforms](https://img.shields.io/badge/Platforms-FRDM_i.MX_8M_Plus_|_FRDM_i.MX_95-blue)](https://www.nxp.com/products/processors-and-microcontrollers/arm-processors/i-mx-applications-processors:IMX_HOME)
[![Category badge](https://img.shields.io/badge/Category-AI/GenAI-orange)](https://www.nxp.com/docs/en/user-guide/UG10166.pdf)
[![BSP](https://img.shields.io/badge/BSP-LF6.18.37--2.1.0-purple)](https://www.nxp.com/design/design-center/software/embedded-software/i-mx-software/embedded-linux-for-i-mx-applications-processors:IMXLINUX)

</div>

Runtime SDK for AI/ML acceleration with Ara240 NPU on i.MX SoCs 🚀🧠💻

<div align="left">

## 📋 Table of Contents

- [Overview](#-overview)
- [Required Hardware](#-required-hardware)
- [Required Software](#-required-software)
- [Helper Scripts](#-helper-scripts)
- [Automatic Device Management](#-automatic-device-management)
- [Service Management](#-service-management)
- [Python API and Optimum-Ara](#-python-api-and-optimum-ara)
- [GStreamer Plugins](#-gstreamer-plugins)
- [UIO DMA Driver](#-uio-dma-driver)
- [Package Components](#-package-components)
- [Release Notes](#-release-notes)
- [Licensing](#%EF%B8%8F-licensing)
</div>

---

## 🎯 Overview

The **rt-sdk-ara2** provides a complete runtime environment for AI/ML acceleration using the Ara240 NPU on NXP i.MX SoCs. It includes:

- 🔧 **Runtime libraries** for Ara240 NPU integration
- 🐍 **Python bindings** (DVAPI) for custom inference applications
- 🤖 **Optimum-Ara framework** for LLMs and VLMs
- 🎥 **GStreamer plugins** for Real-Time Detection Object Applications
- 🛠️ **Helper scripts** for monitoring, benchmarking, and model management
- ⚙️ **Systemd service** for automatic hardware initialization
- 🔌 **Automatic device detection** via udev rules for hot-plug support
- 🚗 **UIO DMA driver** for PCIe communication

### Software Architecture

<div align="center">

```
┌─────────────────────────────────────────────────────────────────────────┐
│                             Application Layer                           │
├─────────────────────────────────────┬───────────────────────────────────┤
│               Python                │           Gstreamer               │
│  ┌───────────────────────────────┐  │  ┌─────────────────────────────┐  │
│  │   Optimum-Ara Framework       │  │  │      GstdvPlugins           │  │
│  ├───────────────────────────────┤  │  ├─────────────────────────────┤  │
│  │  dvapi.py - Python Bindings   │  │  │    ara_vision_infer         │  │
│  └───────────────────────────────┘  │  └─────────────────────────────┘  │
├─────────────────────────────────────┴───────────────────────────────────┤
│              libaraclient.so - Client Library                           │
├─────────────────────────────────────────────────────────────────────────┤
│                          Proxy Daemon                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                   uiodma.ko - UIO DMA Driver                            │
├─────────────────────────────────────────────────────────────────────────┤
│                      ARA240 NPU Hardware                                │
└─────────────────────────────────────────────────────────────────────────┘
```
</div>

<div align="left">

---

### 🔑 Key Components

This SDK integrates the following external components:

| **Component**       | **Repository**                                              | **License**       |
| ------------------- | ----------------------------------------------------------- | ----------------- |
| UIO DMA Driver      | https://github.com/nxp-imx-support/uiodma-driver            | GPL-2.0           |
| Optimum-Ara         | https://github.com/nxp/optimum-ara                          | Apache-2.0        |
| GStreamer Plugins   | https://github.com/nxp-imx-support/gstreamer-plugins-ara240 | LGPL-2.1-or-later |
</div>

---

## 🧰 Required Hardware

- [i.MX 8M Plus FRDM](https://www.nxp.com/design/design-center/development-boards-and-designs/FRDM-IMX8MPLUS) / [i.MX 95 FRDM](https://www.nxp.com/design/design-center/development-boards-and-designs/FRDM-IMX95)
- [Ara240 NPU module](https://www.nxp.com/design/design-center/development-boards-and-designs/ARA240-16GB-M2-MODULE?_gl=1*1at1n92*_ga*MTc3OTgzNTQwMS4xNzcyNzM0MzY3*_ga_WM5LE0KMSH*czE3NzM0MzUxMTMkbzI3JGcwJHQxNzczNDM1MTEzJGo2MCRsMCRoMTE3MzM5MTI2MQ..)
- microSD card (≥ 64GB recommended for LLM/VLM support)
- USB-C debug cable
- Ethernet connection
- Power supply

---

## 💻 Required Software

- [Embedded Linux for i.MX](https://www.nxp.com/design/design-center/software/embedded-software/i-mx-software/embedded-linux-for-i-mx-applications-processors:IMXLINUX) (= LF6.18.37_2.1.0)

---

## 🔧 Helper Scripts

The SDK includes several helper scripts to monitor, manage, and benchmark the Ara240 NPU. These scripts make it easy to check device status, update firmware, download models, and run performance tests.

### ara2-metrics - Monitor NPU Performance

Monitor real-time NPU metrics including utilization, temperature, DRAM usage, and device state:

```bash
ara2-metrics
```

**Interactive menu options:**

```plain
Found endpoints: count = 1

Press 1  to print Device info
Press 2  to print % NPU Utilization continuously
Press 3  to print Dram Info
Press 4  to print NPU Temperature
Press 5  to print NPU Temperature continuously
Press 6  to print % NPU Utilization, Temperature continuously
Press 7  to print NPU state
Press 0  to exit
```

> **Use case:** This tool is invaluable during benchmarking or model execution. You can correlate model performance with thermal behavior and utilization to identify bottlenecks. For example, if performance drops, you can verify whether the NPU is throttling due to temperature or experiencing memory pressure.

### ara2-chip-info - Device Summary

Get a comprehensive overview of the Ara240 NPU configuration and status:

```bash
ara2-chip-info
```

**Displays:**
- Chip ID and revision
- Bus ID and interface type (PCIe)
- System, NPU, and DDR frequencies
- Device temperature and voltage
- Power state
- Firmware version (raw and display format)
- DDR and flash information
- Life cycle and chip part type

> **Note:** This script runs automatically twice during Ara240 boot-up and provides a quick health check of the device.

### ara2-program-flash - Firmware Update

Update the Ara240 NPU firmware to the required version:

```bash
ara2-program-flash
```

> **Important:** Always reboot the board after running this script to ensure the new firmware is fully applied and the device state is clean.

### ara2-fetch-models - Download Pre-compiled Models

Download pre-trained (CNN/LLM) models from Hugging Face Hub for use with the [ARA240 platform](https://huggingface.co/collections/nxp/ara240-models):

```bash
ara2-fetch-models --repo-id  nxp/Ultralytics-YOLOv8-detection-Ara240
```

### ara2-model-perf - Performance Benchmarking

Run performance benchmarks on downloaded models:

```bash
ara2-model-perf
```

**Example output:**

```
==========================
Available Model Categories
==========================
  1) detection
  q) Quit
Enter the number corresponding to the category you want to explore: 1
==========================
Available Models in detection
==========================
1. yolov8n
```

**Key metric:** Look for **HW IPS** (Hardware Inferences Per Second) in the results. This represents the maximum throughput the Ara240 NPU can achieve for the selected model under current conditions.

> **Note:** IPS values may vary depending on thermal conditions, system load, interface type (PCIe), and whether the benchmark runs continuously or in bursts.

---

## 🔌 Automatic Device Management
When an Ara-2 device is present (including at boot), `ara2-device.target` is started automatically by default and handles:
- Loading the `uiodma` kernel driver
- Performing hardware bringup
- Launching the proxy daemon

### How It Works

When an Ara240 device is connected (PCIe or USB):
1. **udev** detects the device and triggers the handler script
2. The handler automatically starts the `ara2-device.target`
3. The target brings up the runtime services:
   `ara2-uiodma-bind.service` (binds the uiodma driver when PCIe devices are present),
   `ara2-hw-init.service`     (hardware bringup), and
   `ara2-proxy.service`       (proxy daemon)

When an Ara240 device is disconnected:
1. **udev** detects the removal and triggers the handler
2. If no other Ara240 devices remain, the service and target are stopped automatically
3. If other devices remain, the service is restarted to reconfigure

### Checking the Stack

After a device is detected, verify the stack came up:
```bash
systemctl is-active ara2-device.target ara2-uiodma-bind.service ara2-hw-init.service ara2-proxy.service
```
Expected output: `active` for the target, `ara2-hw-init.service` and
`ara2-proxy.service`. `inactive` is normal for `ara2-uiodma-bind.service`
(one-shot bind step, exits after binding the driver).

For the full picture, including recent log lines:
```bash
systemctl status ara2-device.target --no-pager -l
```
All udev handler activity is logged to `/var/log/ara2-udev-events.log`.

### Manual Control

Start or stop the whole stack by hand:
```bash
sudo systemctl stop ara2-device.target
sudo systemctl start ara2-device.target
```

---

## ⚙️ Service Management

The SDK uses systemd for service lifecycle management with a target-based architecture for clean device handling.

### Architecture Overview

```
ara2-device.target (device aggregator)
      ├── Requires: ara2-uiodma-bind.service (binds the uiodma driver when PCIe devices are present)
      ├── Requires: ara2-hw-init.service     (hardware bringup)
      └── Requires: ara2-proxy.service       (proxy daemon, ordered after hw-init)

```
Stopping the target stops all member services; restarting it re-runs the whole bring-up sequence (used by the udev handler on device changes).

### Service Operations

**Check service/target status:**
```bash
systemctl status ara2-device.target --no-pager -l
```

**Start the Ara2 stack:**
```bash
systemctl start ara2-device.target
```

**Stop the Ara2 stack:**
```bash
systemctl stop ara2-device.target
```

**Restart the service:**
```bash
systemctl restart ara2-device.target
```

**View detailed logs:**
```bash
journalctl -u ara2-hw-init.service -f
journalctl -u ara2-proxy.service -f
journalctl -u ara2-uiodma-bind.service -f (PCIE only)
```

**View udev event logs:**
```bash
tail -f /var/log/ara2-udev-events.log
```

---

## 📚 Python API and Optimum-Ara

### DVAPI Python Bindings

The SDK includes Python bindings for the DVAPI (DeepVision API):

**Location:** `/usr/share/ara2/dvapi.py`

**Key classes and methods:**
- `DVSession` - Session management for Ara240 NPU
- `DVModel` - Model loading and management
- `DVEndpoint` - Endpoint (device) management
- `dv_infer_wait_for_completion()` - Blocking inference execution

### Optimum-Ara Framework

Optimum-Ara is a framework for running Large Language Models (LLMs) and Vision-Language Models (VLMs) on Ara240.

**Location:** `/usr/share/ara2/optimum-ara/`

**Repository:** https://github.com/nxp/optimum-ara 

**License:** Apache-2.0

**Supported models:**
- **LLMs:** Qwen 2.5 Instruct, Qwen 2.5 Coder
- **VLMs:** Qwen 2.5-VL 

**Example scripts:** `/usr/share/ara/optimum-ara/examples/`

**API documentation:** `/usr/share/ara2/optimum-ara/docs/`

**Example usage:**

```bash
cd /usr/share/ara2/optimum-ara/examples/
python3 -m venv .venv
. .venv/bin/activate
pip install /usr/share/python-wheels/optimum_ara-1.1.0-py3-none-any.whl
```

```bash
python3 causalLM_auto.py
```

---

## 🎥 GStreamer Plugins

The SDK includes a custom GStreamer plugin optimized for zero-copy video inference pipelines with the Ara240 NPU.

**Location:** `/usr/lib/gstreamer-1.0/`

**Repository:** https://github.com/nxp-imx-support/gstreamer-plugins-ara240

**License:** Proprietary

**Name:** libgstdvInf

**Properties:**
   - Unified inference plugin
   - Handles image preprocessing (resizing, normalization, color conversion)
   - Executes inference on Ara240 NPU
   - Manages model loading and execution
   - Processes inference results
   - Zero-copy buffer operations for optimal performance

---

## 🚗 UIO DMA Driver

The **uiodma.ko** kernel driver enables high-speed PCIe communication between the i.MX host processor and the Ara240 NPU module.

**Location:** `/lib/modules/$(uname -r)/extra/uiodma.ko`

**Repository:** https://github.com/nxp-imx-support/uiodma-driver

**License:** GPL-2.0

### Driver Information

The driver is automatically loaded during system boot by the `ara2-uiodma-bind.service`. You can verify it's loaded with:

```bash
lsmod | grep uiodma
```

To manually load the driver:

```bash
modprobe uiodma
```

To manually unload the driver:

```bash
rmmod uiodma
```

> **Note:** The driver is essential for Ara240 NPU operation. Do not unload it while running inference workloads.

---

## 📦 Package Components

The SDK installs components to the following locations:

### Binaries and Scripts
- `/usr/bin/nnapp` - Inference application
- `/usr/bin/ara2-chip-info` - Chip information tool
- `/usr/bin/ara2-model-perf` - Model performance benchmark
- `/usr/bin/ara2-metrics` - Hardware metrics
- `/usr/bin/ara2-fetch-models` - Model download helper
- `/usr/bin/ara2-program-flash` - Flash programming tool
- `/usr/bin/enable-swap`, `/usr/bin/resize-disk-max` - Storage helpers for LLM use

### Runtime Scripts and Hardware Utilities
- `/usr/libexec/ara2/scripts/` -  All utility scripts
- `/usr/libexec/ara2/scripts/ara2-udev-handler` - udev event handler 
- `/usr/libexec/ara2/hw_utils/` - Hardware bringup utilities and firmware
- `/usr/libexec/ara2/proxy/` - Proxy daemon

### Libraries
- `/usr/lib/libaraclient_aarch64.so`/`.a` - Ara client library
- `/usr/lib/libara_vision_inference.so*` - Vision inference library
- `/usr/lib/gstreamer-1.0/` - GStreamer plugins for video inference pipelines

### Python Modules
- `/usr/share/ara2/dvapi.py` - Python bindings for the DVAPI
- `/usr/share/python-wheels/optimum_ara-*.whl` - Optimum-Ara wheel

### System Configuration
- `/usr/lib/systemd/system/ara2-device.target` - Device aggregator target
- `/usr/lib/systemd/system/ara2-uiodma-bind.service` - uiodma driver bind (one-shot, PCIe only)
- `/usr/lib/systemd/system/ara2-hw-init.service` - Hardware bringup service
- `/usr/lib/systemd/system/ara2-proxy.service` - Proxy daemon service
- `/etc/udev/rules.d/99-ara2.rules` - Udev rules for NPU detection
- `/etc/default/ara2` - Board-specific settings (`ARA2_BOARD`, `ARA2_DDR_PLL`, `ARA2_DDR_CFG`, `ARA2_PROXY_CONFIG`)
- `/etc/ara2/proxy_config.yaml` - Proxy configuration
- `/etc/ara2/cnn_config.yaml` - CNN inference configuration

### Drivers
- ` /lib/modules/$(uname -r)/extra/uiodma.ko` - UIO DMA driver location
   - `uiodma.ko` - Kernel module for PCIe communication

### Logs
- `/var/log/ara2-udev-events.log` - Udev event handler logs
---

## 📝 Release Notes

### Version Information

| **Component**      | **Version** |
| ------------------ | ----------- |
| Firmware (raw)     | 131072      |
| Firmware (display) | 2.0.0       |
| Optimum-Ara        | 1.1.0       |

---

## ⚖️ Licensing

This repository is licensed under the [LA_OPT_NXP_Software_License](./LICENSE.txt) license.

---

<div align="center">

**For support and additional information, please refer to the official NXP documentation.**

</div>
