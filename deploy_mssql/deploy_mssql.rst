.. _deploy_mssql:

----------------
Deploying MS SQL
----------------

Traditional database VM deployment resembles the diagram below. The process generally starts with a IT ticket for a database (from Dev, Test, QA, Analytics, etc.). Next one or more teams will need to deploy the storage resources and VM(s) required. Once infrastructure is ready, a DBA needs to provision and configure database software. Once provisioned, any best practices and data protection/backup policies need to be applied. Finally the database can be handed over to the end user. That's a lot of handoffs, and the potential for a lot of friction.

.. figure:: images/0.png

Whereas with a Nutanix cluster and Era, provisioning and protecting a database should take you no longer than it took to read this intro.

**In this lab you will deploy a Microsoft SQL Server VM, by cloning a source MSSQL VM. This VM will act as a master image to create a profile for deploying additional SQL VMs using Era.**

Manual VM Deployment
++++++++++++++++++++

#. In **Prism Central**, select :fa:`bars` **> Virtual Infrastructure > VMs**.

   .. figure:: images/1-1.png

#. Click **Create VM**.

#. Select your assigned cluster and click **OK**.

#. Fill out the following fields:

   - **Name** - *Initials*-MSSQL-Manual
   - **Description** - (Optional) Description for your VM.
   - **vCPU(s)** - 2
   - **Number of Cores per vCPU** - 1
   - **Memory** - 4 GiB

   - Select **+ Add New Disk**
      - **Type** - DISK
      - **Operation** - Clone from Image Service
      - **Image** - MSSQL-2016-VM.qcow2
      - Select **Add**

   - Select **Add New NIC**
      - **VLAN Name** - *Secondary*
      - Select **Add**

#. Click **Save** to create the VM.

#. Select your VM and click **Actions > Power On**.

#. Once powered on, click **Actions > Launch Console** and complete Windows Server setup:

   - Click **Next**
   - **Accept** the licensing agreement
   - Enter an easy to remember password as the Administrator password and click **Finish**

#. Log in to the VM using the Administrator password you configured.

#. Disable Windows Firewall for all. **Do NOT modify the default keyboard mapping.**

#. Launch **File Explorer** and note the current, single disk configuration.

   .. figure:: images/1-2.png

   .. note::

      Best practices for database VMs involve spreading the OS, SQL binaries, databases, TempDB, and logs across separate disks in order to maximize performance. On non-AHV hypervisors, these disks should be properly spread across multiple disk controllers, as shown in the diagram below.

      .. figure:: images/1-2b.png

      For complete details for tuning SQL Server on Nutanix (including guidance around NUMA, hyperthreading, SQL Server configuration settings, and more), see the `Nutanix Microsoft SQL Server Best Practices Guide <https://portal.nutanix.com/#/page/solutions/details?targetId=BP-2015-Microsoft-SQL-Server:BP-2015-Microsoft-SQL-Server>`_.

#. From the desktop, launch the **01 - Rename Server.ps1** PowerShell script shortcut and fill out the following fields:

   - **Enter the Nutanix cluster IP** - *Assigned Nutanix Cluster IP*
   - **Enter the Nutanix user name for...** - admin
   - **Enter the Nutanix password for "admin"** - *your cluster password*

   The script will validate the VM name does not exceed 15 characters and then rename the server to match the VM name.

#. Once VM has rebooted, log in and launch the **02 - Complete Build.ps1** Powershell script shortcut. Fill out the following fields:

   - **Enter the Nutanix cluster IP** - *Assigned Nutanix Cluster IP*
   - **Enter the Nutanix user name for...** - admin
   - **Enter the Nutanix password for "admin"** - emeaX2020!
   - **Enter the Nutanix container name** - Default

   .. note::

      All fields in the above script are case sensitive.

   This script will setup and create disk drives according to best practices place SQL data files on those drives. The SQL Systems File is placed on the D:\ drive and data and logs files are placed on separate drives.

#. Once VM has rebooted, verify the new disk configuration in **Prism** and **File Explorer**

   .. figure:: images/1-3.png

   .. figure:: images/1-4.png

#. Log in to your *Initials*\ **-MSSQL** VM and launch SQL Server Management Studio from the desktop.

#. Connect using **Windows Authentication** and verify the database server is available, with only system databases provisioned.

   .. figure:: images/1-5.png

   Congratulations, you now have a functioning SQL Server VM. While this process could be further automated through ``acli``, Calm, or REST API calls orchestrated by a third party tool, provisioning only solves a Day 1 problem for databases, and does little to address storage sprawl, cloning, or patch management.

#. Shutdown this VM

.. note::
   Shutdown of this VM is important - this will not be required any further in this lab. The purpose of building this VM was to demonstrate how hard it is deploy and apply best practices to a MS SQL VM.
   We can use this VM in performance testing using HammerDB tool

Clone Source MSSQL VM
+++++++++++++++++++++

#. In **Prism Central**, select :fa:`bars` **> Virtual Infrastructure > VMs**.

   .. figure:: images/1.png

#. Select the checkbox for **Win2016SQLSource**, and click **Actions > Clone**.

#. Fill out the following fields:

   - **Number Of Clones** - 1
   - **Name** - *Initials*-MSSQL
   - **vCPU(s)** - 2
   - **Number of Cores per vCPU** - 1
   - **Memory** - 4 GiB

   .. figure:: images/clone_mssql_source.png

#. Click **Save** to create the VM.

#. Select your VM and click **Actions > Power On**.

