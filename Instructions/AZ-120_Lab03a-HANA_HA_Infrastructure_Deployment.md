# AZ 120 モジュール 3: SAP on Azure の実装
# ラボ 3a: Linux を実行する Azure VM に SAP アーキテクチャを実装する

予想時間：120 分

このラボのタスクすべては、Azure portal (Bash Cloud Shell セッションを含む) から実行されます  

   > **注記**: Cloud Shell を使用しない場合、ラボの仮想マシンには Azure CLI がインストールされている必要があります [**https://docs.microsoft.com/ja-jp/cli/azure/install-azure-cli-windows?view=azure-cli-latest**](https://docs.microsoft.com/ja-jp/cli/azure/install-azure-cli-windows?view=azure-cli-latest)。

ラボ ファイル: なし

## シナリオ
  
Adatum 社は、Azure に SAP NetWeaver をデプロイする準備段階として、Linux 上の SUSE ディストリビューションを実行している Azure VM 上の SAP NetWeaver の高可用性の実装を例証するデモを行いたいと考えています。

## Objectives
  
このラボを終了すると、下記ができるようになります。

-   高可用性な SAP NetWeaver のデプロイをサポートするために必要な Azure リソースをプロビジョニングする

-   高可用性な SAP NetWeaver のデプロイをサポートするように、Linux を実行している Azure VM の OS を構成する

-   高可用性な SAP NetWeaver のデプロイをサポートするように、Linux を実行している Azure VM でクラスタリングを構成する

## 必要条件

-   Availability Zones をサポートする Azure リージョンで十分な数の使用可能な DSv3 vCPU (2 x 4) と DSv2 (1 x 1) vCPU を持つ Microsoft Azure サブスクリプション

-   Azure にアクセス可能な Windows 10、Windows Server 2019、または Windows Server 2016 を実行しているラボ コンピューター


## 演習 1: 高可用性な SAP NetWeaver のデプロイをサポートするために必要な Azure リソースのプロビジョニング

時間: 30 分

このエクササイズでは、Linux クラスタリングの構成に必要な Azure インフラストラクチャ コンピューティング コンポーネントをデプロイします。これには、同じ可用性セットで Linux SUSE を実行する Azure VM のペアを作成する必要があります。

### タスク 1: 高可用性な SAP NetWeaver のデプロイをホストする仮想ネットワークを作成します。

1.  ラボ コンピューターから Web ブラウザーを起動し、https://portal.azure.com から Azure portal に移動します。

1.  入力画面が表示されたら、このラボで使用する Azure サブスクリプションの所有者または共同作成者のロールを持つ、職場、学校または個人の Microsoft アカウントを使用してログインします。サブスクリプションには Azure AD テナントのグローバル管理者ロールが関連づけられている必要があります。

1.  Azure portal 上の Cloud Shell で Bash セッションを開始します。 

    > **注記**: 現在の Azure サブスクリプションで Cloud Shell を初めて起動する場合は、Azure ファイル共有を作成して Cloud Shell ファイルを永続化するように求められます。その場合は、既定値に設定すると、自動的に生成されたリソース グループ内にストレージ アカウントが作成されます。

1.  「Cloud Shell」 ウィンドウで、次のコマンドを実行して、Availability Zones をサポートする Azure リージョンと、このラボのリソースを作成する場所を指定します (`<region>` を Azure リージョンの名前に置き換えます)。

    ```
    LOCATION='<region>'
    ```

1.  Cloud Shell ペインで次のコマンドを実行して、指定したリージョンにリソース グループを作成します。

    ```
    RESOURCE_GROUP_NAME='az12003a-sap-RG'

    az group create --resource-group $RESOURCE_GROUP_NAME --location $LOCATION
    ```

1.  Cloud Shell ペインで次のコマンドを実行して、作成したリソース グループ内に 1 つのサブネットを有する仮想ネットワークを作成します。

    ```
    VNET_NAME='az12003a-sap-vnet'

    VNET_PREFIX='10.3.0.0/16'

    SUBNET_NAME='sapSubnet'

    SUBNET_PREFIX='10.3.0.0/24'

    az network vnet create --resource-group $RESOURCE_GROUP_NAME --location $LOCATION --name $VNET_NAME --address-prefixes $VNET_PREFIX --subnet-name $SUBNET_NAME --subnet-prefixes $SUBNET_PREFIX
    ```

1.  Cloud Shell ペインで次のコマンドを実行して、新しく作成した仮想ネットワークにあるサブネットのリソース ID を特定します。

    ```
    az network vnet subnet list --resource-group $RESOURCE_GROUP_NAME --vnet-name $VNET_NAME --query "[?name == '$SUBNET_NAME'].id" --output tsv
    ```

1.  結果の値をクリップボードにコピーします。これは、次のタスクで必要になります。

### タスク 2: 高可用性 SAP NetWeaver のデプロイをホストする Linux SUSE が稼働する Azure VM をプロビジョニングするために Azure Resource Manager テンプレートをデプロイする

1.  ラボ コンピューターでブラウザを起動し、[**https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image-md**](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image-md)を参照します。

    > **注記**: Microsoft Edge またはサード パーティのブラウザーを使用してください。Internet Explorer は使用しないでください。

1.  **SAP NetWeaver 3-tier compatible template using a Marketplace image - MD** (Marketplace イメージを使った SAP NetWeaver 3 層構成互換テンプレート - MD) というタイトルのページで、「**Azure にデプロイ**」 をクリックします。これにより、Azure portal に自動的にリダイレクトされ、**SAP NetWeaver 3-tier (managed disk)** ブレードが表示されます。

1.  **SAP NetWeaver 3-tier (managed disk)** ブレードで、以下の設定を使用してデプロイを開始します。

    -   サブスクリプション: *Azure サブスクリプションの名前*。

    -   リソース グループ: **az12003a-sap-RG**

    -   場所: *このエクササイズの最初のタスクで指定したものと同じ Azure リージョンを選択します。*

    -   SAP システム ID: **I20**

    -   スタック タイプ: **ABAP**

    -   OS タイプ: **SLES 12**

    -   Dbtype: **HANA**

    -   Sap システム サイズ: **デモ**

    -   システムの可用性: **HA**

    -   管理者ユーザー名: **student**

    -   認証タイプ: **パスワード**

    -   管理者パスワードまたはキー: **Pa55w.rd1234**

    -   サブネット ID: *前のタスクでクリップボードにコピーした値*

    -   Availability Zones: **1,2**

    -   場所: **[resourceGroup().location]**

    -   _artifacts の保存先: **https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/sap-3-tier-marketplace-image-md/**

    -   _artifacts の保存先の Sas トークン: *空白のままにする*

1.  デプロイが完了するのを待たずに、次のタスクに進みます。 

    > **注記**: CustomScriptExtension コンポーネントのデプロイ中に**競合**エラー メッセージが表示され、デプロイが失敗した場合は、次の手順を使用してこの問題を修復します。

       - Azure portal の 「**デプロイ**」 ブレードで、デプロイの詳細を確認し、CustomScriptExtension のインストールが失敗した VM を特定します
       - Azure portal で、前の手順で特定した VM のブレードに移動し、「**拡張機能**」を選択し 、「**拡張機能**」 ブレードから CustomScript 拡張機能を削除します
       - Azure portall で、**az12003a-sap-RG** リソース グループ ブレードに移動し、「**デプロイ**」 を選択し、失敗したデプロイへのリンクを選択して 「**再デプロイ**」を選択します。ターゲット リソース グループ (**az12003a-sap-RG** ) を選択して、ルート アカウントのパスワードを指定します (**Pa55w.rd1234** )。


### タスク 3: ジャンプ ホストをデプロイする

   > **注記**: 前のタスクでデプロイした Azure VM はインターネットからアクセスできないため、ジャンプ ホストとして機能する Windows Server 2019 Datacenter を実行する Azure VM をデプロイします。 

1.  ラボ コンピューターの Azure portal で、「**+ リソースの作成**」 をクリックします。

1.  「**新規**」 ブレードから、**Windows Server 2019 Datacenter** イメージを基に新規 Azure VM の作成を開始します。

1.  次の設定を使用して Azure VM をプロビジョニングします。

    -   サブスクリプション: *Azure サブスクリプションの名前*。

    -   リソース グループ： *新しいリソース グループの名前* **az12003a-dmz-RG**

    -   仮想マシン名: **az12003a-vm0**

    -   リージョン: *このエクササイズの最初のタスクで Azure VM をデプロイしたのと同じ Azure リージョン*

    -   可用性オプション: **インフラストラクチャ冗長は必要ありません**

    -   画像: **Windows Server 2019 Datacenter**

    -   サイズ: **スタンダード D2s v3**

    -   ユーザー名: **Student**

    -   パスワード: **Pa55w.rd1234**

    -   パブリック受信ポート: **Allow selected ports (選択したポートを許可する)**

    -   インバウンド ポートを選択: **RDP (3389)**

    -   Windows ライセンスを持っていますか? **いいえ**

    -   OS ディスク タイプ: **標準 HDD**

    -   仮想ネットワーク: **az12003a-sap-vnet**

    -   サブネット: **bastionSubnet(10.3.255.0/24)** *という名前の新しいサブネット*

    -   パブリック IP: **az12003a-vm0-ip** *という名前の新規 IP アドレス*

    -   NIC ネットワーク セキュリティ グループ: **Basic**

    -   パブリック受信ポート: **Allow selected ports (選択したポートを許可する)**

    -   インバウンド ポートを選択: **RDP (3389)**

    -   高速ネットワーキング: **オフ**

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

> **結果**: この演習を完了することにより、高可用性 SAP NetWeaver のデプロイをサポートするために必要となる Azure リソースのプロビジョニングを行いました。


## 演習 2: 可用性の高い SAP NetWeaver デプロイをサポートするように Linux を実行している Azure VM でクラスタリングを構成する

時間: 30 分

このエクササイズでは、可用性の高い SAP NetWeaver に対応するように、SUSE Linux Enterprise Server を実行している Azure VM 上のオペレーティングシステムを構成します。

### タスク 1: データベース層 Azure VM のネットワークを構成します。

   > **注記**: このタスクを開始する前に、前のエクササイズで開始したテンプレートのデプロイが正常に完了したことを確認します。 

1.  ラボ コンピューターの Azure portal で、**i20-db-0** Azure VM のブレードに移動します。

1.  **i20-db-0** ブレードから、**ネットワーク** ブレードに移動します。 

1.  **i20-vm0 - ネットワーク** ブレードから、i20-vm0 のネットワーク インターフェイスに移動します。 

1.  i20-db-0 のネットワーク インターフェイスのブレードから、その IP 構成ブレードに移動し、そこから **ipconfig1** ブレードを表示します。

1.  **ipconfig1** ブレードで、プライベート IP アドレスを **10.3.0.20** に設定し、割り当てを **静的** に変更して変更を保存します。

1.  Azure portal で、**i20-db-1** Azure VM ブレードに移動します。

1.  **i20-db-1** ブレードから、**ネットワーク** ブレードに移動します。 

1.  **i20-db-1 - ネットワーク** ブレードから、i20-db-1 のネットワーク インターフェイスに移動します。 

1.  i20-db-1 のネットワーク インターフェイスのブレードから、その IP 構成ブレードに移動し、そこから **ipconfig1** ブレードを表示します。

1.  **ipconfig1** ブレードで、プライベート IP アドレスを **10.3.0.21** に設定し、割り当てを**静的** に変更して保存します。


### タスク 2: Azure VM のデータベース層に接続します。

1.  ラボ コンピューターの Azure portal で、**az12003a-vm0** ブレードに移動します。

1.  **az12003a-vm0**ブレードから、リモート デスクトップ経由で Azure VM az12003a-vm0 に接続します。 

1.  az12003a-vm0 への RDP セッションのサーバー マネージャーで 「**ローカル サーバー**」 ビューに移動し、「**IE セキュリティ強化の構成**」 をオフにします。

1.  az12003a-vm0 への RDP セッションで、[**https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html**](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) から PuTTY をダウンロードしてインストールします。

1.  PuTTY を使用して、SSH 経由で **i20-db-0** Azure VM に接続します。セキュリティ警告を確認し、プロンプトが表示されたら、次の認証情報を入力します。

    -   ログイン: **student**

    -   パスワード: **Pa55w.rd1234**

1.  PuTTY を使用して、同じ認証情報を使って SSH 経由で **i20 db-1** Azure VM に接続します。


### タスク 3: データベース層 Azure VM のストレージ構成を確認する。

1.  i20-db-0 Azure VM への PuTTY SSH セッション内で、次のコマンドを実行して権限を昇格させます。 

     ```
     sudo su -
     ```

1.  パスワードの入力を求めるメッセージが表示されたら、**Pa55w.rd1234**と入力し、**Enter** キーを押します。 

1.  次のコマンドをが実行して、SSH セッション～i20-db-0 内で、すべての SAP HANA 関連ボリューム (**/usr/sap**、**/hana/shared**、**/hana/backup**、**/hana/data** および **/hana/logs** を含む) が適切にマウントされていることを確認します。

    ```
    df -h
    ```

