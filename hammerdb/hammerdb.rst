.. _hammerdb:

------------------------------------------------
HammerDBを使ったMSSQLの性能試験
------------------------------------------------

このボーナスラボでは、あなたはHammerDBツールをインストールし、それを使って、対象VM上のMSSQL データベース性能をベンチマークします。


ラボアジェンダ
+++++++++++

#. 以下のベストプラクティスに従い設定されたMS SQLデータベース上のデータベース性能をテストする

#. ベストプラクティスに従わずに設定されたMS SQLデータベース上のデータベース性能をテストします。ここでは、データベースファイルは１つのOSドライブ上にあります。

   .. note::
      このラボは60-90分かかり、MS SQL データベースのあなたの習熟度に依存します。

HammerDBのインストール
++++++++++++++++++++

#. あなたの *Initials*-MSSQL-Manual VMを選択し、 **Actions > Power On** をクリックします。

#. Remote Desktop Client/Console を使って、VMにログインします。その際、 :ref:`deploy_mssql` ラボ内の、 **手動でのVMのデプロイ** であなたが設定した管理者パスワードを使用します。

#. HammerDBのセットアップバイナリを `ここから <http://10.42.194.11/workshop_staging/HammerDB/HammerDB-3.3-Win-x86-64-Setup.exe>`_ あなたのVM上にダウンロードします。(リンクアドレスのコピー)

#. ダウンロードした場所にいき、そのファイルを右クリックの上、Advanced をクリックします。

#.  `ここの <https://www.hammerdb.com/docs/ch01s04.html#d0e166>`_ インストラクションを使って、 ``C:`` ドライブ(デフォルト)内にHammerDBをインストールするようにします。

   .. note::
      インストールのURLがうまくいかないとき、標準のwindowsパッケージをインストールするように **exe** ファイルをインストールします。 **Next** をクリックするくらい簡単です。

#. インストールすると、HammerDBウィンドウを **閉じます** 。(HammerDBを起動することを選ぶ場合)

   .. figure:: images/1.png

データベーステスト1 (SQLのベストプラクティスを利用)
+++++++++++++++++++++++++++++++++++++++++++++

この演習では、サンプルデータベース(tpcc)を作成し、異なるドライブ上のデータベースファイルのためのMSSQL ベストプラクティスに従います。いったん作成されると、HammerDBツールを使ってデータベースにデータが投入され、IOテストを実行します。

これはあなたにHammerDBツールの使い方を学ぶ機会を与え、実際にDB性能をテストする状況の準備をします。

#. VMのWindowsメニューから、SQL Server Management Studioを開きます。

#. ユーザ名：administrator と パスワード： *Nutanix/4u* を入力し、connect をクリックします。

#. Windowsエクスプローラに行き、以下のフォルダを作成します。

   ::

     E:\data
     F:\data
     G:\data
     I:\logs

#. データベースを右クリックして、 **New database**.を選択します。

   .. figure:: images/newdb.png

#. **tpcc** としてデータベース名を入力します。

#. Database filesのテーブルにおいて、tpccとtpcc_log フォルダのためのPathを選択します。

#. tpccに関して、path を ``E:\data`` にセットします。

#. tpcc_logに関して、path を ``I:\logs`` にセットします。

#. **Add** ボタンを2回クリックし、2つ以上のdatabase fileを作成します。

#. それらを **tpcc_f** と **tpcc_g** として名前を付けます。

#. pathを2つのdatabase fileにセットします。 ``F:\data`` と ``G:\data`` 。

   .. figure:: images/newdbpath.png

#. テーブルが適切に作成されるようにするために、 **tpcc** DBを右クリックしてproperties を選択します。 **Files** をクリックし、フォルダのパスが正しくセットされているようにしてください。

#. ``C:\Program Files\HammerDB-*.*\Hammerdb`` (windowsのバッチファイル)に移動します。

#. **SQL** をダブルクリックします。

   .. figure:: images/dblclicksql.png

#. **SQL-Server** と **TPC-C** オプションを選択して、 **OK** をクリックします。

   .. figure:: images/selsqltpc-c.png

#. Schema Buildを開いて、Optionsをダブルクリックします。

#. **Sql Server Database** 名を **tpcc** に変更します。

#. warehouseの数を150に変更します。

#. virtual users to build schema を 16に変更します。

#. **OK** をクリックします。

   .. figure:: images/warehousevirtualusers.png

#. **Build** をダブルクリックし、OKをクリックします。データのビルドが開始します。

   .. figure:: images/dblclickbuild.png

#. **Start Transaction Counter**  をクリックしてトランザクションを観察してください。

   .. figure:: images/starttrncnt.png

   .. figure:: images/trncnt.png

#. HammerDBを **閉じない** でください。ただウィンドウを **最小化** してください。

   .. note::
      もしHammerDBをクローズしたら、テータの投入がストップします。

