# AZ 120 モジュール 1: SAP on Azure の基盤
# ラボ 1a: Azure VM での Linux クラスタリングの実装

予想時間：90 分

このラボのタスクすべては、Azure portal (Bash Cloud Shell セッションを含む) から実行されます  

   > **注記**: Cloud Shell を使用しない場合、ラボの仮想マシンには Azure CLI がインストールされている必要があり [**https://docs.microsoft.com/ja-jp/cli/azure/install-azure-cli-windows?view=azure-cli-latest**](https://docs.microsoft.com/ja-jp/cli/azure/install-azure-cli-windows?view=azure-cli-latest)、 [**https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html**](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) から利用可能な SSH クライアント (PuTTY など) が含まれている必要があります。

ラボ ファイル: なし

## シナリオ
  
Azure に SAP HANA を導入する準備として、Adatum Corporation は、Linux の SUSE ディストリビューションを実行する Azure VM にクラスタリングを実装するプロセスを検討したいと考えています。

## Objectives
  
このラボを終了すると、下記ができるようになります。

- 可用性の高い SAP HANA のデプロイをサポートするために必要となる Azure コンピューティング リソースのプロビジョニングを行う

- 可用性の高い SAP HANA のインストールをサポートするために、Linux を実行している Azure VM のオペレーティング システムを構成する

- 可用性の高い SAP HANA のデプロイをサポートするために必要となる Azure ネットワーク リソースのプロビジョニングを行う

## 必要条件

- 使用可能な DSv3 vCPU (4 x 4) と DSv2 (1 x 1) vCPU の十分な数を持つ Microsoft Azure サブスクリプション

- Azure にアクセス可能な Windows 10、Windows Server 2019、または Windows Server 2016 を実行しているラボ コンピューター


## 演習 1: 可用性の高い SAP HANA のデプロイをサポートするために必要となる Azure コンピューティング リソースのプロビジョニングを行う

時間: 30 分

このエクササイズでは、Linux クラスタリングの構成に必要な Azure インフラストラクチャ コンピューティング コンポーネントをデプロイします。これには、同じ可用性セットで Linux SUSE を実行する Azure VM のペアを作成する必要があります。

### タスク 1: Linux SUSE を実行している Azure VM をデプロイする

1. ラボ コンピューターから Web ブラウザーを起動し、https://portal.azure.com から Azure portal に移動します。

1. プロンプトが表示された場合は、このラボで使用する Azure サブスクリプションの所有者または共同作成者のロールを使用して、職場、学校または個人の Microsoft アカウントを使用してログインします。

1. Azure portal で、Azure portal ページの上部にある **「リソース、サービス、およびドキュメントの検索」** テキスト ボックスを使用して、**「近接配置グループ」** ブレードを検索してナビゲートし、**「近接配置グループ」** ブレードで **「+ 追加」** を選択します。

1. **「近接配置グループの作成」** ブレードの **「基本」** タブで次の設定を指定し、**「Review + create」** を選択します。

   - サブスクリプション: *Azure サブスクリプションの名前*

   - リソース グループ:  **az12001a-RG** *という名前の新規リソース グループ*

   - リージョン: *Azure VM をデプロイできて、ラボの場所に最も近い Azure リージョン*

   - 近接配置グループ名: **az12001a-ppg**

1. **「近接配置グループの作成」** ブレードの **「Review + create」** タブで、「**作成**」 を選択します。

   > **注記**: プロビジョニングが完了するのを待ってください。これには 1 分もかかりません。

1. Azure portal で、Azure portal ページの上部にある **「リソース、サービス、ドキュメントの検索」** テキスト ボックスを使用して **「仮想マシン」** ブレードを検索して検索し、**「仮想マシン」** ブレードで **「+ 追加」** を選択し、ドロップダウン メニューで **「仮想マシン」** を選択します。         

1. 「**仮想マシンの作成**」 ブレードの 「**基本**」 タブで、次の設定を指定して **「次へ:」** を選択します **Disks >** (その他の設定はすべてデフォルト値のままにします):

   - サブスクリプション: *Azure サブスクリプションの名前*

   - リソース グループ:  **az12001a-RG** *という名前の新規リソース グループ*

   - 仮想マシン名: **az12001a-vm0**

   - リージョン: *Azure VM をデプロイできて、ラボの場所に最も近い Azure リージョン*

   - 可用性オプション: **可用性セット**

   - 可用性セット: *2 つの障害ドメインと 5 つの更新ドメインを持つ* **az12001a-avset** *という名前の新規可用性セット*

   - 画像: **SAP 12 SP3 用のエンタープライズ Linux - BYOS - Gen1**

   - Azure Spot インスタンス: **いいえ**

   - サイズ: **Standard D2s v3**

   - 認証タイプ: **パスワード**

   - ユーザー名: **student**

   - パスワード: **Pa55w.rd1234**

1. 「**仮想マシンの作成**」 ブレードの 「**ディスク**」 タブ で、既定値を受け入れて 「**次へ**」 をクリックします。**ネットワーク >** (その他の設定はすべてデフォルト値のままにします):

   - OS ディスク タイプ: **Premium SSD**

   - 暗号化の種類: **(デフォルト) プラットフォーム管理キーによる保管時の暗号化**

1. 「**仮想マシンの作成**」 ブレードの 「**ディスク**」 タブ で、以下の設定を指定して 「**次へ**」 をクリックします。**管理 >** (その他の設定はすべてデフォルト値のままにします):

   - 仮想ネットワーク: **az12001a-RG-vnet** *という名前の新規仮想ネットワーク*

   - アドレス空間: **192.168.0.0/20**

   - サブネット名: **subnet-0**

   - サブネット アドレス範囲: **192.168.0.0/24**

   - パブリック IP アドレス: **az12001a-vm0-ip** *という名前の新規 IP アドレス*

   - NIC ネットワーク セキュリティ グループ: **詳細**

   > **注記**: このイメージは NSG ルールを事前に構成しました

   - 高速ネットワーキング: **オン**

   - この仮想マシンを既存の負荷分散ソリューションの背後に配置します。**いいえ**

1. 「**仮想マシンの作成**」 ブレードの 「**管理**」 タブで、次の設定を指定して **「次へ」** を選択します **詳細 >** (その他の設定はすべてデフォルト値のままにします):

   - 基本プランを有効にする (無料): **いいえ**

   > **注記**: この設定は、Azure Security Center プランを既に選択している場合は使用できません。

   - ブート診断: **管理されたストレージ アカウントで有効にする (推奨)**

   - OS ゲスト診断: **オフ**

   - システム割り当てマネージド ID: **オフ**

   - 自動シャットダウンを有効にする: **オフ**

1. 「**仮想マシンの作成**」 ブレードの 「**詳細**」 タブで、次の設定を指定して **「Review + create」** を選択します (既定値を他のユーザーのままにします)。

   - 近接配置グループ: **az12001a-ppg**

1. **「近接配置グループの作成」** ブレードの **「Review + create」** タブで、「**作成**」 を選択します。

   > **注記**: プロビジョニングが完了するのを待ってください。これは 3 分もかかりません。

1. Azure portal で、Azure portal ページの上部にある **「リソース、サービス、ドキュメントの検索」** テキスト ボックスを使用して **「仮想マシン」** ブレードを検索して検索し、**「仮想マシン」** ブレードで **「+ 追加」** を選択し、ドロップダウン メニューで **「仮想マシン」** を選択します。         

1. 「**仮想マシンの作成**」 ブレードの 「**基本**」 タブで、次の設定を指定して **「次へ:」** を選択します **Disks >** (その他の設定はすべてデフォルト値のままにします):

   - サブスクリプション: *Azure サブスクリプションの名前*

   - リソース グループ： **az12001a-RG**

   - 仮想マシン名: **az12001a-vm1**

   - リージョン: *Azure VM をデプロイできて、ラボの場所に最も近い Azure リージョン*

   - 可用性オプション: **可用性セット**

   - 可用性セット: **az12001a-avset**

   - 画像: **SAP 12 SP3 用のエンタープライズ Linux - BYOS - Gen1**

   - Azure Spot インスタンス: **いいえ**

   - サイズ: **Standard D2s v3**

   - 認証タイプ: **パスワード**

   - ユーザー名: **student**

   - パスワード: **Pa55w.rd1234**

1. 「**仮想マシンの作成**」 ブレードの 「**ディスク**」 タブ で、既定値を受け入れて 「**次へ**」 をクリックします。**ネットワーク >** (その他の設定はすべてデフォルト値のままにします):

   - OS ディスク タイプ: **Premium SSD**

   - 暗号化の種類: **(デフォルト) プラットフォーム管理キーによる保管時の暗号化**

1. 「**仮想マシンの作成**」 ブレードの 「**ディスク**」 タブ で、以下の設定を指定して 「**次へ**」 をクリックします。**管理 >** (その他の設定はすべてデフォルト値のままにします):

   - 仮想ネットワーク: **az12001a-RG-vnet**

   - サブネット: **subnet-0 (192.168.0.0/24)**

   - パブリック IP アドレス: **az12001a-vm1-ip** *という名前の新規 IP アドレス*

   - NIC ネットワーク セキュリティ グループ: **詳細**

   > **注記**: このイメージは NSG ルールを事前に構成しました

   - 高速ネットワーキング: **オン**

   - この仮想マシンを既存の負荷分散ソリューションの背後に配置します。**いいえ**

1. 「**仮想マシンの作成**」 ブレードの 「**管理**」 タブで、次の設定を指定して **「次へ」** を選択します **詳細 >** (その他の設定はすべてデフォルト値のままにします):

   - 基本プランを有効にする (無料): **いいえ**

   > **注記**: この設定は、Azure Security Center プランを既に選択している場合は使用できません。

   - ブート診断: **管理されたストレージ アカウントで有効にする (推奨)**

   - OS ゲスト診断: **オフ**

   - システム割り当てマネージド ID: **オフ**

   - 自動シャットダウンを有効にする: **オフ**

1. 「**仮想マシンの作成**」 ブレードの 「**詳細**」 タブで、次の設定を指定して **「Review + create」** を選択します (既定値を他のユーザーのままにします)。

   - 近接配置グループ: **az12001a-ppg**

1. **「近接配置グループの作成」** ブレードの **「Review + create」** タブで、「**作成**」 を選択します。

   > **注記**: プロビジョニングが完了するのを待ってください。これは 3 分もかかりません。


### タスク 2: Azure VM ディスクを作成して構成する

1. Azure portal の Cloud Shell で Bash セッションを開始します。 

   > **注記**: 現在の Azure サブスクリプションで Cloud Shell を初めて起動する場合は、Azure ファイル共有を作成して Cloud Shell ファイルを永続化するように求められます。その場合は、既定値に設定すると、自動的に生成されたリソース グループ内にストレージ アカウントが作成されます。

1. 「Cloud Shell」 ウィンドウで、次のコマンドを実行して、前のタスクでデプロイした最初の Azure VM に接続する 8 個のマネージド ディスクの最初のセットを作成します。

   ```cli
   RESOURCE_GROUP_NAME='az12001a-RG'

   LOCATION=$(az group list --query "[?name == '$RESOURCE_GROUP_NAME'].location" --output tsv)

   for I in {0..7}; do az disk create --resource-group $RESOURCE_GROUP_NAME --name az12001a-vm0-DataDisk$I --size-gb 128 --location $LOCATION --sku Premium_LRS; done
   ```

1. 「Cloud Shell」 ウィンドウで、次のコマンドを実行して、前のタスクでデプロイした 2 番目の Azure VM に接続する 8 個のマネージド ディスクの 2 番目のセットを作成します。

   ```cli
   for I in {0..7}; do az disk create --resource-group $RESOURCE_GROUP_NAME --name az12001a-vm1-DataDisk$I --size-gb 128 --location $LOCATION --sku Premium_LRS; done
   ```

1. Azure portal で、前のタスク (**az12001a-vm0**) でプロビジョニングした最初の Azure VM のブレードに移動します。

1. **az12001a-vm0** ブレードから、**az12001a-vm0 \|** に移動します。「**ディスク**」 ブレード。

1. 「**az12001a-vm0 \| ディスク**」 ブレードで、「**既存のディスクを接続する**」 を選択して、次の設定でデータ ディスクを az12001a-vm0 に接続します。

   - LUN: **0**

   - ディスク名: **az12001a-vm0-DataDisk0**

   - リソース グループ： **az12001a-RG**

   - ホスト キャッシュ: **読み取り専用**

1. 前の手順を繰り返して、プレフィックス **az12001a-vm0-DataDisk** を使用して残りの 7 つのディスク (合計 8 つ) を接続します。ディスク名の最後の文字と一致する LUN 番号を割り当てます。LUN **1** を使用するディスクのホスト キャッシュを**読み取り専用**に設定し、残りのすべてのディスクのホスト キャッシュを **なし**に設定します。

1. 変更内容を保存します｡ 

1. Azure portal で、前のタスク (**az12001a-vm1**) でプロビジョニングした 2 番目の Azure VM のブレードに移動します。

1. **az12001a-vm1** ブレードから、**az12001a-vm1 \|** に移動します。「**ディスク**」 ブレード。

1. **az12001a-vm1 \|** から 「**ディスク**」 ブレードから、az12001a-vm1 に次の設定を持つデータ ディスクを接続します。

   - LUN: **0**

   - ディスク名: **az12001a-vm1-DataDisk0**

   - リソース グループ： **az12001a-RG**

   - ホスト キャッシュ: **読み取り専用**

1. 前の手順を繰り返して、プレフィックス **az12001a-vm1-DataDisk** を使用して残りの 7 つのディスク (合計 8 つ) を接続します。ディスク名の最後の文字と一致する LUN 番号を割り当てます。LUN **1** を使用するディスクのホスト キャッシュを**読み取り専用**に設定し、残りのすべてのディスクのホスト キャッシュを **なし**に設定します。

1. 変更内容を保存します｡ 

> **結果**: このエクササイズを完了すると、可用性の高い SAP HANA のデプロイをサポートするために必要となる Azure コンピューティング リソースのプロビジョニングを行うことができます。


## 演習 2: 可用性の高い SAP HANA のインストールをサポートするために、Linux を実行している Azure VM のオペレーティング システムを構成する

時間: 30 分

このエクササイズでは、SAP HANA のクラスタ化されたインストールに対応するように、SUSE Linux Enterprise Server を実行している Azure VM 上の OS とストレージを構成します。

### タスク 1: Azure Linux VM に接続する

1. Azure portal の Cloud Shell で Bash セッションを開始します。 

1. 「Cloud Shell」 ウィンドウで、次のコマンドを実行して、前のタスクでデプロイした最初の Azure VM のパブリック IP アドレスを特定します。

   ```cli
   RESOURCE_GROUP_NAME='az12001a-RG'

   PIP=$(az network public-ip show --resource-group $RESOURCE_GROUP_NAME --name az12001a-vm0-ip --query ipAddress --output tsv)
   ```

1. Cloud Shell ペインで次のコマンドを実行して、前の手順で特定した IP アドレスに SSH セッションを確立します。

   ```cli
   ssh student@$PIP
   ```

1. 接続を続行するかどうかを確認するメッセージが表示されたら、`yes`と入力し、**Enter** キーを押します。 

1. パスワードの入力を求めるメッセージが表示されたら、`Pa55w.rd1234`と入力し、**Enter** キーを押します。 

1. 「Cloud Shell」 ツールバーの 「**新しいセッションを開く**」 アイコンをクリックして、別の Cloud Shell Bash セッションを開きます。

1. 新しく開いた Cloud Shell Bash セッションで、このタスクのすべての手順を繰り返して、IP アドレス **az12001a-vm0-ip** を介して **az12001a-vm1** Azure VM に接続します。


### タスク 2: Linux を実行している Azure VM のストレージを構成する

1. Cloud Shell ペインの az12001a-vm0 への SSH セッションで、次のコマンドを実行して特権を昇格します。 

   ```cli
   sudo su -
   ```

1. Cloud Shell ペインの az12001a-vm0 への SSH セッションで、次のコマンドを実行して、新しく接続されたデバイスとその LUN 番号の間のマッピングを特定します。
   
   ```cli
   lsscsi
   ```

1. Cloud Shell ペインの az12001a-vm0 への SSH セッションで、次を実行して (8 つのうち) 6 つのデータ ディスクの物理ボリュームを作成します。
   
   ```cli
   pvcreate /dev/sdc
   pvcreate /dev/sdd
   pvcreate /dev/sde
   pvcreate /dev/sdf
   pvcreate /dev/sdg
   pvcreate /dev/sdh
   ```

1. Cloud Shell ペインの az12001a-vm0 への SSH セッションで、次を実行してボリューム グループを作成します。
   
   ```cli
   vgcreate vg_hana_data /dev/sdc /dev/sdd
   vgcreate vg_hana_log /dev/sde /dev/sdf
   vgcreate vg_hana_backup /dev/sdg /dev/sdh
   ```

1. Cloud Shell ペインの az12001a-vm0 への SSH セッションで、次を実行して論理ボリュームを作成します。

   ```cli
   lvcreate -l 100%FREE -n hana_data vg_hana_data
   lvcreate -l 100%FREE -n hana_log vg_hana_log
   lvcreate -l 100%FREE -n hana_backup vg_hana_backup
   ```

   > **注記**: 各ボリューム グループに 1 つの論理ボリュームを作成しています

1. 「Cloud Shell」 ウィンドウの az12001a-vm0 への SSH セッションで、次を実行して論理ボリュームをフォーマットします。

   ```cli
   mkfs.xfs /dev/vg_hana_data/hana_data -m crc=1
   mkfs.xfs /dev/vg_hana_log/hana_log -m crc=1
   mkfs.xfs /dev/vg_hana_backup/hana_backup -m crc=1
   ```

   > **注記**: SUSE Linux Enterprise Server 12 以降では、XFS メタデータの自動チェックサム、ファイル タイプ サポート、およびファイルごとのアクセス制御リスト数の制限を提供する、XFS ファイル システムの新しいディスク上フォーマット (v5) を使用するオプションがあります。YaST を使用して XFS ファイル システムを作成すると、新しいフォーマットが自動的に適用されます。互換性上の理由から古い形式で XFS ファイルシステムを作成するには、`-m crc=1` オプションを指定しないで mkfs.xfs コマンドを使用します。 

1. Cloud Shell ペインの az12001a-vm0 への SSH セッションで、次を実行して **/dev/sdi** ディスクをパーティション化します。

   ```cli
   fdisk /dev/sdi
   ```

1. プロンプトが表示されたら、順番に `n`、`p`、`1` (毎回 **Enter**  キーを押す) と入力し **Enter** キーを 2 回押します。次に `w` と入力して書き込みを完了します。

1. 「Cloud Shell」 ウィンドウの az12001a-vm0 への SSH セッションで、次を実行して **/dev/sdj** ディスクをパーティション化します。

   ```cli
   fdisk /dev/sdj
   ```

1. プロンプトが表示されたら、順番に `n`、`p`、`1` (毎回 **Enter**  キーを押す) と入力し **Enter** キーを 2 回押します。次に `w` と入力して書き込みを完了します。

1. Cloud Shell ペインの ssh セッションで、az12001a-vm0 を実行して新しく作成したパーティションを書式設定します (`y` を入力し、確認を求められたら **Enter** キーを押します)。

   ```cli
   mkfs.xfs /dev/sdi -m crc=1 -f
   mkfs.xfs /dev/sdj -m crc=1 -f
   ```

1. Cloud Shell ペインの az12001a-vm0 への SSH セッションで、次を実行してマウント ポイントの役割を果たすディレクトリを作成します。

   ```cli
   mkdir -p /hana/data
   mkdir -p /hana/log
   mkdir -p /hana/backup
   mkdir -p /hana/shared
   mkdir -p /usr/sap
   ```

1. Cloud Shell ペインの az12001a-vm0 への SSH セッションで、次を実行して論理ボリュームの ID を表示します。

   ```cli
   blkid
   ```

   > **注記**: **/dev/sdi** (**/hana/shared**に使用) と **dev/sdj** (**/usr/sap**に使用) を含む、新しく作成したボリューム グループおよびパーティションに関連付けられた **UUID** 値を特定します。


1. 「Cloud Shell」 ウィンドウの az12001a-vm0 への SSH セッションで、次を実行して vi エディターで  **/etc/fstab** を開きます (他のエディターを自由に使用できます)。

   ```cli
   vi /etc/fstab
   ```

1. エディターで、次のエントリを **/etc/fstab** に追加します (ここで 、`\<UUID of /dev/vg\_hana\_data-hana\_data\>`、`\<UUID of /dev/vg\_hana\_log-hana\_log\>`、`\<UUID of /dev/vg\_hana\_backup-hana\_backup\>`、`\<UUID of /dev/vg_hana_shared-hana_shared (/dev/sdi)\>`、`\<UUID of /dev/vg_usr_sap-usr_sap (/dev/sdj)\>` は、前の手順で特定した ID を表します)。

   ```cli
   /dev/disk/by-uuid/<UUID of /dev/vg_hana_data-hana_data> /hana/data xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<UUID of /dev/vg_hana_log-hana_log> /hana/log xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<UUID of /dev/vg_hana_backup-hana_backup> /hana/backup xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<UUID of /dev/vg_hana_shared-hana_shared (/dev/sdi)> /hana/shared xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<UUID of /dev/vg_usr_sap-usr_sap (/dev/sdj)> /usr/sap xfs  defaults,nofail  0  2
   ```

1. 変更を保存し、エディタを閉じます。

1. Cloud Shell ペインの SSH セッション～az12001a-vm0 内で、次のコマンドを実行して新しいボリュームをマウントします。

   ```cli
   mount -a
   ```

1. Cloud Shell ペインの az12001a-vm0 への SSH セッションで、次を実行してマウントが行われたことを確認します。

   ```cli
   df -h
   ```

1. az12001a-vm1 への Cloud Shell Bash セッションに切り替え、このタスクのすべての手順を繰り返して、**az12001a-vm1** でストレージを構成します。


### タスク 3: クロスノード パスワードレス SSH アクセスを有効にする

1. Cloud Shell ペインの az12001a-vm0 への SSH セッションで、次を実行してパスフレーズレス SSH キーを生成します。

   ```cli
   ssh-keygen -tdsa
   ```

1. プロンプトが表示されたら、**Enter** を 3 回押してから、次を実行して公開キーを表示します。 

   ```cli
   cat /root/.ssh/id_dsa.pub
   ```

1. キーの値をクリップボードにコピーします。

1. SSH セッションを含む Cloud Shell ペインの **az12001a-vm1** に切り替え、次を実行してディレクトリ **/root/.ssh/** を作成します。

   ```cli
   mkdir /root/.ssh
   ```

1. Cloud Shell ペインの az12001a-vm1 への SSH セッションで、次を実行して vi エディターでファイル  **/root/.ssh/authorized\_keys** を作成します (他のエディターを自由に使用できます)。

   ```cli
   vi /root/.ssh/authorized_keys
   ```

1. エディター ウィンドウで、az12001a-vm0 で生成したキーを貼り付けます。

1. 変更を保存し、エディターを閉じます。

1. 「Cloud Shell」 ウィンドウの az12001a-vm1 への SSH セッションで、次を実行してパスフレーズレス SSH キーを生成します。

   ```cli
   ssh-keygen -tdsa
   ```

1. プロンプトが表示されたら、**Enter** を 3 回押してから、次を実行して公開キーを表示します。 

   ```cli
   cat /root/.ssh/id_dsa.pub
   ```

1. キーの値をクリップボードにコピーします。

1. SSH セッションを含む Cloud Shell ペインの az12001a-vm0 に切り替え、次を実行して vi エディターでファイル **/root/.ssh/authorized\_keys** を作成します (他のエディターを自由に使用できます)。

   ```cli
   vi /root/.ssh/authorized_keys
   ```

1. エディター ウィンドウで、az12001a-vm1 で生成したキーを貼り付けます。

1. 変更を保存し、エディターを閉じます。

1. 「Cloud Shell」 ウィンドウの az12001a-vm0 への SSH セッションで、次を実行してパスフレーズレス SSH キーを生成します。

   ```cli
   ssh-keygen -t rsa
   ```

1. プロンプトが表示されたら、**Enter** を 3 回押してから、次を実行して公開キーを表示します。 

   ```cli
   cat /root/.ssh/id_rsa.pub
   ```

1. キーの値をクリップボードにコピーします。

1. SSH セッションを含む Cloud Shell ペインの **az12001a-vm1** に切り替え、次を実行して vi エディターでファイル **/root/.ssh/authorized\_keys** を開きます (他のエディターを自由に使用できます)。

   ```cli
   vi /root/.ssh/authorized_keys
   ```

1. エディター ウィンドウで、新しい行から開始し、az12001a-vm0 で生成したキーを貼り付けます。

1. 変更を保存し、エディタを閉じます。

1. Cloud Shell ペインの SSH セッション～az12001a-vm1 内で、次のコマンドを実行してパスフレーズ不要の SSH キーを生成します。

   ```cli
   ssh-keygen -t rsa
   ```

1. プロンプトが表示されたら、**Enter** を 3 回押してから、次を実行して公開キーを表示します。 

   ```cli
   cat /root/.ssh/id_rsa.pub
   ```

1. キーの値をクリップボードにコピーします。

1. SSH セッションを含む 「Cloud Shell」 ウィンドウの az12001a-vm0 に切り替え、次を実行して vi エディターでファイル **/root/.ssh/authorized\_keys** を開きます (他のエディターを自由に使用できます)。

   ```cli
   vi /root/.ssh/authorized_keys
   ```

1. エディター ウィンドウで、新しい行から開始し、az12001a-vm1 で生成したキーを貼り付けます。

1. 変更を保存し、エディタを閉じます。

1. Cloud Shell ペインの SSH セッション～az12001a-vm0 内で、次のコマンドを実行して vi エディタで **/etc/ssh/sshd\_config** ファイルを開きます (他のエディタも使用可能)。

   ```cli
   vi /etc/ssh/sshd_config
   ```

1. **/etc/ssh/sshd\_config** ファイルで、 **PermitRootLogin** エントリと **AuthorizedKeysFile** エントリを検索し、次のように構成します (必要に応じて先頭の **#** 文字を削除します)。

   ```cli
   PermitRootLogin yes
   AuthorizedKeysFile  /root/.ssh/authorized_keys
   ```

1. 変更を保存し、エディタを閉じます。

1. Cloud Shell ペインの SSH セッション～az12001a-vm0 内で、次のコマンドを実行して sshd デーモンを再起動します。

   ```cli
   systemctl restart sshd
   ```

1. az12001a-vm1 で前の 4 つの手順を繰り返します。

1. 構成が成功したことを確認するには、 「Cloud Shell」 ウィンドウの SSH セッションで az12001a-vm0 に対して、次を実行して az12001a-vm0 から az12001a-vm1 への SSH セッションを **root** として確立します。 

   ```cli
   ssh root@az12001a-vm1
   ```

1. 接続を続行するかどうかを確認するメッセージが表示されたら、`yes`と入力し、**Enter** キーを押します。 

1. パスワードの入力が求められないことを確認します。

1. 次のコマンドを実行して、az12001a-vm0 から az12001a-vm1 までの SSH セッションを閉じます。 

   ```cli
   exit
   ```

1. 次を 2 回実行して、az12001a-vm0 からサインアウトします。

   ```cli
   exit
   ```

1. 構成が成功したことを確認するには、 「Cloud Shell」 ウィンドウの SSH セッションで az12001a-vm1 に対して、次を実行して az12001a-vm1 から az12001a-vm0 への SSH セッションを **root** として確立します。 

   ```cli
   ssh root@az12001a-vm0
   ```

1. 接続を続行するかどうかを確認するメッセージが表示されたら、`yes`と入力し、**Enter** キーを押します。 

1. パスワードの入力が求められないことを確認します。

1. 次のコマンドを実行して、az12001a-vm1 から az12001a-vm0 までの SSH セッションを閉じます。 

   ```cli
   exit
   ```

1. 次を 2 回実行して、az12001a-vm1 からサインアウトします。

   ```cli
   exit
   ```

> **結果**: このエクササイズを完了すると、可用性の高い SAP HANA インストールをサポートするように、Linux を実行している Azure VM のオペレーティング システムを構成することができます。


## 演習 3: 可用性の高い SAP HANA のデプロイをサポートするために必要となる Azure ネットワーク リソースのプロビジョニングを行う

時間: 30 分

このエクササイズでは、SAP HANA のクラスター化されたインストールに対応するために Azure Load Balancers を実装します。


### タスク 1: 負荷分散のセットアップが容易に行えるよう Azure VM を構成します。

   > **注記**: Stardard SKU の Azure Load Balancer のペアを設定するので、まず、負荷分散バックエンド プールとして機能する 2 つの Azure VM のネットワーク アダプターに関連付けられたパブリック IP アドレスを削除する必要があります。

1. Azure portal で、**az12001a-vm0** Azure VM のブレードに移動します。

1. **az12001a-vm0** ブレードから、**az12001a-vm0 \|** に移動します。**「ネットワーク」** ブレードと、**az12001a-vm0 \| 「ネットワーク」** ブレードから、ネットワーク アダプターに関連付けられた **az12001a-vm0-ip** という名前のパブリック IP アドレスを表すエントリを選択します。

1. **「az12001a-vm0-ip」** ブレードで、**「切り離し」** を選択してパブリック IP アドレスをネットワーク インターフェイスから切り離し、次に **「削除」** を選択して削除します。

1. Azure portal で、**az12001a-vm1**  Azure VM のブレードに移動します。

1. **az12001a-vm1** ブレードから、**az12001a-vm1 \|** に移動します。**ネットワーク** ブレードと、**az12001a-vm1 \| 「ネットワーク」** ブレードから、ネットワーク アダプターに関連付けられた **「az12001a-vm1-ip」** という名前のパブリック IP アドレスを表すエントリを選択します。

1. **「az12001a-vm1-ip」** ブレードで、**「切り離す」** を選択してパブリック IP アドレスをネットワーク インターフェイスから切断し、**「削除」** を選択して削除します。

1. Azure portal で、**az12001a-vm0** Azure VM のブレードに移動します。

1. **az12001a-vm0** ブレードから、**az12001a-vm0 \|** に移動します。**ネットワーク** ブレード。 

1. **az12001a-vm0 \|** から **「ネットワーク」** ブレードで、az12001a-vm0 のネットワーク インターフェイスを表すエントリを選択します。 

1. az12001a-vm0 のネットワーク インターフェースのブレードから、その IP 構成ブレードに移動し、そこから **ipconfig1** ブレードを表示します。

1. **ipconfig1** ブレードで、プライベート IP アドレスの割り当てを **Static** に設定し、変更を保存します。

1. Azure portal で、**az12001a-vm1**  Azure VM のブレードに移動します。

1. **az12001a-vm1** ブレードから、**az12001a-vm1 \|** に移動します。**ネットワーク** ブレード。 

1. **az12001a-vm1 \|** から **「ネットワーク」** ブレードから、az12001a-vm1 のネットワーク インターフェイスに移動します。 

1. az12001a-vm1 のネットワーク インターフェースのブレードから、その IP 構成ブレードに移動し、そこから **ipconfig1** ブレードを表示します。

1. **ipconfig1** ブレードで、プライベート IP アドレスの割り当てを **Static** に設定し、変更を保存します。


### タスク 2: 受信トラフィックを処理する Azure Load Balancers を作成して構成する

1. Azure portal で、Azure ポータル ページの上部にある **「リソース、サービス、およびドキュメントの検索」** テキスト ボックスを使用して、**「ロード バランサー」** ブレードを検索して移動し、**「ロード バランサー」** ブレードで **「+ 追加」** を選択します。       

1. 「**ロード バランサ―の作成**」 ブレードの 「**基本**」 タブで、次の設定を指定して **「Review + create」** を選択します (他の設定は既定値のままにします)。

   - サブスクリプション: *Azure サブスクリプションの名前*

   - リソース グループ： **az12001a-RG**

   - 名前: **az12001a-lb0**

   - リージョン: *このラボの最初のエクササイズで Azure VM をデプロイしたのと同じ Azure リージョン*

   - タイプ: **内部**

   - SKU: **Standard**

   - 仮想ネットワーク: **az12001a-RG-vnet**

   - サブネット: **subnet-0**

   - IP アドレスの割り当て: **静的**

   - IP アドレス: **192.168.0.240**

   - 可用性ゾーン: **ゾーン冗長**

1. **「Review + create」** ブレードで、**「作成」** を選択します。

   > **注記**: ロード バランサーがプロビジョニングされるまで待ちます。これには 1 分もかかりません。 

1. Azure portal で、新しくプロビジョニングされた **az12001a-lb0** という名前のロード バランサ―のプロパティを表示するブレードに移動します。 

1. **「az12001a lb0」** ブレードで、**「バックエンド プール」** を選択し、**「+ 追加」** を選択し、**「バックエンド プールを追加」** で次の設定を指定します (他は既定の設定のままにします)。       

   - 名前: **az12001a-lb0-bepool**

   - IP バージョン: **Pa55w.rd1234**

   - 仮想マシン: **az12001a-vm0**  IP 構成: **ipconfig1 (192.168.0.4)**

   - 仮想マシン: **az12001a-vm1**  IP 構成: **ipconfig1  (192.168.0.5)**

1. **「az12001a lb0」** ブレードで、**「正常性プローブ」** を選択して **「追加」** を選択し、**「正常性プローブの追加」** ブレードで次の設定を指定します (他は既定値のままにします)。

   - 名前: **az12001a-lb0-hprobe**

   - プロトコル: **TCP**

   - ポート: **120**

   - 間隔: **5** *秒*

   - 異常しきい値: **2** *つの連続エラー*

1. **「az12001a lb0」** ブレードで、**「負荷分散規則」** を選択し、**「+ 追加」** を選択し、**「負荷分散規則を追加」** で、次の設定を指定します (他は既定のままにします)。

   - 名前: **az12001a-lb0-lbruleAll**

   - IP バージョン: **Pa55w.rd1234**

   - フロントエンド IP アドレス: **192.168.0.240 (LoadBalancerFrontEnd)**

   - HAポート: **有効化された**

   - バックエンド プール: **az12001a-lb0-bepool (2 台の仮想マシン)**

   - 正常性プローブ: **az12001a-lb0-hprobe (TCP:62504)**

   - セッション永続化: **なし**

   - アイドル タイムアウト (分): **4**

   - TCP リセット**無効**

   - Floating IP (Direct Server Return): **有効化された**

### タスク 3: 送信トラフィックを処理する Azure Load Balancers を作成して構成する

1. Azure portal の Cloud Shell で Bash セッションを開始します。 

1. 「Cloud Shell」 ウィンドウで次のコマンドを実行して、2 番目の Load Balancer で使用するパブリック IP アドレスを作成します。

   ```cli
   RESOURCE_GROUP_NAME='az12001a-RG'

   LOCATION=$(az group list --query "[?name == '$RESOURCE_GROUP_NAME'].location" --output tsv)

   PIP_NAME='az12001a-lb1-pip'

   az network public-ip create --resource-group $RESOURCE_GROUP_NAME --name $PIP_NAME --sku Standard --location $LOCATION
   ```

1. 「Cloud Shell」 ウィンドウで次のコマンドを実行して、2 番目の Load Balancer を作成します。

   ```cli
   LB_NAME='az12001a-lb1'

   LB_BE_POOL_NAME='az12001a-lb1-bepool'

   LB_FE_IP_NAME='az12001a-lb1-fe'

   az network lb create --resource-group $RESOURCE_GROUP_NAME --name $LB_NAME --sku Standard --backend-pool-name $LB_BE_POOL_NAME --frontend-ip-name $LB_FE_IP_NAME --location $LOCATION --public-ip-address $PIP_NAME
   ```

1. 「Cloud Shell」 ウィンドウで次のコマンドを実行して、2 番目の Load Balancer のアウトバウンド規則を作成します。

   ```cli
   LB_RULE_OUTBOUND='az12001a-lb1-ruleoutbound'

   az network lb outbound-rule create --resource-group $RESOURCE_GROUP_NAME --lb-name $LB_NAME --name $LB_RULE_OUTBOUND --frontend-ip-configs $LB_FE_IP_NAME --protocol All --idle-timeout 4 --outbound-ports 1000 --address-pool $LB_BE_POOL_NAME
   ```

1. 「Cloud Shell」 ペインを閉じます。

1. Azure portal で、新しくデプロイされた Azure Load Balancer **az12001a-lb** のプロパティを表示するブレードに移動します。

1. **az12001a-lb1** ブレードで、「**バックエンド プール**」 をクリックします。

1. 「**az12001a-lb1 \| バックエンド プール」** ブレードで、**「az12001a-lb1-bepool」** をクリックします。

1. **az12001a-lb1-bepool** ブレードで、次の設定を指定し、「**保存**」 をクリックします。

   - 仮想ネットワーク: **az12001a-rg-vnet (2 VM)**

   - 仮想マシン: **az12001a-vm0**  IP 構成: **ipconfig1 (192.168.0.4)**

   - 仮想マシン: **az12001a-vm1**  IP 構成: **ipconfig1 (192.168.0.5)**

### タスク 4: ジャンプ ホストをデプロイする

   > **注記**: 2 つのクラスター化された Azure VM にはインターネットから直接アクセスできないため、ジャンプ ホストとして機能する Windows Server 2019 Datacenter を実行する Azure VM をデプロイします。 

1. ラボ コンピューターから Azure Portal で、Azure portal ページの上部にある **「リソース、サービス、ドキュメントの検索」** テキスト ボックスを使用して **「仮想マシン」** ブレードを検索して検索し、**「仮想マシン」** ブレードで **「+ 追加」** を選択し、ドロップダウン メニューで **「仮想マシン」** を選択します。         

1. 「**仮想マシンの作成**」 ブレードの 「**基本**」 タブで、次の設定を指定して **「次へ:」** を選択します **Disks >** (その他の設定はすべてデフォルト値のままにします):

   - サブスクリプション: *Azure サブスクリプションの名前*

   - リソース グループ： **az12001a-RG**

   - 仮想マシン名: **az12001a-vm2**

   - リージョン: *このラボの最初のエクササイズで Azure VM をデプロイしたのと同じ Azure リージョン*

   - 可用性オプション: **インフラストラクチャ冗長は必要ありません**

   - 画像: **Windows Server 2019 Datacenter- Gen1**

   - Azure Spot インスタンス: **いいえ**

   - ユーザー名: **student**

   - パスワード: **Pa55w.rd1234**

   - パブリック受信ポート: **Allow selected ports (選択したポートを許可する)**

   - 選択したインバウンド ポート: **RDP (3389)**

   - 既存の Windows サーバー ライセンスを使用しますか? **いいえ**

1. 「**仮想マシンの作成**」 ブレードの 「**ディスク**」 タブ で、既定値を受け入れて 「**次へ**」 をクリックします。**ネットワーク >** (その他の設定はすべてデフォルト値のままにします):

   - OS ディスク タイプ: **標準 HDD**

   - 暗号化の種類: **(デフォルト) プラットフォーム管理キーによる保管時の暗号化**

1. 「**仮想マシンの作成**」 ブレードの 「**ディスク**」 タブ で、以下の設定を指定して 「**次へ**」 をクリックします。**管理 >** (その他の設定はすべてデフォルト値のままにします):

   - 仮想ネットワーク: **az12001a-RG-vnet**

   - サブネット名: **subnet-0 (192.168.0.0/24)**

   - パブリック IP アドレス: **az12001a-vm2-ip** *という名前の新規 IP アドレス*

   - NIC ネットワーク セキュリティ グループ: **Basic**

   - パブリック受信ポート: **Allow selected ports (選択したポートを許可する)**

   - インバウンド ポートを選択: **RDP (3389)**

   - 高速ネットワーキング: **オフ**

   - この仮想マシンを既存の負荷分散ソリューションの背後に配置します。**いいえ**

1. 「**仮想マシンの作成**」 ブレードの 「**管理**」 タブで、次の設定を指定して **「次へ」** を選択します **詳細 >** (その他の設定はすべてデフォルト値のままにします):

   - 基本プランを有効にする (無料): **いいえ**

   > **注記**: この設定は、Azure Security Center プランを既に選択している場合は使用できません。

   - ブート診断: **管理されたストレージ アカウントで有効にする (推奨)**

   - OS ゲスト診断: **オフ**

   - システム割り当てマネージド ID: **オフ**

   - 自動シャットダウンを有効にする: **オフ**

   - バックアップを有効にする: **オフ**

   - ゲスト OS の更新: **手動パッチ: 自分または別の修正プログラムソリューションを使用してパッチをインストールする**

1. 「**仮想マシンの作成**」 ブレードの 「**詳細**」 タブで、**「Review + create」** を選択します (他の設定はすべて既定値のままにします)。

1. **「近接配置グループの作成」** ブレードの **「Review + create」** タブで、「**作成**」 を選択します。

   > **注記**: プロビジョニングが完了するのを待ってください。これは 3 分もかかりません。

1. RDP 経由で新しくプロビジョニングされた Azure VM に接続します。 

1. az12001a-vm2 への RDP セッションで、[**https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html**](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) から PuTTY をダウンロードします。

1. Private IP Addresses (192.168.0.4 と 192.168.0.5) を介して az12001a-vm0 と az12001a-vm1 の両方に SSH セッションを確立できることを確認します。 

> **結果**: このエクササイズを完了すると、可用性の高い SAP HANA のデプロイをサポートするために必要となる Azure ネットワーク リソースのプロビジョニングを行うことができます。


## 演習 4: ラボリソースを削除する

時間: 10 分

このエクササイズでは、このラボでプロビジョニングしたリソースすべてを削除します。

#### タスク 1: Cloud Shell を開く

1. ポータルの上部にある 「**Cloud Shell**」 アイコンをクリックして Cloud Shell ウィンドウを開き、シェルとして Bash を選択します。

1. ポータルの下部にある **「Cloud Shell」** コマンド プロンプトで、次のコマンドを入力し、**Enter** キーを押して、このラボで作成したすべてのリソース グループを一覧表示します。

   ```cli
   az group list --query "[?starts_with(name,'az12001a-')]".name --output tsv
   ```

1. このラボで作成したリソース グループのみが出力に含まれていることを確認します。これらのグループは、次のタスクで削除されます。

#### タスク 2: リソース グループの削除

1. **「Cloud Shell」** コマンド プロンプトで、次のコマンドを入力し、 **「Enter」** キーを押してこの実習ラボで作成したリソース グループを削除します。

   ```cli
   az group list --query "[?starts_with(name,'az12001a-')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

1. ポータルの下部にある **「Cloud Shell」** プロンプトを閉じます。

> **結果**: このエクササイズを完了すると、このラボで使用したリソースが削除されます。
