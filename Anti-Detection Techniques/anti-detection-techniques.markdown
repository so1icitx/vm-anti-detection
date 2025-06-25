# Anti-Detection Techniques for Undetectable VMs

Building a virtual machine (VM) that passes as a physical machine requires countering the detection methods used by anti-cheat systems, malware, and other software. This guide outlines the core anti-detection techniques used in the `qemu-anti-detection` project to make your VM undetectable. These methods focus on masking virtualization traces to fool detection software.

## Overview of Anti-Detection Strategies

The `qemu-anti-detection` project uses three main approaches to hide a VM:
1. **Device Name Modifications**: Rename virtual hardware to mimic real devices.
2. **System Information Masking**: Alter system-level data to look like physical hardware.
3. **Hypervisor Concealment**: Hide the hypervisor’s presence from the guest OS.

Together, these techniques address most detection vectors, though some limitations (like timing attacks) remain. Let’s dive into each strategy.

## 1. Device Name Modifications

Many detection tools check hardware names for VM-specific strings like “QEMU” or “VMware.” This technique replaces those with realistic names to blend in.

- **What It Does**:
  - Changes device names (e.g., “QEMU HARDDISK” to “ASUS HARDDISK”).
  - Modifies serial numbers to avoid predictable VM patterns.
  - Covers input devices, storage, USB, audio, network, and display adapters.
- **Examples**:
  - Keyboard: “QEMU PS/2 Keyboard” → “ASUS PS/2 Keyboard”
  - Mouse: “QEMU USB Mouse” → “ASUS USB Mouse”
  - Storage: “QEMU DVD-ROM” → “ASUS DVD-ROM”
- **How It Works**:
  - Patches QEMU source code to replace VM-specific strings with brands like ASUS or Intel.
  - Ensures device descriptors match real hardware when queried via WMI or registry.
- **Effectiveness**:
  - Bypasses simple string-based checks used by most anti-cheat and VM protection software.
  - Weak against timing or advanced hardware probes.

See the “Device Name Modifications” README for details.

## 2. System Information Masking

Detection software often queries system-level data like SMBIOS or ACPI tables to spot VMs. This technique rewrites that data to mimic a physical machine.

- **What It Does**:
  - Modifies SMBIOS entries (e.g., manufacturer, product, BIOS version).
  - Alters ACPI tables to hide VM-specific OEM IDs.
  - Disables UEFI boot logos (BGRT) that reveal virtualization.
- **Examples**:
  - SMBIOS Manufacturer: “QEMU” → “ASUS”
  - ACPI OEM ID: “BOCHS” → “INTEL”
  - SMBIOS Product: “QEMU Virtual Machine” → “ASUS UX305UA”
- **How It Works**:
  - Uses QEMU patches to rewrite system tables.
  - Configures VM XML to pass custom SMBIOS values (e.g., Intel i9-12900K).
  - Disables VM-specific flags in BIOS and UEFI.
- **Effectiveness**:
  - Counters WMI and SMBIOS-based detection.
  - Essential for passing as a real machine in system info checks.

See the “System Information Masking” README for more.

## 3. Hypervisor Concealment

The hypervisor (e.g., KVM in QEMU) leaves CPU and system traces that detection software can spot. This technique hides those traces.

- **What It Does**:
  - Disables hypervisor CPU feature bits.
  - Replaces hypervisor vendor IDs (e.g., “KVMKVMKVM” → “GenuineIntel”).
  - Uses host CPU passthrough to mimic physical CPU behavior.
  - Hides KVM-specific indicators in the guest OS.
- **Examples**:
  - CPUID Vendor: “KVMKVMKVM” → “GenuineIntel”
  - Hypervisor Feature Bit: Enabled → Disabled
  - CPU Model: “QEMU Virtual CPU” → “Intel i9-12900K”
- **How It Works**:
  - Configures VM XML to disable hypervisor flags and set realistic CPU info.
  - Patches QEMU to hide KVM signatures.
  - Passes host CPU characteristics to the guest.
- **Effectiveness**:
  - Bypasses CPUID and MSR-based detection.
  - Critical for advanced anti-cheat systems.

See the “Hypervisor Concealment” README for specifics.

## Compatibility with Detection Systems

These techniques have been tested against various anti-cheat and VM protection systems:
- **Bypassed**:
  - Anti-Cheat: Easy Anti-Cheat, Anti-Cheat Expert, GameGuard, Mhyprot.
  - Protection: Themida, VMProtect, Enigma Protector, Safegine Shielden.
- **Not Bypassed**:
  - Vanguard (requires extra configuration or may be unfeasible).
- **Partially Bypassed**:
  - Gepard Shield, Roblox (need additional tweaks).

## Limitations

While powerful, these techniques don’t cover everything:
- **RDTSC Timing Attacks**: Timing differences in CPU instructions like RDTSC can still expose the VM. A separate patch (e.g., `RDTSC-KVM-Handler`) may help.
- **WMI Sensor Queries**: VMs lack physical sensor data (e.g., fan, temperature), which WMI queries can detect. Future patches may emulate this.
- **Advanced Probes**: Sophisticated software might use undocumented or niche detection methods.

## How to Implement

To use these techniques:
1. Build a patched QEMU with the `qemu-anti-detection` patches (see “Building Patched QEMU”).
2. Configure your VM with the provided XML settings (see “VM Configuration Setup”).
3. Test the VM against your target software to ensure it passes detection.

For detailed steps on each technique, check the respective READMEs:
- Device Name Modifications
- System Information Masking
- Hypervisor Concealment

By combining these strategies, you can create a VM that’s nearly indistinguishable from physical hardware, but always test thoroughly for your specific use case.