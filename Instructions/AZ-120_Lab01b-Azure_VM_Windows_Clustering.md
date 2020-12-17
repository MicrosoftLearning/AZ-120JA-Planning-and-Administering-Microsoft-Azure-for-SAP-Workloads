# AZ 120 モジュール 1: Azure での SAP 認定オファリング
# ラボ 1b: Azure VM での Windows クラスタリングの実装

推定時間: 120 分間

このラボのタスクすべては、Azure portal (PowerShell Cloud Shell セッションを含む) から実行されます  

   > **注記**: Cloud Shell を使用しない場合、ラボの仮想マシンには Az PowerShell モジュールがインストールされている必要があります [**https://docs.microsoft.com/ja-jp/powershell/azure/install-az-ps-msi?view=azps-2.8.0**](https://docs.microsoft.com/ja-jp/powershell/azure/install-az-ps-msi?view=azps-2.8.0)。

ラボ ファイル: なし

## シナリオ
  
Azure に SAP NetWeaver を導入し、SQL サーバーをデータベース管理システムとして使用する準備として、Adatum Corporation は、Windows Server 2019 を実行する Azure VM にクラスタリングを実装するプロセスを検討したいと考えています。

## 目的
  
このラボを終了すると、下記ができるようになります:

-   可用性の高い SAP NetWeaver のデプロイをサポートするために必要となる Azure コンピューティング リソースのプロビジョニングを行う。

-   可用性の高い SAP NetWeaver のデプロイをサポートするために、Windows Server 2019 を実行している Azure VM のオペレーティング システムを構成する。

-   可用性の高い SAP NetWeaver のデプロイをサポートするために必要となる Azure ネットワーク リソースのプロビジョニングを行う。

## 必要条件

-   使用可能な DSv3 vCPU (4 x 4) と DSv2 (1 x 1) vCPU の十分な数を持つ Microsoft Azure サブスクリプション

-   Azure にアクセス可能な Windows 10、Windows Server 2019、または Windows Server 2016 を実行しているラボ コンピューター

## エクササイズ 1: 可用性の高い SAP NetWeaver のデプロイをサポートするために必要となる Azure コンピューティング リソースのプロビジョニングを行う

時間: 50 分間

このエクササイズでは、Windows Server 2019 を実行する Azure VM でのフェールオーバー クラスタリングの構成に必要な Azure インフラストラクチャ コンピューティング コンポーネントをデプロイします。これには、Active Directory ドメイン コントローラーのペアをデプロイし、同じ仮想ネットワークの同じ可用性セットで Windows Server 2019 を実行する Azure VM のペアをデプロイすることが含まれます。ドメイン コントローラーのデプロイを自動化するには、<https://github.com/Azure/azure-quickstart-templates/tree/master/active-directory-new-domain-ha-2-dc>から利用可能な Azure Resource Manager クイックスタート テンプレートを使用します。

### タスク 1: Azure Resource Manager テンプレートを使用して、可用性の高い Active Directory ドメイン コントローラーを実行する Azure VM のペアをデプロイする

1.  ラボ コンピューターから Web ブラウザーを起動し、https://portal.azure.com から Azure portal に移動します。

1.  プロンプトが表示されたら、このラボで使用する Azure サブスクリプションの所有者または共同作成者ロールを使用して、職場用、学校用、または個人の Microsoft アカウントでサインインします。

1.  Azure portal で、「**+ リソースの作成**」 をクリックします。

1.  「**新規**」 ブレードから、新しい **Template deployment (カスタム テンプレートを使用してデプロイ)** の作成を開始します

1.  「**カスタム デプロイ**」 ブレードの 「**GitHub クイックスタート テンプレートを読み込む**」 ドロップダウン リストから、エントリ **active-directory-new-domain-ha-2-dc** を選択し、「**テンプレートの選択**」 をクリックします。

    > **注記**: または、Azure クイックスタート テンプレート ページ <https://github.com/Azure/azure-quickstart-templates> に移動して、**Create 2 new Windows VMs, create a new AD Forest, Domain, and 2 DCs in an availability set (可用性セットに 2 つの新規 Windows VM, 新しい AD Forest、ドメイン、2 つの DC を作成する)** という名前のテンプレートを検索し、「**Azure に配置する**」 ボタンをクリックしてデプロイを開始します。   

1.  「**2 ドメイン コントローラーを持つ新しい AD ドメインの作成**」 ブレードで、「**テンプレートの編集**」 をクリックします。

1.  「**テンプレートの編集**」 ブレードで、**adVMSize** 変数に値を割り当てる行を探します。

    ```
    "adVMSize": "Standard_DS2_v2"

    ```

1.  「**テンプレートの編集**」 ブレードで、**adVMSize** 変数の値を **Standard_D4S_v3** に設定し、「**保存**」 をクリックします。

    ```
    "adVMSize": "Standard_D4s_v3"

    ```

1.  「**2 つのドメインコントローラーを使用して新しい AD ドメインを作成する**」 ブレードに戻り、次の設定を指定し、「**購入**」 をクリックしてデプロイを開始します。

    -   サブスクリプション: Azure サブスクリプションの名前。

    -   リソース グループ:  **az12001b-ad-RG** *という名前の新規リソース グループ*

    -   場所: *Azure VM をデプロイできて、ラボの場所に最も近い Azure リージョン*

    -   管理者ユーザー名: **Student**

    -   パスワード: **Pa55w.rd1234**

    -   ドメイン名: **adatum.com**

    -   DnsPrefix: *一意の有効なDNS プレフィックス*

    -   Pdc RDP ポート: **3389**

    -   Bdc RDP ポート: **13389**

    -   _artifacts の保存先: **https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/active-directory-new-domain-ha-2-dc/**

    -   _artifacts の保存先の Sas トークン: *空白のままにする*

    -   保存先: **[resourceGroup().location]**

    -   上記の使用条件に同意する: *有効*

    > **注記**: デプロイには約 35 分間かかります。デプロイが完了するのを待ってから、次のタスクに進みます。

### タスク 2: 同じ可用性セットで Windows Server 2016 を実行している Azure VM のペアをデプロイする。

1.  ラボ コンピューターの Azure portal で、「**+ リソースの作成**」 をクリックします。

1.  「**新規**」 ブレードで、次の設定を使用して **Windows Server 2019 Datacenter** Azure VM のプロビジョニングを開始します。

    -   サブスクリプション: Azure サブスクリプションの名前。

    -   リソース グループ:  **az12001b-cl-RG** *という名前の新規リソース グループ*

    -   仮想マシン名 = **az12001b-cl-vm0**

    -   リージョン: *前のタスクで Azure VM をデプロイしたのと同じ Azure リージョン*

    -   可用性オプション: **可用性セット**

    -   可用性セット: *2 つの障害ドメインと 5 つの更新ドメインを持つ* **az12001b-cl-avset** *という名前の新規可用性セット*

    -   画像: **Windows Server 2019 Datacenter**

    -   サイズ: **Standard D4s v3**

    -   ユーザー名: **Student**

    -   パスワード: **Pa55w.rd1234**

    -   パブリック受信ポート: **選択したポートを許可する**

    -   受信ポートを選択: **RDP (3389)**

    -   Windows ライセンスを既にお持ちの場合: **いいえ**

    -   OS ディスク タイプ: **Premium SSD**

    -   仮想ネットワーク: **adVNET**

    -   サブネット名: **clSubnet** *という名前の新しいサブネット* 

    -   サブネットアドレス範囲: **10.0.1.0/24**

    -   パブリック IP アドレス: **az12001b-cl-vm0-ip** *という名前の新規 IP アドレス*

    -   NIC ネットワーク セキュリティ グループ: **Basic**

    -   パブリック受信ポート: **選択したポートを許可する**

    -   受信ポートを選択: **RDP (3389)**

    -   高速ネットワーク: **オン**

    -   この仮想マシンを既存の負荷分散ソリューションの後ろに配置しますか: **いいえ**

    -   無料の Basic プランを有効にする: **いいえ**

    -   ブート診断: **オフ**

    -   OS のゲスト診断: **オフ**

    -   システム割り当てマネージド ID **オフ**

    -   AAD 資格情報を使用してログインする (プレビュー): **オフ**

    -   自動シャットダウンを有効にする: **オフ**

    -   バックアップを有効にする: **オフ**

    -   拡張機能: *なし*

    -   タグ: *なし*

1.  プロビジョニングが完了するのを待たずに、次の手順に進みます。

1.  次の設定を使用して、別の **Windows Server 2019 Datacenter** Azure VM をプロビジョニングします。

    -   サブスクリプション: Azure サブスクリプションの名前。

    -   リソース グループ：**az12001b-cl-RG**

    -   仮想マシン名 = **az12001b-cl-vm1**

    -   リージョン: *前のタスクで Azure VM をデプロイしたのと同じ Azure リージョン*

    -   可用性オプション: **可用性セット**

    -   可用性セット: **az12001b-cl-avset**

    -   画像: **Windows Server 2019 Datacenter**

    -   サイズ: **Standard D4s v3**

    -   ユーザー名: **Student**

    -   パスワード: **Pa55w.rd1234**

    -   パブリック受信ポート: **選択したポートを許可する**

    -   受信ポートを選択: **RDP (3389)**

    -   Windows ライセンスを既にお持ちの場合: **いいえ**

    -   OS ディスク タイプ: **Premium SSD**

    -   仮想ネットワーク: **adVNET**

    -   サブネット名: **clSubnet**

    -   パブリック IP アドレス: **az12001b-cl-vm0-ip** *という名前の新規 IP アドレス*

    -   NIC ネットワーク セキュリティ グループ: **Basic**

    -   パブリック受信ポート: **選択したポートを許可する**

    -   受信ポートを選択: **RDP (3389)**

    -   高速ネットワーク: **オン**

    -   この仮想マシンを既存の負荷分散ソリューションの後ろに配置しますか: **いいえ**

    -   無料の Basic プランを有効にする: **いいえ**

    -   ブート診断: **オフ**

    -   OS のゲスト診断: **オフ**

    -   システム割り当てマネージド ID**オフ**

    -   AAD 資格情報を使用してログインする (プレビュー): **オフ**

    -   自動シャットダウンを有効にする: **オフ**

    -   バックアップを有効にする: **オフ**

    -   拡張機能: *なし*

    -   タグ: *なし*

1.  プロビジョニングが完了するまで待ちます。通常、これには数分かかります。

### タスク 3: Azure VM ディスクを作成して構成する

1.  Azure portal の Cloud Shell で PowerShell セッションを開始します。 

    > **注記**: 現在の Azure サブスクリプションで Cloud Shell を初めて起動する場合は、Azure ファイル共有を作成して Cloud Shell ファイルを保持するように求められます。その場合、既定値を受け入れると、自動的に生成されたリソース グループにストレージ アカウントが作成されます。

1.  「Cloud Shell」 ウィンドウで、次のコマンドを実行して、前のタスクでデプロイした最初の Azure VM に接続する 4 個のマネージド ディスクの最初のセットを作成します。

    ```
    $resourceGroupName = 'az12001b-cl-RG'

    $location = (Get-AzResourceGroup -Name $resourceGroupName).Location

    $diskConfig = New-AzDiskConfig -Location $location -DiskSizeGB 128 -AccountType Premium_LRS -OsType Windows -CreateOption Empty

    for ($i=0;$i -lt 4;$i++) {New-AzDisk -ResourceGroupName $resourceGroupName -DiskName az12001b-cl-vm0-DataDisk$i -Disk $diskConfig}
    ```

1.  「Cloud Shell」 ウィンドウで、次のコマンドを実行して、前のタスクでデプロイした 2 番目の Azure VM に接続する 4 個のマネージド ディスクの 2 番目のセットを作成します。

    ```
    for ($i=0;$i -lt 4;$i++) {New-AzDisk -ResourceGroupName $resourceGroupName -DiskName az12001b-cl-vm1-DataDisk$i -Disk $diskConfig}
    ```

1.  Azure portal で、前のタスク (**az12001b-cl-vm0**) でプロビジョニングした最初の Azure VM のブレードに移動します。

1.  **az12001b-cl-vm0** ブレードから、**az12001b-cl-vm0 - ディスク** ブレードに移動します。

1.  **az12001b-cl-vm0 - ディスク** ブレードから、az12001b-cl-vm0 に次の設定を持つデータ ディスクを接続します。

    -   LUN: **0**

    -   ディスク名: **az12001b-cl-vm0-DataDisk0**

    -   リソース グループ：**az12001b-cl-RG**

    -   ホスト キャッシュ: **読み取り専用**

1.  前の手順を繰り返して、プレフィックス **az12001b-cl-vm0-DataDisk** を使用して残りの 3 つのディスク (合計 4 つ) を接続します。ディスク名の最後の文字と一致する LUN 番号を割り当てます。最後のディスク (LUN **4**) には、ホスト キャッシュを **なし** に設定します。  

1.  変更内容を保存します｡ 

1.  Azure portal で、前のタスク (**az12001b-cl-vm1**) でプロビジョニングした 2 番目の Azure VM のブレードに移動します。

1.  **az12001b-cl-vm1** ブレードから、**az12001b-cl-vm1 - ディスク** ブレードに移動します。

1.  **az12001b-cl-vm1 - ディスク** ブレードから、az12001b-cl-vm1 に次の設定を持つデータ ディスクを接続します。

    -   LUN: **0**

    -   ディスク名: **az12001b-cl-vm1-DataDisk0**

    -   リソース グループ：**az12001b-cl-RG**

    -   ホスト キャッシュ: **読み取り専用**

1.  前の手順を繰り返して、プレフィックス **az12001b-cl-vm1-DataDisk** を使用して残りの 3 つのディスク (合計 4 つ) を接続します。ディスク名の最後の文字と一致する LUN 番号を割り当てます。最後のディスク (LUN **4**) には、ホスト キャッシュを **なし** に設定します。

1.  変更内容を保存します｡ 

> **結果**: このエクササイズを完了すると、可用性の高い SAP NetWeaver のデプロイをサポートするために必要となる Azure コンピューティング リソースのプロビジョニングを行うことができます。


## エクササイズ 1: 可用性の高い SAP NetWeaver のインストールをサポートするために、Windows Server 2019 を実行している Azure VM のオペレーティング システムを構成する。

時間: 40 分間

### タスク 1: Windows Server 2019 Azure VM を Active Directory ドメインに結合させる。

   > **注記**: このタスクを開始する前に、前のエクササイズの最後のタスクで開始したテンプレートのデプロイが正常に完了したことを確認します。 

1.  Azure Portal で、このラボの最初のエクササイズで自動的にプロビジョニングされた仮想ネットワーク **adVNET** のブレードに移動します。

1.  **adVNET - DNS サーバー** ブレードを表示し、仮想ネットワークでは、このラボの最初のエクササイズでデプロイしたドメイン コントローラーに割り当てられたプライベート IP アドレスを DNS サーバーとして構成していることを確認します。 

1.  Azure portal の Cloud Shell で PowerShell セッションを開始します。 

1.  「Cloud Shell」 ウィンドウで次のコマンドを実行して、前のエクササイズの 2 番目のタスクでデプロイした Windows Server 2019 Azure VM を **adatum.com** Active Directory ドメインに結合します。 

    ```
    $resourceGroupName = 'az12001b-cl-RG'

    $location = (Get-AzureRmResourceGroup -Name $resourceGroupName).Location

    $settingString = '{"Name": "adatum.com", "User": "adatum.com\\Student", "Restart": "true", "Options": "3"}'

    $protectedSettingString = '{"Password": "Pa55w.rd1234"}'

    $vmNames = @('az12001b-cl-vm0','az12001b-cl-vm1')

    foreach ($vmName in $vmNames) { Set-AzVMExtension -ResourceGroupName $resourceGroupName -ExtensionType 'JsonADDomainExtension' -Name 'joindomain' -Publisher "Microsoft.Compute" -TypeHandlerVersion "1.0" -Vmname $vmName -Location $location -SettingString $settingString -ProtectedSettingString $protectedSettingString }
    ```

1.  次のタスクを進める前に、スクリプトが完了するのを待ちます。


### タスク 2: 可用性の高い SAP NetWeaver のインストールをサポートするために、Windows Server 2019 を実行している Azure VM のストレージを構成する。

1.  Azure Portal で、このラボの最初のエクササイズでプロビジョニングした仮想仮想マシン **az12001b-cl-vm0** のブレードに移動します。

1.  **az12001b-cl-vm0** ブレードから、リモート デスクトップを使用して仮想マシン ゲスト オペレーティング システムに接続します。  認証を求めるメッセージが表示されたら、次の認証情報を入力します。

    -   ユーザー名: **ADATUM\Student**

    -   パスワード: **Pa55w.rd1234**

1.  az12001b-cl-vm0 への RDP セッションのサーバー マネージャーで 「**ローカル サーバー**」 ビューに移動し、「**IE セキュリティ強化の構成**」 を一時的にオフにします。 

1.  az12001b-cl-vm0 への RDP セッションのサーバー マネージャーで、「**ファイル サービスと記憶域サービス**」 -> 「**サーバー**」  ノードの順に移動します。    

1.  「**記憶域プール**」 ビューに移動し、前のエクササイズで Azure VM に接続したディスクすべてが表示されていることを確認します。 

1.  **新規記憶域プール ウィザード** を使用して、次の設定で新しい記憶域プールを作成します。

    -   名前: **データ記憶域プール**

    -   物理ディスク: *LUN 番号 (0-2) の最初の 3 つに対応するディスク番号を持つ 3 つのディスクを選択し、割り当てを* 「**自動**」 に設定します。

    > **注記**:  **シャーシ** 列のエントリを使用して、 **LUN** 番号を特定します。   

1.  **新規仮想ディスク ウィザード** を使用して、次の設定で新しい仮想ディスクを作成します。

    -   仮想ディスク名: **Data Virtual Disk**

    -   ストレージ レイアウト: **シンプル**

    -   プロビジョニング: **固定**

    -   サイズ: **最大サイズ**

1.  **新規ボリューム ウィザード** を使用して、次の設定で新しいボリュームを作成します。

    -   サーバーとディスク: *既定値を使用する*

    -   サイズ: *規定値を使用する*

    -   ドライブ文字: **M**

    -   ファイル システム: **ReFS**

    -   アロケーション ユニット サイズ: **既定値**

    -   ボリューム ラベル: **データ**

1.  「**記憶域プール**」 ビューに戻り、**新規記憶域プール ウィザード** を使用して、次の設定で新しい記憶域プールを作成します。

    -   名前: **Log Storage Pool**

    -   物理ディスク: *最後の 4 つのディスクを選択し、その割り当てを* 「**自動**」 に設定します。

1.  **新規仮想ディスク ウィザード** を使用して、次の設定で新しい仮想ディスクを作成します。

    -   仮想ディスク名: **Data Virtual Disk**

    -   ストレージ レイアウト: **シンプル**

    -   プロビジョニング: **固定**

    -   サイズ: **最大サイズ**

1.  **新規ボリューム ウィザード** を使用して、次の設定で新しいボリュームを作成します。

    -   サーバーとディスク: *既定値を使用する*

    -   サイズ: *規定値を使用する*

    -   ドライブ文字: **L**

    -   ファイル システム: **ReFS**

    -   アロケーション ユニット サイズ: **既定値**

    -   ボリューム ラベル: **ログ**

1.  az12001b-cl-vm1 でストレージを構成するには、前の手順を繰り返します。

### タスク 3: 可用性の高い SAP NetWeaver のインストールをサポートするために、Windows Server 2019 を実行している Azure VM のフェールオーバー クラスタリングの構成を準備する。

1.  az12001b-cl-vm0 への RDP セッションで、Windows PowerShell ISE セッションを開始し、az12001b-cl-vm0 と az12001b-cl-vm1 の両方にフェールオーバー クラスタリングおよびリモート管理ツール機能をインストールします。

    ```
    $nodes = @('az12001b-cl-vm1', 'az12001b-cl-vm0')

    Invoke-Command $nodes {Install-WindowsFeature Failover-Clustering -IncludeAllSubFeature -IncludeManagementTools} 

    Invoke-Command $nodes {Install-WindowsFeature RSAT -IncludeAllSubFeature -Restart} 
    ```

    > **注記**: これにより、両方の Azure VM のゲスト オペレーティング システムが再起動されます。

1.  ラボ コンピューターで Azure portal を開き、「**+ リソースを作成**」 をクリックします。

1.  「**新規**」 ブレードで、次の設定を使用して新しいストレージ アカウントの作成を開始します。

    -   サブスクリプション: Azure サブスクリプションの名前。

    -   リソース グループ：**az12001b-cl-RG**

    -   ストレージ アカウント名: *3 から24 の文字と 数字で構成される一意の名前*

    -   保存先: *前のエクササイズで Azure VM をデプロイしたのと同じ Azure リージョン*

    -   パフォーマンス: **Standard**

    -   アカウントの種類: **ストレージ (汎用 v1)**

    -   レプリケーション: **ローカル冗長ストレージ (LRS)**

    -   接続方法: **パブリック エンドポイント (すべてのネットワーク)**

    -   安全な転送が必須: **有効**

    -   大きなファイル共有: **無効**

    -   BLOB の論理的な削除: **無効**

    -   階層型名前空間: **無効**

### タスク 4: 可用性の高い SAP NetWeaver のインストールをサポートするために、Windows Server 2019 を実行している Azure VM のフェールオーバー クラスタリングを構成する。

1.  Azure Portal で、このラボの最初のエクササイズでプロビジョニングした仮想仮想マシン **az12001b-cl-vm0** のブレードに移動します。

1.  **az12001b-cl-vm0** ブレードから、リモート デスクトップを使用して仮想マシン ゲスト オペレーティング システムに接続します。認証を求めるメッセージが表示されたら、次の認証情報を入力します。

    -   ユーザー名: **ADATUM\\Student**

    -   パスワード: **Pa55w.rd1234**

1.  az12001b-cl-vm0 への RDP セッションで、サーバー マネージャーの 「**ツール**」 メニューから、**Active Directory 管理センター** を起動します。   

1.  Active Directory 管理センター で、adatum.com ドメインのルートに **クラスター** という名前の新しい組織単位を作成します。

1.  Active Directory Administrative Center で、**az12001b-cl-vm0** および **az12001b-cl-vm1** のコンピューター アカウントを **コンピューター** コンテナーから **クラスター** 組織単位に移動します。

1.  az12001b-cl-vm0 への RDP セッションで、Windows PowerShell ISE セッションを開始し、次を実行して新しいクラスターを作成します。

    ```
    $nodes = @('az12001b-cl-vm0','az12001b-cl-vm1')

    New-Cluster -Name az12001b-cl-cl0 -Node $nodes -NoStorage -StaticAddress 10.0.1.6
    ```

1.  az12001b-cl-vm0 への RDP セッションから 、**Active Directory 管理センター コンソール** に切り替えます。 

1.  Active Directory 管理センターで、 **クラスター** 組織単位に移動し、「**プロパティ**」 ウィンドウを表示します。    

1.  「**クラスター**」 組織単位の 「**プロパティ**」 ウィンドウで、「**拡張機能**」 セクションに移動し、「**セキュリティ**」 タブを表示します。        

1.  「**セキュリティ**」 タブで 「**詳細**」 ボタンをクリックして、「**Advanced Security Settings for Clusters**」 (クラスターの詳細なセキュリティ設定) ウィンドウを開きます。      

1.  「**コンピューターの高度セキュリティ設定**」 (Advanced Security Settings for Computers) ウィンドウの 「**アクセス許可**」 タブで、「**追加**」 をクリックします。

1.  「**Permission Entry for Clusters**」 (クラスターのアクセス許可エントリ) ウィンドウで、「**プリンシパルの選択**」 をクリックします。

1.  「**ユーザー、サービス アカウントまたはグループの選択**」 (Select User, Service Account or Group) ダイアログ ボックスで、「**オブジェクトの種類**」 をクリックし、「**コンピューター**」 エントリの横にあるチェックボックスを有効にし、「**OK**」 をクリックします。 

1.  「**ユーザー、コンピューター、サービス アカウント、またはグループの選択**」 ダイアログ ボックスに戻り、「**選択するオブジェクト名を入力してください**」 に「**az12001b-cl-cl0**」と入力し、「**OK**」 をクリックします。       

1.  「**クラスターのアクセス許可エントリ**」 ウィンドウで、「**タイプ**」 ドロップダウン リストに 「**許可**」 が表示されるようにします。    次に、「**適用先**」 ドロップダウン リストで、「**このオブジェクトとすべての子オブジェクト**」 を選択します。    「**アクセス許可**」 リストで、「**コンピューター オブジェクトの作成**」 チェック ボックスと 「**コンピューター オブジェクトの削除**」 チェック ボックスをオンにし、「**OK**」 を 2 回クリックします。

1.  Windows PowerShell ISE セッションで、次を実行して Az PowerShell モジュールをインストールします。

    ```
    Install-PackageProvider -Name NuGet -Force

    Install-Module -Name Az -Force
    ```

1.  Windows PowerShell ISE セッションで、次のを実行して Azure AD 認証情報を使用して認証します。

    ```
    Add-AzAccount
    ```

    > **注記**: プロンプトが表示されたら、このラボで使用する Azure サブスクリプションの所有者または共同作成者ロールを使用して、職場用、学校用、または個人の Microsoft アカウントでサインインします。

1.  Windows PowerShell ISE セッションで次を実行して、新しいクラスターの Cloud Witness Quorum を設定します。

    ```
    $resourceGroupName = 'az12001b-cl-RG'

    $cwStorageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName

    $cwStorageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $cwStorageAccountName).Value[0]

    Set-ClusterQuorum -CloudWitness -AccountName $cwStorageAccountName -AccessKey $cwStorageAccountKey
    ```

1.  結果の構成を確認するには、az12001b-cl-vm0 への RDP セッションのサーバー マネージャーで、「**ツール**」 メニューから 「**フェールオーバー クラスター マネージャー**」 を起動します。   

1.  「**フェールオーバー クラスター マネージャー**」 コンソールで、**az12001b-cl-cl0** クラスター構成 (ノードを含む) 、監視設定およびネットワーク設定を確認します。    クラスターには共有ストレージがないことに注意してください。

1.  az12001b-cl-vm0 への RDP セッションを 終了します。

> **結果**: このエクササイズを完了すると、可用性の高い SAP NetWeaver インストールをサポートするように、Windows Server 2019 を実行している Azure VM のオペレーティング システムを構成することができます。


## エクササイズ 3: 可用性の高い SAP NetWeaver のデプロイをサポートするために必要となる Azure ネットワーク リソースのプロビジョニングを行う

時間: 30 分間

このエクササイズでは、SAP NetWeaver のクラスター化されたインストールに対応するために Azure Load Balancers を実装します。

### タスク 1: 負荷分散のセットアップが容易に行えるよう Azure VM を構成する。

   > **注記**: Stardard SKU の Azure Load Balancer のペアを設定するので、まず、負荷分散バックエンド プールとして機能する 2 つの Azure VM のネットワーク アダプターに関連付けられたパブリック IP アドレスを削除する必要があります。

1.  ラボ コンピューターの Azure portal で、Azure VM **az12001b-cl-vm0** のブレードに移動します。  

1.  **az12001b-cl-vm0** ブレードから、ネットワーク アダプタに関連付けられたパブリック IP アドレス **az12001b-cl-vm0-ip** のブレードに移動します。

1.  **az12001b-cl-vm0-ip** ブレードで、まずパブリック IP アドレスをネットワーク インターフェイスから切り離してから削除します。

1.  Azure portal で、Azure VM **az12001b-cl-vm1** のブレードに移動します。  

1.  **az12001b-cl-vm1** ブレードから、ネットワーク アダプタに関連付けられたパブリック IP アドレス **az12001b-cl-vm1-ip** のブレードに移動します。

1.  **az12001b-cl-vm1-ip** ブレードで、まずパブリック IP アドレスをネットワーク インターフェイスから切り離してから削除します。

1.  Azure portal で、**az12001a-vm0** Azure VM のブレードに移動します。

1.  **az12001a-vm0** ブレードから、**ネットワーク** ブレードに移動します。 

1.  **az12001a-vm0 - ネットワーク** ブレードから、az12001a-vm0 のネットワーク インターフェイスに移動します。 

1.  az12001a-vm0 のネットワークインターフェースのブレードから、その IP 構成ブレードに移動し、そこから **ipconfig1** ブレードを表示します。

1.  **ipconfig1** ブレードで、プライベート IP アドレスの割り当てを **Static** に設定し、変更を保存します。

1.  Azure portal で、**az12001a-vm1** Azure VM のブレードに移動します。

1.  **az12001a-vm1** ブレードから、**ネットワーク** ブレードに移動します。 

1.  **az12001a-vm1 - ネットワーク** ブレードから、az12001a-vm1 のネットワーク インターフェイスに移動します。

1.  az12001a-vm1 のネットワーク インターフェースのブレードから、その IP 構成ブレードに移動し、そこから **ipconfig1** ブレードを表示します。

1.  **ipconfig1** ブレードで、プライベート IP アドレスの割り当てを **Static** に設定し、変更を保存します。

### タスク 2: 受信トラフィックを処理する Azure Load Balancers を作成して構成する

1.  Azure portal で、「**+ リソースの作成**」 をクリックします。

1.  「**新規**」 ブレードで、次の設定を使用して新しい Azure Load Balancer の作成を開始します。

    -   サブスクリプション: Azure サブスクリプションの名前。

    -   リソース グループ：**az12001b-cl-RG**

    -   名前: **az12001b-cl-lb0**

    -   リージョン: *このラボの最初のエクササイズで Azure VM をデプロイしたのと同じ Azure リージョン*

    -   タイプ: **内部**

    -   SKU: **Standard**

    -   仮想ネットワーク: **adVNET**

    -   サブネット: **clSubnet**

    -   IP アドレスの割り当て: **静的**

    -   IP アドレス: **10.0.1.240**

    -   可用性ゾーン: **ゾーン冗長**

1.  Load Balancer のプロビジョニングが終了するまで待ち、Azure portal のブレードに移動します。

1.  **az12001b-cl-lb0** ブレードで、次の設定を使用してバックエンド プールを追加します。

    -   名前: **az12001b-cl-lb0-bepool**

    -   仮想ネットワーク: **adVNET**

    -   仮想マシン: **az12001b-cl-vm0**  IP アドレス: **ipconfig1**

    -   仮想マシン: **az12001b-cl-vm1**  IP アドレス: **ipconfig1**

1.  **az12001b-cl-lb0** ブレードから、次の設定を使用して正常性プローブを追加します。

    -   名前: **az12001b-cl-lb0-hprobe**

    -   プロトコル: **TCP**

    -   ポート: **59999**

    -   間隔: **5** *秒*

    -   異常しきい値: **2** *つの連続エラー*

1.  **az12001b-cl-lb0** ブレードから、次の設定を使用してネットワーク負荷分散ルールを追加します。

    -   名前: **az12001b-cl-lb0-lbruletcp1433**

    -   IP バージョン: **IPv4**

    -   フロントエンド IP アドレス: **192.168.0.240 (LoadBalancerFrontEnd)**

    -   HA ポート: **無効**

    -   プロトコル: **TCP**

    -   ポート: **1433**

    -   バックエンド ポート: **1433**

    -   バックエンド プール: **az12001b-cl-lb0-bepool (2 台の仮想マシン)**

    -   正常性プローブ: **az12001b-cl-lb0-hprobe (TCP:59999)**

    -   セッション永続化: **なし**

    -   アイドル タイムアウト (分): **4**

    -   フローティング IP (ダイレクト サーバー リターン): **有効**

### タスク 3: 送信トラフィックを処理する Azure Load Balancers を作成して構成する

1.  Azure portal から、Cloud Shell で PowerShell セッションを開始します。 

1.  「Cloud Shell」 ウィンドウで次のコマンドを実行して、2 番目の Load Balancer で使用するパブリック IP アドレスを作成します。

    ```
    $resourceGroupName = 'az12001b-cl-RG'

    $location = (Get-AzResourceGroup -Name $resourceGroupName).Location

    $pipName = 'az12001b-cl-lb0-pip'

    az network public-ip create --resource-group $resourceGroupName --name $pipName --sku Standard --location $location
    ```

1.  「Cloud Shell」 ウィンドウで次のコマンドを実行して、2 番目の Load Balancer を作成します。

    ```
    $lbName = 'az12001b-cl-lb1'

    $lbFeName = 'az12001b-cl-lb1-fe'

    $lbBePoolName = 'az12001b-cl-lb1-bepool'
   
    $pip = Get-AzPublicIpAddress -ResourceGroupName $resourceGroupName -Name $pipName

    $feIpconfiguration = New-AzLoadBalancerFrontendIpConfig -Name $lbFeName -PublicIpAddress $pip

    $bePoolConfiguration = New-AzLoadBalancerBackendAddressPoolConfig -Name $lbBePoolName

    New-AzLoadBalancer -ResourceGroupName $resourceGroupName -Location $location -Name $lbName -Sku Standard -BackendAddressPool $bePoolConfiguration -FrontendIpConfiguration $feIpconfiguration
    ```

1.  「Cloud Shell」 ウィンドウを閉じます。

1.  Azure portal で、**az12001b-cl-lb1** という名前の Azure Load Balancer のプロパティを表示するブレードに移動します。

1.  **az12001b-cl-lb1** ブレードで、「**バックエンド プール**」 をクリックします。

1.  **az12001b-cl-lb1 - バックエンド プール** ブレードで、**az12001b-cl-lb1-bepool** をクリックします。

1.  **az12001b-cl-lb1-bepool** ブレードで、次の設定を指定し、「**保存**」 をクリックします。

    -   仮想ネットワーク: **adVNET (4 VM)**

    -   仮想マシン: **az12001b-cl-vm0**  IP アドレス: **ipconfig1**

    -   仮想マシン: **az12001b-cl-vm1**  IP アドレス: **ipconfig1**

1.  **az12001b-cl-lb1** ブレードで、「**正常性プローブ**」 をクリックします。

1.  **az12001b-cl-lb1 - 正常性プローブ** ブレードから、次の設定を使用して正常性プローブを追加します。

    -   名前: **az12001b-cl-lb1-hprobe**

    -   プロトコル: **TCP**

    -   ポート: **80**

    -   間隔: **5** *秒*

    -   異常しきい値: **2** *つの連続エラー*

1.  **az12001b-cl-lb1** ブレードで、「**負荷分散ルール**」 をクリックします。

1.  **az12001b-cl-lb1 - 負荷分散ルール** ブレードから、次の設定を使用してネットワーク負荷分散ルールを追加します。

    -   名前: **az12001b-cl-lb1-lbharule**

    -   IP バージョン: **IPv4**

    -   フロントエンド IP アドレス: *既定値を使用する*

    -   プロトコル: **TCP**

    -   ポート: **80**

    -   バックエンド ポート: **80**

    -   バックエンド プール: **az12001b-cl-lb1-bepool (2 台の仮想マシン)**

    -   正常性プローブ: **az12001b-cl-lb1-hprobe (TCP:80)**

    -   セッション永続化: **なし**

    -   アイドル タイムアウト (分): **4**

    -   フローティング IP (ダイレクト サーバー リターン): **無効**

### タスク 4: ジャンプ ホストをデプロイする

   > **注記**: 2 つのクラスター化された Azure VM にはインターネットから直接アクセスできないため、ジャンプ ホストとして機能する Windows Server 2019 Datacenter を実行する Azure VM をデプロイします。 

1.  ラボ コンピューターの Azure portal で、「**+ リソースの作成**」 をクリックします。

1.  「**新規**」 ブレードから、**Windows Server 2019 Datacenter** イメージに基づいて新規 Azure VM の作成を開始します。

1.  次の設定を使用して Azure VM をプロビジョニングします。

    -   サブスクリプション: Azure サブスクリプションの名前。

    -   リソース グループ：**az12001b-cl-RG**

    -   仮想マシン名 = **az12001b-vm2**

    -   リージョン: *このラボの最初のエクササイズで Azure VM をデプロイしたのと同じ Azure リージョン*

    -   可用性オプション: **インフラストラクチャ冗長は必要ありません**

    -   画像: **Windows Server 2019 Datacenter**

    -   サイズ: **Standard DS1 v2**

    -   ユーザー名: **Student**

    -   パスワード: **Pa55w.rd1234**

    -   パブリック受信ポート: **選択したポートを許可する**

    -   受信ポートを選択: **RDP (3389)**

    -   Windows ライセンスを既にお持ちの場合: **いいえ**

    -   OS ディスク タイプ: **Standard HDD**

    -   仮想ネットワーク: **adVNET**

    -   サブネット名: **bastionSubnet** *という名前の新しいサブネット*

    -   アドレス範囲: **10.0.255.0/24**

    -   パブリック IP アドレス: **az12001b-vm2-ip** *という名前の新規 IP アドレス*

    -   NIC ネットワーク セキュリティ グループ: **Basic**

    -   パブリック受信ポート: **選択したポートを許可する**

    -   受信ポートを選択: **RDP (3389)**

    -   高速ネットワーク: **オフ**

    -   この仮想マシンを既存の負荷分散ソリューションの後ろに配置しますか: **いいえ**

    -   ブート診断: **オフ**

    -   OS のゲスト診断: **オフ**

    -   システム割り当てマネージド ID **オフ**

    -   AAD 資格情報を使用してログインする (プレビュー): **オフ**

    -   自動シャットダウンを有効にする: **オフ**

    -   バックアップを有効にする: **オフ**

    -   拡張機能: *なし*

    -   タグ: *なし*

1.  プロビジョニングが完了するまで待ちます。通常、これには数分かかります。

1.  RDP 経由で新しくプロビジョニングされた Azure VM に接続します。 

1.  az12001b-vm2 への RDP セッションで、プライベート IP アドレス (それぞれ 10.0.1.4 および 10.0.1.5) を介して az12001b-cl-vm0 と az12001b-cl-vm1 の両方に RDP セッションを確立できることを確認します。 

> **結果**: このエクササイズを完了すると、可用性の高い SAP NetWeaver のデプロイをサポートするために必要となる Azure ネットワーク リソースのプロビジョニングを行うことができます。

## エクササイズ 4: ラボリソースを削除する

時間: 10 分間

このエクササイズでは、このラボでプロビジョニングしたリソースすべてを削除します。

#### タスク 1: Cloud Shell を開く

1. ポータルの上部にある 「**Cloud Shell**」 アイコンをクリックして 「Cloud Shell」 ウィンドウを開き、シェルとして PowerShell を選択します。

1. ポータルの下部にある **Cloud Shell** コマンド プロンプトで、次のコマンドを入力し、「**Enter**」 を押して、このラボで作成したすべてのリソース グループを一覧表示します。

    ```
    Get-AzResourceGroup | Where-Object {$_.ResourceGroupName -like 'az12001b-*'} | Select-Object ResourceGroupName
    ```

1. このラボで作成したリソース グループのみが出力に含まれていることを確認します。これらのグループは、次のタスクで削除されます。

#### タスク 2: リソース グループの削除

1. **Cloud Shell** コマンド プロンプトで、次のコマンドを入力し、「**Enter**」 キーを押して、このラボで作成したリソース グループを削除します。

    ```
    Get-AzResourceGroup | Where-Object {$_.ResourceGroupName -like 'az12001b-*'} | Remove-AzResourceGroup -Force  
    ```

1. ポータルの下部にある **Cloud Shell** プロンプトを閉じます。


> **結果**: このエクササイズを完了すると、このラボで使用したリソースが削除されます。
