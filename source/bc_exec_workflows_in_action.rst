BC-exec.sh Workflows in action
==============================

We are going to show in detail how to use *BC-exec.sh* in practice. We will start on the P-Vol side (the production side, or the node where the Serviceguard package runs on).

BC-exec.sh help
---------------

We have always a built-in help available::

    # ./BC-exec.sh help
    2016-02-03 10:05:15 ERROR: Missing argument "-c configfile"
    Usage: BC-exec.sh [-c /path/configurationfile] [-m mail_destination] [-D log_directory] [-Fdvh]  [Operation]
    
           -c /path/configurationfile
    
           -F : Force a path prefix for MU#0 BCV (MU#1 always uses a prefix)
    
           -m : mail destination (default: )
    
           -D /path_of_log_directory (default: /var/adm/log)
    
           -d : debug mode (default is OFF)
    
           -v : show version and exit
    
           -h : show help (usage) and exit
    
           Operation: supported operations are:
                      validate (default)
                      resync
                      split
                      extract
                      mount
                      umount
                      purgelogs <number of days>
    
         Note that we need at minimum a "-c" option
         ----
    2016-02-03 10:05:15 LOG: Exit code 1
    

BC-exec.sh validate
-------------------

The validate workflow is only meant to inspect the *BC-exec.sh* configuration file and verify we are dealing with a proper RAID manager setup. It can be run on the P-Vol and S-Vol side at any time, and it is aways called with any other operation (except for the help). Furthermore, if we call *BC-exec.sh* without any *operations* parameter, *validate* is the default one (as you can see below)::

    # ./BC-exec.sh -c ./dbciRPS.cfg
    2016-02-03 10:11:11 LOG: BC-exec.sh 1.31
    2016-02-03 10:11:11 LOG: BC-exec.sh -c ./dbciRPS.cfg
    2016-02-03 10:11:11 LOG: PATH=/bin:/usr/bin:/usr/sbin:/sbin:/usr/local/bin:/usr/contrib/bin:/HORCM
    2016-02-03 10:11:11 LOG: CONFIGFILE=./dbciRPS.cfg
    2016-02-03 10:11:11 LOG: OPERATION=validate
    2016-02-03 10:11:11 LOG: MAILUSR=
    2016-02-03 10:11:11 LOG: LOGFILE=/var/adm/log/dbciRPS-validate-BC-exec-20160203-101110-9825.log
    2016-02-03 10:11:11 LOG: Layout of config file ./dbciRPS.cfg is layout version 1.0
    2016-02-03 10:11:11 LOG: WARNING: we prefer a directory name like CONFIGDIR=./BC
    2016-02-03 10:11:11 ERROR: No horcmd daemons running! Please start it manually via horcmstart.sh command
    2016-02-03 10:11:11 ERROR: Exit code 1
    2016-02-03 10:11:11 LOG: Exit code 1
    
The above output clearly mentions that the HORCM daemons are **not** running and therefore, we get a fatal error. We executed this command on the P-Vol side. On the S-Vol side we should do the same to verify if the HORCM daemons are running::

    # /HORCM/usr/bin/horcmstart.sh 5
    starting HORCM inst 5
    HORCM inst 5 starts successfully.

If you wonder why we use *5* for the HORCM instance number? See the HORCM configuration file saved as `/etc/horcm*.conf` and also the BC-exec.sh configuration file mention this next as `PVOL_INST=5` value.

Re-run the `./BC-exec.sh -c ./dbciRPS.cfg` command and now you will see at the end the following::

    2016-02-03 10:25:16 LOG: validate completed successfully.
    2016-02-03 10:25:16 LOG: Exit code 0


BC-exec.sh extract (on P-Vol side)
----------------------------------

