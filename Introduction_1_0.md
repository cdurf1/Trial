# Introducing Intel® Manager for Lustre*

<a id="1.0"></a>
<a href="f1.0">Enterprises</a> and institutions of all sizes use high performance computing to solve today's most intense computing challenges. Just as compute clusters exploit parallel processors and development tools, storage solutions must be parallel to deliver the sustained performance at the large scales that today's applications require. The Lustre* file system is the ideal distributed, parallel file system for high performance computing. 

Accordingly, as storage solutions continue to grow in complexity, powerful, yet easy-to-use software tools to install, configure, monitor, manage, and optimize Lustre-based solutions are essential. Intel® Manager for Lustre* software is purpose-built to simplify the deployment and management of Lustre-based solutions. Intel® Manager for Lustre* software reduces management complexity and costs, enabling storage superusers to exploit the performance and scalability of Lustre storage, and accelerate critical applications and work flows.

Intel® Manager for Lustre* software greatly simplifies the creation and management of Lustre file systems, using either the graphical user interface (GUI) or a command line interface (CLI). The GUI dashboard lets you monitor one or more distributed Lustre file systems. Real-time storage-monitoring lets you track Lustre file system usage, performance metrics, events, and errors at the Lustre level. Plug-ins provided by storage solution providers enable monitoring of hardware-level performance data, disk errors and faults, and other hardware-related information. 

Intel® EE for Lustre\*, when integrated with Linux, aggregates a range of storage hardware into a single Lustre file system that is well-proven for delivering fast IO to applications across high-speed network fabrics such as InfiniBand* and Ethernet.
An existing Lustre file system that has been set up outside of Intel® Manager for Lustre* software can be monitored, but not managed by the manager. In this case, Lustre commands can be used to manage metadata or object storage servers in the Lustre file system. 

<a id="1.1"></a>
## Related Documentation

The following documents are pertinent to Intel® Enterprise Edition for Lustre* software. This list may not be current. Contact your Intel® support representative for the most current information.

- Intel® Enterprise Edition for Lustre* Software Partner Installation Guide
- Creating a Scalable File Service for Windows Networks using Intel® EE for Lustre Software
- Hierarchical Storage Management Configuration Guide
- Installing Hadoop and the Hadoop Adapter for Intel® EE for Lustre* and the Job Scheduler Integration
- Creating an HBase Cluster and Integrating Hive on an Intel® EE for Lustre® File System
- Upgrading a Lustre file system to Intel® Enterprise Edition for Lustre* software (Lustre only)
- Creating a Monitored Lustre* Storage Solution over a ZFS File System
- Creating a High-Availability Lustre* storage Solution over a ZFS File System
- Intel® EE for Lustre* Hierarchical Storage Management Framework White Paper
- Architecting a High-Performance Storage System White Paper

For more information beyond the documents listed above, see: 
**Intel® Solutions for Lustre\* software** - http://www.intel.com/content/www/us/en/software/intel-solutions-for-lustre-software.html

<a id="1.2"></a>
## Overview of Intel® Enterprise Edition for Lustre* software

Intel® Enterprise Edition for Lustre* software is a global single-namespace file system architecture that allows parallel access by many clients to all the data in the file system across many servers and storage devices. Designed to take advantage of the reliability features of enterprise-class storage hardware, Intel® EE for Lustre* software supports availability features such as redundant servers with storage failover. Metadata and data are stored on separate servers to allow each system to be optimized for the different workloads. The components of an Intel® EE for Lustre* software, file storage system include the following:

- Intel® Manager for Lustre server: The server that hosts the Intel® Manager for Lustre* software and GUI, and is the server from which Lustre file systems are created, monitored, and managed. Connected to storage servers via the administrative LAN. This is distinct from the management server, which provides access to the management target.
- Management server(s) (MGS): Provide access to the management target. Paired, redundant management servers provide server failover (high availability) in the event of a server failure.
- Management target (MGT): The MGT stores configuration information for all the Lustre file systems in a cluster and provides this information to other Lustre components. Each Lustre object storage target (OST) contacts the MGT to provide information, and Lustre clients contact the MGT to retrieve information. The MGT can be no larger than 10 gigabytes. 
- Storage servers: Storage servers provide access to the management target, metadata target and the storage targets. Paired, redundant storage servers provide server failover (high availability) in the event of a server failure. 
- Metadata target (MDT): The MDT stores metadata (such as file names, directories, permissions, and file layout) for attached storage and makes this available to clients. Typically, each file system has one MDT, however Intel® EE for Lustre* software supports multiple MDTs.
- Object storage targets (OSTs) - User file data is stored in one or more objects that are located on separate OSTs in the file system. The number of objects per file is configurable by the user and can be tuned to optimize performance for a given workload.
- Lustre clients - Lustre clients are computational, visualization, or desktop nodes that are running Lustre client software, allowing them to mount the Lustre file system.

The servers on which the MGT, MDT, or OSTs are located can all be configured as high-availability (HA) servers, so that if a server for a target fails, a standby server can continue to make the target available.

<a id="f1.1"></a>



![./md_Graphics/lustre-configuration5_zoom40.png][f1.1]

[f1.1]: ./md_Graphics/lustre-configuration5_zoom40.png


