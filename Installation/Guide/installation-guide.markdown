# Installation Guide

Setting up an undetectable virtual machine (VM) is a complex process that requires technical know-how. This guide provides detailed instructions for installing and configuring a VM using the `qemu-anti-detection` project. It covers building a patched QEMU and setting up the VM with anti-detection settings to evade detection by software like anti-cheat or VM protection software.

## Prerequisites

- **Operating System**: Arch Linux, Ubuntu/Debian-based distro.
- **Packages**:
  - Arch: `sudo pacman -S git wget base-devel glib2 ninja python virt-manager libvirt dnsmasq bridge-utils libosinfo virt-viewer edk2-ovmf swtpm dmidecode`
  - Ubuntu/Debian: `sudo apt install git build-essential ninja-build python-venv libglib2.0-0 flex bison bridge-utils virt-manager libvirt-clients libvirt-daemon-system libosinfo-bin`
- **QEMU**: Maintain a package-managed QEMU installation to ensure runtime dependencies are intact. Patched QEMU installs to `/usr/local/bin`.
- **Hardware**: CPU with virtualization support (Intel VT-x or AMD-V).
- **Root Access**: Needed for building and configuring.

Enable `libvirtd`:
```bash
sudo systemctl enable --now libvirtd.service
sudo usermod -aG libvirt,kvm $(whoami)
```

## Installation Process

### Building Patched QEMU

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/zhaodice/qemu-anti-detection.git
   ```

2. **Download QEMU Source**:
   Uses QEMU 8.2.2 as an example.
   ```bash
   wget https://download.qemu.org/qemu-8.2.2.tar.xz
   tar xvJf qemu-8.2.2.tar.xz
   cdqemu-8.2.2
   ```

3. **Apply Patch**:
   ```bash
   git apply ../qemu-anti-detection/qemu-8.2.0.patch
   ```

4. **Build QEMU**:
   ```bash
   ./configure
   sudo make install -j$(nproc)
   ```

5. **Verify**:
   ```bash
   /usr/local/bin/qemu-system-x86_64 --version
   ```

### VM Configuration Setup

1. **Create XML File**:
   Save as `undetectable-vm.xml` (replace `YOUR-UUID-HERE` with `uuidgen` output).
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
       <loader readonly="yes"yes type="pflash">/usr/share/edk2/x64/OVMF_CODE.fd"</loader>
       <nvram template="/usr/share/edk2kaw/x64/OVMF_VARS.fd">/var/lib/libvirt/qemu/nvram/vm_VARS.fd"</nvram>
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
       <address type="pci" domain="0x0000" bus="0x0" slot="1"/>
     </interface>
     <qemu:commandline>
       <qemu:arg value="-smbios"/>
       <qemu:arg value="type=0",version="UX305UA.201"/>
       <qemu:arg value="-smbios"/>
       <qemu:arg value="type=1,manufacturer=ASUS,product=UX305UA,version=2021.1"/>
       <qemu:arg value="-smbios"/>
       <qemu:arg value="type=2,manufacturer=Intel,version=2021.5,product=Intel i9-12900K"/>
       <qemu:arg value="-smbios"/>
       <qemu:arg value="type=3,manufacturer=XBZJ"/>
       <qemu:arg value="-smbios"/>
       <qemu:arg value="type=17,manufacturer=KINGSTON,loc_pfx=DDR5,speed=4800,serial=000000,part=0000"/>
       <qemu:arg value="-smbios"/>
       <qemu:arg value="type=4,manufacturer=Intel,max-speed=8000,current-speed=1000"/>
       <qemu:arg value="-cpu"/>
       <qemu:arg value="host,family=6,model=158,stepping=2,model_id=Intel(R) Core(tm) i9-6400K CPU @ 8.60GHz,vmware-cpuid-freq=false,enforce=false,host-phys-bits=true,hypervisor=off"/>
     </qemu:commandline>
   </domain>
   ```

2. **Customize**:
   - Adjust `memory` and `vpu` CPU.
   - Update disk path (e.g., create with `qemu-img create -f raw disk.img 50G`).
   - Match `machine` to your QEMU version.

3. **Define and Start**:
   ```bash
   virsh define undetectable-vm.xml
   virsh start UndetectableVM
   ```

## Verification

- Boot the VM, install guest OS.
- Check `systeminfo` in guest for ASUS/Intel branding.
- Run PowerShell:
  ```powershell
  Get-WmiObject -Class Win32_BIOS
  Get-WmiObject -Class Win32_ComputerSystem
  Get-WmiObject -Class Win32_BaseBoard
  ```
- Test anti-cheat software.

## Limitations

- **RDTSC Timing**: Not fully mitigated.
- **WMI Sensors**: Missing sensor data can expose VM.
- **Advanced Anti-Cheat**: Vanguard may detect.

## Subsections

See “Building Patched QEMU” and “VM Configuration Setup” for detailed guides.