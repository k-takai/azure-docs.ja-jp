---
title: "Azure SQL Database を使用するマルチテナント アプリで新しいテナントをプロビジョニングする | Microsoft Docs"
description: "Azure SQL Database のマルチテナント SaaS アプリで新しいテナントをプロビジョニングしてカタログ化する方法について説明します。"
keywords: "SQL データベース チュートリアル"
services: sql-database
documentationcenter: 
author: stevestein
manager: craigg
editor: 
ms.assetid: 
ms.service: sql-database
ms.custom: scale out apps
ms.workload: data-management
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/11/2017
ms.author: sstein
ms.translationtype: HT
ms.sourcegitcommit: 2c6cf0eff812b12ad852e1434e7adf42c5eb7422
ms.openlocfilehash: febf9d9f8a32a95ae61734cc2d3632bb74f6c8f5
ms.contentlocale: ja-jp
ms.lasthandoff: 09/13/2017

---
# <a name="learn-how-to-provision-and-catalog-new-tenants-and-register-them-in-the-catalog"></a>新しいテナントをプロビジョニングしてカタログ化し、カタログに登録する方法について説明します。

このチュートリアルでは、SaaS パターンのプロビジョニングおよびカタログと、Wingtip SaaS アプリケーションでの実装方法について学びます。 新規のテナント データベースを作成して初期化し、アプリケーションのテナント カタログに登録します。 カタログ とは、SaaS アプリケーションの多数のテナントとそのデータの間のマッピング情報を保持するためのデータベースを指します。 カタログは、アプリケーションの要求を正しいデータベースに転送するうえで重要な役割を果たします。  

このチュートリアルで学習する内容は次のとおりです。

> [!div class="checklist"]

> * 1 つの新しいテナントをプロビジョニングする方法 (この実装方法の手順の確認を含む)
> * バッチを使って追加のテナントをプロビジョニングする方法


このチュートリアルを完了するには、次の前提条件を満たしておく必要があります。