#. ``E\data``, ``F:\data``, ``F:\data``, ``I:\logs`` ドライブに移動して、フォルダーのサイズが増えているか確認ください。

#. データが生成されるまで待ってください。これは15GBのデータまで生成します。

   .. note::
      データの投入には15-20分かかります。

#. データが生成されると、既に最小化されているhammer dbを開いてください。

#. Destroy Virtual Users をクリックください。

   .. figure:: images/destroyvirtusers.png

#. **Driver Script > Options** をダブルクリックしてください。 **SQL Server Database** の名前が **tpcc** (前の数ステップで作成したデータベース)であるようにしてください。

#. “TPC-C driver script”を **Timed Driver Script** として選択ください。

#. 残りはそのままにして **OK** を選択します。

   .. note::
      **Option: ** IOPSをさらに激しくするための **Keying and thinking time** のオプションの使用を試すこともできます。

   .. figure:: images/drvscript.png

#. **Load** をダブルクリックします。

#. **Virtual users** に移動して、 **Options** をクリックします。

#. ポップアップウィンドウ内の **Virtual users** が17であるようにし、 **OK** をクリックします。

#. **Create** をダブルクリックし、 **Run** 操作をダブルクリックください。

   .. figure:: images/setvirtusers.png

#. IOが生成される間、 **Transactions Counter** をクリックし、 **TPM** をメモします。(もしまだ始まっていないならTPM counter をスタートしてください)

   .. figure:: images/multitpm.png

#. スクリーンショットをとって、TPMの結果を見込み顧客に送付、またはあなた自身のリファレンスに使ってください。


データベーステスト2(SQLのベストプラクティスを利用しない)
+++++++++++++++++++++++++++++++++++++++++++++++

MS SQL データベースのベストプラクティスに従わないシナリオをシミュレーションしましょう。このシナリオでは、SQL データベースのデータとログファイルが同じドライブにあります。

#. 他のデータベースに同じ手順を繰り返します。

#. database **tpcc1** を名前を付けます。

#. tpcc1に関して、pathを ``E:\data`` にセットします。

#. tpcc1_logに関して、pathを ``E:\logs`` にセットします。(logフォルダを作成)

#. HammerDBにおける上述と同じ手順を使って、データベースにデータを投入します。

#. データが投入されるまで待ちます。

   .. note::
      データ投入に15-20分かかります。

#. 上述と同じ手順を使って、データが投入されていることを確認します。

#. Destroy Virtual Users をクリックします。

   .. figure:: images/destroyvirtusers.png

#. **Driver Script > Options** をダブルクリックします。 **SQL Server Database** の名前が **tpcc1** (前の数ステップで作成したデータベース)であるようにしてください。

#. "TPC-C driver script" を **Timed Driver Script** として選択します。

#. 残りをそのままにして **OK** を選択します。

   .. note::
    **Optional:** IOPSをさらに激しくするための **Keying and thinking time** のオプションの使用を試すこともできます。

   .. figure:: images/drvscript.png

#. **Load** をダブルクリックします。

#. **Virtual users** を移動し、 **Options** をクリックします。

#. ポップアップウィンドウ内の **Virtual users** が17であるようにし、 **OK** をクリックします。

#. **Create** をダブルクリックして **Run** 操作をダブルクリックします。

   .. figure:: images/setvirtusers.png

#. IOが生成される間、 **Transactions Counter** をクリックし、 **TPM** をメモします。(もしまだ始まっていないならTPM counter をスタートしてください)

   .. figure:: images/singletpm.png

#. スクリーンショットをとって、TPMの結果を見込み顧客に送付、またはあなた自身のリファレンスに使ってください。

   .. note::
      ベストプラクティスに従わないで設定されたデータベースはベストプラクティスに従って作成されたデータベースより遅いです。
      このケースでは、データベース **tpcc1** は、データベース **tpcc** より4倍遅いです。

   .. note::
      ここで使われたテストは高いI/Oを使用することを確認ください。顧客のワークロードに合うようにあなた自身のテスト環境内で変更することを検討ください。

#. Prism Elementで **I/O メトリクス** をチェックし、HammerDBテストを実行中のVMによって使用される、I/Oパターン、レイテンシ、SSD/HDDの使用、ファイルのブロックサイズを観察できるかどうかをみます。

   .. figure:: images/vmiopattern.png

重要なポイント
++++++++++

#. HammerDBはダミーデータを生成してそれを用いてDB性能をテストする方法をあなたに提供します。

#. HammerDBはフリーで使いやすいです。

#. ベストプラクティスに従うことはSQL DB性能の鍵です。

#. 常にDBとDBサーバーを適切にサイジングします(オーバープロビジョニングやアンダープロビジョニングはしない)。

#. 性能のベンチマークをあなたの顧客にできるだけ導入します。あなたの環境は良くなります。

#. **Nutanix Era** はデータベースをベストプラクティスでデプロイします。
