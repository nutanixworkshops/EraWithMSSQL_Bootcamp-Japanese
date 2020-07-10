.. title:: Databases: Era with MSSQL Bootcamp

.. toctree::
   :maxdepth: 2
   :caption: Era with MSSQL
   :name: _dbs
   :hidden:

   labsetup/labsetup
   deploy_mssql/deploy_mssql
   admin_mssqldb/admin_mssqldb

.. toctree::
  :maxdepth: 2
  :caption: Creating an App with Era
  :name: _apps_with_era
  :hidden:

  deploy_mssql_era/deploy_mssql_era
  aag/aag
  cloning_with_calm/cloning_with_calm

.. toctree::
  :maxdepth: 2
  :caption: Optional Labs
  :name: _optional_labs
  :hidden:

  flow_secure_fiesta/flow_secure_fiesta
  flow_isolate_fiesta/flow_isolate_fiesta
  era_rest_api/era_rest_api
  hammerdb/hammerdb

.. toctree::
  :maxdepth: 2
  :caption: Appendix
  :name: _appendix
  :hidden:

  tools_vms/windows_tools_vm
  tools_vms/linux_tools_vm
  appendix/glossary

.. _getting_started:

---------------
はじめに
---------------

データベース: Era with MSSQL Bootcamp へようこそ！このワークブックはインストラクター主導のセッションがあり、Nutanixのテクノロジーとたくさんのよくある管理タスクをご紹介します。


What's New
++++++++++

- 以下のソフトウェアバージョン用にワークショップはアップデートされました。
    - AOS 5.11.x / 5.15.x / 5.16.x
    - PC 5.16.x

- オプションラボ アップデート:

アジェンダ
++++++

- イントロダクション
- ラボのセットアップ
- MSSQLのデプロイ
- EraでMSSQL管理
- EraでMSSQLのデプロイ
- AAGのセットアップ
- EraとCalmで開発用Fiesta Applicationのデプロイ

Optionのラボ:

- Flowの使用
- Era API エクスプローラ

イントロダクション
+++++++++++++

- お名前
- Nutanixの習熟度

初期セットアップ
+++++++++++++

- 使用する *パスワード* をメモしてください。
- あなたの仮想デスクトップにログインください(以下に接続情報があります)

環境情報
+++++++++++++++++++

Nutanix ワークショップは、Nutanix Hosted POC 環境で実行するように作られています。あなたのクラスタは、演習をコンプリートするために必要な、すべての必要なイメージ、ネットワーク、そしてVMでプロビジョニングされています。

ネットワーク
..........

Hosted POCクラスタは標準の命名規則に従います。

- **クラスタ名** - POC\ *XYZ*
- **サブネット** - 10.**42**.\ *XYZ*\ .0
- **クラスタIP** - 10.**42**.\ *XYZ*\ .37

マーケティングプールからプロビジョニングされる場合、

- **クラスタ名** - MKT\ *XYZ*
- **サブネット** - 10.**20**.\ *XYZ*\ .0
- **クラスタIP** - 10.**20**.\ *XYZ*\ .37

例:

- **クラスタ名** - POC055
- **サブネット** - 10.42.55.0
- **クラスタIP** - 10.42.55.37

ワークショップを通して、あなたのサブネットを表すのに、XYZを適切なオクテットの値の代わりに用います。例えば：

.. list-table::
   :widths: 25 75
   :header-rows: 1

   * - IP アドレス
     - 説明
   * - 10.42.\ *XYZ*\ .37
     - Nutanix Cluster Virtual IP
   * - 10.42.\ *XYZ*\ .39
     - **PC** VM IP, Prism Central
   * - 10.42.\ *XYZ*\ .40
     - **DC** VM IP, NTNXLAB.local Domain Controller

各クラスタはVMが用いることができる2つのVLANが設定されています。

