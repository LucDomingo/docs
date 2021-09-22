# SElinux

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

