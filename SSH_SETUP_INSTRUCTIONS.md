# 🔧 Step-by-Step SSH Setup Instructions

Complete step-by-step guide to set up passwordless SSH access and remote management for any Linux server.

## 📋 Prerequisites Checklist

### **Server Side (Target Machine):**
- [ ] Linux server running (Ubuntu/Debian/CentOS/etc.)
- [ ] SSH server installed and running
- [ ] User account with sudo privileges
- [ ] Server IP address or hostname known
- [ ] Network connectivity between client and server

### **Client Side (Your Main Computer):**
- [ ] SSH client installed (macOS/Linux have it built-in, Windows needs WSL or putty)
- [ ] Modern terminal application (Warp, iTerm2, Windows Terminal, etc.)
- [ ] Basic command line knowledge

## 🚀 Step-by-Step Setup Process

### **Step 1: Verify Server SSH Status**

**On the server, check SSH service:**
```bash
# Check if SSH is running
sudo systemctl status ssh

# If not running, install and start
sudo apt update
sudo apt install openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
```

**Get server information:**
```bash
# Note the server IP address
ip addr show | grep "inet " | grep -v "127.0.0.1"

# Note the username
whoami

# Note the hostname
hostname
```

**Expected output example:**
```
Server IP: 10.0.40.11
Username: jason
Hostname: myserver
```

---

### **Step 2: Test Initial Connection**

**From your main computer:**
```bash
# Replace with your actual server details
ssh username@server-ip
```

**Example:**
```bash
ssh jason@10.0.40.11
```

**What you'll see:**
1. **First connection warning** (normal):
   ```
   The authenticity of host '10.0.40.11 (10.0.40.11)' can't be established.
   ED25519 key fingerprint is SHA256:...
   Are you sure you want to continue connecting (yes/no/[fingerprint])?
   ```
   **Type:** `yes` and press Enter

2. **Password prompt:**
   ```
   username@server-ip's password:
   ```
   **Enter:** Your user password

3. **Successful login:**
   ```
   username@hostname:~$
   ```

**Verify you're connected:**
```bash
hostname && whoami
```

**Exit back to your main computer:**
```bash
exit
```

✅ **Step 2 Verification:** Basic SSH connection works with password

---

### **Step 3: Generate SSH Key Pair**

**On your main computer:**
```bash
# Check if SSH keys already exist
ls -la ~/.ssh/
```

**If you see `id_ed25519` and `id_ed25519.pub`, you already have keys. Skip to Step 4.**

**If no keys exist, generate new ones:**
```bash
ssh-keygen -t ed25519 -C "server-access-$(date +%Y%m%d)"
```

**During key generation:**
1. **File location prompt:**
   ```
   Enter file in which to save the key (/Users/username/.ssh/id_ed25519):
   ```
   **Action:** Press Enter (use default)

2. **Passphrase prompt:**
   ```
   Enter passphrase (empty for no passphrase):
   ```
   **Action:** Press Enter (no passphrase for convenience, or add one for extra security)

3. **Confirm passphrase:**
   ```
   Enter same passphrase again:
   ```
   **Action:** Press Enter

**Expected output:**
```
Your identification has been saved in /Users/username/.ssh/id_ed25519
Your public key has been saved in /Users/username/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:... server-access-20250623
```

**Verify key creation:**
```bash
ls -la ~/.ssh/
cat ~/.ssh/id_ed25519.pub
```

✅ **Step 3 Verification:** You see both private and public key files, and can view the public key content

---

### **Step 4: Copy Public Key to Server**

**From your main computer:**
```bash
ssh-copy-id username@server-ip
```

**Example:**
```bash
ssh-copy-id jason@10.0.40.11
```

**What you'll see:**
```
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/Users/username/.ssh/id_ed25519.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now, it is to install the new key(s)
username@server-ip's password:
```

**Enter your password when prompted.**

**Expected success output:**
```
Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'username@server-ip'"
and check to make sure that only the key(s) you wanted were added.
```

✅ **Step 4 Verification:** You see "Number of key(s) added: 1"

---

### **Step 5: Test Passwordless Authentication**

**From your main computer:**
```bash
ssh username@server-ip
```

**Expected behavior:**
- ✅ **No password prompt** - connects immediately
- ✅ **Successful login** to server
- ✅ **Server prompt** appears

**Verify connection:**
```bash
hostname && whoami && echo "Passwordless SSH working!"
```

