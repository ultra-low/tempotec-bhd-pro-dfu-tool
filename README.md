---

# TempoTec Sonata BHD Pro DFU Tool: Patched Versions

## Overview

The official TempoTec DFU Tool for the Sonata BHD Pro has stopped launching on recent Windows installations. The tool is used to configure DAC filter, volume table, NOS mode, phase compensation, harmonic and IEQ filter settings. It exits silently without showing any UI, which makes it difficult to diagnose without digging into Windows Event Viewer.

This repository has patched versions of the tool that resolve the issue.

---

## What Causes The Problem

After checking Windows Event Viewer and decompiling the tool with dnSpy, the cause turned out to be an expired SSL certificate on HiBy's OTA server (otaserver.hiby.com). The DFU tool is built on HiBy's control platform and tries to fetch device configuration from this server every time it launches. When the TLS handshake fails because of the bad certificate, it throws an unhandled exception and closes straight away with no error message. Nothing in the UI ever appears.

---

## The Two Patched Versions

### TempoTec_DFU_Tool_patched.7z
This version patches the HttpClient constructor to skip SSL certificate validation. It still connects to HiBy's server on launch, so it will work as long as the server stays online. Useful if you want the tool to pick up any future server side changes.

### TempoTec_DFU_Tool_offline.7z (Recommended)
This version replaces the server call entirely with a hardcoded local copy of the BHD Pro's feature configuration JSON, which was captured directly from HiBy's server before it became unreachable. It launches instantly with no network dependency and will keep working permanently regardless of what happens to HiBy's infrastructure. This is the version most people will want.

---

## Important: BHD Pro Only

The offline version has the Sonata BHD Pro's configuration hardcoded into it specifically. It will launch on any Windows machine but will only work correctly with a Sonata BHD Pro connected. It will not work with other TempoTec or HiBy platform devices since those use different device model identifiers and different configuration data stored separately on the server.

Both patched versions need the original TempoTec_DFU_Tool.exe.config file in the same folder to run correctly. This file is included in both archives along with the original unmodified executable for anyone who wants to verify the changes themselves.

---

## VirusTotal False Positives

Scanning the patched executables on VirusTotal shows 3 detections: Ikarus (PUA.MSIL.Agent), MaxSecure (Trojan.Malware.300983.susgen) and SentinelOne (Static AI - Suspicious Archive). All three are false positives. They are heuristic and machine learning based detections that trigger on any executable modified with dnSpy, since the bytecode no longer matches a known clean software signature. There is no actual malware.

If you want to verify this yourself, open either patched executable in dnSpy and check the two changes that were made:

1. The http() constructor in HIBY.Control.http now creates an HttpClient with a handler that skips SSL certificate validation
2. The _get_product_feature_info method in HIBY.Control.http now returns a hardcoded JSON string instead of making a server request

Nothing else was changed.

---

## Repository Contents

```
tempotec-bhd-pro-dfu-tool/
├── README.md
├── TempoTec_DFU_Tool_Patched.7z
└── TempoTec_DFU_Tool_Offline.7z
```

---
