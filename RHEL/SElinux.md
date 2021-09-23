# SElinux

## Table of Contents
[Providing more security forLinux](#Providing-more-security-for-Linux)

[Introducing Linux Security Modules(LSM)](#Introducing-Linux-Security-Modules(LSM))

[Extending regular DAC with SELinux](#Extending-regular-DAC-with-SELinux)

[Enabling SELinux support](#Enabling-SELinux-support)

[Labeling all resources and objects](#Labeling-all-resources-and-objects)

## Providing more security forLinux
Linux system are discretionary; it is up to theusers how the access controls should behave.
The Linux **discretionary access control (DAC)** mechanism is basedon the user and/or group information of the process and 
is matchedagainst the user and/or group information of the file, directory, orother resource being manipulated. Lots of software daemons run as the Linux root user or havesignificant privileges on the system. Errors within those daemonscan easily lead to information leakage or might even lead to remotelyexploitable vulnerabilities. Backup software, monitoring software,change management software, scheduling software, and so on: theyall often run with the highest 
privileged account possible on aregular Linux system.Enter SELinux, which provides an additional access control layer ontop of the standard Linux DAC mechanism. SELinux provides a **mandatory access control (MAC)** system that, unlike its DAC counterpart, 
gives the administrator full control over what is allowedon the system and what isn't.

## Introducing Linux Security Modules(LSM)
Mandatory access control systems such as SELinux are supported inthe Linux kernel through Linux Security Modules (LSM), a Linux subsystem called before processing a user space request (**system call**). Within the LSM framework, two types of security modules exist: **exclusive** and **non-exclusive** modules. Two exclusive modules cannot be active simultaneously.Non-exclusive modules can be combined.A major use case for stacking LSM modules is to enable different
security models within containers running on the system. Right now,it is not possible to implement a different security module within aLinux container, and the security within the container falls back tothe security module of the host.

## Extending regular DAC with SELinux
SELinux does not change the Linux DAC implementation, nor can itoverride denials made by the Linux DAC permissions. If a regularsystem (without SELinux) prevents a particular access, there isnothing SELinux can do to override this decision. This is because theLSM hooks are triggered after the regular DAC permission checksexecute, a conscious design decision from the LSM project.

## Enabling SELinux support
An SELinux implementation contains the following:The SELinux kernel subsystem, implemented in the Linux kernel through LSM Libraries, used by applications that need to interact with SELinux Utilities, used by administrators to interact with SELinux Policies, which define the access controls themselves.

## Labeling all resources and objects
When SELinux has to decide whether it has to allow or deny aparticular action, it makes a decision based on the context of both
the subject (who is initiating the action) and the object (which isthe target of the action). These contexts (or parts of the context) arementioned in the policy rules that SELinux enforces.The context of a process is what identifies the process to SELinux.SELinux has no notion of Linux process ownership and does not carehow the process is called, which process ID it has, and what accountthe process runs as. **All it wants to know is what the context of that process is, represented to users and administrators as a label. Label and context are often used interchangeably, and although there is a technical distinction (one is a representation of the other)**.

Let's look at an example label – the context of the current user:
```
$ id -Z
sysadm_u:sysadm_r:sysadm_t:s0-s0:c0.c1023
```
It shows us the context of thecurrent user.

## Dissecting the SELinux context
```
$ ps -eZ | grep sshd
system_u:system_r:sshd_t:s0-s0:c0.c1023 2629 ? 00:00:00sshd
```
As we can see, the process is assigned a context that contains the following fields: 
- The SELinux user system_u
- The SELinux role system_r
- The SELinux type (also known as the domain when we arelooking at a running process) sshd_t
- The sensitivity level s0-s0:c0.c1023
```
$ ls /proc/$$/attr
current  exec  fscreate  keycreate  prev  sockcreate
```
All these files, if read, display either nothing or an SELinux context.If it is empty, then that means the application has not explicitly set acontext for that particular purpose, and the SELinux context will bededuced either from the policy or inherited from its parent.The meaning of the files are as follows:
- The current file displays the current SELinux context of the process.
- The exec file displays the SELinux context that will be assigned by the next application execution done through this application. It is usually empty.
- The fscreate file displays the SELinux context that will beassigned to the next file written by the application. It is usually empty.
- The keycreate file displays the SELinux context that will beassigned to the keys cached in the kernel by this application. It isusually empty.
 - The prev file displays the previous SELinux context for thisparticular process. This is usually the context of its parentapplication.
 - The sockcreate file displays the SELinux context that will beassigned to the next socket created by the application. It isusually empty.
 
 If an application has multiple subtasks, then the same information isavailable in each subtask directory at/proc/<pid>/task/<taskid>/attr.

## Enforcing access through types
The SELinux type (the third part of an SELinux context) of a process (called
the domain) is the basis of the fine-grained access controls of that process
with respect to itself and other types. Take a look at the following dbus-daemon processes:
```
# ps -eZ | grep dbus-daemon
swift_u:swift_r:swift_dbusd_t:s0-s0:c0.c512 571 ?
00:00:01 dbus-daemon
swift_u:swift_r:swift_dbusd_t:s0-s0:c0.c512 649 ?
00:00:00 dbus-daemon
system_u:system_r:system_dbusd_t:s0-s0:c0.c1023
2498 ? 00:00:00 dbus-daemon
```
In this example, one dbus-daemon process is the system D-Bus daemon
running with the aptly named system_dbusd_t type, whereas two other
ones are running with the swift_dbusd_t type assigned to it. Even
though their binaries are the same, they both serve a different purpose on the
system and as such have a different type assigned. SELinux then uses this
type to govern the actions allowed by the process toward other types,
including how system_dbusd_t can interact with swift_dbusd_t.

