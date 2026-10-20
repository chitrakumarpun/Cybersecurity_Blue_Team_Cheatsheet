# Linux Basics for Blue Team / SOC Analysts
## Identity and current location

### Show current user
```bash
whoami
```
**Use Case:**Confirm which account is running commands, especially before changing files or collecting evidence.

### Show current and past login identity
```bash
id
```

**Use case:** Check the user's UID, GID, and group memberships — useful when verifying if an account has unexpected privileges (e.g. added to `sudo` or `wheel`).

### Show present working directory

```bash
pwd
```

**Use case:** Confirm your location in the filesystem before running destructive or investigative commands, especially during live triage.

### List logged-in users

```bash
who
```

**Use case:** Identify who is currently logged into the system — important when checking for unauthorized or simultaneous sessions during an incident.

### Show detailed logged-in user activity

```bash
w
```


## System and process information

### Show running processes

```bash
ps aux
```

**Use case:** Get a snapshot of all running processes — a first step in spotting unfamiliar or malicious processes.

### Show real-time process activity

```bash
top
```

**Use case:** Monitor CPU/memory usage live — useful for catching cryptominers or processes consuming abnormal resources.

### Show real-time process activity (enhanced)

```bash
htop
```

**Use case:** Same as `top` but with a more readable, interactive interface — handy for quickly sorting and killing suspicious processes during triage.

### Show process tree

```bash
pstree -p
```

**Use case:** Visualize parent-child process relationships — useful for tracing how a suspicious process was spawned (e.g. a shell spawned from a web server process).

### Kill a process by PID

```bash
kill -9 <PID>
```

**Use case:** Terminate a confirmed malicious or hung process during incident response.

### Show system uptime

```bash
uptime
```

**Use case:** Check how long the system has been running — an unexpected reboot can indicate tampering or crash from an exploit attempt.

### Show kernel and OS version

```bash
uname -a
```

**Use case:** Identify kernel version to check against known CVEs during vulnerability assessment.

### Show OS release details

```bash
cat /etc/os-release
```

**Use case:** Confirm the exact Linux distribution and version for patch management or vulnerability correlation.

---

## File and directory operations

### List files with details

```bash
ls -la
```

**Use case:** View hidden files, permissions, ownership, and timestamps — critical when hunting for planted malware or modified configs.

### Change directory

```bash
cd /path/to/directory
```

**Use case:** Navigate to directories of interest such as log folders (`/var/log`) during an investigation.

### Find files by name

```bash
find / -name "filename"
```

**Use case:** Locate a specific file across the filesystem, such as a known malicious filename from threat intel.

### Find recently modified files

```bash
find / -mmin -60 -type f
```

**Use case:** Identify files changed in the last 60 minutes — useful for spotting recent tampering after a suspected breach.

### Find files by permission (e.g. world-writable)

```bash
find / -perm -o+w -type f
```

**Use case:** Identify insecure, world-writable files that an attacker could exploit for persistence.

### Find SUID/SGID binaries

```bash
find / -perm -4000 -o -perm -2000 -type f 2>/dev/null
```

**Use case:** Hunt for unauthorized SUID/SGID binaries, a common privilege escalation vector.

### View file contents

```bash
cat filename
```

**Use case:** Quickly inspect a file's contents, such as a config file or short log.

### View file contents page by page

```bash
less filename
```

**Use case:** Review large files (like logs) without flooding the terminal, with search capability.

### Show first lines of a file

```bash
head -n 50 filename
```

**Use case:** Quickly check the start of a large log file for headers or early entries.

### Show last lines of a file

```bash
tail -n 50 filename
```

**Use case:** Check the most recent log entries — often where the relevant activity is during active monitoring.

### Follow a file in real time

```bash
tail -f /var/log/auth.log
```

**Use case:** Monitor authentication logs live during an active incident or suspicious login attempt.

### Copy a file

```bash
cp source destination
```

**Use case:** Preserve a copy of evidence (e.g. a suspicious binary or log) before further analysis.

### Move or rename a file

```bash
mv source destination
```

**Use case:** Quarantine a suspicious file by moving it to an isolated directory.

### Remove a file

```bash
rm filename
```

**Use case:** Delete confirmed malicious files after proper evidence collection (use cautiously and only after preserving evidence).

### Check file type

```bash
file filename
```

**Use case:** Determine the true type of a file regardless of its extension — useful for spotting disguised executables.

### Calculate file hash (SHA256)

```bash
sha256sum filename
```

**Use case:** Generate a hash to compare against known malware hashes (e.g. via VirusTotal) or to verify file integrity.

### Calculate file hash (MD5)

```bash
md5sum filename
```

