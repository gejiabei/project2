.. highlight:: console

Using SR-IOV
============

SR-IOV is a PCI feature that allows virtual functions (VFs) to be created from
a physical function (PF). The VFs thus share the resources of a PF, while VFs
remain isolated from each other. The isolated VFs are typically assigned to
virtual machines (VMs) on the host. In this way, the VFs allow the VMs to
directly access the PCI device, thereby bypassing the host kernel.

Installing the SR-IOV capable firmware
--------------------------------------

Before installing the SR-IOV capable firmware, ensure that SR-IOV is enabled in
the BIOS of the host machine. If SR-IOV is disabled or unsupported by the
motherboard/chipset being used, the kernel message log will contain a
``PCI SR-IOV:-12`` error when trying to create a VF at a later stage. This can
be queried using the ``dmesg`` tool.

The firmware currently running on the SmartNIC can be determined by the
``ethtool`` command. As an example, Ubuntu 18.04 LTS contains the following
upstreamed firmware::

    # ethtool -i enp2s0np0 | head -3
    driver: nfp
    version: 4.15.0-20-generic SMP mod_unloa
    firmware-version: 0.0.3.5 0.22 nic-2.0.4 nic

From the above output, the upstreamed firmware is ``nic-2.0.4``. The prefix
``nic`` indicates that the firmware implements the basic NIC functionality. The
suffix ``2.0.4`` indicates the firmware version.

Firmware ``sriov-2.1.x`` or greater provides SR-IOV capability. There are two
methods in which the firmware can be obtained, either from the
``linux-firmware`` package or from the support site.

The ``linux-firmware`` package
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The SR-IOV capable firmware has been upstreamed into the ``linux-firmware``
package. For ``rpm`` packages, this will be available from ``linux-firmware``
``20181008-88`` onwards. As of Ubuntu 18.10, the ``linux-firmware`` Debian
package does not yet contain SR-IOV capable firmware.

Ensure that the latest ``linux-firmware`` package is installed.

For RHEL / Fedora / CentOS::

    # yum update linux-firmware

The ``linux-firmware`` package will store the Netronome firmware files in the
``/lib/firmware/netronome`` directory. This directory contains symbolic links
which point to the actual firmware files. The actual firmware files will be
located in subdirectories, with each subdirectory related to a different
SmartNIC functionality. Consider the following tree structure::

    # tree /lib/firmware/netronome
    /lib/firmware/netronome/
    ├── flower
    │   ├── nic_AMDA0081-0001_1x40.nffw -> nic_AMDA0081.nffw
    │   ├── nic_AMDA0081-0001_4x10.nffw -> nic_AMDA0081.nffw
    │   ├── ...
    ├── nic
    │   ├── nic_AMDA0058-0011_2x40.nffw
    │   ├── nic_AMDA0058-0012_2x40.nffw
    │   ├── ...
    ├── nic-sriov
    │   ├── nic_AMDA0058-0011_2x40.nffw
    │   ├── nic_AMDA0058-0012_2x40.nffw
    │   ├── ...
    ├── nic_AMDA0058-0011_2x40.nffw -> nic/nic_AMDA0058-0011_2x40.nffw
    ├── nic_AMDA0058-0012_2x40.nffw -> nic/nic_AMDA0058-0012_2x40.nffw
    ├── ...

As can be seen from the tree structure, three functionalities (``flower``,
``nic``, ``nic-sriov``) are supplied by the ``linux-firmware`` package. If
``nic-sriov`` is missing, follow the :ref:`08_Using_SRIOV:The support site`
method. Point the symbolic links to the specific application required, in this
case ``nic-sriov``::

    # ln -sf /lib/firmware/netronome/nic-sriov/* /lib/firmware/netronome/

The support site
^^^^^^^^^^^^^^^^

The SR-IOV capable firmware can be obtained from the Netronome support site.
Upon downloading the packaged firmware, install the firmware files.

For Debian / Ubuntu::

    # dpkg -i agilio-sriov-firmware-2.1.x.deb

For RHEL / Fedora / CentOS::

    # yum -y install agilio-sriov-firmware-2.1.x.rpm

The ``/lib/firmware/netronome`` directory contains symbolic links which point
to the actual firmware files. When installing the above firmware package, the
symbolic links are automatically updated to point to the new SR-IOV capable
firmware files. This can be confirmed with::

    # ls -og --time-style="+" /lib/firmware/netronome
    ...
    lrwxrwxrwx 1   64  nic_AMDA0058-0011_2x40.nffw -> /opt/netronome/agilio-sriov-firmware/nic_AMDA0058-0011_2x40.nffw
    ...

