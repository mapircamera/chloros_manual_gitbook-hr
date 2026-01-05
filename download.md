---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Download

Download the latest version of Chloros to get started with multispectral image processing.

### System Requirements

| Requirement          | Minimum                         | Recommended                     |
| -------------------- | ------------------------------- | ------------------------------- |
| **Operating System** | Windows 10 (64-bit)             | Windows 11 (64-bit)             |
| **Processor**        | Intel Core i5 or equivalent     | Intel Core i7 or better         |
| **Memory (RAM)**     | 8GB                             | 16GB or more                    |
| **Graphics Card**    | DirectX 11 compatible           | NVIDIA GPU with 4GB+ VRAM       |
| **Storage**          | 6GB free space                  | SSD with 10GB+ free space       |
| **Display**          | 1920x1080                       | 2560x1440 or higher             |
| **Internet**         | Required for license activation | Required for license activation |

{% hint style="info" %}
**GPU Acceleration**: Chloros+ users with NVIDIA GPUs (4GB+ VRAM) can use CUDA acceleration for significantly faster processing. Chloros+ users also gain multi-threaded processing for maximum speed.
{% endhint %}

***

## Download Chloros

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">Download Chloros Here</a>

### Latest Stable Release

**Chloros Installer for Windows**

* **Version**: 1.0.4
* **Release Date**: January 5, 2026
* **File Size (Download)**: 1.8GB
* **File Size (Installed)**: 5.7GB
* **File Type**: .exe (Windows Installer)

#### **Installation Steps:**

1. Download the `CHLOROS INSTALLER - CURRENT VERSION.exe` file
2. Double-click the installer to begin installation
3. Follow the installation wizard prompts
4. Choose installation directory (default: `C:\Program Files\[USER]\Chloros\`)
5. Complete installation and launch Chloros, Chloros (Browser), or Chloros CLI
6. Sign in with your [MAPIR Cloud Chloros+ account](https://cloud.mapir.camera/pricing) (or continue with free version)

{% hint style="success" %}
The installer automatically adds `chloros-cli` to your system PATH for command-line access.
{% endhint %}

***

## Additional Resources

### Python SDK

For developers and automation workflows, install the Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Documentation**: [API: Python SDK](api-python-sdk.md)

**Requirements**: Chloros Desktop must be installed, Chloros+ license login required

***

## What's Included

The Chloros installation includes:

* ✅ **Chloros** - Full-featured graphical interface
* ✅ **Chloros (Browser)** - Web-based interface for lower-spec systems
* ✅ **Chloros CLI** - Command-line interface (requires Chloros+ license)
* ✅ **Chloros SDK** - Python API (requires Chloros+ license)
* ✅ **Camera Profiles** - Pre-configured MAPIR camera templates

***

## Upgrade to Chloros+

Unlock advanced features with a Chloros+ subscription:

* 🚀 **Multi-threaded Processing** - Process images in parallel
* ⚡ **GPU (CUDA) Acceleration** - Leverage NVIDIA GPU power
* 💻 **CLI Access** - Automate with command-line tools
* 🐍 **Python SDK** - Programmatic API access
* 📱 **Multiple Devices** - Use on 2-10+ devices (plan dependent)
* 🧮 **Custom Formulas** - Create custom multispectral indices

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">View Chloros+ Plans &#x26; Pricing</a></p>

***

## Installation Help

### Troubleshooting

**Installation fails with error message:**

* Ensure you have administrator rights
* Temporarily disable antivirus software
* Check that you meet minimum system requirements

**Application won't start:**

* Try Chloros (Browser) version
* Verify Windows 10/11 (64-bit) is installed
* Update graphics drivers
* Check Windows Event Viewer for error details
* Contact support with error logs

**License activation issues:**

* Ensure internet connection is active
* Verify credentials at [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Check firewall isn't blocking Chloros
* See [Chloros+ Login](chloros+-login.md) for detailed instructions

### Getting Support

Need help with installation or setup?

* 📧 **Email**: info@mapir.camera
* 🌐 **Website**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Documentation**: [Getting Started](./)
* ❓ **FAQ**: [Frequently Asked Questions](faq.md)

***

## Change Log

<details>

<summary>Version 1.0.4</summary>

#### **Release Date**: January 5, 2026

**New Features**

* **Image/Metadata Toggle**: Added toggle in File Browser to view selected image's metadata in a table instead of the image grid
* **Image Grid Zoom Slider**: New UI slider to adjust thumbnail size (also supports CTRL + mouse wheel)
* **Image Grid Export Buttons**: Buttons in the top row to switch thumbnails from JPG to processed exports (Targets, Reflectance, Index, LUT)
* **Map Tab**: New interactive 2D map showing image GPS location markers
  * Supports Google Maps and ESRI map tiles (auto-selects best tile service based on zoom level availability)
  * Mouse hover thumbnail preview on map markers

**Bug Fixes**

* Improved support for installing Chloros on non-English language computers

</details>

<details>

<summary>Version 1.0.3</summary>

#### **Release Date**: December 20, 2025

**New Features**

* Initial Launch

**Improvements**

* Initial Launch

**Bug Fixes**

* Initial Launch

**Known Issues**

* Initial Launch

</details>

***

## License Agreement

**Proprietary Software** - Copyright (c) 2025 MAPIR Inc.

Unauthorized use, distribution, or modification is prohibited.

**Free Version**: Available for personal and commercial use with feature limitations

**Chloros+**: Subscription-based license for advanced features and commercial deployments
