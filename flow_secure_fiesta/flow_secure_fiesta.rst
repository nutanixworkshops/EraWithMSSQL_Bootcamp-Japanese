.. _dbflow_secure_fiesta:

-------------------------------
Flowでアプリケーションのセキュリティ向上
-------------------------------

Flowは、アプリケーションセントリックなネットワークセキュリティ製品で、Nutanix AHVとPrism Centralに統合されています。Flowは、AHV上で稼働するVMについて、豊富なネットワークトラフィックの見える化、自動化、セキュリティを提供します。

マイクロセグメンテーションは、VMネットワークを堅牢にする簡単なポリシーベースの管理を使用するFlowのコンポーネントです。Prism Central のカテゴリ(ロジカルグループ)を使って、あなたは強力な分散ファイアウォールを作成できます。これをCalmに統合して、作成時にセキュリティの高いアプリケーションの自動デプロイができます。

**この演習では、Fiestaアプリケーションへのアクセスを制限し、アプリケーション層内のトラフィックを保護します。**

Fiesta アプリケーションの堅牢化
+++++++++++++++++++++++

Flowは革新的な多くのSystemのカテゴリを提供します。例えば、AppType、AppTier、Environmentで、迅速に仮想マシンをグループ化するのに使われます。セキュリティポリシーはこれらのカテゴリを使用するのに適用されます。これらの既存のカテゴリをすぐに使用を開始できます。または、カスタムのグループ用にあなた自身のカテゴリを追加します。

カテゴリの値を定義
........................

Prism Centralはカテゴリをメタデータとして使用してVMにタグを振り、どのポリシーが適用されるかを決定します。

#. **Prism Central** では、 :fa:`bars` **> Virtual Infrastructure > Categories** を選択します。

#. **AppType** のチェックボックスを選択し、 **Actions > Update** をクリックします。

   .. figure:: images/12.png

#. 最後の値の横にある :fa:`plus-circle` アイコンをクリックします。

#. *Initials*-**Fiesta**  をvalue名として指定する。

   .. figure:: images/13.png

#. **Save** をクリックする。

#. **AppTier** のチェックボックスを選択し、 **Actions > Update** をクリックする。

#. 最後のvalueの横にある :fa:`plus-circle` アイコンをクリックして、追加のCategoryのvalueを追加します。

#. *Initials*-**Web** をvalue名として指定します。このカテゴリはアプリケーションウェブ層に提供されます。

#. :fa:`plus-circle` をクリックし、 *Initials*-**DB** を指定します。このカテゴリは、アプリケーションのMSSQLデータベース層に適用されます。

   .. figure:: images/14.png

#. **Save** をクリックします。

セキュリティポリシーの作成
..........................

Nutanix Flowは、ネットワークセントリックのアプローチの代わりに、ワークロードセントリックのアプローチを使用するポリシー駆動のセキュリティフレームワークを含みます。そのため、ネットワーク設定の変更の仕方やデータセンター内にそれらが保存される場所に関わらず、VM間のトラフィックを精査します。ワークロードセントリックで、ネットワーク非依存のアプローチにより仮想化チームはセキュリティポリシーをネットワークセキュリティチームに依存せずに実装できます。

セキュリティポリシーはカテゴリに適用され、VM自身には適用されません。そのため、対象カテゴリ内で起動するVM数は問題ではないです。カテゴリ内のVMに関連するトラフィックは、いかなる規模でも、管理者の介入なしに堅牢化されます。

Fiestaアプリケーションを保護するセキュリティポリシーを作成します。

#. **Prism Central** において、 :fa:`bars` **> Policies > Security Policies** を選択します。

#. **Create Security Policy > Secure Applications (App Policy) > Create** をクリックします。

#. 以下の通り入力します。

   - **Name** - *Initials*-Fiesta
   - **Purpose** - Fiestaへの不必要なアクセスを制限します
   - **Secure this app** - AppType: *Initials*-Fiesta
   - **Filter the app type by category** を選択しないでください。

   .. figure:: images/18.png

#. **Next** をクリックします。

#. 進める場合は、 **Create App Security Policy** ウィザードのチュートリアルダイアログ上の、 **OK, Got it!** をクリックします。

#. セキュリティポリシーのさらに詳細な設定を行うには、 **Set rules on App Tiers, instead** をクリックします。同じルールをすべてのアプリケーションのコンポーネントへの適用はあまりしないです。

   .. figure:: images/19.png

#. **+ Add Tier** をクリックします。

#. ドロップダウンから **AppTier:**\ *Initials*-**Web** を選択します。