1.  i20-db-1 Azure VM で前の手順を繰り返します。


### タスク 4: クロスノード パスワードレス SSH アクセスを有効にする

1.  i20-db-0 への SSH セッションで、次を実行して、vi エディターでファイル **/etc/ssh/sshd\_config** を開きます (または、他のエディターを使用することもできます)。

    ```
    vi /etc/ssh/sshd_config
    ```

1.  **/etc/ssh/sshd\_config** ファイルで、 **PermitRootLogin** エントリと **AuthorizedKeysFile** エントリを検索し、次のように構成します。
    ```
    PermitRootLogin yes
    AuthorizedKeysFile      /root/.ssh/authorized_keys
    ```

1.  変更を保存し、エディタを閉じます。

1.  i20-db-0 への SSH セッションで、次を実行して sshd デーモンを再起動します。

    ```
    systemctl restart sshd
    ```

1.  i20-db-1 Azure VM で前の 4 つの手順を繰り返します。

1.  i20-db-0 への SSH セッションで、次を実行してパスフレーズレス SSH キーを生成します。

    ```
    ssh-keygen -tdsa
    ```

1.  プロンプトが表示されたら、**Enter** を 3 回押してから、次を実行してキーを表示します。 

    ```
    cat /root/.ssh/id_dsa.pub
    ```

