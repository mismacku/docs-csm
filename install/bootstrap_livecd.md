# Bootstrap PIT Node from LiveCD Remote ISO

The Pre-Install Toolkit (PIT) node needs to be bootstrapped from the LiveCD. There are two media available
to bootstrap the PIT node: the RemoteISO or a bootable USB device. This procedure describes using the USB
device.

This page will walk one through setting up storage on the NCN to use as a staging area, by default an arbitrary disk will be used. One may also use a USB stick (removable storage) to accomplish this. Using a local disk entails a necessary wipe prior to [deploying the final NCN](deploy_final_ncn.md) after the CSM install.

**Important:** Before starting this procedure be sure to complete the procedure to
[Prepare Configuration Payload](prepare_configuration_payload.md) for the relevant installation scenario.

## Topics

1. [Attaching and Booting the LiveCD with the BMC](#1-attaching-and-booting-the-livecd-with-the-bmc)
1. [First Login](#2-first-login)
1. [Configure the Running LiveCD](#3-configure-the-running-livecd)
   1. [Load the CSM Artifacts](#31-load-the-csm-artifacts)
   1. [Generate Installation Files](#32-generate-installation-files)
   1. [Prepare Site Init](#33-prepare-site-init)
1. [Start PIT Services and Validate PIT Health](#4-start-pit-services-and-validate-pit-health)
1. [Next Topic](#next-topic)

<a name="1-attaching-and-booting-the-livecd-with-the-bmc"></a>
## 1. Attaching and Booting the LiveCD with the BMC

Obtain and attach the LiveCD `cray-pre-install-toolkit` ISO file to the BMC. Depending on the vendor of the node,
the instructions for attaching to the BMC will differ.

1. Download the CSM software release and extract the LiveCD remote ISO image.

   **Important:** Ensure that you have the CSM release plus any patches or hotfixes by
   following the instructions in [Update CSM Product Stream](../update_product_stream/index.md)

   The `cray-pre-install-toolkit` ISO and other files are now available in the directory from the extracted CSM tar file.
   The ISO will have a name similar to
   `cray-pre-install-toolkit-sle15sp3.x86_64-1.5.8-20211203183315-geddda8a.iso`

   This ISO file can be extracted from the CSM release tar file using the following command:
   ```bash
   linux# tar --wildcards --no-anchored -xzvf <csm-release>.tar.gz 'cray-pre-install-toolkit-*.iso'
   ```

1. See the respective procedure below to attach an ISO and boot.

   1. Start a typescript on an external system.

   This will record this section of activities done on the console of `ncn-m001` using IPMI.

   ```bash
   external# script -a boot.livecd.$(date +%Y-%m-%d).txt
   external# export PS1='\u@\H \D{%Y-%m-%d} \t \w # '
   ```

   1. Follow one of the respective procedures below:

   - **HPE iLO BMCs** Prepare a server on the network to host the `cray-pre-install-toolkit` ISO file. Then follow the [HPE iLO BMCs](../operations/boot_livecd_remoteiso.md#hpe-ilo-bmcs) to boot the RemoteISO.
   - **Gigabyte BMCs** must create a USB stick by following [Bootstrap a LiveCD USB](bootstrap_livecd_usb.md#3-boot-the-livecd).
   - **Intel BMCs** must create a USB stick by following [Bootstrap a LiveCD USB](bootstrap_livecd_usb.md#3-boot-the-livecd).

1. The procedure from the previous step will now have the user in a console observing the server boot.

<a name="2-first-login"></a>
## 2. First Login

On first login the LiveCD will prompt the administrator to change the password.

1. **The initial password is empty**; enter the username of `root` and press `return` twice.

   ```
   pit login: root
   ```

   Expected output looks similar to the following:

   ```
   Password:           <-------just press Enter here for a blank password
   You are required to change your password immediately (administrator enforced)
   Changing password for root.
   Current password:   <------- press Enter here, again, for a blank password
   New password:       <------- type new password
   Retype new password:<------- retype new password
   Welcome to the CRAY Pre-Install Toolkit (LiveOS)
   ```

1. Set up the site-link, enabling SSH to work. You can reconnect with SSH after this step.

   > **Note:** If your site's network authority or network administrator has already provisioned a DHCP IPv4 address for your master node's external NIC(s), **then skip this step**.

   1. Set networking variables.

      > If you have previously created the `system_config.yaml` file for this system, the values for these variables are in it. The
      > following table lists the variables being set, their corresponding `system_config.yaml` fields, and a description of what
      > they are.

      | Variable    | `system_config.yaml`   | Description                                                                        |
      |-------------|------------------------|------------------------------------------------------------------------------------|
      | `site_ip`   | `site-ip`              | The IPv4 address **and CIDR netmask** for the node's external interface(s)         |
      | `site_gw`   | `site-gw`              | The IPv4 gateway address for the node's external interface(s)                      |
      | `site_dns`  | `site-dns`             | The IPv4 domain name server address for the site                                   |
      | `site_nics` | `site-nic`             | The actual NIC name(s) for the external site interface(s)                          |

      > If the `system_config.yaml` file has not yet been generated for this system, the values for `site_ip`, `site_gw`, and
      > `site_dns` should be provided by the site's network administrator or network authority. The `site_nics` interface(s)
      > are typically the first onboard adapter or the first copper 1 GbE PCIe adapter on the PIT node. If multiple interfaces are
      > specified, they must be separated by spaces (for example, `site_nics='p2p1 p2p2 p2p3'`).

      ```bash
      pit# site_ip=172.30.XXX.YYY/20
      pit# site_gw=172.30.48.1
      pit# site_dns=172.30.84.40
      pit# site_nics=em1
      ```

   1. Run the `csi-setup-lan0.sh` script to set up the site link.

      > **Note:** All of the `/root/bin/csi-*` scripts are harmless to run without parameters; doing so will print usage statements.

      ```bash
      pit# /root/bin/csi-setup-lan0.sh $site_ip $site_gw $site_dns $site_nics
      ```

   1. Verify that `lan0` has an IP address

      The script appends `-pit` to the end of the hostname as a means to reduce the chances of confusing the PIT node with an actual, deployed NCN.

      ```bash
      pit# ip a show lan0
      ```
      
1. Mount the PITDATA partition using the **RemoteISO** directions. If the USB path was required, use the **USB** option below.

   - **RemoteISO** directions for using a local disk for PITDATA:
     ```bash
      pit# disk="$(lsblk -l -o SIZE,NAME,TYPE,TRAN | grep -E '(sata|nvme|sas)' | sort -h | awk '{print $2}' | head -n 1 | tr -d '\n')"
      pit# echo $disk
      pit# parted --wipesignatures -m --align=opt --ignore-busy -s /dev/$disk -- mklabel gpt mkpart primary ext4 2048s 100%
      pit# mkfs.ext4 -L PITDATA "/dev/${disk}1"
      pit# mount -vL PITDATA
      ```

   - **USB** directions for mounting the USB's data partition:

      ```bash
      pit# mount -vL PITDATA
      ```

1. Make the admin directory and logout.

   ```bash
   pit# mkdir -pv /var/www/ephemeral/admin
   pit# logout
   
   eniac-ncn-m001-pit login:
   ```

1. Exit the typescript
   
   ```bash
   pit# exit
   Script done.
   ```

<a name="3-configure-the-running-livecd"></a>
## 3. Configure the Running LiveCD

1. Copy the typescript and SSH to the PIT.

   ```bash
   external# scp boot.livecd.*.txt root@eniac-ncn-m001:/var/www/ephemeral/admin/ 
   external# ssh root@eniac-ncn-m001
   ```

1. Change to the `admin` directory, start a new typescript, and set the PS1.

   ```bash
   pit# cd /var/www/ephemeral/admin/
   pit# script -af ~/csm-install.$(date +%Y-%m-%d).txt
   pit# export PS1='\u@\H \D{%Y-%m-%d} \t \w # '
   ```

1. Print information about the booted PIT image for logging purposes.

   ```bash
   pit# /root/bin/metalid.sh
   ```

   Expected output looks similar to the following (the versions in the example below may differ). There should be **no** errors:

   ```
   = PIT Identification = COPY/CUT START =======================================
   VERSION=1.6.0
   TIMESTAMP=20220504161044
   HASH=g10e2532
   2022/05/04 17:08:19 Using config file: /var/www/ephemeral/prep/system_config.yaml
   CRAY-Site-Init build signature...
   Build Commit   : 0915d59f8292cfebe6b95dcba81b412a08e52ddf-main
   Build Time     : 2022-05-02T20:21:46Z
   Go Version     : go1.16.10
   Git Version    : v1.9.13-29-g0915d59f
   Platform       : linux/amd64
   App. Version   : 1.17.1
   metal-ipxe-2.2.6-1.noarch
   metal-net-scripts-0.0.2-20210722171131_880ba18.noarch
   metal-basecamp-1.1.12-1.x86_64
   pit-init-1.2.20-1.noarch
   pit-nexus-1.1.4-1.x86_64
   = PIT Identification = COPY/CUT END =========================================
   ```

1. Add helper variables to PIT node environment.

   > **Important:** All CSM install procedures on the PIT node assume that these variables are set
   > and exported.

   1. Set helper variables.

      ```bash
      pit# CSM_RELEASE=csm-x.y.z
      pit# PITDATA=$(lsblk -o MOUNTPOINT -nr /dev/disk/by-label/PITDATA)
      ```

   1. Add variables to the PIT environment for subsequent SSH (re)connections.

      By adding these to the `/etc/environment` file of the PIT node, these variables will be
      automatically set and exported in shell sessions on the PIT node.

      > The `echo` prepends a newline to ensure that the variable assignment occurs on a unique line,
      > and not at the end of another line.

      ```bash
      pit# echo "
      CSM_RELEASE=${CSM_RELEASE}
      PITDATA=${PITDATA}
      CSM_PATH=${PITDATA}/${CSM_RELEASE}" | tee -a /etc/environment
      ```

   1. Verify that expected environment variables are set in the new login shell.

      ```bash
      pit# echo -e "CSM_PATH=${CSM_PATH}\nCSM_RELEASE=${CSM_RELEASE}\nPITDATA=${PITDATA}\n"
      ```

<a name="31-load-the-csm-artifacts"></a>
### 3.1. Load the CSM Artifacts

1. Download the CSM software release to the PIT node.

   1. Set variable to URL of CSM tarball.

      ```bash
      pit# URL=https://arti.dev.cray.com/artifactory/shasta-distribution-stable-local/csm/${CSM_RELEASE}.tar.gz
      ```

   1. Fetch the release tarball.

      ```bash
      pit# wget ${URL} -O ${CSM_PATH}.tar.gz
      ```

   1. Expand the tarball on the PIT node.

      > **Note:** Expansion of the tarball may take more than 45 minutes.

      ```bash
      pit# tar -C ${PITDATA} -zxvf ${CSM_PATH}.tar.gz && ls -l ${CSM_PATH}
      ```

   1. Copy the artifacts into place.

      ```bash
      pit# rsync -a -P --delete ${CSM_PATH}/images/kubernetes/   ${PITDATA}/data/k8s/ &&
           rsync -a -P --delete ${CSM_PATH}/images/storage-ceph/ ${PITDATA}/data/ceph/
      ```

   > **Note:** The PIT ISO, Helm charts/images, and bootstrap RPMs are now available in the extracted CSM tar file.

1. Install/upgrade CSI; check if a newer version was included in the tarball.

   ```bash
   pit# rpm -Uvh $(find ${CSM_PATH}/rpm/ -name "cray-site-init-*.x86_64.rpm" | sort -V | tail -1)
   ```

1. Install the latest Goss Tests and Server.

   ```bash
   pit# rpm -Uvh --force $(find ${CSM_PATH}/rpm/ -name "goss-servers*.rpm" | sort -V | tail -1) \
                         $(find ${CSM_PATH}/rpm/ -name "csm-testing*.rpm" | sort -V | tail -1)
   ```

1. Install the latest documentation RPM.

   See [Check for Latest Documentation](../update_product_stream/index.md#check-for-latest-documentation)

1. Re-print the Metal ID to capture any differences in versions after loading the CSM tarball:

   ```bash
   pit# /root/bin/metalid.sh
   ```

   Expected output looks similar to the following:

   ```
   = PIT Identification = COPY/CUT START =======================================
   VERSION=1.6.0
   TIMESTAMP=20220504161044
   HASH=g10e2532
   2022/05/04 17:08:19 Using config file: /var/www/ephemeral/prep/system_config.yaml
   CRAY-Site-Init build signature...
   Build Commit   : 0915d59f8292cfebe6b95dcba81b412a08e52ddf-main
   Build Time     : 2022-05-02T20:21:46Z
   Go Version     : go1.16.10
   Git Version    : v1.9.13-29-g0915d59f
   Platform       : linux/amd64
   App. Version   : 1.17.1
   metal-ipxe-2.2.6-1.noarch
   metal-net-scripts-0.0.2-20210722171131_880ba18.noarch
   metal-basecamp-1.1.12-1.x86_64
   pit-init-1.2.20-1.noarch
   pit-nexus-1.1.4-1.x86_64
   = PIT Identification = COPY/CUT END =========================================
   ```

<a name="32-generate-installation-files"></a>
### 3.2. Generate Installation Files

Some files are needed for generating the configuration payload. See [Configuration Payload Files](prepare_configuration_payload.md#configuration-payload-files) topics if one has not already prepared the information for this system.

1. Make the prep directory

   ```bash
   pit# mkdir -pv $PITDATA/prep
   ```

1. **Unless** they're already available from a previous installation, follow the steps in [Configuration Payload Files](prepare_configuration_payload.md#configuration-payload-files) to create them:

   - `application_node_config.yaml` (required if Application Nodes exist)
   - `cabinets.yaml` (Required if multiple cabinents exist)
   - `hmn_connections.json`
   - `ncn_metadata.csv`
   - `switch_metadata.csv`
   - `system_config.yaml`

<a name="33-prepare-site-init"></a>
### 3.3. Prepare Site Init

> **Important:** Although the command prompts in this procedure are `linux#`, the procedure should be
> performed on the PIT node.

Prepare the `site-init` directory by performing the [Prepare Site Init](prepare_site_init.md) procedure.

<a name="4-start-pit-services-and-validate-pit-health"></a>
## 4. Start PIT Services and Validate PIT Health

1. Initialize the PIT.

   The `pit-init.sh` script will prepare the PIT server for deploying NCNs.

   ```bash
   pit# /root/bin/pit-init.sh
   ```

1. Invoke the pre-flight checks

   ```bash
   pit# csi pit validate --livecd-preflight
   ```

<a name="next-topic"></a>
## Next Topic

After completing this procedure, proceed to configure the management network switches.

See [Configure Management Network Switches](index.md#3-configure-management-network-switches).
