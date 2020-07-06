.. _mssqldeploy:

-------------------------
EraでMS SQLのデプロイ
-------------------------

**このラボでは、あなたはMSSQL Software Profileを作成し、Eraを使って、新しいMSSQL Database Serverをデプロイします。**

新しいMSSQL Database Serverの作成
++++++++++++++++++++++++++++++++++++

あなたは既にSQL Server VMをプロビジョニングするのに必要な操作を全て完了しました。以下のステップに従い、Eraにより自動適用されたベストプラクティスで、新しいデータベースサーバのデータベースをプロビジョニングします。

#. **Era** にて、ドロップダウンメニューから **Databases** を選択し、左側メニューから **Sources** を選択します。

#. **+ Provision > Single Node Database** をクリックします。

   .. figure:: images/18.png

#. **Provision a Database** ウィザードにて、以下の通り入力し、データベースサーバを設定します。

   - **Engine** - Microsoft SQL Server
   - **Database Server** - Create New Server
   - **Database Server Name** - *Initials*\ -MSSQL2
   - **Description** - (Optional)
   - **Software Profile** - *Initials*\ _MSSQL_2016
   - **Compute Profile** - CUSTOM_EXTRA_SMALL
   - **Network Profile** - Primary_MSSQL_NETWORK
   - **Database Time Zone** - Pacific Standard time
   - **Join Domain** を選択します。
   - **Windows Domain Profile** - NTNXLAB
   - **Windows License Key** - (空白のまま)
   - **Administrator Password** - Nutanix/4u
   - **Instance Name** - MSSQLSERVER
   - **Server Collation** - SQL_Latin1_General_CP1_CI_AS
   - **Database Parameter Profile** - DEFAULT_SQLSERVER_INSTANCE_PARAMS
   - **SQL Service Startup Account** - ntnxlab.local\\Administrator
   - **SQL Service Startup Account Password** - nutanix/4u
   - **SQL Server Authentication Mode** - Windows Authentication
   - **Domain User Account** - (空白のまま)

   .. figure:: images/19a.png

   .. note::

      **Instance Name** はデータベースサーバの名前で、これはホスト名ではありません。デフォルトは、 **MSSQLSERVER** です。異なるインスタンス名にする限り、同じサーバ上にMSSQLの異なるインスタンスをインストールできます。しかし、物理サーバ上で普通ですが、同じサーバ上で複数のSQLインスタンスを動かすために追加のMSSQLライセンスは必要ないです。

      **サーバ照合順序** は、データベースエンジンがサーバ、データベース、カラムレベルで文字データをどのように扱うかを決める設定です。SQL Serverは、世界で異なる場所にあるユーザーとアプリケーションをサポートする言語と地域の差を処理するための大きな集合の照合順序を含みます。照合順序はデータベース上で大文字小文字の区別を制御します。あたなは、シングルインスタンス上の各データベースの異なる照合順序を持ちます。デフォルトの照合順序は **SQL_Latin1_General_CP1_CI_AS** で、以下のように起こります。

      - **Latin1** は、基本的には **ASCII** の文字コードlatin1を使って、サーバがストリングを扱うようにします。
      - **CP1** はCode Page 1252を表します。CP1252は、英語と他の西洋言語用の、Microsoft Windowsのレガシーなコンポーネントでデフォルトで用いられたラテンアルファベットのシングルバイトの文字コードです。
      - **CI** は大文字小文字を区別し、 **ABC** は **abc** と同一です。
      - **AS** はアクセントを区別し、 **ü** は **u** と同一ではないです。

      **Database Parameter Profiles** は、SQL Serverの始めの最小のサーバメモリとSQL Serverが使用する最大メモリを定義します。デフォルトでは、SQL Serverが全ての利用可能なサーバメモリを使うよう高く設定されます。認証のためにインスタンス上の他のデータベースからデータベースを分離する包含データベースの機能を有効にできます。

#. **Next** をクリックし、以下のフィールドを入力しデータベースを設定します。

   - **Database Name** - *Initials*\ -fiesta
   - **Description** - (Optional)
   - **Size (GiB)** - 200 (Default)
   - **Database Parameter Profile** - DEFAULT_SQLSERVER_DATABASE_PARAMS

   .. figure:: images/20.png

   .. note::

      プリ/ポストインストールスクリプトの共通のアプリケーションは以下を含みます。

      - データマスキングスクリプト
      - DBのモニターソリューションでデータベースを登録する
      - DNS/IPAMをアップデートするスクリプト
      - Oracle PeopleSoftのアプリケーションレベルのクローニングのような、アプリケーションのセットアップを自動化するスクリプト

