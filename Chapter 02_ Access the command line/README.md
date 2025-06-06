# Chapter 2: Access the command line
## Basic Command Syntax

| Main commands |
| --- |
| [whoami](#whoami) |
| [date](#date) |
| [password](#password) |
| [file](#file) |
| [cat](#cat) |
| [head](#head) |
| [tail](#tail) |
| [wc](#wc) |
| [\\](#\\) |
| [history](#history) |
| [Shortcuts](#Shortcuts) |

<a name="whoami"></a>
**Which user is running** 
```console
[user@host ~]$ whoami
user
```

**Typing more than one command in a single line**
```console
[user@host ~]$ command1 ; command2
command1 output
command2 output
```

<a name="date"></a>
**Get the current time**
```console
[user@host ~]$ date
Sun Feb 27 08:32:42 PM EST 2022
[user@host ~]$ date +%R
20:33
[user@host ~]$ date +%x
02/27/2022
```

<a name="password"></a>
**Change current user password**
```console
[user@host ~]$ passwd
Changing password for user user.
Current password: old_password
New password: new_password
Retype new password: new_password
passwd: all authentication tokens updated successfully.
```

<a name="file"></a>
**Display the actual type of a file and its name**
```console
[user@host ~]$ file /etc/passwd
/etc/passwd: ASCII text
[user@host ~]$ file /bin/passwd
/bin/passwd: setuid ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=a467cb9c8fa7306d41b96a820b0178f3a9c66055, for GNU/Linux 3.2.0, stripped
[user@host ~]$ file /home
/home: directory
```

<a name="cat"></a>
**View the content of a file**
```console
[user@host ~]$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
...output omitted...
```
it also can be used on multiple files
```console
[user@host ~]$ cat file1 file2
Hello World!!
Introduction to Linux commands.
```

<a name="head"></a>
<a name="tail"></a>
View first or last ~n (default:10) lines of a file 
```console
[user@host ~]$ head /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
```
```console
[user@host ~]$ tail -n 3 /etc/passwd
gdm:x:42:42::/var/lib/gdm:/sbin/nologin
gnome-initial-setup:x:980:978::/run/gnome-initial-setup/:/sbin/nologin
dnsmasq:x:979:977:Dnsmasq DHCP and DNS server:/var/lib/dnsmasq:/sbin/nologin
```

<a name="wc"></a>
**Counts lines, words, and characters in a file.** Use the -l, -w, or -c options to display only the given number of lines, words, or characters, respectively.
```console
[user@host ~]$ wc /etc/passwd
41   98 2338 /etc/passwd
[user@host ~]$ wc -l /etc/passwd ; wc -l /etc/group
41 /etc/passwd
63 /etc/group
[user@host ~]$ wc -c /etc/group /etc/hosts
883 /etc/group
114 /etc/hosts
997 total
```

<a name="\"></a>
**Write a Long Command on Multiple Lines**
```console
[user@host ~]$ head -n 3 \
/usr/share/dict/words \
/usr/share/dict/linux.words
==> /usr/share/dict/words <==
1080
10-point
10th

==> /usr/share/dict/linux.words <==
1080
10-point
10th
```

<a name="history"></a>
**Display the Command History**
```console
[user@host ~]$ history
...output omitted...
   23  clear
   24  who
   25  pwd
   26  ls /etc
   27  uptime
   28  ls -l
   29  date
   30  history
[user@host ~]$ !ls
ls -l
total 0
drwxr-xr-x. 2 student student 6 Feb 27 19:24 Desktop
...output omitted...
[user@host ~]$ !26
ls /etc
abrt                     hosts                     pulse
adjtime                  hosts.allow               purple
aliases                  hosts.deny                qemu-ga
...output omitted...
```
The exclamation point character (!) is a metacharacter to expand previous commands without retyping them. The !number command expands to the command that matches the specified number. The !string command expands to the most recent command that begins with the specified string.

<a name="Shortcuts"></a>
**Useful Command-line Editing Shortcuts**
| Shortcut | Description |
| :---: | --- |
| Ctrl+A	| Jump to the beginning of the command line. |
| Ctrl+E	| Jump to the end of the command line. |
| Ctrl+U	| Clear from the cursor to the beginning of the command line. |
| Ctrl+K	| Clear from the cursor to the end of the command line. |
| Ctrl+LeftArrow	| Jump to the beginning of the previous word on the command line. |
| Ctrl+RightArrow	| Jump to the end of the next word on the command line. |
| Ctrl+R	| Search the history list of commands for a pattern. |
