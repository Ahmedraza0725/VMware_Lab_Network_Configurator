<div align="center">

# 🧪 VMware Lab Network Auto-Configurator

### Put Kali and your target machines on the same VMnet8 network — in one click

*Scans your VMware library, finds your lab VMs, and rewrites their network settings so they can all talk to each other. No manual clicking through Virtual Network Editor or VM settings.*

![PowerShell](https://img.shields.io/badge/PowerShell-5.1+-5391FE?style=flat&logo=powershell&logoColor=white)
![Platform](https://img.shields.io/badge/Platform-Windows-0078D4?style=flat&logo=windows&logoColor=white)
![VMware](https://img.shields.io/badge/VMware-Workstation%20%2F%20Player-607078?style=flat&logo=vmware&logoColor=white)
![Lab](https://img.shields.io/badge/Use-Pentest%20Lab-a78bfa?style=flat)
![License](https://img.shields.io/badge/License-MIT-brightgreen?style=flat)

</div>

---

## 📖 About

Setting up a penetration-testing lab usually means opening every VM's settings and making sure each one is attached to the same virtual network. Miss one, or pick the wrong adapter type for an old machine, and the attacker VM can't reach the target.

**Configure-VMware-LabNetwork.bat** automates that. It finds your imported lab VMs (Kali, Kioptrix, Metasploitable), attaches all of them to **VMnet8**, picks the correct network adapter driver for each, and keeps a backup of every file it touches.

> ⚠️ **Use only on your own lab.** The script edits `.vmx` configuration files. It is intended for personal training labs with intentionally vulnerable machines.

---

## ✨ Features

| Category | Details |
|---|---|
| 🔍 **Library Scan** | Reads the VMware Workstation inventory (`inventory.vmls`) so only VMs you've imported or opened are touched |
| 🎯 **Smart VM Matching** | Picks VMs whose name contains `kali`, `kiop`, or `metasploit` |
| 🌐 **Unified Network** | Attaches every matched VM to the same **VMnet8** custom network so they share a subnet |
| 🧩 **Per-VM NIC Drivers** | `vlance` (AMD PCnet) for legacy Kioptrix, `e1000` for Kali and everything else |
| 💿 **Kioptrix Fixes** | Disables the missing CD-ROM entries and 3D acceleration that commonly block old VMs from booting cleanly |
| 💾 **Automatic Backup** | Creates a `.vmx.bak` next to each config before changing it (never overwrites an existing backup) |
| 🛡️ **Safety Checks** | Skips VMs that are running or locked, and tells you exactly why |
| 🖥️ **Colored Console Output** | Clear step-by-step progress: `[1/3]` scan, `[2/3]` configure, summary |
| 🧱 **Single-File Design** | A `.bat`/PowerShell polyglot, so you just double-click it. Nothing to install |

---

## ⚙️ How It Works

```
Double-click .bat
      │
      ▼
Batch wrapper launches PowerShell (-NoProfile, ExecutionPolicy Bypass)
      │
      ▼
[1/3] Locate vmrun.exe → list running VMs
      Read %APPDATA%\VMware\inventory.vmls → registered VMs
      Filter by name: kali / kiop / metasploit
      │
      ▼
[2/3] For each matched VM:
      ├─ Skip if running or .vmx is locked
      ├─ Back up  →  <name>.vmx.bak
      ├─ Remove old ethernet0.* connection keys
      ├─ Set ethernet0.connectionType = "custom"
      ├─ Set ethernet0.vnet           = "VMnet8"
      ├─ Set NIC driver (vlance for Kioptrix, e1000 otherwise)
      └─ Kioptrix only: disable CD-ROM entries + 3D
      │
      ▼
[3/3] Print summary → power on your VMs
```

### The `.vmx` settings it writes

| Key | Value |
|---|---|
| `ethernet0.present` | `TRUE` |
| `ethernet0.startConnected` | `TRUE` |
| `ethernet0.connectionType` | `custom` |
| `ethernet0.vnet` | `VMnet8` |
| `ethernet0.displayName` | `VMnet8` |
| `ethernet0.virtualDev` | `vlance` (Kioptrix) / `e1000` (others) |

---

## 🧰 Requirements

- **OS:** Windows 10 / 11
- **Hypervisor:** VMware Workstation or VMware Player
- **Shell:** Windows PowerShell 5.1+ (built into Windows)
- **Your VMs:** already imported or opened in VMware, with `kali`, `kiop`, or `metasploit` in the VM file name
- **State:** all target VMs **powered off**

---

## 🚀 Usage

```text
1. Open VMware Workstation and make sure your Kali / Kioptrix VMs are in the library
2. Power OFF every lab VM and close their tabs
3. Double-click  Configure-VMware-LabNetwork.bat
4. Read the summary, then power on your VMs
```

Then verify connectivity from Kali:

```bash
ip a                     # confirm Kali got an address on the VMnet8 subnet
sudo netdiscover -r <subnet>/24
nmap -sn <subnet>/24     # find the target machine
```

---

## 🔄 Undo / Restore

Every changed VM has a backup next to its config file. To roll back, close the VM and replace the config with the backup:

```text
<VM folder>\<name>.vmx.bak   →   copy over   →   <name>.vmx
```

---

## 📝 Notes & Limitations

- **VMnet8 is VMware's default NAT network.** Machines on it can reach each other and, by default, the internet through the host. If you need a truly sealed lab (for example, to run vulnerable VMs with no outbound access), use a Host-only network such as **VMnet1** instead, or disable NAT in VMware's Virtual Network Editor. The script's `$netName` variable at the top of the PowerShell section controls which network is used.
- Only the **first adapter** (`ethernet0`) is configured.
- VM matching is by **file name**, so a VM called `target1` will not be picked up. Rename it or extend the `$patterns` list.
- The script does not change VMnet8's own settings (subnet, DHCP). It only attaches VMs to it.
- Config files are written as ASCII, so avoid non-English characters in `.vmx` display names.

---

## 📁 Project Structure

```
VMware_Lab_Network_Configurator/
├── Configure-VMware-LabNetwork.bat   # Batch + PowerShell polyglot (double-click to run)
├── LICENSE
├── .gitignore
└── README.md
```

---

## 🗺️ Roadmap

- [ ] Configurable network name via parameter instead of editing the script
- [ ] Custom VM-name patterns from the command line
- [ ] Support for multiple adapters (`ethernet1`, `ethernet2`)
- [ ] Optional Host-only (VMnet1) sealed-lab mode
- [ ] Dry-run mode that previews changes without writing

---

## 📜 License

Distributed under the **MIT License**. See [`LICENSE`](LICENSE) for details.

---

<div align="center">

Built for the home-lab and ethical-hacking community 🔐

</div>
