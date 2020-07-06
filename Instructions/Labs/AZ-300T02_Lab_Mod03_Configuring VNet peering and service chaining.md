﻿---
lab:
    title: 'VNet ピアリングとサービス チェーンの構成'
    module: 'モジュール 3:高度な仮想ネットワークの実装'
---

# 仮想ネットワークの作成と管理
# ラボ: VNet ピアリングとサービス チェーンの構成
  
### シナリオ
  
ADatum コーポレーションは、Azure サブスクリプションに Azure仮想ネットワーク間のサービス チェーンを実装したいと考えています。 


### 目的
  
この課題を終了すると、下記ができるようになります：

- Azure Resource Manager テンプレートを使用して、Azure VM をデプロイします。

- VNet ピアリングの構成。

- ルーティングの実装

- サービスチェーンの検証


### ラボの設定
  
推定時間： 45分間

ユーザー名： **Student**

パスワード： **Pa55w.rd**



## エクササイズ 1: デプロイ テンプレートを使用した Azure ラボ環境の作成
  
このエクササイズの主なタスクは次のとおりです。

1. Azure Resource Manager テンプレートを使用して、最初のAzure仮想ネットワーク環境を作成します

1. Azure Resource Manager テンプレートを使用して、2 番目のAzure 仮想ネットワーク環境を作成します


#### タスク 1: Azure Resource Manager テンプレートを使用して最初の Azure 仮想ネットワーク環境を作成します
  
