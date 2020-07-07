.. _cloning:

-------------------------------
タイムマシン、クローニング、API
-------------------------------

コピーデータマネジメント、すなわち、データベースクローニングは、クリティカルなDay 2 オペレーションで、Dev,QA,分析者などを含む多くのチームが業務と関係ないインスタンスを要求します。既に議論したように、すべてのこれらのクローンは非常に多くのストレージ容量の使用を伴います。最新の業務データに更新されるこれらのインスタンスは絶えず必要なのでさらに悪化します。

Eraはタイムマシンを提供しクローン操作を単純化します。タイムマシンは、スケジュールで定義された通り、元のデータベースのスナップショットとトランザクションログを取得・管理します。Eraに登録する、元の各データーベースについて、タイムマシンはその元のデータベース用に作成されます。あなたはクローンを作成し、(トランザクションログの使用による)ポイントインタイムまで、または、スナップショットを使って、クローンをリフレッシュします。

**このラボでは、Eraを使って、テスト環境の一部として使用される、あなたのSQL Server databaseのクローンを作成します。あなたの業務用のデーターベースに変更が行われる後、あなたはテスト環境を更新します。**

Era UIからクローニング
+++++++++++++++++++++++

この演習では、Eraのウェブインターフェースを通して、データベースをクローンするワークフローを見てみます。 **この演習の最後で、クローニングのプロセスを始めるために、Cloneをクリックしません。代わりに、次の演習では、クローンをプログラマティックに作成します。この演習は単純でUIワークフローをお見せします。**

#. **Era** において、ドロップダウンメニューから **Time Machines** を選択します。

#. あなたの業務用のデータベース (例えば、 *xyz-fiesta_TM*) に関連付けされたTime Machineを選択します。

#. **Actions > Clone Database > Single Node Database** を選択します。

   デフォルトでは、クローンは最新のポイントインタイムから作成します。代わりには、過去のポイントインタイムまたはスナップショットを明示的に指定します。

#. **Next**  をクリックします。

   .. figure:: images/1.png

   データベースは、Eraで自動プロビジョニングされる新しいサーバにクローンされるか、または、既存のサーバ上に追加のデータベースとしてクローンされます。

#. 以下の選択を行い、 **Next** をクリックします。

   - **Database Server** - Create New Server
   - **Database Server Name** - *Default*
   - **Compute Profile** - CUSTOM_EXTRA_SMALL
   - **Network Profile** - Primary-MSSQL_NETWORK
   - **Administrator Password** - nutanix/4u
   - **Join Domain** を選択します。
   - **Windows Domain Profile** - NTNXLAB
   - **Domain User Account** - ntnxlab.local\\Administrator

   .. figure:: images/2.png

#. **Cloneをクリックしないでください。** **API Equivalent (API相当) ** を選択ください。

#. あなたの入力に基づいて、プログラマティックにデータベースクローンを生成するための、Era UIによって提示される **JSON Data** と サンプル **スクリプト(Script)** をレビューします。

   .. figure:: images/3.png

#. **Close** をクリックし、 **X** をクリックして、Clone Database ウィザードを終了します。

Calmからのクローニング
+++++++++++++++++

データベースはアプリケーションではなく、多くのコンポーネントから構成されます。このdev/testワークフローについて、我々はCalmを利用し、Fiesta web層の開発コピーを行います。そして、Eraを呼び出し、プログラマティックに業務用データベースのクローンをプロビジョニングします。

#. `ここを右クリックしてFiesta Blueprintをダウンロードします。 <https://raw.githubusercontent.com/nutanix-japan/EraWithMSSQL_Bootcamp-Japanese/master/cloning_with_calm/FiestaClonedDB.json>`_

#. **Prism Central > Calm** から、左側メニューの **Blueprints** を選択し、 **Upload Blueprint** をクリックします。

#. **FiestaClonedDB.json** を選択します。

#. **Blueprint Name** を更新してあなたのイニシャルを含めてください。

#. あなたのCalmプロジェクトを選択して、 **Upload** をクリックください。

   .. figure:: images/4.png

#. Blueprintを起動するために、あなたはまずネットワークをVMに割り当てる必要があります。 **NodeReact** サービスを選択し、右側の **VM** Configuration メニューで、 **Primary** を **NIC 1** のネットワークとして選択します。

   .. figure:: images/5.png

#. **Credentials** をクリックし、BlueprintによってプロビジョニングされるCentOS VMへの認証に使用される秘密鍵を定義します。

