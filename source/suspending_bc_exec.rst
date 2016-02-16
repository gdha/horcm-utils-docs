Suspending BC-exec.sh
=====================

There are several reasons you can think of to overrule the BC-exec.sh workflows which are normally triggered via a scheduling system. For example, for doing upgrade tests of your database on the BCV server. Afterwards, you could resync the original data and try it again as many times as you wish.

In the configuration part we already mentioned that there is a variable called `SUSPEND_SYNC` which can be set in the configuration file that belongs to your workflow. The name we use for the *suspend flag* is normally *<configuration-file-with-extention>.suspend*, e.g.::

    SUSPEND_SYNC=vgdbRPS.suspend

Of course, if this file is not found under the same directory as the configuration file it will have *no* effect on the workflows of BC-exec.sh. Therefore, it is safe, to configure this always in the configuration file.

Set the suspend flag
--------------------

You need two fullfill two items:
 * make sure the `SUSPEND_SYNC` variable has been defined in the configuration file
 * `touch` the suspend file under the *same* directory as the configuration file on the BCV server (S-Vol side)


Suspending the Resync
---------------------

When the suspend flag has been defined on the BCV server (S-Vol side) then it will not be possible anymore to resync the disks until we clear the suspend flag again::

    # ./BC-exec.sh -c ./dbciRPS.cfg resync
    2016-02-16 09:36:05 LOG: BC-exec.sh 1.31
    2016-02-16 09:36:05 LOG: BC-exec.sh -c ./dbciRPS.cfg resync
    2016-02-16 09:36:05 LOG: PATH=/bin:/usr/bin:/usr/sbin:/sbin:/usr/local/bin:/usr/contrib/bin:/HORCM
    2016-02-16 09:36:05 LOG: CONFIGFILE=./dbciRPS.cfg
    2016-02-16 09:36:05 LOG: OPERATION=resync
    2016-02-16 09:36:05 LOG: MAILUSR=
    2016-02-16 09:36:05 LOG: LOGFILE=/var/adm/log/dbciRPS-resync-BC-exec-20160216-093605-6135.log
    2016-02-16 09:36:05 LOG: Layout of config file ./dbciRPS.cfg is layout version 1.0
    2016-02-16 09:36:05 LOG: WARNING: we prefer a directory name like CONFIGDIR=./BC
    2016-02-16 09:36:05 LOG: Start Pair Resync S-VOL disks on gltbcp01
    2016-02-16 09:36:05 ERROR: SUSPEND_SYNC_FLAG flag [dbciRPS.suspend] was set
    2016-02-16 09:36:05 ERROR: Exit code 1
    2016-02-16 09:36:05 LOG: Exit code 1


