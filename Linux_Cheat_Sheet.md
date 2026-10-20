# Linux Basics for Blue Team / SOC Analysts

## Identity and current location

### Show current user

```bash
whoami
```

**Dissection:**
| Part | Meaning |
|---|---|
| `whoami` | Prints the username of the currently effective user |

**Use case:** Confirm which account is running commands, especially before changing files or collecting evidence.

### Show current and past login identity

```bash
id
```

**Dissection:**
| Part | Meaning |
|---|---|
| `id` | Prints real/effective UID, GID, and all group memberships for the current user |

**Use case:** Check the user's UID, GID, and group memberships — useful when verifying if an account has unexpected privileges (e.g. added to `sudo` or `wheel`).

### Show present working directory

```bash
pwd
```

**Dissection:**
| Part | Meaning |
|---|---|
| `pwd` | Print working directory — prints the full path of the current directory |

**Use case:** Confirm your location in the filesystem before running destructive or investigative commands, especially during live triage.

### List logged-in users

```bash
who
```

**Dissection:**
| Part | Meaning |
|---|---|
| `who` | Lists users currently logged in, their terminal, and login time |

**Use case:** Identify who is currently logged into the system — important when checking for unauthorized or simultaneous sessions during an incident.

### Show detailed logged-in user activity

```bash
w
```

**Dissection:**
| Part | Meaning |
|---|---|
| `w` | Shows logged-in users plus what command each is currently running and idle time |

**Use case:** See what each logged-in user is currently doing (their running process and idle time) — helps spot suspicious active sessions.

### Show last logins

```bash
last
```

**Dissection:**
| Part | Meaning |
|---|---|
| `last` | Reads `/var/log/wtmp` to show login history, including reboots |

**Use case:** Review recent login history (including reboots) to check for logins at unusual times or from unexpected sources.

### Show failed login attempts

```bash
lastb
```

**Dissection:**
| Part | Meaning |
|---|---|
| `lastb` | Reads `/var/log/btmp` to show failed login attempts |

**Use case:** Identify brute-force attempts or failed authentication, a common precursor to account compromise.

---

## System and process information

### Show running processes

```bash
ps aux
```

**Dissection:**
| Part | Meaning |
|---|---|
| `ps` | Process status — reports a snapshot of currently running processes |
| `a` | Show processes for all users (not just the current one) |
| `u` | Display in user-oriented format (user, %CPU, %MEM, etc.) |
| `x` | Include processes without a controlling terminal (daemons, services) |

**Use case:** Get a snapshot of all running processes — a first step in spotting unfamiliar or malicious processes.

### Show real-time process activity

```bash
top
```

**Dissection:**
| Part | Meaning |
|---|---|
| `top` | Launches an interactive, live-updating process viewer sorted by CPU usage by default |

**Use case:** Monitor CPU/memory usage live — useful for catching cryptominers or processes consuming abnormal resources.

### Show real-time process activity (enhanced)

```bash
htop
```

**Dissection:**
| Part | Meaning |
|---|---|
| `htop` | Interactive process viewer (third-party) with color output, scrolling, and mouse/sort support |

**Use case:** Same as `top` but with a more readable, interactive interface — handy for quickly sorting and killing suspicious processes during triage.

### Show process tree

```bash
pstree -p
```

**Dissection:**
| Part | Meaning |
|---|---|
| `pstree` | Displays running processes as a tree showing parent-child relationships |
| `-p` | Show each process's PID alongside its name |

**Use case:** Visualize parent-child process relationships — useful for tracing how a suspicious process was spawned (e.g. a shell spawned from a web server process).

### Kill a process by PID

```bash
kill -9 <PID>
```

**Dissection:**
| Part | Meaning |
|---|---|
| `kill` | Sends a signal to a process (default signal: terminate) |
| `-9` | Sends SIGKILL — forces immediate termination, bypassing cleanup |
| `<PID>` | The target process ID to kill |

**Use case:** Terminate a confirmed malicious or hung process during incident response.

### Show system uptime

```bash
uptime
```

**Dissection:**
| Part | Meaning |
|---|---|
| `uptime` | Shows current time, how long the system has run, logged-in user count, and load averages |

