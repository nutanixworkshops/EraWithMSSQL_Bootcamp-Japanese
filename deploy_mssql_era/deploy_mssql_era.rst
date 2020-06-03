.. _mssqldeploy:

-------------------------
Deploying MS SQL with Era
-------------------------

**In this lab you will create a MSSQL Software Profile, and use Era to deploy a new MSSQL Database Server.**



Creating a New MSSQL Database Server
++++++++++++++++++++++++++++++++++++

You've completed all the one time operations required to be able to provision any number of SQL Server VMs. Follow the steps below to provision a database of a fresh database server, with best practices automatically applied by Era.

#. In **Era**, select **Databases** from the dropdown menu and **Sources** from the lefthand menu.

#. Click **+ Provision > Single Node Database**.

   .. figure:: images/18.png

#. In the **Provision a Database** wizard, fill out the following fields to configure the Database Server:

   - **Engine** - Microsoft SQL Server
   - **Database Server** - Create New Server
   - **Database Server Name** - *Initials*\ -MSSQL2
   - **Description** - (Optional)
   - **Software Profile** - *Initials*\ _MSSQL_2016
   - **Compute Profile** - CUSTOM_EXTRA_SMALL
   - **Network Profile** - Primary_MSSQL_NETWORK
   - **Database Time Zone** - Pacific Standard time
   - Select **Join Domain**
   - **Windows Domain Profile** - NTNXLAB
   - **Windows License Key** - (Leave Blank)
   - **Administrator Password** - Nutanix/4u
   - **Instance Name** - MSSQLSERVER
   - **Server Collation** - Default
   - **Database Parameter Profile** - DEFAULT_SQLSERVER_INSTANCE_PARAMS
   - **SQL Service Startup Account** - ntnxlab.local\\Administrator
   - **SQL Service Startup Account Password** - nutanix/4u
   - **SQL Server Authentication Mode** - Windows Authentication
   - **Domain User Account** - (Leave Blank)

   .. figure:: images/19a.png

   .. note::

      A **Instance Name** is the name of the database server, this is not the hostname. The default is **MSSQLSERVER**. You can install multiple separate instances of MSSQL on the same server as long as they have different instance names. This was more common on a physical server, however, you do not need additional MSSQL licenses to run multiple instances of SQL on the same server.

      **Server Collation** is a configuration setting that determines how the database engine should treat character data at the server, database, or column level. SQL Server includes a large set of collations for handling the language and regional differences that come with supporting users and applications in different parts of the world. A collation can also control case sensitivity on database. You can have different collations for each database on a single instance. The default collation is **SQL_Latin1_General_CP1_CI_AS** which breaks out like below:

      - **Latin1** makes the server treat strings using charset latin 1, basically **ASCII**
      - **CP1** stands for Code Page 1252. CP1252 is  single-byte character encoding of the Latin alphabet, used by default in the legacy components of Microsoft Windows for English and some other Western languages
      - **CI** indicates case insensitive comparisons, meaning **ABC** would equal **abc**
      - **AS** indicates accent sensitive, meaning **ü** does not equal **u**

      **Database Parameter Profiles** define the minimum server memory SQL Server should start with, as well as the maximum amount of memory SQL server will use. By default, it is set high enough that SQL Server can use all available server memory. You can also enable contained databases feature which will isolate the database from others on the instance for authentication.

#. Click **Next**, and fill out the following fields to configure the Database:

   - **Database Name** - *Initials*\ -fiesta
   - **Description** - (Optional)
   - **Size (GiB)** - 200 (Default)
   - **Database Parameter Profile** - DEFAULT_SQLSERVER_DATABASE_PARAMS

   .. figure:: images/20.png

   .. note::

      Common applications for pre/post-installation scripts include:

      - Data masking scripts
      - Register the database with DB monitoring solution
      - Scripts to update DNS/IPAM
      - Scripts to automate application setup, such as app-level cloning for Oracle PeopleSoft

#. Click **Next** and fill out the following fields to configure the Time Machine for your database:

   .. note::

      .. raw:: html

        <strong><font color="red">It is critical to select the BRONZE SLA in the following step. The default BRASS SLA does NOT include Continuous Protection snapshots.</font></strong>

   - **Name** - *initials*\ -fiesta_TM (Default)
   - **Description** - (Optional)
   - **SLA** - DEFAULT_OOB_BRONZE_SLA
   - **Schedule** - (Defaults)

   .. figure:: images/21.png

#. Click **Provision** to begin creating your new database server VM and **fiesta** database.

#. Select **Operations** from the dropdown menu to monitor the provisioning. This process should take approximately 20 minutes.

   .. figure:: images/22.png

   .. note::

      Observe the step for applying best practices in **Operations**.

      Some of the best practices automatically configured by Era include:

      - Distribute databases and log files across multiple vDisks.
      - Do not use Windows dynamic disks or other in-guest volume management
      - Distribute vDisks across multiple SCSI controllers (for ESXi)
      - For each database, use multiple data files: one file per vCPU.
      - Configure initial log file size to 4 GB or 8 GB and iterate by the initial amount to reach the desired size.
      - Use multiple TempDB data files, all the same size.
      - Use available hypervisor network control mechanisms (for example, VMware NIOC).


