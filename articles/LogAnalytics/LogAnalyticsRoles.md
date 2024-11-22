---
title: 
date: 2024-11-31 00:00:00
tags:
  - LogAnalytics
  - How-To
---

こんにちは、Azure Monitoring サポート チームの徳田です。
今回は、Log Analytics 利用するにあたって必要なロールについてご紹介します。

<br>

<!-- more -->
## 目次


## Log Analytics においてよくあるロールに関連するトラブル


## はじめに：アクセス権と Azure ロール
以下では、アクセス権とロールとは何か、を簡単に説明します。
既にご存じの方は、次の項目からご覧ください。

###  アクセス権とは
Azure の各リソースにアクセスしたり、各操作を実行したりするためには、実行者にそれぞれの権限が付与されていることが必要です。
この権限のことをアクセス権といいます。

### Azure ロールとは
まず、Azure では [Azure RBAC](https://learn.microsoft.com/ja-jp/azure/role-based-access-control/overview) という、Azure リソースに対するアクセス管理のためのシステムがあります。
Azure RBAC では、ユーザーやグループに特定の Azure ロールを割り当てることで、リソースに対するアクセス権を制御します。
つまり、"Azure ロール (＊)" はひとことで表すと「ユーザーやグループに一括で付与できるアクセス権のまとまり」です。  

＊ 他のロール (Microsoft Entra ロールなど) と区別するために、"Azure ロール" と表現されることもありますが、単に "ロール" とも表現されます。  
本記事では、以降 Azure ロールのことを、"ロール" と表記します。

Azure ロールの中には、事前に用意・提供される組み込みロールと、独自に定義するカスタム ロールがあります。

また、各ロールは複数のスコープ (管理グループ、サブスクリプション、リソース グループ、リソース) で指定できます。

## Log Analytics に関連する組み込みロール
Log Analytics ワークスペースの利用に関する組み込みロールとして、以下が用意されています。

**■ Log Analytics 閲覧者**  
以下の操作が可能です。
- 全ての Azure リソースの情報を表示、検索する。  
  Log Analytics ワークスペースに対してこの権限を持つ場合、そのワークスペースの設定の読み取りや、そのワークスペースに対するクエリの実行が、制限なく可能です。  
  (ただし、そのワークスペースに対してワークスペース コンテキストでアクセスしている場合に限ります。  
  リソース コンテキストでアクセスする場合については、後述の ~~~ をご確認ください。)
- 監視設定 (診断設定の構成など) を表示する。  

**■ Log Analytics 共同作成者**  
主に、以下の操作が可能です。
- 全ての Azure リソースの情報を表示、検索する。  
  そのワークスペースの設定の読み取りや、そのワークスペースに対するクエリの実行が、制限なく可能です。  
  (ただし、そのワークスペースに対してワークスペース コンテキストでアクセスしている場合に限ります。  
  リソース コンテキストでアクセスする場合については、後述の ~~~ をご確認ください。)
- Azure リソースの監視設定を編集する。  
  例) VM に拡張機能をインストールする (Azure Monitor エージェントなど)、Azure リソースに対して診断設定を構成する  
- Log Analytics ワークスペースにソリューションを追加/削除する。  
  (リソース グループまたはサブスクリプション レベルでアクセス許可が付与されている必要があります。)  
