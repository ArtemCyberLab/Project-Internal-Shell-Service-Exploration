Project Description:
As part of this lab assignment, I conducted an exploration of the target machine (10.10.240.172) using Kali (10.10.228.53) as the attacking system.

Key steps of the project:

Enumeration with Nmap:

Performed a full TCP scan with service version detection:

nmap -p- -T4 10.10.240.172


The results showed the following open ports: 22 (SSH), 4420 (nvm-express), 8080 (HTTP Proxy).

FTP port 21 was filtered, and port 2375 (docker) was also closed/filtered.

Port Knocking:

Port 4420 was initially closed, so I performed port knocking:

knock 10.10.240.172 1111 2222 3333 4444
nmap -p 4420 10.10.240.172


After the correct sequence, port 4420 opened.

FTP and Clue Retrieval:

Connected to FTP as anonymous and downloaded note.txt:

ftp 10.10.240.172
get note.txt
exit
cat note.txt


The note contained the password for the INTERNAL SHELL SERVICE: sardinethecat.

Interacting with INTERNAL SHELL SERVICE:

Connected to port 4420 using netcat:

nc 10.10.240.172 4420


The service accepted the password, but in its restricted environment, standard Linux commands (chmod, cd, etc.) were not available, so it was not possible to generate or use an SSH key within nc.

Reasons why the project was not completed:

The target machine lacks the runme binary and the user account needed to generate the SSH key and retrieve the user flag.

Connecting via local root to the target machine does not allow creating SSH keys for remote access.

Due to these limitations, the full path to the user and root flags could not be completed.

What was accomplished and lessons learned:

Conducted a full Nmap scan with service version detection.

Implemented port knocking to open a closed port.

Used FTP to retrieve clues and analyze the service password.

Explored the restricted INTERNAL SHELL SERVICE and identified limitations and potential access methods using binaries.

Prepared the infrastructure for future SSH access (if the binary was available).


PART 2 
