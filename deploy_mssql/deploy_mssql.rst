.. _deploy_mssql:

----------------
MS SQLのデプロイ
----------------

従来のデータベースVMの導入は以下の絵に似ています。一般的に、プロセスは、(Dev、Test、QA、Analyticsなどからの)データベースのITチケットから始まります。次に、1つ以上のチームが、必要なストレージリソースとVMの導入を行う必要があります。インフラの準備ができると、DBAがデータベースソフトウェアのプロビジョニングと設定を行う必要があります。プロビジョニングすると、ベストプラクティスとデータ保護/バックアップのポリシーを適用する必要があります。最後に、データベースがエンドユーザに渡されます。多くのやりとりがあり、多くの不和を生む可能性があります。

.. figure:: images/0.png

NutanixクラスタとEraでは、データベースのプロビジョニングと保護は、長くてもこのイントロを読む程しかかかりません。

**このラボでは、元のMSSQL VMをクローニングすることで、Microsoft SQL Server VMを導入します。このVMは、マスタイメージとして、Eraを使った追加のSQL VMを導入するためのプロファイルの作成に用いられます。**

手動でのVMのデプロイ
++++++++++++++++++++

#. **Prism Central**で、 :fa:`bars` **> Virtual Infrastructure > VMs** を選択します。

   .. figure:: images/1-1.png

#. **Create VM** をクリックします。

#. あなたにアサインされたクラスタを選択し、 **OK** をクリックします。

#. 以下の通り入力します。

   - **Name** - *Initials*-MSSQL-Manual
   - **Description** - (Optional) あなたのVMの説明。
   - **vCPU(s)** - 8
   - **Number of Cores per vCPU** - 1
   - **Memory** - 16 GiB

   - **+ Add New Disk** を選択します。
      - **Type** - DISK
      - **Operation** - Clone from Image Service
      - **Image** - MSSQL-2016-VM.qcow2
      - **Add** を選択します。

   - **Add New NIC** を選択します。
      - **VLAN Name** - *Secondary*
      - **Add** を選択します。

#. **Save** をクリックし、VMを作成します。

#. あなたのVMを選択し、 **Actions > Power On** をクリックします。

#. 電源をONにし、 **Actions > Launch Console** をクリックし、Windows Server setupを完了します。

   - **Next** をクリック
   - licensing agreementに **Accept**
   - *Nutanix/4u* を管理者パスワードとして入力し、 **Finish** をクリック

#. 設定した管理者パスワードを使ってVMにログインします。

#. Windows Firewallを全て無効にします。 **デフォルトのキーボードマッピングを変更しないでください。**

#. **ファイルエクスプローラー** を起動し、現在のシングルディスクの設定をメモします。

   .. figure:: images/1-2.png

   .. note::

      データベース VMのベストプラクティスは、OS、SQLバイナリ、データベース、TempDB、ログを、異なるディスク上に展開するのに関わります。AHVでないハイパーバイザー上で、下図に示される通り、これらのディスクは適切に複数のディスクコントローラ上に適切に展開されます。

      .. figure:: images/1-2b.png

      Nutanix上でのSQL Serverのチューニングに詳細(NUMA, ハイパースレッディング、SQL Server 設定など)については、 `Nutanix Microsoft SQL Server Best Practices Guide <https://portal.nutanix.com/#/page/solutions/details?targetId=BP-2015-Microsoft-SQL-Server:BP-2015-Microsoft-SQL-Server>`_を参照ください。

#. デスクトップ上で、PowerShell scriptのショートカット **01 - Rename Server.ps1** を起動し、以下の通り入力します。

   - **Enter the Nutanix cluster IP** - *あなたにアサインされたNutanix Cluster IP*
   - **Enter the Nutanix user name for...** - admin
   - **Enter the Nutanix password for "admin"** - *あなたのClusterのパスワード*

   スクリプトは、VM名が15文字を超えないことを確認し、サーバ名をVM名に合うように変更します。

#. VMのリブート後、ログインし、PowerShell scriptのショートカット **02 - Complete Build.ps1** を起動します。以下の通り入力します。

   - **Enter the Nutanix cluster IP** - *あなたにアサインされたNutanix Cluster IP*
   - **Enter the Nutanix user name for...** - admin
   - **Enter the Nutanix password for "admin"** - techX2020!
   - **Enter the Nutanix container name** - Default

   .. note::

      上述のスクリプトのすべての項目は大文字小文字を区別します。

   このスクリプトはベストプラクティスに従ってディスクドライブを設定し作成し、これらのドライブ上にSQLのデータファイルを配置します。SQLのシステムファイルは、D:ドライブ上に配置され、データとログファイルは異なるドライブに配置されます。