- [データ エクスポート規則](https://learn.microsoft.com/ja-jp/azure/azure-monitor/logs/logs-data-export?tabs=portal)を構成する。
- [検索ジョブ](https://learn.microsoft.com/ja-jp/azure/azure-monitor/logs/search-jobs?tabs=portal-1%2Cportal-2)を実行する。
- 長期保持状態の (アーカイブ) データを[復元](https://learn.microsoft.com/ja-jp/azure/azure-monitor/logs/restore?tabs=api-1)する。


なお、上記のロールについては以下公開情報でもご案内しております。  
[Log Analytics ワークスペースへのアクセスを管理する | 組み込みのロール](https://learn.microsoft.com/ja-jp/azure/azure-monitor/logs/manage-access?tabs=portal#built-in-roles)

## Log Analytics に関連するカスタム ロール
### カスタム ロールが必要になるシナリオ
前述の [Log Analytics に関連する組み込みロール] で紹介した権限は、最小で、ワークスペースのスコープで付与できます (＊)。
(＊) リソース グループやサブスクリプションなど、ワークスペース以上のスコープでも付与可能です。  
一方で、カスタム ロールを使用することで、Log Analytics ワークスペースにデータを送信しているリソース レベルでアクセス権を制御することができます。  

リソース レベルでのアクセス権が必要になるのは、リソース コンテキストで Log Analytics ワークスペースにアクセスしており、かつそのワークスペースのアクセス制御モードが "リソースまたはワークスペースのアクセス制御" の場合です。  
この場合は、データが収集されている Log Analytics ワークスペースに対するアクセス権ではなく、データ収集元であるリソース (VM など) に対してアクセス権をもっている必要があります。  

アクセス モードとアクセス制御モードの詳細については、以下のブログにてご紹介しておりますため、こちらをご参照ください。  
- [Log Analytics ワークスペースのアクセス制御モードの違いについて](https://jpazmon-integ.github.io/blog/LogAnalytics/AccessControlMode/)  


### Log Analytics に関連するアクセス権
カスタム ロールとして、以下のアクセス権を含めることで、リソース レベルでのアクセスの制御が可能です。  

`Microsoft.Insights/logs/*/read` :  
リソースのすべてのログ データを表示する。  

`Microsoft.Insights/logs/<tableName>/read` :  
リソースからのデータが収集される特定のテーブルを表示する。 

例) `Microsoft.Insights/logs/Heartbeat/read` 

`Microsoft.Insights/diagnosticSettings/write` :  
リソースの診断設定を構成する。  


### Azure portal でのカスタム ロールの作成方法  
**JSON ファイルから作成する方法**  
1. 手元で以下のような JSON ファイルを用意します。  
   必要に応じて、

   ```
   {
    "properties": {
        "roleName": "<role-name>",
        "description": "<description>",
        "assignableScopes": ["<assignable-scopes>"],
        "permissions": [
                {
                    "actions": ["<action-permission>"],
                    "notActions": ["<notaction-permission>"],
                    "dataActions": ["<dataaction-permission>"],
                    "notDataActions": ["<notdataaction-permission>"]
                }
            ]
        }
   }
   ```

   例)  
   ```
   {
    "properties": {
        "roleName": "LAcustom1",
        "description": "test",
        "assignableScopes": ["/subscriptions/xxxxxxxxxx/resourceGroups/xxxxxxx"],
        "permissions": [
                {
                    "actions": ["Microsoft.Insights/logs/*/read",
                                "Microsoft.Insights/diagnosticSettings/write"
                                ],
                    "notActions": [],
                    "dataActions": [],
                    "notDataActions": []
                }
            ]
        }
    }
   ```
2. Azure portal でカスタム ロールを作成するスコープとなるリソース グループ、サブスクリプション、または管理グループを開きます。
3. 左ペインの [アクセス制御 (IAM)] を押下し、[追加] > [カスタム ロールの追加] を押下します。
4. [基本] タブの "ベースラインのアクセス許可" で "JSON から開始" を選択し、1 で用意したファイルをアップロードします。 
5. [アクセス許可] タブで、必要なアクセス権が追加されていることを確認します。  
   必要であれば、[アクセス許可の追加] から必要なアクセス権を追加します。
6. [割り当て可能なスコープ] で、割り当てたいスコープが選択されていることを確認します。  
   必要であれば、[割り当て可能なスコープの追加] からスコープを追加します。
7. [確認と作成] で問題がなければ、[作成] を押下し、完了です。


**Azure portal でアクセス権を選択し作成する方法**
1. Azure portal でカスタム ロールを作成するスコープとなるリソース グループ、サブスクリプション、または管理グループを開きます。
2. 左ペインの [アクセス制御 (IAM)] を押下し、[追加] > [カスタム ロールの追加] を押下します。
3. [基本] タブの "ベースラインのアクセス許可" で "最初から始める" を選択し、その他必要な情報を入力します。  
4. [アクセス許可] タブで [アクセス許可の追加] を押下します。  
   右ペインでリソース プロバイダー (＊) を検索し、選択します。  
   ロールに追加したいアクセス権にチェックを入れます。 
   ＊ リソース プロバイダーの検索文字が残ったままの場合、アクセス権が表示されない場合があります。  
      必要なアクセス権が表示されない場合は、一度検索欄を空白にしてください。
5. アクセス権の選択が完了したら、[追加] を押下します。
6. 上記手順の 6 と同様です。
7. 上記手順の 7 と同様です。  

### ロールの割り当て方法
ロールの割り当て方法は、組み込みロール、カスタム ロールともに同様です。
1. ロールを割り当てたいスコープのリソースを開きます。
2. 