1.  キーの値をクリップボードにコピーします。

1.  i20-db-1 への SSH セッションで、次を実行してディレクトリ **/root/.ssh/** を作成します。

    ```
    mkdir /root/.ssh
    ```

1.  i20-db-1 への SSH セッションで、次を実行して、vi エディターでファイル **/root/.ssh/authorized\_keys** を開きます。

    ```
    vi /root/.ssh/authorized_keys
    ```

1.  エディター ウィンドウで、i20-db-1 で生成したキーを貼り付けます。

1.  変更を保存し、エディターを閉じます。

1.  i20-db-1 への SSH セッションで、次を実行してパスフレーズレス SSH キーを生成します。

    ```
    ssh-keygen -tdsa
    ```

1.  プロンプトが表示されたら、**Enter** を 3 回押してから、次を実行してキーを表示します。 

    ```
    cat /root/.ssh/id_dsa.pub
    ```

1.  キーの値をクリップボードにコピーします。

1.  i20-db-0 への SSH セッションで、次を実行して、vi エディターでファイル **/root/.ssh/authorized\_keys** を開きます。

    ```
    vi /root/.ssh/authorized_keys
    ```

1.  エディター ウィンドウで、i20-db-1 で生成したキーを貼り付けます。

1.  変更を保存し、エディターを閉じます。

