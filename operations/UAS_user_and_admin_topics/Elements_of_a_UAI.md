# Elements of a UAI

All UAIs can have the following attributes associated with them:

* A required container image
* An optional set of volumes
* An optional resource specification
* An optional collection of other configuration items

This topic explains each of these attributes.

## UAI container image

The container image for a UAI \([UAI image](UAI_Images.md)\) defines and provides the basic environment available to the user. This environment includes, among other things:

* The operating system \(including version\)
* Pre-installed packages

A site can [customize UAI images](Customize_End-User_UAI_Images.md) and add those images to UAS, allowing them to be used for UAI creation. Any number of UAI images can be configured in UAS, though only one will be used by any given UAI.

UAS provides two UAI images by default. These images enable HPE Cray EX administrators to set up UAIs and run many common tasks.
The first image is a standard [End-User UAI](End_User_UAIs.md) image that has the software necessary to support a basic Linux login experience.
This is primarily intended to give administrators a way to get started with UAIs and experiment with their configuration. The second image is a [Broker UAI](Broker_Mode_UAI_Management.md) image.
Broker UAIs present a single SSH endpoint that every user of a given class of UAIs logs into.
The Broker UAI then locates or creates a suitable End-User UAI and redirects the SSH session to that End-User UAI.

## UAI Volumes

The [volumes](Volumes.md) defined for a UAI provide external access to data provided by the host node. Anything that can be defined as a volume in a Kubernetes pod specification can be configured in UAS as a volume and used within a UAI. Examples include:

* Kubernetes ConfigMaps and Secrets
* External file systems used for persistent storage or external data access
* Host node files and directories

When UAIs are created they mount a list of volumes inside their containers to give them access to various data provided either by Kubernetes resources or through Kubernetes by the host node where the UAI runs.
Which volumes are in that list depends on how the UAI is created:

* UAIs created without using a [UAI class](UAI_Classes.md) mount all volumes configured in UAS
* UAIs created using a UAI Class mount only the volumes listed in the UAI Class and configured in UAS

The following are some example use cases for UAI volumes:

* Connecting UAIs to configuration files like /etc/localtime maintained by the host node
* Connect End-User UAIs to Slurm or PBS Professional Workload Manager configuration shared through Kubernetes
* Connecting End-User UAIs to Programming Environment libraries and tools hosted on the UAI host nodes
* Connecting End-User UAIs to Lustre or other external storage for user data
* Connecting Broker UAIs to a directory service or SSH configuration to authenticate and redirect user sessions

Every UAS volume includes the following values in its registration information:

* `mount_path`: Specifies where in the UAI the volume will be mounted.
* `volume_description`: A dictionary with one entry, whose key identifies the kind of Kubernetes volume is described \(for example, `host_path`, `configmap`, `secret`\).
  The value associated with that key is another dictionary containing the Kubernetes volume description itself.
* `volumename`: A required string chosen by the creator of the volume. This may describe or name the volume. It is used inside the UAI pod specification to identify the volume that is mounted in a given location in a container.
  A `volumename` is unique within any given UAI, but not necessarily within UAS. These are useful when searching for a volume if they are unique across the UAS configuration.
* `volume_id`: Used to identify the UAS volume when examining, updating, or deleting a volume and when linking a volume to a UAI class. The `volume_id` is generated by and unique within UAS.

Refer to [Kubernetes Documentation describing Volumes](https://kubernetes.io/docs/concepts/storage/volumes) for more information about Kubernetes volumes.

## Resource Specifications

A resource request tells Kubernetes the minimum amount of a given host node resource to give to each UAI. A resource limit sets the maximum amount of a given host node resource that Kubernetes can give to any UAI.
Kubernetes uses resource limits and requests to manage the system resources available to pods. Because UAIs run as pods under Kubernetes, UAS takes advantage of Kubernetes to manage the system resources available to UAIs.
In UAS, [resource specifications](Resource_Specifications.md) contain that configuration.
A UAI that is assigned a resource specification will use the resource requests and limits found there instead of the default resource limits or requests on the Kubernetes namespace containing the UAI.
This way, resource specifications can be used to fine-tune resources assigned to UAIs of different classes.

UAI resource specifications have three configurable parameters:

* A set of limits which is a JSON string describing Kubernetes resource limits
* A set of requests which is a JSON string describing Kubernetes resource requests
* An optional comment which is a free-form string containing any information an administrator might find useful about the resource specification

Resource specifications also contain a resource-id that is used for examining, updating, or deleting the resource specification as well as linking the resource specification into a UAI class.
Resource-ids are generated by UAS and unique to each resource specification.

Resource specifications configured in UAS contain resource requests, limits, or both, that can be associated with a UAI.
Any resource request or limit that can be set up on a Kubernetes pod can be set up as a resource specification under UAS.

## Other Configuration Items

There are also smaller configuration items that control things such as:

* Whether the UAI can talk to compute nodes over the high-speed network \(needed for workload management\)
* Whether the UAI presents a public facing or private facing IP address for SSH
* Kubernetes scheduling priority
* Timeout values for limiting the lifespan of active or idle UAIs

These items are configured in UAI Classes so only UAIs created from UAI Classes can have these settings.

## UAI Configuration and UAI Classes

All of the above described UAI configuration and more can be encapsulated into a [UAI Class](UAI_Classes.md), which can then be used to create UAIs with greater precision and efficiency.
UAI Classes are especially important when using [Broker UAIs](Broker_Mode_UAI_Management.md) because Broker UAIs use a UAI Creation Class configured into the Broker UAI's class to determine what kind of End-User UAI to create when a user logs into the Broker.

[Top: User Access Service (UAS)](index.md)

[Next Topic: UAI Host Nodes](UAI_Host_Nodes.md)
