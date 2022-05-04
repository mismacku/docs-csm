# Install CSM

## Abstract

Installation of the CSM product stream has many steps in multiple procedures which should be done in a
specific order. Information about the HPE Cray EX system and the site is used to prepare the configuration
payload. The initial node used to bootstrap the installation process is called the PIT node because the
Pre-Install Toolkit (PIT) is installed there.

Once the management network switches have been configured, the other
management nodes can be deployed with an operating system and the software to create a Kubernetes cluster
utilizing Ceph storage. The CSM services provide essential software infrastructure including the API gateway
and many micro-services with REST APIs for managing the system. Once administrative access has been configured,
the installation of CSM software can be validated with health checks before doing operational tasks
like the checking and updating of firmware on system components or the preparation of compute nodes.

Once the CSM installation has completed, other product streams for the HPE Cray EX system can be installed.

## Topics

1. [Validate SHCD](#1-validate-shcd)
1. [Bootstrap PIT Node](#2-bootstrap-pit-node)
1. [Configure Management Network Switches](#3-configure-management-network-switches)
1. [Prepare Management Nodes](#4-prepare-management-nodes)
1. [Deploy Management Nodes](#5-deploy-management-nodes)
1. [Install CSM Services](#6-install-csm-services)
1. [Validate CSM Health Before Final NCN Deployment](#7-validate-csm-health-before-final-ncn-deployment)
1. [Deploy Final NCN](#8-deploy-final-ncn)
1. [Configure Administrative Access](#9-configure-administrative-access)
1. [Validate CSM Health](#10-validate-csm-health)
1. [Configure Prometheus Alert Notifications](#11-configure-prometheus-alert-notifications)
1. [Update Firmware with FAS](#12-update-firmware-with-fas)
1. [Prepare Compute Nodes](#13-prepare-compute-nodes)
1. [Next Topic](#next-topic)
1. [Troubleshooting Installation Problems](#troubleshooting-installation-problems)

The topics in this chapter need to be done as part of an ordered procedure so are shown here with numbered topics.

**`Note`**: If problems are encountered during the installation, some topics have their own [troubleshooting sections found in the operations index](../operations/index.md)
sections, but there is also a general troubleshooting topic.

## Details

<a name="1-validate-shcd"></a>
### 1. Validate SHCD

The cabling should be validated between the nodes and the management network switches. The information in the
Shasta Cabling Diagram (SHCD) can be used to confirm the cables which physically connect components of the system.
Having the data in the SHCD which matches the physical cabling will be needed later in both
[Bootstrap PIT Node](#2-bootstrap-pit-node) and [Configure Management Network](#3-configure-management-network-switches).

> **Note**: If a reinstall or fresh install of this software release is being done on this system and the management
> network cabling has already been validated, then this topic could be skipped and instead move to
> [Bootstrap PIT Node](#2-bootstrap-pit-node).

See [Validate SHCD](../operations/network/management_network/validate_shcd.md).

<a name="2-bootstrap-pit-node"></a>
### 2. Bootstrap PIT Node

The Pre-Install Toolkit (PIT) node needs to be bootstrapped from the LiveCD.

See [Bootstrapping the RemoteISO](bootstrap_livecd.md).

<a name="3-configure-management-network-switches"></a>
### 3. Configure Management Network Switches

Now that the PIT node has been booted with the LiveCD environment and CSI has generated the switch IP addresses,
the management network switches can be configured.

See [Management Network User Guide](../operations/network/management_network/index.md).

<a name="4-prepare-management-nodes"></a>
### 4. Prepare Management Nodes

```
TODO: This section is important for reinstalls; refactor this, ensuring Deploy Management Nodes doesn't have duplicated steps.
```

Some preparation of the management nodes might be needed before starting an install or reinstall.
The preparation includes checking and updating the firmware on the PIT node, quiescing the compute nodes
and application nodes, scaling back DHCP on the management nodes, wiping the storage on the management nodes,
powering off the management nodes, and possibly powering off the PIT node.

See [Prepare Management Nodes](prepare_management_nodes.md).

<a name="5-deploy-management-nodes"></a>
### 5. Deploy Management Nodes

Now that the PIT node has been booted with the LiveCD and the management network switches have been configured,
the other management nodes can be deployed. This procedure will boot all of the management nodes, initialize
Ceph storage on the storage nodes, and start the Kubernetes cluster on all of the worker nodes and the master nodes,
except for the PIT node. The PIT node will join Kubernetes after it is rebooted later in
[Deploy Final NCN](#8-deploy-final-ncn).

See [Deploy Management Nodes](deploy_management_nodes.md).

<a name="6-install_csm_services"></a>
### 6. Install CSM Services

Now that deployment of management nodes is complete with initialized Ceph storage and a running Kubernetes
cluster on all worker and master nodes, except the PIT node, the CSM services can be installed. The Nexus
repository will be populated with artifacts; containerized CSM services will be installed; and a few other configuration steps taken.

See [Install CSM Services](install_csm_services.md).

<a name="7-validate-csm-health-before-final-ncn-deployment"></a>
### 7. Validate CSM Health Before Final NCN Deployment

After installing all of the CSM services, now validate the health of the management nodes and all CSM services.
The reason to do it now is that if there are any problems detected with the core infrastructure or the nodes, it is
easy to rewind the installation to [Deploy Management Nodes](#5-deploy-management-nodes) because the PIT node has not
yet been redeployed. In addition, redeploying the PIT node successfully requires several CSM services to be working
properly, so validating this is important.

See [Validate CSM Health](../operations/validate_csm_health.md).

<a name="8-deploy-final-ncn"></a>
### 8. Deploy Final NCN

Now that all CSM services have been installed and the CSM health checks completed, with the possible exception
of Booting the CSM Barebones Image and the UAS/UAI tests, the PIT node can be rebooted to leave the LiveCD
environment and assume its intended role as one the Kubernetes master nodes.

See [Deploy Final NCN](deploy_final_ncn.md).

<a name="9-configure-administrative-access"></a>
### 9. Configure Administrative Access

Now that all of the CSM services have been installed and the PIT node has been redeployed, administrative access
can be prepared. This may include configuring Keycloak with a local Keycloak account or confirming Keycloak
is properly federating LDAP or other Identity Provider (IdP), initializing the 'cray' CLI for administrative
commands, locking the management nodes from accidental actions such as firmware updates by FAS or power actions by
CAPMC, configuring the CSM layer of configuration by CFS in NCN personalization,and configuring the node BMCs (node
controllers) for nodes in liquid cooled cabinets.

See [Configure Administrative Access](configure_administrative_access.md).

<a name="10-validate-csm-health"></a>
### 10. Validate CSM Health

Now that all management nodes have joined the Kubernetes cluster, CSM services have been installed,
and administrative access has been enabled, the health of the management nodes and all CSM services
should be validated. There are no exceptions to running the tests--all can be run now.

This CSM health validation can also be run at other points during the system lifecycle, such as when replacing
a management node, checking the health after a management node has rebooted because of a crash, as part of doing
a full system power down or power up, or after other types of system maintenance.

See [Validate CSM Health](../operations/validate_csm_health.md).

<a name="11-configure-prometheus-alert-notifications"></a>
### 11. Configure Prometheus Alert Notifications

Now that CSM has been installed and health has been validated, if the system management health monitoring tools and specifically,
Prometheus, are found to be useful, email notifications can be configured for specific alerts defined in Prometheus.
Prometheus upstream documentation can be leveraged for an [Alert Notification Template Reference](https://prometheus.io/docs/alerting/latest/notifications/)
as well as [Notification Template Examples](https://prometheus.io/docs/alerting/latest/notification_examples/). Currently supported notification
types include Slack, Pager Duty, email, or a custom integration via a generic webhook interface.

See [Configure Prometheus Email Alert Notifications](../operations/system_management_health/Configure_Prometheus_Email_Alert_Notifications.md) for example
configuration of an email alert notification for Postgres replication alerts that are defined on the system.

<a name="12-update-firmware-with-fas"></a>
### 12. Update Firmware with FAS

Now that all management nodes and CSM services have been validated as healthy, the firmware on other
components in the system can be checked and updated. The Firmware Action Service (FAS) communicates
with many devices on the system. FAS can be used to update the firmware for all of the devices it
communicates with at once, or specific devices can be targeted for a firmware update.

> **IMPORTANT:**
>  Before FAS can be used to update firmware, refer to the 1.5 _HPE Cray EX System Software Getting Started Guide S-8000_
>  on the HPE Customer Support Center at https://www.hpe.com/support/ex-gsg for more information about how to install
>  the HPE Cray EX HPC Firmware Pack (HFP) product. The installation of HFP will inform FAS of the newest firmware
>  available. Once FAS is aware that new firmware is available, then see
>  [Update Firmware with FAS](../operations/firmware/Update_Firmware_with_FAS.md).

<a name="13-prepare-compute-nodes"></a>
### 13. Prepare Compute Nodes

After completion of the firmware update with FAS, compute nodes can be prepared. Some compute node
types have special preparation steps, but most compute nodes are ready to be used now.

These compute node types require preparation.
   * HPE Apollo 6500 XL645d Gen10 Plus
   * Gigabyte

See [Prepare Compute Nodes](prepare_compute_nodes.md).

<a name="next-topic"></a>
## Next Topic

After completion of the firmware update with FAS and the preparation of compute nodes, the CSM product stream has
been fully installed and configured. Refer to the _HPE Cray EX System Software Getting Started Guide S-8000_
on the HPE Customer Support Center at https://www.hpe.com/support/ex-gsg for more information on other product streams to be installed and configured after CSM.

<a name="troubleshooting-installation"></a>
## Troubleshooting Installation Problems

The installation of the Cray System Management (CSM) product requires knowledge of the various nodes and
switches for the HPE Cray EX system. The procedures in this section should be referenced during the CSM install
for additional information on system hardware, troubleshooting, and administrative tasks related to CSM.

See [Troubleshooting Installation Problems](troubleshooting_installation.md).
