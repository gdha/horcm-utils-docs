CheckHorcmConsistency.sh Script
===============================
In the package we deliver also another script called CheckHorcmConsistency.sh, which has a purpose to investigate the HORCM configuration file. We noticed that adding new disk configuration is not always done in a consistent manner. To bring these bad configuration to our attention we wrote this script.

It can be run on the P-Vol and/or S-Vol side.

Example output of CheckHorcmConsistency.sh (P-Vol side)
-------------------------------------------------------

The script CheckHorcmConsistency.sh output without arguments is quite short::

    #-> /usr/local/sbin/CheckHorcmConsistency.sh
    2016-02-29 07:48:41 LOG:  === Horcm daemon active with instance nummer 0 ===
    2016-02-29 07:49:24 LOG: Pairdisplay of device group vg_dataxp2 contains the same amount of disks as VG vg_dataxp2  [OK]
    2016-02-29 07:49:25 LOG: Pairdisplay of device group vg_mlogxp2 contains the same amount of disks as VG vg_mlogxp2  [OK]
    2016-02-29 07:49:26 LOG: Pairdisplay of device group vg_ologxp2 contains the same amount of disks as VG vg_ologxp2  [OK]
    2016-02-29 07:49:26 LOG: Error count: 0
    See logfiles: /var/adm/log/CheckHorcmConsistency.log and /var/tmp/CheckHorcmConsistency-20160229-074839-44471.log

You can also ask for a more verbose output with the -v option::

    #-> /usr/local/sbin/CheckHorcmConsistency.sh -v
    2016-03-22 08:09:35 VERBOSE: CheckHorcmConsistency.sh revision 1.2
    2016-03-22 08:09:35 VERBOSE: Started as: /opt/jnj/BC/CheckHorcmConsistency.sh -v
    2016-03-22 08:09:35 VERBOSE: LOGFILE=/var/adm/log/CheckHorcmConsistency.log
    2016-03-22 08:09:35 VERBOSE: tmpLOGFILE=/var/tmp/CheckHorcmConsistency-20160322-080935-44706.log
    2016-03-22 08:09:35 VERBOSE: Raid Manager version is 01.29.05
    2016-03-22 08:09:35 VERBOSE: Creating temporary directory /tmp/CheckHorcmConsistency_11185
    2016-03-22 08:09:35 VERBOSE: Capturing the Volume groups with their devices
    2016-03-22 08:09:37 LOG:  === Horcm daemon active with instance nummer 0 ===
    2016-03-22 08:09:37 VERBOSE: Found /etc/horcm0.conf - analyzing...
    2016-03-22 08:09:37 VERBOSE: Capturing the disks with corresponding cu_ldev number for instance number 0
    2016-03-22 08:09:37 VERBOSE: Busy Processing using raidscan...
    2016-03-22 08:09:45 VERBOSE: *** Inspect device group vg_dataxp2 defined with HORCM instance 0 ***
    2016-03-22 08:09:46 VERBOSE: Device group vg_dataxp2 with INST (0) is defined as BC (status: SPLIT)
    2016-03-22 08:09:46 VERBOSE: Save the cu:ldev numbers of the disks into culdev.vg_dataxp2
    2016-03-22 08:09:46 VERBOSE: Find the according culdev of /dev/mapper/devices in lvmtab.out
    2016-03-22 08:10:22 VERBOSE: Created file lvmtab.culdev which maps culdev to volume groups
    2016-03-22 08:10:22 VERBOSE: Find the according culdev to device group vg_dataxp2
    2016-03-22 08:10:22 VERBOSE: Created file lvmtab.BC0.vg_dataxp2 which maps culdev to volume group of device group vg_dataxp2
    2016-03-22 08:10:22 VERBOSE: Compare the devices in Device Group vg_dataxp2 with the corresponding Volume Group vg_dataxp2
    2016-03-22 08:10:22 LOG: Pairdisplay of device group vg_dataxp2 contains the same amount of disks as VG vg_dataxp2  [OK]
    2016-03-22 08:10:22 VERBOSE: *** Inspect device group vg_mlogxp2 defined with HORCM instance 0 ***
    2016-03-22 08:10:23 VERBOSE: Device group vg_mlogxp2 with INST (0) is defined as BC (status: SPLIT)
    2016-03-22 08:10:23 VERBOSE: Save the cu:ldev numbers of the disks into culdev.vg_mlogxp2
    2016-03-22 08:10:23 VERBOSE: Find the according culdev to device group vg_mlogxp2
    2016-03-22 08:10:23 VERBOSE: Created file lvmtab.BC0.vg_mlogxp2 which maps culdev to volume group of device group vg_mlogxp2
    2016-03-22 08:10:23 VERBOSE: Compare the devices in Device Group vg_mlogxp2 with the corresponding Volume Group vg_mlogxp2
    2016-03-22 08:10:23 LOG: Pairdisplay of device group vg_mlogxp2 contains the same amount of disks as VG vg_mlogxp2  [OK]
    2016-03-22 08:10:23 VERBOSE: *** Inspect device group vg_ologxp2 defined with HORCM instance 0 ***
    2016-03-22 08:10:23 VERBOSE: Device group vg_ologxp2 with INST (0) is defined as BC (status: SPLIT)
    2016-03-22 08:10:23 VERBOSE: Save the cu:ldev numbers of the disks into culdev.vg_ologxp2
    2016-03-22 08:10:24 VERBOSE: Find the according culdev to device group vg_ologxp2
    2016-03-22 08:10:24 VERBOSE: Created file lvmtab.BC0.vg_ologxp2 which maps culdev to volume group of device group vg_ologxp2
    2016-03-22 08:10:24 VERBOSE: Compare the devices in Device Group vg_ologxp2 with the corresponding Volume Group vg_ologxp2
    2016-03-22 08:10:24 LOG: Pairdisplay of device group vg_ologxp2 contains the same amount of disks as VG vg_ologxp2  [OK]
    2016-03-22 08:10:24 VERBOSE: Remove all temporary files [executed: rm -rf /tmp/CheckHorcmConsistency_11185]
    2016-03-22 08:10:24 LOG: Error count: 0
    See logfiles: /var/adm/log/CheckHorcmConsistency.log and /var/tmp/CheckHorcmConsistency-20160322-080935-44706.log