* Wingtip SaaS アプリがデプロイされている。 5 分未満でデプロイするには、[Wingtip SaaS アプリケーションのデプロイと確認](sql-database-saas-tutorial.md)に関するページを参照してください。
* Azure PowerShell がインストールされている。 詳細については、「[Azure PowerShell を使ってみる](https://docs.microsoft.com/powershell/azure/get-started-azureps)」をご覧ください。

## <a name="introduction-to-the-saas-catalog-pattern"></a>SaaS カタログ パターンの概要

データベースを基盤とするマルチテナント SaaS アプリケーションでは、テナントの情報が格納される場所を把握する必要があります。 SaaS カタログ パターンでは、各テナントとデータの格納場所との間のマッピングに関する情報を格納するものとしてカタログ データベースを使用します。 Wingtip SaaS アプリで採用されているのは、"データベースごとに 1 つのテナント" アーキテクチャです。もっとも、マルチテナントとシングルテナントのどちらのデータベースを使っている場合でも、テナントとデータベースとの間のマッピング情報をカタログに記録することに関しては、共通の基本パターンが適用されます。

各テナントには、カタログ内の情報を識別するためのキーが割り当てられ、適切なデータベースの場所にマップされます。 Wingtip SaaS アプリでは、このキーがテナント名のハッシュ値を使って生成されます。 これにより、アプリケーションの URL のテナント名の部分を使ってキーを作成できます。 他のテナント キー スキームを使用できます。  

カタログによって、アプリケーションに対する影響を最小限に抑えて、データベースの名前と場所を変更できます。  マルチ テナント データベース モデルでは、カタログはデータベース間のテナントの移動も可能にします。  また、テナントやデータベースがメンテナンスやその他の措置のためにオフラインになっているかどうかを示すためにも、カタログを使用できます。 詳しくは、[シングル テナントの復元に関するチュートリアル](sql-database-saas-tutorial-restore-single-tenant.md)をご覧ください。

さらに、SaaS アプリケーションの管理データベースが有効になっているカタログでは、データベースの層やエディション、スキーマ バージョン、サービス プラン、テナントに提供された SLA のほか、アプリケーション管理、カスタマー サポート、または DevOps プロセスを有効にするその他の情報など、追加のテナントまたはデータベース メタデータを保存できます。  

SaaS アプリケーション以上に、カタログはデータベース ツールを有効にできます。  Wingtip SaaS サンプルでは、カタログはテナント間クエリを有効にするために使用されます。[ad hoc 分析のチュートリアル](sql-database-saas-tutorial-adhoc-analytics.md)をご覧ください。 データベース間のジョブ管理については、[スキーマ管理](sql-database-saas-tutorial-schema-management.md)および[テナント分析](sql-database-saas-tutorial-tenant-analytics.md)のチュートリアルをご覧ください。 

Wingtip SaaS アプリでは、[Elastic Database クライアント ライブラリ (EDCL)](sql-database-elastic-database-client-library.md) のシャード管理機能を使って、カタログが実装されています。 EDCL により、アプリケーションは、データベースに支援されたシャード マップを作成、管理、使用できます。 シャード マップにはシャード (データベース) の一覧と、キー (テナント) およびデータベース間のマッピングが含まれます。  EDCL 関数は、テナント プロビジョニング時にアプリケーションまたは PowerShell スクリプトから使用してシャード マップのエントリを作成したり、アプリケーションから使用して適切なデータベースに効率的に接続したりできます。 EDCL は接続情報をキャッシュして、カタログ データベースへのトラフィックを最小限に抑え、アプリケーションを高速化します。  

> [!IMPORTANT]
> このマッピング データは、カタログ データベースからアクセスできるものの、"*編集することはできません*"。 マッピング データの編集には、必ずエラスティック データベース クライアント ライブラリ API を使用してください。 マッピング データを直接操作することは、カタログが破損するおそれがあるため、サポートしていません。


## <a name="introduction-to-the-saas-provisioning-pattern"></a>SaaS プロビジョニング パターンの概要

シングル テナント データベースを使用する SaaS アプリケーションに新しいテナントをオンボードする場合、新しいテナント データベースがプロビジョニングされる必要があります。  テナント データベースは、適切な場所とサービス層に作成され、適切なスキーマと参照データを使って初期化され、適切なテナント キー下のカタログに登録される必要があります。  

データベース プロビジョニングには、SQL スクリプトの実行、BACPAC の展開、ゴールデン テンプレート データベースのコピーなど、別の方法も使用できます。  

使用するプロビジョニング方法は、全体のスキーマ管理戦略に組み込まれる必要があります。戦略では、新しいデータベースが最新のスキーマでプロビジョニングされることを保証する必要があります。  詳しくは、[スキーマ管理のチュートリアル](sql-database-saas-tutorial-schema-management.md)をご覧ください。  

Wingtip SaaS アプリは、カタログ サーバーに展開されている、basetenantdb という名前のゴールデン データベースをコピーすることで、新しいテナントをプロビジョニングします。  プロビジョニングは、サインアップ エクスペリエンスや、スクリプトを使用してサポートされるオフラインの一部として、アプリケーションに統合できます。 このチュートリアルでは、PowerShell を使ったプロビジョニングについて説明します。 プロビジョニング スクリプトは、basetenantdb をコピーしてエラスティック プールに新しいテナント データベースを作成し、テナント固有情報を使ってこのデータベースを初期化して、カタログ シャード マップに登録します。  サンプル アプリでは、データベースはテナント名に基づいた名前を付与されますが、これは、パターンの主要な部分ではありません。カタログを使用すると、任意の名前をデータベースに割り当てることができます。 


## <a name="get-the-wingtip-application-scripts"></a>Wingtip アプリケーションのスクリプトを取得する

Wingtip SaaS のスクリプトとアプリケーション ソース コードは、[WingtipSaaS](https://github.com/Microsoft/WingtipSaaS) GitHub リポジトリから入手できます。 [Wingtip SaaS のスクリプトをダウンロードする手順](sql-database-wtp-overview.md#download-and-unblock-the-wingtip-saas-scripts)。


## <a name="provision-and-catalog-detailed-walkthrough"></a>プロビジョニングとカタログ登録の詳細手順

Wingtip アプリケーションで新しいテナント プロビジョニングを実装する方法を理解するには、ブレークポイントを追加して、テナントをプロビジョニングしているときのワークフローを確認します。

1. ...\\Learning Modules\\ProvisionAndCatalog\\_Demo-ProvisionAndCatalog.ps1_ を開いて、次のパラメーターを設定します。
   * **$TenantName** = 新しい会場の名称 (たとえば、*Bushwillow Blues*)。
   * **$VenueType** = 事前に定義しておいた会場の種類のいずれか (*ブルース*、クラシック、ダンス、ジャズ、柔道、モーターレース、多目的、オペラ、ロック、サッカー)。
   * **$DemoScenario** = **1**。"*シングル テナントのプロビジョニング*" を行う場合は **1** に設定します。

1. ブレークポイントを追加します。これには、48 行目 (*New-Tenant `* と記述されている行) のどこかにカーソルを置いて、**F9** キーを押します。

   ![ブレーク ポイント](media/sql-database-saas-tutorial-provision-and-catalog/breakpoint.png)

1. **F5** キーを押して、スクリプトを実行します。

1. ブレークポイントでスクリプトの実行が停止したら、**F11** キーを押してコードにステップ インします。

   ![ブレーク ポイント](media/sql-database-saas-tutorial-provision-and-catalog/debug.png)



呼び出された関数にステップ オーバーしたりステップ インしたりする **F10** キーや **F11** キーなどの**デバッグ**のメニュー オプションを使用して、スクリプトの実行をトレースします。 PowerShell スクリプトのデバッグの詳細については、「[PowerShell スクリプトの使用とデバッグに関するヒント](https://msdn.microsoft.com/powershell/scripting/core-powershell/ise/how-to-debug-scripts-in-windows-powershell-ise)」を参照してください。


次の記述は、明示的に従う手順ではなく、スクリプトのデバッグ中にたどるワークフローの説明です。

1. **SubscriptionManagement.psm1 モジュールのインポート**。このモジュールには、Azure にサインインして、使用する Azure サブスクリプションを選択するための関数が含まれています。
1. **CatalogAndDatabaseManagement.psm1 モジュールのインポート**。このモジュールは、カタログとテナントレベルで[シャード管理](sql-database-elastic-scale-shard-map-management.md)関数を抽象化するものです。 このモジュールは、カタログ パターンの大部分をカプセル化するという点で重要であるため、確認する価値があります。
1. **構成情報の取得**。 (F11 キーを押して) Get-Configuration にステップ インして、アプリの構成がどのように指定されているかを確認します。 リソース名などのアプリ固有の値は、ここで定義されています。スクリプトに慣れていないうちは、ここにある値を変更しないでください。
1. **カタログ オブジェクトの取得**。 より高いレベルのスクリプトで使用されるカタログ オブジェクトを構成して返す Get-Catalog にステップ インします。  この関数では、**AzureShardManagement.psm1**からインポートされたシャード管理関数を使用します。 カタログ オブジェクトは、以下で構成されています。
   * $catalogServerFullyQualifiedName。これは、標準的なステム部分とユーザー名を組み合わせた構成になっています (例: _catalog-\<ユーザー\>.database.windows.net_)。
   * $catalogDatabaseName。これは、構成 *tenantcatalog* から取得されます。
   * $shardMapManager オブジェクト。これは、カタログ データベースから初期化されます。
   * $shardMap オブジェクト。これは、カタログ データベースの *tenantcatalog* シャード マップから初期化されます。
   高度なスクリプトでは、カタログ オブジェクトが作成されて返され、使用されます。
1. **新しいテナント キーの計算**。 ハッシュ関数を使用して、テナント名からテナント キーが作成されます。
1. **テナント キーが既に存在するかどうかの確認**。 カタログにキーがあることを確認します。
1. **New-TenantDatabase によるテナント データベースのプロビジョニング。** **F11** キーを使ってステップ インして、[Azure Resource Manager テンプレート](../azure-resource-manager/resource-manager-template-walkthrough.md)によってデータベースがどのようにプロビジョニングされているかを確認します。

データベース名は、どのシャードがどのテナントに属しているかが明確になるよう、テナント名を基に作成されます  (もっとも、データベースに名前を付ける戦略としては、これ以外のものも簡単に使用できます)。また、カタログ サーバーでゴールデン データベース (baseTenantDB) をコピーしてテナント データベースを作成するには、Resource Manager テンプレートを使用します。 別の方法としては、空のデータベースを作成してから、bacpac をインポートして初期化するというやり方もあります。または、既知の場所から初期化スクリプトを実行します。  

Resource Manager テンプレートは、…\Learning Modules\Common\ フォルダーにある *tenantdatabasecopytemplate.json* ファイルです。

テナント データベースが作成されると、**会場 (テナント) の名前と会場の種類の情報でさらに初期化されます**。 また、他の初期化もここで実行されます。

**テナント データベースは、***Add-TenantDatabaseToCatalog* により、テナント キーを使ってカタログに登録されます。 **F11** を使用して詳細にステップ インします。

* シャード マップ (既知のデータベースの一覧) にカタログ データベースが追加されます。
* キー値をシャードにリンクさせるマッピングが作成されます。
* テナントに関する追加のメタ データ (会場の名前) が、カタログのテナント テーブルに追加されます。  テナント テーブルは、ShardManagement スキーマの一部ではなく、EDCL ではインストールされません。  このテーブルは、アプリケーション固有の追加データをサポートするようにカタログ データベースを拡張できる仕組みを例示しています。   


プロビジョニングが完了したら、実行が元の *Demo-ProvisionAndCatalog* スクリプトに戻り、ブラウザーで新しいテナントの **[イベント]** ページが開きます。

   ![events](media/sql-database-saas-tutorial-provision-and-catalog/new-tenant.png)


## <a name="provision-a-batch-of-tenants"></a>テナントのバッチをプロビジョニングする

この演習では、17 のテナントのバッチをプロビジョニングします。 他の Wingtip SaaS チュートリアルを開始する前に、このテナント バッチをプロビジョニングして、多くのデータベースを操作できるようにしておくことをお勧めします。

1. *PowerShell ISE* で ...\\Learning Modules\\ProvisionAndCatalog\\*Demo-ProvisionAndCatalog.ps1* を開き、*$DemoScenario* パラメーターを 3 に設定します。
   * **$DemoScenario** = **3**。"*バッチを使ってテナントをプロビジョニングする*" 場合は **3** に変更します。
1. **F5** キーを押して、スクリプトを実行します。

このスクリプトにより、追加のテナントのバッチがデプロイされます。 このスクリプトでは、バッチを制御し、各データベースのプロビジョニングをリンクされているテンプレートに委任する [Azure Resource Manager テンプレート](../azure-resource-manager/resource-manager-template-walkthrough.md)を使用しています。 テンプレートをこのように使用することによって、Azure Resource Manager がスクリプトのプロビジョニングの処理を仲介できるようになっています。 テンプレートでは、データベースを可能な限り並行してプロビジョニングします。また、必要があればプロセス全体を最適化しながら再試行を処理します。 このスクリプトはべき等です。そのため、スクリプトがなんらかの理由で失敗または停止した場合はもう一度実行してください。

### <a name="verify-the-batch-of-tenants-successfully-deployed"></a>テナントのバッチが正常にデプロイされたかどうかの確認

* [Azure Portal](https://portal.azure.com) でサーバーの一覧を参照し、*tenants1* サーバーを開きます。**[SQL データベース]** をクリックし、今回は一覧に 17 の追加データベースのバッチがあることを確認します。

   ![データベースの一覧](media/sql-database-saas-tutorial-provision-and-catalog/database-list.png)



## <a name="other-provisioning-patterns"></a>プロビジョニングのその他のパターン

このチュートリアルでは取り上げた以外のプロビジョニング パターンとしては、以下のものが存在します。

**データベースの事前プロビジョニング**。 事前プロビジョニング パターンでは、エラスティック プール内のデータベースには追加のコストが発生しないという点が利用されています。 課金の対象となるのはデータベースではなくエラスティック プールであり、アイドル状態のデータベースがリソースを消費することはありません。 プールにデータベースを事前にプロビジョニングしておいて、必要になったら割り当てるという方法を使えば、テナントの準備にかかる時間を大幅に削減できます。 事前にプロビジョニングされるデータベースの数は、予想されるプロビジョニング速度に応じて適切なバッファーを維持できるよう、必要に応じて調整されることがあります。

**自動プロビジョニング**。 自動プロビジョニング パターンでは、サーバー、プール、データベースを必要に応じて自動でプロビジョニングするための専用のサービスを使用します (このサービスでは、必要があればエラスティック プール内のデータベースの事前プロビジョニングも可能です)。 データベースの使用を停止したり、削除したりした場合には、このプロビジョニング サービスによりエラスティック プール間の格差が是正されます。 このようなサービスは単純なこともあれば、多くの地域にまたがるプロビジョニングを処理するなどして複雑になることもあります。また、ディザスター リカバリーのための戦略として geo レプリケーションを使用している場合には、サービスにより geo レプリケーションが自動でセットアップされることもあります。 自動プロビジョニングのパターンではまず、クライアント アプリケーションまたはスクリプトからプロビジョニングの要求がキューに送信され、その要求がプロビジョニング サービスによって処理されます。その後、アプリケーションまたはスクリプトからサービスに対して、処理が完了したかどうかを確認するポーリングが実行されます。 事前プロビジョニングを使用した場合には、バックグラウンドで実行している代替データベースをのプロビジョニングを管理するサービスにより、要求がすばやく処理されます。



## <a name="next-steps"></a>次のステップ

このチュートリアルで学習した内容は次のとおりです。

> [!div class="checklist"]

> * 新しいテナントを 1 件プロビジョニングする方法
> * バッチを使って追加のテナントをプロビジョニングする方法
> * テナントをプロビジョニングしてカタログに登録するまでの流れを詳しく確認する方法

[パフォーマンスの監視に関するチュートリアル](sql-database-saas-tutorial-performance-monitoring.md)を試してください。

## <a name="additional-resources"></a>その他のリソース

* [Wingtip SaaS アプリケーションに基づく作業のための追加のチュートリアル](sql-database-wtp-overview.md#sql-database-wingtip-saas-tutorials)
* [エラスティック データベース クライアント ライブラリ](https://docs.microsoft.com/azure/sql-database/sql-database-elastic-database-client-library)
* [Windows PowerShell ISE でスクリプトをデバッグする方法](https://msdn.microsoft.com/powershell/scripting/core-powershell/ise/how-to-debug-scripts-in-windows-powershell-ise)