Load firmware to SmartNIC
^^^^^^^^^^^^^^^^^^^^^^^^^

Remove and reload the driver. The driver will subsequently install the new
firmware to the SmartNIC::

    # modprobe -r nfp
    # modprobe nfp

The ``ethtool`` command can be used to verify that the correct firmware has
been loaded onto the SmartNIC::

    # ethtool -i enp2s0np0 | head -3
    driver: nfp
    version: 4.15.0-20-generic SMP mod_unloa
    firmware-version: 0.0.3.5 0.22 sriov-2.1.14 nic

Notice that the firmware has successfully changed from ``nic-2.0.4`` to
``sriov-2.1.14``.

.. note::

    Because the ``/lib/firmware/netronome`` directory is managed by the
    ``linux-firmware`` package, an update to this package will cause the
    symbolic links to point back to the ``nic`` firmware files. If a system
    reboot or a driver reload occurs after the links were changed, the
    incorrect firmware will be loaded to the SmartNIC. In this event, repeat
    the :ref:`08_Using_SRIOV:Installing the SR-IOV capable firmware` procedure
    to restore the desired functionality. A workaround is possible, but
    involves additional configuration of the ``initramfs`` file system.
    Customers interested in this workaround can :ref:`XX_contact_us:Contact Us`
    for more information.

Configuring SR-IOV
------------------

At this stage, there are still zero VFs, and only one PF (assuming only one
Netronome SmartNIC is installed)::

    # lspci -kd 19ee:
    02:00.0 Ethernet controller: Netronome Systems, Inc. Device 4000
            Subsystem: Netronome Systems, Inc. Device 4001
            Kernel driver in use: nfp
            Kernel modules: nfp

The number of supported VFs on a ``netdev`` is exposed by ``sriov_totalvfs`` in
``sysfs``. For example, if ``enp2s0np0`` is the interface associated with the
SmartNIC's PF, the following command will return the total supported number of
VFs::

    # cat /sys/class/net/enp2s0np0/device/sriov_totalvfs
    56

VFs can be allocated to a network interface by writing an integer to the
``sysfs`` file. For example, to allocate two VFs to ``enp2s0np0``::

    # echo 2 > /sys/class/net/enp2s0np0/device/sriov_numvfs

The new VFs, together with the PF, can be observed with the ``lspci`` command::

    # lspci -kd 19ee:
    02:00.0 Ethernet controller: Netronome Systems, Inc. Device 4000
            Subsystem: Netronome Systems, Inc. Device 4001
            Kernel driver in use: nfp
            Kernel modules: nfp
    02:08.0 Ethernet controller: Netronome Systems, Inc. Device 6003
            Subsystem: Netronome Systems, Inc. Device 4001
            Kernel driver in use: nfp_netvf
            Kernel modules: nfp
    02:08.1 Ethernet controller: Netronome Systems, Inc. Device 6003
            Subsystem: Netronome Systems, Inc. Device 4001
            Kernel driver in use: nfp_netvf
            Kernel modules: nfp

In this example, the PF is located at PCI address ``02:00.0``. The two VFs are
located at ``02:08.0`` and ``02:08.1``. Notice that the VFs are identified by
``Device 6003``, and that they use the ``nfp_netvf`` kernel driver. For RHEL
7.x systems however, the VFs will use the ``nfp`` driver.

.. note::

    If the SmartNIC has more than one physical port (phyport), the VFs will
    appear to be connected to all the phyports (as reported by the ``ip link``
    command). This happens due to the PF being shared among all VFs. In
    reality, the VFs are only connected to phyport 0.

SR-IOV VFs cannot be reallocated dynamically. In order to change the number of
allocated VFs, existing functions must first be deallocated by writing a ``0``
to the ``sysfs`` file. Otherwise, the system will return a
``device or resource busy`` error::

    # echo 0 > /sys/class/net/enp2s0np0/device/sriov_numvfs

.. note::

    Ensure any VMs are shut down and applications that may be using the VFs are
    stopped before deallocation.

In order to persist the VFs on the system, it is suggested that the system
networking scripts be updated to manage them. The following snippet illustrates
how to do this with *NetworkManager* for the PF ``enp2s0np0``:

.. code-block:: bash
    :linenos:

    cat >/etc/NetworkManager/dispatcher.d/99-create-vfs << EOF
    #!/bin/sh
    # This is a NetworkManager script to persist the maximum number of VFs on a netdev
    [ "enp2s0np0" == "\$1" -a "up" == "\$2" ] && \
        cat /sys/class/net/enp2s0np0/device/sriov_totalvfs > /sys/class/net/enp2s0np0/device/sriov_numvfs
    exit
    EOF
    chmod 755 /etc/NetworkManager/dispatcher.d/99-create-vfs

