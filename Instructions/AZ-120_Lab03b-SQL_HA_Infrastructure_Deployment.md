# AZ 120 モジュール 3: SAP on Azure の実装
# ラボ 3b: Windows を実行する Azure VM に SAP アーキテクチャを実装する

予想時間：150 分

このラボのタスクすべては、Azure portal (PowerShell Cloud Shell セッションを含む) から実行されます  

   > **注記**: Cloud Shell を使用しない場合、ラボ仮想マシンには Az PowerShell モジュールがインストールされている必要があります ([**https://docs.microsoft.com/ja-jp/powershell/azure/install-az-ps-msi?view=azps-2.8.0**](https://docs.microsoft.com/ja-jp/powershell/azure/install-az-ps-msi?view=azps-2.8.0))。

ラボ ファイル: なし

## シナリオ
  
Azure に SAP NetWeaver をデプロイする準備として、Adatum Corporation は、Windows Server 2016 を実行する Azure VM への SAP NetWeaver の高可用性導入を示すデモを導入したいと考えています。

## Objectives
  
このラボを終了すると、下記ができるようになります。

-   可用性の高い SAP NetWeaver のデプロイをサポートするために必要な Azure リソースをプロビジョニングする

-   高可用性な SAP NetWeaver のデプロイをサポートするように、Windows を実行している Azure VM の OS を構成する

-   高可用性な SAP NetWeaver のデプロイをサポートするように、Windows を実行している Azure VM でクラスタリングを構成する

## 必要条件

-   可用性ゾーンをサポートする Azure リージョンで、十分な数の Dsv2 および Dsv3 vCPU (1vCPU の Standard_DS1_v2 VM を 4 台、4vCPU の Standard_D4s_v3 VM を 6 台) が利用可能な Microsoft Azure サブスクリプション

-   Azure Cloud Shell に対応した Web ブラウザーと Azure へのアクセスが可能なラボ コンピューター


## 演習 1: 可用性の高い SAP NetWeaver のデプロイをサポートするために必要な Azure リソースのプロビジョニング

時間: 60 分

このエクササイズでは、Windows クラスタリングの構成に必要な Azure インフラストラクチャ コンピューティング コンポーネントをデプロイします。これには、同じ可用性セットで Windows Server 2016 を実行する Azure VM のペアを作成する必要があります。

### タスク 1: Azure Resource Manager テンプレートを使用して、可用性の高い Active Directory ドメイン コントローラーを実行する Azure VM のペアをデプロイする

1.  ラボ コンピューターから Web ブラウザーを起動し、https://portal.azure.com から Azure portal に移動します。

1.  プロンプトが表示された場合は、このラボで使用する Azure サブスクリプションの所有者または共同作成者のロールを使用して、職場、学校または個人の Microsoft アカウントを使用してログインします。

1.  Azure portal で、「**+ リソースの作成**」 をクリックします。

1.  **新しい** ブレードから、新しい **Template deployment** **(カスタム テンプレートを使用して展開)** の作成を開始します

1.  「**カスタム デプロイ**」 ブレードの、「**GitHub クイックスタート テンプレートの読み込み**」 ドロップダウン リストから、エントリ **active-directory-new-domain-ha-2-dc-zones** を選択し、「**テンプレートの選択**」 をクリックします。

    > **注記**: または、<https://github.com/Azure/azure-quickstart-templates> の Azure クイック スタート テンプレート ページに移動して、**2 つの新しい Windows VM、新しい AD フォレスト、ドメイン、2 つの DC を別々の可用性ゾーンに作成する**という名前のテンプレートを検索し、「**Azure にデプロイ**」 ボタンをクリックしてデプロイを開始することもできます。

1.  **可用性ゾーンを使用して 2 つの DC を持つ新しい AD ドメインを作成する**というラベルの付いたブレードで、次の設定を指定し、「**購入**」 をクリックしてデプロイを開始します。

    -   サブスクリプション: *Azure サブスクリプションの名前*

    -   リソース グループ: *新しいリソース グループの名前* **az12003b-ad-RG**

    -   場所: *Azure VM をデプロイできる Azure リージョン *

    > **注記**: リソースのデプロイには **East US** または **East US2** リージョンの使用を検討してください。 

    -   管理者ユーザー名: **学生**

    -   場所: *上記で指定したのと同じ Azure リージョン*

    -   パスワード: **Pa55w.rd1234**

    -   ドメイン名: **adatum.com**

    -   DnsPrefix: *既定値を受け入れる*

    -   VM サイズ: **Standard D4S\_v3**

    -   _artifacts 場所: *規定値を受け入れる*

    -   _artifacts 場所 SAS トークン: *空白のままにする*

    -   私は上記の契約条件に同意します: *有効*

    > **注記**: デプロイメント過程は約 35 分間かかる場合があります。次のタスクを進める前に、デプロイが完了するのを待ちます。

    > **注記**: CustomScriptExtension コンポーネントのデプロイ中に**競合**エラー メッセージが表示され、デプロイが失敗した場合は、次の手順を使用してこの問題を修復します。

       - Azure portal の 「**デプロイ**」 ブレードで、デプロイの詳細を確認し、CustomScriptExtension のインストールが失敗した VM を特定します

       - Azure portal で、前の手順で特定した VM のブレードに移動し、「**拡張機能**」を選択し 、「**拡張機能**」 ブレードから CustomScript 拡張機能を削除します

       - Azure portal で、**az12003b-sap-RG** リソース グループ ブレードに移動し、「**デプロイ**」 を選択し、失敗したデプロイへのリンクを選択して 「**再デプロイ**」 を選択し、ターゲット リソース グループ (**az12003b-sap-RG** ) を選択して、ルート アカウントのパスワードを指定します (**Pa55w.rd1234** )。

### タスク 2: 可用性の高い SAP NetWeaver デプロイと S2D クラスターを実行する Azure VM をホストするサブネットをプロビジョニングする。

1.  Azure portal で、 **az12003b-ad-RG** リソース グループのブレードに移動します。

1.  **az12003b-ad-RG** リソース グループ ブレードで、リソースの一覧から **adVNET** 仮想ネットワークを検索してエントリをクリックして **adVNET** ブレードを表示します。

1.  **adVNET** ブレードから **adVNET - サブネット**ブレードに移動します。 

1.  **adVNET - サブネット** ブレードから、次の設定を使用して新しいサブネットを作成します。

    -   名前: **sapSubnet**

    -   アドレス範囲 (CIDR ブロック): **10.0.1.0/24**

1.  **adVNET - サブネット** ブレードから、次の設定を使用して新しいサブネットを作成します。

    -   名前: **s2dSubnet**

    -   アドレス範囲 (CIDR ブロック): **10.0.2.0/24**

1.  Azure portal 上の Cloud Shell で PowerShell セッションを開始します。 

    > **注記**: 現在の Azure サブスクリプションで Cloud Shell を初めて起動する場合は、Azure ファイル共有を作成して Cloud Shell ファイルを永続化するように求められます。その場合は、既定値に設定すると、自動的に生成されたリソース グループ内にストレージ アカウントが作成されます。

1. Cloud Shell ペインで、次のコマンドを実行して、変数 `$resourceGroupName` の値を、前のタスクでプロビジョニングしたリソースを含むリソース グループ名に設定します。

    ```
    $resourceGroupName = 'az12003b-ad-RG'
    ```

1.  Cloud Shell ペインで、次のコマンドを実行して、前のタスクで作成した仮想ネットワークを識別します。

    ```
    $vNetName = 'adVNet'

    $subnetName = 'sapSubnet'
    ```

1.  Cloud Shell ペインで次のコマンドを実行して、新しく作成したサブネットのリソース ID を特定します。

    ```
    $vNet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name $vNetName
    
    (Get-AzVirtualNetworkSubnetConfig -Name $subnetName -VirtualNetwork $vNet).Id
    ```

1.  結果の値をクリップボードにコピーします。これは、次のタスクで必要になります。

### タスク 3: 可用性の高い SAP NetWeaver のデプロイをホストする Windows Server 2016 を実行する Azure VM をプロビジョニングする Azure Resource Manager テンプレートをデプロイする

1.  Azure portal で、「**テンプレートのデプロイ (カスタム テンプレートを使用したデプロイ**」を検索して選択します。

1.  「**カスタム デプロイ**」 ブレードの 「**テンプレートの選択 (免責事項)」**」 ボックスの一覧で、「**sap-3-tier-marketplace-image-md**」と入力し、「**テンプレートの選択**」 をクリックします。

    > **注記**: Microsoft Edge またはサード パーティのブラウザーを使用してください。Internet Explorer は使用しないでください。

1.  **SAP NetWeaver 3-tier (managed disk)** ブレードで、**「テンプレートの編集」**.を選択します。

1.  **「テンプレートの編集」** ブレードで、次の変更を適用して **「保存」**を選択します。

    -   **197**の行で `"dbVMSize": "Standard_E8s_v3",` を `"dbVMSize": "Standard_D4s_v3",` に置換

    -   **198**の行で `"ascsVMSize": "Standard_D2s_v3",` を `"ascsVMSize": "Standard_DS1_v2",` に置換

    -   **199**の行で`"diVMSize": "Standard_D2s_v3",` を `"diVMSize": "Standard_DS1_v2",` に置換

1.  **SAP NetWeaver 3-tier (managed disk)** ブレードに戻り、以下の設定を使用してデプロイを開始します。

    -   サブスクリプション: *Azure サブスクリプションの名前*

    -   リソース グループ: *t新しいリソース グループの名前* **az12003b-sap-RG**

    -   場所: *このエクササイズの最初のタスクで指定したものと同じ Azure リージョン*

    -   SAP システム ID: **Pa55w.rd1234**

    -   スタック タイプ: **ABAP**

    -   OS タイプ: **Windows Server 2016 Datacenter**

    -   Dbtype: **SQL**

    -   Sap システム サイズ: **デモ**

    -   システムの可用性: **HA**

    -   管理者ユーザー名: **学生**

    -   認証タイプ: **パスワード**

    -   管理者パスワードまたはキー: **Pa55w.rd1234**

    -   サブネット ID: *前のタスクでクリップボードにコピーした値*

    -   Availability Zones: **1,2**

    -   場所: **[resourceGroup().location]**

    -   _artifacts の保存先: **https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/sap-3-tier-marketplace-image-md/**

    -   _artifacts の保存先の Sas トークン: *空白のままにする*

    -   私は上記の契約条件に同意します: *有効*

1.  デプロイが完了するのを待たず、代わりに次のタスクに進みます。 

### タスク 5: スケールアウト ファイル サーバー (SOFS) クラスターをデプロイする

このタスクでは、 [**https://github.com/robotechredmond/301-storage-spaces-direct-md**](https://github.com/robotechredmond/301-storage-spaces-direct-md) で利用可能な GitHub の Azure Resource Manager クイックスタート テンプレートを使用して、SAP ASCS サーバーのファイル共有をホストするスケールアウト ファイル サーバー (SOFS) クラスターをデプロイします。 

1.  ラボ コンピューターでブラウザーを起動し、[**https://github.com/robotechredmond/301-storage-spaces-direct-md**](https://github.com/robotechredmond/301-storage-spaces-direct-md) に移動します。 

    > **注記**: Microsoft Edge またはサード パーティのブラウザーを使用してください。Internet Explorer は使用しないでください。

1.  「 **マネージド ディスクを使用して、Windows Server 2016 でストレージ スペース ダイレクト (S2D) スケールアウト ファイル サーバー (SOFS) を作成する**」というタイトルのページで、「**Azure にデプロイ**」 をクリックします。これにより、Azure portal に自動的にリダイレクトされ、**カスタム デプロイ** ブレードが表示されます。

1.  **カスタム デプロイ** ブレードから、次の設定を使用してデプロイを開始します。

    -   サブスクリプション: **Azure サブスクリプションの名前**。

    -   リソース グループ: *新しいリソース グループの名前* **az12003b-s2d-RG**

    -   リージョン: *このエクササイズの最初のタスクで Azure VM をデプロイしたのと同じ Azure リージョン*

    -   名前プレフィックス: **i20**

    -   VM サイズ: **Standard D4S\_v3**

    -   高速ネットワークを有効にする: **True**

    -   イメージの SKU: **2016-Datacenter-Server-Core**

    -   VM カウント: **2**

    -   VM ディスク サイズ: **120**

    -   VM ディスク カウント: **3**

    -   既存のドメイン名: **adatum.com**

    -   管理者ユーザー名: **学生**

    -   管理者パスワード: **Pa55w.rd1234**

    -   既存の仮想ネットワーク RG 名: **az12003b-ad-RG**

    -   既存の仮想ネットワーク名: **adVNet**

    -   既存のサブネット名: **s2dSubnet**

    -   Sofs 名: **sapglobalhost**

    -   共有名: **sapmnt**

    -   スケジュールされた更新日: **日曜日**

    -   スケジュールされた更新時間: **3:00 PM**

    -   リアルタイム マルウェア対策有効: **False**

    -   スケジュールされたマルウェア対策有効: **いいえ**

    -   スケジュールされたマルウェア対策時間: **120**

    -   \_artifacts 場所: **既定値を受け入れる**

    -   \_artifacts 場所 SAS トークン: **既定値のままにする**

    -   私は上記の契約条件に同意します: *有効*

1.  デプロイには約 20 分間かかります。デプロイが完了するのを待たず、代わりに次のタスクに進みます。

### タスク 6: ジャンプ ホストをデプロイする

   > **注記**: 前のタスクでデプロイした Azure VM にはインターネットから直接アクセスできないため、ジャンプ ホストとして機能する Windows Server 2016 Datacenter を実行する Azure VM をデプロイします。 

1.  ラボ コンピューターで Azure portal インターフェイスを開き、「**+ リソースを作成**」 をクリックします。

1.  「**新規**」 ブレードから、**Windows Server 2019 Datacenter** イメージを基に新規 Azure VM の作成を開始します。

1.  次の設定を使用して Azure VM をプロビジョニングします。

    -   サブスクリプション: Azure サブスクリプションの名前。

    -   リソース グループ： 新しいリソース グループ **az12003b-dmz-RG**の名前

    -   仮想マシン名: **az12003b-vm0**

    -   リージョン: *このエクササイズの最初のタスクで Azure VM をデプロイしたのと同じ Azure リージョン*

    -   可用性オプション: **インフラストラクチャ冗長は必要ありません**

    -   画像: **Windows Server 2019 Datacenter**

    -   サイズ: **スタンダード DS1 v2**

    -   ユーザー名: **学生**

    -   パスワード: **Pa55w.rd1234**

    -   パブリック受信ポート: **Allow selected ports (選択したポートを許可する)**

    -   インバウンド ポートを選択: **RDP (3389)**

    -   Windows ライセンスを持っていますか? **いいえ**

    -   OS ディスク タイプ: **標準 HDD**

    -   仮想ネットワーク: **adVNET**

    -   サブネット: **dmzSubnet (10.0.255.0/24)** *という名前の新しいサブネット*

    -   パブリック IP: **az12003b-vm0-ip*** という名前の新しい IP アドレス*

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

    -   タグ: **なし**

1.  プロビジョニングが完了するまで待ちます。通常、これには数分かかります。

> **結果**: この演習を完了することにより、高可用性 SAP NetWeaver のデプロイをサポートするために必要となる Azure リソースのプロビジョニングを行いました。


## 演習 2: 可用性の高い SAP NetWeaver デプロイをサポートするように、Windows を実行している Azure VM のオペレーティング システムを構成する

時間: 60 分

この演習では、可用性の高い SAP NetWeaver に対応するように、Windows Server を実行している Azure VM 上のオペレーティング システムを構成します。

### タスク 1: Windows Server 2016 Azure VM を Active Directory ドメインに結合します。

   > **注記**: このタスクを開始する前に、前のエクササイズで開始したテンプレートのデプロイが正常に完了したことを確認します。 

1.  Azure portal で、このラボの最初の演習で自動的にプロビジョニングされた **adVNET** と名付けられた仮想ネットワークのブレードに移動します。

1.  **adVNET - DNS サーバー** ブレードを表示します。仮想ネットワークは、このラボの最初のエクササイズでデプロイされたドメイン コントローラに割り当てられた Private IP Addresses を DNS サーバーとして構成されます。

1.  Azure portal 上の Cloud Shell で PowerShell セッションを開始します。 

1. Cloud Shell ペインで、次のコマンドを実行して、変数 `$resourceGroupName` の値を、前のタスクでプロビジョニングしたリソースを含むリソース グループ名に設定します。

    ```
    $resourceGroupName = 'az12003b-sap-RG'
    ```

1.  「Cloud Shell」 ウィンドウで次のコマンドを実行して、前のエクササイズの 3 番目のタスクでデプロイした Windows Server Azure VM を **adatum.com** Active Directory ドメインに結合します。

    ```
    $location = (Get-AzResourceGroup -Name $resourceGroupName).Location

    $settingString = '{"Name": "adatum.com", "User": "adatum.com\\Student", "Restart": "true", "Options": "3"}'

    $protectedSettingString = '{"Password": "Pa55w.rd1234"}'

    $vmNames = @('i20-ascs-0','i20-ascs-1','i20-db-0','i20-db-1','i20-di-0','i20-di-1')

    foreach ($vmName in $vmNames) { Set-AzVMExtension -ResourceGroupName $resourceGroupName -ExtensionType 'JsonADDomainExtension' -Name 'joindomain' -Publisher "Microsoft.Compute" -TypeHandlerVersion "1.0" -Vmname $vmName -Location $location -SettingString $settingString -ProtectedSettingString $protectedSettingString }
    ```

### タスク 2: データベース層 Azure VM のストレージ構成を確認する。

1.  ラボ コンピューターの Azure portal で、**az12003b-vm0** ブレードに移動します。

1.  **az12003b-vm0** ブレードから、リモート デスクトップ経由で Azure VM az12003b-vm0 に接続します。プロンプトが表示されたら、次の認証情報を入力します。

    -   ログイン: **受講生**

    -   パスワード: **Pa55w.rd1234**

1.  az12003b-vm0 への RDP セッションから 、リモート デスクトップを使用して **i20-db-0.adatum.com** Azure VM に接続します。プロンプトが表示されたら、次の認証情報を入力します。

    -   ログイン: **ADATUM\\Student**

    -   パスワード: **Pa55w.rd1234**

1.  リモート デスクトップを使用して、同じ認証情報で **i20-db-1.adatum.com** Azure VM に接続します。

1.  i20-db-0.adatum.com への RDP セッション内で、サーバー マネージャーでファイル サービスと記憶域サービスを使用してディスク構成を確認します。データベースファイルとログファイルのストレージを提供するために、ボリュームマウントを介して単一のデータディスクが構成されていることに注意してください。 

1.  i20-db-1.adatum.com への RDP セッション内で、サーバー マネージャーでファイル サービスと記憶域サービスを使用してディスク構成を確認します。データベースファイルとログファイルのストレージを提供するために、ボリュームマウントを介して単一のデータディスクが構成されていることに注意してください。 


### タスク 3: 可用性の高い SAP NetWeaver インストールをサポートするために、Windows Server 2016 を実行している Azure VM でフェールオーバー クラスタリングの構成を準備します。

1.  RDP セッション～i20-db-0.adatum.com 内で、Windows PowerShell ISE セッションを開始し、ASCS および SQL サーバー クラスターのノードとなる ASCS と DB サーバーのペアで次のコマンドを実行して、フェールオーバー クラスタリングおよびリモート管理ツール機能をインストールします。

    ```
    $nodes = @('i20-ascs-0','i20-ascs-1','i20-db-0','i20-db-1')

    Invoke-Command $nodes {Install-WindowsFeature Failover-Clustering -IncludeAllSubFeature -IncludeManagementTools} 

    Invoke-Command $nodes {Install-WindowsFeature RSAT -IncludeAllSubFeature -Restart} 
    ```

    > **注記**: これにより、Azure VM 4 つすべてのゲストオペレーティングシステムが再起動されます。

1.  ラボ コンピューターで Azure portal を開き、「**+ リソースを作成**」 をクリックします。

1.  **「新規」** ブレードから、次の設定を使用して**ストレージ アカウント**の新規作成を開始します。

    -   サブスクリプション: *お客様のAzureサブスクリプションの名称*

    -   リソース グループ: *可用性の高い SAP NetWeaver のデプロイをホストする Azure VM をデプロイしたリソース グループの名前*

    -   ストレージ アカウント名: *3 ~ 24 の英数字で構成される一意の名前*

    -   場所: *前のエクササイズで Azure VM をデプロイしたのと同じ Azure リージョン*

    -   パフォーマンス：**Standard**

    -   アカウント種類：**ストレージ (汎用 v1)**

    -   レプリケーション：**ローカル冗長ストレージ (LRS)**

    -   接続方法: **パブリック エンドポイント (すべてのネットワーク)**

    -   セキュリティ保護された転送が必要: **有効化された**

    -   大きなファイル共有: **無効**

    -   BLOB ソフト削除: **無効**

    -   階層型名前空間: **無効**


### タスク 4: 可用性の高い SAP NetWeaver のデータベース階層のインストールをサポートするために、Windows Server 2016 を実行している Azure VM のフェールオーバー クラスタリングを構成します。

1.  必要に応じて、RDP セッションから az12003b-vm0 まで、リモート デスクトップを使用して **i20-db-0.adatum.com** Azure VM に再接続します。プロンプトが表示されたら、次の認証情報を入力します。

    -   ログイン: **ADATUM\\Student**

    -   パスワード: **Pa55w.rd1234**

1.  i20-db-0.adatum.com への RDP セッションのサーバー マネージャーで 「**ローカル サーバー**」 ビューに移動し、「**IE セキュリティ強化の構成**」 をオフにします。

1.  i20-db-0.adatum.com への RDP セッションで、サーバー マネージャーの 「**ツール**」 メニューから、**Active Directory 管理センター**を起動します。

1.  Active Directory 管理センター で、adatum.com ドメインのルートに**クラスター**という名前の新しい組織単位を作成します。

1.  Active Directory Administrative Center で、i20-db-0 および i20-db-1 のコンピューター アカウントをコンピューター コンテナーからクラスター組織単位に移動します。

1.  i20-db-0 への RDP セッションで、Windows PowerShell ISE セッションを開始し、次を実行して新しいクラスターを作成します。

    ```
    $nodes = @('i20-db-0','i20-db-1')

    New-Cluster -Name az12003b-db-cl0 -Node $nodes -NoStorage -StaticAddress 10.0.1.6
    ```

1.  i20-db-0.adatum.com への RDP セッションから、**Active Directory 管理センター** コンソールに切り替えます。

1.  Active Directory 管理センターで、 **クラスター**組織単位に移動し、「**プロパティ**」 ウィンドウを表示します。 

1.  「**クラスター**」 組織単位の 「**プロパティ** 」 ウィンドウで、「**拡張機能**」 セクションに移動し、「**セキュリティ**」 タブを表示します。 

1.  「**セキュリティ**」 タブで 「**詳細**」 ボタンをクリックして、「**Advanced Security Settings for Clusters**」 (クラスターの詳細なセキュリティ設定) ウィンドウを開きます。 

1.  「**コンピューターの高度セキュリティ設定**」 (Advanced Security Settings for Computers) ウィンドウの 「**アクセス許可**」 タブで、「**追加**」 をクリックします。

1.  「**クラスターのアクセス許可エントリ**」 ウィンドウで、「**プリンシパルの選択**」 をクリックします

1.  「**ユーザー、サービス アカウントまたはグループの選択**」 (Select User, Service Account or Group) ダイアログ ボックスで、「**オブジェクトの種類**」 をクリックし、「**コンピューター**」 エントリの横にあるチェックボックスを有効にし、「**OK**」 をクリックします。 

1.  「**ユーザー、コンピューター、サービス アカウント、またはグループの選択**」 ダイアログ ボックスに戻り、「**選択するオブジェクト名を入力する**」 に「**az12003b-db-cl0**」と入力し、「**OK**」 をクリックします。

1.  「**クラスターのアクセス許可エントリ**」 ウィンドウで、「**タイプ**」 ドロップダウン リストに 「**許可**」 が表示されるようにします。次に、「**適用先**」 ドロップダウン リストで、「**このオブジェクトとすべての子オブジェクト**」 を選択します。「**アクセス許可**」 リストで、「**コンピューター オブジェクトの作成**」 チェック ボックスと 「**コンピューター オブジェクトの削除**」 チェック ボックスをオンにし、「**OK**」 を 2 回クリックします。

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

1.  Windows PowerShell ISE セッション内で、次のコマンドを実行して、変数 `$resourceGroupName` の値を、前のタスクでプロビジョニングしたストレージ アカウントを含むリソース グループ名に設定します。

    ```
    $resourceGroupName = 'az12003b-sap-RG'
    ```

1.  Windows PowerShell ISE セッション内で、次のコマンドを実行して、新しいクラスターの Cloud Witness Quorum を設定します。

    ```
    $cwStorageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName

    $cwStorageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $cwStorageAccountName).Value[0]

    Set-ClusterQuorum -CloudWitness -AccountName $cwStorageAccountName -AccessKey $cwStorageAccountKey
    ```

1.  結果の構成を確認するには、i20-db-0.adatum.com への RDP セッションのサーバー マネージャーで、「**ツール**」 メニューから 「**フェールオーバー クラスター マネージャー**」 を起動します。

1.  「**フェールオーバー クラスター マネージャー**」 コンソールで、**az12003b-db-cl0** クラスター構成 (ノードを含む) 、監視設定およびネットワーク設定を確認します。クラスターには共有ストレージがないことに注意してください。


### タスク 6: 可用性の高い SAP NetWeaver の ASCS 階層のインストールをサポートするために、Windows Server 2016 を実行している Azure VM のフェールオーバー クラスタリングを構成します。

> **注記**: このタスクを開始する前に、演習 1 のタスク 4 で開始した S2D クラスターのデプロイが正常に完了していることを確認します。

1.  az12003b-vm0 への RDP セッションから 、リモート デスクトップを使用して **i20-ascs-0.adatum.com** Azure VM に接続します。プロンプトが表示されたら、次の認証情報を入力します。

    -   ログイン: **ADATUM\\Student**

    -   パスワード: **Pa55w.rd1234**

1.  i20-ascs-0.adatum.com への RDP セッションのサーバー マネージャーで 「**ローカル サーバー**」 ビューに移動し、「**IE セキュリティ強化の構成**」 をオフにします。

1.  i20-ascs-0.adatum.com への RDP セッションで、サーバー マネージャーの 「**ツール**」 メニューから、**Active Directory 管理センター**を起動します。

1.  Active Directory 管理センターで、**コンピューター** コンテナーに移動します。 

1.  Active Directory Administrative Center で、i20-ascs-0 および i20-ascs-1 のコンピューター アカウントをコンピューター コンテナーからクラスター組織単位に移動します。

1.  i20-ascs-0.adatum.com への RDP セッションで、Windows PowerShell ISE セッションを開始し、次を実行して新しいクラスターを作成します。

    ```
    $nodes = @('i20-ascs-0','i20-ascs-1')

    New-Cluster -Name az12003b-ascs-cl0 -Node $nodes -NoStorage -StaticAddress 10.0.1.7
    ```

1.  i20-ascs-0.adatum.com への RDP セッションから、**Active Directory 管理センター** コンソールに切り替えます。

1.  Active Directory 管理センターで、 **クラスター**組織単位に移動し、「**プロパティ**」 ウィンドウを表示します。 

1.  「**クラスター**」 組織単位の 「**プロパティ** 」 ウィンドウで、「**拡張機能**」 セクションに移動し、「**セキュリティ**」 タブを表示します。 

1.  「**セキュリティ**」 タブで 「**詳細**」 ボタンをクリックして、「**Advanced Security Settings for Clusters**」 (クラスターの詳細なセキュリティ設定) ウィンドウを開きます。 

1.  「**コンピューターの高度セキュリティ設定**」 (Advanced Security Settings for Computers) ウィンドウの 「**アクセス許可**」 タブで、「**追加**」 をクリックします。

1.  「**クラスターのアクセス許可エントリ**」 ウィンドウで、「**プリンシパルの選択**」 をクリックします

1.  「**ユーザー、サービス アカウントまたはグループの選択**」 (Select User, Service Account or Group) ダイアログ ボックスで、「**オブジェクトの種類**」 をクリックし、「**コンピューター**」 エントリの横にあるチェックボックスを有効にし、「**OK**」 をクリックします。 

1.  「**ユーザー、コンピューター、サービス アカウント、またはグループの選択**」 ダイアログ ボックスに戻り、「**選択するオブジェクト名を入力する**」 に「**az12003b-ascs-cl0**」と入力し、「**OK**」 をクリックします。

1.  「**クラスターのアクセス許可エントリ**」 ウィンドウで、「**タイプ**」 ドロップダウン リストに 「**許可**」 が表示されるようにします。次に、「**適用先**」 ドロップダウン リストで、「**このオブジェクトとすべての子オブジェクト**」 を選択します。「**アクセス許可**」 リストで、「**コンピューター オブジェクトの作成**」 チェック ボックスと 「**コンピューター オブジェクトの削除**」 チェック ボックスをオンにし、「**OK**」 を 2 回クリックします。

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

1.  Windows PowerShell ISE セッション内で、次のコマンドを実行して、変数 `$resourceGroupName` の値を、このエクササイズでプロビジョニングしたストレージ アカウントを含むリソース グループ名に設定します。

    ```
    $resourceGroupName = 'az12003b-sap-RG'
    ```

1.  Windows PowerShell ISE セッション内で、次のコマンドを実行して、クラスターの Cloud Witness Quorum を設定します。

    ```
    $cwStorageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName

    $cwStorageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $cwStorageAccountName).Value[0]

    Set-ClusterQuorum -CloudWitness -AccountName $cwStorageAccountName -AccessKey $cwStorageAccountKey
    ```

1.  結果の構成を確認するには、i20-ascs-0.adatum.com への RDP セッションのサーバー マネージャーで、「**ツール**」 メニューから 「**フェールオーバー クラスター マネージャー**」 を起動します。

1.  **「フェールオーバー クラスター マネージャー」** コンソールで、**az12003b-ascs-cl0** クラスター構成 (ノードを含む) 、監視設定およびネットワーク設定を確認します。クラスターには共有ストレージがないことに注意してください。


### タスク 7: \\\\GLOBALHOST\\sapmnt 共有にアクセス許可を設定する

このタスクでは、**\\\\GLOBALHOST\\sapmnt** 共有に共有レベルのアクセス許可を設定します。

> **注記**: 既定では、フル コントロールのアクセス許可は ADATUM\Student アカウントにのみ付与されます。 

1.  i20-ascs-0.adatum.com へのリモート デスクトップ セッションで、**Windows PowerShell ISE** ウィンドウから、次を実行します。

    ```
    $remoteSession = New-CimSession -ComputerName SAPGLOBALHOST

    Grant-SmbShareAccess -Name sapmnt -AccountName 'ADATUM\Domain Admins' -AccessRight Full -CimSession $remoteSession -Force
   
    ```

### タスク 8: SAP NetWeaver ASCS およびデータベース コンポーネントをインストールするためのオペレーティング システムの前提条件を構成する

1.  i20-ascs-0.adatum.com へのリモート デスクトップ セッションで、Windows PowerShell ISE セッションから次を実行して、SAP ASCS コンポーネントのインストールと仮想名の使用を調整するのに必要なレジストリ エントリを構成します。

    ```
    $nodes = ('i20-db-0','i20-db-1')

    Invoke-Command $nodes {
        $registryPath = 'HKLM:\SYSTEM\CurrentControlSet\Services\lanmanworkstation\parameters'
        $registryEntry = 'DisableCARetryOnInitialConnect'
        $registryValue = 1
        New-ItemProperty -Path $registryPath -Name $registryEntry -Value $registryValue -PropertyType DWORD -Force
    }

    Invoke-Command $nodes {
        $registryPath = 'HKLM:\SYSTEM\CurrentControlSet\Control\LSA'
        $registryEntry = 'DisableLoopbackCheck'
        $registryValue = 1
        New-ItemProperty -Path $registryPath -Name $registryEntry -Value $registryValue -PropertyType DWORD -Force
    }

    Invoke-Command $nodes {
        $registryPath = 'HKLM:\SYSTEM\CurrentControlSet\Services\lanmanserver\parameters'
        $registryEntry = 'DisableStrictNameChecking'
        $registryValue = 1
        New-ItemProperty -Path $registryPath -Name $registryEntry -Value $registryValue -PropertyType DWORD -Force
    }
    ```

> **結果**: このエクササイズを完了すると、可用性の高い SAP NetWeaver デプロイをサポートするように、Windows を実行している Azure VM のオペレーティング システムを構成することができます。


## 演習 3: ラボリソースを削除する

時間: 10 分

このエクササイズでは、このラボでプロビジョニングしたリソースすべてを削除します。

#### タスク 1: Cloud Shell を開く

1. ポータルの上部にある **「Cloud Shell」** アイコンをクリックして Cloud Shell ペインを開き、シェルとして PowerShell を選択します。

1. Cloud Shell ペインで、次のコマンドを実行して、変数 `$resourceGroupName` の値を、このラボの最初のエクササイズでプロビジョニングした **Windows Server 2019 Datacenter** Azure VM のペアを含むリソース グループ名に設定します。

    ```
    $resourceGroupNamePrefix = 'az12003b-'
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

1. 「Cloud Shell」ペインを閉じます。

> **結果**: このエクササイズを完了すると、このラボで使用したリソースが削除されます。