#. VMのリブート後、 **Prism** と **File Explorer** で、新しいディスク設定を確認します。

   .. figure:: images/1-3.png

   .. figure:: images/1-4.png

#. あたたの *Initials*\ **-MSSQL** にログインし、デスクトップからSQL Server Management Studioを起動します。

#. **Windows Authentication** を使って接続し、Systemのデータベースだけがプロビジョニングされて、データベースサーバが利用可能であるか確認します。

   .. figure:: images/1-5.png

   おめでとうございます。あなたは今機能するSQL Server VMがあります。このプロセスは、acliやCalmや3rdパーティツールからREST APIをコールしたりして、さらに自動化できます。プロビジョニングはデータベースのDay 1の問題を解決し、ストレージ スプロール(無秩序な増加)、クローニングやパッチマネジメントにはほとんど関係ないです。

#. このVMをシャットダウンします。

.. note::
   このVMのシャットダウンは重要で、このラボではこれ以上必要ありません。このVMを立てる目的は、デプロイしてベストプラクティスをMS SQL VMに適用することがいかに大変かを体感することが目的でした。このVMを使って、性能試験をHammerDB toolで行うことができます。

元のMSSQL VMをクローン
+++++++++++++++++++++

#. **Prism Central** で、 :fa:`bars` **> Virtual Infrastructure > VMs** を選択します。

   .. figure:: images/1.png

#. **Win2016SQLSource** のチェックボックスを選択し、 **Actions > Clone** をクリックします。

#. 以下の通り入力します。

   - **Number Of Clones** - 1
   - **Name** - *Initials*-MSSQL
   - **vCPU(s)** - 2
   - **Number of Cores per vCPU** - 1
   - **Memory** - 4 GiB

   .. figure:: images/clone_mssql_source.png

#. **Save** をクリックし、VMを作成します。

#. あなたのVMを選択し、 **Actions > Power On** をクリックします。

#. VMにログインします。(Shutdown Event Trackerで **Cancel** をクリック)

   - **Username** - Administrator
   - **Password** - **Nutanix/4u**

#. Windowsファイアウォールを全て **無効** にします。

#. SQL Server Managment Studio (SSMS)を開き、Windows認証で **接続** します。

#. **SampleDB** をブラウズして確認します。

Eraのリソースの確認
+++++++++++++++++++++++

Eraは、AHVまたはESXi上でインストールされる仮想アプライアンスとして分散されています。メモリリソースを節約するために、共有のEraサーバが既にあなたのクラスタ上にデプロイされています。

.. note::

   もし興味があれば、 `ここに <https://portal.nutanix.com/#/page/docs/details?targetId=Nutanix-Era-User-Guide-v12:era-era-installing-on-ahv-t.html>`_、Eraアプライアンスの簡単な導入ガイドがあります。

#. **Prism Central > VMs > List** で、IPアドレスの列で、 **EraServer-\*** にアサインされる **IP Addresses** と特定します。

#. 新しいブラウザタブで、 \https://*ERA-VM-IP:8443*/ を開きます。

#. 以下の認証情報でログインします。

   - **Username** - admin
   - **Password** - nutanix/4u

#. **Dashboard** のドロップダウンから、 **Administration** を選択します。

#. **Cluster Details** において、あなたのアサインされたクラスタにEraが既に設定されていることを確認ください。

   .. figure:: images/6.png

#. 左側メニューから **Era Resources** を選択ください。

#. 設定されたネットワークを確認ください。もし、 **VLANs Available for Network Profiles** にネットワークが表示されていないとき、 **Add** をクリックください。 **Secondary** VLANを選択し、 **Add** をクリックください。

   .. note::
      クラスタのIPAMを利用してアドレスを管理するので、 **Manage IP Address Pool** のチェックははずしたままにしてください。

   .. note::
      Secondary networkが既に設定されるなら、次のステップに進んでください。

   .. figure:: images/era_networks_001.png

#. ドロップダウンメニューから、 **SLAs** を選択ください。

   .. figure:: images/7a.png

   Eraはビルトインの5つのSLA(Gold, Silver, Bronze, Zero, Brass)があります。SLAはデータベースサーバのバックアップの仕方を制御します。継続的な保護、日次、週次、月次、四半期の保護の間隔の組み合わせで行われます。

