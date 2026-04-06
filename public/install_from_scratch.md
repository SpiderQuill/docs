# Installing Quill OS from Scratch

This guide provides a step-by-step walkthrough for installing Quill OS on a PineNote device from source code.

**DISCLAIMER:** Quill OS for the PineNote is in its early stages of development. Expect rough edges, missing features (e.g., screen rotation), and potential bugs. This installation process is complex and carries risks, including the possibility of data loss (if you don't follow it correctly) or a temporarily non-functional device. Proceed with caution and at your own risk.

---
## Part 1: Prerequisites & Environment Setup
### Hardware & Software Requirements
- **Linux machine**. [Fedora](https://www.fedoraproject.org/kde/) and [Ubuntu](https://ubuntu.com/download/desktop) have been tested to work. If you're using a [Windows](https://www.virtualbox.org/wiki/Downloads) or [MacOS](https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion) devices it's recommended that you use a Virtual Machine.
- A [**PineNote**](https://pine64.com/product/pinenote-community-edition-coming-soon/) and its **UART serial adapter**.
-  [**Visual Studio Code**](https://code.visualstudio.com/docs/setup/linux), VSCodium doesn't seem to work properly at this time
-  [**Docker Engine**](https://docs.docker.com/engine/install/), Ensure your user account is in the `docker` group
-  Around **100 GB** of free disk space for backups, the build environment, and source code.
- A Magnet

### Step 1: Configure Host System for ARM Emulation

The build process uses QEMU to run ARM binaries (but it tries to natively cross compile most things). You must enable `binfmt` on your host machine.

On Debian/Ubuntu based systems: 
```bash
sudo apt-get update
sudo apt-get install git binfmt-support qemu-user-static -y
```

On Fedora/Rocky based systems:
```bash
sudo dnf update
sudo dnf install binfmt-dispatcher git -y
```

### Step 2: Clone the Quillstrap Repository

`quillstrap` is the bootstrap tool used to build and deploy Quill OS.

```bash
git clone https://github.com/PorQ-Pine/quillstrap
```

### Step 3: Set Up the VSCode Development Container

The entire build process takes place inside a managed Docker container.

1.  **Install the Dev Containers Extension:** Open VSCode, go to the Extensions view, and search for and install `ms-vscode-remote.remote-containers`. You can also run `code --install-extension ms-vscode-remote.remote-containers`
2.  **Open the Project:** In VSCode, select `File > Open Folder` and choose the `quillstrap` directory you just cloned. You can also run `code .` in the quillstrap directory
3.  **Reopen in Container:** VSCode should detect the `.devcontainer` configuration and show a notification. Click "Reopen in Container". If it doesn't, open the Command Palette (`Ctrl+Shift+P`), type `Reopen in Container`, and select the corresponding command.
4.  **Wait for the Build:** The first time you do this, Docker will pull the base image and build the development environment. This can take a significant amount of time.
5. **Download Source Code**: Once the Docker environment has been built, you will need to run 2 more scripts, **First Time Setup** and **Welcome Back**. there are 2 buttons that you can find in the bottom left hand corner of VSCode that will let you run those scripts, otherwise they can be located in `/workspaces/quillstrap/other/setup` 

Once complete, your VSCode window will be connected to a terminal running inside the Docker container. All subsequent commands should be run in this VSCode terminal unless specified otherwise.

---
## Part 2: Backing Everything Up

With the environment ready, the next step is to make sure everything is properly backed up. You will lose your data if you don't do this properly, and this is needed create the boot partition.

If you're having a hard time getting this to work, Please take a look at [this video](https://www.youtube.com/watch?v=AFbMoCCuCwQ) by Carbonated Caffeine to have a better  overview of what's going on.
### Step 1: Booting into RockUSB mode
You're going to need to boot your PineNote into RockUSB mode to be able to extract your waveform partition. You will Here are the steps you need to take:

1. If you have your PineNote plugged into your computer, unplug it
2. Power off your PineNote
3. Once It's powered off, flip your PineNote so it's screen is facing down
4. Locate the small circle in the upper right hand corner of your PineNote.
5. Place the magnet on top of the small circle. 
6. Plug your PineNote into you computer

This will activate RockUSB mode on your PineNote and will let you use `rkdeveloptool` to back everything up. If you run `lsusb` you should now see:
`ID 2207:350a Fuzhou Rockchip Electronics Company USB download gadget`
If you see anything else you need to reboot your PineNote and try the steps again to get it into the proper mode. to do so, just hold down the power button for 30 seconds, and then give it a quick press once again to get it to boot. that should reset the device and you'd be able to retry the steps once again. 

Note: the PineNote did not play very nicely with the virtual machine that I was using for demo purposes, so I'm reiterating again that It's the best if you use a bare metal Linux machine. 

### Step 2: Run Backups
Once that is complete you are able to backup your PineNote. Run the following commands in the terminal to be able to back everything up

```bash
  # create backups (do this BEFORE altering any of the partitions!)
  # If you already altered partitions, skip this step and write back
  # partition backup you did before in the later step
  mkdir backups
  rkdeveloptool read 0 41943040 backups/first_40mb_of_disc.img
  rkdeveloptool read-partition boot backups/part_boot.img
  rkdeveloptool read-partition trust backups/part_trust.img
  rkdeveloptool read-partition waveform backups/part_waveform.img
  rkdeveloptool read-partition uboot backups/part_uboot.img
  rkdeveloptool read-partition logo backups/part_logo.img
  rkdeveloptool read-partition recovery backups/part_recovery.img
```

---
## Part 3: Building the Core OS Components

With the environment ready, the next step is to get and compile things
### The `rq` Command

The `quillstrap` repository contains a helper script named `rq`. Key flags include: 
-  `-g`: **Get**
-  `-b`: **Build**
-  `-d`: **Deploy**
-  `-c`: **Clean**
-  `-r`: **Run**
-  `-i`: **Ignore built checks**
-  `-a`: **Auto mode**

### Step 1: Initial Build

Depending on the power of your machine, these steps could take hours

```bash
# Get everything
rq -g all
rq -a -i -b uboot # this will build the U-Boot system
rq -a -i -b rootfs # this builds the root filesystem, 
# Building the rootfs can take hours depending on your system
# You can continue with the rest of the build 
# while this is running in the background 
```

You can verify that the u-boot components are built using:

```bash
rq --is-built uboot
```

This should report that everything it lists is built.

---
## Part 4: Flashing the New U-Boot

This is the first risky step. You will replace the device's stock bootloader.  Ensure your PineNote is well-charged.
### Step 1: Run the U-Boot Deploy Command

The `rq -d uboot` command uses `rkdeveloptool` to flash the new bootloader. It's an interactive process that will guide you.

```bash
rq -d uboot
```

Follow the on-screen instructions precisely. They will tell you when to connect the UART adapter, when to connect the USB cable, and how to put the device into Rockchip's "Download Mode".

### Troubleshooting: `rkdeveloptool` Connection Issues

It is very common to have trouble getting the PineNote to be recognized by `rkdeveloptool`.

-  **"No Rockchip device found":** This is the most common issue.
    -  **Cable & Ports:** Try different USB ports and cables. Some cables are for charging only and don't carry data.
    -  Try connecting another device with a usb port next to the pinenote connected one. Idk, weird fix but worked for someone

-  **Laggy/Unresponsive UART:** If you can't type commands reliably in the U-Boot console:
    -  **Reboot Your Host PC:** This often resolves issues with serial port drivers.
    -  **Exclusive Access:** Ensure no other programs (like `minicom`, another VSCode terminal, etc.) are using the serial device (`/dev/ttyUSB0`).
    -  **Clean Contacts:** Use contact cleaner on the UART adapter USB port and PineNote

### Step 2: Verify the Flash

After `rq -d uboot` completes successfully, reboot the PineNote. Remove the Magnet, and press down the Power button for 30 seconds, and then press it for 1 second. You should be greeted with a new, "prettier" boot screen featuring the Quill OS logo. This confirms the new U-Boot is installed.

---
## Part 5: Backing Up and Re-Partitioning the eMMC

This is the second, and most destructive, risky step. You will back up your entire device memory and then wipe most of it. **DO NOT SKIP THE BACKUP.**

### Step 1: Boot into UMS (USB Mass Storage) Mode

1. Unplug the PineNote from your computer 
2. Reboot the PineNote.
3.  In the U-Boot menu,  navigate  to  the **`UMS kernel`** option.
4.  Wait for 10 seconds, then connect it to your computer with a USB cable. The internal eMMC will now be exposed as a block device (e.g., `/dev/sdb`).

**Important Note on Auto-Mounting:**
Modern Linux desktops (especially Ubuntu) will automatically mount the device's partitions. This **will interfere** with the backup and partitioning scripts. You must unmount them.
-  Use your file manager to "Eject" each mounted PineNote partition.
-  Alternatively, use the `umount` command from your host system's terminal (e.g., `umount /dev/sdb1`).
-  Verify with `lsblk` on the host to ensure no partitions on the PineNote device have a `MOUNTPOINT`.

### Step 2: Create a Full eMMC Backup

This will create a compressed image of your device's entire 128 GB storage. It will take 2-3 hours and consume around ~20 GB of disk space. (Depends on storage usage on your PineNote)

```bash
# In the VSCode terminal, run:
rq -r backup # Takes backups of the most important partitions
rq -r backup_mmc # The real backup
```

The tool will list available block devices. Choose the one corresponding to your PineNote (e.g., `sdb`). The script will then create `quillstrap/build_all/low/backup_mmc/pinenote_disk.qcow2`. **Keep this file in a safe place. Or just don't touch it, don't remote the parent directories** It is the only easy way to restore your device to its original state.

### Step 3: Partitioning
Don't assume anything in the PineNote storage will survive this step. That's why we have taken a full backup

### Automatic

The automatic partitioning script (`rq -r partition_setup`) assumes a stock partition layout. If you have ever installed another OS or modified the partitions, it will likely fail. While it has failchecks implemented, it's still risky.

It also assummes the partitions are not much used in case of storage (as `os1` will be resized to be smaller, `data` partitions also needs to be unused). If you still want to continue:

1.  Boot the PineNote into **UMS mode** and connect it to your host computer. Ensure no partitions are mounted.
2. Run `rq -r partition_setup`
3. Proceed to the "Manually" section which tells you how to verify if the partitioning succeeded

### Manually

1.  Boot the PineNote into **UMS mode** and connect it to your host computer. Ensure no partitions are mounted.
2.  On your **host machine** (not the VSCode container), open a partitioning tool like `gparted`. You may need to install it: `sudo apt-get install gparted` or `sudo dnf install parted`.
3.  In `gparted`, select the PineNote device (e.g., `/dev/sdX`).
4.  You will see the existing partition layout. **DO NOT TOUCH THE FIRST FOUR PARTITIONS (`uboot`, `waveform`, `uboot_env`, `logo`).**
5.  Delete the existing `os2`, and `data` partitions. (`os1` can be moved and resized, then still be usable)
6.  Create the new partitions according to the exact layout below. You must set the "Name" field correctly for each partition.

| Partition Name   | File System | Size      | Notes                             |
|------------------|-------------|-----------|-----------------------------------|
| `os1`            | `ext4`      | 14.6 GiB  | Kept for a potential legacy OS.   |
| `data`           | `ext4`      | 10.0 GiB  | Data partition for the legacy OS. |
| `quill_boot`     | `ext4`      | 256.0 MiB | Quill OS boot partition.          |
| `quill_recovery` | `ext4`      | 10.0 GiB  | Quill OS recovery system.         |
| `quill_main`     | `ext4`      | ~80.0 GiB | Main Quill OS root filesystem.    |

7.  Apply the changes in `gparted`. This will permanently erase the data on the old partitions.

### Using rkdeveloptool

1. Download the [rk356x_spl_loader](https://github.com/PNDeb/pinenote-debian-image/releases/download/v20250101_01/rk356x_spl_loader_v1.12.112.bin) and [partition_table_dev.txt](partition_table_dev.txt)
2. Boot the PineNote into RockUSB mode either through the magnet method or using the U-Boot menu
3. Run these commands to get the partition table flashed onto your PineNote
```bash
# enable `write-partition-table` commands
rkdeveloptool reboot-maskrom
rkdeveloptool boot rk356x_spl_loader_v1.12.112.bin
# write new GPT partition table
rkdeveloptool write-partition-table partition_table_dev.txt
# enable looking at the partiton table again
rkdeveloptool reboot
# verify that the partiton table is written
rkdeveloptool list-partitions
```
4.  You're now going to need to write the ext4 filesystem to the partitons to be able to go on to the next steps 
```bash
mkfs.ext4 /dev/sdX6 # data
mkfs.ext4 /dev/sdX7 # quill_boot
mkfs.ext4 /dev/sdX8 # quill_recovery
mkfs.ext4 /dev/sdX9 # quill_main
```


---
## Part 6: Building, Deploying and Booting Quill OS

Now you will deploy the built OS components to the newly created partitions.

### Step 1: Build and Deploy the Boot Partition

This step prepares the files needed to boot into recovery and the main OS.

```bash
# Build the boot partition components
rq -a -i -b boot_partition
# this step depends on backing up your partitons,
# if you're running into any errors make sure you've 
# ran through the eMMC Backup (rq -r backup)
```

Now, put the PineNote in **UMS mode**, ensure partitions are unmounted, and run:
```bash
# Deploy the files to the quill_boot partition
rq -d boot_partition
```

### Step 2: Test the Recovery System

1.  Connect the UART serial adapter to your PineNote and computer. Open a serial monitor to view the output (you can do that via `rq -r serial`).
2.  Reboot the PineNote.
3.  In the U-Boot menu, select **`Boot quill os recovery`**.
4.  A simple GUI should appear on the PineNote screen. This confirms that your `uboot` and `quill_boot` partitions are working correctly.

### Step 3: Built, Deploy the Main Root Filesystem

This is the final deployment step. First let's build stuff

This can take hours, depending on your host machine performance
```bash
# if you have already run this, try running it without the -i to speed things up
rq -a -i -b rootfs 
```

Put the PineNote into **UMS mode** again and deploy the final filesystem.
```bash
# Deploy the root filesystem to the quill_main partition
rq -d rootfs
```

### Step 4: The First Boot

(There is a bug on first boot that could fail to show the login screen, simply force reboot the device via the power button)

You are now ready to boot into Quill OS.

1.  Keep the UART serial adapter connected and monitored.
2.  Reboot the PineNote.
3.  From the U-Boot menu, select the default **`Boot Quill OS`** option.
4.  Observe the UART output. The device will run through its first-time boot sequence.
5.  On the PineNote's screen, you should see the Quill OS logo with three animating dots below it.
6.  The UART output will eventually present a login prompt.

Congratulations! You have successfully installed Quill OS from scratch.

---
## Part 7: Troubleshooting First Boot and Initial Login

After the final reboot, the device will display the Quill OS logo, and the UART serial console will present a login prompt. If there are errors, initial interaction with the OS happens through this serial connection.
### Logging In

You will need to use a serial terminal application (we use `rq -r serial`) connected to your UART adapter to log in. (Just like you monitored it before)

The default credentials are `root` and `root`
### Initial Setup

You need to create a user manually

```bash
# Actually create that user:
export USER=<username you want to use>
useradd $USER
passwd $USER

# (Now we need to copy skel there)
gocryptfs /home/.$USER /home/$USER
cp -r /etc/skel/.* /home/$USER/
sudo chown -R szybet:$USER /home/$USER # Important permissions
umount /home/$USER
```

Then reboot and log in via the GUI

### Setting the Timezone

In /boot/boot_config.ron, set the timezone string to "America/New_York" or any other relevant timezone. 