#. **AppTier:**\ *Initials*-**DB** 用にステップ7-8を繰り返します。

   .. figure:: images/20.png

   次に、 **インバウンド(Inbound)** のルールを定義し、どの制御、どの対象のソースに対して許可を与え、あなたのアプリケーションと通信するかを決めます。全てのインバウンドトラフィックを許可できますし、ホワイトリストの対象のソースを定義することもできます。デフォルトでは、全てのインカミングのトラフィックを拒否するように、セキュリティポリシーがセットされています。

   このシナリオでは、全てのクライアントからのTCP port 80上のweb層に対するインバウンドTCPトラフィックを許可したいです。

#. **Inbound** にて、 **+ Add Source** をクリックします。

#. 以下を入力して全てのインバウンドのIPアドレスを許可します。

   - **Add source by:** - **Subnet/IP** を選択します
   - **0.0.0.0/0** を指定します

   .. note::

     ソースはCategoriesによっても指定でき、ネットワークの変更に関わらずこの指定はVMを追跡し、ネットワークのさらなる柔軟性を可能にします。

#. インバウンドのルールを作るためには、 **AppTier:**\ *Initials*-**Web** の左にある **+** アイコンを選択します。

   .. figure:: images/21.png

#. 以下の通り入力します。

   - **Protocol** - TCP
   - **Ports** - 80

   .. figure:: images/22.png

   .. note::

     たくさんのプロトコルとポートを１つのルールに加えることができます。

#. **Save** をクリックします。

   Calmもweb VMへのアクセスを必要とし、スケールアウト、スケールイン、アップグレードを含むワークフローで利用されます。CalmはSSHを通してTCP port22を使って、これらのVMと通信します。

#. **Inbound** にて、 **+ Add Source** をクリックします。

#. 以下の通り入力します。

   - **Add source by:** - **Subnet/IP** を選択します
   - *あなたのPrism Central IP*\ /32 を指定します

   .. note::

     **/32** は、サブネットの幅とは異なり、1つのIPを定義します。

   .. figure:: images/23.png

#. **Add** をクリックします。

#. **AppTier:**\ *Initials*-**Web** の左側にある **+**  アイコンを選択し、 **TCP** port **22** を指定し、 **Save** をクリックします。

#. **AppTier:**\ *Initials*-**DB** に関しステップ15から18を繰り返し、EraサーバのIPアドレスがTCP port  **1433** 上のデータベースと通信できるように許可します。

   .. figure:: images/24.png

   デフォルトでは、セキュリティポリシーによりアプリケーションは全てのアウトバウンド(Outbound)トラフィックをどの送付先にも送ることができます。これは必要に応じて制限できます。しかし、この例に関しては、すべてのアウトバウンドのアクセスを許可しましょう。

#. **Outbound** にて、ドロップダウンメニューから **Allow All** を選択します。

   .. figure:: images/25.png

   アプリケーションの各層が他の層と通信します。そして、ポリシーはこのトラフィックを許可する必要があります。webのようないくつかの層では他の層との通信を必要としません。

#. イントラのappの通信を定義するために、 **Set Rules within App** をクリックします。

   .. figure:: images/27.png

#. **AppTier:**\ *Initials*-**Web** をクリックし、 **No** を選択して、この層でのVM間の通信を妨げます。層内のシングルのweb VMだけがあります。

#. **AppTier:**\ *Initials*-**Web** がいまだ選択されている間、 :fa:`plus-circle` アイコン をクリックし、 **AppTier:**\ *Initials*-**DB** の右側のアイコンをクリックし、層対層のルールを作成します。

#. 以下の通り入力して、webとdetabase層の間のTCP port **1433** 上の通信を許可します。

   - **Protocol** - TCP
   - **Ports** - 1433

   .. figure:: images/28.png

#. **Save** をクリックします。

#. **Next** をクリックし、セキュリティポリシーをレビューします。

#. **Save and Monitor** をクリックし、ポリシーを保存します。

カテゴリの値の割当
.........................

Fiesta blueprintからプロビジョニングしたVMに対して以前作成したカテゴリを適用します。Flow カテゴリはCalm blueprintの一部として割り当てられます。しかし、この演習の目的は、既存の仮想マシンに対するカテゴリ割り当てを理解することです。

#. **Prism Central** において、 :fa:`bars` **> Virtual Infrastructure > VMs** を選択します。

#. **Filters** をクリックし、 *あなたのInitials* を **NAME** の項目に入力して、あなたのデータベース VMを表示します。

   .. figure:: images/15.png

#. チェックボックスを使用して、アプリケーションに関連付けられたDB VMを選択し、 **Actions > Manage Categories** を選択します。

   .. figure:: images/16.png

#. search barに、 **AppType:**\ *Initials*-**Fiesta** をタイプし、 :fa:`plus-circle` アイコンをクリックして、2番目のカテゴリを加えます。

#. **AppTier:**\ *Initials*-**DB** を入力して、 **Save** を選択してカテゴリをVMに適用します。

   .. figure:: images/16a.png

