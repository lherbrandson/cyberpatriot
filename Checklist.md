CyberPatriot Linux Security Checklist

1. Preliminary Steps
Read ReadMe File: Identify critical instructions (allowed ports/users, services, etc.).
Answer Forensics Questions: Complete forensics questions first to avoid losing required data.
Document Changes: Log all actions in case of errors or rollback needs.

2. Update System
Apply Updates:
bash
Copy code
sudo apt-get update && sudo apt-get upgrade && sudo apt-get dist-upgrade
Set Automatic Updates:
GUI: Update Manager > Settings > Updates > Check for updates: Daily.
CLI:
bash
Copy code
sudo apt-get install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades

3. Secure Accounts
Disable Root Login (SSH):
bash
Copy code
sudo nano /etc/ssh/sshd_config
PermitRootLogin no
Then restart SSH:
bash
Copy code
sudo systemctl restart ssh
Delete Unauthorized Users:
bash
Copy code
sudo userdel -r <username>
sudo groupdel <groupname>
Check for Non-Root UID 0 Users:
bash
Copy code
awk -F: '($3 == "0" && $1 != "root") {print $1}' /etc/passwd
Remove any found users.
Secure User Groups:
Remove unauthorized users from sudo and admin groups:
bash
Copy code
sudo gpasswd -d <username> sudo

4. Enforce Password and Account Policies
Set Password Policies:
Edit /etc/login.defs:
Copy code
PASS_MAX_DAYS 90
PASS_MIN_DAYS 10
PASS_WARN_AGE 7
Configure PAM for complexity:
bash
Copy code
sudo apt-get install libpam-cracklib
sudo nano /etc/pam.d/common-password
Add:
makefile
Copy code
minlen=8 ucredit=-1 lcredit=-1 dcredit=-1 ocredit=-1 remember=5
Set Account Lockout Policy:
bash
Copy code
sudo nano /etc/pam.d/common-auth
auth required pam_tally2.so deny=5 onerr=fail unlock_time=1800

5. Configure Network Security
Enable Firewall:
bash
Copy code
sudo apt-get install ufw
sudo ufw enable
sudo ufw default deny incoming
sudo ufw default allow outgoing
Allow specific ports:
bash
Copy code
sudo ufw allow <port>
Disable IPv6:
bash
Copy code
echo "net.ipv6.conf.all.disable_ipv6 = 1" | sudo tee -a /etc/sysctl.conf
Prevent IP Spoofing:
bash
Copy code
echo "nospoof on" | sudo tee -a /etc/host.conf

6. Remove Unauthorized Tools and Files
Find and Remove Suspicious Files:
bash
Copy code
find / -type f \( -name "*.mp3" -o -name "*.mp4" -o -name "*.jpg" -o -name "*.zip" -o -name "*.tar.gz" \)
Remove Hacking Tools:
bash
Copy code
sudo apt-get purge nmap zenmap netcat ophcrack wireshark metasploit apache2 samba

7. Harden SSH
Restrict SSH Configuration:
Edit /etc/ssh/sshd_config:
perl
Copy code
PermitRootLogin no
PasswordAuthentication no
ChallengeResponseAuthentication no
PermitEmptyPasswords no
Restart SSH:
bash
Copy code
sudo systemctl restart ssh
Set SSH KeepAlive and Idle Timeout:
bash
Copy code
ClientAliveInterval 300
ClientAliveCountMax 0

8. Secure System Services
List Active Services:
bash
Copy code
service --status-all
Disable unnecessary services:
bash
Copy code
sudo systemctl disable <service>
Check Open Ports:
bash
Copy code
sudo ss -ln
Close unneeded ports using ufw or by disabling services.

9. Install and Run Security Tools
Antivirus:
bash
Copy code
sudo apt-get install clamav
sudo clamscan -r /
Rootkit Checkers:
bash
Copy code
sudo apt-get install chkrootkit rkhunter
sudo chkrootkit
sudo rkhunter --update
sudo rkhunter --check

10. File and Directory Security
Set Home Directory Permissions:
bash
Copy code
for user in $(awk -F: '($3 >= 1000) {print $1}' /etc/passwd); do chmod 750 /home/$user; done
Find World-Writable Files:
bash
Copy code
find / -xdev -type d \( -perm -0002 -a ! -perm -1000 \) -print
Find No-Owner Files:
bash
Copy code
find / -xdev \( -nouser -o -nogroup \) -print

11. Final Checks
Check Logs for Errors or Suspicious Activity:
bash
Copy code
sudo tail -f /var/log/auth.log
Validate Configuration Files:
bash
Copy code
sudo sshd -t
