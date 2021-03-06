Scheduling Jobs to use BC-exec.sh
=================================

We have discussed the general use of the tool *BC-exec.sh*, but the real force lies in the use of scheduling jobs. What do we mean by this? In general way a database need to get backed up, and therefore, we should freeze the database during the backup which could take too much time. Using business copy is a practice that is in use for a long time to overcome these kind of delays.

BC-exec.sh was exactly designed with this in mind, that is using it with business copy devices and not doing anything with the main production data. In big lines we must do the following:

 * **extract**: On the P-Vol side (where the real production data is residing on) extracting the information about the Volume Group that we want to back-up. This will *not* interfere with any daily operational procedure and is non-disruptive. You only need to run this once, and afterwards, when the Volume Group has changed (e.g. adding disks, logical volumes and so on) again.

 * **resync**: we sync the S-Vol disks (on the BCV server) with the P-Vol disks to make sure the disks are exactly the same (paired). This operation waits until the pairs are 100% complete, or when the session times out (the *BC_TIMEOUT* variable in seconds multiplied with 12)

 * **Put database in backup mode**:  before splitting the paired disks we must put the database in backup mode (on the P-Vol side). During this time the redo logs will be used to keep up with the transactions (on the P-Vol side).

 * **split**: once the disks are 100% in pair we should split these immediately before going on (on the BCV server)

 * **Bring database out of backup mode**: when the disks are splitted, then we can bring the database out of backup mode and replay all the redo logs (on the P-Vol side).

 * **mount**: mount the the S-Vol disks (Volume Group) on their original mount points (or with a prefix) on the BCV server.

 * **start the backup process**: now we can backup all the data mounted with our backup software which we normally use (on the BCV server). Time is of no essence as we do not interfere with the production database running on the P-Vol side.

 * **umount**: when we are done with the backup we can un-mount and destroy the Volume Group. Keeping the S-Vol disks intact until the next *resync* round. In case of emergency we could always execute a reverse sync (which resync the S-Vol disks to the P-Vol disks).
