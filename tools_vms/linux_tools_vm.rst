.. _linux_tools_vm:

---------------
Linux Tools VM
---------------

概要
+++++++++

このCentOS VMイメージは、複数のラボ演習をサポートするために使用されるパッケージを持ちます。

**Lab Setup**.の一環として指示された場合は、割り当てられたクラスターにこのVMをデプロイします。

.. raw:: html

  <strong><font color="red">VMは1度だけデプロイします。ラボの完了時にクリーンアップする必要はありません。</font></strong>

CentOSのデプロイ
++++++++++++++++

**Prism Central** > :fa:`bars` **> Virtual Infrastructure > VMs** と進み、 **Create VM** をクリックします。

以下の通り入力します。

- **Name** - *Initials*-Linux-ToolsVM
- **Description** - (Option) あなたのVMを説明します。
- **vCPU(s)** - 1
- **Number of Cores per vCPU** - 2
- **Memory** - 2 GiB

- **+ Add New Disk** を選択します。
    - **Type** - DISK
    - **Operation** - Clone from Image Service
    - **Image** - CentOS7.qcow2
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

ツールのインストール
++++++++++++++++

SSHやコンソールのセッションでVMにログインします。以下の認証情報を使用します。

- **Username** - root
- **password** - nutanix/4u

以下コマンドを実行して必要なソフトウェアをインストールします。

.. code-block:: bash

  yum update -y
  yum install -y ntp ntpdate unzip stress nodejs python-pip s3cmd awscli
  npm install -g request
  npm install -g express


NTPの設定
...............

以下のコマンドを実行してNTPを有効化して設定します。

.. code-block:: bash

  systemctl start ntpd
  systemctl enable ntpd
  ntpdate -u -s 0.pool.ntp.org 1.pool.ntp.org 2.pool.ntp.org 3.pool.ntp.org
  systemctl restart ntpd

FirewallとSELinuxを無効化
..............................

以下のコマンドを実行してfirewallとSELinuxを無効化します。

.. code-block:: bash

  systemctl disable firewalld
  systemctl stop firewalld
  setenforce 0
  sed -i 's/enforcing/disabled/g' /etc/selinux/config /etc/selinux/config

Pythonのインストール
.................

以下のコマンドを実行してPythonをインストールします。

.. code-block:: bash

  yum -y install python36
  python3.6 -m ensurepip
  yum -y install python36-setuptools
  pip install -U pip
  pip install boto3
