.. _admin_mssqldb:

--------------------------
EraでDB管理
--------------------------

Eraで標準のデータベース管理の行い方を見ることができます。

**このラボでは、あなたはあなたのMSSQL DBを管理します**

あなたのデータベースを登録します
++++++++++++++++++++++

#. **Era** で、ドロップダウンメニューから **Databases** を選択し、左側のメニューから **Sources** を選択します。

   .. figure:: images/1.png

#. **+ Register**  をクリックし、以下の通り入力します。

   - **Engine** - Microsoft SQL Server
   - **Database is on a Server that is:** - Registered
   - **Registered Database Servers** - あたたの登録された *Initials*\ -MSSQL VM を選択します。

   .. figure:: images/2.png

   - **Unregistered Databases** - SampleDB を選択します。
   - **Database Name in Era** - *Initials*\ -LABSQLDB

   .. figure:: images/3.png

   - **Recovery Model** - Full
   - **Manage Log Backups with** - Era
   - **Name** - *Initials*\ -MSSQL_TM
   - **SLA** - DEFAULT_OOB_BRASS_SLA (no continuous replay)

   .. figure:: images/4.png

#. **Register** をクリックします。

#. ドロップダウンメニューから **Operations** を選択し登録をモニターします。このプロセスはおよそ5分かかります。

あなたのデータベースのスナップショットを取得
++++++++++++++++++++++

あなたのデータベースを手動でスナップショットを取る前、SampleDBに新しいテーブルを書き込みましょう。

データベースに新しいテーブルを書き込む
.............................

#. RDP/Consoleであなたの *Initials*\ -MSSQL VM に接続します。

#. SQL Server Managment Studio (SSMS)を開き、Windows認証で **接続** します。

#. **SampleDB** を右クリックし、 **New Query** を選択します。

   .. figure:: images/5.png

#. 以下のSQL Queryを **実行(Execute)** します。

   .. code-block:: Bash

      select * into dbo.testlabtable from sales.orders;

   .. figure:: images/6.png

#. **Tables** 上でrefreshを行い、新しいテーブルがあるか確認します。

データベースのスナップショットを手動で取る
................................

#. **Era** にて、ドロップダウンメニューから **Databases** を選択し、左側のメニューから **Sources** を選択します。

#. あなたのデータベース *Initials*\ -MSSQL_TM のTime Machine をクリックします。

   .. figure:: images/7.png

#. 完了後、 **Actions > Snapshot** をクリックします。

   .. Figure:: images/8.png

   - **Snapshot Name** - *Initials*\ -MSSQL-1st-Snapshot

   .. Figure:: images/9.png

#. **Create** をクリックします。

#. ドロップダウンメニューから **Operations** を選択し登録をモニターします。このプロセスはおよそ2-5分かかります。

あなたのデータベースサーバとデータベースのクローン
+++++++++++++++++++++++++++++++++++++

#. **Era** にて、ドロップダウンメニューから **Time Machines** を選択し、 *Initials*\ -MSSQL_TM を選択します。

#. **Actions > Clone Database > Single Node Database** をクリックします。

   - **Snapshot** - *Initials*\ -MSSQL-1st-Snapshot (Date Time)

   .. figure:: images/10.png

   - **Database Server** - Create New Server
   - **Database Server Name** - *Initials*\ -MSSQL_Clone1
   - **Compute Profile** - DEFAULT_OOB_COMPUTE
   - **Network Profile** - Primary-MSSQL-Network
   - **Administrator Password** - Nutanix/4u

   .. figure:: images/11.png

   - **Clone Name** - *Initials*\ -LABSQLDB_Clone1
   - **Database Name on VM** - SampleDB_Clone1
   - **Instance Name** - MSSQLSERVER

   .. figure:: images/12.png

#. **Clone** をクリックします。

#. ドロップダウンメニューから **Operations** を選択し登録をモニターします。このプロセスはおよそ30-50分かかります。

テーブルの削除とクローンのリフレッシュ
++++++++++++++++++++++++++++++

テーブルや他のデータを誤って削除してしまう時があり、あなたはそれを戻したいです。ここでは、テーブルを削除し、最後に取ったスナップショットからEraのクローンリフレッシュを使います。

テーブルの削除
............

#. RDP/Consoleであなたの *Initials*\ -MSSQL_Clone1 VM に接続します。

#. SQL Server Managment Studio (SSMS)を開き、Windows認証で **接続** します。

#. **SampleDB_Clone1 > Tables** と開き、 **dbo.testlabtable** 上で右クリックし、 **Delete** を選択し **OK** します。

クローンのリフレッシュ
.............

#. **Era** にて、ドロップダウンメニューから **Databases** を選択し、左側のメニューから **Clones** を選択します。

#. あなたのデータベース  *Initials*\ -LABSQLDB_Clone1 のクローンを選択し、 **Refresh** をクリックします。

   - **Snapshot** - *Initials*\ -MSSQL-1st-Snapshot (Date Time)


#. **Refresh** をクリックします。

#. ドロップダウンメニューから **Operations** を選択し登録をモニターします。このプロセスはおよそ2-5分かかります。

テーブルが戻ったことの確認
....................

#. RDP/Consoleであなたの *Initials*\ -MSSQL_Clone1 VM に接続します。

#. SQL Server Managment Studio (SSMS)を開き、Windows認証で **接続** します。

#. **SampleDB_Clone1 > Tables** と開き、 **dbo.testlabtable** があることを確認します。
