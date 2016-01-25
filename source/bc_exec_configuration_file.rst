BC-exec.sh Configuration File
=============================

The name convention of the configuration file is typically ``<package-name>_BC<i>.cfg`` where <package-name> is the name of Serviceguard package and <i> is "1" for the primary Business Copy Volume and "2" is the secondary Business Copy Volume.
The preferred location for the configuration file is 
``/opr_<package-name>/BC/<package-name>_BC<i>.cfg``

However, the configuration file can be stored anywhere, but then it is up to the user to keep it in sync between the different cluster nodes and the BCV server.

The configuration file can come in two layout formats:
 * **LAYOUT=1.0** . up to version 1.27 of BC-exec.sh script is using the LAYOUT=1.0 format.
 * **LAYOUT=2.0** . BC-exec.sh version 1.28 and beyond are able to work with both LAYOUT 1.0 and 2.0 formats (one or the other of course). The main difference will be explained below.

Layout 1.0
----------

The configuration file described in **LAYOUT=1.0** format is only containing variable definitions and there can only be *one* volume group defined (with corresponding device group). The configuration file will be sourced by BC-exec.sh script. Therefore, treat the content of the configuration file as a shell script.

An example configuration file might look like::

    # Configuration file used by BC-exec.sh
    # Layout:
    # Option value
    
    # PVOL_INST HORCM instance number that defines the PVOLs
    # SVOL_INST HORCM instance number that defines the SVOLs
    PVOL_INST=0
    SVOL_INST=1
    
    # BC_TIMEOUT is the number of seconds that pairevtwait will wait
    BC_TIMEOUT=300
    
    # DEV_GRP Device group as specified in HORCM_LDEV section
    DEV_GRP=vgdbRPS
    
    # VOL_GRP Name of the volume group on the PVOL-side
    VOL_GRP=/dev/vgdbRPS
    
    # REMOVE_CLUSTER_MODE:
    # set this flag to Y if you're importing the VG that's part of a 
    # cluster outside that cluster
    REMOVE_CLUSTER_MODE=Y
    
    # Exclude mount points from being mounted on the BCV side
    # e.g. EXCLUDE_MOUNTPOINTS="/mnt1 /mnt2"
    EXCLUDE_MOUNTPOINTS="/oracle/RPS /opr_dbciRPS"
    
    # Suspend Sync Flag (is an empty file under the same directory as
    # the #configuration file).
    # Variable is by default empty, if not empty then it contains a 
    # file name
    # Attention: absolute paths are ignored as it should be in the same # directory as the configuration file.
    # If file name is used it is preferred to use "configfile.suspend" 
    # (without the .cfg extention)
    SUSPEND_SYNC=
    

The **PVOL_INST** variable is an integer which refers to the ``/etc/horcm<PVOL_INST>.conf`` file on the P-VOL system. The **SVOL_INST** variable is an integer which refers to the ``/etc/horcm<SVOL_INST>.conf`` file on the BCV side.

The **DEV_GRP** variable is part of the HORCM_LDEV section of horcm configuration files (must be defined on both sides). The **VOL_GRP** variable defines the Volume Group belonging to the DEV_GRP and the P-VOL side.
On the BCV side the Volume Group is normally not required (but it is not prohibited as we import the Volume Group with a unique name  as described in section *BC-exec.sh Software Requirements*).

If the Volume Group is part of a Serviceguard cluster then we should set the variable **REMOVE_CLUSTER_MODE** to "Y".

The variable **EXCLUDE_MOUNTPOINTS** allows us to define file system to be excluded during the mounting on the BCV side. Please note, if the ``/opr_<package-name>`` file system is part of the Serviceguard configuration file we should add this in the **EXCLUDE_MOUNTPOINTS** variable as otherwise on the BCV side we will get an error that it cannot mount ``/opr_<package-name>`` file system because it gets auto-mounted.

There is one additional variable which could be added to the configuration file:

**FORCE_MOUNT_PREFIX=Y** to force a mount prefix name to the mount points on the BCV side. The mount points listed in the file systems file will get mounted under /mnt

Be aware, if for some reason the ``/opr_<package-name>``  directory (mount point) is not available, because the package is down, the BC-exec.sh script will search for a failback configuration file under ``/var/tmp/BC/<package-name>/``

The **BC_TIMEOUT** variable is the time in seconds used by the pairevtwait command will wait before it bails out with an error. Be aware, that the value mentioned in the configuration file will be used for the pair and split operation. However, for the split operation the BC_TIMEOUT value will be multiplied with 12 automatically.

The **SUSPEND_SYNC** variable allows you to prevent re-syncs to happen when set. To prevent re-syncs or splits to happen you could define the following in the configuration file::
    
    SUSPEND_SYNC=<package-name>_BC<i>.suspend
    
Of course you need to touch this file as follow on the NFS (exported) directory::
    
    # touch /opr_<package-name>/BC/<package-name>_BC<i>.suspend

Be careful, you must manual remove the suspend file to release the lock on re-syncs and splits.

Layout 2.0
----------

The main difference with LAYOUT 1.0 format is that LAYOUT 2.0 format allows defining more than one volume group with their corresponding device groups.

An example configuration file in LAYOUT 2.0 format might look like::

    # Configuration file used by BC-exec.sh
    # Layout:
    [LAYOUT]
    2.0
    
    # Option value
    # PVOL_INST HORCM instance number that defines the PVOLs
    [PVOL_INST]
    0
    
    # SVOL_INST HORCM instance number that defines the SVOLs
    [SVOL_INST]
    1
    
    # BC_TIMEOUT is the number of seconds that pairevtwait will wait
    [BC_TIMEOUT]
    300
    
    # DEV_GRP and VOL_GRP have been merged into DEVGRP_VG
    # DEV_GRP Device group as specified in HORCM_LDEV section
    # Per line use: Device-Group  Volume-Group
    [DEVGRP_VG]
    vgplulogs /dev/vgplulogs
    vgpludata /dev/vgpludata
    
    # REMOVE_CLUSTER_MODE:
    # set this flag to Y if you're importing the VG that's part of a
    # cluster outside that cluster
    [REMOVE_CLUSTER_MODE]
    Y
    
    # Exclude mount points from being mounted on the BCV side
    # e.g. [EXCLUDE_MOUNTPOINTS]
    # /mnt1
    # /mnt2
    [EXCLUDE_MOUNTPOINTS]
    
    [SUSPEND_SYNC]
    
The meanings of the variables (enclosed with bracket braces) are the same as with LAYOUT 1.0 format. If a variable has no definition then that means that the variable is empty (see above **EXCLUDE_MOUNTPOINTS** and **SUSPEND_SYNC** settings).

The layout 2.0 is the deferred format for the future.