#. **CENTOS** credentialを開いて、あなたの希望するSSHの鍵を使用します。または、以下の値を **SSHの秘密鍵(Private Key)** としてペーストください。

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

#. **era_creds** のcredential(認証情報)を開いて、 **Era** のパスワードを入力します。

   .. figure:: images/6.png

#. **Save**  をクリックし、Blueprintの保存が完了すると、 **Back** をクリックします。

#. **Launch** をクリックし、以下のように入力します。

   - **Name of the Application** - XYZ-DevFiesta
   - **cloned_db_name** - *この値を空白のままにすると、クローンされる元のデータベース名に基づいて新しいデータベースサーバを作成します*
   - **db_dialect** - mssql
   - **db_domain_name** - ntnxlab.local
   - **db_password** - nutanix/4u
   - **db_username** - Administrator
   - **era_ip** - *あなたにアサインされたEra サーバのIPアドレス*
   - **source_db_name** - *クローンされるEra データベース (Time Machine名ではないです)*

   .. note::

      パラメータはラボで表示されるものと異なる順序で表示されるかもしれません。適切な項目に正しい情報を入力するようにしてください。

   .. figure:: images/7.png

#. **Create** をクリックします。

#. **Audit** タブを選択してデプロイをモニターします。NodeReact VMは、データベースのクローンと並行してプロビジョニングされることを確認ください。ただし、web層はデータベースが利用可能かに依存するので、NodeReact VMのパッケージインストールはクローニングが完了するまで起こりません。

   .. figure:: images/8.png

   **Era > Operations** ページでデータベースクローンの進捗をモニターできます。

   .. figure:: images/9.png

   このプロセスは25分以下で完了します。

#. クローン操作が行われている間、これを機会と思って、もっとBlueprintを見てみましょう。Blueprintへ戻って、 **DBClone** サービス選択ください。 **VM** Configurationパネルを見てください。Calmは仮想マシンをデプロイしておらず、むしろ **既存のマシン(Existing Machine)** の設定を利用しています。

   .. figure:: images/10.png

#. **Services > DBClone > VM > Pre-create** の下で、Era インスタンスに接続するために起動しているスクリプトをみてください。ランタイム変数で定義されている **source_db_name** に基づいて、クローンを作成するのに必要な情報を得ます。

   .. figure:: images/11.png

#. **5CloneDb** タスクを選択して、 **スクリプト(Script)** のフィールドを最大化にします。 このスクリプト上のJSONの **ペイロード(payload)** は前の演習でEra UIによって提供されたものであることを確認ください。

   .. figure:: images/12.png

   このスクリプトの後、 **6MonitorOperation** はEraにポーリングしてクローン操作が成功して完了したかどうかを決めます。 クローンが完了すると、 **CLONE_SERVER_IP** が決まって、 **CloneDb** サービスにアサインされます。

#. **Services > NodeReact > Package > Install** にて、Fiestaアプリケーションに必要なソフトウェアをインストールしてデータベース接続を設定するために実行しているスクリプトをみてください。

   .. figure:: images/13.png

#. **ConfApp** タスクを選択し、スクリプトのフィールドを最大化します。EraでクローンされたデータベースサーバのIPアドレスを使用するためにどのようにappが設定されているか考えてみてください。

   .. figure:: images/14.png

#. Calmでは、アプリケーションステータスが **実行状態(Running)** に変更すると、 **Services** タブを選択し、 **NodeReact** サービスを選択して、あなたのウェブサーバの **IP Address** を得ることができます。

   .. figure:: images/15.png

#. 新しいブラウザタブで \http://*NODEREACT-IP-ADDRESS*/ を開き、 **Fiesta** アプリケーションの開発のインスタンスにアクセスします。

クローンされたデータベースのリフレッシュ
+++++++++++++++++++++++++++

あなたは機能する開発環境を持っています。あなたの業務用の環境内で変更を実施する時です。

#. 新しいブラウザのタブで、あなたの **業務用(Production)** Fiesta web appに戻り、 **Products > Add New Product** をクリックします。

   .. figure:: images/16.png

#. 以下の通り入力し、 **Submit** をクリックします。

   - **Product Name** - The Best Balloons
   - **Suggested Retail Price** - 100.00
   - **Product Image URL** - https://partycity6.scene7.com/is/image/PartyCity/_pdp_sq_?$_1000x1000_$&$product=PartyCity/251182
   - **Product Comments** - Everybody Knows

   .. figure:: images/17.png