**Use case:** Check how long the system has been running — an unexpected reboot can indicate tampering or crash from an exploit attempt.

### Show kernel and OS version

```bash
uname -a
```

**Dissection:**
| Part | Meaning |
|---|---|
| `uname` | Prints system information |
| `-a` | Show all available info: kernel name, hostname, kernel release/version, architecture |

**Use case:** Identify kernel version to check against known CVEs during vulnerability assessment.

### Show OS release details

```bash
cat /etc/os-release
```

**Dissection:**
| Part | Meaning |
|---|---|
| `cat` | Prints file contents to the terminal |
| `/etc/os-release` | Standard file holding distro name, version, and identifiers |

**Use case:** Confirm the exact Linux distribution and version for patch management or vulnerability correlation.

---

## File and directory operations

### List files with details

```bash
ls -la
```

**Dissection:**
| Part | Meaning |
|---|---|
| `ls` | Lists directory contents |
| `-l` | Long format: permissions, owner, group, size, modified date |
| `-a` | Include hidden files (dotfiles) |

**Use case:** View hidden files, permissions, ownership, and timestamps — critical when hunting for planted malware or modified configs.

### Change directory

```bash
cd /path/to/directory
```

**Dissection:**
| Part | Meaning |
|---|---|
| `cd` | Change directory — built-in shell command |
| `/path/to/directory` | Target path to move into |

**Use case:** Navigate to directories of interest such as log folders (`/var/log`) during an investigation.

### Find files by name

```bash
find / -name "filename"
```

**Dissection:**
| Part | Meaning |
|---|---|
| `find` | Searches a directory tree for files matching criteria |
| `/` | Starting point of the search — the whole filesystem |
| `-name "filename"` | Match files by exact (or wildcard) name |

**Use case:** Locate a specific file across the filesystem, such as a known malicious filename from threat intel.

### Find recently modified files

```bash
find / -mmin -60 -type f
```

**Dissection:**
| Part | Meaning |
|---|---|
| `find /` | Search the entire filesystem |
| `-mmin -60` | Modified within the last 60 minutes |
| `-type f` | Restrict results to regular files (not directories) |

**Use case:** Identify files changed in the last 60 minutes — useful for spotting recent tampering after a suspected breach.

### Find files by permission (e.g. world-writable)

```bash
find / -perm -o+w -type f
```

**Dissection:**
| Part | Meaning |
|---|---|
| `find /` | Search the entire filesystem |
| `-perm -o+w` | Match files where the "other" permission bit includes write access |
| `-type f` | Restrict to regular files |

**Use case:** Identify insecure, world-writable files that an attacker could exploit for persistence.

### Find SUID/SGID binaries

```bash
find / -perm -4000 -o -perm -2000 -type f 2>/dev/null
```

