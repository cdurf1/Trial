<a id="4.0"></a>
# Monitoring Lustre* file systems

You can easily monitor one or more file systems at the Dashboard, and Status, and Logs windows. The Dashboard window displays a set of charts that provide usage and performance data at several levels in the file systems being monitored, while the Status and Logs windows keep you informed of file system activity relevant to current and past file system health and performance. 

- <a href="#4.1">View charts on the Dashboard</a>
- <a href="#4.2">Check file systems status</a>
- <a href="#4.3">View alerts and events status messages</a>
- <a href="#4.4">View commands and status messages on the Status window</a>
- <a href="#4.5">View Logs</a>
- <a href="#4.6">View and change file system parameters</a>
- <a href="#4.7">View a server's parameters</a>

<a id="4.1"></a>
## View charts on the Dashboard

The Dashboard displays a set of graphical charts that provide real-time usage and performance data at several levels in the file systems being monitored. All Dashboards charts are available for both monitored-only and managed/monitored file systems.

At the top, the Dashboard lists the file system(s) being managed or monitored-only. The following information is provided for each file system:

- **File System:** The name assigned to this file system during its creation on the Configuration window.
- **Type:** Monitored or Managed. Managed file systems are configured and managed for high availability (HA). Managed file systems are both monitored and managed, whereas monitored file systems are monitored-only and do not support failover via Intel® Manager for Lustre* software.
- **Space Used / Total:** This indicates the amount of file system capacity consumed, versus the total file system capacity. 
- **Files Used / Total:** This indicates the total number of inodes consumed by file creation versus the total number of inodes established for this file system.
- **Clients:** Indicates the number of clients accessing the file system at this moment.

**Persistent Chart Configuration**

You can configure certain data display parameters for each chart, and your chart configuration will persist until you reload/refresh the Dashboard page, using the browser. 


See:

- <a href="#4.1.1">View charts for one or all file systems (including all OSTs, MDTs, and servers)</a>
- <a href="#4.1.2">View charts for one or all servers</a>
- <a href="#4.1.3">View charts for an OST or MDT</a>

<a id="4.1.1"></a>
### View charts for one or all file systems

When you first login, the Dashboard displays the following six charts for all file systems combined. Click on the links here to learn more.
MAKEREFS

- Read/Write Heat Map chart
- OST Balance chart
- Metadata Operations chart
- Read/Write Bandwidth chart
- Metadata Servers chart
- Object Storage Servers chart

To view these six charts for a single file system:

1. If it is not displayed, click Dashboard to access the **Dashboard** window. The default view is for all six charts to be displayed. 
1. Click **Configure Dashboard**.
1. Under **File System**, selected the file system you wish to view.
1. Click **Update**.

<a id="4.1.2"></a>
### View charts for one or all servers

When you first login, the Dashboard displays six charts for all file systems combined. 

**View charts for all servers combined**

Viewing charts for all servers is the same thing as viewing charts for all file systems. To do this:

1. On the Dashboard, click Configure Dashboard.
1. Leave All Servers selected in the Server drop-down menu.
1. Click Update.

**View charts for an individual server**

1. On the Dashboard, click Configure Dashboard.
1. Select Server.
1. Under Server, select the server of interest and click Update.

The following charts are displayed for an individual server. Click on the links to learn about these charts. 
MAKEREFS

- Read/Write Bandwidth
- CPU Usage
- Memory Usage


<a id="4.1.3"></a>
### View charts for an OST or MDT

To view charts for a specific OST or MDT:

1. On the Dashboard, click **Configure Dashboard**.
1. Select **Server**.
1. At the Server drop-down menu, select the sever hosting the desired target.
1. At the Target drop-down menu, select the desired target. Then click **Update**.

The following charts are displayed for OSTs. Click on the links here to learn about these charts.
MAKEREF

- Read/Write Bandwidth
- Space Usage
- Object Usage

The following charts are displayed for MDTs:
MAKEREF

- Metadata Operations
- Space Usage
- File Usage

<a id="4.2"></a>
## Check file systems status

The file systems Status light ![md_Graphics/status_light.png][f4.1] provides a quick glance of the status and health of the all file systems managed by Intel® Manager for Lustre* software. This indicator is located along the top banner of the manager GUI. The indicator reflects the worst-case condition. For example, and Error message for any file system will always display a red Status light. Click **Status** to open the Status window and learn more about status.

- A green Status light ![md_Graphics/status_light.png][f4.1] indicates that all is normal. No errors or warnings have been received. The file system is operating normally.
- A yellow Status light ![md_Graphics/yellow_status.png][f4.2] indicates that one or more warning alerts have been received. The file system may be operating in a degraded mode, for example a target has failed over, so performance may be degraded. 
- A red Status light ![md_Graphics/red_status.png][f4.3] indicates that one or more errors alerts have been received. This file system may be down or is severely degraded. One or more file system components may be currently unavailable, for example, both the primary and secondary servers for a target are not running.

Click **Status** to open the Status window. See MAKEREFView status messages on the Status window.


[f4.1]:md_Graphics/status_light.png
[f4.2]:md_Graphics/yellow_status.png
[f4.3]:md_Graphics/red_status.png
