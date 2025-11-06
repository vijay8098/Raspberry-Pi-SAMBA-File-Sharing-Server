# Raspberry-Pi-SAMBA-File-Sharing-Server

That's an excellent idea\! Setting up a **SAMBA File Sharing Server on a Raspberry Pi** is a common and practical project.

Here is a structured, detailed **GitHub Documentation** that you can use as a template for your repository's **README.md** file, outlining the project, setup steps, and configuration.

-----

# 📂 Raspberry Pi SAMBA File Sharing Server

## Project Overview

This project details the setup of a secure and reliable network-attached storage (NAS) solution using a **Raspberry Pi 4 Model B** and the **SAMBA** file-sharing protocol. SAMBA allows Linux-based systems to provide file and print services to Windows, macOS, and other Linux clients over a local network.

This setup transforms the Raspberry Pi into a dedicated file server, enabling easy, cross-platform access for storing and sharing files.

### Key Features

  * **Cross-Platform Access:** Seamless file sharing with Windows, macOS, and Linux clients.
  * **Secure Authentication:** User-level security using SAMBA passwords.
  * **Network Storage:** Utilizes a connected hard drive/MicroSD for centralized storage.
  * **Headless Operation:** Can run without a monitor or keyboard once configured.

## 🛠️ Hardware & Software Requirements

| Category | Component | Notes |
| :--- | :--- | :--- |
| **Hardware** | Raspberry Pi 4 Model B (2GB/4GB) | Used as the server host. |
| | MicroSD Card ($\ge 32\text{GB}$) | For the Raspberry Pi OS. |
| | External Storage (Optional) | USB Hard Drive or SSD for large file storage. |
| | Ethernet/WiFi Connectivity | Required for network access. |
| **Software** | Raspberry Pi OS (Lite recommended) | The operating system (we used the 64-bit version). |
| | **SAMBA Package** | The core software for the file-sharing protocol. |
| | Linux Utilities | `fdisk`, `mount`, `chown` for disk and permission management. |

## ⚙️ Installation and Setup

### Step 1: Prepare Raspberry Pi OS

1.  **Install OS:** Flash the latest **Raspberry Pi OS** onto the MicroSD card.
2.  **Enable SSH:** For headless setup, enable SSH via the `raspi-config` tool or by placing an empty file named `ssh` in the boot partition.
3.  **Update System:** Access the Pi via SSH and run the following commands:
    ```bash
    sudo apt update
    sudo apt full-upgrade -y
    ```

### Step 2: Install SAMBA

Install the SAMBA daemon and required utilities:

```bash
sudo apt install samba samba-common-bin -y
```

### Step 3: Configure Shared Directory

Create a directory that will be shared across the network.

1.  **Create Directory:**
    ```bash
    # Create a general share directory
    sudo mkdir -p /mnt/samba/myshare
    ```
2.  **Set Permissions:** Grant full ownership of the directory to the default Pi user to simplify access for testing.
    ```bash
    sudo chown -R pi:pi /mnt/samba/myshare
    sudo chmod -R 0775 /mnt/samba/myshare
    ```
    *Note: For production, use specific user/group permissions.*

### Step 4: Configure SAMBA (`smb.conf`)

Edit the main SAMBA configuration file to define the shared folder.

1.  **Backup the original config:**
    ```bash
    sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
    ```
2.  **Open the config file:**
    ```bash
    sudo nano /etc/samba/smb.conf
    ```
3.  **Add the Share Definition:** Scroll to the bottom of the file and add the following block:
    ```ini
    [myshare]
       comment = Raspberry Pi File Share
       path = /mnt/samba/myshare
       browseable = yes
       writeable = yes
       valid users = pi
       create mask = 0775
       directory mask = 0775
       public = no
       read only = no
    ```

### Step 5: Set SAMBA User Password

SAMBA requires its own password database, separate from the Linux system password.

1.  **Add the user (e.g., `pi`) to SAMBA:**
    ```bash
    sudo smbpasswd -a pi
    # Enter a strong password when prompted (it can be the same as your Linux password or different)
    ```

### Step 6: Restart SAMBA Service

Apply the changes by restarting the SAMBA services:

```bash
sudo systemctl restart smbd
sudo systemctl restart nmbd
```

The output you showed, `samba.service... Active: active (running)`, confirms this step was successful.

## 🌐 Accessing the Share

Clients on the network can now access the share using the following methods:

| Operating System | Access Method | Example Path |
| :--- | :--- | :--- |
| **Windows** | File Explorer, Network Location | `\\<Pi's_IP_Address>\myshare` |
| **macOS** | Go $\rightarrow$ Connect to Server | `smb://<Pi's_IP_Address>/myshare` |
| **Linux (Nautilus/Thunar)** | Connect to Server | `smb://<Pi's_IP_Address>/myshare` |

You will be prompted to enter the username (`pi`) and the **SAMBA password** you set in Step 5.

## ⚠️ Troubleshooting and Status Checks

| Command | Purpose | Output Example (Relevant) |
| :--- | :--- | :--- |
| `ip a` | Check the Raspberry Pi's IP address. | `inet 192.168.x.y/24` |
| `sudo systemctl status smbd` | Verify the SAMBA service is running. | `Active: active (running)` |
| `testparm` | Check for syntax errors in `smb.conf`. | `Loaded services file OK.` |

-----

## 🚀 Next Steps (Future Enhancements)

  * **Dedicated Drive:** Integrate an external USB drive and mount it persistently to `/mnt/samba/myshare` for increased storage.
  * **Security:** Create dedicated, non-root users and set more restrictive permissions (e.g., read-only access for certain shares).
  * **Performance:** Optimize the `smb.conf` settings for faster read/write speeds.
