.. _deploy_mssql:

----------------
Deploying MS SQL
----------------

Traditional database VM deployment resembles the diagram below. The process generally starts with a IT ticket for a database (from Dev, Test, QA, Analytics, etc.). Next one or more teams will need to deploy the storage resources and VM(s) required. Once infrastructure is ready, a DBA needs to provision and configure database software. Once provisioned, any best practices and data protection/backup policies need to be applied. Finally the database can be handed over to the end user. That's a lot of handoffs, and the potential for a lot of friction.

.. figure:: images/0.png

Whereas with a Nutanix cluster and Era, provisioning and protecting a database should take you no longer than it took to read this intro.

**In this lab you will deploy a Microsoft SQL Server VM, by cloning a source MSSQL VM. This VM will act as a master image to create a profile for deploying additional SQL VMs using Era.**

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
   - **Password** - Nutanix/4u

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
