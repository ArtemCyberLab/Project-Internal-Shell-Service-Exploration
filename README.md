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


PART 2 (KALI)
Goal:
The goal of this project is to perform reconnaissance, identify open services, and attempt initial access to the target machine 10.201.113.65 (attacker IP 10.201.3.32). The target hosts a phpBB forum and an internal shell service that requires port knocking.

Description of Actions & Code:

Nmap Full Port Scan & Service Detection

nmap -p- -sV 10.201.113.65


Detected services:

22/tcp SSH (OpenSSH 7.6p1 Ubuntu)

4420/tcp nvm-express (internal shell service)

8080/tcp HTTP (Apache 2.4.46 + PHP 7.3.27)

Other ports like 21/tcp FTP and 2375/tcp Docker were filtered.

Nikto Web Scan

nikto -host http://10.201.113.65:8080


Found phpBB forum on port 8080.

Headers missing X-Frame-Options, X-XSS-Protection, X-Content-Type-Options.

Found /download/ and /web.config which might be interesting.

Curl Initial HTML Grab

curl -sS http://10.201.113.65:8080 | sed -n '1,200p'


Retrieved main page of the Cat Pictures forum, confirming phpBB deployment.

Port Knocking Attempt (Outdated Approach)

for i in 1111 2222 3333 4444; do nmap -Pn -p $i --host-timeout 201 --max-retries 0 10.201.113.65; done


Attempted to open internal shell port 4420 via knocking sequence.

Ports 1111, 2222, 3333, 4444 reported as closed, indicating knockd or similar service may be required.

Knockd Installation Attempt

apt install knockd


Failed due to outdated repository (404 Not Found on Kali mirrors).

Reason: The project documentation was based on an older Kali version where knockd was available. Current Kali rolling repositories have updated mirrors, so the package cannot be installed directly.

Manual Port Knocking via Nmap

for p in 1111 2222 3333 4444; do nmap -Pn -p $p --max-retries 0 --host-timeout 201 cat.thm; done


Alternative approach to simulate knocking without knockd.

Verification of Internal Shell

nmap -p 4420 cat.thm


Confirmed port 4420 is open, indicating knocking sequence may have succeeded.

Limitations / Reason for Not Completing

Unable to install knockd due to outdated repositories.

Some scripts like runme do not work in the internal shell environment.

Further exploitation (FTP login, reverse shell, SSH key retrieval) could not be completed because the knocking or internal shell commands were partially non-functional in modern Kali.

Summary:
The project successfully identified the target services, performed initial reconnaissance, and attempted port knocking. The code and walkthrough from the original guide are partially outdated, mainly because:

knockd package is no longer available in the current Kali repository.

Internal shell scripts require a proper interactive shell which is not fully compatible with modern netcat/Nmap-only setups.

Next steps would include either manually performing port knocking via Nmap or setting up an older Kali VM that supports knockd, then proceeding with FTP, internal shell access, and SSH key retrieval.
