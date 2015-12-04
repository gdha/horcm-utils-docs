BC-exec.sh Logging
==================

Logging actions and steps are key in business copy operations necessary to track what happened and in case of issues to find out what and when something went wrong. Therefore, we foresee two manners of logging:

 * Logging to log files
 * Logging to a central syslog file

The log files created by BC-exec.sh
-----------------------------------

The BC-exec.sh sends all output to the **/var/adm/log** directory. After each run a new file is created::

    <ConfigFile(without .cfg)>-<operation>-BC-exec-<YYYYMMDD>-<HHMMSS>-<PID>.log

To view the log files use the **cat** command::

    $ cat dbciRPS-BC1-mount-BC-exec-20131030-141414-14773.log

We can also modify on the command line the target log directory, therefore, use the ``-D`` option.

Information Logged via syslogd
------------------------------

The BC-exec.sh script sends short messages to the syslogd which will be added to the **/var/adm/syslog/syslog.log** (on HP-UX) or **/var/log/messages** (Linux). To view relevant messages use the grep command::

    $ grep BC-exec /var/adm/syslog/syslog.log
    Sep 17 13:48:45 gltbcp01 BC-exec.sh[1765]: <error> SUSPEND_SYNC_FLAG flag [suspend_synC] was set
    Sep 17 13:55:58 gltbcp01 BC-exec.sh[2004]: <info> pairdisplay -IBC6 -g vgdbRPS (VG vgBC6_vgdbRPS) executed with success
    Sep 17 13:57:03 gltbcp01 BC-exec.sh[2095]: <info> pairsplit -BC6 -g vgdbRPS (VG vgBC6_vgdbRPS) executed with success
    
