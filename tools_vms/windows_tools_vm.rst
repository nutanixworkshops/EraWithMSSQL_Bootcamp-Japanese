.. _windows_tools_vm:

----------------
Windows Tools VM
----------------

Overview
+++++++++

このWindows Server 2012 R2のイメージは以下の数多くのツールがプリインストールされています。

- Microsoft Remote Server Administration Tools (RSAT)
- PuTTY, CyberDuck, WinSCP
- Sublime Text 3, Visual Studio Code
- OpenOffice
- Python
- pgAdmin
- Chocolatey Package Manager

**Lab Setup** の一環として指示された場合は、割り当てられたクラスターにこのVMをデプロイします。

.. raw:: html

  <strong><font color="red">VMは1度だけデプロイします。ラボの完了時にクリーンアップする必要はありません。</font></strong>

Tools VMのデプロイ
++++++++++++++++++

**Prism Central** > :fa:`bars` **> Virtual Infrastructure > VMs** と進み、 **Create VM** をクリックします。

以下を入力します。

- **Name** - *Initials*-Windows-ToolsVM
- **Description** - (Option) あなたのVMを説明します。
- **vCPU(s)** - 1
- **Number of Cores per vCPU** - 2
- **Memory** - 4 GiB

- **+ Add New Disk** を選択します。
    - **Type** - DISK
    - **Operation** - Clone from Image Service
    - **Image** - ToolsVM.qcow2
    - **Add** を選択します。

.. -------------------------------------------------------------------------------------
.. The Below as soon as 5.11 is GA and we want to run that version for our workshops!!!!

.. - **Boot Configuration**
 ..  - Leave the default selected **Legacy Boot**

   .. .. note::
   ..  At the following URL you can find the supported Operating Systems
   ..  http://my.nutanix.com/uefi_boot_support

.. -------------------------------------------------------------------------------------

- **Add New NIC** を選択します。
    - **VLAN Name** - Secondary
    - **Add** を選択します。

**Save** をクリックし、VMを作成します。

VMの電源を入れます。

RDPやコンソールのセッションでVMにログインします。以下の認証情報を使用します。

- **Username** - NTNXLAB\\Administrator
- **password** - nutanix/4u
