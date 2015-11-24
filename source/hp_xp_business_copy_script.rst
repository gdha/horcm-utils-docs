HP XP Business Copy Script (BC-exec.sh)
=======================================

The HP XP Business Copy Software used is a combination of HP XP Raid Manager Software running on the host computer and a shell script (called BC-exec.sh) which assists the different phases foreseen within our organization to create off host backups.

Pre-requisites for HP XP Business Copy
--------------------------------------

The HP XP Business Copy Software can only be used with the HP XP Storage Arrays such as the P9500.
On the host computer we need also the HP XP Raid Manager Software fully installed and configured.
On the host computer we also need a script, called BC-exec.sh, to assist in the creation of Business 
Copy snapshots or mirrors. The BC-exec.sh script is part of the HP-UX software depot BC_UTILS.


BC-exec.sh Script
-----------------

The purpose of the script BC-exec.sh is to automate the Business Copy processes such as splitting fully mirrored disks, re-sync the disks, mount the volume groups of those disks and make the file systems available on the (backup) server for other purposes such as backup.
The script should be present and able to run on all host systems involved in the Business Copy processes. This is on the P-VOL system itself, and on any system defined to use the S-VOL disks, be it MU#0 or MU#1.
The script is useable on different host computer Operating Systems, such as HP-UX, Linux and Solaris. It is also be backwards compatible within a certain Operating System type, e.g. HP-UX 11.11, HP-UX 11.23 and HP-UX 11.31.
The script is able to handle LVM1 and LVM2 based Volume Groups.
Raw disks (without a volume group) are out of scope within the BC-exec.sh script.

BC-exec.sh Software Pre-requisites
----------------------------------

The script is written in Korn Shell, therefore, it is important that the `ksh` program is available on the host computers. If that is not the case it must be installed before running the BC-exec.sh script.

The BC-exec.sh script also relies on the XPinfo program, therefore, it must be present on all host computers as well. The script is a kind of wrapper around the HP XP Raid Manager Software; therefore, Raid Manager should be installed and configured on all involved host computers.


BC-exec.sh Command Arguments
----------------------------

The BC-exec.sh script has the following usage::

    #-> /opt/jnj/BC/BC-exec.sh -h
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
    2015-11-24 12:56:06 LOG: Exit code 1
    

BC-exec.sh Software Requirements
--------------------------------

The BC-exec.sh is designed to work in different workflows and the script relies on certain input or configuration files which are generated on the source system (where the P-VOLs are residing) and these input files should be made available via NFS export to the system where the S-VOLs are defined on. Therefore, the file system /opr_<package-name> is NFS exported to all target host systems (S-VOLs).
On environments where the /opr_<package-name> is not available, it is also possible to create this directory on the S-VOL system (which is typically not clustered), nfs-exported to the P-VOL systems and then mounted on the P-VOL side. The configuration file location is provided as a parameter for the script: -c </path/configuration_file>. Keep in mind, that the input files must be available on the ./path. directory, if not, the script will return an error.

The SAP teams required the possibility to exclude certain SAP mount points, such as /oracle/<SID>. This should be a variable in the configuration file for BC-exec.sh script. 

Furthermore, the SAP teams like the S-VOL MU#0 disks be mounted on their original mount points (from the P-VOL) for backup reasons (with the original mount points). This makes the recovery on the source host (where P-VOL resides) much easier. However, BC-exec.sh is able to mount it on another path if required (by setting an argument option, such as .-F.). The S-VOL MU#1 (second set of Business Copy disks) will always be mounted with a path prefix, such as /mnt/vgBC<SVOL_INST>_<Device-Group-name>.
The Volume Group created on the BCV server will always use the following syntax: 
/dev/vgBC<SVOL_INST>_<Device-Group-name>.


