# SElinux

## Providing more security forLinux
Linux system are discretionary; it is up to theusers how the access controls should behave.
The Linux discretionary access control (DAC) mechanism is basedon the user and/or group information of the process and 
is matchedagainst the user and/or group information of the file, directory, orother resource being manipulated. Lots of software daemons run as the Linux root user or havesignificant privileges on the system. Errors within those daemonscan easily lead to information leakage or might even lead to remotelyexploitable vulnerabilities. Backup software, monitoring software,change management software, scheduling software, and so on: theyall often run with the highest 
privileged account possible on aregular Linux system.Enter SELinux, which provides an additional access control layer ontop of the standard Linux DAC mechanism. SELinux provides amandatory access control (MAC) system that, unlike its DACcounterpart, 
gives the administrator full control over what is allowedon the system and what isn't
