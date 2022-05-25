Release Update History
=============================================

+------------+---------------+------------------------------------+
| Revision   | Date          | Description                        |
+============+===============+====================================+
| V22.04     | April 1, 2022 | Initial release.                   |
+------------+---------------+------------------------------------+ 

---------------------------------------------------------------------------------

1. Overview
-------------------

This document covers the version change instructions for Corigine Agilio CoreNic. 

1.1 Version Mapping
^^^^^^^^^^^^^^^^^^^^^^^^^^^

+-------------+----------------------------------------------+
| OS          | Kernel newer than 4.15 (or RHEL equivalent)  |
+-------------+----------------------------------------------+
| DPDK        | Newer than 17.11                             |
+-------------+----------------------------------------------+
| BSP         | V22.04                                       |
+-------------+----------------------------------------------+

1.2 Standards and Regulations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Agilio SmartNICs adhere to the following regulations.

1.2.1 Environmental Compliance
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

   * European Union RoHS II Directive: 2011/65/EU
   * European Union REACH Directive: 2006/121/EC
   * Administrative Measure on the Control of Pollution Caused by Electronic Information Products ("China ROHS")
   * Congo Conflict Minerals Act of 2009 (Section 1502 of Dodd-Frank Wall Street Reform and Consumer Protection Act including SEC ruling 17 CFR PARTS 240 and 249b)
 
1.2.2 Regulatory Compliance
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

   * CFR 47 FCC Part 15 Subpart B Class A emissions requirements (USA)
   *	European Union EMC Directive: 2004/108/EC
   *	ICES-0003 Issue 4 Class A Digital Apparatus emissions requirements (Canada)
   *	EN 55022:2010/AC:2011 Class A ITE emissions requirements (EU / CE Mark)
   *	EN 55024:2010 ITE - immunity characteristics (EU / CE Mark)
   *	EN 61000-4-2
   *	EN 61000-4-3
   *	EN 61000-4-4
   *	EN 61000-4-6
   *	EN 61000-4-8

2. Support Features
-----------------------------

2.1 New Features
^^^^^^^^^^^^^^^^^^^^^

+------------------------------+-------------------------------------------------------------+
|Feature                       | Description                                                 |
+==============================+=============================================================+
| FW version                   | getting device hardware firmware info                       |
+------------------------------+-------------------------------------------------------------+
| UEFI PXE boot                | Support UEFI BIOS PXE booting                               |
+------------------------------+-------------------------------------------------------------+
| Legacy PXE boot              | Suport Legacy BIOS PXE booting                              |
+------------------------------+-------------------------------------------------------------+
| Firmware flashing            | Ethtool support BSP firmware updating                       |
+------------------------------+-------------------------------------------------------------+
| Extended stats               | Support extended ethtool statistics reporting               |
+------------------------------+-------------------------------------------------------------+
| TSO (LSO)                    | TCP (Large) Segment Offload                                 |
+------------------------------+-------------------------------------------------------------+
| SR-IOV                       | Single-Root Virtual Functions and virtual ethernet bridge   |
+------------------------------+-------------------------------------------------------------+
| SR-IOV MAC VLAN              | VLAN offload from SRIOV MACs                                |
+------------------------------+-------------------------------------------------------------+
| RSS: Receive Side Scaling    | Core/Interrupt/Queue packet routing                         | 
+------------------------------+-------------------------------------------------------------+
| TCP/UDP checksum             | Rx/Tx                                                       |
+------------------------------+-------------------------------------------------------------+
| Jumbo frame support          | MTU setting                                                 | 
+------------------------------+-------------------------------------------------------------+
| eBPF offload                 | eBPF rules on NFP                                           | 
+------------------------------+-------------------------------------------------------------+
| VxLAN                        | Full Encap/Decap                                            |
+------------------------------+-------------------------------------------------------------+
| Inner L3 checksum            | Wildcard flows                                              |
+------------------------------+-------------------------------------------------------------+
| Inner L4 checksum            | Ethtool offloads                                            |
+------------------------------+-------------------------------------------------------------+
| Outer L3 checksum            | Max MTU                                                     |
+------------------------------+-------------------------------------------------------------+
| Support for TSO              | Fallback path for unsupported flows                         |
+------------------------------+-------------------------------------------------------------+
| NVGRE                        | Microsoft, included tenant network id over GRE              |
+------------------------------+-------------------------------------------------------------+
| Adaptive RX/TX               | Change interrupt rates under load (IRQ handling)            |
+------------------------------+-------------------------------------------------------------+
| CX 2x25GbE v2 10G+25G        | Allow 10G+25G in any port-combination on 2-port cards       |
+------------------------------+-------------------------------------------------------------+
| Support CX 2x25GbE v2 board  | Firmware build for the new card type                        |
+------------------------------+-------------------------------------------------------------+

2.2 Modified Features
^^^^^^^^^^^^^^^^^^^^^^^^

None.


2.3 Deleted Features
^^^^^^^^^^^^^^^^^^^^^^^^^^^

None.

3. Resolved Issues
----------------------------

None.

4. Known Issues
-----------------------

None.
