---
title: "アクティビティ ログ アラートの作成 | Microsoft Docs"
description: "アクティビティ ログで特定のイベントが発生した場合に、SMS、webhook、および電子メールで通知を受け取ります。"
author: johnkemnetz
manager: orenr
editor: 
services: monitoring-and-diagnostics
documentationcenter: monitoring-and-diagnostics
ms.assetid: 
ms.service: monitoring-and-diagnostics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/03/2017
ms.author: johnkem
ms.translationtype: HT
ms.sourcegitcommit: 25e4506cc2331ee016b8b365c2e1677424cf4992
ms.openlocfilehash: 3885469ec0e1fcc31386dd0ad7fe6cb5d03ab28e
ms.contentlocale: ja-jp
ms.lasthandoff: 08/24/2017

---
# <a name="create-activity-log-alerts"></a>アクティビティ ログ アラートの作成

## <a name="overview"></a>概要
アクティビティ ログ アラートは、アラートに指定した条件と一致する新しいアクティビティのログ イベントが発生したときにアクティブになるアラートです。 これらは Azure リソースであり、Azure Resource Manager テンプレートを使用して作成できます。 これらは、Azure Portal で作成、更新、削除することもできます。 この記事では、アクティビティ ログ アラートの背後の概念について説明します。 その後、Azure Portal を使用してアクティビティ ログのイベントにアラートを設定する方法について説明します。

通常、アクティビティ ログ アラートを作成して、通知を受け取るのは次の場合です。

* Azure サブスクリプションでリソースに特定の変更が発生した場合。多くの場合、特定のリソース グループまたはリソースを対象とします。 たとえば、myProductionResourceGroup 内の仮想マシンが削除されたときに通知を受け取ることができます。 また、サブスクリプション内のユーザーに新しい役割が割り当てられた場合に通知を受け取ることもできます。
* サービス正常性イベントが発生した場合。 サービス正常性イベントには、サブスクリプション内のリソースに適用されるインシデント イベントとメンテナンス イベントの通知が含まれます。

いずれの場合でも、アクティビティ ログ アラートでは、アラートが作成されたサブスクリプション内のイベントのみが監視されます。

JSON オブジェクトの任意の最上位プロパティに基づいて、アクティビティ ログ イベントのアクティビティ ログ アラートを構成できます。 ただし、ポータルには最も一般的なオプションが表示されます。