#. ドロップダウンメニューから、 **Profiles** を選択します。

   Profilesはリソースと設定を事前定義し、一貫性のあるプロビジョニング環境をシンプルにし、設定のスプロール(無秩序な増加)を減らします。例えば、Compute Profileはデータベースサーバのサイズを指定し、vCPU、core per vCPU、memoryのような詳細も含みます。

#. もし **Network** 内に定義されたネットワークがないなら、 **+ Create** をクリックします。

   .. figure:: images/8.png

#. 以下のように入力し、 **Create** をクリックします。

   - **Engine** - Microsoft SQL Server
   - **Name** - Primary-MSSQL-NETWORK
   - **Public Service VLAN** - Secondary

   .. figure:: images/9.png

あなたのMSSQL VMの登録
+++++++++++++++++++++++++

Eraにデータベースサーバを登録して、データベースをリソースにデプロイでき、また、そのリソースをSoftware Profileのベースとして使用することができます。

EraでSQL Server databaseを登録する前、以下の前提条件を満たす必要があります。

- ローカルユーザアカウント、または、管理者権限のドメインユーザアカウントが必要です。
- WindowsアカウントまたはSQLログインアカウントはsysadminロールのメンバーである必要があります。
- SQL Serverインスタスはrunningの状態である必要があります。
- データベースファイルは、C:ドライブ上に配置してはいけない。
- データベースはオンラインの状態である必要があります。
- Windows remote management (WinRM)は有効にする必要があります。

.. note::

   あなたの *XYZ*\ **-MSSQL** は全てのこれらの基準を満たします。

#. **Era** において、ドロップダウンメニューから **Database Servers** を選択し、左側メニューから **List** を選択します。

   .. figure:: images/11.png

#. **+ Register** をクリックし、以下を入力します。

   - **Engine** - Microsoft SQL Server
   - **IP Address or Name of VM** - *Initials*\ -MSSQL
   - **Windows Administrator Name** - Administrator
   - **Windows Administrator Password** - Nutanix/4u
   - **Instance** - MSSQLSERVER (これは認証情報を提供後、自動入力されます)
   - **Connect to SQL Server Admin** - Windows Admin User
   - **User Name** - Administrator

   .. note::

      もし、 **Instance** が自動入力されないなら、あなたの *XYZ*\ **-MSSQL** のWindows Firewallを無効にしてください。

   .. figure:: images/12.png

   .. note::

    Eraにおける多くの操作のための **API Equivalent** をクリックし、インタラクティブなウィザードを入力します。この際、UI内で入力または選択したJSONのペイロードベースのデータや多くの言語(cURL, Python, Golang, Javascript, and Powershell)でのAPIコールのサンプルを提供します。

    .. figure:: images/17.png

#. **Register** をクリックし、Database ServerをEraに取り込み始めます。

#. ドロップダウンメニューから **Operations** を選択し登録をモニターします。このプロセスはおよそ5分かかります。

   .. figure:: images/13.png

   .. note::

      あるサーバ上の既存のデータベースを登録することもでき、そのデータベースサーバを登録することもできます。

#. あなたの *Initials*\ -MSSQL VM をリブートします。

   .. note::

      このVMのリブートはEraのsoftware profileを作成するのに必要です。さもなければ、profileは壊れる可能性があり、新しいデータベースVMをプロビジョンするのに使用できません。

Software Profileの作成
+++++++++++++++++++++++++++

追加のSQL Server VMがプロビジョンされる前、Software Profileは、まず、前のステップで登録されたdatabase server VMから作成する必要があります。Software Profileは、SQL Server databaseとoperating systemを含むテンプレートです。このテンプレートは、あなたのNutanixストレージ上で隠れてクローンされたディスクイメージとして存在します。

#. ドロップダウンメニューから **Profiles** を選択し、左側のメニューから **Software** を選択します。

   .. figure:: images/14.png

#. **+ Create** をクリックし、以下の通り入力します。

   - **Engine** - Microsoft SQL Server
   - **Name** - *Initials*\ _MSSQL_2016
   - **Description** - (Optional)
   - **Database Server** - あなたの登録された *Initials*\ -MSSQL VM を選択ください。

   .. figure:: images/15.png

#. **Create** をクリックします。

#. ドロップダウンメニューから **Operations** を選択し登録をモニターします。このプロセスはおよそ5分かかります。

   .. figure:: images/16.png

   .. note::

       適切にシャットダウンしていないサーバからProfileを作成する場合、そのProfileは壊れていて、うまくプロビジョニングできない可能性があります。profileをEraに登録する前に、DBServerは適切にシャットダウンし適切に起動するようにしてください。
