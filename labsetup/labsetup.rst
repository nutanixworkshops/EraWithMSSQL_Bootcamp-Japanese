.. _labsetup:

----------------------
MSSQLのラボセットアップ
----------------------

データベースのブートキャンプへようこそ。このブートキャンプでは、Nutanixがデータベースのワークロードの理想的なプラットフォームである理由を直接体験できます。

もともと、SQL Serverを仮想化することは困難でした。それは、従来の仮想化スタックは高コストであり、SANベースのアーキテクチャは性能に影響を及ぼすからです。ビジネスとIT部門は、コスト、運用の単純化、一貫性のある予測可能な性能について議論してきました。

Nutanixエンタープライズクラウドは、これら多くの困難を取り除き、SQL Serverのようなビジネスクリティカルアプリケーションの仮想化を一段と容易にします。Acropolis分散ストレージファブリック(DSF)は、エンタープライズSANで期待された代表的な全ての機能を提供するソフトウエアデファインドのソリューションで、SANの物理的な制限やボトルネックを回避します。SQL Serverは特に以下のDSFの機能から恩恵を得ます。

- ローカライズされたI/Oと、インデックスと主要なデータベースファイル用にフラッシュを使用して操作時のレイテンシ低減
- ランダムとシーケンシャルの両方のワークロードを処理する高度な分散処理
- 新しいノードを追加可能でシステムダウンタイムや性能への影響なしにインフラを拡張可能
- バックアップ操作やビジネス継続のプロセスを単純化する、Nutanixのデータ保護とディザスタリカバリのワークフロー

ビジネスクリティカルなアプリケーションをホストする際の共通のインフラの問題を解決するだけでなく、Nutanixはデータベース管理に関わる主要な多くの問題の解決を目指します。

.. figure:: images/4.png

1000人以上の従業員を持つ500の北米企業の2018年IDCの調査によると、以下の通り推定されています。

- 77%の組織が200以上のデータベースインスタンスを業務システム内に持ちます。
- 82%は各DBの10以上のコピーを持ちます。
- 45%-60%のストレージ容量の合計はコピーデータを保管用に占有されています。
- 32%のデータベースのクローンは開発／テストの分析用に日々更新が必要です。
- コピーデータにIT部門は2020年556.3億ドルのコストを費やします。

現状のままだとストレージの使用や管理者の時間は非効率です。Nutanix Eraを検討してください。

.. figure:: images/5.png

Nutanix EraはエンタープライズクラウドでDBaaSを提供します。Nutanixエンタープライズクラウドを利用して、データ、計算資源、ソフトウェアのフルスタックの能力を得ます。Nutanix Eraはデータベースの操作の複雑さを排除し、複数のデータベースエンジン向けに、共通のAPI、CLI、コンシューマグレードのGUIを提供します。クローンのようなデータベースの操作を効率的にし、顧客のデータベース管理のTCOを低減します。


Projectの設定
+++++++++++++++++++++

このラボでは、アプリケーションをプロビジョニングするための事前に定義された複数のCalm Blueprintを利用します。

#. **Prism Central**で、 :fa:`bars` **> Services > Calm**を選択します。\

#. 左側のメニューから **Projects** を選択し、 **+ Create Project** をクリックします。

   .. figure:: images/2.png

#. 以下の通り入力します

   - **Project Name** - *Initials*\ -Project
   - **Users, Groups, and Roles** にて、 **+ User** を選択します。
      - **Name** - Administrators
      - **Role** - Project Admin
      - **Action** - Save
   - **Infrastructure** にて、 **Select Provider > Nutanix** を選択します。
   - **Select Clusters & Subnets** をクリックします。
   - *あなたにアサインされたクラスタ* を選択します。
   - **Subnets** にて、 **Primary** 、 **Secondary** を選択して、 **Confirm** をクリックします。
   -  :fa:`star` をクリックして、デフォルトネットワークとして **Primary** をマークします。

   .. figure:: images/3.png

#.  **Save & Configure Environment** をクリックします。

Windows Tools VMのデプロイ
++++++++++++++++++++++++++++

このトラックのいくつかの演習ではWindows Tools VMを利用します。以下のステップに従い、あなたの個人のVMをディスクイメージからプロビジョニングします。

#. **Prism Central** にて、 :fa:`bars` **> Virtual Infrastructure > VMs** を選択します。

#. **+ Create VM** をクリックします。

#. 以下の通り入力して、User VMのリクエストを完成します。

   - **Name** - *Initials*\ -WinToolsVM
   - **Description** - Manually deployed Tools VM
   - **vCPU(s)** - 2
   - **Number of Cores per vCPU** - 1
   - **Memory** - 4 GiB

   - **+ Add New Disk** を選択します。
      - **Type** - DISK
      - **Operation** - Clone from Image Service
      - **Image** - WinToolsVM.qcow2
      - **Add** を選択します。

   - **Add New NIC** を選択します。
      - **VLAN Name** - Secondary
      - **Add** を選択します。

#. **Save** をクリックしてVMを作成します。

#. あなたの *Initials*\ **-WinToolsVM** の電源をONにします。
