# Chrome PWA Vulkan GPU Stability Patch for Arch Linux / Mesa

An automated utility script and diagnostic framework to resolve rendering crashes (`SIGSEGV` in `libgallium`) on hybrid graphics laptops running Arch Linux and Wayland when launching Google Chrome and Progressive Web Apps (PWAs).

---

## 🔍 The Problem

When running graphics-heavy HTML5 Canvas workflows (e.g., **Conceptboard**, **Canva**, or **Figma**) in Google Chrome on Arch Linux under Wayland with AMD integrated graphics, the browser will frequently crash with a `SIGSEGV` signal in the Mesa Gallium OpenGL state tracker (`libgallium-*.so`).

### The Triple-Bypass Bug
1. **The VRAM fragmentation**: Heavy canvas textures exhaust the integrated AMD GPU's 512MB dedicated UMA VRAM pool. The Gallium OpenGL allocator fails to handle this gracefully under memory pressure, returning a null pointer that triggers a browser-wide crash.
2. **Missing Vulkan Drivers**: Arch Linux installations on hybrid laptop systems often lack the proper open-source Vulkan driver for the AMD iGPU (`vulkan-radeon`), causing Chrome to fall back to the crash-prone OpenGL backend even if Vulkan flags are set.
3. **The Launcher Bypass**: Standard user configuration flags in `~/.config/chrome-flags.conf` are **completely ignored** by Progressive Web Apps (PWAs). This is because PWAs launch `/opt/google/chrome/chrome` directly, bypassing the standard `/usr/bin/google-chrome-stable` wrapper script that loads user configurations.

---

## 🛠️ The Solution

This repository provides a two-part solution:

### Part 1: Install the AMD Vulkan Driver
Ensure the open-source Vulkan driver (`RADV`) is installed on your Arch Linux machine so the AMD iGPU is properly discovered:
```bash
sudo pacman -S vulkan-radeon
```

### Part 2: Automate Launcher Patching
The utility script `fix_chrome_pwa_flags.sh` automatically detects all your installed Chrome PWAs, creates backups, and injects stable Vulkan GPU flags directly into the `.desktop` launchers:
* **Enables Vulkan backend** (`--enable-features=Vulkan,UseSkiaRenderer`, `--use-angle=vulkan`)
* **Forces native Wayland rendering** (`--ozone-platform=wayland`)
* **Optimizes GPU compositing under memory pressure** (`--enable-gpu-rasterization`, `--ignore-gpu-blocklist`, `--disable-gpu-sandbox`)

---

## 🚀 How to Use

1. **Clone this repository**:
   ```bash
   git clone https://github.com/YOUR_USERNAME/chrome-pwa-gpu-patch.git
   cd chrome-pwa-gpu-patch
   ```

2. **Make the script executable**:
   ```bash
   chmod +x fix_chrome_pwa_flags.sh
   ```

3. **Run the patcher**:
   ```bash
   ./fix_chrome_pwa_flags.sh
   ```

The script will safely backup all your PWA files to `~/antigravity_gpu_debug/pwa_backups/` and apply the robust Vulkan configurations.

---

## 📊 Technical Diagnostics & Forensics

For a deep dive into the system diagnostics, backtrace symbols, and VRAM memory exhaustion proof, please read our comprehensive [Technical Debugging & Forensic Report](technical_report.md).

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
