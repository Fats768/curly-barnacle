# 🧹 Debian 13 + Wine Cleanup & Optimization Guide

**Complete documentation for removing Wine mess and system bloat**

Last updated: May 4, 2026  
Target: Debian 13 (Trixie)

---

## 📋 Quick Start: Pick Your Scenario

| Your Situation | Do This | Time |
|---|---|---|
| **Wine crashes/conflicts** | Read `WINE_CLEANUP_REINSTALL.md` | 30-45 min |
| **Too many .exe/.dll files** | Read `DEBIAN_CLEANUP.md` | 20-30 min |
| **Both problems + slow system** | Do both guides in order | 60-90 min |
| **Not sure what to do** | Read this README first | 5-10 min |

---

## 📚 What's in This Repository

### 1️⃣ **README.md** (You are here)
Quick reference, safety checklist, FAQ, and decision guide.

### 2️⃣ **WINE_CLEANUP_REINSTALL.md** (6 Phases)
Complete Wine removal and fresh installation.
- Backup game saves
- Purge all Wine packages
- Clean user data completely
- Fresh install (3 options: WineHQ, Debian, or Proton)
- Setup DirectX and drivers
- Test and troubleshoot

### 3️⃣ **DEBIAN_CLEANUP.md** (8 Phases)
System-wide cleanup for Debian 13.
- Find and remove Windows files
- Clean orphaned packages
- Remove temporary files and logs
- Delete build artifacts
- Optimize system services
- Automated cleanup script

---

## 🎯 Decision Tree

```
Do you have Wine/game problems?
├─ YES → Is Wine crashing? → YES → Start with WINE_CLEANUP_REINSTALL.md
│        └─ NO → Is disk full? → YES → Start with DEBIAN_CLEANUP.md
└─ NO → Is disk/system slow? → YES → Start with DEBIAN_CLEANUP.md
        └─ NO → Check FAQ section below
```

---

## ⚠️ Safety Checklist

**Before you do ANYTHING, check these boxes:**

- [ ] **You have backups** - Have you backed up important game saves?
  ```bash
  mkdir -p ~/wine-backups
  cp -r ~/.wine/drive_c/users/$USER/Documents ~/wine-backups/
  ```

- [ ] **You've read the guide completely** - Don't skip to commands!

- [ ] **You understand the risks** - Some commands delete files permanently

- [ ] **You have sudo access** - Can you run `sudo whoami`?
  ```bash
  sudo whoami  # Should show "root"
  ```

- [ ] **You have disk space** - At least 5GB free
  ```bash
  df -h /
  ```

- [ ] **You have internet** - To download packages if needed
  ```bash
  ping -c 1 8.8.8.8
  ```

**If any of these fail, STOP and fix them first.**

---

## 🚀 Recommended Execution Order

### Scenario A: Wine + System Problems (Best approach)

1. ✅ Read this README (5 min)
2. ✅ Backup game saves (see above)
3. ✅ Run DEBIAN_CLEANUP.md Phases 1-3 (clean system first)
4. ✅ Run WINE_CLEANUP_REINSTALL.md completely (fresh Wine)
5. ✅ Run DEBIAN_CLEANUP.md Phases 4-8 (final optimization)
6. ✅ Test your games

### Scenario B: Wine Problems Only

1. ✅ Read this README (5 min)
2. ✅ Backup game saves
3. ✅ Run WINE_CLEANUP_REINSTALL.md completely
4. ✅ Test games

### Scenario C: System Cleanup Only

1. ✅ Read this README (5 min)
2. ✅ Run DEBIAN_CLEANUP.md completely
3. ✅ Verify system still works

---

## 📖 How to Use These Guides

### Reading the Guides

- Each guide is **numbered by Phase** (1, 2, 3...)
- Each phase has **Step numbers** (1.1, 1.2, 1.3...)
- Phases should be done in order
- Steps within phases should be done in order
- **Read the entire phase before running any commands**

### Running Commands

```bash
# ALWAYS verify first (dry-run)
sudo apt autoremove --dry-run

# Then run the actual command
sudo apt autoremove -y

# Verify the result
apt list --installed | wc -l
```

### If Something Goes Wrong

Every guide has a **Rollback section**. Use it to undo changes:

```bash
# Example rollback
sudo apt install wine wine32 wine64 -y
cp -r ~/wine-backups/.wine ~/.wine
```

---

## ❓ FAQ

### Q: Will this delete my games?
**A:** Not if you follow the guides. Games installed in `~/.wine/drive_c/Program Files` will be deleted. That's why you backup first. Extract the backup after reinstalling Wine.

### Q: How long will this take?
**A:** 
- WINE only: 30-45 minutes
- DEBIAN only: 20-30 minutes  
- Both: 60-90 minutes
- Most time is waiting for downloads/installations

### Q: Can I do this on an old laptop?
**A:** Yes, but:
- May take longer (1-2 hours)
- Ensure 5GB+ free disk space
- Close unnecessary programs first

### Q: What if I only want to clean, not reinstall Wine?
**A:** Do DEBIAN_CLEANUP.md only. It won't touch Wine packages.

