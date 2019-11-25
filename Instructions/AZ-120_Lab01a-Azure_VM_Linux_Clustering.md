---
lab:
    title: '3a: Azure VM での Linux クラスタリングの実装'
    module: 'モジュール 3: Azure での SAP 認定オファリング'
---

# AZ 120 モジュール 3: Azure での SAP 認定オファリング
# ラボ 3a: Azure VM での Linux クラスタリングの実装

推定時間: 90 分間

このラボのタスクすべては、Azure portal (Bash Cloud Shell セッションを含む) から実行されます  

   > **注記**: Cloud Shell を使用しない場合、ラボの仮想マシンには Azure CLI がインストールされている必要があり [**https://docs.microsoft.com/ja-jp/cli/azure/install-azure-cli-windows?view=azure-cli-latest**](https://docs.microsoft.com/ja-jp/cli/azure/install-azure-cli-windows?view=azure-cli-latest)、 [**https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html**](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) から利用可能な SSH クライアント (PuTTY など) が含まれている必要があります。 

ラボ ファイル: なし

## シナリオ
  
Azure に SAP HANA を導入する準備として、Adatum Corporation は、Linux の SUSE ディストリビューションを実行する Azure VM にクラスタリングを実装するプロセスを検討したいと考えています。

## 目的
  
このラボを終了すると、下記ができるようになります:

-   可用性の高い SAP HANA のデプロイをサポートするために必要となる Azure コンピューティング リソースのプロビジョニングを行う

-   可用性の高い SAP HANA のインストールをサポートするために、Linux を実行している Azure VM のオペレーティング システムを構成する

-   可用性の高い SAP HANA のデプロイをサポートするために必要となる Azure ネットワーク リソースのプロビジョニングを行う

## 必要条件

-   使用可能な DSv3 vCPU (2 x 4) と DSv2 (1 x 1) vCPU の十分な数を持つ Microsoft Azure サブスクリプション

-   Azure にアクセス可能な Windows 10、Windows Server 2019、または Windows Server 2016 を実行しているラボ コンピューター


## エクササイズ 1: 可用性の高い SAP HANA のデプロイをサポートするために必要となる Azure コンピューティング リソースのプロビジョニングを行う

時間: 30 分間

このエクササイズでは、Linux クラスタリングの構成に必要な Azure インフラストラクチャ コンピューティング コンポーネントをデプロイします。これには、同じ可用性セットで Linux SUSE を実行する Azure VM のペアを作成する必要があります。

### タスク 1: Linux SUSE を実行している Azure VM をデプロイする

1.  ラボ コンピューターから Web ブラウザーを起動し、https://portal.azure.com から Azure portal に移動します。

1.  プロンプトが表示されたら、このラボで使用する Azure サブスクリプションの所有者または共同作成者ロールを使用して、職場用、学校用、または個人の Microsoft アカウントでサインインします。

1.  Azure portal で、[**+ リソースの作成 **] をクリックします。

1.  [**新規**] ブレードから、**SLES 12 SP3 (BYOS)** イメージに基づいて新規 Azure VM の作成を開始します。 

    -   サブスクリプション: *Azure サブスクリプションの名前*。

    -   リソース グループ:  **az12001a-RG** *という名前の新規リソース グループ*

    -   仮想マシン名 = **az12001a-vm0**

    -   リージョン: *Azure VM をデプロイできて、ラボの場所に最も近い Azure リージョン*

    -   可用性オプション: **可用性セット**

    -   可用性セット: *2 つの障害ドメインと 5 つの更新ドメインを持つ* **az12001a-avset** *という名前の新規可用性セット*

    -   画像: **SUSE Linux Enterprise Server (SLES) 12 SP3 (BYOS)**

    -   サイズ: **Standard D4s v3**

    -   認証タイプ:**パスワード**

    -   ユーザー名: **受講者**

    -   パスワード: **Pa55w.rd1234**

    -   パブリック受信ポート: **選択したポートを許可する**

    -   選択した受信ポート: **SSH (22)**

    -   OS ディスク タイプ: **Premium SSD**

    -   仮想ネットワーク: **az12001a-RG-vnet** *という名前の新規仮想ネットワーク* 

    -   アドレス空間: **192.168.0.0/20**

    -   サブネット名: **subnet-0**

    -   サブネット アドレス範囲: **192.168.0.0/24**

    -   パブリック IP アドレス: **az12001a-vm0-ip** *という名前の新規 IP アドレス* 

    -   NIC ネットワーク セキュリティ グループ: **詳細**

    -   ネットワーク セキュリティ グループの構成: **az12001a-vm0-nsg** *という名前の新しいネットワーク セキュリティ グループ (受信 SSH 接続を許可する)*

    -   高速ネットワーク: **オフ**

    -   この仮想マシンを既存の負荷分散ソリューションの後ろに配置しますか: **いいえ**

    -   無料の Basic プランを有効にする: **いいえ**

    -   ブート診断: **オフ**

    -   OS のゲスト診断: **オフ**

    -   システム割り当てマネージド ID**オフ**

    -   AAD 資格情報を使用してログインする (プレビュー): **オフ**

    -   自動シャットダウンを有効にする: **オフ**

    -   拡張機能: *なし*

    -   タグ: **なし**

1.  プロビジョニングが完了するまで待ちます。通常、これには数分かかります。

1.  Azure portal で、[**+ リソースの作成 **] をクリックします。

1.  [**新規**] ブレードから、**SLES 12 SP3 (BYOS)** イメージに基づいて新規 Azure VM の作成を開始します。

    -   サブスクリプション: Azure サブスクリプションの名前。

    -   リソース グループ：**az12001a-RG**

    -   仮想マシン名 = **az12001a-vm1**

    -   リージョン: *最初の Azure VM をデプロイしたのと同じ Azure リージョン*

    -   可用性オプション: **可用性セット**

    -   可用性セット: **az12001a-avset**

    -   画像: **SUSE Linux Enterprise Server (SLES) 12 SP3 (BYOS)**

    -   サイズ: **Standard D4s v3**

    -   認証タイプ: **パスワード**

    -   ユーザー名: **受講者**

    -   パスワード: **Pa55w.rd1234**

    -   パブリック受信ポート: **選択したポートを許可する**

    -   選択した受信ポート: **SSH (22)**

    -   OS ディスク タイプ: **Premium SSD**

    -   仮想ネットワーク: **az12001a-RG-vnet**

    -   サブネット名: **subnet-0 (192.168.0.0/24)**

    -   パブリック IP アドレス: **az12001a-vm1-ip** *という名前の新規 IP アドレス* 

    -   NIC ネットワーク セキュリティ グループ: **詳細**

    -   ネットワーク セキュリティ グループの構成: **az12001a-vm1-nsg** *という名前の新しいネットワーク セキュリティ グループ (受信 SSH 接続を許可する)*

    -   高速ネットワーク: **オフ**

    -   この仮想マシンを既存の負荷分散ソリューションの後ろに配置しますか: **いいえ**

    -   無料の Basic プランを有効にする: **いいえ**

    -   ブート診断: **オフ**

    -   OS のゲスト診断: **オフ**

    -   システム割り当てマネージド ID **オフ**

    -   AAD 資格情報を使用してログインする (プレビュー): **オフ**

    -   自動シャットダウンを有効にする: **オフ**

    -   拡張機能: *なし*

    -   タグ: **なし**

1.  プロビジョニングが完了するまで待ちます。通常、これには数分かかります。


### タスク 2: Azure VM ディスクを作成して構成する

1.  Azure portal の Cloud Shell で Bash セッションを開始します。 

    > **注記**: 現在の Azure サブスクリプションで Cloud Shell を初めて起動する場合は、Azure ファイル共有を作成して Cloud Shell ファイルを保持するように求められます。その場合、既定値を受け入れると、自動的に生成されたリソース グループにストレージ アカウントが作成されます。

1.  [Cloud Shell] ウィンドウで、次のコマンドを実行して、前のタスクでデプロイした最初の Azure VM に接続する 8 個のマネージド ディスクの最初のセットを作成します。

```
RESOURCE_GROUP_NAME='az12001a-RG'

LOCATION=$(az group list --query "[?name == '$RESOURCE_GROUP_NAME'].location" --output tsv)

for I in {0..7}; do az disk create --resource-group $RESOURCE_GROUP_NAME --name az12001a-vm0-DataDisk$I --size-gb 128 --location $LOCATION --sku Premium_LRS; done
```

1.  [Cloud Shell] ウィンドウで、次のコマンドを実行して、前のタスクでデプロイした 2 番目の Azure VM に接続する 8 個のマネージド ディスクの 2 番目のセットを作成します。

```
for I in {0..7}; do az disk create --resource-group $RESOURCE_GROUP_NAME --name az12001a-vm1-DataDisk$I --size-gb 128 --location $LOCATION --sku Premium_LRS; done
```

1.  Azure portal で、前のタスク (**az12001a-vm0**) でプロビジョニングした最初の Azure VM のブレードに移動します。

1.  **az12001a-vm0** ブレードから、**az12001a-vm0 - ディスク** ブレードに移動します。   

1.  **az12001a-vm0 - ディスク** ブレードから、az12001a-vm0 に次の設定を持つデータ ディスクを接続します。

    -   LUN: **0**

    -   ディスク名: **az12001a-vm0-DataDisk0**

    -   リソース グループ：**az12001a-RG**

    -   ホスト キャッシュ: **なし**

1.  前の手順を繰り返して、プレフィックス **az12001a-vm0-DataDisk** を使用して残りの 7 つのディスク (合計 8 つ) を接続します。ディスク名の最後の文字と一致する LUN 番号を割り当てます。

1.  変更内容を保存します｡ 

1.  Azure portal で、前のタスク (**az12001a-vm1**) でプロビジョニングした 2 番目の Azure VM のブレードに移動します。

1.  **az12001a-vm1** ブレードから、**az12001a-vm1 - ディスク** ブレードに移動します。

1.  **az12001a-vm1 - ディスク** ブレードから、az12001a-vm1 に次の設定を持つデータ ディスクを接続します。

    -   LUN: **0**

    -   ディスク名: **az12001a-vm1-DataDisk0**

    -   リソース グループ：**az12001a-RG**

    -   ホスト キャッシュ: **読み取り専用**

1.  前の手順を繰り返して、プレフィックス **az12001a-vm1-DataDisk** を使用して残りの 7 つのディスク (合計 8 つ) を接続します。ディスク名の最後の文字と一致する LUN 番号を割り当てます。LUN **1** を使用するディスクのホスト キャッシュを **読み取り専用** に設定し、残りのすべてのディスクのホスト キャッシュを **なし** に設定します。     

1.  変更内容を保存します｡ 

> **結果**: このエクササイズを完了すると、可用性の高い SAP HANA のデプロイをサポートするために必要となる Azure コンピューティング リソースのプロビジョニングを行うことができます。


## エクササイズ 2: 可用性の高い SAP HANA のインストールをサポートするために、Linux を実行している Azure VM のオペレーティング システムを構成する

時間: 30 分間

このエクササイズでは、Sap HANA のクラスタ化されたインストールに対応するように、SUSE Linux Enterprise Server を実行している Azure VM 上のオペレーティング システムとストレージを構成します。

### タスク 1: Azure Linux VM に接続する

1.  Azure portal の Cloud Shell で Bash セッションを開始します。 

1.  [Cloud Shell] ウィンドウで、次のコマンドを実行して、前のタスクでデプロイした最初の Azure VM のパブリック IP アドレスを特定します。

```
RESOURCE_GROUP_NAME='az12001a-RG'

PIP=$(az network public-ip show --resource-group $RESOURCE_GROUP_NAME --name az12001a-vm0-ip --query ipAddress --output tsv)
```

1.  [Cloud Shell] ウィンドウで次のコマンドを実行して、前の手順で特定した IP アドレスに SSH セッションを確立します。

```
ssh student@$PIP
```

1.  接続を続行するかどうかを確認するメッセージが表示されたら、`はい`と入力し、**Enter** キーを押します。  

1.  パスワードの入力を求めるメッセージが表示されたら、`Pa55w.rd1234`と入力し、**Enter** キーを押します。  

1.  [Cloud Shell] ツールバーの [**新しいセッションを開く**] アイコンをクリックして、別の Cloud Shell Bash セッションを開きます。

1.  新しく開いた Cloud Shell Bash セッションで、このタスクのすべての手順を繰り返して、IP アドレス **az12001a-vm0-ip** を介して **az12001a-vm1** Azure VM に接続します。


### タスク 2: Linux を実行している Azure VM のストレージを構成する

1.  [Cloud Shell] ウィンドウの SSH セッションで az12001a-vm0 に対して、次のコマンドを実行して特権を昇格させ、プロンプトが表示されたら、学生ユーザー アカウントのパスワードを入力します。 

```
sudo su -
```

1.  パスワードの入力を求めるメッセージが表示されたら、`Pa55w.rd1234`と入力し、**Enter** キーを押します。 

1.  [Cloud Shell] ウィンドウの az12001a-vm0 への SSH セッションで、`lsscsi` を実行して、新しく接続されたデバイスとその LUN 番号の間のマッピングを特定します。
    
```
lsscsi
```

1.  [Cloud Shell] ウィンドウの az12001a-vm0 への SSH セッションで、次を実行して (8 つのうち) 6つのデータ ディスクの物理ボリュームを作成します。
    
```
pvcreate /dev/sdc
pvcreate /dev/sdd
pvcreate /dev/sde
pvcreate /dev/sdf
pvcreate /dev/sdg
pvcreate /dev/sdh
```

1.  [Cloud Shell] ウィンドウの az12001a-vm0 への SSH セッションで、次を実行してボリューム グループを作成します。
    
```
vgcreate vg_hana_data /dev/sdc /dev/sdd
vgcreate vg_hana_log /dev/sde /dev/sdf
vgcreate vg_hana_backup /dev/sdg /dev/sdh
```

1.  [Cloud Shell] ウィンドウの az12001a-vm0 への SSH セッションで、次を実行して論理ボリュームを作成します。

```
lvcreate -l 100%FREE -n hana_data vg_hana_data
lvcreate -l 100%FREE -n hana_log vg_hana_log
lvcreate -l 100%FREE -n hana_backup vg_hana_backup
```

    > **注記**: 各ボリューム グループに 1 つの論理ボリュームを作成しています

1.  [Cloud Shell] ウィンドウの az12001a-vm0 への SSH セッションで、次を実行して論理ボリュームをフォーマットします。

```
mkfs.xfs /dev/vg_hana_data/hana_data
mkfs.xfs /dev/vg_hana_log/hana_log
mkfs.xfs /dev/vg_hana_backup/hana_backup
```

1.  [Cloud Shell] ウィンドウの az12001a-vm0 への SSH セッションで、次を実行して **/dev/sdi** ディスクをパーティション化します。

```
fdisk /dev/sdi
```

1.  プロンプトが表示されたら、順番に `n`、`p`、`1` (毎回 **Enter**  キーを押す) と入力し **Enter** キーを 2 回押します。次に`w`と入力して書き込みを完了します。

1.  [Cloud Shell] ウィンドウの az12001a-vm0 への SSH セッションで、次を実行して **/dev/sdj** ディスクをパーティション化します。

```
fdisk /dev/sdj
```

1.  プロンプトが表示されたら、順番に `n`、`p`、`1` (毎回 **Enter**  キーを押す) と入力し **Enter** キーを 2 回押します。次に`w`と入力して書き込みを完了します。

1.  [Cloud Shell] ウィンドウの az12001a-vm0 への SSH セッションで、次を実行して新しく作成したパーティションをフォーマットします。

```
mkfs.xfs /dev/sdi -f
mkfs.xfs /dev/sdj -f
```

1.  [Cloud Shell] ウィンドウの az12001a-vm0 への SSH セッションで、次を実行してマウント ポイントの役割を果たすディレクトリを作成します。

```
mkdir -p /hana/data
mkdir -p /hana/log
mkdir -p /hana/backup
mkdir -p /hana/shared
mkdir -p /usr/sap
```

1.  [Cloud Shell] ウィンドウの az12001a-vm0 への SSH セッションで、次を実行して論理ボリュームの ID を表示します。

```
blkid
```

    > **注記**:  **dev/sdi** (**/hana/shared** に使用) と **/dev/sdj** (**/usr/sap** に使用) を含む、新しく作成したボリューム グループおよびパーティションに関連付けられた **UUID** 値を特定します。


1.  [Cloud Shell] ウィンドウの az12001a-vm0 への SSH セッションで、次を実行して vi エディターで  **/etc/fstab** を開きます (他のエディターを自由に使用できます)。 

```
vi /etc/fstab
```

1.  エディターで、次のエントリを **/etc/fstab** に追加します (ここで 、`\<UUID of /dev/vg\_hana\_data-hana\_data\>`、`\<UUID of /dev/vg\_hana\_log-hana\_log\>`、`\<UUID of /dev/vg\_hana\_backup-hana\_backup\>`、`\<UUID of /dev/vg_hana_shared-hana_shared (/dev/sdi)\>`、`\<UUID of /dev/vg_usr_sap-usr_sap (/dev/sdj)\>` は、前の手順で特定した ID を表します) 

```
/dev/disk/by-uuid/<UUID of /dev/vg_hana_data-hana_data> /hana/data xfs  defaults,nofail  0  2
/dev/disk/by-uuid/<UUID of /dev/vg_hana_log-hana_log> /hana/log xfs  defaults,nofail  0  2
/dev/disk/by-uuid/<UUID of /dev/vg_hana_backup-hana_backup> /hana/backup xfs  defaults,nofail  0  2
/dev/disk/by-uuid/<UUID of /dev/vg_hana_shared-hana_shared (/dev/sdi)> /hana/shared xfs  defaults,nofail  0  2
/dev/disk/by-uuid/<UUID of /dev/vg_usr_sap-usr_sap (/dev/sdj)> /usr/sap xfs  defaults,nofail  0  2
```

1.  変更を保存し、エディターを閉じます。

1.  [Cloud Shell] ウィンドウの az12001a-vm0 への SSH セッションで、次を実行して新しいボリュームをマウントします。

```
mount -a
```

1.  [Cloud Shell] ウィンドウの az12001a-vm0 への SSH セッションで、次を実行してマウントが行われたことを確認します。

```
df -h
```

1. az12001a-vm1 への Cloud Shell Bash セッションに切り替え、このタスクのすべての手順を繰り返して、**az12001a-vm1** でストレージを構成します。


### タスク 3: クロスノード パスワードレス SSH アクセスを有効にする

1.  [Cloud Shell] ウィンドウの az12001a-vm0 への SSH セッションで、次を実行してパスフレーズレス SSH キーを生成します。

```
ssh-keygen -tdsa
```

1.  プロンプトが表示されたら、**Enter** を 3 回押してから、次を実行して公開キーを表示します。 

```
cat /root/.ssh/id_dsa.pub
```

1.  キーの値をクリップボードにコピーします。

1.  SSH セッションを含む [Cloud Shell] ウィンドウの **az12001a-vm1** に切り替え、次を実行してディレクトリ **/root/.ssh/** を作成します。

```
mkdir /root/.ssh
```

1.  [Cloud Shell] ウィンドウの az12001a-vm1 への SSH セッションで、次を実行して vi エディターでファイル  **/root/.ssh/authorized\_keys** を作成します (他のエディターを自由に使用できます)。

```
vi /root/.ssh/authorized_keys
```

1.  エディター ウィンドウで、az12001a-vm0 で生成したキーを貼り付けます。

1.  変更を保存し、エディターを閉じます。

1.  [Cloud Shell] ウィンドウの az12001a-vm1 への SSH セッションで、次を実行してパスフレーズレス SSH キーを生成します。

```
ssh-keygen -tdsa
```

1.  プロンプトが表示されたら、**Enter** を 3 回押してから、次を実行して公開キーを表示します。 

```
cat /root/.ssh/id_dsa.pub
```

1.  キーの値をクリップボードにコピーします。

1.  SSH セッションを含む [Cloud Shell] ウィンドウの az12001a-vm0 に切り替え、次を実行して vi エディターでファイル **/root/.ssh/authorized\_keys** を作成します (他のエディターを自由に使用できます)。

```
vi /root/.ssh/authorized_keys
```

1.  エディター ウィンドウで、az12001a-vm1 で生成したキーを貼り付けます。

1.  変更を保存し、エディターを閉じます。

1.  [Cloud Shell] ウィンドウの az12001a-vm0 への SSH セッションで、次を実行してパスフレーズレス SSH キーを生成します。

```
ssh-keygen -t rsa
```

1.  プロンプトが表示されたら、**Enter** を 3 回押してから、次を実行して公開キーを表示します。 

```
cat /root/.ssh/id_rsa.pub
```

1.  キーの値をクリップボードにコピーします。

1.  SSH セッションを含む [Cloud Shell] ウィンドウの **az12001a-vm1** に切り替え、次を実行して vi エディターでファイル **/root/.ssh/authorized\_keys** を開きます (他のエディターを自由に使用できます)。

```
vi /root/.ssh/authorized_keys
```

1.  エディター ウィンドウで、新しい行から開始し、az12001a-vm0 で生成したキーを貼り付けます。

1.  変更を保存し、エディターを閉じます。

1.  [Cloud Shell] ウィンドウの az12001a-vm1 への SSH セッションで、次を実行してパスフレーズレス SSH キーを生成します。

```
ssh-keygen -t rsa
```

1.  プロンプトが表示されたら、**Enter** を 3 回押してから、次を実行して公開キーを表示します。 

```
cat /root/.ssh/id_rsa.pub
```

1.  キーの値をクリップボードにコピーします。

1.  SSH セッションを含む [Cloud Shell] ウィンドウの az12001a-vm0 に切り替え、次を実行して vi エディターでファイル **/root/.ssh/authorized\_keys** を開きます (他のエディターを自由に使用できます)。

```
vi /root/.ssh/authorized_keys
```

1.  エディター ウィンドウで、新しい行から開始し、az12001a-vm1 で生成したキーを貼り付けます。

1.  変更を保存し、エディターを閉じます。

1.  [Cloud Shell] ウィンドウの az12001a-vm0 への SSH セッションで、次を実行して vi エディターでファイル  **/etc/ssh/sshd\_config** を開きます (他のエディターを自由に使用できます)。

```
vi /etc/ssh/sshd_config
```

1.  **/etc/ssh/sshd\_config** ファイルで、 **PermitRootLogin** エントリと **AuthorizedKeysFile** エントリを検索し、次のように構成します(必要に応じて、先頭の **#** 文字を削除します。。
```
PermitRootLogin yes
AuthorizedKeysFile      /root/.ssh/authorized_keys
```

1.  変更を保存し、エディターを閉じます。

1.  [Cloud Shell] ウィンドウの az12001a-vm0 への SSH セッションで、次を実行して sshd デーモンを再起動します。

```
systemctl restart sshd
```

1.  az12001a-vm1 で前の 4 つの手順を繰り返します。

1.  構成が成功したことを確認するには、 [Cloud Shell] ウィンドウの SSH セッションで az12001a-vm0 に対して、次を実行して az12001a-vm0 から az12001a-vm1 への SSH セッションを **root** として確立します。 

```
ssh root@az12001a-vm1
```

1.  接続を続行するかどうかを確認するメッセージが表示されたら、「はい」 と入力し、**Enter** キーを押します。 

1.  パスワードの入力が求められないことを確認します。

1.  次のコマンドを実行して、az12001a-vm0 から az12001a-vm1 までの SSH セッションを閉じます。 

```
exit
```

1.  次を 2 回実行して、az12001a-vm0 からサインアウトします。

```
exit
```

1.  構成が成功したことを確認するには、 [Cloud Shell] ウィンドウの SSH セッションで az12001a-vm1 に対して、次を実行して az12001a-vm1 から az12001a-vm0 への SSH セッションを **root** として確立します。

```
ssh root@az12001a-vm0
```

1.  接続を続行するかどうかを確認するメッセージが表示されたら、「はい」 と入力し、**Enter** キーを押します。 

1.  パスワードの入力が求められないことを確認します。

1.  次のコマンドを実行して、az12001a-vm1 から az12001a-vm0 までの SSH セッションを閉じます。 

```
exit
```

1.  次を 2 回実行して、az12001a-vm1 からサインアウトします。

```
exit
```

> **結果**: このエクササイズを完了すると、可用性の高い SAP HANA インストールをサポートするように、Linux を実行している Azure VM のオペレーティング システムを構成することができます。


## エクササイズ 3: 可用性の高い SAP HANA のデプロイをサポートするために必要となる Azure ネットワーク リソースのプロビジョニングを行う

時間: 30 分間

このエクササイズでは、SAP HANA のクラスター化されたインストールに対応するために Azure Load Balancers を実装します。


### タスク 1: 負荷分散のセットアップが容易に行えるよう Azure VM を構成する。

   > **注記**: Stardard SKU の Azure Load Balancer のペアを設定するので、まず、負荷分散バックエンド プールとして機能する 2 つの Azure VM のネットワーク アダプターに関連付けられたパブリック IP アドレスを削除する必要があります。

1.  Azure portal で、**az12001a-vm0** Azure VM のブレードに移動します。 

1.  **az12001a-vm0** ブレードから、ネットワーク アダプターに関連付けられた **az12001a-vm0-ip** という名前のパブリック IP アドレスのブレードに移動します。

1.  **az12001a-vm0-ip** ブレードで、まずパブリック IP アドレスをネットワーク インターフェイスから切り離してから削除します。 

1.  Azure portal で、**az12001a-vm1** Azure VM のブレードに移動します。 

1.  **az12001a-vm1** ブレードから、ネットワーク アダプターに関連付けられた **az12001a-vm1-ip** という名前のパブリック IP アドレスのブレードに移動します。

1.  **az12001a-vm1-ip** ブレードで、まずパブリック IP アドレスをネットワーク インターフェイスから切り離してから削除します。

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

1.  Azure portal で、[**+ リソースの作成 **] をクリックします。

1.  [**新規**] ブレードで、次の設定を使用して新しい Azure Load Balancer の作成を開始します。 

    -   サブスクリプション: Azure サブスクリプションの名前。

    -   リソース グループ：**az12001a-RG**

    -   名前: **az12001a-lb0**

    -   リージョン: *このラボの最初のエクササイズで Azure VM をデプロイしたのと同じ Azure リージョン*

    -   タイプ: **内部**

    -   SKU: **Standard**

    -   仮想ネットワーク: **az12001a-RG-vnet**

    -   サブネット: **subnet-0**

    -   IP アドレスの割り当て: **静的**

    -   IP アドレス: **192.168.0.240**

    -   可用性ゾーン: **ゾーン冗長**

1.  Load Balancer のプロビジョニングが終了するまで待ち、Azure portal のブレードに移動します。

1.  **az12001a-lb0** ブレードで、次の設定を使用してバックエンド プールを追加します。

    -   名前: **az12001a-lb0-bepool**

    -   仮想ネットワーク: **az12001a-rg-vnet**

    -   仮想マシン: **az12001a-vm0**  IP アドレス: **ipconfig1**

    -   仮想マシン: **az12001a-vm1**  IP アドレス: **ipconfig1**

1.  **az12001a-lb0** ブレードから、次の設定を使用して正常性プローブを追加します。

    -   名前: **az12001a-lb0-hprobe**

    -   プロトコル: **TCP**

    -   ポート: **62500**

    -   間隔: **5** *秒*

    -   異常しきい値: **2** *つの連続エラー *

1.  **az12001a-lb0** ブレードから、次の設定を使用してネットワーク負荷分散ルールを追加します。

    -   名前: **az12001a-lb0-lbruleAll**

    -   IP バージョン: **IPv4**

    -   フロントエンド IP アドレス: **192.168.0.240 (LoadBalancerFrontEnd)**

    -   HA ポート: **有効**

    -   バックエンド プール: **az12001a-lb0-bepool (2 台の仮想マシン)**

    -   正常性プローブ: **az12001a-lb0-hprobe (TCP:62504)**

    -   セッション永続化: **なし**

    -   アイドル タイムアウト (分): **4**

    -   フローティング IP (ダイレクト サーバー リターン): **有効**

### タスク 3: 送信トラフィックを処理する Azure Load Balancers を作成して構成する

1.  Azure portal の Cloud Shell で Bash セッションを開始します。 

1.  [Cloud Shell] ウィンドウで次のコマンドを実行して、2 番目の Load Balancer で使用するパブリック IP アドレスを作成します。

```
RESOURCE_GROUP_NAME='az12001a-RG'

LOCATION=$(az group list --query "[?name == '$RESOURCE_GROUP_NAME'].location" --output tsv)

PIP_NAME='az12001a-lb1-pip'

az network public-ip create --resource-group $RESOURCE_GROUP_NAME --name $PIP_NAME --sku Standard --location $LOCATION
```

1.  [Cloud Shell] ウィンドウで次のコマンドを実行して、2 番目の Load Balancer を作成します。

```
LB_NAME='az12001a-lb1'

LB_BE_POOL_NAME='az12001a-lb1-bepool'

LB_FE_IP_NAME='az12001a-lb1-fe'

az network lb create --resource-group $RESOURCE_GROUP_NAME --name $LB_NAME --sku Standard --backend-pool-name $LB_BE_POOL_NAME --frontend-ip-name $LB_FE_IP_NAME --location $LOCATION --public-ip-address $PIP_NAME
```

1.  [Cloud Shell] ウィンドウで次のコマンドを実行して、2 番目の Load Balancer のアウトバウンド規則を作成します。

```
LB_RULE_OUTBOUND='az12001a-lb1-ruleoutbound'

az network lb outbound-rule create --resource-group $RESOURCE_GROUP_NAME --lb-name $LB_NAME --name $LB_RULE_OUTBOUND --frontend-ip-configs $LB_FE_IP_NAME --protocol All --idle-timeout 4 --outbound-ports 1000 --address-pool $LB_BE_POOL_NAME
```

1.  [Cloud Shell] ウィンドウを閉じます。

1.  Azure portal で、**az12001a-lb1** という名前の Azure Load Balancer のプロパティを表示するブレードに移動します。

1.  **az12001a-lb1** ブレードで、[**バックエンド プール**] をクリックします。

1.  **az12001a-lb1 - バックエンド プール** ブレードで、**az12001a-lb1-bepool** をクリックします。

1.  **az12001a-lb1-bepool** ブレードで、次の設定を指定し、[**保存**] をクリックします。

    -   仮想ネットワーク: **az12001a-rg-vnet (2 VM)**

    -   仮想マシン: **az12001a-vm0**  IP アドレス: **ipconfig1**

    -   仮想マシン: **az12001a-vm1**  IP アドレス: **ipconfig1**

### タスク 4: ジャンプ ホストをデプロイする

   > **注記**: 2 つのクラスター化された Azure VM にはインターネットから直接アクセスできないため、ジャンプ ホストとして機能する Windows Server 2019 Datacenter を実行する Azure VM をデプロイします。 

1.  ラボ コンピューターの Azure portal で、[**+ リソースの作成**] をクリックします。

1.  [**新規**] ブレードから、**Windows Server 2019 Datacenter** イメージに基づいて新規 Azure VM の作成を開始します。

1.  次の設定を使用して Azure VM をプロビジョニングします。

    -   サブスクリプション: *Azure サブスクリプションの名前*。

    -   リソース グループ： **az12001a-RG**

    -   仮想マシン名： **az12001a-vm2**

    -   リージョン: *このラボの最初のエクササイズで Azure VM をデプロイしたのと同じ Azure リージョン*

    -   可用性オプション: **インフラストラクチャ冗長は必要ありません**

    -   画像: **Windows Server 2019 Datacenter**

    -   サイズ: **Standard DS1 v2**

    -   ユーザー名: **受講者**

    -   パスワード: **Pa55w.rd1234**

    -   パブリック受信ポート: **選択したポートを許可する**

    -   選択した受信ポート: **RDP (3389)**

    -   Windows ライセンスを既にお持ちの場合: **いいえ**

    -   OS ディスク タイプ: **Standard HDD**

    -   仮想ネットワーク: **az12001a-RG-vnet**

    -   サブネット名: **subnet-0 (192.168.0.0/24)**

    -   パブリック IP アドレス: **az12001a-vm2-ip** *という名前の新規 IP アドレス* 

    -   NIC ネットワーク セキュリティ グループ: **Basic**

    -   パブリック受信ポート: **選択したポートを許可する**

    -   受信ポートを選択**RDP (3389)**

    -   高速ネットワーク: **オフ**

    -   この仮想マシンを既存の負荷分散ソリューションの後ろに配置しますか: **いいえ**

    -   無料の Basic プランを有効にする: **いいえ**

    -   ブート診断: **オフ**

    -   OS のゲスト診断: **オフ**

    -   システム割り当てマネージド ID**オフ**

    -   AAD 資格情報を使用してログインする (プレビュー): **オフ**

    -   自動シャットダウンを有効にする: **オフ**

    -   バックアップを有効にする: **オフ**

    -   拡張機能: *なし*

    -   タグ: **なし**

1.  プロビジョニングが完了するまで待ちます。通常、これには数分かかります。

1.  RDP 経由で新しくプロビジョニングされた Azure VM に接続します。 

1.  az12001a-vm2 への RDP セッションで、[**https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html**](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) から PuTTY をダウンロードします。 

1.  プライベート IP アドレス (それぞれ 192.168.0.4 および 192.168.0.5) を介して az12001a-vm0 と az12001a-vm1 の両方に SSH セッションを確立できることを確認します。

> **結果**: このエクササイズを完了すると、可用性の高い SAP HANA のデプロイをサポートするために必要となる Azure ネットワーク リソースのプロビジョニングを行うことができます。


## エクササイズ 4: ラボリソースを削除する

時間: 10 分間

このエクササイズでは、このラボでプロビジョニングしたリソースすべてを削除します。

#### タスク 1: Cloud Shell を開く

1. ポータルの上部にある [**Cloud Shell**] アイコンをクリックして Cloud Shell ウィンドウを開き、シェルとして Bash を選択します。

1. ポータルの下部にある **Cloud Shell** コマンド プロンプトで、次のコマンドを入力し、[**Enter**] を押して、このラボで作成したすべてのリソース グループを一覧表示します。

```
az group list --query "[?starts_with(name,'az12001a-')]".name --output tsv
```

1. このラボで作成したリソース グループのみが出力に含まれていることを確認します。これらのグループは、次のタスクで削除されます。

#### タスク 2: リソース グループの削除

1. **Cloud Shell** コマンド プロンプトで、次のコマンドを入力し、[**Enter**] キーを押して、このラボで作成したリソース グループを削除します。

```
az group list --query "[?starts_with(name,'az12001a-')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
```

1. ポータルの下部にある **Cloud Shell** プロンプトを閉じます。


> **結果**: このエクササイズを完了すると、このラボで使用したリソースが削除されます。
