---
lab:
    title: 'Azure から Azure への移行の実装'
    module: 'モジュール 1: Azure へのサーバーの移行の評価と実行'
---

# Azure へのサーバー移行の評価と実行

# ラボ: Azure から Azure への移行の実装

### シナリオ

Adatum Corporation は、既存の Azure VM を別のリージョンに移行しようとしています。

### 目的

この課題を終了すると、下記ができるようになります。

- Azure Site Recovery コンテナーの実装

- Azure リージョン間に Azure VM を移行します

### ラボの設定

推定時間：45 分間

ユーザー名： **Student**

パスワード： **Pa55w.rd**

## エクササイズ 1: Azure Site Recovery を使用して Azure VM を移行するための前提条件を実装します

このエクササイズの主なタスクは次のとおりです:

1. 移行する Azure VM をデプロイします

1. Recovery Services コンテナーを作成します

#### タスク 1: 移行する Azure VM をデプロイします

1. ラボの仮想マシンから Microsoft Edge を起動し、[**http://portal.azure.com**](http://portal.azure.com) で Azure portalを参照し、ターゲット Azure サブスクリプションの所有者ロールを持つ Microsoft アカウントを使用してサインインします。

1. Azure portal には、Microsoft Edge ウィンドウに、**Cloud Shell** 内の **PowerShell** セッションを開始 します。

1. **マウントされたストレージがない**というメッセージが表示された場合は、次の設定を使用してストレージを構成します

   - サブスクリプション： ターゲット Azure サブスクリプションの名前

   - Cloud Shell リージョン： サブスクリプションに使用でき、課題の場所に最も近い Azure リージョンの名前

   - リソース グループ: 新しいリソース グループ **az3000600-LabRG** の名前

   - ストレージ アカウント: 新しいストレージ アカウントの名前

   - ファイル共有:新しいファイル共有の名前

1. Cloud Shell ウィンドウから、下記を実行してリソース グループを作成します (`<Azure region>` プレースホルダを、サブスクリプションに使用でき、ラボの場所に最も近い Azure リージョンの名前に置き換えます)。

```pwsh
   New-AzResourceGroup -Name az3000601-LabRG -Location <Azure region>
```

1. Cloud Shell ペインから、Azure Resource Manager テンプレート **\\allfile\\AZ-300T02\\Module_01\\azuredeploy06.json** をホーム ディレクトリにアップロードします。

1. Cloud Shell ペインから、パラメータ ファイル **\\allfile\\AZ-300T02\\Module_01\\azuredeploy06.parameters.json**をホーム ディレクトリにアップロードします。

1. Cloud Shell ウィンドウから、次の方法で Windows Server 2016 データセンターをホストする Azure VM をデプロイします。

```pwsh
   New-AzResourceGroupDeployment -ResourceGroupName az3000601-LabRG -TemplateFile azuredeploy06.json -TemplateParameterFile azuredeploy06.parameters.json
```

   > **注意**: デプロイが完了するのを待たず、代わりに次のタスクに進みます。

   > **注意**: azuredeploy06.json が見つからない場合は、-TemplateFile $HOME/azuredeploy06.json および -TemplateParameterFile $HOME/azuredeploy06.parameters.json を使用します。

#### タスク 2: Azure Site Recovery コンテナーを作成します

1. Azure Portal から、次のの設定を使用して、**Recovery Services コンテナー** のインスタンスを作成します。

   - 名前:**vaultaz3000602**

   - サブスクリプション：ターゲット　Azure　サブスクリプションの名前

   - リソース グループ: **az3000602-LabRG** という名前の新しいリソース グループ 

   - 場所: サブスクリプションに使用可能で、前のタスクに Azure VM をデプロイしたリージョンと **異なる** Azure リージョンの名前

1. コンテナーがプロビジョニングされるまで待ちます。これにはおよそ1分間かかります。

> **結果**： このエクササイズを完了したら、移行する Azure VM と、Azure VM の移行されたディスク ファイルをホストする Azure Site Recovery コンテナーを作成しました。

## エクササイズ 2: Azure Site Recovery を使用して Azure リージョン間に Azure VM を移行します

このエクササイズの主なタスクは次のとおりです:

1. Azure VM レプリケーションの構成

1. Azure VM レプリケーション設定のレビュー

1. Azure VM のレプリケーションを無効にし、Azure Recovery Services コンテナーを削除します

#### タスク 1: Azure VM レプリケーションの構成

1. Azure portal に、新しくプロビジョニングされた Azure Recovery Services コンテナーのブレードに移動します。

1. Recovery Services コンテナー ブレードに、**+ レプリケート** ボタンをクリックします。

1. 次の設定を指定してレプリケーションを有効にします。

   - ソース: **Azure**

   - ソースの場所: このラボの前のエクササイズに Azure VM をデプロイしたのと同じ Azure リージョン

   - Azure 仮想マシン デプロイ モデル : **Resource Manager**

   - ソース リソース グループ: **az3000601-LabRG**

   - 仮想マシン: **az3000601-vm**

   - ターゲットの場所: サブスクリプションに使用可能で、前のタスクにAzure VM をデプロイしたリージョンと異なる Azure リージョンの名前

   - ターゲット リソース グループ: **(新規) az3000601-LabRG-asr**

   - ターゲット仮想ネットワーク: **(新規) az3000601-vnet-asr**

   - キャッシュ ストレージ アカウント: 既定の設定を受け入れます

   - レプリカ マネージド ディスク: **(新規) プレミアム ディスク 1 台、標準ディスク 0**

   - ターゲット可用性セット: **該当なし（単一インスタンス）**

   - レプリケーション ポリシー: **新規作成**

   - 名前: **12-hour-retention-policy**

   - 復旧ポイントのリテンション期間: **12 時間**

   - アプリ整合性のスナップショット頻度: **6 時間**

   - マルチ VM の整合性: **いいえ**

1. ターゲット リソースの作成を開始します。

1. レプリケーションを有効にします。

   > **注意**: レプリケーションの有効化操作が完了するまで待ちます。そして、次のタスクに進みます。

#### タスク 2: Azure VM レプリケーション設定の確認

1. Azure portal には、Azure Site Recovery コンテナー ブレードから、レプリケートされたアイテム の **az3000601-vm** を表すAzure VMに移動します。

2. レプリケートされたアイテムブレードで、**正常性と状態**、**利用可能な最新の回復ポイント**、および**フェールオーバー準備** セクションを確認します。ツールバーの **フェールオーバー** エントリと **テスト フェイルオーバー** エントリに注意してください。**インフラストラクチャ ビュー** までスクロール ダウンします。

3. 時間が許す場合は、Azure VM の状態が **保護済み** に変わるまで待ちます。これはおよそ 15 - 20 分間かかるかもしれません。そのポイントに、値の **クラッシュ整合性** と **アプリ整合性** のある復旧ポイントを調べます。**RPO** を表示するには、テスト フェールオーバーを実行する必要があります。

#### タスク 3: Azure VM のレプリケーションを無効にし、Azure 回復サービス ボールトを削除します

1. Azure portal で、Azure VM **az3000601-vm** のレプリケーションを無効にします。

2. レプリケーションが無効になるまで待ちます。

3. Azure portal から、Recovery Services コンテナーを削除します。

   > **注意**: コンテナーを削除する前に、レプリケートされたアイテムが最初に削除されていることを確認する必要があります。

> **結果**: このエクササイズを完了した後、Azure VM の自動レプリケーションを実装しました。

## 演習 3: ラボリソースを削除

#### タスク 1：Cloud Shell を開く

1. ポータルの上部にある **Cloud Shell** アイコンをクリックして、Cloud Shell ペインを開きます。

1. 必要に応じて、Cloud Shell ペインの左上隅にあるドロップ ダウン リストを使用して、Bash シェル セッションに切り替えます。

1. **「Cloud Shell」** コマンドプロンプトで、次のコマンドを入力し、**「Enter」**キーを押して、このラボで作成したすべてのリソース グループを一覧表示します。

```
   az group list --query "[?starts_with(name,'az30006')].name" --output tsv
```

1. 出力に、この実習ラボで作成したリソース グループのみが含まれていることを確認します。これらのグループは、次のタスクで削除されます。

#### タスク 2: リソース グループの削除

1. **Cloud Shell** コマンド プロンプトで、次のコマンドを入力し、**Enter** キー を押してこのラボで作成したリソース グループを削除します:

```sh
   az group list --query "[?starts_with(name,'az30006')].name" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
```

1. ポータルの下部にある **Cloud Shell** プロンプトを閉じます。

> **結果**: このエクササイズでは、このラボで使用するリソースを削除しました。
