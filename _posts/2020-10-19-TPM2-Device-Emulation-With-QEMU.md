# Introduction
End-to-end development with physical hardware can be challenging due to a myriad
of factors. Things like persistent state, physical wear, slow and difficult
to update hardware bugs, lagging features, etc can pose additional hurdles to
development tasks.

A potential way to overcome this, is to use [QEMU](https://www.qemu.org) instance with an attached
virtualized TPM2.0 device. This device is made available to the guest OS, and
with the appropriate versions of Linux, will expose the familiar /dev/tpm0
and /dev/tpmrm0 interfaces.

In this brief tutorial, we provide instructions on how to build such a system by leveraging other documentation as required.

# Prerequisites
Prior art does exist on this topic, and details used in this tutorial have references from
the following resources:
  - https://www.qemu.org/docs/master/specs/tpm.html
  - https://graspingtech.com/ubuntu-desktop-18.04-virtual-machine-macos-qemu/

## Install the proper TPM2.0 Simulator

In this tutorial, we demonstrate how to leverage the [swtpm](https://github.com/stefanberger/swtpm) as the TPM simulator.
The project wiki has instructions for building and installing the simulator and its dependency, [libtpms](https://github.com/stefanberger/libtpms).

## Install QEMU

Next, you need to install QEMU. This is operating system dependent. Details on installing
QEMU can be found by visiting their website:
  - https://www.qemu.org/

**The minimum version of QEMU to support this is 4.0. In this tutorial, the author tested with version 5.2.**

# Install The Guest OS

The author installed Ubuntu 20.04, so the commands will be specific to that ISO, but another ISO
could be substituted. Additionally, the naming convention on things like hard-drive could be changed
to reflect your environment more closely.

Install the guest OS. This will be guest-OS specific. The general commands are to build
a virtual disk:
```console
qemu-img create -f qcow2 ubuntu-20.04-amd64.img 30G
```

Then attach it to a VM and start it with the installation media, usually an ISO:
```console
qemu-system-x86_64 -hda ~/qemu-images/ubuntu-20.04-amd64.img -boot d -cdrom ~/Downloads/ubuntu-20.04.1-desktop-amd64.iso -m 2048 -enable-kvm
```

# Start the Guest with a TPM2.0 Device

Now start the guest with a virtualized TPM2.0 device. To do this, one needs to start the SWTPM simulator in tpm2 mode using the option
`--tpm2`, like so:
```console
mkdir /tmp/emulated_tpm
swtpm socket --tpmstate dir=/tmp/emulated_tpm --ctrl type=unixio,path=/tmp/emulated_tpm/swtpm-sock --log level=20 --tpm2
```

Then start the guest:
```console
qemu-system-x86_64 -hda ~/qemu-images/ubuntu-20.04-amd64.img -boot d -m 2048 -enable-kvm \
  -chardev socket,id=chrtpm,path=/tmp/emulated_tpm/swtpm-sock \
  -tpmdev emulator,id=tpm0,chardev=chrtpm -device tpm-tis,tpmdev=tpm0
```

Now verify that the device nodes are present in the guest VM by opening a console and running the following command:
```console
ls /dev/tpm*
```

You should see `/dev/tpm0` and `/dev/tpmrm0` devices in the output of the `ls` command.

# Conclusion

One of the major benefits to the emulated environment is being able to test end-to-end development without the need for physical
hardware and it's associated drawbacks across a wide variety of environments. QEMU has the ability to emulate multiple physical
CPU architectures. Couple that with the ability to install a wide array of operating systems, and you have a flexible system for
debugging and building new features from the lowest portions of the stack all they way to end client applications.

# Author
William Roberts