**Dissection:**
| Part | Meaning |
|---|---|
| `-perm -4000` | Match files with the SUID bit set (runs as file owner, often root) |
| `-o` | Logical OR between the two `-perm` conditions |
| `-perm -2000` | Match files with the SGID bit set (runs as file's group) |
| `-type f` | Restrict to regular files |
| `2>/dev/null` | Discard "permission denied" error output |

**Use case:** Hunt for unauthorized SUID/SGID binaries, a common privilege escalation vector.

### View file contents

```bash
cat filename
```

**Dissection:**
| Part | Meaning |
|---|---|
| `cat` | Concatenate and print file contents to standard output |
| `filename` | Target file |

**Use case:** Quickly inspect a file's contents, such as a config file or short log.

### View file contents page by page

```bash
less filename
```

**Dissection:**
| Part | Meaning |
|---|---|
| `less` | Opens a file in a scrollable, searchable pager |
| `filename` | Target file |

**Use case:** Review large files (like logs) without flooding the terminal, with search capability.

### Show first lines of a file

```bash
head -n 50 filename
```

**Dissection:**
| Part | Meaning |
|---|---|
| `head` | Prints the beginning of a file |
| `-n 50` | Limit output to the first 50 lines |
| `filename` | Target file |

**Use case:** Quickly check the start of a large log file for headers or early entries.

### Show last lines of a file

```bash
tail -n 50 filename
```

**Dissection:**
| Part | Meaning |
|---|---|
| `tail` | Prints the end of a file |
| `-n 50` | Limit output to the last 50 lines |
| `filename` | Target file |

**Use case:** Check the most recent log entries — often where the relevant activity is during active monitoring.

### Follow a file in real time

```bash
tail -f /var/log/auth.log
```

**Dissection:**
| Part | Meaning |
|---|---|
| `tail` | Prints the end of a file |
| `-f` | Follow mode — keeps streaming new lines as they're appended |
| `/var/log/auth.log` | Target log file |

**Use case:** Monitor authentication logs live during an active incident or suspicious login attempt.

### Copy a file

```bash
cp source destination
```

**Dissection:**
| Part | Meaning |
|---|---|
| `cp` | Copies files or directories |
| `source` | File to copy |
| `destination` | Target location/name |

**Use case:** Preserve a copy of evidence (e.g. a suspicious binary or log) before further analysis.

### Move or rename a file

```bash
mv source destination
```

**Dissection:**
| Part | Meaning |
|---|---|
| `mv` | Moves or renames files/directories |
| `source` | Original file |
| `destination` | New location or name |

**Use case:** Quarantine a suspicious file by moving it to an isolated directory.

### Remove a file

```bash
rm filename
```

**Dissection:**
| Part | Meaning |
|---|---|
| `rm` | Removes (deletes) files |
| `filename` | Target file to delete |

**Use case:** Delete confirmed malicious files after proper evidence collection (use cautiously and only after preserving evidence).

### Check file type

```bash
file filename
```

**Dissection:**
| Part | Meaning |
|---|---|
| `file` | Determines file type by inspecting content/magic bytes, not just extension |
| `filename` | Target file |

**Use case:** Determine the true type of a file regardless of its extension — useful for spotting disguised executables.

### Calculate file hash (SHA256)

```bash
sha256sum filename
```

**Dissection:**
| Part | Meaning |
|---|---|
| `sha256sum` | Computes the SHA-256 cryptographic hash of a file |
| `filename` | Target file |

**Use case:** Generate a hash to compare against known malware hashes (e.g. via VirusTotal) or to verify file integrity.

### Calculate file hash (MD5)

```bash
md5sum filename
```

**Dissection:**
| Part | Meaning |
|---|---|
| `md5sum` | Computes the MD5 hash of a file (faster, but weaker than SHA256) |
| `filename` | Target file |

**Use case:** Quick hash comparison against threat intel feeds, though SHA256 is preferred for integrity verification.

---

## Permissions and ownership

### View file permissions

```bash
ls -l filename
```

**Dissection:**
| Part | Meaning |
|---|---|
| `ls` | Lists directory/file info |
| `-l` | Long format showing permissions, owner, group, size, and date |
| `filename` | Target file |

**Use case:** Check if a file has unusually permissive access (e.g. world-writable) that could indicate tampering.

### Change file permissions

```bash
chmod 644 filename
```

**Dissection:**
| Part | Meaning |
|---|---|
| `chmod` | Changes file mode (permission) bits |
| `644` | Octal permission: owner read/write (6), group read-only (4), others read-only (4) |
| `filename` | Target file |

**Use case:** Lock down a file's permissions after discovering it was modified insecurely.

### Change file ownership

```bash
chown user:group filename
```

**Dissection:**
| Part | Meaning |
|---|---|
| `chown` | Changes file owner and/or group |
| `user:group` | New owner and group to assign |
| `filename` | Target file |

**Use case:** Restore correct ownership on a file that was altered during a compromise.

---

## Networking

### Show network interfaces and IPs

```bash
ip a
```

**Dissection:**
| Part | Meaning |
|---|---|
| `ip` | Modern tool for network configuration and info |
| `a` | Shorthand for `address` — show interfaces and their assigned IPs |

**Use case:** Confirm the host's network configuration, useful baseline info during an investigation.

### Show active network connections

```bash
netstat -tulnp
```

**Dissection:**
| Part | Meaning |
|---|---|
| `netstat` | Displays network connections, routing tables, and socket stats |
| `-t` | Show TCP connections |
| `-u` | Show UDP connections |
| `-l` | Show only listening sockets |
| `-n` | Show addresses/ports numerically (skip DNS/service-name resolution) |
| `-p` | Show the PID/process name owning each socket |

**Use case:** Identify listening ports and established connections — key for spotting unauthorized services or reverse shells.

### Show active network connections (modern)

```bash
ss -tulnp
```

**Dissection:**
| Part | Meaning |
|---|---|
| `ss` | Modern, faster replacement for `netstat` |
| `-t` | TCP sockets |
| `-u` | UDP sockets |
| `-l` | Listening sockets only |
| `-n` | Numeric addresses/ports |
| `-p` | Show owning process |

**Use case:** Faster, modern alternative to `netstat` for the same purpose — checking for suspicious open ports or connections.

### Show routing table

```bash
ip route
```

**Dissection:**
| Part | Meaning |
|---|---|
| `ip` | Network configuration tool |
| `route` | Show the kernel's routing table |

**Use case:** Verify routing configuration hasn't been tampered with to redirect traffic.

### Test connectivity to a host

```bash
ping -c 4 8.8.8.8
```

**Dissection:**
| Part | Meaning |
|---|---|
| `ping` | Sends ICMP echo requests to test reachability |
| `-c 4` | Send exactly 4 packets, then stop |
| `8.8.8.8` | Target host/IP |

**Use case:** Confirm network connectivity or test if a suspicious external IP is reachable.

### Trace network path to a host

```bash
traceroute 8.8.8.8
```

**Dissection:**
| Part | Meaning |
|---|---|
| `traceroute` | Maps the hop-by-hop path packets take to a destination |
| `8.8.8.8` | Target host/IP |

**Use case:** Map the network path to an external IP, useful when investigating C2 (command-and-control) communication.

### Perform DNS lookup

```bash
dig domain.com
```

**Dissection:**
| Part | Meaning |
|---|---|
| `dig` | DNS lookup utility, queries records with detailed output |
| `domain.com` | Target domain to query |

**Use case:** Investigate a suspicious domain's DNS records, which can reveal infrastructure tied to an attacker.

### Reverse DNS/simple domain lookup

```bash
nslookup domain.com
```

**Dissection:**
| Part | Meaning |
|---|---|
| `nslookup` | Simple DNS query tool for resolving domains/IPs |
| `domain.com` | Target domain to query |

**Use case:** Quickly check what IP a suspicious domain resolves to, or what domain an IP maps back to.

### Download a file from a URL

```bash
curl -O http://example.com/file
```

**Dissection:**
| Part | Meaning |
|---|---|
| `curl` | Command-line tool to transfer data from/to a URL |
| `-O` | Save output using the remote file's name |
| `http://example.com/file` | Target URL |

**Use case:** Retrieve a suspicious file from a URL found in logs for safe analysis in an isolated environment (use a sandboxed VM, never production).

### Capture network packets

```bash
tcpdump -i eth0 -w capture.pcap
```

**Dissection:**
| Part | Meaning |
|---|---|
| `tcpdump` | Command-line packet capture/analysis tool |
| `-i eth0` | Capture on interface `eth0` |
| `-w capture.pcap` | Write raw packets to a file for later analysis (e.g. in Wireshark) |

**Use case:** Capture live traffic on an interface for later analysis in Wireshark during an active incident.

---

## Logs and auditing

### View system log (systemd)

```bash
journalctl -xe
```

**Dissection:**
| Part | Meaning |
|---|---|
| `journalctl` | Queries the systemd journal (system logs) |
| `-x` | Add explanatory help text to log entries where available |
| `-e` | Jump to the end of the log (most recent entries) |

**Use case:** Review recent system logs with explanations, useful for triaging errors and security-relevant events.

### View authentication log

```bash
cat /var/log/auth.log
```

**Dissection:**
| Part | Meaning |
|---|---|
| `cat` | Prints file contents |
| `/var/log/auth.log` | Debian/Ubuntu log of authentication events (SSH, sudo, etc.) |

**Use case:** Review SSH logins, sudo usage, and authentication failures (Debian/Ubuntu systems).

### View secure log (RHEL/CentOS equivalent)

```bash
cat /var/log/secure
```

**Dissection:**
| Part | Meaning |
|---|---|
| `cat` | Prints file contents |
| `/var/log/secure` | RHEL/CentOS equivalent of `auth.log` |

**Use case:** Same purpose as `auth.log` but for RHEL-based systems — check for brute-force or unauthorized access attempts.

### Search logs for a keyword

```bash
grep "Failed password" /var/log/auth.log
```

**Dissection:**
| Part | Meaning |
|---|---|
| `grep` | Searches text for lines matching a pattern |
| `"Failed password"` | The exact pattern to search for |
| `/var/log/auth.log` | File to search within |

**Use case:** Quickly filter logs for specific indicators like failed login attempts, a common sign of brute-forcing.

### Search recursively across files

```bash
grep -r "malicious_string" /var/log/
```

**Dissection:**
| Part | Meaning |
|---|---|
| `grep` | Pattern search tool |
| `-r` | Recursive — search all files within the directory tree |
| `"malicious_string"` | Pattern/IOC to search for |
| `/var/log/` | Directory to search within |

**Use case:** Hunt for a known indicator of compromise (IOC) across all log files in a directory.

### Show command history

```bash
history
```

**Dissection:**
| Part | Meaning |
|---|---|
| `history` | Shell built-in that lists previously executed commands in the session |

**Use case:** Review commands run by the current shell session — useful for seeing what an attacker (or user) executed if a shell was compromised.

---

## Scheduled tasks and persistence checks

### List current user's cron jobs

```bash
crontab -l
```

**Dissection:**
| Part | Meaning |
|---|---|
| `crontab` | Manages per-user scheduled cron jobs |
| `-l` | List the current user's crontab entries |

**Use case:** Check for unauthorized scheduled tasks, a common persistence mechanism for attackers.

### List all users' cron jobs

```bash
ls /etc/cron.d/ /etc/cron.daily/ /etc/cron.hourly/
```

**Dissection:**
| Part | Meaning |
|---|---|
| `ls` | Lists directory contents |
| `/etc/cron.d/` etc. | System-wide cron directories storing scheduled jobs outside individual user crontabs |

**Use case:** Audit system-wide scheduled jobs for malicious entries placed outside a single user's crontab.

### List systemd services

```bash
systemctl list-units --type=service
```

**Dissection:**
| Part | Meaning |
|---|---|
| `systemctl` | Controls and inspects the systemd service manager |
| `list-units` | Lists currently loaded units |
| `--type=service` | Filter to only show services |

**Use case:** Identify unfamiliar or newly created services that could indicate attacker-installed persistence.

### Check for unusual startup services

```bash
systemctl list-unit-files --state=enabled
```

**Dissection:**
| Part | Meaning |
|---|---|
| `systemctl` | systemd control tool |
| `list-unit-files` | Lists all installed unit files (not just currently running) |
| `--state=enabled` | Filter to units configured to start automatically |

**Use case:** Spot services configured to auto-start at boot that shouldn't be there, another common persistence technique.

---

## User and account management

### List all user accounts

```bash
cat /etc/passwd
```

**Dissection:**
| Part | Meaning |
|---|---|
| `cat` | Prints file contents |
| `/etc/passwd` | System file listing all user accounts and their basic attributes |

**Use case:** Check for unauthorized or unexpected user accounts created by an attacker.

### List all groups

```bash
cat /etc/group
```

**Dissection:**
| Part | Meaning |
|---|---|
| `cat` | Prints file contents |
| `/etc/group` | System file listing all groups and their members |

**Use case:** Verify group memberships haven't been altered to grant unauthorized privileges.

### Show sudo privileges configuration

```bash
cat /etc/sudoers
```

**Dissection:**
| Part | Meaning |
|---|---|
| `cat` | Prints file contents |
| `/etc/sudoers` | Configuration file defining which users/groups can run commands as another user (typically root) |

**Use case:** Confirm no unauthorized entries were added granting a user elevated privileges.

### Check currently installed packages

```bash
dpkg -l
```

**Dissection:**
| Part | Meaning |
|---|---|
| `dpkg` | Debian package manager tool |
| `-l` | List all installed packages |

**Use case:** Review installed software for anything unexpected or not sanctioned (Debian/Ubuntu).

## Blue-team reminder
Use commands only on systems and data you own or are explicitly authorized to investigate.