The *extract* operation should only be run on the P-Vol side as its main purpose is to collect information about the volume groups and disks belonging to this Business Copy pairs::

    # ./BC-exec.sh -c ./dbciRPS.cfg extract
    2016-02-03 10:26:50 LOG: BC-exec.sh 1.31
    2016-02-03 10:26:50 LOG: BC-exec.sh -c ./dbciRPS.cfg extract
    2016-02-03 10:26:50 LOG: PATH=/bin:/usr/bin:/usr/sbin:/sbin:/usr/local/bin:/usr/contrib/bin:/HORCM
    2016-02-03 10:26:50 LOG: CONFIGFILE=./dbciRPS.cfg
    2016-02-03 10:26:50 LOG: OPERATION=extract
    2016-02-03 10:26:50 LOG: MAILUSR=
    2016-02-03 10:26:50 LOG: LOGFILE=/var/adm/log/dbciRPS-extract-BC-exec-20160203-102650-16733.log
    2016-02-03 10:26:50 LOG: Layout of config file ./dbciRPS.cfg is layout version 1.0
    2016-02-03 10:26:50 LOG: WARNING: we prefer a directory name like CONFIGDIR=./BC
    2016-02-03 10:26:50 LOG: Start extracting source data on gltdbp01
    2016-02-03 10:26:50 LOG: FILESYSTEMS=./vgdbRPS.fs
    2016-02-03 10:26:50 LOG: GRPFILE=./vgdbRPS.grp
    2016-02-03 10:26:50 LOG: Getting Major Minor number: ls -l /dev/vgdbRPS/group
    2016-02-03 10:26:50 LOG: MAPFILE=./vgdbRPS.map
    2016-02-03 10:26:50 LOG: Create mapfile ./vgdbRPS.map for vgdbRPS
    vgexport: Volume group "vgdbRPS" is still active.
    vgexport: Preview of vgexport on volume group "vgdbRPS" succeeded.
    2016-02-03 10:26:50 LOG: Gather filesystem information for VG /dev/vgdbRPS
    2016-02-03 10:26:51 LOG: Making a copy of all files under . to /var/tmp/BC/dbciRPS
    2016-02-03 10:26:51 LOG: extract completed successfully.
    2016-02-03 10:26:51 LOG: Exit code 0

After this run we will get new or updated files (on HP-UX these are)::

    # ls
    dbciRPS.cfg  vgdbRPS.fs   vgdbRPS.grp  vgdbRPS.map

If the current directory is not NFS shared (e.g. via automounting) then manually copy over these to the same location (very important) to the BCV server (or S-Vol side)::

    # scp  * bcv-server:$PWD

Do not forget to re-run the *extract* operation every time you modify the Volume Groups belonging to these Business Copy Groups. And, make sure that the latest files are accessible on the S-Vol side as well.

BC-exec.sh resync (on S-Vol side)
---------------------------------