Example output of CheckHorcmConsistency.sh (S-Vol side)
-------------------------------------------------------

We can run this script also on the BCV server (S-Vol side)::

    #-> /usr/local/sbin/CheckHorcmConsistency.sh
    2016-02-29 07:47:22 LOG:  === Horcm daemon active with instance nummer 3 ===
    2016-02-29 07:48:06 LOG: Pairdisplay of device group vg_datagpz contains the same amount of disks as VG vgBC3_vg_datagpz  [OK]
    2016-02-29 07:48:06 LOG: Pairdisplay of device group vg_mloggpz contains the same amount of disks as VG vgBC3_vg_mloggpz  [OK]
    2016-02-29 07:48:07 LOG: Pairdisplay of device group vg_ologgpz contains the same amount of disks as VG vgBC3_vg_ologgpz  [OK]
    2016-02-29 07:48:07 LOG:  === Horcm daemon active with instance nummer 1 ===
    2016-02-29 07:48:13 LOG: Pairdisplay of device group vg_dataxp2 contains the same amount of disks as VG vgBC1_vg_dataxp2  [OK]
    2016-02-29 07:48:14 LOG: Pairdisplay of device group vg_mlogxp2 contains the same amount of disks as VG vgBC1_vg_mlogxp2  [OK]
    2016-02-29 07:48:15 LOG: Pairdisplay of device group vg_ologxp2 contains the same amount of disks as VG vgBC1_vg_ologxp2  [OK]
    2016-02-29 07:48:15 LOG: Error count: 0
    See logfiles: /var/adm/log/CheckHorcmConsistency.log and /var/tmp/CheckHorcmConsistency-20160229-074719-55395.log

Whenever, an issue is noticed you will see a warning so you can investigate deeper to resolve the HORCM configuration on both sides (P-Vol and S-Vol side).
