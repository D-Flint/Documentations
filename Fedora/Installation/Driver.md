# This is a documentation for driver installation

## Nvidia GPU Drivers

Before performing any operations, ensure your system is prepared:

### Prerequisites

##### Check Secure Boot

- Secure Boot must be disabled in BIOS/UEFI. Nvidia proprietary drivers are not signed by your motheerboard manufacturer, and Secure Boot will block them from loading.

###### Run

> bash: 'mokutil --sb-state'

The output should return something like " SecureBoot disabled "

##### Repsitory

- Use RPM Fusion. This is the official Fedora-sanctioned method, ensuring drivers are packaged to survive kernal updates.

> Setup: RPM Fusion NVIDIA Howto 

### Driver Installation

The recommended way to install drivers is via the 'akmod' package. 'akmod' automatically recompiles the driver whenever your kernel updates.

> Install the necessary packages
> bash: 'sudo dnf install akmod-nvidia  xorg-x11-drv-nvidia-cuda'

**Crucial:** After installation, do not reboot immediately. Wait at least 5 Minutes. The 'akmod' service needs time to compile the driver for your current kernel.