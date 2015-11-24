BC-exec.sh Operations
=====================

The Business Copy Operations are defined within the BC-exec.sh script itself and these operations will be integrated as workflows by scheduling (e.g. Tidal) scripts in a later phase. The following operations are known::

 * Validate::
 check if the host system is eligible to run the script. Operating System and version will be checked. All the pre-requisite software will be checked to see if these are available on the system itself. And, of course is this host system able to use Business Copy at all.
Is the configuration file for the script accessible (via automount or direct access)?

 * Resync::
 pair the Business Copy disks (MU#0 or MU#1).

 * Extract::
 save the source input files for the P-VOL disks on the /opr_<SID> path.

 * Split::
 split the paired disks (must be done before the mount workflow).

 * Mount::
 the S-VOL disks (be it MU#0 or MU#1) must be imported and mounted on their mount point. For MU#0 the original mount points will be used, if it presented on another system then the P-VOL system, otherwise, a prefix will be used. For the MU#1 we will mount the disks with a prefix added to the original mount points.

 * Umount::
 the S-VOL disks will be un-mounted on the system and the volume group will be exported.

 * Reversesync::
 The S-VOL disks will be reversed sync.ed onto the P-VOL disks (only use in case of emergency as all data will be lost which was present on the P-VOL disks as these disks will be overwritten with older data from the S-VOL side). Therefore, reversesync is a hidden operation (not shown with the help option).

 * Purgelogs::
 to remove old log files like `*BC-exec*` from the /var/adm/log directory older than <number> of days (default is 30 days; typically when no day argument was given).

The default operation is "validate" when no operation value is specified. Workflows itself are a combination of operations.