#. メニューから **Stores** をクリックし、利用可能なstoreの1つから **View Store** を選択します。

#. **Add New Store Product** をクリックします。以下の通り入力し、 **Submit** をクリックします。

   - **Product Name** - The Best Balloons
   - **Local Product Price** - 99.99
   - **Initial Qty** - 1000

#. **Store Details** ページ上で加えられたproduct(製品)のiventory(インベントリ)を確認します。

   .. figure:: images/18.png

#. 異なるブラウザタブで、あなたの **Dev** Fiesta web appを開きます。 **業務用(Production)** インスタンスに加えられたproductとinventoryが表示されていないことを確認します。

#. **Era > Time Machines** で、あなたの業務用のデータベースに対応するTime Machineを選択します。 **Actions > Log Catch Up > Yes** を選択して、最新のデータベースエントリがディスクにフラッシュされたことを確認します。

   .. figure:: images/19.png

#. **Operations** ページ上で、log catch up(ログキャッチアップ)をモニターします。これはおよそ1分かかります。

   .. figure:: images/20.png

#. **Era > Databases > Clones** で、あなたのクローンされたデータベースを選択し、 **Refresh** をクリックします。

   .. figure:: images/21.png

#. デフォルトでは、データベースは最新の **ポイントインタイム(Point in Time)** にリフレッシュされます。しかし、あなたは手動で時間または個々のスナップショットを指定します。この演習の目的について、最新の時間を使用します。 **Refresh**  をクリックします。

   .. figure:: images/22.png

#. **Operations** ページ上でリフレッシュをモニターします。これはおよそ4分かかります。

#. リフレッシュが完了すると、あなたの **Dev**  Fiesta web appを開き、productとinventoryのデータがあなたの業務用データベースに合うことを確認します。

   .. figure:: images/18.png

   少しのマウスクリックで、あなたのDBAは現在の業務用データをクローンされたデータベースにプッシュできました。これはEra CLI やAPIでさらに自動化することができます。

(Optional) 追加のデータベースを既存のサーバにプロビジョニング
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

test/devの環境において、複数のデータベースを稼働するシングルデータベースサーバは一般的です。この演習では、次の世代のバージョンのFiestaアプリケーションのための追加のデータベースを、あなたの既存の開発用SQL Server VMにプロビジョニングします。

#. **Era > Databases > Sources** において、 **Provision > Single Node Database** をクリックします。

#. **Provision a Database** ウィザードにおいて、以下の通り入力して、Database Server を設定します。

   - **Engine** - Microsoft SQL Server
   - **Database Server** - Use Registered Server
   - **Name** - *あたながクローンしたDatabase Serverを選択します*

   .. figure:: images/23.png

#. **Next** をクリックし、以下の通り入力してDatabaseを設定します。

   - **Database Name** - *Initials*\ -fiesta2
   - **Description** - (Optional)
   - **Size (GiB)** - 200 (Default)
   - **Database Parameter Profile** - DEFAULT_SQLSERVER_DATABASE_PARAMS

   .. figure:: images/24.png

#. **Next**  をクリックし、以下の通り入力して、あなたのデータベースのTime Machineを設定します。

   - **Name** - *initials*\ -fiesta2_TM (Default)
   - **Description** - (Optional)
   - **SLA** - DEFAULT_OOB_BRASS_SLA
   - **Schedule** - (Defaults)

   .. figure:: images/25.png

#. **Provision** をクリックし、あなたの既存のサーバ上に、 **fiesta2**  databaseを作成します。

#. ドロップダウンメニューから **Operations**  を選択し、プロビジョニングをモニターします。このプロセスはおよそ8分かかります。

   .. figure:: images/26.png

#. 操作が完了すると、そのクローンされた開発用のDatabase ServerにRDP接続し、 **SQL Server Management Studio** で、あなたの **fiesta2**  databaseがあなたの開発用サーバ上で利用可能なことを、確認します。

   .. figure:: images/27.png

重要なポイント
+++++++++

このラボで学んだ重要なことは何でしょうか。

- Eraは、効率的なリソースの作成を簡単にします。どのポイントインタイムに対しても、ゼロバイトでデーターベースクローンを行います。
- Eraは、クローンに業務用のようなQoSを提供します。高速な作成とデータ更新を行います。
- Eraの操作は、REST APIを通して実行でき、Nutanix Calmや3rdパーティの自動化ソリューションとの統合も容易にします
