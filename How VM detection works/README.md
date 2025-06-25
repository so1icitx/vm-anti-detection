# How VM Detection Works

Virtual machine (VM) detection is used by software to figure out if it’s running on virtualized hardware instead of a physical machine. This is common in anti-cheat systems, malware, and licensed software to prevent cheating, analysis, or unauthorized use. Here’s a breakdown of how detection works, so you can understand what you’re up against when building an undetectable VM.

## Why Detect VMs?

Software has different reasons to check for VMs:
- **Anti-Cheat Systems**: Games use them to stop cheaters from running hacks in VMs, which can hide malicious processes.
- **Malware**: Avoids running in VMs to dodge security researchers analyzing it in sandboxes.
- **Licensing**: Some apps restrict usage in VMs to enforce license terms.
- **Reverse Engineering Protection**: Developers block VMs to make it harder to crack their software.

## Common Detection Techniques

Detection software looks for signs that scream “this is a VM!” Here’s how they do it:

### 1. Device Name and String Checks
VMs often use generic hardware names that give them away. Detection tools scan device descriptors via Windows Management Instrumentation (WMI), registry, or ACPI/PCI queries.
- **Examples**:
  - Keyboard: “QEMU Keyboard” vs. “ASUS Keyboard” on real hardware.
  - Storage: “QEMU HARDDISK” vs. “Samsung SSD”.
  - Mouse: “QEMU Mouse” vs. “Logitech Mouse”.
- **How It’s Done**: Tools query device strings and look for “QEMU,” “VMware,” or other VM-specific identifiers.

### 2. Serial Number Patterns
VM hardware often has predictable or default serial numbers:
- Sequential or zero-filled BIOS serials.
- Generic motherboard IDs.
- Serials containing “QEMU” or “VMware.”
- Detection software checks these via SMBIOS or WMI to spot VMs.

### 3. SMBIOS and DMI Data
System Management BIOS (SMBIOS) provides system info like manufacturer and product names. VMs often have generic entries like:
- Manufacturer: “QEMU” or “KVM.”
- Product: “Virtual Machine.”
Detection tools query SMBIOS to find these VM-specific values.

### 4. WMI Queries
Windows Management Instrumentation (WMI) lets software inspect system details. Detection tools look for:
- VM-specific hardware identifiers.
- Missing physical hardware classes, like:
  - `Win32_Fan`
  - `Win32_CacheMemory`
  - `Win32_VoltageProbe`
  - `CIM_TemperatureSensor`
These absences suggest a VM, as physical machines typically have sensors.

### 5. Hypervisor Detection
Modern CPUs and hypervisors leave clues:
- **CPUID Instruction**: Returns hypervisor vendor IDs (e.g., “KVMKVMKVM”) or feature bits indicating virtualization.
- **Model-Specific Registers (MSRs)**: Can reveal hypervisor presence.
- **Hypervisor Leaf**: CPUID leaf `0x40000000` exposes hypervisor info.
- **UEFI VM Bit**: Firmware flags in VMs indicate virtualization.

### 6. Memory and I/O Port Checks
Detection software probes:
- Hypervisor-specific memory regions.
- Virtualization communication ports.
- Privileged memory access patterns that behave differently in VMs.

### 7. Timing-Based Detection
VMs introduce subtle timing differences:
- **RDTSC (Read Time-Stamp Counter)**: This CPU instruction runs slower in VMs due to hypervisor overhead.
- **Other Timing Checks**:
  - CPU instruction execution times.
  - Memory access delays.
  - Interrupt handling inconsistencies.
  - Network timing anomalies.

## Why It’s Hard to Evade Detection

Detection methods are layered and sophisticated. A single slip—like a “QEMU” string or a hypervisor flag—can expose the VM. Advanced systems combine multiple checks, making it tough to hide every trace. Timing-based attacks, like RDTSC, are especially tricky because they exploit fundamental differences in how VMs handle CPU instructions.

## How to Counter Detection

To make a VM undetectable, you need to:
- Rename virtual hardware to mimic physical devices (e.g., “ASUS” instead of “QEMU”).
- Mask SMBIOS and ACPI data to show realistic hardware info.
- Hide hypervisor signatures by disabling CPU feature bits and modifying vendor IDs.
- Mitigate timing attacks (though this is harder and may require additional patches).

Check out the “Anti-Detection Techniques” README for details on how to tackle these challenges.