.. list-table::
  :widths: 25 25 10 40
  :header-rows: 1

  * - Network 名
    - アドレス
    - VLAN
    - DHCP スコープ
  * - Primary
    - 10.42.\ *XYZ*\ .1/25
    - 0
    - 10.42.\ *XYZ*\ .50-10.21.\ *XYZ*\ .124
  * - Secondary
    - 10.42.\ *XYZ*\ .129/25
    - *XYZ1*
    - 10.42.\ *XYZ*\ .132-10.21.\ *XYZ*\ .253

認証情報
...........

.. note::

  *<Cluster Password>* は、各クラスタにユニークで、インストラクターによって提供されます。

.. list-table::
   :widths: 25 35 40
   :header-rows: 1

   * - 認証情報
     - ユーザ名
     - パスワード
   * - Prism Element
     - admin
     - *<Clusterのパスワード>*
   * - Prism Central
     - admin
     - *<Clusterのパスワード>*
   * - Controller VM
     - nutanix
     - *<Clusterのパスワード>*
   * - Prism Central VM
     - nutanix
     - *<Clusterのパスワード>*

各クラスタは、占有のドメインコントローラ VM があり、 **NTNXLAB.local** のADサービスを提供します。ドメインは以下のユーザとグループが設定されています。

.. list-table::
   :widths: 25 35 40
   :header-rows: 1

   * - グループ
     - ユーザ名
     - パスワード
   * - Administrators
     - Administrator
     - nutanix/4u
   * - SSP Admins
     - adminuser01-adminuser25
     - nutanix/4u
   * - SSP Developers
     - devuser01-devuser25
     - nutanix/4u
   * - SSP Consumers
     - consumer01-consumer25
     - nutanix/4u
   * - SSP Operators
     - operator01-operator25
     - nutanix/4u
   * - SSP Custom
     - custom01-custom25
     - nutanix/4u
   * - Bootcamp Users
     - user01-user25
     - nutanix/4u

アクセスの方法
+++++++++++++++++++

Nutanix Hosted POC環境へのアクセスにはいくつかの方法があります。

ラボアクセスのユーザ認証
...........................

PHX Based Clusters:
**ユーザ名:** PHX-POCxxx-User01 (PHX-POCxxx-User20 まで), **パスワード:** *<インストラクタにより提供されます>*

RTP Based Clusters:
**ユーザ名:** RTP-POCxxx-User01 (RTP-POCxxx-User20 まで), **パスワード:** *<インストラクタにより提供されます>*

Frame VDI
.........

ログイン先： https://frame.nutanix.com/x/labs

**Nutanixの従業員** - あなたの **NUTANIXDC** 認証を使用。
**Nutanixの従業員でない方** -  **Lab Access User** 認証を使用。

Parallels VDI
.................

PHXベースのクラスタのログイン先： https://xld-uswest1.nutanix.com

RTPベースのクラスタのログイン先： https://xld-useast1.nutanix.com

**Nutanixの従業員** - あなたの **NUTANIXDC** 認証を使用。
**Nutanixの従業員でない方** - **Lab Access User** 認証を使用。

従業員のPulse Secure VPN
..........................

クライアントをダウンロードします。

PHXベースのクラスタのログイン先： https://xld-uswest1.nutanix.com

RTPベースのクラスタのログイン先：: https://xld-useast1.nutanix.com

**Nutanixの従業員** - あなたの **NUTANIXDC** 認証を使用。
**Nutanixの従業員でない方** - **Lab Access User** 認証を使用。

クライアントをインストールします。

Pluse Secure Clientでは、接続を **追加(add)** します。

PHXに関して

- **Type** - Policy Secure (UAC) または Connection Server
- **Name** - X-Labs - PHX
- **Server URL** - xlv-uswest1.nutanix.com

RTPに関して

- **Type** - Policy Secure (UAC) または Connection Server
- **Name** - X-Labs - RTP
- **Server URL** - xlv-useast1.nutanix.com


Nutanix バージョン情報
++++++++++++++++++++

- **AHV Version** - AHV 20170830.337
- **AOS Version** - 5.11.2.3
- **PC Version** - 5.11.2.1