#. **Filters** をクリックし、 **Categories** の項目内に *あなたのイニシャル* を入力して、 **CalmApplication:\ *XYZ_Fiesta*** カテゴリの一部であるweb VMを表示します。

   .. figure:: images/16b.png

#. あなたの *nodereact* VMを選択して、 **Actions > Manage Categories** を選択します。 **AppTier:**\ *Initials*-**Web** カテゴリを指定し、 :fa:`plus-circle` アイコンをクリックして2番目のカテゴリを加えます。

#. **AppType:**\ *Initials*-**Fiesta** を入力して、 **Save** をクリックします。

   .. figure:: images/17.png

#. 最後に、ステップ7を繰り返して、 **Environment:Dev** をあなたのWindows Tools VMに割り当てます。

セキュリティポリシーのモニタリングと適用
+++++++++++++++++++++++++++++++++++++++++

Flowポリシーを適用する前に、Fiesta アプリケーションは期待通りに機能することを確認します。

アプリケーションのテスト
.......................

#. **Prism Central > Virtual Infrastructure > VMs** から、あなたの **-nodereact...** and **-MSSQL-...** VMのIPアドレスを確認します。

#. あなたの *Initials*\ **-WinToolsVM** VM のコンソールを起動します。

#. *Initials*\ **-WinToolsVM** コンソールから、ブラウザを開き、http://node-VM-IP/ にアクセスします。

#. アプリケーションがロードし、product(製品)が加えられて削除できることを確認します。

   .. figure:: images/30.png

#. **コマンドプロンプト(Command Prompt)** を開き、 ``ping -t MSSQL-VM-IP`` を実行し、クライアントとデータベース間で接続を確認します。pingを起動したままにしてください。

#. 2つ目の **コマンドプロンプト(Command Prompt)** を開き、 ``ping -t node-VM-IP`` を実行し、クライアントとウェブサーバ間の接続を確認します。pingを起動したままにしてください。

   .. figure:: images/31.png

Flow による見える化
........................

#. **Prism Central** に戻り、 :fa:`bars` **> Virtual Infrastructure > Policies > Security Policies >**\ *Initials*-**Fiesta** を選択します。

#. **Environment: Dev** はインバウンドのソースとして表示されることを確認します。ソースとラインは黄色で表示され、トラフィックがあなたのクライアントVMから検知されたことを示します。

   .. figure:: images/32.png

   他に検知されたアウトバウンドトラフィックフローはありますか？これらの接続にカーソルを重ねて、どのポートが使用中か決定します。

#. **Update** をクリックし、ポリシーを編集します。

   .. figure:: images/34.png

#. **Next** をクリックし、検知されたトラフィックフローが投入するのを待ちます。

#. **AppTier:**\ *Initials*-**Web** に接続する **Environment: Dev** ソースにマウスを重ねて、表示される :fa:`check` アイコンをクリックします。

   .. figure:: images/35.png

#. **OK** をクリックし、ルールの追加を完了します。

   **Environment: Dev** ソースは青色になるはずで、ポリシーの一部を示します。フローラインにマウスを重ねて、両方のICMP(ping traffic)とTCP port 80が表示されるか確認します。

#. **Next > Save and Monitor** をクリックし、ポリシーを更新します。

Flowのポリシーの適用
......................

あなたが定義したポリシーを実施するためには、ポリシーが適用されている必要があります。

#. *Initials*-**Fiesta** を選択して、 **Actions > Apply** をクリックします。

   .. figure:: images/36.png

#. 確認ダイアログで **APPLY** をタイプし、 **OK** をクリックしてトラフィックのブロックを開始します。

#. *Initials*\ **-WinToolsVm** コンソールに戻ります。

   Windowsクライアントからデータベースサーバへの継続的なpingトラフィックで何が起こるでしょうか。このトラフィックはブロックされますか？

#. ウェブブラウザとウェブサーバのIPアドレスを使って、Windows クライアント VMがFiesta アプリケーションにアクセスすることを確認します。

   **Products** 内の新しいproductを追加して、 **Inventory** 内のproductの量を更新できますか?

重要なポイント
+++++++++

- データセンター内で発生して水平方向にマシンからマシンへ広がる悪意のある脅威に対し、マイクロセグメンテーションは追加の保護を提供します。
- Prism Centralで作成されたカテゴリはCalm blueprint内で利用可能です。
- セキュリティポリシーは、Prism Centralにあるテキストベースのカテゴリを利用します。
- Flowは、AHV上のVMのあるポートとプロトコル上のトラフィックを制限します。
- ポリシーは **Monitor** モードで作成され、トラフィックがポリシーが適用されるまでブロックされないことを意味します。これは、接続を学習し、意図せずにトラフィックがブロックされないようにするのに役立ちます。
