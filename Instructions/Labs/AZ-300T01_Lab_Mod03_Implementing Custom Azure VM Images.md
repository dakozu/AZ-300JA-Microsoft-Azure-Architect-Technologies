﻿---
lab:
    title: 'カスタム Azure VM イメージの実装'
    module: 'モジュール 3: 仮想マシン (VM) のデプロイと管理'
---

# 仮想マシン (VM) のデプロイと管理

# ラボ：カスタム Azure VM イメージの実装

### シナリオ

アダタムコーポレーションは、カスタムの Azure VM イメージを作成したいと考えています

### 目的

このラボを終了すると、下記ができるようになります：

- HashiCorp Packer を使用してカスタム VM イメージを作成する

- カスタム イメージに基づく　Azure VM をデプロイします

### ラボのセットアップ

推定時間：45 分間

インタフェース: **BASH モードで Azure Cloud Shell を使用する**

## 演習 1: カスタムイメージを作成します。

このエクササイズの主なタスクは次のとおりです：

1. パッカー テンプレートの構成

1. パッカーに基づくイメージの構築

#### タスク 1：パッカー テンプレートの構成

1. ラボの仮想マシンから Microsoft Edge を起動し、[**http://portal.azure.com**](http://portal.azure.com) で Azure portalを参照 し、ターゲット Azure サブスクリプションの所有者ロールを持つ Microsoft アカウントを使用してサインインします。

1. Azure portal　に、Microsoft Edge ウィンドウに、**Cloud Shell** 内で **Bash** セッションを開始 します。

1. **マウントされたストレージがない** というメッセージが表示された場合は、次の設定を使用してストレージを構成します：

   - サブスクリプション： ターゲット Azure サブスクリプションの名前

   - Cloud Shell リージョン： サブスクリプションに使用でき、ラボの場所に最も近い Azure リージョンの名前

   - リソース グループ： **az3000300-LabRG**

   - ストレージ アカウント： 新しいストレージ アカウントの名前

   - ファイル共有： 新しいファイル共有の名前

1. Cloud Shell　ペインから、次の操作を実行してリソース グループを作成し、JSON 出力を変数に保存します (`<Azure region>`プレースホルダを、サブスクリプションに使用でき、ラボの場所に最も近い　Azure リージョンの名前に置き換えます)。

```sh
   RG=$(az group create --name az3000301-LabRG --location <Azure region>)
```
   > **注記**: Azure リージョンを一覧表示するには、`az account list-locations --output table` を実行します

1. Cloud Shell　ペインから、次の操作を実行して、Packer で使用されるサービス プリンシパルを作成し、JSON 出力を変数に保存します：

```sh
   AAD_SP=$(az ad sp create-for-rbac)
```

1. Cloud Shell　ペインから、次の操作を実行してサービス プリンシパル appId の値を取得し、変数に保存します。

```sh
   CLIENT_ID=$(echo $AAD_SP | jq -r .appId)
```

1. Cloud Shell　ペインから、次の操作を実行して、サービス プリンシパル パスワードの値を取得し、変数に保存します。

```sh
   CLIENT_SECRET=$(echo $AAD_SP | jq -r .password)
```

1. Cloud Shell　ペインから、次の操作を実行して、サービス プリンシパル テナント ID の値を取得し、変数に保存します。

```sh
   TENANT_ID=$(echo $AAD_SP | jq -r .tenant)
```

1. Cloud Shell　ペインから、次の操作を実行して、サブスクリプション ID の値を取得し、変数に保存します：

```sh
   SUBSCRIPTION_ID=$(az account show --query id | tr -d '"')
```

1. Cloud Shell　ペインから、次の操作を実行して、リソース グループの場所の値を取得し、変数に保存します：

```sh
   LOCATION=$(echo $RG | jq -r .location)
```

1. Cloud Shell　ペインから、パッカー テンプレート **\\allfiles\\AZ-300T01\\Module_03\\template03.json** を主目次にアップロードします。ファイルをアップロードするには、Cloud Shell　ペインに上下矢印の付いたドキュメント アイコンをクリックします。 

1. Cloud Shell　ペインから、次の操作を実行して、**client_id** パラメーターの値のプレースホルダを Packer テンプレートの **\$CLIENT_ID** 変数の値に置き換えます：

```sh
   sed -i.bak1 's/"$CLIENT_ID"/"'"$CLIENT_ID"'"/' ~/template03.json
```

1. Cloud Shell　ペインから、次の操作を実行して、**client_secret** パラメーターの値のプレースホルダを Packer テンプレートの **\$CLIENT_SECRET** 変数の値に置き換えます。

```sh
   sed -i.bak2 's/"$CLIENT_SECRET"/"'"$CLIENT_SECRET"'"/' ~/template03.json
```

1. Cloud Shell　ペインから、**tenant_id** パラメーターのプレースホルダを Packer テンプレートの **\$TENANT_ID** 変数の値に置き換えます：

```sh
   sed -i.bak3 's/"$TENANT_ID"/"'"$TENANT_ID"'"/' ~/template03.json
```

1. Cloud Shell　ペインから、次の操作を実行して、**subscription_id** パラメーターの値のプレースホルダをPacker テンプレートの **\$SUBSCRIPTION_ID** 変数の値に置き換えます。

```sh
   sed -i.bak4 's/"$SUBSCRIPTION_ID"/"'"$SUBSCRIPTION_ID"'"/' ~/template03.json
```

1. Cloud Shell　ペインから、次の操作を実行して、**location** パラメーターの値のプレースホルダを Packer テンプレートの **\$LOCATION** 変数の値に置き換えます。

```sh
   sed -i.bak5 's/"$LOCATION"/"'"$LOCATION"'"/' ~/template03.json
```

#### タスク 2：パッカーに基づくイメージの構築

1. Cloud Shell　ペインから、次の操作を実行して、パッカーに基づくイメージを構築します。

```sh
   packer build template03.json
```

1. 構築の過程が完了するまで監視します。

   > **注記**：構築の過程は約 10　分間かかる場合があります。

> **まとめ**：このエクササイズでは、Packer テンプレートを作成し、それを使用してカスタム イメージを作成しました。

## 演習 2: カスタム イメージのデプロイ

このエクササイズの主なタスクは次のとおりです：

1. カスタム イメージに基づく　Azure VM をデプロイします

1. Azure VM のデプロイメントの検証

#### タスク 1：カスタム イメージに基づく　Azure VM をデプロイします

1. Cloud Shell　ペインから、次の操作を実行して、カスタム イメージに基づく Azure VM をデプロイします。

```sh
   az vm create --resource-group az3000301-LabRG --name az3000301-vm --image az3000301-image --admin-username student --generate-ssh-keys
```

1. デプロイメントが完了するのを待ってください。

   > **注記**：デプロイメント過程は約 3 分間かかる場合があります。

1. デプロイメントが完了した後、Cloud Shell　ペインから次の操作を実行して、TCP ポート 80 で新しくデプロイされた VM への着信トラフィックを許可します。

```sh
   az vm open-port --resource-group az3000301-LabRG --name az3000301-vm --port 80
```

#### タスク 2：Azure VM のデプロイメントの検証

1. Cloud Shell　ペインから、次の操作を実行して、新しくデプロイされた Azure VM に関連する IP アドレスを確認します。

```sh
   az network public-ip show --resource-group az3000301-LabRG --name az3000301-vmPublicIP --query ipAddress
```

1. Microsoft Edge を起動し、前のステップに確認した IP アドレスにアクセスします。

1. Microsoft Edge に **nginx　へようこそ!** ページが表示されていることを確認します。

> **まとめ**：このエクササイズでは、カスタム イメージに基づく Azure VM をデプロイし、デプロイメントを検証しました。

## 演習 3: ラボリソースを削除

#### タスク 1：Cloud Shell　を開きます

1. Portal　の上部にある **Cloud Shell** アイコンをクリックして、Cloud Shell　ペインを開きます。

1. **Cloud Shell** コマンド プロンプトに下記のコマンドを入力し、**Enter** キーを押して、このラボで作成したすべてのリソース グループを一覧表示します：

```sh
   az group list --query "[?starts_with(name,'az3000301')]".name --output tsv
```

1. 出力に、このラボで作成したリソース グループのみが含まれていることを確認します。これらのグループは、次のタスクで削除されます。

#### タスク 2：リソース グループの削除

1. **Cloud Shell** コマンド プロンプトに下記のコマンドを入力し、**Enter** キーを押してこのラボで作成したリソース グループを削除します。

```sh
   az group list --query "[?starts_with(name,'az3000301')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
```

1. Portal　の下部にある **Cloud Shell** プロンプトを閉じます。

> **結果**：このエクササイズでは、このラボで使用するリソースを削除しました。