#. **Next** をクリックし、以下の通り入力し、あなたのデータベースのTime Machineを設定します。

   .. note::

      .. raw:: html

        <strong><font color="red">以下の手順でBRONZE SLA を選択することは重要です。デフォルトのBRASS SLA は、継続的な保護(Continuous Protection) スナップショットは含みません。</font></strong>

   - **Name** - *initials*\ -fiesta_TM (Default)
   - **Description** - (Optional)
   - **SLA** - DEFAULT_OOB_BRONZE_SLA
   - **Schedule** - (Defaults)

   .. figure:: images/21.png

#. **Provision** をクリックし、あなたの新しいDatabase server VMと **fiesta** databaseの作成を開始します。

#. ドロップダウンメニューから **Operations** を選択し登録をモニターします。このプロセスはおよそ20分かかります。

   .. figure:: images/22.png

   .. note::

      **Operations** において、ベストプラクティスを適用するステップを見ます。

      Eraで自動設定されるベストプラクティスは以下のものがあります。

      - データベースとログファイルを複数のvDiskに分散します。
      - Windows ダイナミック ディスク や他のin-guestのボリューム管理は使用しません。
      - (ESXiでは) 複数のSCSIコントローラでvDiskを分散します。
      - 各データベースでは、複数のデータファイルを使用します（vCPUあたり1ファイル）
      - 始めのログファイルのサイズは4GB または 8GBに設定し、始めの量だけ繰り返して希望のサイズに達することができます
      - 複数のTempDBデータファイルを使用し、すべて同じサイズです。
      - 利用可能なハイパーバイザーのネットワークコントロールの機構を使用します(例えば、VMware NIOC)


プロビジョニングされたDBサーバの確認
++++++++++++++++++++++++++++++++++++

#. **Prism Element > Storage > Table > Volume Groups** にて、 **ERA_**\ *Initials*\ **_MSSQL2_\** ** をクリックし、 Virtual Disk タブ上のレイアウトを見ます。 <これは何を示しているのでしょうか？>

   .. figure:: images/23.png

#. Prismで、あなたが新しくプロビジョニングしたVMのディスクレイアウトを見ます。 <全てのこれらのディスクは何で、登録したオリジナルのVMから異なりますか?>

   .. figure:: images/24.png

#. Prismで、あなたの *Initials*\ **-MSSQL2** VM のIPアドレスをメモして、以下の認証情報でRDP経由でそれに接続ください。

   - **User Name** - NTNXLAB\\Administrator
   - **Password** - nutanix/4u

#. **Start > Run > diskmgmt.msc** を開いて、in-guestのディスクレイアウトを見ます。ラベルのないボリュームを右クリックし、 **Change Drive Letter and Paths** を選択して、Eraがボリュームをマウントしたパスを見ます。あなたが手動でベストプラクティスを適用したオリジナルのSQL Serverと同じように、SQLのデータとログの位置に対応する占有ドライブがありますので確認ください。

   .. figure:: images/25.png

Fiesta App データの移行
+++++++++++++++++++++++++

この演習では、あなたは他のデータベースからエクスポートしたバックアップから、あなたのデータベースに直接データをインポートします。これはデータ移行の適切な方法ですが、アプリケーションのダウンタイムを伴う可能性があり、データベースは最新のデータを持っていない可能性があります。

他のアプローチでは、あなたの新しいデータベースを既存のデータベースクラスタ(AlwaysOn Availability Group)に加えることになり、あなたのEraでプロビジョニングしたデータベースにレプリケーションします。アプリケーションレベルの同期または非同期のレプリケーション(SQL Server AAGやOracle RACなど)は、クローニングとタイムマシンのようなEraのメリットを、業務のインスタンスがベアメタルまたはNutanixでないインフラ上で稼働するデータベースに提供します。

#. あなたの *Initials*\ **-MSSQL2** のRDPのセッションから、 **Microsoft SQL Server Management Studio** を起動し、 **Connect** をクリックし、現在のログインユーザとして認証します。

   .. figure:: images/26.png

#. *Initials*\ **-fiesta** を開いて、テーブルが含まれていないことを確認します。データベースを選択して、メニューから **New Query** をクリックし、あなたの業務用アプリケーションデータをインポートします。

   .. figure:: images/27.png

