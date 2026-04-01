# 🤠 TryHackMe: Bounty Hunter Writeup
> **Target OS:** Linux | **Difficulty:** Easy | **Category:** FTP Enumeration / SSH Brute-force / Sudo Exploitation

---

## 🔍 1. Reconnaissance

The engagement started with an **Nmap** scan to identify open ports and services. The scan revealed three open ports: **21 (FTP)**, **22 (SSH)**, and **80 (HTTP)**.

![Nmap Scan](1.png)

### Directory Discovery
I used **Gobuster** to enumerate hidden directories on the web server. While it found `/images` and `/javascript`, these didn't provide an immediate path for exploitation, leading me to focus on other services.

![Gobuster Scan](2.png)

---

## 📂 2. FTP Enumeration

The reconnaissance phase indicated that **anonymous login** was enabled on the FTP server. I connected using the username `anonymous` and a blank password.

![FTP Login](3.png)

### Sensitive File Leakage
Upon listing the files, I discovered two interesting text files: `task.txt` and `locks.txt`. I downloaded them to my local machine using the `get` command for analysis.

* **task.txt:** Contained a developer note mentioning a potential system user: `lin`.
* **locks.txt:** Contained a list of strings that appeared to be a custom password wordlist.

![task.txt](4.png)
![locks.txt](5.png)

---

## 🔑 3. Initial Access (SSH)

Equipped with a username (`lin`) and a targeted wordlist (`locks.txt`), I performed a brute-force attack on the **SSH** service using **Hydra**.

```bash
hydra -l lin -P locks.txt ssh://<TARGET_IP> -t 4
Hydra successfully identified a valid password: RedDr4g0nSynd1cat3.

Foothold
I logged into the machine via SSH using the recovered credentials.

After gaining access, I located the first flag in the Desktop directory.

User Flag: THM{CR1M3_SyNd1C4T3}

⚡ 4. Privilege Escalation
To identify a path to root, I checked the sudo permissions for the user lin:

Bash
sudo -l
The output confirmed that lin could run the /bin/tar binary as root without a password.

Tar Exploitation (GTFOBins)
Following the GTFOBins methodology, I abused the --checkpoint feature of the tar utility to execute a shell. I ran the following command:

Bash
sudo /bin/tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
This successfully spawned a root shell. I then navigated to the /root directory to claim the final flag.

Root Flag: THM{80UN7Y_h4cK3r}

🏁 Summary
The Bounty Hunter room was a great exercise in:

Anonymous FTP Access: Identifying leaked sensitive information.

Custom Wordlists: Using found data for successful brute-forcing.

Sudo Misconfiguration: Exploiting tar via checkpoint actions for privilege escalation.