- **[カテゴリ]**: [管理]、[サービス正常性]、[自動スケール]、[推奨]。 詳細については、[Azre アクティビティ ログの概要](./monitoring-overview-activity-logs.md#categories-in-the-activity-log)に関する記事を参照してください。 サービス正常性イベントについて詳しくは、[サービス通知のアクティビティ ログ アラートの受け取り](./monitoring-activity-log-alerts-on-service-notifications.md)に関する記事をご覧ください。
- **[リソース グループ]**
- **リソース**
- **[リソースの種類]**
- **[操作名]**: Resource Manager のロールベースのアクセス制御の操作名。
- **[レベル]**: イベントの重大度レベル ([詳細]、[情報]、[警告]、[エラー]、[重大])。
- **[状態]**: イベントの状態 (通常は [開始]、[失敗]、または [成功])。
- **[イベント開始者]**: "呼び出し元" とも呼ばれます。 操作を実行したユーザーの電子メール アドレスまたは Azure Active Directory 識別子。

>[!NOTE]
>アラートには、上記の条件の 2 つ以上 (うち 1 つはカテゴリ) を指定する必要があります。 アクティビティ ログ内にイベントが作成されるたびにアクティブ化するアラートを作成することはできません。
>
>

アクティビティ ログ アラートがアクティブになると、アクションまたは通知の生成にアクション グループが使用されます。 アクション グループは、通知レシーバー (電子メール アドレス、webhook URL、または SMS の電話番号) の再利用可能なセットです。 これらのレシーバーは、複数のアラートから参照でき、通知チャネルの一元管理とグループ化を行うことができます。 アクティビティ ログ アラートを定義するときには 2 つの方法があります。 次のようにすることができます。

* アクティビティ ログ アラートで既存のアクション グループを使用します。 
* 新しいアクション グループを作成できます。 

アクション グループの詳細については、「[Azure Portal でのアクション グループの作成および管理](monitoring-action-groups.md)」を参照してください。

サービス正常性通知について詳しくは、[サービス正常性通知のアクティビティ ログ アラートの受け取り](monitoring-activity-log-alerts-on-service-notifications.md)に関する記事をご覧ください。

## <a name="create-an-alert-on-an-activity-log-event-with-a-new-action-group-by-using-the-azure-portal"></a>Azure Portal を使用した新しいアクション グループによるアクティビティ ログ イベントのアラートの作成
1. [ポータル](https://portal.azure.com)で、**[モニター]** を選択します。

    ![[モニター] サービス](./media/monitoring-activity-log-alerts/home-monitor.png)
2. **[アクティビティ ログ]** セクションで、**[アラート]** を選択します。

    ![[アラート] タブ](./media/monitoring-activity-log-alerts/alerts-blades.png)
3. **[アクティビティ ログ アラートの追加]** を選択し、各フィールドに入力します。

4. **[アクティビティ ログ アラート名]** ボックスに名前を入力し、**[説明]** を入力します。

    ![[アクティビティ ログ アラートの追加] コマンド](./media/monitoring-activity-log-alerts/add-activity-log-alert.png)

5. **[サブスクリプション]** ボックスには、現在のサブスクリプションが自動入力されます。 このサブスクリプションにアクション グループが保存されます。 アラート リソースがこのサブスクリプションにデプロイされ、そのアクティビティ ログ イベントを監視します。

    ![[アクティビティ ログ アラートの追加] ダイアログ ボックス](./media/monitoring-activity-log-alerts/activity-log-alert-new-action-group.png)

6. アラート リソースを作成する **[リソース グループ]** を選択します。 これは、アラートによって監視されるリソース グループではありません。 アラート リソースが配置されるリソース グループです。

7. 必要に応じて **[イベント カテゴリ]** を選択して、表示される追加のフィルターを変更します。 管理イベントの場合、フィルターには **[リソース グループ]**、**[リソース]**、**[リソースの種類]**、**[操作名]**、**[レベル]**、**[状態]**、および **[イベント開始者]** があります。 これらの値を使用して、このアラートで監視するイベントを識別します。

    >[!NOTE]
    >アラートでは、上記の条件の少なくとも 1 つを指定する必要があります。 アクティビティ ログ内にイベントが作成されるたびにアクティブ化するアラートを作成することはできません。
    >
    >

8. **[アクション グループ名]** ボックスおよび **[短い名前]** ボックスに名前を入力します。 短い名前は、通知がこのグループを使用して送信されるときに長い名前の代わりに使用されます。

9.  アクションについての次の情報を指定して、アクションの一覧を定義します。

    a. **[名前]**: アクションの名前、別名、または識別子を入力します。

    b. **[アクションの種類]**: SMS、電子メール、または webhook を選択します。

    c. **[詳細]**: アクションの種類に基づいて、電話番号、電子メール アドレス、または webhook の URI を入力します。

10. **[OK]** を選択してアラートを作成します。

アラートが完全に反映されてアクティブになるまで数分かかります。 アラートは、新しいイベントがアラート条件に一致するとトリガーされます。

詳しくは、[アクティビティ ログ アラートで使用される webhook スキーマの理解](monitoring-activity-log-alerts-webhook.md)に関する記事をご覧ください。

>[!NOTE]
>上記の手順で定義されたアクション グループは、今後すべてのアラート定義で既存のアクション グループとして再利用できます。
>
>

## <a name="create-an-alert-on-an-activity-log-event-for-an-existing-action-group-by-using-the-azure-portal"></a>Azure Portal を使用した既存のアクション グループのアクティビティ ログ イベントに対するアラートの作成
1. 前のセクションの手順 1 から 7 に従って、アクティビティ ログ アラートを作成します。

2. **[通知手段]** で、**[既存]** アクション グループ ボタンを選択します。 一覧から既存のアクション グループを選択します。

3. **[OK]** を選択してアラートを作成します。

アラートが完全に反映されてアクティブになるまで数分かかります。 アラートは、新しいイベントがアラート条件に一致するとトリガーされます。

## <a name="manage-your-alerts"></a>アラートの管理

アラートを作成すると、[モニター] ブレードの [アラート] セクションにアラートが表示されます。 次の操作を行うために管理するアラートを選択します。

* 編集する。
* 削除する。
* 無効または有効にしてそのアラートの通知受信を一時的に停止または再開する。

## <a name="next-steps"></a>次のステップ
- [アラートの概要](monitoring-overview-alerts.md)について把握します。
- [通知のレート制限](monitoring-alerts-rate-limiting.md)について学習します。
- [アクティビティ ログ アラート webhook スキーマ](monitoring-activity-log-alerts-webhook.md)を確認します。
- [アクション グループ](monitoring-action-groups.md)の詳細について学習します。  
- [サービス正常性の通知](monitoring-service-notifications.md)について学習します。
- [アクティビティ ログ アラートを作成して、サブスクリプションで自動スケールのエンジン操作をすべて監視します](https://github.com/Azure/azure-quickstart-templates/tree/master/monitor-autoscale-alert)。
- [アクティビティ ログ アラートを作成して、サブスクリプションで失敗した自動スケールのスケールイン/スケールアウト操作をすべて監視します](https://github.com/Azure/azure-quickstart-templates/tree/master/monitor-autoscale-failed-alert)。