**Exit back to main computer:**
```bash
exit
```

✅ **Step 5 Verification:** SSH connects without asking for password

---

### **Step 6: Create SSH Config File**

**On your main computer:**
```bash
# Create/edit SSH config
nano ~/.ssh/config
```

**Add this configuration (replace with your details):**
```
Host homelab
    HostName 10.0.40.11
    User jason
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    ServerAliveInterval 60
    ServerAliveCountMax 3
    Compression yes
    ForwardAgent yes
```

**For multiple servers, add additional blocks:**
```
Host server2
    HostName 192.168.1.100
    User admin
    Port 22
    IdentityFile ~/.ssh/id_ed25519

Host production
    HostName prod.example.com
    User deploy
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
```

**Save and exit:**
- **In nano:** Ctrl+X, then Y, then Enter
- **In vim:** :wq and Enter

**Set proper permissions:**
```bash
chmod 600 ~/.ssh/config
```

✅ **Step 6 Verification:** Config file created and permissions set

---

### **Step 7: Test SSH Config**

**From your main computer:**
```bash
# Test the new alias
ssh homelab
```

**Expected behavior:**
- ✅ **Connects immediately** without password
- ✅ **Uses the alias** instead of full address
- ✅ **Logs into server** successfully

**Test remote commands:**
```bash
# Exit first
exit

# Test single remote command
ssh homelab "hostname && uptime"
```

**Expected output:**
```
myserver
 23:45:01 up 15 days,  2:13,  1 user,  load average: 0.00, 0.00, 0.00
```

✅ **Step 7 Verification:** Both interactive login and remote commands work with alias

---

### **Step 8: Set Up Shell Aliases**

**Determine your shell:**
```bash
echo $SHELL
```

**Edit appropriate file:**
- **If `/bin/zsh`:** `nano ~/.zshrc`
- **If `/bin/bash`:** `nano ~/.bashrc` or `nano ~/.bash_profile`

**Add these aliases at the end:**
```bash
# SSH aliases for server management
alias server-ssh="ssh homelab"
alias server-status="ssh homelab 'docker-compose ps 2>/dev/null || systemctl status docker'"
alias server-update="ssh homelab 'sudo apt update && sudo apt upgrade'"
alias server-reboot="ssh homelab 'sudo reboot'"
alias server-logs="ssh homelab 'sudo journalctl --since today'"
alias server-tmux="ssh homelab -t 'tmux attach || tmux'"

# File management aliases
alias server-copy-to="scp"  # Usage: server-copy-to file.txt homelab:/path/
alias server-copy-from="scp"  # Usage: server-copy-from homelab:/path/file.txt .
```

**For homelab-specific aliases (if you have Docker Compose setup):**
```bash
# Homelab-specific aliases
alias homelab-ssh="ssh homelab"
alias homelab-status="ssh homelab 'docker-compose ps'"
alias homelab-update="ssh homelab 'homelab-update'"
alias homelab-sync="ssh homelab 'homelab-sync'"
alias homelab-logs="ssh homelab 'docker-compose logs --tail=50'"
alias homelab-tmux="ssh homelab -t 'tmux attach || tmux'"
```

**Save and reload shell:**
```bash
# For zsh
source ~/.zshrc

# For bash
source ~/.bashrc
# or
source ~/.bash_profile
```

✅ **Step 8 Verification:** Aliases are loaded and work

---

### **Step 9: Install tmux on Server**

**Connect to server:**
```bash
ssh homelab
```

**Install tmux:**
```bash
# Ubuntu/Debian
sudo apt update && sudo apt install tmux

# CentOS/RHEL
sudo yum install tmux
# or
sudo dnf install tmux

# Verify installation
tmux -V
```

**Exit server:**
```bash
exit
```

✅ **Step 9 Verification:** tmux is installed on server

---

### **Step 10: Test Complete Setup**

**Test all functionality:**

1. **Basic connection:**
   ```bash
   ssh homelab
   exit
   ```

2. **Remote commands:**
   ```bash
   ssh homelab "hostname && date"
   ```

3. **Aliases (if configured):**
   ```bash
   server-status
   ```

4. **Persistent session:**
   ```bash
   ssh homelab -t "tmux attach || tmux"
   # Inside tmux: Ctrl+B, D to detach
   # Reconnect: ssh homelab -t "tmux attach"
   ```