#. Log in to the VM (**Cancel** Shutdown Event Tracker):

   - **Username** - Administrator
   - **Password** - **Request your Instructor for password**

#. Disable Windows Firewall for all.

#. Open SQL Server Managment Studio (SSMS), and **Connect** using Windows Authentication.

#. Verify you can browse the **SampleDB**.

Exploring Era Resources
+++++++++++++++++++++++

Era is distributed as a virtual appliance that can be installed on either AHV or ESXi. For the purposes of conserving memory resources, a shared Era server has already been deployed on your cluster.

.. note::

   If you're interested, instructions for the brief installation of the Era appliance can be found `here <https://portal.nutanix.com/#/page/docs/details?targetId=Nutanix-Era-User-Guide-v12:era-era-installing-on-ahv-t.html>`_.

#. In **Prism Central > VMs > List**, identify the IP address assigned to the **EraServer-\*** VM using the **IP Addresses** column.

#. Open \https://*ERA-VM-IP:8443*/ in a new browser tab.

#. Login using the following credentials:

   - **Username** - admin
   - **Password** - nutanix/4u

#. From the **Dashboard** dropdown, select **Administration**.

#. Under **Cluster Details**, note that Era has already been configured for your assigned cluster.

   .. figure:: images/6.png

#. Select **Era Resources** from the left-hand menu.

#. Review the configured Networks. If no Networks show under **VLANs Available for Network Profiles**, click **Add**. Select **Secondary** VLAN and click **Add**.

   .. note::

      Leave **Manage IP Address Pool** unchecked, as we will be leveraging the cluster's IPAM to manage addresses

   .. figure:: images/era_networks_001.png

#. From the dropdown menu, select **SLAs**.

   .. figure:: images/7a.png

   Era has five built-in SLAs (Gold, Silver, Bronze, Zero, and Brass). SLAs control how the database server is backed up. This can be with a combination of Continuous Protection, Daily, Weekly Monthly and Quarterly protection intervals.

#. From the dropdown menu, select **Profiles**.

   Profiles pre-define resources and configurations, making it simple to consistently provision environments and reduce configuration sprawl. For example, Compute Profiles specifiy the size of the database server, including details such as vCPUs, cores per vCPU, and memory.

#. If you do not see any networks defined under **Network**, click **+ Create**.

   .. figure:: images/8.png

#. Fill out the following fields and click **Create**:

   - **Engine** - Microsoft SQL Server
   - **Name** - Primary-MSSQL-NETWORK
   - **Public Service VLAN** - Secondary

   .. figure:: images/9.png

Registering Your MSSQL VM
+++++++++++++++++++++++++

Registering a database server with Era allows you to deploy databases to that resource, or to use that resource as the basis for a Software Profile.

You must meet the following requirements before you register a SQL Server database with Era:

- A local user account or a domain user account with administrator privileges on the database server must be provided.
- Windows account or the SQL login account provided must be a member of sysadmin role.
- SQL Server instance must be running.
- Database files must not exist in C:\ Drive.
- Database must be in an online state.
- Windows remote management (WinRM) must be enabled

.. note::

   Your *XYZ*\ **-MSSQL** VM meets all of these criteria.

#. In **Era**, select **Database Servers** from the dropdown menu and **List** from the lefthand menu.

   .. figure:: images/11.png

#. Click **+ Register** and fill out the following fields:

   - **Engine** - Microsoft SQL Server
   - **IP Address or Name of VM** - *Initials*\ -MSSQL
   - **Windows Administrator Name** - Administrator
   - **Windows Administrator Password** - Nutanix/4u
   - **Instance** - MSSQLSERVER (This should auto-populate after providing credentials)
   - **Connect to SQL Server Admin** - Windows Admin User
   - **User Name** - Administrator

   .. note::

      If **Instance** does not automatically populate, disable the Windows Firewall in your *XYZ*\ **-MSSQL** VM.

   .. figure:: images/12.png

   .. note::

    You can click **API Equivalent** for many operations in Era to enter an interactive wizard providing JSON payload based data you've input or selected within the UI, and examples of the API call in multiple languages (cURL, Python, Golang, Javascript, and Powershell).

    .. figure:: images/17.png

#. Click **Register** to begin ingesting the Database Server into Era.

#. Select **Operations** from the dropdown menu to monitor the registration. This process should take approximately 5 minutes.

   .. figure:: images/13.png

   .. note::

      It is also possible to register existing databases on any server, which will also register the database server it is on.

Creating A Software Profile
+++++++++++++++++++++++++++

Before additional SQL Server VMs can be provisioned, a Software Profile must first be created from the database server VM registered in the previous step. A software profile is a template that includes the SQL Server database and operating system. This template exists as a hidden, cloned disk image on your Nutanix storage.

#. Select **Profiles** from the dropdown menu and **Software** from the lefthand menu.

   .. figure:: images/14.png

#. Click **+ Create** and fill out the following fields:

   - **Engine** - Microsoft SQL Server
   - **Name** - *Initials*\ _MSSQL_2016
   - **Description** - (Optional)
   - **Database Server** - Select your registered *Initials*\ -MSSQL VM

   .. figure:: images/15.png

#. Click **Create**.

#. Select **Operations** from the dropdown menu to monitor the registration. This process should take approximately 5 minutes.

   .. figure:: images/16.png

   .. note::

       If creating a profile from a not-cleanly shutdown server it may be corrupt or may not provision successfully. Please ensure that the DBServer had a clean shutdown and clean startup before registering profile to Era.
