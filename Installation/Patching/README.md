# Building Patched QEMU for Undetectable VMs

Building a patched version of QEMU is the first step to creating an undetectable virtual machine (VM). The `qemu-anti-detection` project provides patches to modify QEMU, masking VM-specific indicators so the guest OS sees it as physical hardware. This guide walks you through the build process.

## Why Patch QEMU?

QEMU’s default setup exposes virtualization traces, like “QEMU” device names or hypervisor flags, that anti-cheat, malware, or protection software can detect. Patching QEMU lets you:
- Rename virtual hardware to mimic real devices.
- Mask system info and hypervisor signatures.
- Evade most detection methods.

## Prerequisites

- **Operating System**: Arch Linux, Ubuntu, or Debian-based distro.
- **Packages**:
  - Arch: `sudo pacman -S git wget base-devel glib2 ninja python`
  - Ubuntu/Debian: `sudo apt install git build-essential ninja-build python-venv libglib2.0-0 flex bison`
- **QEMU**: Keep a package-managed QEMU installation to ensure runtime dependencies. Patched QEMU installs to `/usr/local/bin`, which takes precedence.
- **Root Access**: Required for installation.

## Build Steps

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/zhaodice/qemu-anti-detection.git
   ```

2. **Download QEMU Source**:
   This example uses QEMU 8.2.2, but check the repository for supported versions.
   ```bash
   wget https://download.qemu.org/qemu-8.2.2.tar.xz
   tar xvJf qemu-8.2.2.tar.xz
   cd qemu-8.2.2
   ```

3. **Apply the Patch**:
   ```bash
   git apply ../qemu-anti-detection/qemu-8.2.0.patch
   ```
   The `qemu-8.2.0.patch` is compatible with QEMU 8.2.2 despite the version name.

4. **Configure QEMU**:
   ```bash
   ./configure
   ```

5. **Build and Install**:
   ```bash
   sudo make install -j$(nproc)
   ```
   Uses all CPU cores for faster compilation.

6. **Verify Installation**:
   ```bash
   /usr/local/bin/qemu-system-x86_64 --version
   ```
   Should show QEMU 8.2.2.

## What the Patch Changes

The patch modifies QEMU components to hide VM traces:
- **Device Names**: Replaces “QEMU” with “ASUS” or “Intel” for hardware like disks, keyboards, and USBs.
- **SMBIOS/ACPI**: Alters system info to mimic physical machines.
- **Hypervisor**: Hides KVM signatures and CPU feature bits.
- **Serial Numbers**: Changes VM-specific serials to realistic ones.
- **UEFI**: Disables VM-specific boot logos.

## Supported QEMU Versions

The repository includes patches for:
- 8.2.x, 8.2.0, 8.1.x, 8.0.2/8.0.5
- 7.2.x, 7.0.x, 6.2.x

Pick the patch closest to your QEMU version.

## Troubleshooting

- **Patch Failure**: Ensure QEMU version compatibility. Try a different patch if needed.
- **Missing Dependencies**: Install required packages (e.g., glib2.0-dev` on Ubuntu).
- **Compilation Errors**: Check patch notes for known issues or version mismatches.
- **Permission Issues**: Use `sudo` for `make install`.

## Next Steps

After building, configure your VM with anti-detection settings (see “VM Configuration Setup”). Test the VM to ensure it passes detection checks.

See the “Installation Guide” for the complete setup process, and “Anti-Detection Techniques” for more on how the patches work.