Exploring the Provisioned DB Server
++++++++++++++++++++++++++++++++++++

#. In **Prism Element > Storage > Table > Volume Groups**, locate the **ERA_**\ *Initials*\ **_MSSQL2_\** VG and observe the layout on the **Virtual Disk** tab. <What does this tell us?>

   .. figure:: images/23.png

#. View the disk layout of your newly provisioned VM in Prism. <What are all of these disks and how is this different from the original VM we registered?>

   .. figure:: images/24.png

#. In Prism, note the IP address of your *Initials*\ **-MSSQL2** VM and connect to it via RDP using the following credentials:

   - **User Name** - NTNXLAB\\Administrator
   - **Password** - nutanix/4u

#. Open **Start > Run > diskmgmt.msc** to view the in-guest disk layout. Right-click an unlabeled volume and select **Change Drive Letter and Paths** to view the path to which Era has mounted the volume. Note there are dedicated drives corresponding to SQL data and log locations, similar to the original SQL Server to which you manually applied best practices.

   .. figure:: images/25.png

Migrating Fiesta App Data
+++++++++++++++++++++++++

In this exercise you will import data directly into your database from a backup exported from another database. While this is a suitable method for migrating data, it potentially involved downtime for an application, or our database potentially not having the very latest data.

Another approach could involve adding your new Era database to an existing database cluster (AlwaysOn Availability Group) and having it replicate to your Era provisioned database. Application level synchronous or asynchronous replication (such as SQL Server AAG or Oracle RAC) can be used to provide Era benefits like cloning and Time Machine to databases whose production instances run on bare metal or non-Nutanix infrastructure.

#. From your *Initials*\ **-MSSQL2** RDP session, launch **Microsoft SQL Server Management Studio** and click **Connect** to authenticate as the currently logged in user.

   .. figure:: images/26.png

#. Expand the *Initials*\ **-fiesta** database and note that it contains no tables. With the database selected, click **New Query** from the menu to import your production application data.

   .. figure:: images/27.png

#. Copy and paste the following script into the query editor and click **Execute**:

   .. literalinclude:: FiestaDB-MSSQL.sql
     :caption: FiestaDB Data Import Script
     :language: sql

   .. figure:: images/28.png

#. Note the status bar should read **Query executed successfully**.

#. You can view the contents of the database by clicking **New Query** and executing the following:

   .. code-block:: sql

      SELECT * FROM dbo.products
      SELECT * FROM dbo.stores
      SELECT * FROM dbo.InventoryRecords

   .. figure:: images/29.png

#. In **Era > Time Machines**, select your *initials*\ **-fiesta_TM** Time Machine. Select **Actions > Log Catch Up > Yes** to ensure the imported data has been flushed to disk prior to the cloning operation in the next lab.

Provision Fiesta Web Tier
+++++++++++++++++++++++++

Manipulating data using **SQL Server Management Studio** is boring. In this section you'll deploy the web tier of the application and connect it to your production database.

#. `Download the Fiesta Blueprint by right-clicking here <https://raw.githubusercontent.com/nutanixworkshops/EraWithMSSQL/master/db/mssqldeploy/FiestaNoDB.json>`_. This single-VM Blueprint is used to provision only the web tier portion of the application.

#. From **Prism Central > Calm**, select **Blueprints** from the lefthand menu and click **Upload Blueprint**.

   .. figure:: images/30.png

#. Select **FiestaNoDB.json**.

#. Update the **Blueprint Name** to include your initials. Even across different projects, Calm Blueprint names must be unique.

#. Select *Initials*\ -Project as the Calm project and click **Upload**.

   .. figure:: images/31.png

#. In order to launch the Blueprint you must first assign a network to the VM. Select the **NodeReact** Service, and in the **VM** Configuration menu on the right, select **Secondary** as the **NIC 1** network.

   .. figure:: images/32a.png

#. Click **Credentials** to define a private key used to authenticate to the CentOS VM that will be provisioned by the Blueprint.