**Use case:** Quick hash comparison against threat intel feeds, though SHA256 is preferred for integrity verification.

---

## Permissions and ownership

### View file permissions

```bash
ls -l filename
```

**Use case:** Check if a file has unusually permissive access (e.g. world-writable) that could indicate tampering.

### Change file permissions

```bash
chmod 644 filename
```

**Use case:** Lock down a file's permissions after discovering it was modified insecurely.

### Change file ownership

```bash
chown user:group filename
```

**Use case:** Restore correct ownership on a file that was altered during a compromise.

---

## Networking

### Show network interfaces and IPs

```bash
ip a
```

**Use case:** Confirm the host's network configuration, useful baseline info during an investigation.

### Show active network connections

```bash
netstat -tulnp
```

**Use case:** Identify listening ports and established connections — key for spotting unauthorized services or reverse shells.

### Show active network connections (modern)

```bash
ss -tulnp
```

**Use case:** Faster, modern alternative to `netstat` for the same purpose — checking for suspicious open ports or connections.

### Show routing table

```bash
ip route
```

**Use case:** Verify routing configuration hasn't been tampered with to redirect traffic.

### Test connectivity to a host

```bash
ping -c 4 8.8.8.8
```

**Use case:** Confirm network connectivity or test if a suspicious external IP is reachable.

### Trace network path to a host

```bash
traceroute 8.8.8.8
```

**Use case:** Map the network path to an external IP, useful when investigating C2 (command-and-control) communication.

### Perform DNS lookup

```bash
dig domain.com
```

**Use case:** Investigate a suspicious domain's DNS records, which can reveal infrastructure tied to an attacker.

### Reverse DNS/simple domain lookup

```bash
nslookup domain.com
```

**Use case:** Quickly check what IP a suspicious domain resolves to, or what domain an IP maps back to.

### Download a file from a URL

```bash
curl -O http://example.com/file
```

**Use case:** Retrieve a suspicious file from a URL found in logs for safe analysis in an isolated environment (use a sandboxed VM, never production).

### Capture network packets

```bash
tcpdump -i eth0 -w capture.pcap
```

**Use case:** Capture live traffic on an interface for later analysis in Wireshark during an active incident.

---

## Logs and auditing

### View system log (systemd)

```bash
journalctl -xe
```

**Use case:** Review recent system logs with explanations, useful for triaging errors and security-relevant events.

### View authentication log

```bash
cat /var/log/auth.log
```

**Use case:** Review SSH logins, sudo usage, and authentication failures (Debian/Ubuntu systems).

### View secure log (RHEL/CentOS equivalent)

```bash
cat /var/log/secure
```

**Use case:** Same purpose as `auth.log` but for RHEL-based systems — check for brute-force or unauthorized access attempts.

### Search logs for a keyword

```bash
grep "Failed password" /var/log/auth.log
```

**Use case:** Quickly filter logs for specific indicators like failed login attempts, a common sign of brute-forcing.

### Search recursively across files

```bash
grep -r "malicious_string" /var/log/
```

**Use case:** Hunt for a known indicator of compromise (IOC) across all log files in a directory.

### Show command history

```bash
history
```

**Use case:** Review commands run by the current shell session — useful for seeing what an attacker (or user) executed if a shell was compromised.

---

## Scheduled tasks and persistence checks

### List current user's cron jobs

```bash
crontab -l
```

**Use case:** Check for unauthorized scheduled tasks, a common persistence mechanism for attackers.

### List all users' cron jobs

```bash
ls /etc/cron.d/ /etc/cron.daily/ /etc/cron.hourly/
```

**Use case:** Audit system-wide scheduled jobs for malicious entries placed outside a single user's crontab.

### List systemd services

```bash
systemctl list-units --type=service
```

**Use case:** Identify unfamiliar or newly created services that could indicate attacker-installed persistence.

### Check for unusual startup services

```bash
systemctl list-unit-files --state=enabled
```

**Use case:** Spot services configured to auto-start at boot that shouldn't be there, another common persistence technique.

---

## User and account management

### List all user accounts

```bash
cat /etc/passwd
```

**Use case:** Check for unauthorized or unexpected user accounts created by an attacker.

### List all groups

```bash
cat /etc/group
```

**Use case:** Verify group memberships haven't been altered to grant unauthorized privileges.

### Show sudo privileges configuration

```bash
cat /etc/sudoers
```

**Use case:** Confirm no unauthorized entries were added granting a user elevated privileges.

### Check currently installed packages

```bash
dpkg -l
```

**Use case:** Review installed software for anything unexpected or not sanctioned (Debian/Ubuntu).

## Blue-team reminder
Use commands only on systems and data you own or are explicitly authorized to investigate.
