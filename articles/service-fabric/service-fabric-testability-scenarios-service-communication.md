---
title: "Testability: サービス通信 | Microsoft Docs"
description: "サービス間通信は、Service Fabric アプリケーションの重要な統合ポイントです。 この記事では、設計の考慮事項とテスト手法について説明します。"
services: service-fabric
documentationcenter: .net
author: vturecek
manager: timlt
editor: 
ms.assetid: 017557df-fb59-4e4a-a65d-2732f29255b8
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 06/29/2017
ms.author: vturecek
ms.translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 6356b4e69892b90ff74aa9db0157930dc00f4a08
ms.contentlocale: ja-jp
ms.lasthandoff: 11/17/2016


---
# Service Fabric Testability シナリオ: サービス通信
<a id="service-fabric-testability-scenarios-service-communication" class="xliff"></a>
マイクロサービスおよびサービス指向アーキテクチャ スタイルは、Azure Service Fabric に無理なく適用できます。 このような種類の分散アーキテクチャでは、コンポーネント化されたマイクロサービス アプリケーションは、相互に通信する必要がある複数のサービスで構成されることが一般的です。 一般的に、最も単純な場合でも、相互に通信する必要があるステートレス Web サービスとステートフル データ ストレージ サービスが最低限必要です。

各サービスは他のサービスに対してリモート API を公開するため、サービス間の通信はアプリケーションの重要な統合ポイントです。 I/O を伴う一連の API 境界を操作するには、一般的に多少の注意と、テストと検証を十分に行う必要があります。

分散システムでこれらのサービス境界を接続するときには、さまざまな考慮事項があります。

* *トランスポート プロトコル*。 相互運用性を高める HTTP を使用するか、スループットを最大化するカスタム バイナリ プロトコルを使用するか。
* *エラー処理*。 永続的なエラーと一時的なエラーをどのように処理するか。 サービスが異なるノードに移動した場合は何が起こるか。
* *タイムアウトと待機時間*。 多層アプリケーションで、各サービス レイヤーがスタックからユーザーまでの待機時間をどのように処理するか。

Service Fabric によって提供されるいずれかの組み込みサービス通信コンポーネントを使用する場合も、独自のコンポーネントを構築する場合も、サービス間の対話をテストすることは、アプリケーションの回復性を確保するために不可欠です。

## サービスを移動する準備をする
<a id="prepare-for-services-to-move" class="xliff"></a>
サービス インスタンスは、時間の経過と共に移動することがあります。 これは、カスタム調整された最適なリソース分散のためのロード メトリックが構成されている場合に特に当てはまります。 Service Fabric は、アップグレード、フェールオーバー、スケールアウト、および分散システムの有効期間に発生するその他の状況下でも、サービス インスタンスを移動することでその可用性を最大化します。

サービスはクラスター全体を移動するため、クライアントやその他サービスは、サービスとの通信時に対応できるように、2 つのシナリオに合わせて準備する必要があります。

* サービス インスタンスまたはパーティション レプリカが、前回の通信時から移動している場合。 これはサービスのライフサイクルの正常な処理であり、アプリケーションの有効期間中に発生することが予想されます。
* サービス インスタンスまたはパーティション レプリカが移動の処理中である場合。 1 つのノードから別のノードへのサービスのフェールオーバーは Service Fabric で非常に高速に行われますが、通信コンポーネントの開始に時間がかかると、可用性にかかわる遅延が発生する場合があります。

これらのシナリオを正しく処理することは、システムのスムーズな実行にとって重要です。 これを行うには、次の点に留意します。

* 接続できるすべてのサービスには、リッスンする*アドレス* (HTTP や WebSocket など) があります。 サービス インスタンスまたはパーティションが移動すると、そのアドレス エンドポイントが変わります  (別の IP アドレスを持つ別のノードに移動します)。組み込み通信コンポーネントを使用すると、コンポーネントは再解決するサービスのアドレスを処理します。
* サービス インスタンスがリスナーを再度開始するときに、サービスの待機時間が一時的に増加する可能性があります。 これは、サービス インスタンスが移動された後で、サービスがどれだけ迅速に開くかによって決まります。
* 既存の接続を閉じ、新しいノードでサービスが開いた後で再度開く必要があります。 グレースフルなノードのシャットダウンまたは再起動では、既存の接続を正常にシャットダウンするための時間が与えられます。

