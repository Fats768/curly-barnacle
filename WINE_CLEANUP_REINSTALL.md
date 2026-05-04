# 🍷 Complete Wine Cleanup & Reinstall Guide (Debian 13)

**Last Updated:** May 4, 2026  
**Target OS:** Debian 13 (Trixie)  
**Scope:** Complete removal and fresh installation of Wine

---

## ⚠️ WARNING: This is Destructive

This guide **WILL DELETE** all Wine installations, prefixes, and game data. **Backup anything you want to keep first.**

---

## Phase 1: Backup Important Data (Optional)

If you have game saves you want to keep, back them up now:

```bash
# Create a backup directory
mkdir -p ~/wine-backups

# Back up NFS Underground saves (example)
cp -r ~/.wine/drive_c/users/$USER/Documents/My\ Games ~/wine-backups/

# Back up other important game data
# Find game prefixes and copy manually if needed
ls -la ~/.wine/drive_c/Program\ Files*/
```

---

## Phase 2: Remove All Wine Packages

### Step 2.1: Purge Wine packages from Debian repos

```bash
# Remove all Wine-related packages
sudo apt purge wine wine32 wine64 wine-i386 wine-amd64 wine-common wine-stable wine-development -y

# Remove Wine staging if installed
sudo apt purge wine-staging -y

# Clean up related packages
sudo apt purge winetricks protontricks -y
```

### Step 2.2: Clean package manager

```bash
# Remove orphaned dependencies
sudo apt autoremove -y

# Clean cache
sudo apt clean
sudo apt autoclean
```

### Step 2.3: Remove WineHQ repository (if added)

```bash
# Remove WineHQ repository
sudo rm -f /etc/apt/sources.list.d/winehq.list
sudo rm -f /etc/apt/sources.list.d/wine-staging.list

# Remove WineHQ GPG keys
sudo rm -f /etc/apt/keyrings/winehq-archive.key
sudo rm -f /usr/share/keyrings/winehq-archive.gpg

# Update package list
sudo apt update
```

---

## Phase 3: Remove All User Wine Data

```bash
# CAREFUL: This deletes ALL Wine prefixes, saves, and configurations
rm -rf ~/.wine

# Remove Wine cache
rm -rf ~/.cache/wine
rm -rf ~/.cache/winetricks

# Remove Wine config
rm -rf ~/.config/wine

# Remove local Wine data
rm -rf ~/.local/share/wine

# Remove any manually created prefixes (example)
# rm -rf ~/Games/NFS-Underground  (only if you don't need saves)
```

**Verify it's gone:**
```bash
ls -la ~/ | grep -i wine
# Should return nothing
```

---

## Phase 4: Fresh Installation

Choose ONE of these options based on your needs:

### Option A: WineHQ Stable (Recommended - Most Compatible)

```bash
# Add WineHQ repository
sudo dpkg --add-architecture i386
wget -qO- https://dl.winehq.org/wine-builds/winehq.key | sudo gpg --dearmor -o /etc/apt/keyrings/winehq-archive.key

echo "deb [signed-by=/etc/apt/keyrings/winehq-archive.key] https://dl.winehq.org/wine-builds/debian bookworm main" | sudo tee /etc/apt/sources.list.d/winehq-bookworm.list

# Update and install
sudo apt update
sudo apt install winehq-stable -y
```

### Option B: Debian Default (Simple - No extra repos)

```bash
# Install from Debian repos
sudo apt install wine wine32 wine64 -y

# Optional: Install tools
sudo apt install winetricks -y
```

### Option C: Proton (Best for Gaming)

```bash
# Proton is part of Steam, but can be installed standalone
sudo apt install proton -y

# Or via Flatpak
flatpak install flatseal steam
```

**Choose based on:**
- **Option A**: Most games, best support, requires manual repo
- **Option B**: Simple, works for many games, uses older Wine version
- **Option C**: Best for modern games, requires Steam/Flatpak

---

## Phase 5: Post-Installation Setup

### Step 5.1: Install DirectX 9 and Visual C++ Redistributables

```bash
# Install winetricks first (if not already installed)
sudo apt install winetricks -y

# Create a fresh prefix
export WINEPREFIX=~/.wine
wine wineboot

# Install common libraries
winetricks vcrun2019 dotnet48 d3dx9
```

### Step 5.2: Install Fonts

```bash
# Install Windows fonts
winetricks allfonts

# Or manually (Microsoft fonts pack)
winetricks corefonts
```

### Step 5.3: Configure Wine Settings

```bash
# Open Wine configuration
winecfg

# In the dialog:
# - Graphics tab: Enable CSMT (for better performance)
# - Audio tab: Select your audio device
# - Drives tab: Add CD/DVD drives if needed
# - About tab: Set Windows version (7, 10, or 11)
```

### Step 5.4: Optimize for Gaming (Optional)

```bash
# Create a game-specific prefix
export WINEPREFIX=~/.wine-games
WINEARCH=win32 wine wineboot

# Install directsound and OpenGL support
winetricks dsound opengl
```

---

## Phase 6: Install Your Game

### For NFS Underground (Example):

```bash
# Navigate to game installer
cd ~/Downloads

# Run the installer
wine ./nfs-underground-installer.exe

# Or use a pre-configured prefix from ProtonDB
```

### Test the Installation:

```bash
# Launch the game
wine ~/.wine/drive_c/Program\ Files\ \(x86\)/Need\ for\ Speed\ Underground/Speed.exe

# Or create a launcher script
cat > ~/launch-nfs.sh << 'EOF'
#!/bin/bash
export WINEPREFIX=~/.wine
export WINEARCH=win32
wine ~/.wine/drive_c/Program\ Files\ \(x86\)/Need\ for\ Speed\ Underground/Speed.exe
EOF

chmod +x ~/launch-nfs.sh
./launch-nfs.sh
```

---

## Troubleshooting

### Wine won't install
```bash
# Check if i386 architecture is enabled
sudo dpkg --print-architecture
sudo dpkg --print-foreign-architectures

# If i386 is missing
sudo dpkg --add-architecture i386
sudo apt update
```

### Game crashes on startup
```bash
# Try different Windows version
winecfg
# Change Windows version in About tab

# Enable esync or fsync (if available)
WINEESYNC=1 wine game.exe
```

### Audio not working
```bash
# Reinstall audio drivers
winetricks audiodriver=pulseaudio

# Or test with speakers
winetricks sound=speakers
```

### Game runs slowly
```bash
# Enable DXVK (better DirectX 9 support)
winetricks dxvk

# Enable CSMT in winecfg
# Graphics tab → Enable CSMT
```

---

## Verification Checklist

After installation, verify everything works:

- [ ] Wine is installed: `wine --version`
- [ ] 32-bit support enabled: `dpkg --print-foreign-architectures` shows `i386`
- [ ] No errors on `wine wineboot`
- [ ] `winecfg` opens without errors
- [ ] Game installer runs
- [ ] Game launches and displays correctly
- [ ] Audio works in game
- [ ] No page fault crashes (like your earlier issue)

---

## Rollback (If Something Goes Wrong)

If the new installation doesn't work:

```bash
# Keep the current prefix backed up
cp -r ~/.wine ~/.wine-broken

# Go back to Option B (Debian default)
sudo apt purge winehq-stable -y
sudo apt install wine wine32 wine64 -y

# Restore previous config
cp -r ~/.wine-broken ~/.wine
```

---

## Next Steps

1. Run Phase 2-3 completely
2. Choose ONE installation method (Phase 4)
3. Run Phase 5 setup completely
4. Install your game (Phase 6)
5. Test thoroughly before removing backups

Good luck! 🎮

