# 🧹 Complete Debian 13 System Cleanup Guide

**Last Updated:** May 4, 2026  
**Target OS:** Debian 13 (Trixie)  
**Purpose:** Remove unnecessary Windows files, clean packages, and optimize system

---

## ⚠️ Safety First

This guide includes **potentially destructive commands**. Read each section fully before executing. Always run dry-run versions first.

---

## Phase 1: Find and Remove Unnecessary Windows Files

### Step 1.1: Identify Windows executables and DLLs

```bash
# Find all .exe files (usually shouldn't exist on Linux)
echo "=== .EXE files found ==="
find ~ -type f -name "*.exe" 2>/dev/null | head -20

# Find all .dll files (Windows DLLs)
echo "=== .DLL files found ==="
find ~ -type f -name "*.dll" 2>/dev/null | head -20

# Find all .msi files (Windows installers)
echo "=== .MSI files found ==="
find ~ -type f -name "*.msi" 2>/dev/null | head -20
```

**What to look for:** Files in `~/Downloads`, `~/Games`, `~/wine-*`, or random home directories

### Step 1.2: Clean up downloads and game directories

```bash
# EXAMINE FIRST - don't blindly delete!

# Example: Clean game installers (if you've already installed the games)
ls -lh ~/Downloads/*.exe
ls -lh ~/Games/*.exe

# Remove specific files (one at a time)
rm ~/Downloads/game-installer.exe

# Or remove entire old game directories
# rm -rf ~/Games/OldGame123
```

### Step 1.3: Clean Wine-related temporary files

```bash
# Wine temporary files
rm -rf ~/.wine/dosdevices/
rm -rf ~/.wine/*.log
rm -rf ~/.cache/wine-mono*
rm -rf ~/.cache/wine-gecko*

# Proton temporary files (if using Proton)
rm -rf ~/.steam/root/compatibilitytools/Proton*/  # ONLY if not actively using!

# Verify nothing critical was removed
ls -la ~/.wine/ | head
```

---

## Phase 2: Remove Orphaned and Unnecessary Packages

### Step 2.1: Remove packages from learning/experiments

```bash
# List packages you might not need anymore
apt search '~o'  # Shows orphaned packages

# Example: Remove packages that were only for testing
sudo apt purge build-essential -y  # Only if you're not developing!
sudo apt purge git -y  # Only if you're not using it!
sudo apt purge gcc -y  # Only if you don't need compilation!

# SAFER: Just list what's installed
apt list --installed | grep -E 'wine|windows|msvc|build' | head -20
```

### Step 2.2: Remove all orphaned dependencies

```bash
# Find packages no longer needed
sudo apt autoremove -y

# More aggressive: remove unused libraries
sudo apt autoremove --purge -y

# Show what will be removed (dry run)
sudo apt autoremove --dry-run

# Actually remove it
sudo apt autoremove -y
```

### Step 2.3: Clean package cache

```bash
# Remove cached .deb packages
sudo apt clean

# More aggressive: remove old versions of packages
sudo apt autoclean

# Check cache size before/after
du -sh /var/cache/apt/archives/

# Nuclear option: remove ALL cached packages (use if disk space critical)
sudo rm -rf /var/cache/apt/archives/*
```

---

## Phase 3: Clean Temporary Files and Logs

### Step 3.1: Clean system temporary directories

```bash
# Show what's in /tmp
ls -lah /tmp | head -30

# Remove old temporary files (older than 10 days)
sudo find /tmp -type f -atime +10 -delete

# Remove entire /tmp and /var/tmp (be careful!)
# Restart services that depend on these directories
sudo systemctl restart systemd-tmpfiles-clean.service

# Or manually (if safe)
# sudo rm -rf /tmp/*
# sudo rm -rf /var/tmp/*
```

### Step 3.2: Clean log files

```bash
# Show log file sizes
du -sh /var/log/*

# SAFELY truncate old logs (not delete)
sudo journalctl --vacuum=30d  # Keep only 30 days of system logs

# Truncate individual log files
sudo truncate -s 0 /var/log/syslog
sudo truncate -s 0 /var/log/apt/history.log
sudo truncate -s 0 /var/log/apt/term.log

# Or remove old logs (safer approach)
sudo find /var/log -type f -name "*.log" -mtime +30 -delete
```

### Step 3.3: Clean user cache

```bash
# Show cache size
du -sh ~/.cache/

# Clean specific caches
rm -rf ~/.cache/pip
rm -rf ~/.cache/npm
rm -rf ~/.cache/thumbnails

# Clean all cache (might slow down apps temporarily)
rm -rf ~/.cache/*

# Or safely keep important caches
rm -rf ~/.cache/* --except ~/.cache/google-chrome  # or equivalent
```

---

## Phase 4: Remove Build Artifacts and Compilation Leftovers

```bash
# Find build directories from learning projects
find ~ -name "build" -type d 2>/dev/null
find ~ -name "*.o" -type f 2>/dev/null  # Object files
find ~ -name "*.out" -type f 2>/dev/null  # Compiled executables
find ~ -name "node_modules" -type d 2>/dev/null  # Node packages

# Example cleanup (examine first!)
# rm -rf ~/my-project/build
# rm -rf ~/node-learning/node_modules
```

