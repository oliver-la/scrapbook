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
Traffic can be separated by giving the port groups a VLAN id other than 0.
Questions:
  - Can I access the VLAN from the outside?
  - Does a VLAN have to be on its own subnet?
  - Why use a VLAN if I can just create a new subnet?
  - Does a VLAN need a router to work? What do I put as the gateway?
  - What do I put as the gateway when using subnets? Can I see devices on a different subnet, if I just give my client a subnet mask of 255.255.0.0? e.g. access 192.168.10.1 from 192.168.20.1