1.  i20-db-0 への SSH セッションで、次を実行してパスフレーズレス SSH キーを生成します。

    ```
    ssh-keygen -t rsa
    ```

1.  プロンプトが表示されたら、**Enter** を 3 回押してから、次を実行してキーを表示します。 

    ```
    cat /root/.ssh/id_rsa.pub
    ```

1.  キーの値をクリップボードにコピーします。

1.  i20-db-1 への SSH セッションで、次を実行して、vi エディターでファイル **/root/.ssh/authorized\_keys** を開きます。

    ```
    vi /root/.ssh/authorized_keys
    ```

1.  エディター ウィンドウで、新しい行から開始する i20-db-0 で生成したキーを貼り付けます。

1.  変更を保存し、エディタを閉じます。

1.  SSH セッション～i20-db-1 内で、次のコマンドを実行してパスフレーズ不要の SSH キーを生成します。

    ```
    ssh-keygen -t rsa
    ```

1.  プロンプトが表示されたら、**Enter** を 3 回押してから、次を実行してキーを表示します。 

    ```
    cat /root/.ssh/id_rsa.pub
    ```

1.  キーの値をクリップボードにコピーします。

1.  i20-db-0 への SSH セッションで、次を実行して、vi エディターでファイル **/root/.ssh/authorized\_keys** を開きます。

    ```
    vi /root/.ssh/authorized_keys
    ```

