.. _dbflow_isolate_fiesta:

-----------------------------------------
Flowでデータベース環境を分離
-----------------------------------------

分離(isolation)ポリシーは、ホワイトリストの例外なしに他のグループのVMとの通信から1つのグループのVMが完全にブロックする必要がある時に用いられます。よくあるの例は、分離ポリシーを使って、 **Environment: Dev** のタグ付けされたVMを、 **Environment: Production** にあるVMから通信するのをブロックします。2つのグループ間で例外を作成するなら、分離ポリシーを使用しないでください。代わりに、アプリケーションポリシーを使って、ホワイトリストモデルを使います。

**この演習では、新しい環境カテゴリを作成してそれらをクローンされたFiesta アプリケーションVMに割り当てることで、業務用と開発用のFiesta アプリケーションを保護します。次に、分離セキュリティポリシーを作成実装して、新しく作成されたカテゴリを使って２つの環境を通信しないようにします。**

カテゴリの作成と割当
+++++++++++++++++++++++++++++++++

#. **Prism Central** において、 :fa:`bars` **> Virtual Infrastructure > Categories** を選択します。

#. **Environment** のチェックボックスを選択して、 **Actions > Update** をクリックします。

#. 追加のカテゴリの値を追加するために最後の値の側の :fa:`plus-circle` アイコンをクリックします。

#. *Initials*-**Prod** を値として指定し、業務用環境のカテゴリを追加します。

#. *Initials*-**Dev** を入力して開発環境のカテゴリを作成します。

   .. figure:: images/37.png

#. **Save** をクリックします。

#. **Prism Central** において、 :fa:`bars` **> Virtual Infrastructure > VMs** を選択します。

#. **Filters** をクリックし、 **NAME** の項目にて *Initials*-**MSSQL** を検索し、あなたの業務用と開発用のデーターベースの仮想マシンを表示します。

   .. note::

     もしあなたが以前あなたのアプリケーションVM用にラベルを作成したなら、あなたはそのラベルを検索できます。 他には、Filtersペインから **AppType:** *Initials*-**Fiesta** カテゴリを検索できます。

   .. figure:: images/38.png

#. チェックボックスを使って、業務用アプリケーションと関連している *Initials*-**MSSQL2** のデータベースVMを選択し、 **Actions > Manage Categories** を選択します。

#. search barに **Environment:**\ *Initials*-**Prod** を指定し、 **Save** アイコンをクリックして業務用カテゴリをこのVMに割り当てます。

   .. figure:: images/39.png

#. 過去のステップを繰り返して、 **Environment:**\ *Initials*-**Dev** を開発用VM *Initials*-**MSSQL2**\_ *date* に割り当てます。

#. **Filters** をクリックし、 **CATEGORIES** にある *Initials*-**_Fiesta** を検索し、あなたの業務用web VMを表示します。

   .. figure:: images/40.png

#. チェックボックスを使用して、業務用アプリケーションに関連したweb VMを選択し、 **Actions > Manage Categories** を選択します。

#. search barに **Environment:**\ *Initials*-**Prod** を指定し、 **Save** アイコンをクリックしてカテゴリをこのVMに割り当てます。

#. **Filters** をクリックし、 **CATEGORIES** 内の *Initials*-**DevFiesta** を検索し、あなたの開発用web VMを表示します。

#. search barに **Environment:**\ *Initials*-**Dev** を指定して、 **Save** アイコンをクリックし、カテゴリをこのVMに割り当てます。

分離ポリシーの作成
++++++++++++++++++++++++++++

#. **Prism Central** において、 :fa:`bars` **> Virtual Infrastructure > Policies > Security Policies** を選択します。

#. **Create Security Policy > Isolate Environments (Isolation Policy) > Create**  をクリックします。

#. 以下の通り入力します。

   - **Name** - *Initials*-Isolate-dev-prod
   - **Purpose** - *Initials* - Isolate dev from prod
   - **Isolate This Category** - Environment:*Initials*Dev
   - **From This Category** - Environment:*Initials*-Prod
   - **Apply this isolation only within a subset of the datacenter** を選択しないでください。このオプションで、ポリシーの範囲を制限でき、3番目の相互カテゴリに割り当てられたVMにのみ適用します。

   .. figure:: images/41.png

#. **Apply Now** をクリックして、ポリシーを保存してください。即座に実行を開始します。

#. 業務用データベース *Initials*\ **-MSSQL-2** のコンソールを開きます。

   業務用データベースから業務用のFiesta web VMへpingできますか？どのポリシーがこのトラフィックをブロックしますか？

   業務用のデータベースから開発用のFiesta web VMにpingできますか？

   これらのシンプルなポリシーを使って、業務用と開発用のようなVMグループ間のトラフィックをブロックでき、ラボシステムを分離します。つまり、開発とwebデータベースに分離を適用します。

Monitorモードでポリシーを導入
++++++++++++++++++++++++++++++++

#. **Prism Central** にて、 :fa:`bars` **> Virtual Infrastructure > Policies > Security Policies** を選択します。

#. *Initials*-**Isolate-dev-prod** を選択し、 **Actions > Monitor** をクリックします。

#. confirmation(確認) ダイアログにて **MONITOR** をタイプし、 **OK** をクリックし、ポリシーを無効にします。

#. *Initials*\ **-MSSQL2** コンソールに戻ります。開発用 web VMが業務用からpingを使ってアクセスできることを確認します。

重要なポイント
+++++++++

- この演習では、あなたはカテゴリを作成し、容易でネットワーク設定の変更なしに、分離のセキュリティポリシーを作成しました。
- カテゴリを作成してVMにタグ付けした後、VMは単に属するポリシーに従って動きます。
- 分離ポリシーは、アプリケーションセキュリティポリシーより高い優先度とみなされます。
