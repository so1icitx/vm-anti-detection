# Hypervisor Concealment for VM Anti-Detection

Hiding the hypervisor is a critical step in making a virtual machine (VM) undetectable. Detection software—like anti-cheat or VM protection tools—often looks for hypervisor signatures in CPU features, vendor IDs, or system behavior. This guide details how the `qemu-anti-detection` project conceals the hypervisor (e.g., KVM in QEMU) to make your VM appear as a physical machine.

## Why Conceal the Hypervisor?

Hypervisors leave traces that detection software can spot:
- CPUID instructions reveal hypervisor vendor IDs (e.g., “KVMKVMKVM”).
- Hypervisor feature bits indicate virtualization.
- Model-specific registers (MSRs) or privileged instructions behave differently in VMs.
By masking these, you prevent the guest OS from detecting the virtual environment.

## How It Works

The `qemu-anti-detection` project uses QEMU patches and VM XML configurations to hide hypervisor traces, making the VM’s CPU and system behavior mimic a physical machine.

### Key Concealment Techniques
- **Hide KVM Signatures**:
  - Disables KVM-specific indicators visible to the guest OS.
  - Config: `<kvm><hidden state="on"/></kvm>`
- **Modify CPU Vendor ID**:
  - Changes hypervisor ID from “KVMKVMKVM” to “GenuineIntel” or “AuthenticAMD.”
  - Config: `<vendor_id state="on" value="GenuineIntel"/>`
- **Disable Hypervisor Feature Bit**:
  - Removes the CPUID hypervisor flag (ECX[31]).
  - Config: `<feature policy="disable" name="hypervisor"/>`
- **Host CPU Passthrough**:
  - Passes host CPU characteristics to the guest, avoiding VM-specific CPU models.
  - Config: `<cpu mode="host-passthrough" check="none" migratable="on">`
- **Custom CPU Model**:
  - Sets a realistic CPU name (e.g., “Intel i9-12900K”).
  - Config: `-cpu host,model_id=Intel(R) Core(TM) i9-12900K CPU @ 2.60GHz`

### Implementation
- **QEMU Patches**: Modify CPUID and MSR handling to remove hypervisor traces.
- **VM XML Configuration**: Specifies hypervisor concealment settings.
  ```xml
  <features>
    <hyperv mode="custom">
      <vendor_id state="on" value="GenuineIntel"/>
    </hyperv>
    <kvm>
      <hidden state="on"/>
    </kvm>
    <cpu mode="host-passthrough" check="none" migratable="on">
      <feature policy="disable" name="hypervisor"/>
    </cpu>
  </features>
  <qemu:commandline>
    <qemu:arg value="-cpu"/>
    <qemu:arg value="host,family=6,model=158,stepping=2,model_id=Intel(R) Core(TM) i9-12900K CPU @ 2.60GHz,vmware-cpuid-freq=false,enforce=false,host-phys-bits=true,hypervisor=off"/>
  </qemu:commandline>
  ```
- **Command-Line Args**: Fine-tune CPU parameters to ensure consistency.

When detection software runs CPUID or MSR checks, it sees a legitimate CPU without hypervisor clues.

## Effectiveness

- **Strengths**:
  - Bypasses CPUID, MSR, and hypervisor flag-based detection.
  - Essential for advanced anti-cheat (e.g., Easy Anti-Cheat, Mhyprot).
  - Makes CPU behavior nearly identical to physical hardware.
- **Weaknesses**:
  - Doesn’t counter RDTSC timing attacks, which detect VM-specific timing delays.
  - Requires careful configuration to avoid inconsistencies.

## Configuration Variations

- **Intel CPUs**: Use `GenuineIntel` vendor ID. Enable `vmx` if using nested Hyper-V.
- **AMD CPUs**: Use `AuthenticAMD` vendor ID. Enable `topoext` for topology extensions.
  ```xml
  <hyperv mode="passthrough">
    <vendor_id state="on" value="AuthenticAMD"/>
  </hyperv>
  <cpu mode="host-passthrough" check="none" migratable="on">
    <feature policy="require" name="topoext"/>
    <feature policy="disable" name="hypervisor"/>
  </cpu>
  ```

## Integration with Other Techniques

Combine hypervisor concealment with:
- **Device Name Modifications**: To rename virtual hardware.
- **System Information Masking**: To hide SMBIOS and ACPI VM traces.

This ensures a comprehensive anti-detection setup.

## How to Apply

1. Build a patched QEMU with the `qemu-anti-detection` patches (see “Building Patched QEMU”).
2. Configure your VM’s XML with hypervisor concealment settings (see “VM Configuration Setup”).
3. Verify in the guest OS using CPUID tools or anti-cheat software.

See the “Anti-Detection Techniques” README for an overview, and the “Installation Guide” for complete setup steps.