### Q: Can I run both guides at once?
**A:** No. Do DEBIAN_CLEANUP phases 1-3, then WINE guide, then DEBIAN phases 4-8.

### Q: Will this improve game performance?
**A:** Sometimes:
- Removing old/broken packages: ✅ Yes
- Cleaning logs/cache: ✅ Minor improvement
- Fresh Wine install: ✅ Can help with crashes
- System optimization: ✅ May help overall speed

### Q: What if my system breaks?
**A:** 
- Press Ctrl+Alt+T to open terminal
- Follow the Rollback section of the guide
- If desktop won't start: Boot into Recovery mode (hold Shift at startup)

### Q: Can I do this on a virtual machine?
**A:** Yes, and safer! Take a snapshot first:
```bash
# In VM manager, create snapshot before cleanup
```

---

## 🔍 Verification Steps

After each major phase, run these:

```bash
# Check system is healthy
sudo apt check
sudo apt upgrade --dry-run

# Verify Wine (if applicable)
wine --version

# Check disk usage
df -h /

# Check for broken packages
sudo apt --fix-broken install

# Verify desktop still works
echo "Logout and login to test desktop environment"
```

---

## 🐛 Troubleshooting Common Issues

### "Command not found: wine"
**Solution:**
```bash
sudo apt install wine -y
# Then run WINE_CLEANUP_REINSTALL.md Phase 5
```

### "E: Unable to correct problems, you have held broken packages"
**Solution:**
```bash
sudo apt --fix-broken install -y
sudo apt autoremove -y
```

### "No space left on device"
**Solution (do DEBIAN_CLEANUP first):**
```bash
sudo apt clean
sudo apt autoclean
sudo journalctl --vacuum=7d  # Keep only 7 days of logs
```

### Desktop won't start after cleanup
**Solution:**
```bash
sudo apt install debian-desktop-environment gdm3 -y
sudo systemctl restart gdm3
```

### Wine/games run very slow
**Solution (after reinstalling Wine):**
```bash
winetricks vcrun2019 directsound d3dx9
# Then run winecfg and enable CSMT
```

---

## 📊 Expected Disk Space Changes

After cleanup, you should see:

| Action | Typical Space Freed |
|---|---|
| Remove old .exe files | 500MB - 5GB |
| Clean package cache | 100MB - 500MB |
| Remove orphaned packages | 200MB - 2GB |
| Clean logs | 50MB - 500MB |
| Remove Wine prefixes | 2GB - 10GB |
| **Total: 3GB - 18GB** | ✅ Varies greatly |

---

## 🔒 Important Security Notes

**Do NOT:**
- Run these guides with `sudo` unless instructed
- Delete files you don't recognize
- Mix and match commands from different guides
- Skip the backup phase
- Ignore error messages

**DO:**
- Read commands before running them
- Understand what each command does
- Keep backups for 7+ days after cleanup
- Test games after reinstalling Wine
- Ask for help if unsure

---

## 📞 Getting Help

If something goes wrong:

1. **Check the guide's Troubleshooting section** first
2. **Run the Rollback section** to undo changes
3. **Check system logs:** `sudo journalctl -xe`
4. **Verify package integrity:** `sudo apt check`
5. **Ask on:** 
   - WineHQ forums: https://forum.winehq.org
   - Debian forums: https://forums.debian.net
   - r/debian or r/linux on Reddit

---

## 📝 Success Criteria

You're done when:

- ✅ No page fault errors (like your backtrace)
- ✅ Games launch without crashing
- ✅ Audio/video work properly
- ✅ System boots normally
- ✅ Disk space improved by at least 1GB
- ✅ No broken package errors
- ✅ Desktop runs smoothly

---

## 🎓 Learning from This

After cleanup, you'll have learned:

- How Wine works and where it stores data
- How package management works in Debian
- How to safely delete system files
- How to use backup/rollback procedures
- How to diagnose and fix system problems

This knowledge applies to many Linux system maintenance tasks!

---

## 📋 Pre-Cleanup Checklist

Copy this and use it:

```
Pre-Cleanup Checklist:
☐ Backed up game saves to ~/wine-backups/
☐ Read the relevant guide completely
☐ Closed all running games
☐ Closed browser (saves memory)
☐ Have internet connection
☐ Have at least 5GB disk space (df -h /)
☐ Can access sudo (sudo whoami)
☐ Created system snapshot (if on VM)
☐ Know my WiFi password (in case internet drops)
☐ Have 1-2 hours available

Ready to start!
```

---

## 🎉 Next Steps

1. **Pick your scenario** from the Decision Tree above
2. **Read the appropriate guide** completely (don't skim!)
3. **Follow the Pre-Cleanup Checklist** above
4. **Execute one phase at a time**
5. **Test after each major phase**
6. **Report back** if anything breaks (use Rollback)

Good luck! 🚀

---

## 📚 Additional Resources

- **WineHQ Installation Guide:** https://wiki.winehq.org/Download
- **Debian Package Management:** https://wiki.debian.org/PackageManagement
- **ProtonDB for Gaming:** https://protondb.com
- **Wine Prefixes:** https://wiki.winehq.org/FAQ#head-12c0b1d4eac588b47e8e0a289d4d1176b8ed11ee
