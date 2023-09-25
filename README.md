# cloudinit-breakout-un-deRSE23
Repo for the Breakoutsession https://github.com/DE-RSE/un-deRSE23-breakouts/issues/16 of the un-deRSE23

# Local Virtual Machine Setup with Cloud-Init

This tutorial guides you through setting up a virtual machine (VM) provisioned by cloud-init on your local machine using QEMU as the hypervisor. This allows you prior to deploying your VM to a public environment, that all provisioning steps are working as expected.

## Prerequisites

While you can use any OS, such as a Linux distribution, macOS, or Windows, this tutorial focuses on Ubuntu 22.04 commands. Ensure you have root access for the operations.

Verify your OS version:

```bash
lsb_release -a
```

Expected output:

```
Ubuntu 22.04 jammy
```

Check if virtualization is supported on your system:

```bash
LC_ALL=C lscpu | grep Virtualization
```

## Installation and Setup

### Install Necessary Software

```bash
sudo apt update
sudo apt install qemu-system-x86 python3
```

### Clone Cloud-Init Configuration

```bash
git clone git@gitlab.com:dmp4cs/server-cloud-init.git
cd server-cloud-init
```

### Obtain a Cloud-Init Enabled Image

This tutorial employs the Debian cloud-init enabled image (opt for the generic version). Download it:

```bash
wget https://cloud.debian.org/images/cloud/bookworm/20230910-1499/debian-12-generic-amd64-20230910-1499.qcow2 -O debian.qcow2
```

Other distribution images are available:

- [Arch Linux](https://gitlab.archlinux.org/archlinux/arch-boxes)
- [Ubuntu](https://cloud-images.ubuntu.com/)

### Generate SSH Keys for Testing

```bash
ssh-keygen -t ed25519 -f ~/.ssh/demo
```

## Configuration Details

### meta-data
This file is utilized for specifying the hostname and instance ID.

### vendor-data
This is typically provided by your cloud provider. It can remain empty for a local setup.

### user-data
This primary configuration file encompasses:

- User setup with password or SSH authentication
- Cross-distribution package installations
- Shell script executions

To explore additional features, refer to the future link: TODO link

## Data Sources

Cloud-init requires a method to access your config files when the VM starts for the first time. While cloud providers often facilitate this through a web interface or API, for our local setup, we'll use the NoCloud Datasource.

More about data sources can be found [here](https://cloudinit.readthedocs.io/en/latest/reference/datasources.html).

## Initiating the VM

Prepare a snapshot image:

```bash
qemu-img create -f qcow2 -b debian.qcow2 -F qcow2 snapshot.qcow2
```

In a new terminal, while in the same directory, initiate an HTTP server to make the files accessible to cloud-init:

```bash
python3 -m http.server
```

### Modify Configuration

Update the `user-data` file by inserting your SSH key.

### Start the VM with QEMU

Execute the following command:

```bash
sudo qemu-system-x86_64 -machine accel=kvm:tcg -cpu host -m 512 -nographic -drive format=qcow2,file=snapshot.qcow2 -smbios type=1,serial=ds='nocloud-net;s=http://10.0.2.2:8000/' -nic user,id=vmnic,hostfwd=tcp::5555-:22 
```

**Explanation of Parameters**:
- `qemu-system-x86_64` emulate a X86-64 Bit desktop architecture
- `-machine accel=kvm:tcg`: Use efficient hardware virtualization when available.
- `-cpu host`: Emulate the host's CPU.
- `-m 512`: Allocate 512 MiB RAM for the VM.
- ```-nographic``` Don't emulate a GPU, text mode only
- `-drive format=qcow2,file=snapshot.qcow2`: Designate our snapshot as the VM's hard drive.
- `-smbios type=1,serial=ds='nocloud-net;s=http://10.0.2.2:8000/'`: Use the specified method as a data source. More details [here](https://cloudinit.readthedocs.io/en/latest/reference/datasources/nocloud.html#method-3-custom-webserver-kernel-commandline-or-smbios). The server address for the source is set as bios parameter in the virtual machine.
- ```-nic user,id=vmnic,hostfwd=tcp::5555-:22 ``` setup virtual  network with port forwarding for SSH from port 22 (guest) to port 5555 host


To exit QEMU, use `ctrl-a x`. This is first Control and A together, release and then x.

### Handy Cloud-Init Commands

Check the status:

```bash
cloud-init status --wait
```

To execute cloud-init on the next boot:

```bash
sudo cloud-init clean --logs
```
