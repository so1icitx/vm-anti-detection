# System Information Masking for VM Anti-Detection

System information masking is crucial for making a virtual machine (VM) undetectable. Detection software often checks system-level data, like SMBIOS or ACPI tables, for signs of virtualization. This guide explains how the `qemu-anti-detection` project rewrites these data structures to mimic a physical machine, helping your VM evade detection.

## Why Mask System Information?

Many detection tools query system info to spot VMs. For example:
- SMBIOS might show “QEMU” as the manufacturer or “Virtual Machine” as the product.
- ACPI tables could have VM-specific OEM IDs like “BOCHS.”
- UEFI boot logos (BGRT) can reveal virtualization.
By altering these to realistic values, you make the VM look like a real computer.

## How It Works

The `qemu-anti-detection` project uses QEMU patches and VM XML configurations to modify system information, ensuring it matches physical hardware when queried by the guest OS.

### Modified System Data
Here’s what gets changed:
- **SMBIOS Entries**:
  - Type 0 (BIOS): Disables VM bit, sets version to “UX305UA.201.”
  - Type 1 (System): Manufacturer: “QEMU” → “ASUS”; Product: “QEMU Virtual Machine” → “UX305UA.”
  - Type 2 (Motherboard): Manufacturer: “QEMU” → “Intel”; Product: “Intel i9-12900K.”
  - Type 17 (Memory): Manufacturer: “KINGSTON”; Speed: “4800.”
- **ACPI Tables**:
  - OEM ID: “BOCHS” → “INTEL”
  - Table ID: “BXPC” → “PC8086”
  - Creator ID: “QEMU” → “PTL”
- **UEFI Boot Graphics (BGRT)**:
  - Disabled to hide VM-specific boot logos.
- **CPU Identity**:
  - Vendor ID: “KVMKVMKVM” → “GenuineIntel”
  - Model: “QEMU Virtual CPU” → “Intel i9-12900K”

### Implementation
- **QEMU Patches**: Modify how SMBIOS and ACPI tables are generated to remove VM-specific identifiers.
- **VM XML Configuration**: Specifies custom SMBIOS values and disables VM flags.
  ```xml
  <sysinfo type="smbios">
    <system>
      <entry name="manufacturer">ASUS</entry>
      <entry name="product">UX305UA</entry>
    </system>
    <baseBoard>
      <entry name="manufacturer">Intel</entry>
      <entry name="product">Intel i9-12900K</entry>
    </baseBoard>
  </sysinfo>
  ```
- **UEFI Tweaks**: Disables BGRT and VM bits in firmware.

When detection software queries via WMI or PowerShell, it sees these realistic values instead of VM giveaways.

## Effectiveness

- **Strengths**:
  - Counters SMBIOS, ACPI, and WMI-based detection.
  - Makes system info indistinguishable from a physical machine.
  - Bypasses checks in anti-cheat (e.g., GameGuard) and protection software (e.g., VMProtect).
- **Weaknesses**:
  - Doesn’t address timing attacks (e.g., RDTSC).
  - Missing sensor data (e.g., temperature, fan) can still expose the VM via WMI.

## Integration with Other Techniques

Combine system information masking with:
- **Device Name Modifications**: To rename virtual hardware.
- **Hypervisor Concealment**: To hide CPU and hypervisor signatures.

This trio covers most detection vectors for a robust anti-detection setup.

## How to Apply

1. Build a patched QEMU with the `qemu-anti-detection` patches (see “Building Patched QEMU”).
2. Configure your VM’s XML with custom SMBIOS and ACPI settings (see “VM Configuration Setup”).
3. Use `dmidecode` on a physical machine to get authentic SMBIOS values for your config.
4. Verify in the guest OS with `Get-WmiObject -Class Win32_BIOS` or similar.

See the “Anti-Detection Techniques” README for an overview, and the “Installation Guide” for full setup steps.