#. 以下のスクリプトをquery editorにコピー&ペーストし、 **Execute** をクリックします。

   .. literalinclude:: FiestaDB-MSSQL.sql
     :caption: FiestaDB Data Import Script
     :language: sql

   .. figure:: images/28.png

#. ステータスバーで **Query executed successfully** が表示されていることを確認します。

#. **New Query** をクリックし以下を実行して、データベースの内容を見ます。

   .. code-block:: sql

      SELECT * FROM dbo.products
      SELECT * FROM dbo.stores
      SELECT * FROM dbo.InventoryRecords

   .. figure:: images/29.png

#. **Era > Time Machines** で、あなたの *initials*\ **-fiesta_TM** Time Machineを選択します。 **Actions > Log Catch Up > Yes** を選択して、次のラボでクローン操作する前に、インポートされたデータがディスクにフラッシュされたか確認します。

Fiesta Web 層のプロビジョニング
+++++++++++++++++++++++++

**SQL Server Management Studio** でのデータの操作はたいくつです。このセクションでは、アプリケーションのWeb層をデプロイし、あなたの業務用のデータベースに接続します。

#. `ここを右クリックしてFiesta Blueprintをダウンロードします。 <https://raw.githubusercontent.com/nutanixworkshops/EraWithMSSQL_Bootcamp-Japanese/master/deploy_mssql_era/FiestaNoDB.json>`_ このシングルVMのBlueprintはアプリケーションのWeb層の部分のみをプロビジョニングするのに用いられます。

#. **Prism Central > Calm** から、左側メニューの **Blueprints** を選択し、 **Upload Blueprint** をクリックします。

   .. figure:: images/30.png

#. **FiestaNoDB.json** を選択します。

#. **Blueprint Name** を更新してあなたのイニシャルを含めてください。異なるプロジェクトで、Calm Blueprint名は重複しないようにする必要があります。

#. *Initials*\ -Project をCalmプロジェクトとして選択し、 **Upload** をクリックします。

   .. figure:: images/31.png

#. Blueprintを起動するために、あなたはまずネットワークをVMに割り当てる必要があります。 **NodeReact** サービスを選択し、右側の **VM** Configuration メニューで、 **Secondary** を **NIC 1** のネットワークとして選択します。

   .. figure:: images/32a.png

#. **Credentials** をクリックし、BlueprintによってプロビジョニングされるCentOS VMへの認証に使用される秘密鍵を定義します。

#. **CENTOS** credentialを開いて、あなたの希望するSSHの鍵を使用します。または、以下の値を **SSH Private Key** としてペーストください。

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

#. **Save** をクリックし、Blueprintの保存が完了すると、 **Back** をクリックします。

#. **Launch** をクリックし、以下のように入力します。

   - **Name of the Application** - *Initials*\ -Fiesta
   - **db_password** - nutanix/4u
   - **db_name** - *Initials*\ -fiesta (Eraでデプロイ時に設定したように)
   - **db_dialect** - mssql
   - **db_domain_name** - ntnxlab.local
   - **db_username** - Administrator
   - **db_host_address** - あなたの *Initials*\ **-MSSQL2** VM のIP

   .. figure:: images/34.png

#. **Create** をクリックします。

#. **Audit** タブを選択し、デプロイをモニタします。このプロセスは5分以下です。

   .. figure:: images/35.png

#. アプリケーションのステータスが **Running** に変わると、 **Services** タブを選択し、 **NodeReact** サービスを選択してあなたのウェブサーバーの **IP Address** を得ます。

   .. figure:: images/36.png

#. 新しいブラウザタブで \http://*NODEREACT-IP-ADDRESS:5001*/ を開き、 **Fiesta** アプリケーションにアクセスします。

   .. figure:: images/37.png

   おめでとうございます！あなたの業務のアプリケーションのデプロイが完了しました。

重要なポイント
+++++++++

このラボで学んだ重要なことは何でしょうか。

- 既存のデータベースを容易にEraに登録でき、テンプレートにすることができます。
- 開発の必要な既存のデータベースをEraに登録することができます。
- プロファイルによって管理者は公開基準に基づいてリソースをプロビジョニングできます。
- カスタマイズできるリカバリのSLAによって、あなたのアプリケーションの要件に従い、あなたは継続的、日次、月次のRPOをチューニングできる。
- Eraはたくさんのデータベースエンジンの1クリックプロビジョニングを提供し、自動のデータベースアプリケーションのベストプラクティスも含まれます。
