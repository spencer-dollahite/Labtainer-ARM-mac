# Running Labtainers on Apple Silicon (M series) Macs using UTM

This guide explains how to run **Labtainers**, the Linux-based cybersecurity lab environment from the Naval Postgraduate School (NPS), on an Apple Silicon (M series) Mac using **UTM**.

> **About this copy.** This guide was originally written by [CamilYed](https://github.com/CamilYed/Labtainer-ARM-mac) for a MacBook Pro M3. This fork carries corrections for the current UTM (choose **Emulate**, not Virtualize, for the x86_64 image; leave **UEFI Boot** disabled; set **Network Mode** to **Emulated VLAN**), verified on an M2 (2026-09), and adds the native arm64 appliance option. The NPS Labtainers page links here.

---

## Which option should I use?

There are two ways to run Labtainers on an Apple Silicon Mac. They differ mainly in speed.

| | **Option A: native arm64 appliance** | **Option B: official x86_64 QCOW2** |
|---|---|---|
| UTM mode | **Virtualize** (Apple's hypervisor runs arm64 code directly) | **Emulate** (QEMU translates x86_64 instructions in software) |
| Speed | Near native. Boots in seconds; labs feel like a local Linux box. | Slow. Expect a boot measured in minutes and sluggish desktop and Docker work, since every instruction is translated. Usable, but plan for it. |
| Lab coverage | Only labs whose Docker images have been published for arm64. Your instructor will tell you whether your course is covered. | Every Labtainers lab, exactly as on x86 hardware. |
| Where to get it | Download the `.utm.zip` from the NPS OneDrive folder: [Labtainers arm64 appliance](https://nps01-my.sharepoint.com/:f:/g/personal/spencer_dollahite_nps_edu/IgAZgs-majINRZLvHu-oPPj0AQei1zJ9S0xHeJThBPQamjE) (or the copy your instructor gives you). | Hosted by NPS today: [nps.edu/web/c3o/virtual-machine-images](https://nps.edu/web/c3o/virtual-machine-images) |

**Rule of thumb:** if your course is covered by the arm64 labs, use Option A. Otherwise use Option B; it is the same image NPS supports for everyone, it just runs slower on Apple Silicon.

---

## Table of Contents
1. [Which option should I use?](#which-option-should-i-use)
2. [For students](#for-students)
    - [Option A: native arm64 appliance (fast)](#option-a-native-arm64-appliance-fast)
    - [Option B: official x86_64 QCOW2 image (slower, all labs)](#option-b-official-x86_64-qcow2-image-slower-all-labs)
3. [For instructors and lab designers: building the image yourself](#for-instructors-and-lab-designers-building-the-image-yourself)
    - [Preparing the Labtainer Image Manually](#preparing-the-labtainer-image-manually)
    - [Setting Up the Virtual Machine in UTM](#setting-up-the-virtual-machine-in-utm)
    - [Building the arm64 appliance](#building-the-arm64-appliance)
4. [Overview of Tools and File Formats](#overview-of-tools-and-file-formats)

---

## For students

Both options need UTM. Download it once:

- [UTM Download Link](https://github.com/utmapp/UTM/releases/latest/download/UTM.dmg)

Open the `.dmg` file and drag UTM to your Applications folder.

### Option A: native arm64 appliance (fast)

> **Download:** the appliance (`.utm.zip` + its `.sha256` checksum) is in the NPS OneDrive folder [Labtainers arm64 appliance](https://nps01-my.sharepoint.com/:f:/g/personal/spencer_dollahite_nps_edu/IgAZgs-majINRZLvHu-oPPj0AQei1zJ9S0xHeJThBPQamjE). Your instructor may also hand you a copy directly. If your course is not covered by the arm64 labs, use [Option B](#option-b-official-x86_64-qcow2-image-slower-all-labs).

The appliance is a ready-made UTM bundle (a `.utm.zip` file). Everything is pre-configured: **Virtualize**, **aarch64**, 4 cores, 8 GB RAM, UEFI boot, **Emulated VLAN** networking, and clipboard sharing with macOS. You do not create a VM by hand.

1. **Verify the download** against the `.sha256` file in the same folder (or the checksum your instructor gave you). In Terminal: `shasum -a 256 ~/Downloads/<file>.utm.zip` and compare.
2. **Unzip** the file. You get a `.utm` bundle.
3. **Double-click the `.utm` bundle** (or in UTM choose **File > Open**). UTM imports it into its library.
4. **Start the VM** and log in with the credentials supplied with the download.
5. Open a terminal and run your course's lab exactly as your instructor describes. The first start of each lab pulls its arm64 Docker images, so it needs internet access; later starts are offline-capable.

If the VM loses its network after the Mac sleeps or changes Wi-Fi, shut it down and start it again. Do not switch the network mode away from **Emulated VLAN**.

### Option B: official x86_64 QCOW2 image (slower, all labs)

NPS provides an official QCOW2 image of the standard Labtainers VM, prepared from the instructions in the instructor section below. Because it is an x86_64 image, UTM must **emulate** the CPU; that is why it is slow, and it is the reason the settings below matter.

#### Step 1: Download the QCOW2 image

- [Official QCOW2 Image for Labtainer on Mac](https://nps.edu/web/c3o/virtual-machine-images)

#### Step 2: Set up the VM in UTM

After downloading the QCOW2 image:

1. **Unzip the file**.
2. Open **UTM** and create a new virtual machine:
    - Click the **+** button and select **Emulate** (not Virtualize; the image is x86_64 and your Mac is arm64).
    - Follow the wizard steps:
        - **System Settings**: Architecture **x86_64**; assign at least 4 GB of memory (8 GB recommended).
        - Complete the wizard to create the VM. Do not worry about the disk settings during this step.
    - After the VM is created, open its configuration settings:
        - **System > Boot Options**: **uncheck UEFI Boot**. The image will not boot with it enabled.
        - **Drives**: add a new drive and select the **QCOW2** file as the drive source.
        - **Network**: set **Network Mode** to **Emulated VLAN** (see [Step 6](#step-6-configure-the-vm-settings) for why).
3. **Save and start the VM**. The first boot takes several minutes under emulation; that is normal. Refer to the [YouTube tutorial](https://youtu.be/ckBRtSlhcww) if you need guidance.

---

## For instructors and lab designers: building the image yourself

### Preparing the Labtainer Image Manually

This part is for **instructors and lab designers** who want to build the x86_64 image themselves (for example to package a newer Labtainers release, or to understand what the official QCOW2 contains). Students do not need any of it.

#### Step 1: Download UTM
UTM is a virtual machine manager for macOS that supports running VMs on both x86_64 and ARM-based Macs. Download the latest version of UTM for macOS from GitHub:

- [UTM Download Link](https://github.com/utmapp/UTM/releases/latest/download/UTM.dmg)

After downloading, open the `.dmg` file and install UTM by dragging it to your Applications folder.

#### Step 2: Download Labtainer OVA File
Labtainer is distributed as an OVA file. Download the pre-configured Labtainer OVA image for VMware:

- [Labtainer OVA Download Link](https://nps.box.com/shared/static/2582mm4x58mn6rqy049no0bspj5vdmqv.ova)

#### Step 3: Install QEMU via Homebrew
To convert the OVA file to a QCOW2 format compatible with UTM, we’ll need **QEMU**, a free and open-source emulator. Install QEMU via Homebrew:

```bash
brew install qemu
```

> **Note**: Homebrew is a package manager for macOS. If you don’t have it installed, follow the instructions on [Homebrew’s website](https://brew.sh).

#### Step 4: Convert OVA to QCOW2 Format
The OVA file is essentially a compressed archive. To extract it, use the following command in your terminal:

```bash
tar -xvf LabtainerVM24a-VMWare.ova
```
![Extracting OVA file](images/unpacke-ova.png)

After extracting, you should see several files, including a `.vmdk` file (the virtual disk for VMware). We will convert this `.vmdk` file to the QCOW2 format:

```bash
qemu-img convert -O qcow2 LabtainerVM-VMWare-disk1.vmdk labtainer-utm-vm.qcow2
```

This command creates a file called `labtainer-utm-vm.qcow2`, which is compatible with UTM.

---

---

### Setting Up the Virtual Machine in UTM

#### Step 5: Create a New Virtual Machine in UTM
1. Open **UTM** and click the **+** button to create a new virtual machine.
2. Select **Emulate** to create a virtual machine with x86_64 emulation.

#### Step 6: Configure the VM Settings

On the original author's MacBook Pro M3 (36 GB RAM) the following configuration was used; the same settings were verified on an M2:

1. **System**:
    - **Architecture**: Set to **x86_64** (even though your Mac is ARM, UTM can emulate x86).
    - **CPU Cores**: Set to **4 cores**.
    - **Multithreading**: Enabled (recommended to improve performance).
    - **Boot Options**: Disable **UEFI Boot** (uncheck this option).

2. **Memory**: Allocate **8 GB of RAM** for the virtual machine. If you have less RAM, you may need to adjust this, but 8 GB is recommended for stable performance in Labtainers.

3. **Drive**: After creating the VM:
    - Open the VM’s configuration settings.
    - Add a new drive, select **Browse**, and attach the `labtainer-utm-vm.qcow2` file.

4. **Display**:
    - **Graphics Card**: Select **virtio-gpu-gl-pci (GPU Supported)** to enable better graphical performance.
    - **SPICE**: Ensure **SPICE** is selected if you need enhanced display options, such as clipboard sharing.

5. **Network**:
    - **Network Mode**: Set to **Emulated VLAN**.
    - Why: **Shared Network** relies on macOS's own DHCP service for the VM's IPv4 address, and that service often stops answering after the Mac sleeps, changes Wi-Fi networks, or connects to a VPN. The guest then shows only IPv6 addresses on `enp0s1` and reports `Network is unreachable`, so Labtainers cannot pull its Docker images. **Emulated VLAN** has QEMU provide NAT and DHCP inside the VM itself, with no dependency on the host, so the VM keeps its internet access. This is also the mode UTM recommends for emulated (non-native) guests.
    - If a VM you already created loses its network, shut it down, change this setting, and start it again.

#### Step 7: Boot and Test the VM
1. Start the VM in UTM.
2. Once the VM boots, log in and check that Labtainers is operational.

> **Additional Resources**: For a visual guide on how to import a QCOW2 image into UTM, you may find this [YouTube tutorial](https://www.youtube.com/watch?v=enF3zbyiNZA) helpful.


### Building the arm64 appliance

The native appliance (Option A) is not derived from the OVA. It starts from a stock Ubuntu arm64 cloud image, installs Labtainers natively, and is packaged as a UTM bundle with the settings listed under Option A. It only runs labs whose Docker images are published for `linux/arm64`, so a course has to build and publish its lab images for both architectures before its students can use it. The build tooling lives with the course's lab sources rather than in this guide; when a public download exists, it will be linked from the student section above.

---

## Overview of Tools and File Formats

### What is an OVA file?
An **OVA** (Open Virtualization Archive) file is a standardized file format that packages all components of a virtual machine into a single file, making it easy to share and distribute VMs. OVA files can contain the VM disk image (usually in VMDK format), configuration files, and metadata.

### What is QCOW2?
**QCOW2** (QEMU Copy-On-Write version 2) is a disk image format used by QEMU. It supports advanced features such as snapshots, compression, and encryption, ideal for use in virtualized environments like UTM.