5. **File operations:**
   ```bash
   # Create test file
   echo "test" > testfile.txt
   
   # Copy to server
   scp testfile.txt homelab:/tmp/
   
   # Verify on server
   ssh homelab "cat /tmp/testfile.txt"
   
   # Copy back
   scp homelab:/tmp/testfile.txt ./testfile_from_server.txt
   
   # Clean up
   rm testfile.txt testfile_from_server.txt
   ssh homelab "rm /tmp/testfile.txt"
   ```

✅ **Step 10 Verification:** All functionality works correctly

---

## 🎯 Common Use Cases

### **Daily Server Management:**
```bash
# Quick health check
ssh homelab "uptime && df -h && free -h"

# System updates
ssh homelab "sudo apt update && sudo apt upgrade"

# Service management
ssh homelab "sudo systemctl status nginx"
ssh homelab "sudo systemctl restart apache2"

# Log monitoring
ssh homelab "sudo tail -f /var/log/syslog"
```

### **File Management:**
```bash
# Edit configuration files
ssh homelab "sudo nano /etc/nginx/nginx.conf"

# Copy configuration backups
scp homelab:/etc/nginx/nginx.conf ./nginx-backup.conf

# Sync directories
rsync -av ./local-configs/ homelab:/home/user/configs/
```

### **Development Workflow:**
```bash
# Deploy code
scp -r ./project/ homelab:/var/www/html/

# Run remote commands
ssh homelab "cd /var/www/html/project && npm install && npm start"

# Monitor application
ssh homelab -t "tmux attach -t app-session || tmux new -s app-session"
```

## 🔧 Troubleshooting Common Issues

### **"Permission denied (publickey)"**
```bash
# Check if SSH agent is running
ssh-add -l

# Add key to agent if needed
ssh-add ~/.ssh/id_ed25519

# Verify key on server
ssh homelab "cat ~/.ssh/authorized_keys"

# Check key permissions
ls -la ~/.ssh/
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

### **"Connection refused"**
```bash
# Check if SSH service is running on server
ssh homelab "sudo systemctl status ssh"

# Check firewall
ssh homelab "sudo ufw status"

# Test connectivity
ping server-ip
telnet server-ip 22
```

### **"Host key verification failed"**
```bash
# Remove old host key
ssh-keygen -R server-ip
ssh-keygen -R hostname

# Reconnect to accept new key
ssh homelab
```

### **tmux Issues**
```bash
# List existing sessions
ssh homelab "tmux list-sessions"

# Kill stuck session
ssh homelab "tmux kill-session -t session-name"

# Force kill all tmux sessions
ssh homelab "pkill -f tmux"
```

## 🛡️ Security Hardening

### **Server-Side Security:**
```bash
# Disable password authentication (after key setup works)
ssh homelab "sudo nano /etc/ssh/sshd_config"
# Set: PasswordAuthentication no
# Set: PubkeyAuthentication yes
# Set: PermitRootLogin no

# Restart SSH service
ssh homelab "sudo systemctl restart ssh"

# Enable firewall
ssh homelab "sudo ufw enable"
ssh homelab "sudo ufw allow ssh"
```

### **Client-Side Security:**
```bash
# Secure SSH directory permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
chmod 600 ~/.ssh/config
chmod 644 ~/.ssh/known_hosts

# Use SSH agent for key management
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

## 📚 Quick Reference Commands

### **Connection Commands:**
```bash
ssh homelab                    # Connect interactively
ssh homelab "command"          # Run single command
ssh homelab -t "command"       # Run command with terminal
ssh homelab -L 8080:localhost:80  # Local port forward
```

### **File Transfer:**
```bash
scp file.txt homelab:/path/           # Copy file to server
scp homelab:/path/file.txt .          # Copy file from server
scp -r dir/ homelab:/path/            # Copy directory
rsync -av dir/ homelab:/path/         # Sync directory
```

### **Session Management:**
```bash
ssh homelab -t "tmux attach || tmux"     # Persistent session
ssh homelab "tmux list-sessions"         # List sessions
ssh homelab "tmux kill-session -t name"  # Kill session
```

## 🎊 Setup Complete!

You now have:
- ✅ **Passwordless SSH access** with key authentication
- ✅ **Convenient aliases** for easy connection
- ✅ **Remote command execution** capabilities
- ✅ **Persistent sessions** with tmux
- ✅ **Secure configuration** with best practices
- ✅ **Professional workflow** for server management

Your remote server management is now fully automated and secure! 🚀