1.  エディター ウィンドウで、新しい行から開始する i20-db-1 で生成したキーを貼り付けます。

1.  変更を保存し、エディタを閉じます。

1.  i20-db-0 への SSH セッションで構成が成功したことを確認するには、次を実行して、i20-db-0 から i20-db-1 への SSH セッションを **root** として確立します。 

    ```
    ssh root@i20-db-1
    ```

1.  接続を続行するかどうかを確認するメッセージが表示されたら、`yes`と入力し、**Enter** キーを押します。 

1.  パスワードの入力が求められないことを確認します。

1.  次の手順を実行して、i20-db-0 から i20-db-1 への SSH セッションを閉じます。 

    ```
    exit
    ```

1.  i20-db-1 への SSH セッションで、次の手順を実行して、i20-db-1 から i20-db-0 への SSH セッションを **root** として確立します。 

    ```
    ssh root@i20-db-0
    ```

1.  接続を続行するかどうかを確認するメッセージが表示されたら、`yes`と入力し、**Enter** キーを押します。 

1.  パスワードの入力が求められないことを確認します。

1.  次の手順を実行して、i20-db-1 から i20-db-0 への SSH セッションを閉じます。 

    ```
    exit
    ```

### タスク 5: YaST パッケージを追加し、Linux オペレーティング システムを更新し、HA 拡張機能をインストールする

1.  i20-db-0 への SSH セッションで、次を実行して YaST を起動します。

    ```
    yast
    ```

1.  **YaST コントロール センター**で、**ソフトウェア -\> アドオン製品** を選択し、**Enter** を押します。これにより **Package Manager** が読み込まれます。

1.  「**インストールされたアドオン製品**」 画面で、**パブリック クラウド モジュール**が既にインストールされていることを確認します。次に、**F9** を 2 回押してシェル プロンプトに戻ります。

1.  i20-db-0 への SSH セッションで、次を実行してオペレーティング システムを更新します (プロンプトが表示されたら、**y** と入力し、**Enter** キーを押します)。

    ```
    zypper update
    ```

1. i20-db-0 への SSH セッションで、次を実行して HA 拡張機能の依存関係を更新します (プロンプトが表示されたら、**y** と入力して **Enter** キーを押し、**SUSE エンド ユーザー使用許諾契約**を読んで **q** と入力し、**はい**と入力してライセンス条件に同意し、**Enter** キーをもう一度押します)。

    ```
    zypper install sle-ha-release fence-agents
    ```

1. i20-db-1 の前の手順を繰り返します。

> **結果**: このエクササイズを完了すると、可用性の高い SAP NetWeaver デプロイをサポートするように、Linux を実行している Azure VM のオペレーティング システムを構成することができます

## 演習 3: 可用性の高い SAP NetWeaver デプロイをサポートするように Linux を実行している Azure VM でクラスタリングを構成

時間: 60 分

このエクササイズでは、可用性の高い SAP NetWeaver のデプロイをサポートするために、Linux を実行している Azure VM にクラスタリングを構成します。

### タスク 1: クラスタリングを構成する

1.  az12003a-vm0 への RDP セッション内で、i20-db-0 への PuTTY ベースの SSH セッションから次を実行して、i20-db-0 上の HA クラスターの構成を開始します。

    ```
    ha-cluster-init
    ```

1.  プロンプトが表示されたら、次の情報を入力します。

    -   続行しますか (y/n)?: **y**

    -   /root/.ssh/id_rsa は既に存在します - 上書きしますか (y/n)? **n**

    -   リング0 [10.3.0.20] のアドレス: **入力**

    -   ring0 [5405] のポート: **入力**

    -   SBD を使用しますか (y/n)?: **n**

    -   仮想 IP アドレスを構成しますか (y/n)?: **n**

   > **注記**: クラスタリングのセットアップでは、パスワードが **Linux** に設定された **hacluster** アカウントが生成されます。これついては、タスクの後半で取り上げます。

1.  az12003a-vm0 への RDP セッション内で、i20-db-1 への PuTTY ベースの SSH セッションから次を実行して、i20-db-1 から i20-db-0 上に HA クラスターを結合します。

    ```
    ha-cluster-join
    ```