---

## Phase 5: Optimize System Services

### Step 5.1: Disable unnecessary services

```bash
# List running services
systemctl list-units --type=service --state=running

# EXAMINE FIRST! Only disable what you don't need:

# Example: Disable Bluetooth (if you don't use it)
sudo systemctl disable bluetooth.service

# Disable cups printing service (if you don't print)
sudo systemctl disable cups.service

# Disable avahi mDNS (if you don't use it)
sudo systemctl disable avahi-daemon.service

# Verify changes
sudo systemctl status bluetooth.service
```

### Step 5.2: Remove unused locales

```bash
# Show installed locales
locale -a

# Remove unnecessary locales (if you only use English)
sudo apt purge language-pack-gnome-* language-pack-* -y  # CAREFUL!

# Or more targeted
locale-gen --no-purge en_US.UTF-8
```

---

## Phase 6: Disk Usage Analysis

### Find what's taking space

```bash
# Top 20 largest files/folders
du -sh /* 2>/dev/null | sort -rh | head -20

# Large individual files
find ~ -type f -size +100M 2>/dev/null

# Largest directories
du -sh ~/* 2>/dev/null | sort -rh | head -20

# Wine prefix size
du -sh ~/.wine
```

---

## Phase 7: Safe Cleanup Script (Automated)

Save this as `~/cleanup.sh` and review before running:

```bash
#!/bin/bash
set -e

echo "=== Debian 13 Cleanup Script ==="
echo "This script will clean temporary files and caches"
echo ""
read -p "Continue? (y/n) " -n 1 -r
echo
if [[ ! $REPLY =~ ^[Yy]$ ]]; then
    exit 1
fi

# Phase 1: Package cleanup
echo "Removing orphaned packages..."
sudo apt autoremove -y
sudo apt autoclean -y

# Phase 2: Cache cleanup
echo "Cleaning cache..."
sudo apt clean
rm -rf ~/.cache/thumbnails
rm -rf ~/.cache/pip
rm -rf ~/.cache/npm

# Phase 3: Log cleanup
echo "Truncating old logs..."
sudo journalctl --vacuum=30d
sudo find /var/log -type f -name "*.log" -mtime +30 -delete

# Phase 4: Temp cleanup
echo "Cleaning temporary files..."
sudo find /tmp -type f -atime +10 -delete 2>/dev/null

echo ""
echo "=== Cleanup Complete ==="
df -h /  # Show new disk usage
```

**Run it:**
```bash
chmod +x ~/cleanup.sh
./cleanup.sh
```

---

## 🚫 DANGEROUS COMMANDS TO AVOID

**DO NOT RUN THESE** (unless you know exactly what you're doing):

```bash
# ❌ Deletes ENTIRE home directory
# rm -rf ~/*

# ❌ Deletes system files - WILL BREAK DEBIAN
# sudo rm -rf /etc
# sudo rm -rf /usr
# sudo rm -rf /var

# ❌ Removes system packages needed for desktop
# sudo apt purge ubuntu-desktop  # or debian-desktop

# ❌ Removes package manager itself
# sudo apt purge apt apt-get

# ❌ Removes boot files
# sudo rm -rf /boot

# ❌ Clears all logs, might hide errors
# sudo truncate -s 0 /var/log/*

# ❌ Removes all temporary files (might break running apps)
# sudo rm -rf /tmp/*

# ❌ Removes critical system libraries
# sudo apt purge libc6
```

---

## Phase 8: Verification Checklist

After cleanup:

- [ ] Boot still works: `sudo reboot`
- [ ] Disk space improved: `df -h`
- [ ] No error messages on login
- [ ] Wine/Games still work (if applicable)
- [ ] Audio/Video still working: `speaker-test -t sine`
- [ ] Network connection OK: `ping 8.8.8.8`
- [ ] No broken packages: `sudo apt check`

---

## Verification Commands

```bash
# Check system health after cleanup
sudo apt check
sudo apt upgrade --dry-run
dpkg --configure -a  # Fix any broken packages

# Show what will be cleaned next time
sudo apt --dry-run autoremove

# Check disk usage
du -sh /* 2>/dev/null | sort -rh
```

---

## Rollback (If Something Breaks)

If you accidentally deleted something important:

```bash
# Reinstall Debian desktop environment
sudo apt install debian-desktop-environment -y

# Reinstall desktop manager (if missing)
sudo apt install gdm3 -y

# Reinstall Wine (if deleted)
sudo apt install wine wine32 wine64 -y
```

---

## Recommended Cleanup Frequency

- **Weekly:** Run `apt autoclean` and cache cleanup
- **Monthly:** Full cleanup script
- **Quarterly:** Remove old log files and rebuild package cache

---

## Final Tips

1. **Always back up** before major cleanup
2. **Run dry-run versions** first (--dry-run flag)
3. **One phase at a time** - don't rush
4. **Monitor disk space** after each phase
5. **Keep system updates current** after cleanup

Good luck! 🚀

