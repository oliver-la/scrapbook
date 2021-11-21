# ESXi conventions

- ESXi works well if installed on a usb thumb drive, as most of the config is stored with the VMs themselves anyway
  - Only exception is networking, and host-specific config
- For the love of god, do not create a datastore on the very same thumb drive, OR get the bright idea to create a datastore on a dedicated usb thumb drive.
  - To host a NAS in a VM and use it as the main datastore, consider installing a SSD drive just for the VM. Even if it's just the bootloader.
  - => ESXi needs a datastore to write "scrap" files to. The SSD will be used for writing as well. A flash drive will die much quicker if it has a datastore on it.

## Networking

### Port Groups

A port group either connects ONE VMKernel or multiple VMs to a vSwitch. Port groups can be segmented into VLANs, thus allowing multiple VMKernels on the same vSwitch.

### VMKernel

- A VMKernel essentially connects the ESXi host to the vSwitch. If there was none, the ESXi host wouldn't see the network.
- A VMKernel cannot share its port group with other VMs.

### vSwitch

- Requires at least one physical uplink if a VMKernel is supposed to join the network.
  - In my tests, the two port groups (VM Portgroup, VMKernel portgroup) were not connected with each other in the vSwitch topology.
- VMs sharing the same port group, on the same vSwitch, (theoretically) do not need a physical uplink.
- Not sure if vSwitches have limits in terms of bandwidth. Only option was to set jumbo frames via MTU.
  - If there's no remote traffic incoming/outgoing, does the physical NIC still process the packets? (tests suggest no, as I was able to reach 10Gbps bandwidth between two VMs on a 1Gbps NIC)

### VLAN

(still testing this)
Traffic can be separated by giving the port groups a VLAN id other than 0. Either a managed switch or a router (virtual or physical) is required, otherwise devices in the same VLAN (even if on the same host and port group) are not able to communicate with each other. VLAN is a feature the router has to support. Some routers have the option to create a guest wifi network in an dedicated subnet. In my case, the Synology Router RT2600ac doesn't support custom VLANs, but it does include a guest network. So I just used that, since I rarely have guests anyway to justify the use of a guest wifi. 

The VLAN ID for the Guest Network on Synology Router RT2600ac is 1733.

Questions I had (no clue about networking mind you) and was able to answer myself:

  - Can I access the VLAN from the outside?
    - Yes, if the router has a static route set up to allow subnets to talk to each other. (static routes on the client using the router as gateway should also be possible)
  - Does a VLAN have to be on its own subnet?
    - Yes
  - Why use a VLAN if I can just create a new subnet?
    - A VLAN is a subnet. If we're talking about separating a network, then VLAN is the technical term.
  - Does a VLAN need a router to work? What do I put as the gateway?
    - Yes, and the gateway should be a router (virtual or physical)


## UPS

I have an APC UPS C350 with a RJ45 data port. The data port is NOT ethernet, a RJ45-USB cable (APC AP9827) is required. ESXi 6.7 doesn't have native support for UPS, but there's a community .vib to add a UPS server client. (UPS server = NUT server)
The .vib can be found here: http://www.networkupstools.org/download.html

My solution is to pass-through the UPS, connected with the USB-cable to the ESXi host, to a virtual machine running Alpine Linux. (but you can use any distro, just skip alpine linux specific stuff I'll mention below)

### Preparing the virtual machine (Alpine Linux)

Initially, the UPS was detected by Alpine Linux, but there was only a /dev/hidraw1 device. As far as I know, hidraw is used as a fallback if no suitable driver was able to claim the device. Meaning, neither apcupsd nor "nut" was able to find or talk to the UPS. Now there's 2 issues with Alpine Linux. First, if the linux-virt kernel is used, you won't have the proper drivers installed. Ain't nobody got time to research how to add the driver, so the fix is simple: `apk add linux-lts` and make sure it's used as the default in the /boot/syslinux.conf file. `linux-virt` is a slimmed down version of `linux-lts` suitable for virtual environments. But linux-lts will work just fine in a VM.
Next, and I wasn't sure if I fixed the issue with the kernel switch already, Alpine Linux uses mdev instead of udev. mdev/udev is basically responsible to discover devices and add them to /dev/. Using mdev was asking for trouble, because udev does things differently and is way more common.

### General setup

I use both apcupsd and nut. Nut has a wrapper for apcupsd and doesn't talk to the UPS directly. apcupsd does. The .vib I mentioned above is a NUT client so naturally you'd want to have a NUT server.
apcupsd is configured with usb mode, and usb device. The DEVICE directive can be left blank. If you use udev, apcupsd should be able to discover it. mdev might cause it to not detect it. The default is that apcupsd will shutdown your virtual machine in case of power failure. But this won't shutdown the ESXi host. Thus, disable the "poweroff" stage. (refer to the apcupsd manual)

NUT is configured as a "netserver", with a LISTEN directive and a user with nothing more than a password and the "slave" flag.

ESXi can now be configured to talk to the NUT server on the VM. Simply search for "nut" in the advanced configuration section.
