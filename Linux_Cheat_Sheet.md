
# Linux Basics

## Identity and current location

### Show current user

```bash
whoami
```

**Use case:** Confirm which account is running commands, especially before changing files or collecting evidence.

### Show current directory

```bash
pwd
```

**Use case:** Confirm where you are before listing, creating, moving, or deleting files.

## Navigate directories

### List files with details

```bash
ls -lah
```

**Use case:** View hidden files, permissions, ownership, sizes, and timestamps.

**Breakdown:**
- `-l`: Long format.
- `-a`: Include hidden files.
- `-h`: Human-readable sizes.

### Change directory

```bash
cd /var/log
```

**Use case:** Move to a directory that commonly contains system and application logs.

### Return to the previous directory

```bash
cd -
```

## Help and command discovery

### Read a manual page

```bash
man ls
```

**Use case:** Learn a command safely from its locally installed documentation.

### Search command help

```bash
grep --help
```

**Use case:** Quickly view available options without opening a full manual page.

## Blue-team reminder

Use commands only on systems and data you own or are explicitly authorized to investigate.