1.  プロンプトが表示されたら、次の情報を入力します。

    -   続行しますか (y/n)?**y**

    -   既存のノードの IP アドレスまたはホスト名 (例: 192.168.1.1) \[\]: **i20-db-0**

    -   /root/.ssh/id\_rsa は既に存在します - 上書きしますか (y/n)? **n**

    -   /root/.ssh/id\_dsa は既に存在します - 上書きしますか (y/n)? **n**

    -   リング0 [10.3.0.21] のアドレス: **入力**

1.  i20-db-0 への PuTTY ベースの SSH セッションから、次を実行して、**hacluster** アカウントのパスワードを **Pa55w.rd1234** に設定します (プロンプトが表示されたら新しいパスワードを入力します)。 
    ```
    passwd hacluster

    ```

1.  i20-db-1 の前の手順を繰り返します。

### タスク 2: corosync 構成の確認

1.  az12003a-vm0 への RDP セッション内で、i20-db-0 への PuTTY ベースの SSH セッションから次を実行して、**/etc/corosync/corosync.conf** ファイルのコンテンツを確認します。

    ```
    cat /etc/corosync/corosync.conf
    ```

1. `transport: udpu` エントリと `nodelist` セクションに注意してください。
    ```
    [...]
       interface { 
           [...] 
       }
       transport:      udpu
    } 
    nodelist {
       node {
         ring0_addr:     10.3.0.20
         nodeid:     1
       }
       node {
         ring0_addr:     10.3.0.21
         nodeid:     2
       } 
    }
    logging {
        [...]
    ```

1.  i20-db-1 の前の手順を繰り返します。


### タスク 3: STONITH クラスタリング オプションを構成する

1.  az12003a-vm0 への RDP セッション内で、i20-db-0 への PuTTY ベースの SSH セッションから、以下のコンテンツを持つ **crm-defaults.txt** という名前の新しいファイルを作成します。

    ```
    property $id="cib-bootstrap-options" \
      no-quorum-policy="ignore" \
      stonith-enabled="true" \
      stonith-action="reboot" \
      stonith-timeout="150s"
    rsc_defaults $id="rsc-options" \
      resource-stickiness="1000" \
      migration-threshold="5000"
    op_defaults $id="op-options" \
      timeout="600"
    ```

1.  変更を保存し、エディタを閉じます。

1.  PuTTY ベースの SSH セッション～i20-db-0 内で、次のコマンドを実行して新しく作成したファイルの設定を適用します。

    ```
    crm configure load update crm-defaults.txt
    ```

### タスク 4: Azure サブスクリプション ID と Azure AD テナント ID の値を特定する