In Ubuntu systems, *networkd-dispatcher* can be used in place of
*NetworkManager*, using a similar approach to setting up the PF:

.. code-block:: bash
    :linenos:

    #!/bin/sh
    cat > /usr/lib/networkd-dispatcher/routable.d/50-ifup-noaddr << 'EOF'
    #!/bin/sh
    ip link set mtu 9216 dev enp2s0np0
    ip link set up dev enp2s0np0
    cat /sys/class/net/enp2s0np0/device/sriov_totalvfs > /sys/class/net/enp2s0np0/device/sriov_numvfs
    EOF
    chmod u+x /usr/lib/networkd-dispatcher/routable.d/50-ifup-noaddr

To enable PCI passthrough, edit the kernel command line at
``/etc/default/grub``. Add the parameters ``intel_iommu=on iommu=pt`` to the
existing command line::

    GRUB_CMDLINE_LINUX_DEFAULT="console=tty1 console=ttyS0,115200 intel_iommu=on iommu=pt"

Then::

    # update-grub

Ensure that the ``/boot/grub/grub.cfg`` file is updated with the aforementioned
parameters::

    # reboot

After reboot, confirm that the kernel has been started with the parameters::

    # cat /proc/cmdline
    BOOT_IMAGE=/boot/vmlinuz-4.15.0-20-generic root=UUID=179b45a3-def2-48b0-8f2f-7a5b6b3f913b ro console=tty1 console=ttyS0,115200 intel_iommu=on iommu=pt

Using virtio-forwarder
----------------------

virtio-forwarder is a userspace networking application that forwards
bi-directional traffic between SR-IOV VFs and virtio networking devices in QEMU
virtual machines. virtio-forwarder implements a virtio backend driver using the
DPDK's vhost-user library and services designated VFs by means of the DPDK poll
mode driver (PMD) mechanism.

.. _virtio-forwarder-site: https://virtio-forwarder.readthedocs.io/en/latest/README.html
.. _virtio-forwarder-requirements-site: https://virtio-forwarder.readthedocs.io/en/latest/README.html#requirements
.. _virtio-forwarder-hugepages-site: https://virtio-forwarder.readthedocs.io/en/latest/README.html#hugepages
.. _virtio-forwarder-installation-site: https://virtio-forwarder.readthedocs.io/en/latest/README.html#installation

The steps shown here closely correlate with the comprehensive
`virtio-forwarder <virtio-forwarder-site_>`_ docs. Ensure that the
`Requirements <virtio-forwarder-requirements-site_>`_ are met and that the
setup of :ref:`08_Using_SRIOV:Using SR-IOV` has been completed.

.. The following steps can be executed as is:

.. * `Requirements <virtio-forwarder-requirements-site_>`_
.. * `Hugepages <virtio-forwarder-hugepages-site_>`_
.. * `Installation <virtio-forwarder-installation-site_>`_

Installing virtio-forwarder
^^^^^^^^^^^^^^^^^^^^^^^^^^^

For Debian / Ubuntu::

    # add-apt-repository ppa:netronome/virtio-forwarder
    # apt-get update
    # apt-get install virtio-forwarder

For RHEL / Fedora / CentOS::

    # yum install yum-plugin-copr
    # yum copr enable netronome/virtio-forwarder
    # yum install virtio-forwarder

virtio-forwarder makes use of the DPDK library, therefore DPDK has to be
installed. Carry out the instructions of :ref:`07_Using_DPDK:Installing DPDK`.

Configuring hugepages
^^^^^^^^^^^^^^^^^^^^^

For Ubuntu, modify libvirt's apparmor permissions to allow read/write access to
the hugepages directory and library files for QEMU. Add the following lines to
the end of ``/etc/apparmor.d/abstractions/libvirt-qemu``:

