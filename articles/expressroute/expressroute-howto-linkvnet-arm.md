---
title: "ExpressRoute 回線への仮想ネットワークのリンク: PowerShell: Azure | Microsoft Docs"
description: "このドキュメントでは、Resource Manager デプロイメント モデルと PowerShell を使用して ExpressRoute 回線に仮想ネットワーク (VNet) をリンクする方法の概要について説明します。"
services: expressroute
documentationcenter: na
author: ganesr
manager: timlt
editor: 
tags: azure-resource-manager
ms.assetid: daacb6e5-705a-456f-9a03-c4fc3f8c1f7e
ms.service: expressroute
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/05/2017
ms.author: ganesr
ms.translationtype: HT
ms.sourcegitcommit: 54774252780bd4c7627681d805f498909f171857
ms.openlocfilehash: 8c2f3036f754a98090ab860f95900416690ebf83
ms.contentlocale: ja-jp
ms.lasthandoff: 07/28/2017

---
# <a name="connect-a-virtual-network-to-an-expressroute-circuit"></a>ExpressRoute 回線に仮想ネットワークを接続する
> [!div class="op_single_selector"]
> * [Azure ポータル](expressroute-howto-linkvnet-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-linkvnet-arm.md)
> * [Azure CLI](howto-linkvnet-cli.md)
> * [ビデオ - Azure Portal](http://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-a-connection-between-your-vpn-gateway-and-expressroute-circuit)
> * [PowerShell (クラシック)](expressroute-howto-linkvnet-classic.md)
>

この記事では、Resource Manager デプロイメント モデルと PowerShell を使用して Azure ExpressRoute 回線に仮想ネットワーク (VNet) をリンクする方法について説明します。 仮想ネットワークは、同じサブスクリプションにあっても、別のサブスクリプションの一部であってもかいまいません。 この記事では、仮想ネットワークのリンクを更新する方法も示します。 

## <a name="before-you-begin"></a>開始する前に
* Azure PowerShell モジュールの最新バージョンをインストールします。 詳細については、「 [Azure PowerShell のインストールと構成の方法](/powershell/azure/overview)」を参照してください。
* 構成を開始する前に、[前提条件](expressroute-prerequisites.md)、[ルーティングの要件](expressroute-routing.md)、[ワークフロー](expressroute-workflows.md)を確認します。
* アクティブな ExpressRoute 回線が必要です。 
  * 手順に従って、 [ExpressRoute 回線を作成](expressroute-howto-circuit-arm.md) し、接続プロバイダー経由で回線を有効にしてください。 
  * 回線用に Azure プライベート ピアリングが構成されていることを確認してください。 ルーティング手順については、 [ルーティングの構成](expressroute-howto-routing-arm.md) に関する記事を参照してください。 
  * Azure プライベート ピアリングが構成されていることを確認します。また、エンド ツー エンド接続を有効にできるように、ネットワークと Microsoft の間の BGP ピアリングを起動しておく必要があります。
  * 仮想ネットワークと仮想ネットワーク ゲートウェイを作成し、完全にプロビジョニングします。 指示に従って [ExpressRoute 用の仮想ネットワーク ゲートウェイを作成](expressroute-howto-add-gateway-resource-manager.md)します。 ExpressRoute 用の仮想ネットワーク ゲートウェイは、VPN ではなく GatewayType 'ExpressRoute' を使用します。

* 最大 10 個の仮想ネットワークを標準 ExpressRoute 回線に接続できます。 標準 ExpressRoute 回線を使用する場合は、すべての仮想ネットワークが同じ地理的リージョンに存在する必要があります。 

* ExpressRoute Premium アドオンを有効にした場合は、ExpressRoute 回線の地理的リージョンの外部にある仮想ネットワークをリンクしたり、さらに多くの仮想ネットワークを ExpressRoute 回線にリンクしたりすることができます。 Premium アドオンの詳細については、 [FAQ](expressroute-faqs.md) を確認してください。


## <a name="connect-a-virtual-network-in-the-same-subscription-to-a-circuit"></a>同じサブスクリプション内の仮想ネットワークを回線に接続する
次のコマンドレットを使用し、ExpressRoute 回線に仮想ネットワーク ゲートウェイを接続できます。 コマンドレットを実行する前に、仮想ネットワーク ゲートウェイが作成されており、リンクの準備ができていることを確認してください。

```powershell
$circuit = Get-AzureRmExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
$gw = Get-AzureRmVirtualNetworkGateway -Name "ExpressRouteGw" -ResourceGroupName "MyRG"
$connection = New-AzureRmVirtualNetworkGatewayConnection -Name "ERConnection" -ResourceGroupName "MyRG" -Location "East US" -VirtualNetworkGateway1 $gw -PeerId $circuit.Id -ConnectionType ExpressRoute
```

## <a name="connect-a-virtual-network-in-a-different-subscription-to-a-circuit"></a>別のサブスクリプション内の仮想ネットワークを回線に接続する
複数のサブスクリプションで ExpressRoute 回線を共有できます。 下図に、複数のサブスクリプションで ExpressRoute 回線を共有するしくみについて概略を示します。

大規模クラウド内のそれぞれの小規模クラウドは、組織内のさまざまな部門に属するサブスクリプションを表すために使用されています。 組織内の各部門はサービスのデプロイ用に固有のサブスクリプションを使用できますが、1 つの ExpressRoute 回線を共有することで、オンプレミス ネットワークに接続し直すことができます。 1 つの部門 (この例では IT) で ExpressRoute 回線を所有できます。 組織内の他のサブスクリプションも ExpressRoute 回線を使用できます。

> [!NOTE]
> ExpressRoute 回線の接続と帯域幅の料金は、サブスクリプションの所有者が負担することになります。 すべての仮想ネットワークが同じ帯域幅を共有します。
> 
> 

![サブスクリプション間接続](./media/expressroute-howto-linkvnet-classic/cross-subscription.png)


### <a name="administration---circuit-owners-and-circuit-users"></a>管理 - 回線所有者と回線ユーザー

"回線所有者" は、ExpressRoute 回線リソースの権限のあるパワー ユーザーです。 回線所有者は、"回線ユーザー" が利用できる承認を作成できます。 回線ユーザーは、ExpressRoute 回線と同じサブスクリプション内にない仮想ネットワーク ゲートウェイの所有者です。 回線ユーザーは、承認を利用できます (仮想ネットワークごとに 1 つの承認)。

回線所有者は、承認をいつでも変更し、取り消す権限を持っています。 承認を取り消すと、アクセスが取り消されたサブスクリプションからすべてのリンク接続が削除されます。

### <a name="circuit-owner-operations"></a>回線所有者の操作

**承認を作成するには**

回線所有者は、承認を作成します。 その結果、回線ユーザーが各自の仮想ネットワーク ゲートウェイを ExpressRoute 回線に接続するために使用できる承認キーが作成されます。 承認は、1 つの接続に対してのみ有効です。

次のコマンドレット スニペットは、承認を作成する方法を示しています。

```powershell
$circuit = Get-AzureRmExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
Add-AzureRmExpressRouteCircuitAuthorization -ExpressRouteCircuit $circuit -Name "MyAuthorization1"
Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $circuit

$circuit = Get-AzureRmExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
$auth1 = Get-AzureRmExpressRouteCircuitAuthorization -ExpressRouteCircuit $circuit -Name "MyAuthorization1"
```


このコマンドレットの応答に、承認キーと状態が含まれます。

    Name                   : MyAuthorization1
    Id                     : /subscriptions/&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&/resourceGroups/ERCrossSubTestRG/providers/Microsoft.Network/expressRouteCircuits/CrossSubTest/authorizations/MyAuthorization1
    Etag                   : &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&& 
    AuthorizationKey       : ####################################
    AuthorizationUseStatus : Available
    ProvisioningState      : Succeeded



**承認を確認するには**

回線所有者は、次のコマンドレットを実行し、特定の回線で発行されるすべての承認を確認できます。

```powershell
$circuit = Get-AzureRmExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
$authorizations = Get-AzureRmExpressRouteCircuitAuthorization -ExpressRouteCircuit $circuit
```

**承認を追加するには**

回線所有者は、次のコマンドレットを使用して承認を追加できます。

```powershell
$circuit = Get-AzureRmExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
Add-AzureRmExpressRouteCircuitAuthorization -ExpressRouteCircuit $circuit -Name "MyAuthorization2"
Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $circuit

$circuit = Get-AzureRmExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
$authorizations = Get-AzureRmExpressRouteCircuitAuthorization -ExpressRouteCircuit $circuit
```

**承認を削除するには**

回線所有者は、次のコマンドレットを実行して、ユーザーに対する承認を取り消したり削除したりすることができます。

```powershell
Remove-AzureRmExpressRouteCircuitAuthorization -Name "MyAuthorization2" -ExpressRouteCircuit $circuit
Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $circuit
```    

### <a name="circuit-user-operations"></a>回線ユーザーの操作

回線ユーザーは、ピア ID と回線所有者が作成した承認キーを必要とします。 承認キーは GUID です。

ピア ID は、次のコマンドから確認できます。

```powershell
Get-AzureRmExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
```

**接続の承認を利用するには**

回線ユーザーは次のコマンドレットを実行し、リンク承認を利用できます。

```powershell
$id = "/subscriptions/********************************/resourceGroups/ERCrossSubTestRG/providers/Microsoft.Network/expressRouteCircuits/MyCircuit"    
$gw = Get-AzureRmVirtualNetworkGateway -Name "ExpressRouteGw" -ResourceGroupName "MyRG"
$connection = New-AzureRmVirtualNetworkGatewayConnection -Name "ERConnection" -ResourceGroupName "RemoteResourceGroup" -Location "East US" -VirtualNetworkGateway1 $gw -PeerId $id -ConnectionType ExpressRoute -AuthorizationKey "^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^"
```

**接続の承認を解除するには**

ExpressRoute 回線を仮想ネットワークにリンクしている接続を削除することで、承認を解除できます。

## <a name="modify-a-virtual-network-connection"></a>仮想ネットワーク接続を変更する
仮想ネットワーク接続の特定のプロパティを更新することができます。 

**接続の重みを更新するには**

仮想ネットワークは、複数の ExpressRoute 回線に接続できます。 複数の ExpressRoute 回線から同じプレフィックスを受け取る場合があります。 このプレフィックスを宛先とするトラフィックをどの接続が送信するかを選択するには、接続の *RoutingWeight* を変更します。 *RoutingWeight* が最も高い接続でトラフィックが送信されます。

```powershell
$connection = Get-AzureRmVirtualNetworkGatewayConnection -Name "MyVirtualNetworkConnection" -ResourceGroupName "MyRG"
$connection.RoutingWeight = 100
Set-AzureRmVirtualNetworkGatewayConnection -VirtualNetworkGatewayConnection $connection
```

*RoutingWeight* の範囲は 0 ～ 32000 です。 既定値は 0 です。

## <a name="next-steps"></a>次のステップ
ExpressRoute の詳細については、「 [ExpressRoute のFAQ](expressroute-faqs.md)」をご覧ください。

