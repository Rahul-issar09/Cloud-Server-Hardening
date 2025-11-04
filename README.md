# Cloud-Server-Hardening
Cloud Server Hardening &amp; Secure Access (Ubuntu, UFW, Fail2ban, Auditd)

# 🔐 Project 2: Cloud Server Hardening & Secure Access

## 🧭 Objective
The objective of this project is to deploy and harden a cloud-based Ubuntu server following industry best practices.  
The goal is to **limit SSH access**, **create IAM-like user roles**, **apply strict firewall rules**, and **log and audit all access attempts**.

---

## ☁️ Environment Setup

| Component | Details |
|------------|----------|
| **Cloud Platform** | Google Cloud (Free Trial) |
| **Instance Type** | e2-micro (Always Free) |
| **Operating System** | Ubuntu 22.04 LTS |
| **Tools Used** | `ufw`, `fail2ban`, `auditd` |
| **SSH Client** | Windows PowerShell |
| **User Created** | `adminuser` |

---

## ⚙️ Step-by-Step Implementation

 1️⃣ System Update
 
sudo apt update && sudo apt upgrade -y
sudo apt install vim curl unzip htop -y

2️⃣ Create IAM-like User

sudo adduser adminuser
sudo usermod -aG sudo adminuser
echo "adminuser ALL=(ALL) ALL" | sudo tee /etc/sudoers.d/adminuser
sudo chmod 440 /etc/sudoers.d/adminuser

✅ Result:
Created a new user adminuser with administrative (sudo) privileges using a secure sudoers file.

3️⃣ Enable SSH Key Authentication

SSH key pair generated locally on Windows:

ssh-keygen -t ed25519 -C "rahul@project2" -f C:\Users\rahul\.ssh\project2_key

Public key added to VM:

sudo mkdir -p /home/adminuser/.ssh
echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKHH8UT2rlfkvb5zK4h0Ww/9T2pI5oJZPr3ynuvnWaXi rahul@project2" | sudo tee /home/adminuser/.ssh/authorized_keys
sudo chown -R adminuser:adminuser /home/adminuser/.ssh
sudo chmod 700 /home/adminuser/.ssh
sudo chmod 600 /home/adminuser/.ssh/authorized_keys

✅ Result:
Key-based authentication configured successfully for adminuser.

4️⃣ Disable Root Login and Password Authentication

sudo sed -i 's/^#\?PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
sudo sed -i 's/^#\?PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo systemctl restart ssh

✅ Result:
Root login and password authentication are disabled, enforcing key-only access.

5️⃣ Configure UFW Firewall

sudo apt install ufw -y
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow OpenSSH
sudo ufw enable
sudo ufw status verbose

✅ Result:
All inbound traffic is denied except for SSH on port 22.
Firewall logging is enabled for monitoring.

6️⃣ Configure Fail2ban

sudo apt install fail2ban -y
sudo systemctl enable --now fail2ban
sudo tee /etc/fail2ban/jail.d/sshd.local > /dev/null <<'EOF'
[sshd]
enabled = true
port = ssh
logpath = /var/log/auth.log
maxretry = 3
bantime = 600
EOF
sudo systemctl restart fail2ban
sudo fail2ban-client status sshd

✅ Result:
Fail2ban is actively monitoring SSH authentication logs and bans IPs after 3 failed attempts.

7️⃣ Enable Auditing with auditd

sudo apt install auditd audispd-plugins -y
sudo systemctl enable --now auditd
sudo tee /etc/audit/rules.d/50-project.rules > /dev/null <<'EOF'
-w /var/log/auth.log -p wa -k logins
-w /etc/sudoers -p wa -k sudo_config
-w /etc/sudoers.d/ -p wa -k sudo_config
-a always,exit -F arch=b64 -S execve -F euid=0 -k sudo_cmds
EOF
sudo augenrules --load
sudo systemctl restart auditd

✅ Result:
Auditd is configured to monitor login events, sudo command executions, and configuration changes.