#. Expand the **CENTOS** credential and use your preferred SSH key, or paste in the following value as the **SSH Private Key**:

   ::

     -----BEGIN RSA PRIVATE KEY-----
     MIIEowIBAAKCAQEAii7qFDhVadLx5lULAG/ooCUTA/ATSmXbArs+GdHxbUWd/bNG
     ZCXnaQ2L1mSVVGDxfTbSaTJ3En3tVlMtD2RjZPdhqWESCaoj2kXLYSiNDS9qz3SK
     6h822je/f9O9CzCTrw2XGhnDVwmNraUvO5wmQObCDthTXc72PcBOd6oa4ENsnuY9
     HtiETg29TZXgCYPFXipLBHSZYkBmGgccAeY9dq5ywiywBJLuoSovXkkRJk3cd7Gy
     hCRIwYzqfdgSmiAMYgJLrz/UuLxatPqXts2D8v1xqR9EPNZNzgd4QHK4of1lqsNR
     uz2SxkwqLcXSw0mGcAL8mIwVpzhPzwmENC5OrwIBJQKCAQB++q2WCkCmbtByyrAp
     6ktiukjTL6MGGGhjX/PgYA5IvINX1SvtU0NZnb7FAntiSz7GFrODQyFPQ0jL3bq0
     MrwzRDA6x+cPzMb/7RvBEIGdadfFjbAVaMqfAsul5SpBokKFLxU6lDb2CMdhS67c
     1K2Hv0qKLpHL0vAdEZQ2nFAMWETvVMzl0o1dQmyGzA0GTY8VYdCRsUbwNgvFMvBj
     8T/svzjpASDifa7IXlGaLrXfCH584zt7y+qjJ05O1G0NFslQ9n2wi7F93N8rHxgl
     JDE4OhfyaDyLL1UdBlBpjYPSUbX7D5NExLggWEVFEwx4JRaK6+aDdFDKbSBIidHf
     h45NAoGBANjANRKLBtcxmW4foK5ILTuFkOaowqj+2AIgT1ezCVpErHDFg0bkuvDk
     QVdsAJRX5//luSO30dI0OWWGjgmIUXD7iej0sjAPJjRAv8ai+MYyaLfkdqv1Oj5c
     oDC3KjmSdXTuWSYNvarsW+Uf2v7zlZlWesTnpV6gkZH3tX86iuiZAoGBAKM0mKX0
     EjFkJH65Ym7gIED2CUyuFqq4WsCUD2RakpYZyIBKZGr8MRni3I4z6Hqm+rxVW6Dj
     uFGQe5GhgPvO23UG1Y6nm0VkYgZq81TraZc/oMzignSC95w7OsLaLn6qp32Fje1M
     Ez2Yn0T3dDcu1twY8OoDuvWx5LFMJ3NoRJaHAoGBAJ4rZP+xj17DVElxBo0EPK7k
     7TKygDYhwDjnJSRSN0HfFg0agmQqXucjGuzEbyAkeN1Um9vLU+xrTHqEyIN/Jqxk
     hztKxzfTtBhK7M84p7M5iq+0jfMau8ykdOVHZAB/odHeXLrnbrr/gVQsAKw1NdDC
     kPCNXP/c9JrzB+c4juEVAoGBAJGPxmp/vTL4c5OebIxnCAKWP6VBUnyWliFhdYME
     rECvNkjoZ2ZWjKhijVw8Il+OAjlFNgwJXzP9Z0qJIAMuHa2QeUfhmFKlo4ku9LOF
     2rdUbNJpKD5m+IRsLX1az4W6zLwPVRHp56WjzFJEfGiRjzMBfOxkMSBSjbLjDm3Z
     iUf7AoGBALjvtjapDwlEa5/CFvzOVGFq4L/OJTBEBGx/SA4HUc3TFTtlY2hvTDPZ
     dQr/JBzLBUjCOBVuUuH3uW7hGhW+DnlzrfbfJATaRR8Ht6VU651T+Gbrr8EqNpCP
     gmznERCNf9Kaxl/hlyV5dZBe/2LIK+/jLGNu9EJLoraaCBFshJKF
     -----END RSA PRIVATE KEY-----

   .. figure:: images/33.png

#. Click **Save** and click **Back** once the Blueprint has completed saving.

#. Click **Launch** and fill out the following fields:

   - **Name of the Application** - *Initials*\ -Fiesta
   - **db_password** - nutanix/4u
   - **db_name** - *Initials*\ -fiesta (as configured when you deployed through Era)
   - **db_dialect** - mssql
   - **db_domain_name** - ntnxlab.local
   - **db_username** - Administrator
   - **db_host_address** - The IP of your *Initials*\ **-MSSQL2** VM

   .. figure:: images/34.png

#. Click **Create**.

#. Select the **Audit** tab to monitor the deployment. This process should take < 5 minutes.

   .. figure:: images/35.png

#. Once the application status changes to **Running**, select the **Services** tab and select the **NodeReact** service to obtain the **IP Address** of your web server.

   .. figure:: images/36.png

#. Open \http://*NODEREACT-IP-ADDRESS:5001*/ in a new browser tab to access the **Fiesta** application.

   .. figure:: images/37.png

   Congratulations! You've completed the deployment of your production application.

Takeaways
+++++++++

What are the key things we learned in this lab?

- Existing databases can be easily onboarded into Era, and turned into templates
- Existing brownfield databases can also be registered with Era
- Profiles allow administrators to provision resources based on published standards
- Customizable recovery SLAs allow you to tune continuous, daily, and monthly RPO based on your app's requirements
- Era provides One-click provisioning of multiple database engines, including automatic application of database best practices
