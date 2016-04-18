Script horcmd-initscript-rhel-script.sh
=======================================

The HORCM installation comes without an init [#cit1]_ start-up script, which means after each reboot of the system you need to restart ther horcmd daemons manually again. This is not very practical and even bad system administation practice.
Therefore, we wrote this small script to automate this task for you at least on RedHat Linux versions (in my customer base it was mostly RHEL 6). For RHEL 7 we probably need a systemd kind of script. Also, for other Linux distributions this script may not be 100% compatible, but at least it will give an indication how an initscript should look like.

The script is self explaining and does not need any manually intervention. If horcmd daemons are running it will automatically created the proper settings in `/etc/sysconfig/horcmd`

The source of the script is available at `GitHub <https://github.com/gdha/horcm-utils/blob/master/usr/local/sbin/horcmd-initscript-rhel-script.sh>`_

If you create a similar init script for another kind of Linux distribution let me know via opening an `issue on GitHub <https://github.com/gdha/horcm-utils/issues>`_ and we will be glad to add it to the HORCM Utilities.

.. rubric:: Citations

.. [#cit1] http://fedoraproject.org/wiki/EPEL:SysVInitScript
