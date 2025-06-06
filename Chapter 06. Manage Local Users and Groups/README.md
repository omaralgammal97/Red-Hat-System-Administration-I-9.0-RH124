# Chapter 6. Manage Local Users and Groups

| content |
| --- |
| [6.1 Describe User and Group Concepts](#6.1) |
| [6.3 Gain Superuser Access](#6.3) |
| [6.5 Manage Local User Accounts](#6.5) |
| [6.7 Manage Local Group Accounts](#6.7) |
| [6.9 Manage User Passwords](#6.9) |


<a name="6.1"></a>
## 6.1 Describe User and Group Concepts

| Commands |
| --- |
| [id](#id) |
| [ls -l](#-l) |
| [ls -ld](#-ld) |
| [ps](#ps) |
| [cat /etc/passwd](#/etc/passwd) |
| [cat /etc/group](#/etc/group) |

<a name="id"></a>
* Use the ```id``` command to show information about the currently logged-in user:
```console
[user01@host ~]$ id
uid=1000(user01) gid=1000(user01) groups=1000(user01) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```
* To view basic information about another user, pass the username to the ```id``` command as an argument:
```console
[user01@host ~]$ id user02
uid=1002(user02) gid=1001(user02) groups=1001(user02) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```
<a name="-l"></a>
* Use the ```ls -l``` command to view the owner of a file. 
```console
[user01@host ~]$ ls -l mytextfile.txt
-rw-rw-r--. 1 user01 user01 0 Feb  5 11:10 mytextfile.txt
```
<a name="-ld"></a>
* Use the ```ls -ld``` command to view the owner of a directory, rather than the contents of that directory. In the following output, the third column shows the username.
```console
[user01@host]$ ls -ld Documents
drwxrwxr-x. 2 user01 user01 6 Feb  5 11:10 Documents
```
<a name="ps"></a>
* Use the ```ps``` command to view process information. The default is to show only processes in the current shell. Use the ps command ```-a``` option to view all processes with a terminal. Use the ps command ```-u``` option to view the user that is associated with a process. In the following output, the first column shows the username.
```console
[user01@host ~]$ ps -au
USER     PID %CPU %MEM    VSZ   RSS TTY    STAT START  TIME COMMAND
root    1690  0.0  0.0 220984  1052 ttyS0  Ss+  22:43  0:00 /sbin/agetty -o -p -- \u --keep-baud 1
user01  1769  0.0  0.1 377700  6844 tty2   Ssl+ 22:45  0:00 /usr/libexec/gdm-x-session --register-
user01  1773  1.3  1.3 528948 78356 tty2   Sl+  22:45  0:03 /usr/libexec/Xorg vt2 -displayfd 3 -au
user01  1800  0.0  0.3 521412 19824 tty2   Sl+  22:45  0:00 /usr/libexec/gnome-session-binary
user01  3072  0.0  0.0 224152  5756 pts/1  Ss   22:48  0:00 -bash
user01  3122  0.0  0.0 225556  3652 pts/1  R+   22:49  0:00 ps -au
```
<a name="/etc/passwd"></a>
* Each line in the ```/etc/passwd``` file contains information about one user. The file is divided into seven colon-separated fields.
  ```console
  [user01@host ~]$ cat /etc/passwd
  ...output omitted...
  user01:x:1000:1000:User One:/home/user01:/bin/bash
  ```
  Consider each part of the code block, separated by a colon:
  * ```user01``` : The username for this user.
  * ```x``` : The user's encrypted password was historically stored here; this is now a placeholder.
  * ```1000``` : The UID number for this user account.
  * ```1000``` : The GID number for this user account's primary group. Groups are discussed later in this section.
  * ```User One``` : A brief comment, description, or the real name for this user.
  * ```/home/user01``` : The user's home directory, and the initial working directory when the login shell starts.
  * ```/bin/bash``` : The default shell program for this user that runs at login. Some accounts use the ```/sbin/nologin``` shell to disallow interactive logins with that account.
<a name="/etc/group"></a>
* Each line in the ```/etc/group``` file contains information about one group. Each group entry is divided into four colon-separated fields.
  ```console
  [user01@host ~]$ cat /etc/group
  ...output omitted...
  group01:x:10000:user01,user02,user03
  ```
  Consider each part of the code block, separated by a colon:
  * ```group01``` : Name for this group.
  * ```x``` : Obsolete group password field; this is now a placeholder.
  * ```10000``` : The GID number for this group (```10000```).
  * ```user01,user02,user03``` : A list of users that are members of this group as a secondary group.

* Primary Groups and Secondary Groups

  Every user has exactly one primary group. For local users, this group is listed by GID in the ```/etc/passwd``` file. The primary group owns files that the user creates.
  
  When creating a regular user, a group is created with the same name as the user, to be the primary group for the user. The user is the only member of this *User Private Group*. This group membership design simplifies the management of file permissions, to have user groups segregated be default.
  
  Users might also have secondary groups. Membership in secondary groups is stored in the ```/etc/group``` file. Users are granted access to files based on whether any of their groups have access, regardless of whether the groups are primary or secondary. For example, if the ```user01``` user has a ```user01``` primary group and ```wheel``` and ```webadmin``` secondary groups, then that user can read files that any of those three groups can read.

  The ```id``` command can show group membership for a user. In the following example, the ```user01``` user has the ```user01``` group as their primary group (```gid```). The ```groups``` item lists all group memberships for this user, and the user also has the ```wheel``` and ```group01``` groups as secondary groups.
  ```console
  [user01@host ~]$ id
  uid=1001(user01) gid=1003(user01) groups=1003(user01),10(wheel),10000(webadmin) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
  ```

<a name="6.3"></a>
## 6.3 Gain Superuser Access

| Commands |
| --- |
| [su -](#su) |
| [sudo -i](#-i) |
| [/etc/sudoers](#/etc/sudoers) |

<a name="su"></a>
* With the su command, users can switch to a different user account.
  ```console
  [user01@host ~]$ su - user02
  Password: user02_password
  [user02@host ~]$
  ```
  ```console
  [user01@host ~]$ su -
  Password: root_password
  [root@host ~]#
  ```
* The command ```su``` starts a non-login shell, while the command ```su -``` (with the dash option) starts a login shell. The main distinction between the two commands is that ```su -``` sets up the shell environment as if it is a new login as that user, while su starts a shell as that user, but uses the original user's environment settings.
  | | ```su``` | ```su -``` | ```sudo``` |
  | --- | --- | --- | --- |
  | Become new user| Yes | Yes | Per escalated command |
  | Environment| Current user's | New user's | Current user's |
  | Password required| New user's | New user's | Current user's |
  | Privileges| Same as new user | Same as new user | Defined by configuration |
  | Activity logged| ```su``` command only | ```su``` command only | Per escalated command |
* Additionally, you can configure the ```sudo``` command to allow specific users to run any command as some other user, or only some commands as that user. For example, if you configure the ```sudo``` command to allow the ```user01``` user to run the ```usermod``` command as ```root```, then you can run the following command to lock or unlock a user account:
  ```console
  [user01@host ~]$ sudo usermod -L user02
  [sudo] password for user01: user01_password
  [user01@host ~]$ su - user02
  Password: user02_password
  su: Authentication failure
  [user01@host ~]$
  ```
* If a user tries to run a command as another user, and the ```sudo``` configuration does not permit it, then bash blocks the command, logs the attempt, and sends by default an email to the ```root``` user.
  ```console
  [user02@host ~]$ sudo tail /var/log/secure
  [sudo] password for user02: user02_password
  user02 is not in the sudoers file.  This incident will be reported.
  [user02@host ~]$
  ```
<a name="-i"></a>
* To access the ```root``` account with ```sudo```, use the ```sudo -i``` command. This command switches to the root account and runs that user's default shell (usually bash) and associated interactive login scripts. To run the shell without the interactive scripts, use the ```sudo -s``` command.

  For example, an administrator can get an interactive shell as ```root``` on an AWS Elastic Cloud Computing (EC2) instance by using SSH public-key authentication to log in as the ```ec2-user``` normal user, and then run the ```sudo -i``` command to access the ```root``` user's shell.
  ```console
  [ec2-user@host ~]$ sudo -i
  [sudo] password for ec2-user: ec2-user_password
  [root@host ~]#
  ```
  * ```sudo -i``` is generally preferred over ```su -``` because it provides a more secure way to run commands with elevated privileges. When you use ```sudo -i```, you are still using your own user account but with root privileges, whereas ```su -``` switches to the root user account entirely. This means that with ```sudo -i```, any commands that you run will be logged to your account, making it easier to track what was done with root privileges.
  * ```su -``` is useful when you need to run multiple commands as the root user or when you need to switch to another user account with root privileges. With ```su -```, you can switch to any user account, not just the root user account.
  * ```sudo -i``` is also useful when you want to run a command with root privileges but need to preserve the environment variables of your current user account. This is because ```sudo -i``` starts a new login shell with root privileges, which loads the root user's environment variables.
<a name="/etc/sudoers"></a>
* The ```/etc/sudoers``` file is the main configuration file for the ```sudo``` command. To avoid problems if multiple administrators try to edit the file at the same time, you can edit it only with the special ```visudo``` command. The ```visudo``` editor also validates the file, to ensure no syntax errors.
  ```bash
  %wheel        ALL=(ALL:ALL)       ALL
  ```
  * The ```%wheel``` string is the user or group that the rule applies to. The ```%``` symbol before the word ```wheel``` specifies a group.
  * The ```ALL=(ALL:ALL)``` command specifies that on any host with this file (the first ```ALL```), users in the ```wheel``` group can run commands as any other user (the second ```ALL```) and any other group (the third ```ALL```) on the system.
  * The final ```ALL``` command specifies that users in the ```wheel``` group can run any command.

  
<a name="6.5"></a>
## 6.5 Manage Local User Accounts

| Commands |
| --- |
| [useradd](#useradd) |
| [usermod](#usermod) |
| [userdel](#userdel) |
| [passwd](#passwd) |

<a name="useradd"></a>
* The ```useradd username``` command creates a user called ```username```.
* ```useradd``` command assigns new users the first free UID that is greater than or equal to 1000, unless you explicitly specify one by using the ```-u``` option.
<a name="usermod"></a>
* The ```useradd --help``` command displays the basic options to override the defaults. In most cases, you can use the same options with the ```usermod``` command to modify an existing user.
  
  | usermod options: | Usage |
  | --- | --- |
  | ```-a```, ```--append``` | Use it with the ```-G``` option to add the secondary groups to the user's current set of group memberships instead of replacing the set of secondary groups with a new set. |
  | ```-c```, ```--comment``` COMMENT | Add the ```COMMENT``` text to the comment field. |
  | ```-d```, ```--home``` HOME_DIR | Specify a home directory for the user account. |
  | ```-g```, ```--gid``` GROUP | Specify the primary group for the user account. |
  | ```-G```, ```--groups``` GROUPS | Specify a comma-separated list of secondary groups for the user account. |
  | ```-L```, ```--lock``` | Lock the user account. |
  | ```-m```, ```--move-home``` | Move the user's home directory to a new location. You must use it with the ```-d``` option. |
  | ```-s```, ```--shell``` SHELL | Specify a particular login shell for the user account. |
  | ```-U```, ```--unlock``` | Unlock the user account. |
  
<a name="userdel"></a>
* The ```userdel username``` command removes the ```username``` user from ```/etc/passwd```, but leaves the user's home directory intact. The ```userdel -r username``` command removes the user from ```/etc/passwd``` and deletes the user's home directory
* The following example demonstrates how this can lead to information leakage:
  ```console
  [root@host ~]# useradd user01
  [root@host ~]# ls -l /home
  drwx------. 3 user01  user01    74 Mar  4 15:22 user01
  [root@host ~]# userdel user01
  [root@host ~]# ls -l /home
  drwx------. 3    1000    1000   74 Mar  4 15:22 user01
  [root@host ~]# useradd -u 1000 user02
  [root@host ~]# ls -l /home
  drwx------. 3 user02     user02       74 Mar  4 15:23 user02
  drwx------. 3 user02     user02       74 Mar  4 15:22 user01
  ```
  Notice that ```user02``` now owns all files that ```user01``` previously owned. The ```root``` user can use the ```find / -nouser -o -nogroup``` command to find all unowned files and directories.

<a name="passwd"></a>
```console
[root@host ~]# passwd user01
Changing password for user user01.
New password: redhat
BAD PASSWORD: The password is shorter than 8 characters
Retype new password: redhat
passwd: all authentication tokens updated successfully.
[root@host ~]#
```

* UID Ranges
  * **UID 0** : The superuser (```root```) account UID.
  * **UID 1-200** : System account UIDs statically assigned to system processes.
  * **UID 201-999** : UIDs assigned to system processes that do not own files on this system. Software that requires an unprivileged UID is dynamically assigned UID from this available pool.
  * **UID 1000+** : The UID range to assign to regular, unprivileged users.

<a name="6.7"></a>
## 6.7 Manage Local Group Accounts

| Commands |
| --- |
| [groupadd](#groupadd) |
| [groupmod](#groupmod) |
| [groupdel](#groupdel) |

<a name="groupadd"></a>
* The ```groupadd``` command creates groups. Without options, the ```groupadd``` command uses the next available GID from the range specified by the GID_MIN and GID_MAX variables in the ```/etc/login.defs``` file.
* The groupadd command -g option specifies a particular GID for the group to use.
  ```console
  [root@host ~]# groupadd -g 10000 group01
  [root@host ~]# tail /etc/group
  ...output omitted...
  group01:x:10000:
  ```
* The ```groupadd``` command ```-r``` option creates system groups. As with normal groups, system groups use a GID from the range of listed valid system GIDs in the ```/etc/login.defs``` file. The SYS_GID_MIN and SYS_GID_MAX configuration items in the ```/etc/login.defs``` file define the range of system GIDs.
  ```console
  [root@host ~]# groupadd -r group02
  [root@host ~]# tail /etc/group
  ...output omitted...
  group01:x:10000:
  group02:x:988:
  ```
  
<a name="groupmod"></a>
* The ```groupmod``` command changes the properties of an existing group. The ```groupmod``` command ```-n``` option specifies a new name for the group.
  ```console
  [root@host ~]# groupmod -n group0022 group02
  [root@host ~]# tail /etc/group
  ...output omitted...
  group0022:x:988:
  ```
* Notice that the group name updates to ```group0022``` from ```group02```. The ```groupmod``` command ```-g``` option specifies a new GID.
  ```console
  [root@host ~]# groupmod -g 20000 group0022
  [root@host ~]# tail /etc/group
  ...output omitted...
  group0022:x:20000:
  ```
  ```console
  [root@host ~]# id user02
  uid=1006(user02) gid=1008(user02) groups=1008(user02)
  [root@host ~]# usermod -g group01 user02
  [root@host ~]# id user02
  uid=1006(user02) gid=10000(group01) groups=10000(group01)
  ```
* Use the ```usermod -aG``` command to add a user to a secondary group.
  ```console
  [root@host ~]# id user03
  uid=1007(user03) gid=1009(user03) groups=1009(user03)
  [root@host ~]# usermod -aG group01 user03
  [root@host ~]# id user03
  uid=1007(user03) gid=1009(user03) groups=1009(user03),10000(group01)
  ```
  
<a name="groupdel"></a>
* The ```groupdel``` command removes groups.
```console
[root@host ~]# groupdel group0022
```

<a name="6.9"></a>
## 6.9 Manage User Passwords

| Content |
| --- |
| [/etc/shadow](#/etc/shadow) |
| [chage](#chage) |

<a name="/etc/shadow"></a>
* Each user has an entry with in the ```/etc/shadow``` file.
  ```console
  [root@host ~]# cat /etc/shadow
  ...output omitted...
  user03:$6$CSsXsd3rwghsdfarf:17933:0:99999:7:2:18113:
  ```
  Each field of this code block is separated by a colon:
  * **user03** : Name of the user account.
  * **$6$CSsXsd3rwghsdfarf** : The encrypted password of the user.
  * **17933** : The days from the epoch when the password was last changed, where the epoch is 1970-01-01 in the UTC time zone.
  * **0** : The minimum days since the last password change before the user can change it again.
  * **99999** : The maximum days without a password change before the password expires. An empty field means that the password never expires.
  * **7** : The number of days ahead to warn the user that their password will expire.
  * **2** : The number of days without activity, starting with the day the password expired, before the account is automatically locked.
  * **18113** : The day when the account expires in days since the epoch. An empty field means that the account never expires.
  * The last field is typically empty and reserved for future use.

* The encrypted password field stores three pieces of information: the hashing algorithm in use, the *salt*, and the encrypted hash. Each piece of information is delimited by the dollar (```$```) character.
  ```$6$CSsXcYG1L/4ZfHr/$2W6evvJahUfzfHpc9X.45Jc6H30E```
  * **6** : The hashing algorithm in use for this password. A 6 indicates a SHA-512 hash, the RHEL 9 default, a 1 indicates MD5, and a 5 indicates SHA-256.
  * **CSsXcYG1L/4ZfHr/** : The salt in use to encrypt the password; originally chosen at random.
  * **2W6evvJahUfzfHpc9X.45Jc6H30E** : The encrypted hash of the user's password; combining the salt and the unencrypted password and then encrypting to generate the password hash.

<a name="chage"></a>
* The following example demonstrates the ```chage``` command to change the password policy of the sysadmin05 user. The command defines a minimum age (```-m```) of zero days, a maximum age (```-M```) of 90 days, a warning period (```-W```) of 7 days, and an inactivity period (```-I```) of 14 days.
  ```console
  [root@host ~]# chage -m 0 -M 90 -W 7 -I 14 sysadmin05
  ```
  ```console
  [root@host ~]# date +%F
  2022-03-10
  [root@host ~]# date -d "+30 days" +%F
  2022-04-09
  [root@host ~]# chage -E $(date -d "+30 days" +%F) cloudadmin10
  [root@host ~]# chage -l cloudadmin10 | grep "Account expires"
  Account expires						: Apr 09, 2022
  ```
  
