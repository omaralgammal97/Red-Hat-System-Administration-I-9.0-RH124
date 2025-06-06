# Chapter 11. Analyze and Store Logs

| Content |
| --- |
| [11.1 Describe System Log Architecture](#11.1) |
| [11.3 Review Syslog Files](#11.3) |
| [11.5 Review System Journal Entries](#11.5) |
| [11.7 Preserve the System Journal](#11.7) |
| [11.9 Maintain Accurate Time](#11.9) |


<a name="11.1"></a>
## 11.1 Describe System Log Architecture

| Content |
| --- |
| [systemd-journald](#systemd-journald) |
| [rsyslog](#rsyslog) |

<a name="systemd-journald"></a>
* The ```systemd-journald``` service is at the heart of the operating system event logging architecture. The ```systemd-journald``` service collects event messages from many sources:
  * System kernel
  * Output from the early stages of the boot process
  * Standard output and standard error from daemons
  * Syslog events

<a name="rsyslog"></a>
* The ```rsyslog``` service reads syslog messages that the ```systemd-journald``` service receives from the journal as they arrive. The ```rsyslog``` service then processes the syslog events, and records them to its log files or forwards them to other services according to its own configuration.
* The ```rsyslog``` service sorts and writes syslog messages to the log files that do persist across reboots in the ```/var/log``` directory.
  | Log file | Type of stored messages |
  | --- | --- |
  | ```/var/log/messages``` | Most syslog messages are logged here. Exceptions include messages about authentication and email processing, scheduled job execution, and purely debugging-related messages. |
  | ```/var/log/secure``` | Syslog messages about security and authentication events. |
  | ```/var/log/maillog``` | Syslog messages about the mail server. |
  | ```/var/log/cron``` | Syslog messages about scheduled job execution. |
  | ```/var/log/boot.log``` | Non-syslog console messages about system startup. |


<a name="11.3"></a>
## 11.3 Review Syslog Files

| Content |
| --- |
| [Syslog Facilities](#facilities) |
| [Syslog Priorities](#priorities) |
| [Log File Rotation ```logrotate```](#logrotate) |
| [Analyze a Syslog Entry](#analyze) |
| [Send Syslog Messages Manually](#manually) |

* Many programs use the syslog protocol to log events to the system. Each log message is categorized by ***facility*** (which subsystem produces the message) and ***priority*** (the message's severity).
  <a name="facilities"></a>
  * Overview of Syslog Facilities

     | Code | Facility | Facility description |
     | --- | --- | --- |
     | 0 | kern | Kernel messages |
     | 1 | user | User-level messages |
     | 2 | mail | Mail system messages |
     | 3 | daemon | System daemons messages |
     | 4 | auth | Authentication and security messages |
     | 5 | syslog | Internal syslog messages |
     | 6 | lpr | Printer messages |
     | 7 | news | Network news messages |
     | 8 | uucp | UUCP protocol messages |
     | 9 | cron | Clock daemon messages |
     | 10 | authpriv | Non-system authorization messages |
     | 11 | ftp | FTP protocol messages |
     | 16-23 | local0 | to local7	Custom local messages |

  <a name="priorities"></a>
  * Overview of Syslog Priorities

     | Code | Priority | Priority description |
     | --- | --- | --- |
     | 0 | emerg | System is unusable |
     | 1 | alert | Action must be taken immediately |
     | 2 | crit | Critical condition |
     | 3 | err | Non-critical error condition |
     | 4 | warning | Warning condition |
     | 5 | notice | Normal but significant event |
     | 6 | info | Informational event |
     | 7 | debug | Debugging-level message |

  * Sample Rules of the ```rsyslog``` Service
    ```bash
    #### RULES ####

    # Log all kernel messages to the console.
    # Logging much else clutters up the screen.
    #kern.*                                                 /dev/console

    # Log anything (except mail) of level info or higher.
    # Don't log private authentication messages!
    *.info;mail.none;authpriv.none;cron.none                /var/log/messages

    # The authpriv file has restricted access.
    authpriv.*                                              /var/log/secure

    # Log all the mail messages in one place.
    mail.*                                                  -/var/log/maillog


    # Log cron stuff
    cron.*                                                  /var/log/cron

    # Everybody gets emergency messages
    .emerg                                                 :omusrmsg:

    # Save news errors of level crit and higher in a special file.
    uucp,news.crit                                          /var/log/spooler

    # Save boot messages also to boot.log
    local7.*                                                /var/log/boot.log
    ```
   
<a name="logrotate"></a>
* The ```logrotate``` command rotates log files to prevent them from taking too much space in the ```/var/log``` directory. When a log file is rotated, it is renamed with an extension that indicates the rotation date. For example, the old ```/var/log/messages``` file is renamed to the ```/var/log/messages-20220320``` file when it is rotated on 2022-03-20. After the old log file rotates, it creates a log file and notifies the service that wrote the log file.
* After rotations during typically four weeks, the oldest log file is discarded to free disk space. A scheduled job runs the ```logrotate``` command daily to see the rotation requirement of any log files. Most log files rotate weekly; the ```logrotate``` command rotates some log files faster, or more slowly, or when they reach a specific size.

<a name="analyze"></a>
* Log messages start with the oldest message at the start and the newest message at the end of the log file. The ```rsyslog``` service uses a standard format for recording entries in log files. The following example explains the anatomy of a log message in the ```/var/log/secure``` log file.
  ```bash
  Mar 20 20:11:48 localhost sshd[1433]: Failed password for student from 172.25.0.10 port 59344 ssh2
  ```
  * ```Mar 20 20:11:48``` : Records the time stamp of the log entry.
  * ```localhost``` : The host that sends the log message.
  * ```sshd[1433```] : The program or process name and PID number that sent the log message.
  * ```Failed password for …```: The message that was sent.

<a name="manually"></a>
* The ```logger``` command sends messages to the ```rsyslog``` service. By default, it sends the message to the user type with the ```notice``` priority (```user.notice```) unless specified otherwise with the ```-p``` option. It is helpful to test any change to the ```rsyslog``` service configuration.

	To send a message to the rsyslog service to be recorded in the ```/var/log/boot.log``` log file, execute the following ```logger``` command:
  ```console
  [root@host ~]# logger -p local7.notice "Log entry created on host"
  ```


<a name="11.5"></a>
## 11.5 Review System Journal Entries

| Content |
| --- |
| [```journalctl```](#journalctl) |
| [```-n```](#-n) |
| [```-f```](#-f) |
| [```-p```](#-p) |
| [```-u```](#-u) |
| [```--since``` and ```--until```](#--since_--until) |
| [```verbose```](#verbose) |

* The ```systemd-journald``` service stores logging data in a structured, indexed binary file called ```journal```.
<a name="journalctl"></a>
* You can use the ```journalctl``` command to view all messages in the journal, or to search for specific events based on a wide range of options and criteria. If you run the command as ```root```, then you have full access to the journal. Regular users can also use the ```journalctl``` command, but the system restricts them from seeing certain messages.
  ```console
  [root@host ~]# journalctl
  ...output omitted...
  Mar 15 04:42:16 host.lab.example.com systemd[2127]: Listening on PipeWire Multimedia System Socket.
  Mar 15 04:42:16 host.lab.example.com systemd[2127]: Starting Create User's Volatile Files and Directories...
  Mar 15 04:42:16 host.lab.example.com systemd[2127]: Listening on D-Bus User Message Bus Socket.
  Mar 15 04:42:16 host.lab.example.com systemd[2127]: Reached target Sockets.
  Mar 15 04:42:16 host.lab.example.com systemd[2127]: Finished Create User's Volatile Files and Directories.
  Mar 15 04:42:16 host.lab.example.com systemd[2127]: Reached target Basic System.
  Mar 15 04:42:16 host.lab.example.com systemd[1]: Started User Manager for UID 0.
  Mar 15 04:42:16 host.lab.example.com systemd[2127]: Reached target Main User Target.
  Mar 15 04:42:16 host.lab.example.com systemd[2127]: Startup finished in 90ms.
  Mar 15 04:42:16 host.lab.example.com systemd[1]: Started Session 6 of User root.
  Mar 15 04:42:16 host.lab.example.com sshd[2110]: pam_unix(sshd:session): session opened for user root(uid=0) by (uid=0)
  Mar 15 04:42:17 host.lab.example.com systemd[1]: Starting Hostname Service...
  Mar 15 04:42:17 host.lab.example.com systemd[1]: Started Hostname Service.
  lines 1951-2000/2000 (END) q
  ```
* The ```journalctl``` command highlights important log messages; messages at ```notice``` or ```warning``` priority are in bold text, while messages at the ```error``` priority or higher are in red text.

<a name="-n"></a>
* By default, the journalctl command -n option shows the last 10 log entries.
  ```console
  [root@host ~]# journalctl -n 5
  Mar 15 04:42:17 host.lab.example.com systemd[1]: Started Hostname Service.
  Mar 15 04:42:47 host.lab.example.com systemd[1]: systemd-hostnamed.service: Deactivated successfully.
  Mar 15 04:47:33 host.lab.example.com systemd[2127]: Created slice User Background Tasks Slice.
	Ma   15 04:47:33 host.lab.example.com systemd[2127]: Starting Cleanup of User's Temporary Files and Directories...
  Mar 15 04:47:33 host.lab.example.com systemd[2127]: Finished Cleanup of User's Temporary Files and Directories.
  ```

<a name="-f"></a>
* Similar to the ```tail``` command, the ```journalctl``` command ```-f``` option outputs the last 10 lines of the system journal and continues to output new journal entries as the journal appends them. To exit the ```journalctl``` command ```-f``` option, use the **Ctrl+C** key combination.
  ```console
  [root@host ~]# journalctl -f
  Mar 15 04:47:33 host.lab.example.com systemd[2127]: Finished Cleanup of User's Temporary Files and Directories.
  Mar 15 05:01:01 host.lab.example.com CROND[2197]: (root) CMD (run-parts /etc/cron.hourly)
  Mar 15 05:01:01 host.lab.example.com run-parts[2200]: (/etc/cron.hourly) starting 0anacron
  Mar 15 05:01:01 host.lab.example.com anacron[2208]: Anacron started on 2022-03-15
  Mar 15 05:01:01 host.lab.example.com anacron[2208]: Will run job `cron.daily' in 29 min.
  Mar 15 05:01:01 host.lab.example.com anacron[2208]: Will run job `cron.weekly' in 49 min.
  Mar 15 05:01:01 host.lab.example.com anacron[2208]: Will run job `cron.monthly' in 69 min.
  Mar 15 05:01:01 host.lab.example.com anacron[2208]: Jobs will be executed sequentially
  Mar 15 05:01:01 host.lab.example.com run-parts[2210]: (/etc/cron.hourly) finished 0anacron
  Mar 15 05:01:01 host.lab.example.com CROND[2196]: (root) CMDEND (run-parts /etc/cron.hourly)
  ^C
  [root@host ~]#
  ```

<a name="-p"></a>
* To help to troubleshoot problems, you might want to filter the output of the journal by the priority of the journal entries. The ```journalctl``` command ```-p``` option shows the journal entries at a specified priority level (by name or by number) or higher. The journalctl command processes the ```debug```, ```info```, ```notice```, ```warning```, ```err```, ```crit```, ```alert```, and ```emerg``` priority levels, in ascending priority order.
  
  As an example, run the following journalctl command to list journal entries at the err priority <ins>***or higher***</ins>:
  ```console
  [root@host ~]# journalctl -p err
  Mar 15 04:22:00 host.lab.example.com pipewire-pulse[1640]: pw.conf: execvp error 'pactl': No such file or direct
  Mar 15 04:22:17 host.lab.example.com kernel: Detected CPU family 6 model 13 stepping 3
  Mar 15 04:22:17 host.lab.example.com kernel: Warning: Intel Processor - this hardware has not undergone testing by Red Hat and might not be certif>
  Mar 15 04:22:20 host.lab.example.com smartd[669]: DEVICESCAN failed: glob(3) aborted matching pattern /dev/discs/disc*
  Mar 15 04:22:20 host.lab.example.com smartd[669]: In the system's table of devices NO devices found to scan
  ```

<a name="-u"></a>
* You can show messages for a specified systemd unit by using the ```journalctl``` command ```-u``` option and the unit name.
  ```console
  [root@host ~]# journalctl -u sshd.service
  May 15 04:30:18 host.lab.example.com systemd[1]: Starting OpenSSH server daemon...
  May 15 04:30:18 host.lab.example.com sshd[1142]: Server listening on 0.0.0.0 port 22.
  May 15 04:30:18 host.lab.example.com sshd[1142]: Server listening on :: port 22.
  May 15 04:30:18 host.lab.example.com systemd[1]: Started OpenSSH server daemon.
  May 15 04:32:03 host.lab.example.com sshd[1796]: Accepted publickey for user1 from 172.25.250.254 port 43876 ssh2: RSA SHA256:1UGy...>
  May 15 04:32:03 host.lab.example.com sshd[1796]: pam_unix(sshd:session): session opened for user user1(uid=1000) by (uid=0)
  May 15 04:32:26 host.lab.example.com sshd[1866]: Accepted publickey for user2 from ::1 port 36088 ssh2: RSA SHA256:M8ik...
  May 15 04:32:26 host.lab.example.com sshd[1866]: pam_unix(sshd:session): session opened for user user2(uid=1001) by (uid=0)
  lines 1-8/8 (END) q
  ```

<a name="--since_--until"></a>
* When looking for specific events, you can limit the output to a specific time frame. The ```journalctl``` command has two options to limit the output to a specific time range, the ```--since``` and ```--until``` options. Both options take a time argument in the "***YYYY-MM-DD hh:mm:ss***" format (the double quotation marks are required to preserve the space in the option).
* The ```journalctl``` command assumes that the day starts at **00:00:00** when you omit the time argument. The command also assumes the current day when you omit the day argument. Both options take ```yesterday```, ```today```, and ```tomorrow``` as valid arguments in addition to the date and time field.
  ```console
  [root@host ~]# journalctl --since today
  ...output omitted...
  Mar 15 05:04:20 host.lab.example.com systemd[1]: Started Session 8 of User student.
  Mar 15 05:04:20 host.lab.example.com sshd[2255]: pam_unix(sshd:session): session opened for user student(uid=1000) by (uid=0)
  Mar 15 05:04:20 host.lab.example.com systemd[1]: Starting Hostname Service...
  Mar 15 05:04:20 host.lab.example.com systemd[1]: Started Hostname Service.
  Mar 15 05:04:50 host.lab.example.com systemd[1]: systemd-hostnamed.service: Deactivated successfully.
  Mar 15 05:06:33 host.lab.example.com systemd[2261]: Starting Mark boot as successful...
  Mar 15 05:06:33 host.lab.example.com systemd[2261]: Finished Mark boot as successful.
  lines 1996-2043/2043 (END) q
  ```
  ```console
  [root@host ~]# journalctl --since "2022-03-11 20:30" --until "2022-03-14 10:00"
  ...output omitted...
  ```
  ```console
  [root@host ~]# journalctl --since "-1 hour"
  ...output omitted...
  ```

<a name="verbose"></a>
* you can view additional log entries if you turn on the ```verbose``` output.
  ```console
  [root@host ~]# journalctl -o verbose
  Tue 2022-03-15 05:10:32.625470 EDT [s=e7623387430b4c14b2c71917db58e0ee;i...]
      _BOOT_ID=beaadd6e5c5448e393ce716cd76229d4
      _MACHINE_ID=4ec03abd2f7b40118b1b357f479b3112
      PRIORITY=6
      SYSLOG_FACILITY=3
      SYSLOG_IDENTIFIER=systemd
      _UID=0
      _GID=0
      _TRANSPORT=journal
      _CAP_EFFECTIVE=1ffffffffff
      TID=1
      CODE_FILE=src/core/job.c
      CODE_LINE=744
      CODE_FUNC=job_emit_done_message
      JOB_RESULT=done
      _PID=1
      _COMM=systemd
      _EXE=/usr/lib/systemd/systemd
      _SYSTEMD_CGROUP=/init.scope
      _SYSTEMD_UNIT=init.scope
      _SYSTEMD_SLICE=-.slice
      JOB_TYPE=stop
      MESSAGE_ID=9d1aaa27d60140bd96365438aad20286
      _HOSTNAME=host.lab.example.com
      _CMDLINE=/usr/lib/systemd/systemd --switched-root --system --deserialize 31
      _SELINUX_CONTEXT=system_u:system_r:init_t:s0
      UNIT=user-1000.slice
      MESSAGE=Removed slice User Slice of UID 1000.
      INVOCATION_ID=0e5efc1b4a6d41198f0cf02116ca8aa8
      JOB_ID=3220
      _SOURCE_REALTIME_TIMESTAMP=1647335432625470
  lines 46560-46607/46607 (END) q
  ```

* The following list shows the common fields of the system journal that you can use to search for relevant lines to a particular process or event:
  * ```_COMM``` is the command name.
  * ```_EXE``` is the path to the executable file for the process.
  * ```_PID``` is the PID of the process.
  * ```_UID``` is the UID of the user that runs the process.
  * ```_SYSTEMD_UNIT``` is the ```systemd``` unit that started the process.
  ```console
  [root@host ~]# journalctl _SYSTEMD_UNIT=sshd.service _PID=2110
  Mar 15 04:42:16 host.lab.example.com sshd[2110]: Accepted publickey for root from 172.25.250.254 port 46224 ssh2: RSA SHA256:1UGybTe52L2jzEJa1HLVKn9QUCKrTv3ZzxnMJol1Fro
  Mar 15 04:42:16 host.lab.example.com sshd[2110]: pam_unix(sshd:session): session opened for user root(uid=0) by (uid=0)
  ```


<a name="11.7"></a>
## 11.7 Preserve the System Journal

| Content |
| --- |
| [System Journal Storage](#explain) |
| [Configure Persistent System Journals](#configure_persistent) |

<a name="explain"></a>
* By default, Red Hat Enterprise Linux 9 stores the system journal in the ```/run/log``` directory, and the system clears the system journal after a reboot. You can change the configuration settings of the ```systemd-journald``` service in the ```/etc/systemd/journald.conf``` file so that the journals persist across a reboot.

* The ```Storage``` parameter in the ```/etc/systemd/journald.conf``` file defines whether to store system journals in a volatile manner or persistently across a reboot. Set this parameter to ```persistent```, ```volatile```, ```auto```, or ```none``` as follows:
  * ```persistent```: Stores journals in the ```/var/log/journal``` directory, which persists across reboots. If the ```/var/log/journal``` directory does not exist, then the ```systemd-journald``` service creates it.
  * ```volatile```: Stores journals in the volatile ```/run/log/journal``` directory. As the ```/run``` file system is temporary and exists only in the runtime memory, the data in it, including system journals, does not persist across a reboot.
  * ```auto```: If the ```/var/log/journal``` directory exists, then the ```systemd-journald``` service uses persistent storage; otherwise it uses volatile storage. This action is the default if you do not set the ```Storage``` parameter.
  * ```none```: Do not use any storage. The system drops all logs, but you can still forward the logs.

* The advantage of persistent system journals is that the historical data is available immediately at boot. However, even with a persistent journal, the system does not keep all data forever. The journal has a built-in log rotation mechanism that triggers monthly. In addition, the system does not allow the journals to get larger than 10% of the file system that they are on, or to leave less than 15% of the file system free. You can modify these values for both the runtime and persistent journals in the ```/etc/systemd/journald.conf``` configuration file.
  * The following command output shows the journal entries that reflect the current size limits:
    ```console
    [user@host ~]$ journalctl | grep -E 'Runtime Journal|System Journal'
    Mar 15 04:21:14 localhost systemd-journald[226]: Runtime Journal (/run/log/journal/4ec03abd2f7b40118b1b357f479b3112) is 8.0M, max 113.3M, 105.3M free.
    Mar 15 04:21:19 host.lab.example.com systemd-journald[719]: Runtime Journal (/run/log/journal/4ec03abd2f7b40118b1b357f479b3112) is 8.0M, max 113.3M, 105.3M free.
    Mar 15 04:21:19 host.lab.example.com systemd-journald[719]: System Journal (/run/log/journal/4ec03abd2f7b40118b1b357f479b3112) is 8.0M, max 4.0G, 4.0G free.
    ```

<a name="configure_persistent"></a>
* set the ```Storage``` parameter to the ```persistent``` value in the ```/etc/systemd/journald.conf``` file. 
  ```bash
  [Journal]
  Storage=persistent
  ...output omitted...
  ```
* Restart the ```systemd-journald``` service to apply the configuration changes.
  ```console
  [root@host ~]# systemctl restart systemd-journald```
  ```

* If the ```systemd-journald``` service successfully restarts, then the service creates the ```/var/log/journal``` directory and it contains one or more subdirectories. These subdirectories have hexadecimal characters in their long names and contain files with the ```.journal``` extension. The ```.journal``` binary files store the structured and indexed journal entries.
  ```console
  [root@host ~]# ls /var/log/journal
  4ec03abd2f7b40118b1b357f479b3112
  [root@host ~]# ls /var/log/journal/4ec03abd2f7b40118b1b357f479b3112
  system.journal  user-1000.journal
  ```
* While the system journals persist after a reboot, the ```journalctl``` command output includes entries from the current system boot as well as the previous system boots. To limit the output to a specific system boot, use the ```journalctl``` command ```-b``` option. The following ```journalctl``` command retrieves the entries from the first system boot only:
  ```console
  [root@host ~]# journalctl -b 1
  ...output omitted...
  ```
* The following ```journalctl``` command retrieves the entries from the second system boot only. The argument is meaningful only if the system was rebooted at least twice:
  ```console
  [root@host ~]# journalctl -b 2
  ...output omitted...
  ```
* You can list the system boot events that the ```journalctl``` command recognizes by using the ```--list-boots``` option.
  ```console
  [root@host ~]# journalctl --list-boots
    -6 27de... Wed 2022-04-13 20:04:32 EDT—Wed 2022-04-13 21:09:36 EDT
    -5 6a18... Tue 2022-04-26 08:32:22 EDT—Thu 2022-04-28 16:02:33 EDT
    -4 e2d7... Thu 2022-04-28 16:02:46 EDT—Fri 2022-05-06 20:59:29 EDT
    -3 45c3... Sat 2022-05-07 11:19:47 EDT—Sat 2022-05-07 11:53:32 EDT
    -2 dfae... Sat 2022-05-07 13:11:13 EDT—Sat 2022-05-07 13:27:26 EDT
    -1 e754... Sat 2022-05-07 13:58:08 EDT—Sat 2022-05-07 14:10:53 EDT
     0 ee2c... Mon 2022-05-09 09:56:45 EDT—Mon 2022-05-09 12:57:21 EDT
  ```
* The following ```journalctl``` command retrieves the entries from the current system boot only:
  ```console
  [root@host ~]# journalctl -b
  ...output omitted...
  ```


<a name="11.9"></a>
## 11.9 Maintain Accurate Time

| Content |
| --- |
| [timedatectl](#timedatectl) |
| [tzselect](#tzselect) |
| [Configure and Monitor the chronyd Service](#chronyd) |

<a name="timedatectl"></a>
* The ```timedatectl``` command shows an overview of the current time-related system settings, including the current time, time zone, and NTP synchronization settings of the system.
  ```console
  [user@host ~]$ timedatectl
                 Local time: Wed 2022-03-16 05:53:05 EDT
             Universal time: Wed 2022-03-16 09:53:05 UTC
                   RTC time: Wed 2022-03-16 09:53:05
                  Time zone: America/New_York (EDT, -0400)
  System clock synchronized: yes
                NTP service: active
            RTC in local TZ: no
  ```

* You can list a database of time zones with the ```timedatectl``` command ```list-timezones``` option.
  ```console
  [user@host ~]$ timedatectl list-timezones
  Africa/Abidjan
  Africa/Accra
  Africa/Addis_Ababa
  Africa/Algiers
  Africa/Asmara
  Africa/Bamako
  ...output omitted...
  ```
  * The Internet Assigned Numbers Authority (IANA) provides a public time zone database, and the ```timedatectl``` command bases the time zone names on that database. IANA names time zones based on the continent or ocean, and then typically (not always) the largest city within the time zone region. For example, most of the US Mountain time zone is ```America/Denver```.
  * Some localities inside the time zone have different daylight saving time rules. For example, in the US, much of the state of Arizona (US Mountain time) does not have a daylight saving time adjustment at all and is in the time zone ```America/Phoenix```.

* The ```root``` user can change the system setting to update the current time zone with the ```timedatectl``` command ```set-timezone``` option. For example, the following ```timedatectl``` command updates the current time zone to ```America/Phoenix```.
  ```console
  [root@host ~]# timedatectl set-timezone America/Phoenix
  [root@host ~]# timedatectl
                 Local time: Wed 2022-03-16 03:05:55 MST
             Universal time: Wed 2022-03-16 10:05:55 UTC
                   RTC time: Wed 2022-03-16 10:05:55
                  Time zone: America/Phoenix (MST, -0700)
  System clock synchronized: yes
                NTP service: active
            RTC in local TZ: no
  ```

* Use the ```timedatectl``` command ```set-tim```e option to change the system's current time. You might specify the time in the "***YYYY-MM-DD hh:mm:ss***" format, where you can omit either the date or time. For example, the following ```timedatectl``` command changes the time to ```09:00:00```.
  ```console
  [root@host ~]# timedatectl set-time 9:00:00
  [root@host ~]# timedatectl
                 Local time: Fri 2019-04-05 09:00:27 MST
             Universal time: Fri 2019-04-05 16:00:27 UTC
                   RTC time: Fri 2019-04-05 16:00:27
                  Time zone: America/Phoenix (MST, -0700)
  System clock synchronized: yes
                NTP service: active
            RTC in local TZ: no
  ```

* The ```timedatectl``` command ```set-ntp``` option enables or disables NTP synchronization for automatic time adjustment. The option requires either a ```true``` or ```false``` argument to turn it on or off. For example, the following ```timedatectl``` command turns off NTP synchronization.
```console
[root@host ~]# timedatectl set-ntp false
```

<a name="tzselect"></a>
* Use the ```tzselect``` command to identify the correct time zone name. This command interactively prompts the user with questions about the system's location, and outputs the name of the correct time zone. It does not change the time zone setting of the system.

<a name="chronyd"></a>
* The ```chronyd``` service keeps on track the usually inaccurate local Real-Time Clock (RTC) by synchronizing it to the configured NTP servers. If no network connectivity is available, then the chronyd service calculates the RTC clock drift, and records it in the file that the ```driftfile``` value specifies in the ```/etc/chrony.conf``` configuration file.
* By default, the ```chronyd``` service uses servers from the NTP Pool Project to synchronize time and requires no additional configuration. You might need to change the NTP servers for a machine that runs on an isolated network.

* The ***stratum*** of the NTP time source determines its quality. The stratum determines the number of hops the machine is away from a high-performance reference clock. The reference clock is a ```stratum 0``` time source. An NTP server that is directly attached to the reference clock is a ```stratum 1``` time source, while a machine that synchronizes time from the NTP server is a ```stratum 2``` time source.

* The server and peer are the two categories of time sources that you can declare in the ```/etc/chrony.conf``` configuration file. The server is one stratum above the local NTP server, and the peer is at the same stratum level. You can define multiple servers and peers in the ```chronyd``` configuration file, one per line.

* The first argument of the ```server``` line is the IP address or DNS name of the NTP server. Following the server IP address or name, you can list a series of options for the server. Red Hat recommends to use the ```iburst``` option, because then the ```chronyd``` service takes four measurements in a short time period for a more accurate initial clock synchronization after the service starts. Use the ```man 5 chrony.conf``` command for more information about the ```chronyd``` configuration file options.

  * As an example, with the following ```server classroom.example.com iburst``` line in the ```/etc/chrony.conf``` configuration file, the ```chronyd``` service uses the ```classroom.example.com``` server as the NTP time source.
    ```bash
    # Use public servers from the pool.ntp.org project.
    ...output omitted...
    server classroom.example.com iburst
    ...output omitted...
    ```
  * Restart the service after pointing the ```chronyd``` service to the ```classroom.example.com``` local time source.
    ```console
    [root@host ~]# systemctl restart chronyd
    ```

* The ```chronyc``` command acts as a client to the ```chronyd``` service. After setting up NTP synchronization, verify that the local system is seamlessly using the NTP server to synchronize the system clock by using the ```chronyc sources``` command. For more verbose output with additional explanations about the output, use the ```chronyc sources -v``` command.
  ```console
  [root@host ~]# chronyc sources -v

    .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
   / .- Source state '*' = current best, '+' = combined, '-' = not combined,
  | /             'x' = may be in error, '~' = too variable, '?' = unusable.
  ||                                                 .- xxxx [ yyyy ] +/- zzzz
  ||      Reachability register (octal) -.           |  xxxx = adjusted offset,
  ||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
  ||                                \     |          |  zzzz = estimated error.
  ||                                 |    |           \
  MS Name/IP address         Stratum Poll Reach LastRx Last sample
  ===============================================================================
  ^* 172.25.254.254                3   6    17    26  +2957ns[+2244ns] +/-   25ms
  ```
  * The asterisk character (*) in the ```S``` (Source state) field indicates that the ```chronyd``` service uses the ```classroom.example.com``` server as a time source and is the NTP server that the machine is currently synchronized to.