.. code-block:: bash
    :linenos:

    /tmp/virtio-forwarder/** rwmix,
    # for latest QEMU
    /usr/lib/x86_64-linux-gnu/qemu/* rmix,
    # for access to hugepages
    owner "/dev/hugepages/libvirt/qemu/**" rw,
    owner "/dev/hugepages-1G/libvirt/qemu/**" rw,

Also edit the existing line, such that:

.. code-block:: bash
    :linenos:

    /tmp/{,**} r,

Restart the apparmor service::

    # systemctl restart apparmor.service

For virtio-forwarder, 2M hugepages are required whereas QEMU/KVM performs
better with 1G hugepages. It is recommended that at least 1375 pages of 2M be
reserved for virtio-forwarder. The hugepages can be configured during boot
time, for which the following should be added to the Linux kernel command line
parameters::

    hugepagesz=2M hugepages=1375 default_hugepagesz=1G hugepagesz=1G hugepages=8

Alternatively, hugepages can be configured manually after each boot. Reserve at
least 1375 * 2M for virtio-forwarder::

    # echo 2048 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages

Reserve 8G for application hugepages (modify this as needed)::

    # echo 8 > /sys/kernel/mm/hugepages/hugepages-1048576kB/nr_hugepages

Since non-fragmented memory is required for hugepages, it is recommended that
hugepages be configured during boot time.

``hugetlbfs`` needs to be mounted on the file system to allow applications to
create and allocate handles to the mapped memory. The following lines mount the
two types of hugepages on ``/dev/hugepages`` (2M) and ``/dev/hugepages-1G``
(1G)::

    # grep hugetlbfs /proc/mounts | grep -q "pagesize=2M" || \
    ( mkdir -p /dev/hugepages && mount nodev -t hugetlbfs -o rw,pagesize=2M /dev/hugepages/ )
    # grep hugetlbfs /proc/mounts | grep -q "pagesize=1G" || \
    ( mkdir -p /dev/hugepages-1G && mount nodev -t hugetlbfs -o rw,pagesize=1G /dev/hugepages-1G/ )

Finally, libvirt requires a special directory inside the hugepages mounts with
the correct permissions in order to create the necessary per-VM handles::

    # mkdir /dev/hugepages-1G/libvirt
    # mkdir /dev/hugepages/libvirt
    # chown [libvirt-]qemu:kvm -R /dev/hugepages-1G/libvirt
    # chown [libvirt-]qemu:kvm -R /dev/hugepages/libvirt

.. note::

    Substitute ``/dev/hugepages[-1G]`` with your actual hugepage mount
    directory. A 2M hugepage mount location is created by default by some
    distributions.

Restart the libvirt daemon::

    # systemctl restart libvirtd

To check that hugepages are correctly reserved for each page size, the
``hugeadm`` utility can be used::

    # hugeadm --pool-list

          Size  Minimum  Current  Maximum  Default
       2097152     2048     2048     2048        *
    1073741824        8        8        8

Binding to vfio-pci
^^^^^^^^^^^^^^^^^^^

Since the VFs need to communicate directly with virtio-forwarder, a
pass-through style driver, such as ``vfio-pci`` is required. The ``vfio-pci``
module is the preferred driver, compared to ``uio_pci_generic`` and
``igb_uio``, of which the former lacks SR-IOV compatibility whereas the latter
is considered outdated.

First, unbind the VF PCI devices from their current drivers::

    # lspci -Dd 19ee:6003 | awk '{print $1}' | xargs -I{} echo \
    "echo {} > /sys/bus/pci/devices/{}/driver/unbind;" | bash

The VFs which now have their drivers unbound, can be observed with the
``lspci`` command::

    # lspci -kd 19ee:
    02:00.0 Ethernet controller: Netronome Systems, Inc. Device 4000
            Subsystem: Netronome Systems, Inc. Device 4001
            Kernel driver in use: nfp
            Kernel modules: nfp
    02:08.0 Ethernet controller: Netronome Systems, Inc. Device 6003
            Subsystem: Netronome Systems, Inc. Device 4001
            Kernel modules: nfp
    02:08.1 Ethernet controller: Netronome Systems, Inc. Device 6003
            Subsystem: Netronome Systems, Inc. Device 4001
            Kernel modules: nfp

Notice that the ``Kernel driver in use`` attribute was removed. To bind the
``vfio-pci`` driver to the VFs, first load the vfio-pci driver to the Linux
kernel::

    # modprobe vfio-pci

Then bind the driver to the VFs::

    # echo 19ee 6003 > /sys/bus/pci/drivers/vfio-pci/new_id

The VFs are now bound to the vfio-pci driver::

    # lspci -kd 19ee:
    02:00.0 Ethernet controller: Netronome Systems, Inc. Device 4000
            Subsystem: Netronome Systems, Inc. Device 4001
            Kernel driver in use: nfp
            Kernel modules: nfp
    02:08.0 Ethernet controller: Netronome Systems, Inc. Device 6003
            Subsystem: Netronome Systems, Inc. Device 4001
            Kernel driver in use: vfio-pci
            Kernel modules: nfp
    02:08.1 Ethernet controller: Netronome Systems, Inc. Device 6003
            Subsystem: Netronome Systems, Inc. Device 4001
            Kernel driver in use: vfio-pci
            Kernel modules: nfp

Launching virtio-forwarder
^^^^^^^^^^^^^^^^^^^^^^^^^^

In this guide, the use case will be virtio-forwarder acting as a server. This
means virtio-forwarder will create and host the sockets to which VMs can
connect at a later stage. To configure virtio-forwarder as the server, edit
``/etc/default/virtioforwarder`` so that ``VIRTIOFWD_VHOST_CLIENT`` is assigned
a blank value::

    # Non-blank enables vhostuser client mode (default: server mode)
    VIRTIOFWD_VHOST_CLIENT=

The virtio-forwarder service can be configured to start during boot time::

    # systemctl enable virtio-forwarder

To manually start the service after installation, run::

    # systemctl start virtio-forwarder

Adding VF ports to virtio-forwarder
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Modify socket permissions::

    # chown -R libvirt-qemu:kvm /tmp/virtio-forwarder/

Dynamically map the PCI address of each VF to virtio-forwarder as follows::

    # /usr/lib/virtio-forwarder/virtioforwarder_port_control.py add \
    --virtio-id 1 --pci-addr 02:08.0
    status: OK
    # /usr/lib/virtio-forwarder/virtioforwarder_port_control.py add \
    --virtio-id 2 --pci-addr 02:08.1
    status: OK

The ``virtio-id`` parameter is compulsory and denotes the id of the relay
through which traffic is routed. A relay can accept only a single PCI device
and a single VM.

The VF ports added to virtio-forwarder can be confirmed with::

    # /usr/lib/virtio-forwarder/virtioforwarder_stats.py \
    --include-inactive | grep DPDK_ADDED
    relay_1.vf_to_vm.internal_state=DPDK_ADDED
    relay_2.vf_to_vm.internal_state=DPDK_ADDED

The VF ports can be removed in a similar fashion::

    # /usr/lib/virtio-forwarder/virtioforwarder_port_control.py remove \
    --virtio-id 1 --pci-addr 02:08.0
    status: OK
    # /usr/lib/virtio-forwarder/virtioforwarder_port_control.py remove \
    --virtio-id 2 --pci-addr 02:08.1
    status: OK

It is useful to watch the virtio-forwarder journal while adding or removing
ports::

    # journalctl -fu virtio-forwarder

The VF entries can also be modified statically within the
``/etc/default/virtioforwarder`` file. Consult the
`virtio-forwarder <virtio-forwarder-site_>`_ docs for more information.

Modify guest VM XML files
^^^^^^^^^^^^^^^^^^^^^^^^^

The snippets in this section should be inserted in each VM's XML file.

The following snippet configures the connection between the VM and the
virtio-forwarder service. Note that ``virtio-forwarder1.sock`` refers to
``virtio-id 1`` and ``relay_1``. The MAC address should be assigned the value
of the specific VF to be paired with the VM. If left unassigned, libvirt will
assign a random MAC address which will cause the VM's traffic to be rejected by
the SmartNIC. The PCI address is internal to the VM and can be chosen
arbitrarily, but should be unique within the VM itself.

.. code-block:: xml
    :linenos:

    <devices>
    <interface type='vhostuser'>
        <mac address='1e:a3:32:f8:3e:83'/>
        <source type='unix' path='/tmp/virtio-forwarder/virtio-forwarder1.sock' mode='client'/>
        <model type='virtio'/>
        <alias name='net1'/>
        <address type='pci' domain='0x0000' bus='0x00' slot='0x06' function='0x0'/>
    </interface>
    </devices>

The VM also has to be configured to make use of the 1G hugepages that was
reserved for this purpose:

.. code-block:: xml
    :linenos:

    <memoryBacking>
    <hugepages>
        <page size='1048576' unit='KiB' nodeset='0'/>
    </hugepages>
    </memoryBacking>

Allocate CPUs and memory to the VM. It is especially important to specify
``memAccess='shared'``, as this allows the host and guest VM to share RAM. This
is required by virtio-forwarder to write the packets to the VM.

.. code-block:: xml
    :linenos:

    <cpu mode='custom' match='exact'>
    <model fallback='allow'>SandyBridge</model>
    <feature policy='require' name='ssse3'/>
    <numa>
        <cell id='0' cpus='0-1' memory='3670016' unit='KiB' memAccess='shared'/>
    </numa>
    </cpu>

The VMs can now be booted. Observing the host's CPU usage (e.g. ``htop``) will
show that some of the cores will be utilized to the maximum (polling
mechanism). The default number of cores dedicated for virtio-forwarder is 2,
and can be adjusted in ``/etc/default/virtioforwarder`` by modifying the
`` VIRTIOFWD_CPU_MASK`` value.