Resyncing the Business Copy pairs is an essential part in keeping the BC disks in sync. This step is always done before re-splitting the disks to prepare for backup mode::

    # ./BC-exec.sh -c ./dbciRPS.cfg resync
    2016-02-03 10:37:50 LOG: BC-exec.sh 1.31
    2016-02-03 10:37:50 LOG: BC-exec.sh -c ./dbciRPS.cfg resync
    2016-02-03 10:37:50 LOG: PATH=/bin:/usr/bin:/usr/sbin:/sbin:/usr/local/bin:/usr/contrib/bin:/HORCM
    2016-02-03 10:37:50 LOG: CONFIGFILE=./dbciRPS.cfg
    2016-02-03 10:37:50 LOG: OPERATION=resync
    2016-02-03 10:37:50 LOG: MAILUSR=
    2016-02-03 10:37:50 LOG: LOGFILE=/var/adm/log/dbciRPS-resync-BC-exec-20160203-103750-25610.log
    2016-02-03 10:37:50 LOG: Layout of config file ./dbciRPS.cfg is layout version 1.0
    2016-02-03 10:37:50 LOG: WARNING: we prefer a directory name like CONFIGDIR=./BC
    2016-02-03 10:37:50 LOG: Start Pair Resync S-VOL disks on gltbcp01
    2016-02-03 10:37:50 LOG: Check if VG vgBC6_vgdbRPS is inactive.
    vgdisplay: Volume group "/dev/vgBC6_vgdbRPS" does not exist in the "/etc/lvmtab" file.
    vgdisplay: Volume group "/dev/vgBC6_vgdbRPS" does not exist in the "/etc/lvmtab_p" file.
    vgdisplay: Cannot display volume group "vgBC6_vgdbRPS".
    2016-02-03 10:37:50 LOG: Execute: pairdisplay -IBC6 -g vgdbRPS -fcx
    Group   PairVol(L/R) (Port#,TID, LU-M) ,Seq#,LDEV#.P/S,Status,   % ,P-LDEV# M
    vgdbRPS 40:06_40:4b(L) (CL1-A-3, 3,   7-0 )85827  404b.S-VOL PAIR,   99    4006 -
    vgdbRPS 40:06_40:4b(R) (CL1-A-1, 0,   4-0 )85827  4006.P-VOL PAIR,   99    404b -
    vgdbRPS 40:07_40:4c(L) (CL1-A-3, 4,   0-0 )85827  404c.S-VOL PAIR,   99    4007 -
    vgdbRPS 40:07_40:4c(R) (CL1-A-1, 0,   5-0 )85827  4007.P-VOL PAIR,   99    404c -
    vgdbRPS 40:45_40:50(L) (CL1-A-3, 2,   7-0 )85827  4050.S-VOL PAIR,  100    4045 -
    vgdbRPS 40:45_40:50(R) (CL1-A-1, 3,   7-0 )85827  4045.P-VOL PAIR,  100    4050 -
    2016-02-03 10:37:50 LOG: Execute: pairresync -IBC6 -g vgdbRPS
    2016-02-03 10:37:51 LOG: Execute: pairevtwait -IBC6 -g vgdbRPS -t 3600 -s pair -ss pair
    pairevtwait : Wait status done.
    2016-02-03 10:37:54 LOG: Execute: pairdisplay -IBC6 -g vgdbRPS -fcx (should show PAIR)
    Group   PairVol(L/R) (Port#,TID, LU-M) ,Seq#,LDEV#.P/S,Status,   % ,P-LDEV# M
    vgdbRPS 40:06_40:4b(L) (CL1-A-3, 3,   7-0 )85827  404b.S-VOL PAIR,   99    4006 -
    vgdbRPS 40:06_40:4b(R) (CL1-A-1, 0,   4-0 )85827  4006.P-VOL PAIR,   99    404b -
    vgdbRPS 40:07_40:4c(L) (CL1-A-3, 4,   0-0 )85827  404c.S-VOL PAIR,   99    4007 -
    vgdbRPS 40:07_40:4c(R) (CL1-A-1, 0,   5-0 )85827  4007.P-VOL PAIR,   99    404c -
    vgdbRPS 40:45_40:50(L) (CL1-A-3, 2,   7-0 )85827  4050.S-VOL PAIR,  100    4045 -
    vgdbRPS 40:45_40:50(R) (CL1-A-1, 3,   7-0 )85827  4045.P-VOL PAIR,  100    4050 -
    2016-02-03 10:37:54 LOG: resync completed successfully.
    2016-02-03 10:37:54 LOG: Exit code 0

In above output we could see that the BC disks were already paired and therefore, no *resync* was necessary. Otherwise, it would have taken more time to finish the resync operation.

BC-exec.sh split (on S-Vol side)
--------------------------------

We split the BC disks normally after we have put the database in backup mode so that we are sure that the data inside the database is consistent::

    # ./BC-exec.sh -c ./dbciRPS.cfg split
    2016-02-03 10:51:46 LOG: BC-exec.sh 1.31
    2016-02-03 10:51:46 LOG: BC-exec.sh -c ./dbciRPS.cfg split
    2016-02-03 10:51:46 LOG: PATH=/bin:/usr/bin:/usr/sbin:/sbin:/usr/local/bin:/usr/contrib/bin:/HORCM
    2016-02-03 10:51:46 LOG: CONFIGFILE=./dbciRPS.cfg
    2016-02-03 10:51:46 LOG: OPERATION=split
    2016-02-03 10:51:46 LOG: MAILUSR=
    2016-02-03 10:51:46 LOG: LOGFILE=/var/adm/log/dbciRPS-split-BC-exec-20160203-105146-26068.log
    2016-02-03 10:51:46 LOG: Layout of config file ./dbciRPS.cfg is layout version 1.0
    2016-02-03 10:51:46 LOG: WARNING: we prefer a directory name like CONFIGDIR=./BC
    2016-02-03 10:51:46 LOG: Start Splitting S-VOL disks on gltbcp01
    2016-02-03 10:51:46 LOG: Check if VG vgBC6_vgdbRPS is inactive.
    2016-02-03 10:51:47 LOG: VG vgBC6_vgdbRPS is "not" active.
    2016-02-03 10:51:47 LOG: Execute: pairdisplay -IBC6 -g vgdbRPS -fcx
    Group   PairVol(L/R) (Port#,TID, LU-M) ,Seq#,LDEV#.P/S,Status,   % ,P-LDEV# M
    vgdbRPS 40:06_40:4b(L) (CL1-A-3, 3,   7-0 )85827  404b.S-VOL PAIR,   99    4006 -
    vgdbRPS 40:06_40:4b(R) (CL1-A-1, 0,   4-0 )85827  4006.P-VOL PAIR,   99    404b -
    vgdbRPS 40:07_40:4c(L) (CL1-A-3, 4,   0-0 )85827  404c.S-VOL PAIR,   99    4007 -
    vgdbRPS 40:07_40:4c(R) (CL1-A-1, 0,   5-0 )85827  4007.P-VOL PAIR,   99    404c -
    vgdbRPS 40:45_40:50(L) (CL1-A-3, 2,   7-0 )85827  4050.S-VOL PAIR,  100    4045 -
    vgdbRPS 40:45_40:50(R) (CL1-A-1, 3,   7-0 )85827  4045.P-VOL PAIR,  100    4050 -
    2016-02-03 10:51:47 LOG: Execute: pairsplit -IBC6 -g vgdbRPS
    2016-02-03 10:51:47 LOG: Execute: pairevtwait -IBC6 -g vgdbRPS -t 300 -s psus -ss ssus
    pairevtwait : Wait status done.
    2016-02-03 10:51:53 LOG: Execute: pairdisplay -IBC6 -g vgdbRPS -fcx
    Group   PairVol(L/R) (Port#,TID, LU-M) ,Seq#,LDEV#.P/S,Status,   % ,P-LDEV# M
    vgdbRPS 40:06_40:4b(L) (CL1-A-3, 3,   7-0 )85827  404b.S-VOL SSUS,  100    4006 -
    vgdbRPS 40:06_40:4b(R) (CL1-A-1, 0,   4-0 )85827  4006.P-VOL PSUS,  100    404b W
    vgdbRPS 40:07_40:4c(L) (CL1-A-3, 4,   0-0 )85827  404c.S-VOL SSUS,  100    4007 -
    vgdbRPS 40:07_40:4c(R) (CL1-A-1, 0,   5-0 )85827  4007.P-VOL PSUS,  100    404c W
    vgdbRPS 40:45_40:50(L) (CL1-A-3, 2,   7-0 )85827  4050.S-VOL SSUS,  100    4045 -
    vgdbRPS 40:45_40:50(R) (CL1-A-1, 3,   7-0 )85827  4045.P-VOL PSUS,  100    4050 W
    2016-02-03 10:51:54 LOG: split completed successfully.
    2016-02-03 10:51:54 LOG: Exit code 0

Once the split was successfully executed we can bring the database back out of backup mode to avoid too many redo log files are created and therefore, filling up the redo log directory.

BC-exec.sh mount (on S-Vol side)
--------------------------------

The purpose on the BCV server is to create a backup residing on the S-Vol disks without interrupting the production data (on the P-Vol disks). The backup can run as long as necessary to fullfill its job. However, before starting the backup we should mount the file systems::

    # ./BC-exec.sh -c ./dbciRPS.cfg mount
    2016-02-03 10:58:44 LOG: BC-exec.sh 1.31
    2016-02-03 10:58:44 LOG: BC-exec.sh -c ./dbciRPS.cfg mount
    2016-02-03 10:58:44 LOG: PATH=/bin:/usr/bin:/usr/sbin:/sbin:/usr/local/bin:/usr/contrib/bin:/HORCM
    2016-02-03 10:58:44 LOG: CONFIGFILE=./dbciRPS.cfg
    2016-02-03 10:58:44 LOG: OPERATION=mount
    2016-02-03 10:58:44 LOG: MAILUSR=
    2016-02-03 10:58:44 LOG: LOGFILE=/var/adm/log/dbciRPS-mount-BC-exec-20160203-105844-26277.log
    2016-02-03 10:58:44 LOG: Layout of config file ./dbciRPS.cfg is layout version 1.0
    2016-02-03 10:58:44 LOG: WARNING: we prefer a directory name like CONFIGDIR=./BC
    2016-02-03 10:58:44 LOG: Start mounting S-VOL disks on gltbcp01
    2016-02-03 10:58:44 LOG: MAPFILE=./vgdbRPS.map
    2016-02-03 10:58:44 LOG: Check if we have a map file for VG vgBC6_vgdbRPS.
    2016-02-03 10:58:44 LOG: GRPFILE=./vgdbRPS.grp
    2016-02-03 10:58:44 LOG: Check if we have a group file for VG vgBC6_vgdbRPS.
    2016-02-03 10:58:44 LOG: Check if VG vgBC6_vgdbRPS is inactive.
    2016-02-03 10:58:44 LOG: VG vgBC6_vgdbRPS is "not" active.
    2016-02-03 10:58:44 LOG: mkdir -p -m 755 /dev/vgBC6_vgdbRPS
    2016-02-03 10:58:44 LOG: Check if our PID (26277) is locked
    2016-02-03 10:58:44 LOG: lock succeeded: 26277 - /tmp/BC-exec-LOCKDIR/BC-exec-PIDFILE
    2016-02-03 10:58:45 LOG: Create the /dev/vgBC6_vgdbRPS/group file
    2016-02-03 10:58:45 LOG: Successfully removed the lock directory (/tmp/BC-exec-LOCKDIR)
    2016-02-03 10:58:45 LOG: Major, minor VG nrs are 128 0x006000 /dev/vgBC6_vgdbRPS/group
    2016-02-03 10:58:45 LOG: Change the VG id on /dev/vgBC6_vgdbRPS
    2016-02-03 10:58:46 LOG: Import vgBC6_vgdbRPS via mapfile ./vgdbRPS.map
    vgimport: Beginning the import process on Volume Group "vgBC6_vgdbRPS".
    Logical volume "/dev/vgBC6_vgdbRPS/lvmntRPS" has been successfully created
    with minor number 1.
    Logical volume "/dev/vgBC6_vgdbRPS/lvtransRPS" has been successfully created
    with minor number 2.
    Logical volume "/dev/vgBC6_vgdbRPS/lvascsRPS" has been successfully created
    with minor number 4.
    Logical volume "/dev/vgBC6_vgdbRPS/lvoracleRPS" has been successfully created
    with minor number 5.
    Logical volume "/dev/vgBC6_vgdbRPS/lvoriglogARPS" has been successfully created
    with minor number 6.
    Logical volume "/dev/vgBC6_vgdbRPS/lvoriglogBRPS" has been successfully created
    with minor number 7.
    Logical volume "/dev/vgBC6_vgdbRPS/lvmirrlogARPS" has been successfully created
    with minor number 8.
    Logical volume "/dev/vgBC6_vgdbRPS/lvmirrlogBRPS" has been successfully created
    with minor number 9.
    Logical volume "/dev/vgBC6_vgdbRPS/lvoraarcRPS" has been successfully created
    with minor number 10.
    Logical volume "/dev/vgBC6_vgdbRPS/lvsapreorgRPS" has been successfully created
    with minor number 11.
    Logical volume "/dev/vgBC6_vgdbRPS/lvsapdata1RPS" has been successfully created
    with minor number 12.
    Logical volume "/dev/vgBC6_vgdbRPS/lvoprRPS" has been successfully created
    with minor number 13.
    Logical volume "/dev/vgBC6_vgdbRPS/lvtidal" has been successfully created
    with minor number 3.
    Volume group "/dev/vgBC6_vgdbRPS" has been successfully created.
    Warning: A backup of this volume group may not exist on this machine.
    Please remember to take a backup using the vgcfgbackup command after activating the volume group.
    2016-02-03 10:58:46 LOG: vgchange -c n if REMOVE_CLUSTERMODE(Y) = Y
    Configuration change completed.
    Volume group "vgBC6_vgdbRPS" has been successfully changed.
    2016-02-03 10:58:46 LOG: Activating VG vgBC6_vgdbRPS.
    Activated volume group.
    Volume group "vgBC6_vgdbRPS" has been successfully changed.
    2016-02-03 10:58:46 LOG: Using existing mount point /export/sapmnt/RPS.
    2016-02-03 10:58:46 LOG: Running fsck on /dev/vgBC6_vgdbRPS/lvmntRPS
    log replay in progress
    replay complete - marking super-block as CLEAN
    2016-02-03 10:58:47 LOG: Mounting -F vxfs -o ioerror=mwdisable,largefiles,delaylog,nodatainlog /dev/vgBC6_vgdbRPS/lvmntRPS /export/sapmnt/RPS
    2016-02-03 10:58:48 LOG: Using existing mount point /export/usr/sap/transRPSPRD.
    2016-02-03 10:58:48 LOG: Running fsck on /dev/vgBC6_vgdbRPS/lvtransRPS
    log replay in progress
    replay complete - marking super-block as CLEAN
    2016-02-03 10:58:48 LOG: Mounting -F vxfs -o ioerror=mwdisable,largefiles,delaylog,nodatainlog /dev/vgBC6_vgdbRPS/lvtransRPS /export/usr/sap/transRPSPRD
    2016-02-03 10:58:48 LOG: Using existing mount point /usr/sap/RPS/ASCS05.
    2016-02-03 10:58:48 LOG: Running fsck on /dev/vgBC6_vgdbRPS/lvascsRPS
    log replay in progress
    replay complete - marking super-block as CLEAN
    2016-02-03 10:58:48 LOG: Mounting -F vxfs -o ioerror=mwdisable,largefiles,delaylog,nodatainlog /dev/vgBC6_vgdbRPS/lvascsRPS /usr/sap/RPS/ASCS05
    2016-02-03 10:58:48 LOG: Using existing mount point /opr_dbciRPS.
    2016-02-03 10:58:48 LOG: Running fsck on /dev/vgBC6_vgdbRPS/lvoprRPS
    log replay in progress
    replay complete - marking super-block as CLEAN
    2016-02-03 10:58:49 LOG: Mounting -F vxfs -o ioerror=mwdisable,largefiles,delaylog,nodatainlog /dev/vgBC6_vgdbRPS/lvoprRPS /opr_dbciRPS
    2016-02-03 10:58:49 LOG: Using existing mount point /opt/TIDAL/dbciRPS.
    2016-02-03 10:58:49 LOG: Running fsck on /dev/vgBC6_vgdbRPS/lvtidal
    log replay in progress
    replay complete - marking super-block as CLEAN
    2016-02-03 10:58:49 LOG: Mounting -F vxfs -o ioerror=mwdisable,largefiles,delaylog,nodatainlog /dev/vgBC6_vgdbRPS/lvtidal /opt/TIDAL/dbciRPS
    2016-02-03 10:58:49 LOG: Using existing mount point /oracle/RPS/origlogA.
    2016-02-03 10:58:49 LOG: Running fsck on /dev/vgBC6_vgdbRPS/lvoriglogARPS
    log replay in progress
    replay complete - marking super-block as CLEAN
    2016-02-03 10:58:49 LOG: Mounting -F vxfs -o ioerror=mwdisable,largefiles,mincache=direct,delaylog,nodatainlog,convosync=direct /dev/vgBC6_vgdbRPS/lvoriglogARPS /oracle/RPS/origlogA
    2016-02-03 10:58:49 LOG: Using existing mount point /oracle/RPS/origlogB.
    2016-02-03 10:58:49 LOG: Running fsck on /dev/vgBC6_vgdbRPS/lvoriglogBRPS
    log replay in progress
    replay complete - marking super-block as CLEAN
    2016-02-03 10:58:50 LOG: Mounting -F vxfs -o ioerror=mwdisable,largefiles,mincache=direct,delaylog,nodatainlog,convosync=direct /dev/vgBC6_vgdbRPS/lvoriglogBRPS /oracle/RPS/origlogB
    2016-02-03 10:58:50 LOG: Using existing mount point /oracle/RPS/mirrlogA.
    2016-02-03 10:58:50 LOG: Running fsck on /dev/vgBC6_vgdbRPS/lvmirrlogARPS
    log replay in progress
    replay complete - marking super-block as CLEAN
    2016-02-03 10:58:50 LOG: Mounting -F vxfs -o ioerror=mwdisable,largefiles,mincache=direct,delaylog,nodatainlog,convosync=direct /dev/vgBC6_vgdbRPS/lvmirrlogARPS /oracle/RPS/mirrlogA
    2016-02-03 10:58:50 LOG: Using existing mount point /oracle/RPS/mirrlogB.
    2016-02-03 10:58:50 LOG: Running fsck on /dev/vgBC6_vgdbRPS/lvmirrlogBRPS
    log replay in progress
    replay complete - marking super-block as CLEAN
    2016-02-03 10:58:50 LOG: Mounting -F vxfs -o ioerror=mwdisable,largefiles,mincache=direct,delaylog,nodatainlog,convosync=direct /dev/vgBC6_vgdbRPS/lvmirrlogBRPS /oracle/RPS/mirrlogB
    2016-02-03 10:58:50 LOG: Using existing mount point /oracle/RPS/oraarch.
    2016-02-03 10:58:50 LOG: Running fsck on /dev/vgBC6_vgdbRPS/lvoraarcRPS
    log replay in progress
    replay complete - marking super-block as CLEAN
    2016-02-03 10:58:50 LOG: Mounting -F vxfs -o ioerror=mwdisable,largefiles,mincache=direct,delaylog,nodatainlog,convosync=direct /dev/vgBC6_vgdbRPS/lvoraarcRPS /oracle/RPS/oraarch
    2016-02-03 10:58:50 LOG: Using existing mount point /oracle/RPS/sapdata1.
    2016-02-03 10:58:50 LOG: Running fsck on /dev/vgBC6_vgdbRPS/lvsapdata1RPS
    log replay in progress
    replay complete - marking super-block as CLEAN
    2016-02-03 10:58:51 LOG: Mounting -F vxfs -o ioerror=mwdisable,largefiles,mincache=direct,delaylog,nodatainlog,convosync=direct /dev/vgBC6_vgdbRPS/lvsapdata1RPS /oracle/RPS/sapdata1
    2016-02-03 10:58:51 LOG: Using existing mount point /oracle/RPS/sapreorg.
    2016-02-03 10:58:51 LOG: Running fsck on /dev/vgBC6_vgdbRPS/lvsapreorgRPS
    log replay in progress
    replay complete - marking super-block as CLEAN
    2016-02-03 10:58:51 LOG: Mounting -F vxfs -o ioerror=mwdisable,largefiles,mincache=direct,delaylog,nodatainlog,convosync=direct /dev/vgBC6_vgdbRPS/lvsapreorgRPS /oracle/RPS/sapreorg
    2016-02-03 10:58:51 LOG: Making a copy of all files under . to /var/tmp/BC/dbciRPS
    2016-02-03 10:58:51 LOG: mount completed successfully.
    2016-02-03 10:58:51 LOG: Exit code 0

You should be able to see the mounted file systems::

    #-> mount | grep BC6
    /export/sapmnt/RPS on /dev/vgBC6_vgdbRPS/lvmntRPS ioerror=mwdisable,largefiles,delaylog,nodatainlog,dev=80006001 on Wed Feb  3 10:58:48 2016
    /export/usr/sap/transRPSPRD on /dev/vgBC6_vgdbRPS/lvtransRPS ioerror=mwdisable,largefiles,delaylog,nodatainlog,dev=80006002 on Wed Feb  3 10:58:48 2016
    /usr/sap/RPS/ASCS05 on /dev/vgBC6_vgdbRPS/lvascsRPS ioerror=mwdisable,largefiles,delaylog,nodatainlog,dev=80006004 on Wed Feb  3 10:58:48 2016
    /opt/TIDAL/dbciRPS on /dev/vgBC6_vgdbRPS/lvtidal ioerror=mwdisable,largefiles,delaylog,nodatainlog,dev=80006003 on Wed Feb  3 10:58:49 2016
    /oracle/RPS/origlogA on /dev/vgBC6_vgdbRPS/lvoriglogARPS ioerror=mwdisable,largefiles,mincache=direct,delaylog,nodatainlog,convosync=direct,dev=80006006 on Wed Feb  3 10:58:49 2016
    /oracle/RPS/origlogB on /dev/vgBC6_vgdbRPS/lvoriglogBRPS ioerror=mwdisable,largefiles,mincache=direct,delaylog,nodatainlog,convosync=direct,dev=80006007 on Wed Feb  3 10:58:50 2016
    /oracle/RPS/mirrlogA on /dev/vgBC6_vgdbRPS/lvmirrlogARPS ioerror=mwdisable,largefiles,mincache=direct,delaylog,nodatainlog,convosync=direct,dev=80006008 on Wed Feb  3 10:58:50 2016
    /oracle/RPS/mirrlogB on /dev/vgBC6_vgdbRPS/lvmirrlogBRPS ioerror=mwdisable,largefiles,mincache=direct,delaylog,nodatainlog,convosync=direct,dev=80006009 on Wed Feb  3 10:58:50 2016
    /oracle/RPS/oraarch on /dev/vgBC6_vgdbRPS/lvoraarcRPS ioerror=mwdisable,largefiles,mincache=direct,delaylog,nodatainlog,convosync=direct,dev=8000600a on Wed Feb  3 10:58:50 2016
    /oracle/RPS/sapdata1 on /dev/vgBC6_vgdbRPS/lvsapdata1RPS ioerror=mwdisable,largefiles,mincache=direct,delaylog,nodatainlog,convosync=direct,dev=8000600c on Wed Feb  3 10:58:51 2016
    /oracle/RPS/sapreorg on /dev/vgBC6_vgdbRPS/lvsapreorgRPS ioerror=mwdisable,largefiles,mincache=direct,delaylog,nodatainlog,convosync=direct,dev=8000600b on Wed Feb  3 10:58:51 2016

BC-exec.sh umount (on S-Vol side)
---------------------------------

After the backup has been finished there is no need to keep the file systems mounted. And, before we run a *resync* operation we must be sure that all file systems are un-mounted and the Volume Groups are exported. We can do this in one go with this **umount** operation as you see below::

    # ./BC-exec.sh -c ./dbciRPS.cfg umount
    2016-02-03 12:04:26 LOG: BC-exec.sh 1.31
    2016-02-03 12:04:26 LOG: BC-exec.sh -c ./dbciRPS.cfg umount
    2016-02-03 12:04:26 LOG: PATH=/bin:/usr/bin:/usr/sbin:/sbin:/usr/local/bin:/usr/contrib/bin:/HORCM
    2016-02-03 12:04:26 LOG: CONFIGFILE=./dbciRPS.cfg
    2016-02-03 12:04:26 LOG: OPERATION=umount
    2016-02-03 12:04:26 LOG: MAILUSR=
    2016-02-03 12:04:26 LOG: LOGFILE=/var/adm/log/dbciRPS-umount-BC-exec-20160203-120426-28254.log
    2016-02-03 12:04:26 LOG: Layout of config file ./dbciRPS.cfg is layout version 1.0
    2016-02-03 12:04:26 LOG: WARNING: we prefer a directory name like CONFIGDIR=./BC
    2016-02-03 12:04:26 LOG: Check if VG vgBC6_vgdbRPS is active.
    2016-02-03 12:04:26 LOG: VG vgBC6_vgdbRPS is active.
    2016-02-03 12:04:26 LOG: Umount file system /export/sapmnt/RPS
    2016-02-03 12:04:26 LOG: Umount file system /export/usr/sap/transRPSPRD
    2016-02-03 12:04:26 LOG: Umount file system /usr/sap/RPS/ASCS05
    2016-02-03 12:04:26 LOG: Umount file system /opr_dbciRPS
    2016-02-03 12:04:26 LOG: Umount file system /opt/TIDAL/dbciRPS
    2016-02-03 12:04:26 LOG: Umount file system /oracle/RPS/origlogA
    2016-02-03 12:04:27 LOG: Umount file system /oracle/RPS/origlogB
    2016-02-03 12:04:27 LOG: Umount file system /oracle/RPS/mirrlogA
    2016-02-03 12:04:27 LOG: Umount file system /oracle/RPS/mirrlogB
    2016-02-03 12:04:27 LOG: Umount file system /oracle/RPS/oraarch
    2016-02-03 12:04:27 LOG: Umount file system /oracle/RPS/sapdata1
    2016-02-03 12:04:27 LOG: Umount file system /oracle/RPS/sapreorg
    2016-02-03 12:04:27 LOG: De-activating VG vgBC6_vgdbRPS
    Volume group "vgBC6_vgdbRPS" has been successfully changed.
    2016-02-03 12:04:27 LOG: Export the VG vgBC6_vgdbRPS
    Beginning the export process on Volume Group "vgBC6_vgdbRPS".
    /dev/disk/disk38
    /dev/disk/disk40
    /dev/disk/disk29
    vgexport: Volume group "vgBC6_vgdbRPS" has been successfully removed.
    2016-02-03 12:04:27 LOG: umount completed successfully.
    2016-02-03 12:04:27 LOG: Exit code 0

After this the cyclus can restart with a new **resync** operation and so on.
