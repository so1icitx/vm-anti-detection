# Device Name Modifications for VM Anti-Detection

Device name modifications are a key part of making a virtual machine (VM) undetectable. Many detection tools—like anti-cheat or VM protection software—check hardware names for signs of virtualization, such as “QEMU” or “VMware.” This guide explains how the `qemu-anti-detection` project renames virtual hardware to mimic real devices, helping your VM pass as a physical machine.

## Why Modify Device Names?

Detection software scans device descriptors for VM-specific strings. For example:
- A VM keyboard might show as “QEMU PS/2 Keyboard,” while a real one says “ASUS PS/2 Keyboard.”
- Storage devices in VMs often appear as “QEMU HARDDISK” instead of “Samsung SSD.”
By renaming these to realistic brands like ASUS or Intel, you can fool most string-based checks.

## How It Works

The `qemu-anti-detection` patches replace VM-specific identifiers in QEMU’s source code with names that mimic physical hardware. This affects device names, serial numbers, and vendor IDs across multiple hardware categories.

### Modified Device Categories
Here’s what gets renamed:
- **Input Devices**:
  - QEMU PS/2 Keyboard → ASUS PS/2 Keyboard
  - QEMU USB Mouse → ASUS USB Mouse
  - QEMU HID Tablet → ASUS HID Tablet
- **Storage Devices**:
  - QEMU HARDDISK → ASUS HARDDISK
  - QEMU DVD-ROM → ASUS DVD-ROM
  - Serial: QM00001 → ASUS00001
- **USB Controllers**:
  - QEMU USB Hub → ASUS USB Hub
- **Audio Devices**:
  - QEMU USB Audio → ASUS USB Audio
- **Network Adapters**:
  - QEMU USB Network → ASUS USB Network
- **Display Adapters**:
  - QEMU Monitor → DEL Monitor
- **ACPI IDs**:
  - QEMU0002 → ASUS0002

### Implementation
- **String Replacement**: Patches swap “QEMU” for “ASUS” (or other brands) in device descriptors.
- **Serial Numbers**: VM-specific serials (e.g., “QM12345”) are changed to realistic patterns (e.g., “ASUS12345”).
- **Vendor IDs**: PCI and USB vendor IDs are set to match real manufacturers like Intel (0x8086).

When the guest OS queries devices via WMI, registry, or ACPI, it sees these modified names, making the VM look like a physical machine.

## Effectiveness

- **Strengths**:
  - Bypasses the most common detection method: string-based device checks.
  - Works against most anti-cheat (e.g., Easy Anti-Cheat) and protection software (e.g., Themida).
  - Simple to implement via QEMU patches.
- **Weaknesses**:
  - Doesn’t counter timing-based detection (e.g., RDTSC attacks).
  - Advanced tools might detect other VM signatures beyond device names.

## Integration with Other Techniques

Device name modifications alone aren’t enough. Combine them with:
- **System Information Masking**: To hide VM traces in SMBIOS and ACPI tables.
- **Hypervisor Concealment**: To remove hypervisor flags and CPU signatures.

Together, these make your VM nearly indistinguishable from real hardware.

## How to Apply

1. Build a patched QEMU with the `qemu-anti-detection` patches (see “Building Patched QEMU”).
2. Configure your VM’s XML to use realistic device settings, like SATA disks and e1000e network interfaces (see “VM Configuration Setup”).
3. Verify device names in the guest OS using `systeminfo` or WMI queries.

Check the main “Anti-Detection Techniques” README for an overview, and the “Installation Guide” for step-by-step setup instructions.