### テスト: サービス インスタンスの移動
<a id="test-it-move-service-instances" class="xliff"></a>
Service Fabric の Testability ツールを使用して、このような状況をさまざまな方法でテストするためのテスト シナリオを作成できます。

1. ステートフル サービスのプライマリ レプリカを移動します。
   
    ステートフル サービス パーティションのプライマリ レプリカは、さまざまな理由で移動できます。 これを使用して、特定のパーティションのプライマリ レプリカをターゲットとし、サービスが移動にどのように反応するかを高度に制御された方法で確認します。
   
    ```powershell
   
    PS > Move-ServiceFabricPrimaryReplica -PartitionId 6faa4ffa-521a-44e9-8351-dfca0f7e0466 -ServiceName fabric:/MyApplication/MyService
   
    ```
2. ノードを停止します。
   
    ノードが停止すると、そのノード上にあったすべてのサービス インスタンスやパーティションが、Service Fabric によって、クラスター内の他の使用可能なノードのいずれかに移されます。 このシナリオを使用して、ノードがクラスターから失われ、そのノード上のすべてのサービス インスタンスとレプリカを移動する必要がある状況をテストできます。
   
    ノードは、PowerShell の **Stop-ServiceFabricNode** コマンドレットを使用して停止できます。
   
    ```powershell
   
    PS > Restart-ServiceFabricNode -NodeName Node_1
   
    ```

## サービスの可用性の維持
<a id="maintain-service-availability" class="xliff"></a>
プラットフォームとして、Service Fabric は、サービスの高可用性を実現するように設計されています。 ただし、極端な場合は、基盤となるインフラストラクチャの問題が原因で、可用性が失われる可能性があります。 これらのシナリオをテストすることも重要です。

ステートフル サービスは、クォーラム ベースのシステムを使用して、高可用性の状態をレプリケートします。 つまり、書き込み操作を実行するためには、レプリカのクォーラムを利用できる必要があります。 広範囲にわたるハードウェアの障害などのまれなケースで、レプリカのクォーラムを使用できない場合があります。 この場合、書き込み操作を実行することはできませんが、読み取り操作は実行できます。

### テスト: 書き込み操作を実行できない
<a id="test-it-write-operation-unavailability" class="xliff"></a>
Service Fabric の Testability ツールを使用して、テストとしてクォーラム損失を誘発させる障害を注入できます。 そのようなシナリオはめったにありませんですが、ステートフル サービスに依存するクライアントとサービスは、ステートフル サービスに書き込み要求ができない状況に対処できるように準備しておくことが重要です。 ステートフル サービス自体がこのような可能性を認識し、呼び出し元に対してグレースフルな通知を行えることも同様に重要です。

クォーラム損失は、PowerShell の **Invoke-ServiceFabricPartitionQuorumLoss** コマンドレットを使用して誘発させることができます。

```powershell

PS > Invoke-ServiceFabricPartitionQuorumLoss -ServiceName fabric:/Myapplication/MyService -QuorumLossMode QuorumReplicas -QuorumLossDurationInSeconds 20

```

この例では、`QuorumLossMode` を `QuorumReplicas` に設定して、すべてのレプリカを停止させることなくクォーラム損失を誘発させることを指示します。 これにより、読み取り操作は引き続き可能です。 パーティション全体が使用できない状況をテストする場合は、このスイッチを `AllReplicas`に設定することができます。

## 次のステップ
<a id="next-steps" class="xliff"></a>
[Testability アクションの詳細](service-fabric-testability-actions.md)

[Testability シナリオの詳細](service-fabric-testability-scenarios.md)


