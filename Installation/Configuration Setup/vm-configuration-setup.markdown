# VM Configuration Setup for Anti-Detection

Configuring your virtual machine (VM) with anti-detection settings is essential to make it appear as a physical machine. This guide details how to set up a VM using the `qemu-anti-detection` project’s XML configuration for `libvirt`, ensuring it hides virtualization traces from detection software like anti-cheat or VM protection.

## Why Configure the VM?

The patched QEMU (from “Building Patched QEMU”) masks many VM indicators, but the VM’s configuration fine-tunes system settings are needed to:
- Set realistic hardware IDs via SMBIOS.
- Hide the hypervisor with CPU and firmware tweaks.
- Use non-virtualized device interfaces to blend in real hardware.

## Prerequisites

- **Patched QEMU**: Built with `qemu-anti-detection` patches.
- **Libvirt**: Installed and running (`sudo systemctl enable --now libvirtd.service`).
- **Packages**: `virt-manager`, `edk2-ovmf`, `swtpm`, `dmidecode` (see “Installation Guide”).
- **Disk Image**: Create one with `qemu-img create -f raw disk.img 50G`.

## Configuration Steps

1. **Create an XML File**:
   Save the following as `undetectable-vm.xml`. Replace `YOUR-UUID-HERE` with a UUID from `uuidgen`.
   ```xml
   <domain xmlns:qemu="http://libvirt.org/schemas/domain/qemu/1.0" type="kvm">
     <name>UndetectableVM</name>
     <uuid>YOUR-UUID-HERE</uuid>
     <memory unit="KiB">8388608</memory>
     <currentMemory unit="KiB">8388608</currentMemory>
     <memoryBacking>
       <source type="memfd"/>
       <access mode="shared"/>
     </memoryBacking>
     <vcpu placement="static">8</vcpu>
     <os firmware="efi">
       <type arch="x86_64" machine="pc-q35-8.2">hvm</type>
       <loader readonly="yes" type="pflash">/usr/share/edk2/x64/OVMF_CODE.fd</loader>
       <nvram template="/usr/share/edk2/x64/OVMF_VARS.fd">=/var/lib/libvirt/qemu/nvram/vm_VARS.fd</nvram>
       <smbios mode="sysinfo"/>
     </os>
     <features>
       <acpi/>
       <apic/>
       <hyperv mode="custom">
         <relaxed state="on"/>
         <vapic state="on"/>
         <spinlocks state="on" retries="8191"/>
         <vendor_id state="on" value="GenuineIntel"/>
       </hyperv>
       <kvm>
         <hidden state="on"/>
       </kvm>
       <vmport state="off"/>
       <smm state="on"/>
       <ioapic driver="kvm"/>
     </features>
     <cpu mode="host-passthrough" check="none" migratable="on">
       <topology sockets="1" dies="1" cores="4" threads="2"/>
       <feature policy="disable" name="hypervisor"/>
     </cpu>
     <disk type="file" device="disk">
       <driver name="qemu" type="raw" cache="none" io="native" discard="unmap"/>
       <source file="/path/to/disk.img"/>
       <target dev="sda" bus="sata"/>
       <serial>590347474223828</serial>
       <boot order="1"/>
       <address type="drive" controller="0" bus="0" target="0" unit="0"/>
     </disk>
     <interface type="network">
       <mac address="f0:bc:8e:cd:6e:ec"/>
       <source network="default"/>
       <model type="e1000e"/>
       <address type="pci" domain="0x0000" bus="0x01" slot="0x00" function="0x0"/>
     </interface>
     <qemu:commandline>
       <qemu:arg value="-smbios"/>
       <qemu:arg value="type=0,version=UX305UA.201"/>
       <qemu:arg value="-smbios"/>
       <qemu:arg value="type=1,manufacturer=ASUS,product=UX305UA,version=2021.1"/>
       <qemu:arg value="-smbios"/>
       <qemu:arg value="type=2,manufacturer=Intel,version=2021.5,product=Intel i9-12900K"/>
       <qemu:arg value="-smbios"/>
       <qemu:arg value="type=3,manufacturer=XBZJ"/>
       <qemu:arg value="-smbios"/>
       <qemu:arg value="type=17,manufacturer=KINGSTON,loc_pfx=DDR5,speed=4800,serial=000000,part=0000"/>
       <qemu:arg value="-smbios"/>
       <qemu:arg value="type=4,manufacturer=Intel,max-speed=4800,current-speed=4800"/>
       <qemu:arg value="-cpu"/>
       <qemu:arg value="host,family=6,model=158,stepping=2,model_id=Intel(R) Core(TM) i9-12900K CPU @ 2.60GHz,vmware-cpuid-freq=false,enforce=false,host-phys-bits=true,hypervisor=off"/>
       <qemu:arg value="-machine"/>
       <qemu:arg value="q35,kernel_irqchip=on"/>
     </qemu:commandline>
   </domain>
   ```
   Update `/path/to/disk.img` to your disk image path.

2. **Customize the XML**:
   - Set `memory` and `vcpu` based on your hardware.
   - Match `machine` to your QEMU version (e.g., `pc-q35-8.2`).
   - Use `dmidecode` on a physical machine for realistic SMBIOS values.
   - Adjust CPU topology to match common hardware (e.g., 4 cores, 2 threads).

3. **Define and Start the VM**:
   ```bash
   virsh define undetectable-vm.xml
   virsh start UndetectableVM
   ```

## Key Configuration Elements

- **SMBIOS**: Sets realistic hardware IDs (e.g., ASUS, Intel) to counter WMI checks.
- **Hypervisor Concealment**: Hides KVM and disables hypervisor CPU flags.
- **CPU Passthrough**: Uses host CPU features for authenticity.
- **Devices**: Uses SATA and e1000e instead of VirtIO to avoid VM-specific drivers.
- **UEFI**: Configures OVMF for EFI booting with custom SMBIOS.

## Verification

- Boot the guest OS (e.g., Windows 10).
- Run `systeminfo` to check manufacturer/model (should show ASUS, Intel).
- Use PowerShell:
  ```powershell
  Get-WmiObject -Class Win32_BIOS
  Get-WmiObject -Class Win32_ComputerSystem
  Get-WmiObject -Class Win32_BaseBoard
  ```
- Test with anti-cheat or detection tools to confirm undetection.

## Limitations

- **RDTSC Timing**: Not addressed; may need additional patches.
- **WMI Sensors**: Missing sensor data can expose the VM.
- **Specific Anti-Cheat**: Vanguard may require extra tweaks.

## Troubleshooting

- **VM Won’t Start**: Check XML syntax, disk paths, and QEMU version.
- **Detection Failure**: Verify SMBIOS, CPU, and hypervisor settings. Rebuild QEMU if needed.
- **Permission Errors**: Ensure `sudo` for `virsh` commands.

## Next Steps

See the “Installation Guide” for the full setup process, and “Anti-Detection Techniques” for details on anti-detection strategies.