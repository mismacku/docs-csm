# Bootstrap PIT Node from LiveCD USB

The Pre-Install Toolkit (PIT) node needs to be bootstrapped from the LiveCD. There are two media available
to bootstrap the PIT node: the RemoteISO or a bootable USB device. This procedure describes using the USB
device. If not using the USB device, see [Bootstrap PIT Node from LiveCD Remote ISO](bootstrap_livecd_remote_iso.md).

These steps provide a bootable USB with SSH enabled, capable of installing this CSM release.

## Topics

1. [Create the Bootable Media](#create-the-bootable-media)
1. [Boot the LiveCD](#boot-the-livecd)
   1. [First Login](#first-login)
1. [Configure the Running LiveCD](#configure-the-running-livecd)
1. [Next Topic](#next-topic)

<a name="download-and-expand-the-csm-release"></a>

## 1. Download and Expand the CSM Release

Fetch the base installation CSM tarball, extract it, and install the contained CSI tool.

1. Create a working area for this procedure:

   ```bash
   linux# mkdir usb
   linux# cd usb
   ```

1. Set up the initial typescript.

   ```bash
   linux# SCRIPT_FILE=$(pwd)/csm-install-usb.$(date +%Y-%m-%d).txt
   linux# script -af ${SCRIPT_FILE}
   linux# export PS1='\u@\H \D{%Y-%m-%d} \t \w # '
   ```

<a name="create-the-bootable-media"></a>

## 2. Create the Bootable Media

Cray Site Init will create the bootable LiveCD. Before creating the media, identify
which device will be used for it.

1. Identify the USB device.

    This example shows the USB device is `/dev/sdd` on the host.

    ```bash
    linux# lsscsi
    ```

    Expected output looks similar to the following:

    ```text
    [6:0:0:0]    disk    ATA      SAMSUNG MZ7LH480 404Q  /dev/sda
    [7:0:0:0]    disk    ATA      SAMSUNG MZ7LH480 404Q  /dev/sdb
    [8:0:0:0]    disk    ATA      SAMSUNG MZ7LH480 404Q  /dev/sdc
    [14:0:0:0]   disk    SanDisk  Extreme SSD      1012  /dev/sdd
    [14:0:0:1]   enclosu SanDisk  SES Device       1012  -
    ```

    In the above example, internal disks are the `ATA` devices and USB drives are final two devices.

    Set a variable with your disk to avoid mistakes:

    ```bash
    linux# USB=/dev/sd<disk_letter>
    ```

1. Format the USB device.

   **TODO:** Step to copy `write-livecd.sh` into place.
    * On Linux, use the CSI application to do this:

        ```bash
        linux# write-livecd.sh ${USB} ${CSM_PATH}/cray-pre-install-toolkit-*.iso 50000
        ```

<a name="boot-the-livecd"></a>
## 3. Boot the LiveCD

Some systems will boot the USB device automatically if no other OS exists (bare-metal). Otherwise the
administrator may need to use the BIOS Boot Selection menu to choose the USB device.

If an administrator has the node booted with an operating system which will next be rebooting into the LiveCD,
then use `efibootmgr` to set the boot order to be the USB device. See the
[set boot order](../background/ncn_boot_workflow.md#set-boot-order) page for more information about how to set the
boot order to have the USB device first.

> **Note:** UEFI booting must be enabled in order for the system to find the USB device's EFI bootloader.

1. Confirm that the IPMI credentials work for the BMC by checking the power status.

   Set the `BMC` variable to the hostname or IP address of the BMC of the PIT node.

   > `read -s` is used in order to prevent the credentials from being displayed on the screen or recorded in the shell history.

   ```bash
   external# BMC=eniac-ncn-m001-mgmt
   external# read -s IPMI_PASSWORD
   external# export IPMI_PASSWORD ; ipmitool -I lanplus -U root -E -H ${BMC} chassis power status
   ```

1. Power the NCN on and connect to the IPMI console.

   > **`NOTE`** The bootdevice can be set via IPMI, below we use the `floppy` option. At a glance this seems incorrect,
   > however it selects the primary removable media. This step instructs the user to power off the node to ensure 
   > the BIOS has the best chance at finding the USB via a cold-boot.
   > 
   > ```bash
   > external# ipmitool chassis bootdev
   >    Received a response with unexpected ID 0 vs. 1
   >    bootdev <device> [clear-cmos=yes|no]
   >    bootdev <device> [options=help,...]
   >    none  : Do not change boot device order
   >    pxe   : Force PXE boot
   >    disk  : Force boot from default Hard-drive
   >    safe  : Force boot from default Hard-drive, request Safe Mode
   >    diag  : Force boot from Diagnostic Partition
   >    cdrom : Force boot from CD/DVD
   >    bios  : Force boot into BIOS Setup
   >    floppy: Force boot from Floppy/primary removable media
   > ```

   ```bash
   external# ipmitool -I lanplus -U root -E -H ${BMC} chassis bootdev floppy options=efiboot
   external# ipmitool -I lanplus -U root -E -H ${BMC} chassis power off
   ```
   
1. Insert the USB stick into a USB 3 port (USB2 is compatible, USB3 offers the best performance).

1. Power the server on
   
   ```bash
   external# ipmitool -I lanplus -U root -E -H ${BMC} chassis power on
   external# ipmitool -I lanplus -U root -E -H ${BMC} sol activate
   ```

1. Do not exit the typescript. After completing this procedure, proceed to [First Login](./bootstrap_livecd.md#3-first-login).
