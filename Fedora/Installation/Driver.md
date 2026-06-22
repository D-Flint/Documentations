# This is a documentation for driver installation

## Nvidia GPU Drivers

Before performing any operations, ensure your system is prepared:

##### Check Secure Boot

- Secure Boot must be disabled in BIOS/UEFI. Nvidia proprietary drivers are not signed by your motheerboard manufacturer, and Secure Boot will block them from loading.

###### Run

> bash: mokutil --sb-state

The output should return something like " SecureBoot disabled "

##### Repsitory

- Use RPM Fusion. This is the official Fedora-sanctioned method, ensuring drivers are packaged to survive kernal updates.

> Setup: RPM Fusion NVIDIA Howto