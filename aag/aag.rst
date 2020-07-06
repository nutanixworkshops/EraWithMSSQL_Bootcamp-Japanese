.. _aag:

---------------------------------
データベースの可用性を簡単化
---------------------------------

ここまでは、私たちはEraを使ってシングルインスタンスのデータベースを作成しました。実際の業務用のデータベースについて、あなたは、クラスタのソリューションを使って、高い可用性を提供し、あなたのアプリケーションやビジネスのダウンタイムの可能性を軽減したいと考えるかもしれません。Eraは、Microsoft SQL Server AlwaysOn Availability GroupとOracle RACでクラスタ化したデータベースのプロビジョニングと管理をサポートします。

SQL Server AAG クラスタは、(動作時に)たくさんの動くパーツがあり、シングルクラスタのデプロイは数時間以上かかります。

**このラボでは、あなたの既存の業務用のSQL Server データベース を データベースクラスタにクローンして、Fiesta appを使って可用性をテストします。**

Eraの管理ネットワークの作成
+++++++++++++++++++++++++++++++

#. **Era > Administration > Era Resources** において、設定されたネットワークをレビューします。もし、 **EraManaged(Era管理の)** ネットワークが、 **VLANs Available for Network Profiles** 内に表示されないなら、 **Add** をクリックします。

   .. figure:: images/3.png

#. 以下の通り入力して、 **Add** をクリックします。

   .. note:: 以下のネットワーク設定がだいたい以下のアドレスにマッチすべきです。多くのケースにおいて：

       - "Gateway" は、 10.x.x.129
       - "Subnet Mask" は、 255.255.255.128
       - "Primary DNS" は、 10.x.x.41
       - "First Address" は、 10.x.x.220
       - "Last Address" は、 10.x.x.253

   - **Select a VLAN** - EraManaged
   - **Manage IP Address Pool** を選択します
   - **Gateway** - *Secondary Network と同じ*
   - **Verify** を選択します
   - **Subnet Mask** - 255.255.255.128
   - **Primary DNS** - 10.x.x.41 *Bootcampのリーダーによって提供されたネットワーク情報を参照します*
   - **DNS Domain** - ntnxlab.local
   - **First Address** - 10.x.x.220 *Bootcampのリーダーによって提供されたネットワーク情報を参照します*
   - **Last Address** - 10.x.x.253 *Bootcampのリーダーによって提供されたネットワーク情報を参照します*

   .. figure:: images/4.png

#. **Era > Profiles > Network** において、 **+ Create** をクリックし、 **EraManaged(Era管理の)** ネットワークをプロファイルに加えます。

#. 以下のように入力します。

   - **Engine** - Microsoft SQL Server
   - **Name** - ERAMANAGED_MSSQL_NETWORK
   - **Public Service VLAN** - EraManaged

   .. figure:: images/5.png

#. **Create** をクリックします。

.. _provisioningaag:

AAGのプロビジョニング
+++++++++++++++++++

#. **Era** において、ドロップダウンメニューから **Time Machines** を選択します。

#. あなたの業務用のデータベース (例えば、 *xyz-fiesta_TM*, *xyz-fiesta2_TM* ではないです) に関連づいたTime Machine を選択ください。

#. **Actions > Clone Database > Cluster Database** を選択します。

   デフォルトでは、クローンは最新の **Point in Time** から作成します。代わりには、過去のポイントインタイムまたはスナップショットを明示的に指定します。

#. **Next** をクリックします。

   .. figure:: images/6.png

#. 以下の通り入力し、 **Next** をクリックします。

   - **Windows Cluster** - Create New Cluster
   - **Windows Cluster Name** - *Initials*\ -clusterdb (Cluster名は15文字の制限があります)
   - **Compute Profile** - CUSTOM_EXTRA_SMALL
   - **Network Profile** - ERAMANAGED_MSSQL_NETWORK
   - **Windows Domain Profile** - NTNXLAB
   - **Administrator Password** - Nutanix/4u
   - **Instance Name** - MSSQLSERVER
   - **SQL Service Startup Account** - ntnxlab.local\\Administrator
   - **SQL Service Startup Account Password** - nutanix/4u
   - **SQL Server Authentication Mode** - Windows Authentication
   - **Domain User Account** - ntnxlab.local\\Administrator

   .. figure:: images/7a.png

