# AZ 120 モジュール 1: SAP on Azure の基盤
# ラボ 1b: Azure VM 上に Windows クラスタリングを実装する

予想時間：120 分

このラボのタスクすべては、Azure portal (PowerShell Cloud Shell セッションを含む) から実行されます  

   > **注記**: Cloud Shell を使用しない場合、ラボ仮想マシンには Az PowerShell モジュールがインストールされている必要があります ([**https://docs.microsoft.com/ja-jp/powershell/azure/install-az-ps-msi?view=azps-2.8.0**](https://docs.microsoft.com/ja-jp/powershell/azure/install-az-ps-msi?view=azps-2.8.0))。

ラボ ファイル: なし

## シナリオ
  
Azure に SAP NetWeaver を導入し、SQL Server をデータベース管理システムとして使用する準備として、Adatum Corporation は、Windows Server 2019 を実行する Azure VM にクラスタリングを実装するプロセスを検討したいと考えています。

## Objectives
  
このラボを終了すると、下記ができるようになります。

-   高可用性な SAP NetWeaver のデプロイをサポートするために必要な Azure コンピューティング リソースをプロビジョニングする。

-   高可用性な SAP NetWeaver のデプロイをサポートするように、Windows Server 2019 を実行している Azure VM の OS を構成する

-   高可用性な SAP NetWeaver のデプロイをサポートするために必要な Azure ネットワーク リソースをプロビジョニングする

## 必要条件

-   このラボで使用する予定の Azure リージョンで、十分な数の Dsv2 および Dsv3 vCPU (1 vCPU の Standard_DS1_v2 VM 1台と、4 vCPU の Standard_D4s_v3 VM 4 台) が利用可能な Microsoft Azure サブスクリプション

-   Azure Cloud Shell に対応した Web ブラウザーと Azure へのアクセスが可能なラボ コンピューター

> **注記**: リソースのデプロイには **East US** または **East US2** リージョンの使用を検討してください。

## 演習 1: 可用性の高い SAP NetWeaver デプロイをサポートするために必要な Azure コンピューティング リソースのプロビジョニング

時間: 50 分

この演習では、Windows Server 2019 を実行している Azure VM 上フェールオーバー クラスタリングを構成するために必要な Azure インフラストラクチャのコンピューティング コンポーネントをデプロイします。これには、Active Directory ドメイン コントローラーのペアをデプロイした後、同じ仮想ネットワーク内の同じ可用性セットで Windows Server 2019 を実行する Azure VM のペアをデプロイすることが含まれます。ドメイン コントローラーのデプロイを自動化するには、<https://github.com/polichtm/azure-quickstart-templates/tree/master/active-directory-new-domain-ha-2-dc> から入手できる Azure Resource Manager クイックスタート テンプレートを使用します。

### タスク 1: Azure Resource Manager テンプレートを使用して、可用性の高い Active Directory ドメイン コントローラーを実行する Azure VM のペアをデプロイする

1.  ラボ コンピューターから Web ブラウザーを起動し、https://portal.azure.com から Azure portal に移動します。

1.  プロンプトが表示された場合は、このラボで使用する Azure サブスクリプションの所有者または共同作成者のロールを使用して、職場、学校または個人の Microsoft アカウントを使用してログインします。

1.  新しい Web ブラウザー タブを開き、<https://github.com/polichtm/azure-quickstart-templates> にある Azure クイック スタート テンプレート ページに移動し、 **2 つの新しい Windows VM、新しい AD フォレスト、ドメイン、2 つの DC を1 つの可用性セットに作成する**という名前のテンプレートを検索し、 **「Azure にデプロイ」** ボタンをクリックしてデプロイを開始します。

1.  **カスタム デプロイ** ブレードで、次の設定を指定し、**「Review + create」**、次に **「作成」** をクリックしてデプロイを開始します。

    -   サブスクリプション: *Azure サブスクリプションの名前*

    -   リソース グループ: *新しいリソース グループの名前* **az12001b-ad-RG**

    -   場所: *Azure VM をデプロイできる Azure リージョン*

    > **注記**: リソースのデプロイには **East US** または **East US2** リージョンの使用を検討してください。 

    -   管理者ユーザー名: **受講生**

    -   パスワード: **Pa55w.rd1234**

    -   ドメイン名: **adatum.com**

    -   DnsPrefix: *一意の有効な DNS プレフィックス*

    -   Pdc RDP ポート: **3389**

    -   Bdc RDP ポート: **13389**

    -   _artifacts の場所: **https://raw.githubusercontent.com/polichtm/azure-quickstart-templates/master/active-directory-new-domain-ha-2-dc/**

    -   _artifacts 場所 SAS トークン: *空白のままにする*

    > **注記**: デプロイメント過程は約 35 分間かかる場合があります。次のタスクを進める前に、デプロイが完了するのを待ちます。

    > **注記**: CustomScriptExtension コンポーネントのデプロイ中に**競合**エラー メッセージが表示され、デプロイが失敗した場合は、次の手順を使用してこの問題を修復します。

       - Azure portal の**デプロイ** ブレードで、デプロイの詳細を確認し、CustomScriptExtension のインストールが失敗した VM を特定します

       - Azure portal で、前の手順で特定した VM のブレードに移動し、**拡張機能**を選択し 、**拡張機能** ブレードから CustomScript 拡張機能を削除します

       - Azure portal で、**az12001b-ad-RG** リソース グループ ブレードに移動し、**「デプロイ」** を選択し、失敗したデプロイへのリンクを選択して **「再デプロイ」** を選択し、ターゲット リソース グループ (**az12001b-ad-RG**) を選択して、ルート アカウントのパスワードを指定します (**Pa55w.rd1234**)。


### タスク 2: 新しい可用性セットで Windows Server 2019 を実行する Azure VM のペアをデプロイします。

1.  ラボ コンピューターの Azure portal で、**「+ リソースの作成」** をクリックします。

1.  **「新規」** ブレードから、次の設定を使用して**Windows Server 2019 Datacenter** のプロビジョニングを開始します。

    -   サブスクリプション: *Azure サブスクリプションの名前*

    -   リソース グループ: *新しいリソース グループの名前* **az12001b-cl-RG**

    -   仮想マシン名: **az12001b-cl-vm0**

    -   リージョン *前のタスクで Azure VM をデプロイしたのと同じ Azure リージョン*

    -   可用性オプション: **可用性セット**

    -   可用性セット: *2 つの障害ドメインと 5 つの更新ドメインを持つ* **az12001b-cl-avset** *という名前の新規可用性セット*

    -   画像: **Windows Server 2019 Datacenter**

    -   サイズ: **Standard D2s v3**

    -   ユーザー名: **受講生**

    -   パスワード: **Pa55w.rd1234**

    -   パブリック受信ポート: **Allow selected ports (選択したポートを許可する)**

    -   インバウンド ポートを選択: **RDP (3389)**

    -   Windows ライセンスを持っていますか? **いいえ**

    -   OS ディスク タイプ: **Premium SSD**

    -   仮想ネットワーク: **adVNET**

    -   サブネット名: **clSubnet** *という名前の新規サブネット* 

    -   サブネット アドレス範囲: **10.0.1.0/24**

    -   パブリック IP アドレス: **az12001b-cl-vm0-ip** *という名前の新規 IP アドレス*

    -   NIC ネットワーク セキュリティ グループ: **Basic**

    -   パブリック受信ポート: **Allow selected ports (選択したポートを許可する)**

    -   インバウンド ポートを選択: **RDP (3389)**

    -   高速ネットワーキング: **オン**

    -   この仮想マシンを既存の負荷分散ソリューションの背後に配置します。**いいえ**

    -   基本プランを有効にする (無料): **いいえ**

    -   ブート診断: **オフ**

    -   OS ゲスト診断: **オフ**

    -   システム割り当てマネージド ID: **オフ**

    -   AAD 資格情報を使用してログインする (プレビュー): **オフ**

    -   自動シャットダウンを有効にする: **オフ**

    -   バックアップを有効にする: **オフ**

    -   拡張機能: *なし*

    -   タグ: *なし*

1.  プロビジョニングが完了するのを待たず、次の手順に進みます。

1.  次の設定で別の **Windows Server 2019 Datacenter** Azure VM をプロビジョニングします。

    -   サブスクリプション: *Azure サブスクリプションの名前*

    -   リソース グループ: *このタスクで最初の **Windows Server 2019 Datacenter** Azure VM をデプロイするときに使用したリソース グループの名前*

    -   仮想マシン名: **az12001b-cl-vm1**

    -   リージョン: *このタスクで最初の **Windows Server 2019 Datacenter** Azure VM をデプロイするときに使用したものと同じ Azure リージョン*

    -   可用性オプション: **可用性セット**

    -   可用性セット: **az12001b-cl-avset**

    -   画像: **Windows Server 2019 Datacenter**

    -   サイズ: **Standard D2s v3**

    -   ユーザー名: **受講生**

    -   パスワード: **Pa55w.rd1234**

    -   パブリック受信ポート: **Allow selected ports (選択したポートを許可する)**

    -   インバウンド ポートを選択: **RDP (3389)**

    -   Windows ライセンスを持っていますか? **いいえ**

    -   OS ディスク タイプ: **Premium SSD**

    -   仮想ネットワーク: **adVNET**

    -   サブネット名: **clSubnet**

    -   パブリック IP アドレス: **az12001b-cl-vm1-ip** *という名前の新規 IP アドレス*

    -   NIC ネットワーク セキュリティ グループ: **Basic**

    -   パブリック受信ポート: **Allow selected ports (選択したポートを許可する)**

    -   インバウンド ポートを選択: **RDP (3389)**

    -   高速ネットワーキング: **オン**

    -   この仮想マシンを既存の負荷分散ソリューションの背後に配置します。**いいえ**

    -   基本プランを有効にする (無料): **いいえ**

    -   ブート診断: **オフ**

    -   OS ゲスト診断: **オフ**

    -   システム割り当てマネージド ID: **オフ**

    -   AAD 資格情報を使用してログインする (プレビュー): **オフ**

    -   自動シャットダウンを有効にする: **オフ**

    -   バックアップを有効にする: **オフ**

    -   拡張機能: *なし*

    -   タグ: *なし*

1.  プロビジョニングが完了するまで待ちます。通常、これには数分かかります。

### タスク 3: Azure VM ディスクを作成して構成する

1.  Azure portal 上の Cloud Shell で PowerShell セッションを開始します。 

    > **注記**: 現在の Azure サブスクリプションで Cloud Shell を初めて起動する場合は、Azure ファイル共有を作成して Cloud Shell ファイルを永続化するように求められます。その場合は、既定値に設定すると、自動的に生成されたリソース グループ内にストレージ アカウントが作成されます。

1. Cloud Shell ペインで、次のコマンドを実行して、変数 `$resourceGroupName` の値を、前のタスクでプロビジョニングしたリソースを含むリソース グループ名に設定します。

    ```
    $resourceGroupName = 'az12001b-cl-RG'
    ```

1.  Cloud Shell ペインで、次のコマンドを実行して、前のタスクでデプロイした最初の Azure VM に接続する 4 個のマネージド ディスクの最初のセットを作成します。

    ```
    $location = (Get-AzResourceGroup -Name $resourceGroupName).Location

    $diskConfig = New-AzDiskConfig -Location $location -DiskSizeGB 128 -AccountType Premium_LRS -OsType Windows -CreateOption Empty

    for ($i=0;$i -lt 4;$i++) {New-AzDisk -ResourceGroupName $resourceGroupName -DiskName az12001b-cl-vm0-DataDisk$i -Disk $diskConfig}
    ```

1.  Cloud Shell ペインで、次のコマンドを実行して、前のタスクでデプロイした 2 番目の Azure VM に接続する 4 個のマネージド ディスクの 2 番目のセットを作成します。

    ```
    for ($i=0;$i -lt 4;$i++) {New-AzDisk -ResourceGroupName $resourceGroupName -DiskName az12001b-cl-vm1-DataDisk$i -Disk $diskConfig}
    ```

1.  Azure portal で、前のタスク (**az12001b-cl-vm0**) でプロビジョニングした最初の Azure VM のブレードに移動します。

1.  **az12001b-cl-vm0** ブレードから、**az12001b-cl-vm0 - Disks** ブレードに移動します。

1.  **az12001b-cl-vm0 - Disks** ブレードから、az12001b-cl-vm0 に次の設定を持つデータ ディスクを接続します。

    -   LUN: **0**

    -   ディスク名: **az12001b-cl-vm0-DataDisk0**

    -   リソース グループ: *このタスクで最初の **Windows Server 2019 Datacenter** Azure VM のペアをデプロイするときに使用したリソース グループの名前*

    -   ホスト キャッシュ: **読み取り専用**

1.  前の手順を繰り返して、プレフィックス **az12001b-cl-vm0-DataDisk** を使用して残りの 3 つのディスク (合計 4 つ) を接続します。ディスク名の最後の文字と一致する LUN 番号を割り当てます。最後のディスク (LUN **3**) については、HOST CACHING を **「なし」**に設定します。

1.  変更内容を保存します｡ 

1.  Azure portal で、前のタスク (**az12001b-cl-vm1**) でプロビジョニングした 2 番目の Azure VM のブレードに移動します。

1.  **az12001b-cl-vm1** ブレードから、**az12001b-cl-vm1 - Disks** ブレードに移動します。

1.  **az12001b-cl-vm1 - Disks** ブレードから、az12001b-cl-vm1 に次の設定を持つデータ ディスクを接続します。

    -   LUN: **0**

    -   ディスク名: **az12001b-cl-vm1-DataDisk0**

    -   リソース グループ: *このタスクで最初の **Windows Server 2019 Datacenter** Azure VM のペアをデプロイするときに使用したリソース グループの名前*

    -   ホスト キャッシュ: **読み取り専用**

1.  前の手順を繰り返して、プレフィックス **az12001b-cl-vm1-DataDisk** を使用して残りの 3 つのディスク (合計 4 つ) を接続します。ディスク名の最後の文字と一致する LUN 番号を割り当てます。最後のディスク (LUN **3**) については、HOST CACHING を **「なし」** に設定します。

1.  変更内容を保存します｡ 

> **結果**: この演習を完了すると、可用性の高い SAP NetWeaver のデプロイをサポートするために必要となる Azure コンピューティング リソースのプロビジョニングを行うことができます。


## 演習 2: 可用性の高い SAP NetWeaver インストールをサポートするために、Windows Server 2019 を実行している Azure VM のオペレーティング システムの構成

時間: 40 分

### タスク 1: Windows Server 2019 Azure VM を Active Directory ドメインに結合します。

   > **注記**: このタスクを開始する前に、前の演習の最後のタスクで開始したテンプレートのデプロイが正常に完了したことを確認します。 

1.  Azure portal で、このラボの最初の演習で自動的にプロビジョニングされた **adVNET** という仮想ネットワークのブレードに移動します。

1.  **adVNET - DNS サーバー** ブレードを表示します。仮想ネットワークは、このラボの最初の演習でデプロイされたドメイン コントローラに割り当てられた Private IP Addresses を DNS サーバーとして構成されます。

1.  Azure portal 上の Cloud Shell で PowerShell セッションを開始します。 

1. Cloud Shell ペインで、次のコマンドを実行して、変数 `$resourceGroupName` の値を、前の演習でプロビジョニングした **Windows Server 2019 Datacenter** Azure VM のペアを含むリソース グループ名に設定します。

    ```
    $resourceGroupName = 'az12001b-cl-RG'
    ```

1.  Cloud Shell ペインで次のコマンドを実行して、前の演習の 2 番目のタスクでデプロイした Windows Server 2019 Azure VM を **adatum.com** Active Directory ドメインに結合します。

    ```
    $location = (Get-AzureRmResourceGroup -Name $resourceGroupName).Location

    $settingString = '{"Name": "adatum.com", "User": "adatum.com\\Student", "Restart": "true", "Options": "3"}'

    $protectedSettingString = '{"Password": "Pa55w.rd1234"}'

    $vmNames = @('az12001b-cl-vm0','az12001b-cl-vm1')

    foreach ($vmName in $vmNames) { Set-AzVMExtension -ResourceGroupName $resourceGroupName -ExtensionType 'JsonADDomainExtension' -Name 'joindomain' -Publisher "Microsoft.Compute" -TypeHandlerVersion "1.0" -Vmname $vmName -Location $location -SettingString $settingString -ProtectedSettingString $protectedSettingString }
    ```

1.  次のタスクを進める前に、スクリプトが完了するのを待ちます。


### タスク 2: 可用性の高い SAP NetWeaver インストールをサポートするために、Windows Server 2019 を実行している Azure VM のストレージを構成します。

1.  Azure portal で、このラボの最初の演習でプロビジョニングした **az12001b-cl-vm0** という仮想マシンのブレードに移動します。

1.  **az12001b-cl-vm0** ブレードから、リモート デスクトップを使用して仮想マシン ゲスト オペレーティング システムに接続します。認証を求めるプロンプトが表示されたら、次の認証情報を入力します。

    -   ユーザー名: **ADATUM\Student**

    -   パスワード: **Pa55w.rd1234**

1.  az12001b-cl-vm0 への RDP セッションのサーバー マネージャーで **ローカル サーバ** ビューに移動し、一時的に **「IE セキュリティ強化の構成」** をオフにします。

1.  az12001b-cl-vm0 への RDP セッション内で、Server Manager の **「ファイルとストレージ サービス」** -> **「サーバー」** ノードに移動します。 

1.  **記憶域プール** ビューに移動し、前の演習で Azure VM に接続したすべてのディスクが表示されていることを確認します。

1.  **「新規記憶域プール ウィザード」** を使用して、次の設定で新しい記憶域プールを作成します。

    -   名前: **Data Storage Pool**

    -   物理ディスク: *最初の 3 つの LUN 番号 (0~2) に対応するディスク番号を持つ 3 つのディスクを選択し、それらの割り当てを* **「自動」** に設定します

    > **注記**: **Chassis** 列のエントリを使用して、**LUN** 番号を識別します。

1.  **「新規仮想ディスク ウィザード」** を使用して、次の設定で新しい仮想ディスクを作成します。

    -   仮想ディスク名: **Data Virtual Disk**

    -   記憶域のレイアウト: **シンプル**

    -   プロビジョニング: **固定**

    -   サイズ: **最大サイズ**

1.  **「新規ボリューム ウィザード」** を使用して、次の設定で新しいボリュームを作成します。

    -   サーバーとディスク: *既定値を受け入れる*

    -   サイズ: *既定値を受け入れる*

    -   ドライブ文字: **M**

    -   ファイル システム: **ReFS**

    -   アロケーション ユニット サイズ: **既定**

    -   ボリューム ラベル: **データ**

1.  **記憶域プール** ビューに戻り、**「新規記憶域プール ウィザード」** を使用して、次の設定で新しい記憶域プールを作成します。

    -   名前: **ログ記憶域プール**

    -   物理ディスク: *4 つのディスクのうち最後の 1 つを選び、そのディスクの割り当てを* **「自動」** に設定します

1.  **「新規仮想ディスク ウィザード」** を使用して、次の設定で新しい仮想ディスクを作成します。

    -   仮想ディスク名: **Log Virtual Disk**

    -   記憶域のレイアウト: **シンプル**

    -   プロビジョニング: **固定**

    -   サイズ: **最大サイズ**

1.  **「新規ボリューム ウィザード」** を使用して、次の設定で新しいボリュームを作成します。

    -   サーバーとディスク: *既定値を受け入れる*

    -   サイズ: *既定値を受け入れる*

    -   ドライブ文字: **L**

    -   ファイル システム: **ReFS**

    -   アロケーション ユニット サイズ: **既定**

    -   ボリューム ラベル: **ログ**

1.  このタスクの前の手順を繰り返して、az12001b-cl-vm1 に記憶域を設定します。

### タスク 3: 可用性の高い SAP NetWeaver インストールをサポートするために、Windows Server 2019 を実行している Azure VM でフェールオーバー クラスタリングの構成を準備します。

1.  az12001b-cl-vm0 への RDP セッション内で、Windows PowerShell ISE セッションを開始し、次のコマンドを実行して、az12001b-cl-vm0 と az12001b-cl-vm1 の両方にフェールオーバー クラスタリングとリモート管理ツールの機能をインストールします。

    ```
    $nodes = @('az12001b-cl-vm1', 'az12001b-cl-vm0')

    Invoke-Command $nodes {Install-WindowsFeature Failover-Clustering -IncludeAllSubFeature -IncludeManagementTools} 

    Invoke-Command $nodes {Install-WindowsFeature RSAT -IncludeAllSubFeature -Restart} 
    ```

    > **注記**: これにより、両方の Azure VM でゲスト オペレーティングシステムが再起動されます。

1.  ラボ コンピューターで Azure portal を開き、**「+ リソースを作成」** をクリックします。

1.  **新規**ブレードから、次の設定を使用して**ストレージ アカウント**の新規作成を開始します。

    -   サブスクリプション: *お客様のAzureサブスクリプションの名称*

    -   リソース グループ: *前の演習でプロビジョニングした **Windows Server 2019 Datacenter** Azure VM のペアを含むリソース グループの名前*

    -   ストレージ アカウント名: *3 ~ 24 の英数字で構成される一意の名前*

    -   場所: *前の演習で Azure VM をデプロイしたのと同じ Azure リージョン*

    -   パフォーマンス：**Standard**

    -   アカウント種類：**ストレージ (汎用 v1)**

    -   レプリケーション：**ローカル冗長ストレージ (LRS)**

    -   接続方法: **パブリック エンドポイント (すべてのネットワーク)**

    -   セキュリティ保護された転送が必要: **有効化された**

    -   大きなファイル共有: **無効**

    -   BLOB ソフト削除: **無効**

    -   階層型名前空間: **無効**

### タスク 4: 可用性の高い SAP NetWeaver インストールをサポートするために、Windows Server 2019 を実行している Azure VM のフェールオーバー クラスタリングを構成します。

1.  Azure portal で、このラボの最初の演習でプロビジョニングした **az12001b-cl-vm0** という仮想マシンのブレードに移動します。

1.  **az12001b-cl-vm0** ブレードから、リモート デスクトップを使用して仮想マシン ゲスト オペレーティング システムに接続します。認証を求めるプロンプトが表示されたら、次の認証情報を入力します。

    -   ユーザー名: **ADATUM\\Student**

    -   パスワード: **Pa55w.rd1234**

1.  az12001b-cl-vm0 への RDP セッションで、サーバー マネージャーの 「**ツール**」 メニューから、**Active Directory 管理センター**を起動します。

1.  Active Directory 管理センター で、adatum.com ドメインのルートに**クラスター**という名前の新しい組織単位を作成します。

1.  Active Directory 管理センターで、**az12001b-cl-vm0** および **az12001b-cl-vm1** のコンピューター アカウントを **コンピューター** コンテナーから**クラスター**組織単位に移動します。

1.  az12001b-cl-vm0 への RDP セッションで、Windows PowerShell ISE セッションを開始し、次を実行して新しいクラスターを作成します。

    ```
    $nodes = @('az12001b-cl-vm0','az12001b-cl-vm1')

    New-Cluster -Name az12001b-cl-cl0 -Node $nodes -NoStorage -StaticAddress 10.0.1.6
    ```

1.  az12001b-cl-vm0 への RDP セッションから、**Active Directory 管理センター** コンソールに切り替えます。

1.  Active Directory 管理センターで、**クラスター**組織単位に移動し、**プロパティ** ウィンドウを表示します。 

1.  **クラスター**組織単位の**プロパティ**ウィンドウで、**「拡張機能」** セクションに移動し、**「セキュリティ」** タブを表示します。 

1.  **「セキュリティ」** タブで **「詳細」** ボタンをクリックして、**「Advanced Security Settings for Clusters」** (クラスターの詳細なセキュリティ設定) ウィンドウを開きます。 

1.  **「Advanced Security Settings for Computers」** (コンピューターの高度なセキュリティ設定) ウィンドウの **「アクセス許可」** タブで、**追加」** をクリックします。

1.  **「クラスターのアクセス許可エントリ」** ウィンドウで、**「プリンシパルの選択」** をクリックします

1.  **「Select User, Service Account or Group」** (ユーザー、サービス アカウントまたはグループの選択) ダイアログ ボックスで、**「オブジェクトの種類」** をクリックし、**「コンピューター」** エントリの横にあるチェックボックスを有効にし、**「OK」** をクリックします。 

1.  **「ユーザー、コンピューター、サービス アカウント、またはグループの選択」** ダイアログ ボックスに戻り、**「選択するオブジェクト名を入力する」** に **「az12001b-ascs-cl0」** と入力し、**「OK」** をクリックします。

1.  **「クラスターのアクセス許可エントリ」** ウィンドウで、**「タイプ」** ドロップダウン リストに **「許可」** が表示されるようにします。次に、**「適用先」** ドロップダウン リストで、**「このオブジェクトとすべての子オブジェクト」** を選択します。**「アクセス許可」** リストで、**「コンピューター オブジェクトの作成」** チェック ボックスと **「コンピューター オブジェクトの削除」** チェック ボックスをオンにし、**「OK」** を 2 回クリックします。

1.  Windows PowerShell ISE セッション内で、次を実行して Az PowerShell モジュールをインストールします。

    ```
    Install-PackageProvider -Name NuGet -Force

    Install-Module -Name Az -Force
    ```

1.  Windows PowerShell ISE セッション内で、次を実行して Azure AD 資格情報を使用して認証します。

    ```
    Add-AzAccount
    ```

    > **注記**: プロンプトが表示された場合は、このラボで使用する Azure サブスクリプションの所有者または共同作成者のロールを使用して、職場、学校または個人の Microsoft アカウントを使用してログインします。

1.  Windows PowerShell ISE セッション内で、次のコマンドを実行して、新しいクラスターの Cloud Witness Quorum を設定します。

    ```
    $resourceGroupName = 'az12001b-cl-RG'

    $cwStorageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName

    $cwStorageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $cwStorageAccountName).Value[0]

    Set-ClusterQuorum -CloudWitness -AccountName $cwStorageAccountName -AccessKey $cwStorageAccountKey
    ```

1.  結果の構成を確認するには、az12001b-cl-vm0 への RDP セッションのサーバー マネージャーで、**「ツール」** メニューから **「フェールオーバー クラスター マネージャー」** を起動します。

1.  **「フェールオーバー クラスター マネージャー」** コンソールで、**az12001b-cl-cl0** クラスター構成 (ノードを含む) 、監視設定およびネットワーク設定を確認します。クラスターには共有ストレージがないことに注意してください。

1.  az12001b-cl-vm0 への RDP セッションを終了します。

> **結果**: この演習を完了すると、可用性の高い SAP NetWeaver のインストールをサポートするように、Windows Server 2019 を実行している Azure VM のオペレーティング システムを構成することができます。


## 演習 3: 可用性の高い SAP NetWeaver のデプロイをサポートするために必要な Azure ネットワーク リソースのプロビジョニング

時間: 30 分

この演習では、SAP NetWeaver のクラスター化されたインストールに対応するために Azure Load Balancers を実装します。

### タスク 1: 負荷分散のセットアップが容易に行えるよう Azure VM を構成します。

   > **注記**: Stardard SKU の Azure Load Balancer のペアを設定するので、まず、負荷分散バックエンド プールとして機能する 2 つの Azure VM のネットワーク アダプターに関連付けられたパブリック IP アドレスを削除する必要があります。

1.  ラボ コンピューターで、Azure portal の Azure VM **az12001b-cl-vm0** ブレードに移動します。 

1.  **az12001b-cl-vm0** ブレードから、ネットワーク アダプターに関連付けられた**az12001b-cl-vm0-ip** という名前のパブリック IP アドレスをのブレードに移動します。

1.  **az12001b-cl-vm0-ip** ブレードで、パブリック IP アドレスをネットワーク インターフェイスから関連付けを解除し、削除します。

1.  Azure portal で、Azure VM **az12001b-cl-vm1** ブレードに移動します。 

1.  **az12001b-cl-vm1** ブレードから、ネットワーク アダプターに関連付けられた**az12001b-cl-vm1-ip** という名前のパブリック IP アドレスをのブレードに移動します。

1.  **az12001b-cl-vm1-ip** ブレードで、パブリック IP アドレスをネットワーク インターフェイスから関連付けを解除し、削除します。

1.  Azure portal で、**az12001a-vm0** Azure VM のブレードに移動します。

1.  **az12001a-vm0** ブレードから、**ネットワーク** ブレードに移動します。 

1.  **az12001a-vm0 - Networking** ブレードから、az12001a-vm0 のネットワーク インターフェイスに移動します。 

1.  az12001a-vm0 のネットワーク インターフェースのブレードから、その IP 構成ブレードに移動し、そこから **ipconfig1** ブレードを表示します。

1.  **ipconfig1** ブレードで、プライベート IP アドレスの割り当てを **Static** に設定し、変更を保存します。

1.  Azure portal で、**az12001a-vm1** Azure VM のブレードに移動します。

1.  **az12001a-vm1** ブレードから、**ネットワーク** ブレードに移動します。 

1.  **az12001a-vm1 - Networking** ブレードから、az12001a-vm1 のネットワーク インターフェイスに移動します。 

1.  az12001a-vm1 のネットワーク インターフェースのブレードから、その IP 構成ブレードに移動し、そこから **ipconfig1** ブレードを表示します。

1.  **ipconfig1** ブレードで、プライベート IP アドレスの割り当てを **Static** に設定し、変更を保存します。

### タスク 2: 受信トラフィックを処理する Azure Load Balancers を作成して構成する

1.  Azure portal で、**「+ リソースの作成」** をクリックします。

1.  **新規**ブレードから、次の設定を使用して Azure Load Balancer の新規作成を開始します。

    -   サブスクリプション: *Azure サブスクリプションの名前*

    -   リソース グループ: *このラボの最初の演習でプロビジョニングした **Windows Server 2019 Datacenter** Azure VM のペアを含むリソース グループの名前ラボ*

    -   名前: **az12001b-cl-lb0**

    -   リージョン: *このラボの最初の演習で Azure VM をデプロイしたのと同じ Azure リージョン*

    -   タイプ: **内部**

    -   SKU: **Standard**

    -   仮想ネットワーク: **adVNET**

    -   サブネット: **clSubnet**

    -   IP アドレスの割り当て: **静的**

    -   IP アドレス: **10.0.1.240**

    -   可用性ゾーン: **ゾーン冗長**

1.  ロード バランサーのプロビジョニングが完了するのを待って、Azure portal のブレードに移動します。

1.  **az12001b-cl-lb0** ブレードから、次の設定でバックエンド プールを追加します。

    -   名前: **az12001b-cl-lb0-bepool**

    -   仮想ネットワーク: **adVNET**

    -   仮想マシン: **az12001b-cl-vm0** IP アドレス: **ipconfig1**

    -   仮想マシン: **az12001b-cl-vm1** IP アドレス: **ipconfig1**

1.  **az12001b-cl-lb0** ブレードから、次の設定で正常性プローブを追加します。

    -   名前: **az12001b-cl-lb0-hprobe**

    -   プロトコル: **TCP**

    -   ポート: **59999**

    -   間隔: **5** *秒*

    -   異常しきい値: **2** *つの連続エラー*

1.  **az12001b-cl-lb0** ブレードから、次の設定でネットワーク負荷分散ルールを追加します。

    -   名前: **az12001b-cl-lb0-lbruletcp1433**

    -   IP バージョン: **Pa55w.rd1234**

    -   フロントエンド IP アドレス: **192.168.0.240 (LoadBalancerFrontEnd)**

    -   HAポート: **無効**

    -   プロトコル: **TCP**

    -   ポート: **1433**

    -   バックエンド ポート: **1433**

    -   バックエンド プール: **az12001b-cl-lb0-bepool (2 台の仮想マシン)**

    -   正常性プローブ: **az12001b-cl-lb0-hprobe (TCP:59999)**

    -   セッション永続化: **なし**

    -   アイドル タイムアウト (分): **4**

    -   Floating IP (Direct Server Return): **有効化された**

### タスク 3: 送信トラフィックを処理する Azure Load Balancers を作成して構成する

1.  Azure portal の Cloud Shell で PowerShell セッションを開始します。 

1. Cloud Shell ペインで、次のコマンドを実行して、変数 `$resourceGroupName` の値を、このラボの最初の演習でプロビジョニングした **Windows Server 2019 Datacenter** Azure VM のペアを含むリソース グループ名に設定します。

    ```
    $resourceGroupName = 'az12001b-cl-RG'
    ```

1.  Cloud Shell ペインで次のコマンドを実行して、2 番目の Load Balancer で使用するパブリック IP アドレスを作成します。

    ```
    $location = (Get-AzResourceGroup -Name $resourceGroupName).Location

    $pipName = 'az12001b-cl-lb0-pip'

    az network public-ip create --resource-group $resourceGroupName --name $pipName --sku Standard --location $location
    ```

1.  Cloud Shell ペインで次のコマンドを実行して、2 番目の Load Balancer を作成します。

    ```
    $lbName = 'az12001b-cl-lb1'

    $lbFeName = 'az12001b-cl-lb1-fe'

    $lbBePoolName = 'az12001b-cl-lb1-bepool'
   
    $pip = Get-AzPublicIpAddress -ResourceGroupName $resourceGroupName -Name $pipName

    $feIpconfiguration = New-AzLoadBalancerFrontendIpConfig -Name $lbFeName -PublicIpAddress $pip

    $bePoolConfiguration = New-AzLoadBalancerBackendAddressPoolConfig -Name $lbBePoolName

    New-AzLoadBalancer -ResourceGroupName $resourceGroupName -Location $location -Name $lbName -Sku Standard -BackendAddressPool $bePoolConfiguration -FrontendIpConfiguration $feIpconfiguration
    ```

1.  Cloud Shell ペインを閉じます。

1.  Azure portal で、Azure Load Balancer **az12001b-cl-lb1** のプロパティを表示するブレードに移動します。

1.  **az12001b-cl-lb1** ブレードで、**「バックエンド プール」** をクリックします。

1.  **az12001b-cl-lb1 - Backend pools** ブレードで、**az12001b-cl-lb1-bepool** をクリックします。

1.  **az12001b-cl-lb1-bepool** ブレードで、次の設定を指定し、**「保存」** をクリックします。

    -   仮想ネットワーク: **adVNET (4 VM)**

    -   仮想マシン: **az12001b-cl-vm0** IP アドレス: **ipconfig1**

    -   仮想マシン: **az12001b-cl-vm1** IP アドレス: **ipconfig1**

1.  **az12001b-cl-lb1** ブレードで、**「正常性プローブ」** をクリックします。

1.  **az12001b-cl-lb1 - Health probes** ブレードから、次の設定で正常性プローブを追加します。

    -   名前: **az12001b-cl-lb1-hprobe**

    -   プロトコル: **TCP**

    -   ポート: **80**

    -   間隔: **5** *秒*

    -   異常しきい値: **2** *つの連続エラー*

1.  **az12001b-cl-lb1** ブレードで、**「負荷分散ルール」** をクリックします。

1.  **az12001b-cl-lb1 - Load balancing rules** ブレードから、次の設定でネットワーク負荷分散ルールを追加します。

    -   名前: **az12001b-cl-lb1-lbharule**

    -   IP バージョン: **Pa55w.rd1234**

    -   フロントエンド IP アドレス: *既定値を受け入れる*

    -   プロトコル: **TCP**

    -   ポート: **80**

    -   バックエンド ポート: **80**

    -   バックエンド プール: **az12001b-cl-lb1-bepool (2 台の仮想マシン)**

    -   正常性プローブ: **az12001b-cl-lb1-hprobe (TCP:80)**

    -   セッション永続化: **なし**

    -   アイドル タイムアウト (分): **4**

    -   Floating IP (Direct Server Return): **無効**

### タスク 4: ジャンプ ホストをデプロイする

   > **注記**: 2 つのクラスター化された Azure VM にはインターネットから直接アクセスできないため、ジャンプ ホストとして機能する Windows Server 2019 Datacenter を実行する Azure VM をデプロイします。 

1.  ラボ コンピューターの Azure portal で、**「+ リソースの作成」** をクリックします。

1.  **「新規」** ブレードから、**Windows Server 2019 Datacenter** イメージを基に新規 Azure VM の作成を開始します。

1.  次の設定を使用して Azure VM をプロビジョニングします。

    -   サブスクリプション: Azure サブスクリプションの名前。

    -   リソース グループ: *このラボの最初の演習でプロビジョニングした **Windows Server 2019 Datacenter** Azure VM のペアを含むリソース グループの名前ラボ*

    -   仮想マシン名: **az12001b-vm2**

    -   リージョン: *このラボの最初の演習で Azure VM をデプロイしたのと同じ Azure リージョン*

    -   可用性オプション: **インフラストラクチャ冗長は必要ありません**

    -   画像: **Windows Server 2019 Datacenter**

    -   サイズ: **Standard DS1 v2** *または類似のもの*

    -   ユーザー名: **受講生**

    -   パスワード: **Pa55w.rd1234**

    -   パブリック受信ポート: **Allow selected ports (選択したポートを許可する)**

    -   インバウンド ポートを選択: **RDP (3389)**

    -   Windows ライセンスを持っていますか? **いいえ**

    -   OS ディスク タイプ: **標準 HDD**

    -   仮想ネットワーク: **adVNET**

    -   サブネット: **bastionSubnet** *という名前の新しいサブネット*

    -   アドレス範囲: **10.0.255.0/24**

    -   パブリック IP アドレス: **az12001b-vm2-ip** *という名前の新規 IP アドレス*

    -   NIC ネットワーク セキュリティ グループ: **Basic**

    -   パブリック受信ポート: **Allow selected ports (選択したポートを許可する)**

    -   インバウンド ポートを選択: **RDP (3389)**

    -   高速ネットワーキング: **オフ**

    -   この仮想マシンを既存の負荷分散ソリューションの背後に配置します。**いいえ**

    -   ブート診断: **オフ**

    -   OS ゲスト診断: **オフ**

    -   システム割り当てマネージド ID: **オフ**

    -   AAD 資格情報を使用してログインする (プレビュー): **オフ**

    -   自動シャットダウンを有効にする: **オフ**

    -   バックアップを有効にする: **オフ**

    -   拡張機能: *なし*

    -   タグ: *なし*

1.  プロビジョニングが完了するまで待ちます。通常、これには数分かかります。

1.  RDP 経由で新しくプロビジョニングされた Azure VM に接続します。 

1.  az12001b-vm2 への RDP セッション内で、プライベート IP アドレス (10.0.1.4 と 10.0.1.5) を介して az12001b-vm0 と az12001b-vm1 の両方に SSH セッションを確立できることを確認します。 

> **結果**: この演習を完了すると、可用性の高い SAP NetWeaver のデプロイをサポートするために必要となる Azure ネットワーク リソースのプロビジョニングを行うことができます

## 演習 4: ラボ リソースを削除する

時間: 10 分

この演習では、このラボでプロビジョニングしたリソースすべてを削除します。

#### タスク 1: Cloud Shell を開く

1. ポータルの上部にある **「Cloud Shell」** アイコンをクリックして Cloud Shell ペインを開き、シェルとして PowerShell を選択します。

1. Cloud Shell ペインで、次のコマンドを実行して、変数 `$resourceGroupName` の値を、このラボの最初の演習でプロビジョニングした **Windows Server 2019 Datacenter** Azure VM のペアを含むリソース グループ名に設定します。

    ```
    $resourceGroupNamePrefix = 'az12001b-'
    ```

1. Cloud Shell ペインで次のコマンドを実行して、このラボで作成したすべてのリソース グループをリストアップします。

    ```
    Get-AzResourceGroup | Where-Object {$_.ResourceGroupName -like "$resourceGroupNamePrefix*"} | Select-Object ResourceGroupName
    ```

1. このラボで作成したリソース グループのみが出力に含まれていることを確認します。これらのグループは、次のタスクで削除されます。

#### タスク 2: リソース グループの削除

1. Cloud Shell ペインで次のコマンドを実行して、新しく作成したサブネットのリソース ID を特定します。

    ```
    Get-AzResourceGroup | Where-Object {$_.ResourceGroupName -like "$resourceGroupNamePrefix*"} | Remove-AzResourceGroup -Force  
    ```

1. Cloud Shell ペインを閉じます。

> **結果**: この演習を完了すると、このラボで使用したリソースが削除されます。
