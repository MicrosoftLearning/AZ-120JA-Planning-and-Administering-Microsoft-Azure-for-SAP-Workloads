
# AZ 120: ラボの前提条件

## vCPU コア要件

-   このコースの最後のラボを終了するには、このラボにデプロイされる Azure VM が存在する可用性ゾーンをサポートする Azure リージョンで利用可能な、少なくとも 28 の vCPUを持つ Microsoft Azure サブスクリプションが必要です。

    -   4 x Standard_DS1_v2 (各 1 vCPU) = 4

    -   6 x Standard_D4s_v3 (各 4 vCPU) = 24

    > **注記**: リソースのデプロイには **East US** または **East US2** リージョンの使用を検討してください。

    > **注記**: 可用性ゾーンをサポートする Azure リージョンを特定するには、[https://docs.microsoft.com/ja-jp/azure/availability-zones/az-overview]<https://docs.microsoft.com/ja-jp/azure/availability-zones/az-overview> を参照してください

このコースの最初の 3 つのラボの vCPU 要件は低いですが、クォータを増やすプロセスには時間がかかる場合があるため (クォータの増加要求が同じ営業日に完了する場合もあります)、ラボ全体の要件を満たせるようクォータの増加を要求するようお勧めします。

## ハンズオン ラボの前

時間: 120 分

### タスク 1: vCPU コアが十分であることを確認する

1.  Azure portal <http://portal.azure.com> にログインします。 

1.  Azure portal の Cloud Shell で PowerShell セッションを開始します。 

    > **注記**: 現在の Azure サブスクリプションで Cloud Shell を初めて起動する場合は、Azure ファイル共有を作成して Cloud Shell ファイルを永続化するように求められます。その場合は、既定値に設定すると、自動的に生成されたリソース グループ内にストレージ アカウントが作成されます。

1.  Azure Portal の **Cloud Shell** ペインで、PowerShell プロンプトで、次を実行します。`<Azure_region>` は、このラボで使用する対象の Azure リージョンを指定します。(例: `eastus2`)

    ```powershell
    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'StandardDSv3Family'}

    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'StandardDSv2Family'}
    ``` 

    > **注記**: Azure リージョンの名前を特定するには、**Cloud Shell** の Bash プロンプトで `(Get-AzLocation).Location` を実行します。
   
1.  前の手順で実行したコマンドの出力の現在の値と制限エントリを確認し、Azure リージョンに十分な数の vCPU があることを確認します。

1.  vCPU の数が十分でない場合は、Azure portal でサブスクリプション ブレードに戻り、**「使用状況 + クォータ」** をクリックします。 

1.  サブスクリプションの **「使用状況 + クォータ」** ブレードで、**「増量のリクエスト」** をクリックします。

1.  **Basic** ブレードで、次を指定して **「次へ」** をクリックします。

    -   問題の種類: **サービスとサブスクリプションの制限 (クォータ)**

    -   サブスクリプション: このラボで使用する Azure サブスクリプションの名前

    -   クォータの種類: **コンピューティング/VM (コア/vCPU) サブスクリプション制限の増加)。**

1.  **「詳細」** ブレードで、**「詳細の入力」** ブレードをクリックします。 

1.  **クォータの詳細**ブレードで、次を指定し、**「保存して続行」** をクリックします。

    -   デプロイメントモデル：**Resource Manager**

    -   保存先: このラボで使用する対象の Azure リージョン

    -   SKU ファミリ: **DSv3 シリーズ**および **DSv2 シリーズ**

1.  **詳細**ブレードで、各 SKU シリーズの新しい制限を指定し、**「次へ」** をクリックします。**確認と作成**:

    -   重大度: **C - 最小限の影響**

    -   希望する連絡方法: 希望するオプションを選択し、連絡先の詳細を入力する

1.  **確認と作成**ブレードで、**「作成」** をクリックします

   > **注記**: クォータの増加要求は、通常、同じ営業日に完了します。