1. ラボの仮想マシンから Microsoft Edge を起動し、[**http://portal.azure.com**](http://portal.azure.com) で Azure portalを参照し、ターゲット Azure サブスクリプションの所有者ロールを持つ Microsoft アカウントを使用してサインインします。

1. Azure portal の Microsoft Edge ウィンドウで、**Cloud Shell** 内の **Bash** セッションを開始します。 

1. **マウントされたストレージがない** というメッセージが表示された場合は、次の設定を使用してストレージを構成します

    - サブスクリプション： ターゲット Azure サブスクリプションの名前

    - Cloud Shell リージョン： サブスクリプションに使用でき、課題の場所に最も近い Azure リージョンの名前

    - リソース グループ: 新しいリソース グループ **az3000400-LabRG** の名前

    - ストレージ アカウント: 新しいストレージ アカウントの名前

    - ファイル共有:新しいファイル共有の名前

1. Cloud Shell ペーンから、下記を実行して2つのリソース グループを作成します (`<Azure region>` プレースホルダを、サブスクリプションに使用でき、ラボの場所に最も近い Azure リージョンの名前に置き換えます)。

```
   az group create --resource-group az3000401-LabRG --location <Azure region>
   az group create --resource-group az3000402-LabRG --location <Azure region>
```

1. Cloud Shell ペーンから、最初の Azure Resource Manager テンプレート **\\allfiles\\AZ-300T02\\Module_03\\azuredeploy0401.json** をホーム ディレクトリにアップロードします。

1. Cloud Shell ペインから、パラメータ ファイル **\\allfiles\\AZ-300T02\\Module_03\\azuredeploy04.parameters.json** をホーム ディレクトリにアップロードします。

1. Cloud Shell ペインから、下記を実行して、Windows Server 2016 データセンターをホストする Azure VM を最初の仮想ネットワークにデプロイします：

```
   az group deployment create --resource-group az3000401-LabRG --template-file azuredeploy0401.json --parameters @azuredeploy04.parameters.json --no-wait
```

    > **注意**: デプロイメントが完了するのを待たず、次のタスクに進みます。


#### タスク 2: Azure Resource Manager テンプレートを使用して2 番目の Azure 仮想ネットワーク環境を作成します

1. Cloud Shell ペーンから、2 番目の Azure Resource Manager テンプレート **\\allfiles\\AZ-300T02\\Module_03\\azuredeploy0402.json** をホーム ディレクトリにアップロードします。

1. Cloud Shell ペーンから、下記を実行して、Windows Server 2016 データセンターをホストする Azure VM を 2 番目の仮想ネットワークにデプロイします。

```
   az group deployment create --resource-group az3000402-LabRG --template-file azuredeploy0402.json --parameters @azuredeploy04.parameters.json --no-wait
```

    > **注意**: 2 番目のテンプレートは、同じパラメータ ファイルを使用します。 

    > **注意**: デプロイメントが完了するのを待たず、次のエクササイズに進みます。

> **結果**： このエクササイズを完了したら、Windows Server 2016 データセンターを実行する Azure VM をホストする二つの Azure 仮想ネットワークを作成する必要があります。 


## エクササイズ 2: VNet ピアリングの構成 
  
このエクササイズのタスクは次のとおりです。

1. 両方の仮想ネットワークの VNet ピアリングを構成します

#### タスク 1: 両方の仮想ネットワークの VNet ピアリングを構成します
  
1. Azure portal を表示する Microsoft Edge ウィンドウで、**az3000401-vnet** 仮想ネットワーク ブレードに移動します。

1. **Az3000401-vnet** ブレードから、次の設定で VNet ピアリングを作成します。

    - 最初の仮想ネットワークから 二番目の仮想ネットワークへのピアリングの名前: **az3000401-vnet-to-az3000402-vnet**

    - 仮想ネットワーク デプロイメント モデル: **Resource Manager**

    - サブスクリプション: このラボのために使用するAzureサブスクリプションの名前

    - 仮想ネットワーク: **az3000402-vnet**
    
    - 最初の仮想ネットワークから 2 番目の仮想ネットワークへのピアリングの名前: **az3000402-vnet-to-az3000401-vnet** 
    
    - 最初の仮想ネットワークから 二番目の仮想ネットワークへの仮想ネットワーク アクセスを許可します。**有効にする**
    
    - 2 番目の仮想ネットワークから最初の仮想ネットワークへの仮想ネットワーク アクセスを許可します。**有効にする**    

    - 最初の仮想ネットワークから 2 番目の仮想ネットワークへの転送トラフィックを許可します。**無効**
    
    - 2 番目の仮想ネットワークから最初の仮想ネットワークへの転送トラフィックを許可します。**無効**

    - ゲートウェイトランジットの許可: 無効

## エクササイズ 3: ルーティングの実装
  
このエクササイズの主なタスクは次のとおりです。

1. IP 転送を有効にします

1. ユーザー定義ルーティングの構成 

1. Windows Server 2016 を実行している Azure VM にルーティングを構成します


#### タスク 1: IP 転送を有効にします 
  
1. Microsoft Edge で **az3000401-nic2** ブレードの (**az3000401-vm2** の NIC) に移動します。

1. **az3000401-nic2** ブレードの**IP 構成**から、**IP 転送** を **有効** に設定して保存します。


#### タスク 2: ユーザー定義ルーティングの構成 

1. Azure portalから、次の設定で新しいルート テーブルを作成します:

    - 名前: **az3000402-rt1**

    - サブスクリプション: このラボに使用する Azure サブスクリプションの名前

    - リソース グループ: **az3000402-LabRG**

    - 場所: 仮想ネットワークを作成したのと同じ Azure リージョン
  
    - 仮想ネットワーク ゲートウェイ ルートの伝播： **無効**

    ルートテーブルの作成が完了したら、[**リソースに移動**] をクリックします。

1. Azure portal で、前の手順で作成したルート テーブル az3000402-rt1 で、[**設定**] の下の [**ルート**] をクリックし、次の設定でルートを追加します。 

    - ルート名: **custom-route-to-az3000401-vnet**

    - アドレス プレフィックス: **10.0.0.0/22**

    - 次のホップの種類: **仮想アプライアンス**

    - 次のホップのアドレス: **10.0.1.4**

1. Azure portal で、**az3000402-vnet** の **subnet-1** にルート テーブルを関連付けます。


#### タスク 3: Windows Server 2016 を実行している Azure VM にルーティングを構成します

1. ラボ コンピュータで、Azure portal から **az3000401-vm2** Azure VM へのリモート デスクトップ セッションを開始します。 

1. 認証を求める場合は、次の資格情報を指定します。

    - ユーザー名: **Student**

    - パスワード: **Pa55w.rd1234**

1. az3000401-vm2 に接続されたら リモート デスクトップ セッションで **Server Manager** から 、**Remote Access** の役割と Remote Access の **Routing** ロール サービスとすべての必要な機能をインストールします。 

1. az3000401-vm2 へのリモート デスクトップ セッションで、 **Routing and Remote Access** コンソールを起動します。 

1. **Routing and Remote Access** コンソールで、サーバー az3000401-vm2 の名前を右クリックし、[**Configure and Enable Routing and Remote Access**] を選択して、**Routing and Remote Access Server Setup Wizard** を実行します。

1. **Routing and Remote Access Server Setup Wizard** では、**Configuration** の下で **Custom configuration** を選択し、**LAN routing** を有効にします。 

1. **Routing and Remote Access** サービスを開始します。

1. az3000401-vm2 のリモート デスクトップ セッションで、**Windows Firewall with Advanced Security** を起動し、すべてのプロファイルに対してInbound Rules の **File and Printer Sharing (Echo Request - ICMPv4-In)** のを有効にします。

> **結果**: ここのエクササイズを完了したら、2 番目の仮想ネットワーク内にカスタム ルーティングを構成する必要があります。


## エクササイズ 4: サービス チェーンの検証
  
このエクササイズの主なタスクは次のとおりです。

1. Azure VM に高度なセキュリティを使用して Windows ファイアウォールを構成します。

1. ピアリングされた仮想ネットワーク間のサービス チェーンのテストします


#### タスク 1: ターゲット Azure VM に高度なセキュリティを使用して Windows ファイアウォールを構成します。
  
1. ラボ コンピュータに、Azure portal から **az3000401-vm1** Azure VM にリモート デスクトップ セッションを開始します。 

1. 認証を求める場合は、次の資格情報を指定します。

    - ユーザー名: **Student**

    - パスワード: **Pa55w.rd1234**

1. az3000401-vm1 へのリモート デスクトップ セッションで、**Windows Firewall with Advanced Security** を起動し、すべてのプロファイルに対してInbound Rules の **File and Printer Sharing (Echo Request - ICMPv4-In)** のを有効にします。


#### タスク 2: ピアリングされた仮想ネットワーク間のサービス チェーンのテストします
  
1. ラボ コンピュータに、Azure portal から **az3000402-vm1** Azure VM にリモート デスクトップ セッションを開始します。 

1. 認証を求める場合は、次の資格情報を指定します。

    - ユーザー名: **Student**

    - パスワード: **Pa55w.rd1234**

1. リモート デスクトップ セッションを通して az3000402-vm1 に接続されたら、**Windows PowerShell** を起動します。

1. **Windows PowerShell** ウィンドウに、次を実行します。

```pwsh
   Test-NetConnection -ComputerName 10.0.0.4 -TraceRoute
```

1. テストの成功を確認し、接続が 10.0.1.4 を超えてルーティングされたことを確認します。

>  **結果**: このエクササイズをでは、ピアリングされた仮想ネットワーク間のサービス チェーンを検証しました。

## 演習 5: ラボリソースを削除

#### タスク1： Cloud Shell を開く

1. ポータルの上部にある **Cloud Shell** アイコンをクリックして、Cloud Shell ペインを開きます。

1. 必要に応じて、Cloud Shell ペインの左上隅にあるドロップ ダウン リストを使用して、Bash シェル セッションに切り替えます。

1. **「Cloud Shell」** コマンドプロンプトで、次のコマンドを入力し、**「Enter」** キーを押して、このラボで作成したすべてのリソース グループを一覧表示します。

```
   az group list --query "[?starts_with(name,'az30004')]".name --output tsv
```

1. 出力に、この実習ラボで作成したリソース グループのみが含まれていることを確認します。これらのグループは、次のタスクで削除されます。

#### タスク2： リソース グループの削除 

1. **「Cloud Shell」** コマンド プロンプトで、次のコマンドを入力し、 **「Enter」** キーを押してこの実習ラボで作成したリソース グループを削除します。

```sh
   az group list --query "[?starts_with(name,'az30004')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
```

1. ポータルの下部にある **「Cloud Shell」** プロンプトを閉じます。

> **結果**: このエクササイズでは、このラボで使用するリソースを削除しました。
