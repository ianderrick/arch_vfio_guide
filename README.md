# arch_vfio_guide

**Step One: Enable IOMMU**

1. Go to /boot/loader/entries/arch.conf:
```
title Arch Linux
linux /vmlinuz-linux
initrd /intel-ucode.img
initrd /initramfs-linux.img
options root=PARTUUID=71ec4afa-3461-4f9c-9f3e-becc45cbeafc intel_iommu=on pci_stub.ids=10de:13c2,10de:0fbb
rw
```
2. Add "intel_iommu=on" The ids will be added later.

**Step Two: Locate GPU and GPU audio**

1. Run command `lspci` where you should see your GPU's video and audio output:
```
01:00.0 VGA compatible controller: NVIDIA Corporation GM204 [GeForce GTX 970] (rev a1)
01:00.1 Audio device: NVIDIA Corporation GM204 High Definition Audio Controller (rev a1)
```
2. Make a note of IOMMU numbers and grouping, for example 01:00.0 and 01:00.1 in this case. Make sure they are grouped together and not split, such as 01:00.0 and 01:00.1 versus 01:00.0 and 03:00.2.

3. Enter in the command `lspci -nk` and find the ID of your GPU and GPU audio:
```
01:00.0 0300: 10de:13c2 (rev a1)
Subsystem: 1043:8508
Kernel driver in use: vfio-pci
Kernel modules: nouveau
01:00.1 0403: 10de:0fbb (rev a1)
Subsystem: 1043:8508
Kernel driver in use: vfio-pci
Kernel modules: snd_hda_intel
```
4. Note the ids: "10de:13c2" and "10de:0fbb".
5. As mentioned back at step one, add `pci_stub.ids=10de:13c2,10de:0fbb` to the /boot/loader/entries/arch.conf file.
6. Reboot, run the command `lspci -nk` and look at the ids again. You should now see `pci-stub` after `kernel in use`.

**Step Three: Install Virt-Manager**

1. Run the command `sudo pacman -S qemu virt-manager samba libvirt`

2. Run the command `systemctl enable libvirtd`

**Step Four: Setup Virtual Machine**

1. Open Virt-Manager which should show QEMU/KVM in the window.
2. Create new machine.
3. When making the drive, you can point to specific hardware, the XML will look like this:

```
<disk type="block" device="disk">
  <driver name="qemu" type="raw" cache="none" io="native" discard="unmap"/>
  <source dev="/dev/disk/by-id/ata-Samsung_SSD_850_EVO_250GB_S21NNSAG461307K-part1"/>
  <target dev="sda" bus="sata"/>
  <boot order="1"/>
  <address type="drive" controller="0" bus="0" target="0" unit="0"/>
</disk>
```
4. Click on `Add hardware` and add your GPU and GPU audio.
5. Start the VM and install Windows. Setup as needed with your usual programs and GPU drivers.
6. Once finished. shutdown the VM.
7. In your VM's config, remove the following:
```
Display Vice
USB Redirectors
Video QXL
Channel Splice
Channel 1
```