#. 以下のデフォルトの **Topology** を修正し、 **Next** をクリックします。

   - **Always on Availability Group Name** - *Initials*\ -aag

   .. figure:: images/8.png

   .. note::

      SQL 2016以降は9つのセカンダリのレプリカまでサポートします。

      **Primary(プライマリ)**  サーバは、どのホストをAAGに起動させるかを指します。

      **Auto Failover** により、AAGは、 **Primary(プライマリ)** のホストが利用不可であることを検知した際に自動的にフェイルオーバーします。これは多くのデプロイにおいて好まれており、追加の管理者の介入が不要で、最大のアプリケーションの稼働時間を可能にします。

      **Availability Mode** が **Synchronous(同期)** または **Asynchronous(非同期)** で設定されます。

      - **Synchronous-commit replicas(同期-コミット レプリカ)** - データは同時にプライマリとセカンダリのノードにコミットされます。このモードは **Automatic(自動)** と **Manual(手動)** の **Failover(フェイルオーバー)** の方法をサポートします。
      - **Asynchronous-commit replicas(非同期-コミット レプリカ)** - データは、はじめにプライマリのノードにコミットされ、インターバル後に、データはセカンダリのノードにコミットされます。このモードは **Manual Failover(手動フェイルオーバー)** のみをサポートします。

      **Readable Secondaries(読み取り可能なセカンダリ)** により、あなたはプライマリのレプリカから読み取り専用のセカンダリのワークロードをオフロードでき、あなたのミッションクリティカルなワークロードのリソースを節約します。もし、ミッションクリティカルな読み取りワークロードや数秒程度しかレイテンシを許容できないワークロードがある場合、プライマリ上でそれを稼働する必要があります。

#. **Clone** をクリックします。

   .. figure:: images/9.png

#. **Operations**  ページ上で更新をモニターします。この操作はおよそ35分かかります。 **あなたのデータベースサーバがプロビジョニングされている間、次のセクションに進みます。**

   .. figure:: images/10.png

AAGのFiestaの設定
++++++++++++++++++++++++

追加のFiesta web server VMをデプロイするより、あなたの既存のVMの設定をアップデートして、データベース クラスタにポイントするようにします。

#. **Era > Databases > Clones** にて、あなたの最近のクローンを選択して、AAG デプロイの詳細を見ます。Always on Availability Groupの **リスナーのIPアドレス(Listener IP Address)** を確認します。

   .. figure:: images/11.png

#. **Prism Central > Calm > Applications** にて、あなたの *Initials*\ **-DevFiesta** のデプロイを選択します。 **Services** タブにて、 **NodeReact** サービスを選択し、 **Open Terminal > Proceed** をクリックし、VMへのSSHセッションで新しいタブを開きます。

   .. figure:: images/12.png

#. cat Fiesta/config/config.js を実行し、DB_HOST_ADDRESS の値を確認します。

   .. figure:: images/13.png

#. 以下のコマンドを実行します。

   ::

     sudo sed -i 's/CURRENT_DB_HOST_ADDRESS_VALUE/AAG_LISTENER_IP_ADDRESS_VALUE/g' ~/Fiesta/config/config.js

   .. note::

      コマンド例はここです。あなたのSQL AAGのリスナーのIPアドレスを使用します。

      $ sudo sed -i 's/10.38.193.147/10.38.193.215/g' ~/Fiesta/config/config.js

#. configファイルをcatで実行してIPアドレスの更新を確認します。

   ::

      cat Fiesta/config/config.js

   .. figure:: images/14.png

#. systemctlをsudoで実行してfiestaを再起動します。

クラスタサーバの障害
++++++++++++++++++++++++

壊すときがきました!

#. あなたの **Dev Fiesta** web appを開き、store(ストア)の削除(Delete Store)と追加のproduct(製品)をstoreへ追加(Add New Store)など変更を行います。

   .. figure:: images/15.png

#. **Prism Central > VMs** で、 *Initials*\ **-clusterdb-1** VM を電源OFFします。

   .. note:: どのVMが現在プライマリのAAGのメンバであるかダブルチェックします。具体的には、Prism Centralで、どのVMが現在AAGのリスナーIPアドレスとWindows クラスタIPを表示するかを確認します。

   .. figure:: images/16.png

#. **Prism Central** を更新し、 **リスナー(Listener)** と **クラスタ(Cluster)** のIPアドレスが他の **clusterdb** VMに割り当てられていることを確認ください。

   .. figure:: images/17.png

#. あなたの **Dev Fiesta** web appを更新し、データが適切に表示されているか確認ください。

重要なポイント
+++++++++

このラボで学んだ重要なことは何でしょうか。

- 業務用のデータベースは、ダウンタイムを防ぐために高いレベルの可用性が必要です。
- Eraは、シングルインスタンスのデータベースと同じくらい容易に速く複雑なクラスタ化されたデータベースをデプロイできます。