1.  ラボ コンピューターのブラウザー ウィンドウで、Azure portal (**https://portal.azure.com**) を開き、サブスクリプションに関連付けられた Azure AD テナントでグローバル管理者ロールを持つユーザー アカウントでサインインしていることを確認します。

1.  Azure portal の Cloud Shell で Bash セッションを開始します。 

1.  「Cloud Shell」 ウィンドウで、次のコマンドを実行して、Azure サブスクリプションの ID と対応する Azure AD テナントの ID を特定します。

    ```
    az account show --query '{id:id, tenantId:tenantId}' --output json
    ```

1.  結果の値を Notepad にコピーします。これは、次のタスクで必要になります。


### タスク 5: STONITH デバイス用の Azure AD アプリケーションの作成

1.  Azure portal で、Azure Active Directory ブレードに移動します。

1.  Azure Active Directory ブレードから、「**アプリの登録**」 ブレードに移動し、「**+ 新規登録**」 をクリックします。

1.  **アプリケーションの登録**ブレードで、次の設定を指定し、「**登録**」 をクリックします。

    -   名前: **Stonith app**

    -   サポートされるストレージの種類: **この組織ディレクトリ内のアカウントのみ**

1.  **Stonith アプリ** ブレードで、**アプリケーション (クライアント) ID** の値を Notepad にコピーします。これは、このエクササイズの後半で **login_id** として参照します。

1.  **Stonith アプリ**ブレードで、「**証明書とシークレット**」 をクリックします。

1.  **Stonith アプリ - 証明書とシークレット** ブレードで、「**+ 新しいクライアントシークレット**」 をクリックします。

1.  「**クライアント シークレットの追加**」 ウィンドウの 「**説明**」 テキスト ボックスの 「**有効期限**」 セクションに 「**STONITH app key**」と入力し、既定の 「**年 1 **」 のままにして、「**追加**」 をクリック します。

1.  結果のシークレット値を Notepad にコピーします (このエントリは、「**追加**」 をクリックした後 1 度だけ表示されます)。これは、このエクササイズの後半で**パスワード**として参照します。


### タスク 6: STONITH アプリのサービス プリンシパルに Azure VM へのアクセス許可を付与する 

1.  Azure portal で、 **i20-db-0** Azure VM のブレードに移動します。

1.  **i20-db-0** ブレードから、**i20-db-0 - アクセスの制御 (IAM)** ブレードを表示します。

1.  **i20-db-0 - アクセスの制御 (IAM)** ブレードから、次の設定でロールの割り当てを追加します。

    -   ロール: **所有者**

    -   アクセスの割り当て先: **Azure AD ユーザー、グループ、またはサービス プリンシパル**

    -   選択: **Stonith app**

1.  前の手順を繰り返して、Stonith アプリの所有者ロールを **i20-db-1** Azure VM に割り当てます。


### タスク 7: STONITH クラスター デバイスを構成する 

1.  az12003a-vm0 への RDP セッションの i20-db-0 への PuTTY ベースの SSH セッションで、次のコンテンツの **crm-fencing.txt** という名前の新しいファイルを作成します (`subscription_id`、`tenant_id`、`login_id`、`password` は、エクササイズ 3 タスク 5 で特定した値のプレースホルダー)。

    ```
    primitive rsc_st_azure_1 stonith:fence_azure_arm \
         params subscriptionId="subscription_id" resourceGroup="az12003a-sap-RG" tenantId="tenant_id" login="login_id" passwd="password"
    primitive rsc_st_azure_2 stonith:fence_azure_arm \
         params subscriptionId="subscription_id" resourceGroup="az12003a-sap-RG" tenantId="tenant_id" login="login_id" passwd="password"
    colocation col_st_azure -2000: rsc_st_azure_1:Started rsc_st_azure_2:Started
    ```

1.  s03-db-0 の SSH セッションから、**crm configure load update crm-fencing.txt** を実行して、ファイルに設定を適用します。
    ```
    crm configure load update crm-fencing.txt
    ```

### タスク 8: Hawk を使用して Linux を実行している Azure VM のクラスタリング構成を確認する

1.  az12003a-vm0 への RDP セッション内で、Internet Explorer を起動し、**https://i20-db-0:7630** に移動します。SUSE Hawk サインイン ページが表示されます。

   > **注記**: **このサイトはセキュリティで保護されていません**のメッセージを無視します。

1.  SUSE Hawk サインイン ページで、次の認証情報を使用してログインします。

    -   ユーザー名: **hacluster**

    -   パスワード: **Pa55w.rd1234**

1.  クラスターの状態が正常であることを確認します。2 つのクラスター ノードの 1 つが不完全であることを示すメッセージが表示される場合は、Azure portal からそのノードを再起動します。

> **結果**: このエクササイズを完了すると、可用性の高い SAP NetWeaver デプロイをサポートするように、Linux を実行している Azure VM にクラスタリングを構成することができます。


## 演習 4: ラボリソースを削除する

時間: 10 分

このエクササイズでは、このラボでプロビジョニングしたリソースすべてを削除します。

#### タスク 1: Cloud Shell を開く

1. ポータルの上部にある 「**Cloud Shell**」 アイコンをクリックして Cloud Shell ウィンドウを開き、シェルとして Bash を選択します。

1. ポータルの下部にある **「Cloud Shell」** コマンド プロンプトで、次のコマンドを入力し、**Enter** キーを押して、このラボで作成したすべてのリソース グループを一覧表示します。

    ```
    az group list --query "[?starts_with(name,'az12003a-')]".name --output tsv
    ```

1. このラボで作成したリソース グループのみが出力に含まれていることを確認します。これらのグループは、次のタスクで削除されます。

#### タスク 2: リソース グループの削除

1. **「Cloud Shell」** コマンド プロンプトで、次のコマンドを入力し、 **「Enter」** キーを押してこの実習ラボで作成したリソース グループを削除します。

    ```
    az group list --query "[?starts_with(name,'az12003a-')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

1. ポータルの下部にある **「Cloud Shell」** プロンプトを閉じます。


> **結果**: このエクササイズを完了すると、このラボで使用したリソースが削除されます。
