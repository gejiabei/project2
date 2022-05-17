Release Notes
=============

The latest version of the Basic Nic Firmware is 2.1.16, released
2018/11/04.

.. note::

    For optimal performance with the Basic Nic Firmware, please ensure to use a
    kernel newer than 4.15 (or RHEL equivalent); or the kmods driver.  See
    :ref:`0B_Install_oot_nfp_driver:Appendix B: Installing the Out-of-Tree NFP
    Driver`

.. note::

    Rapidly loading and unloading the kernel driver module can result in the
    card becoming unresponsive.  Such a situation requires a host reboot
    to resolve.

.. note::

    In this firmware release, tunnel offloading capabilities are disabled on VF
    ports.

Release History
===============

2.1.16
------

- New SRIOV capable firmware file

  - Support for 48 single-queue VFs on physical port 0

- Tunnel inner header RSS support for VXLAN and Geneve

- Improved performance

2.0.7
-----

- TCP (Large) Segment Offload (TSO / LSO)

- TCP and UDP Receive Side Scaling (RSS) with CRC32 algorithm

- Receive and Transmit Checksum Offload

- Jumbo Frame Support up to 9216B Frames
