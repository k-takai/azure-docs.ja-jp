---
title: "Azure Analysis Services チュートリアルのレッスン 4: リレーションシップを作成する | Microsoft Docs"
description: "この Azure Analysis Services チュートリアル プロジェクトでは、リレーションショップを作成する方法について説明します。"
services: analysis-services
documentationcenter: 
author: Minewiskan
manager: erikre
editor: 
tags: 
ms.assetid: 
ms.service: analysis-services
ms.devlang: NA
ms.topic: get-started-article
ms.tgt_pltfrm: NA
ms.workload: na
ms.date: 05/26/2017
ms.author: owend
ms.translationtype: Human Translation
ms.sourcegitcommit: 43aab8d52e854636f7ea2ff3aae50d7827735cc7
ms.openlocfilehash: d79af3915c718a79f60e5f589527eb4c2ae8b367
ms.contentlocale: ja-jp
ms.lasthandoff: 06/03/2017

---
# <a name="lesson-4-create-relationships"></a>レッスン 4: リレーションシップを作成する

[!INCLUDE[analysis-services-appliesto-aas-sql2017-later](../../../includes/analysis-services-appliesto-aas-sql2017-later.md)]

このレッスンでは、データをインポートしたときに自動的に作成されるリレーションシップを確認し、別のテーブル間に新しいリレーションシップを追加します。 リレーションシップは、テーブル内のデータの関連付け方法を確立する 2 つのテーブル間の接続です。 たとえば、DimProduct テーブルと DimProductSubcategory テーブルは、各製品がサブカテゴリに属しているという事実に基づいてリレーションシップを持っています。 詳細については、[リレーションシップ](https://docs.microsoft.com/sql/analysis-services/tabular-models/relationships-ssas-tabular)を参照してください。
  
このレッスンの推定所要時間: **10 分**  
  
## <a name="prerequisites"></a>前提条件  
このトピックは、表形式モデルのチュートリアルの一部であり、チュートリアルでの順番に従って実行する必要があります。 このレッスンのタスクを実行する前に、前のレッスン「[レッスン 3: 日付テーブルとしてマークする](../tutorials/aas-lesson-3-mark-as-date-table.md)」を完了する必要があります。 
  
## <a name="review-existing-relationships-and-add-new-relationships"></a>既存のリレーションシップの確認と新しいリレーションシップの追加  
Get Data を使用してデータをインポートすると、AdventureWorksDW2014 データベースから 7 つのテーブルが取得されました。 通常、リレーショナル ソースからデータをインポートする際には、既存のリレーションシップがデータと共に自動的にインポートされます。 ただし、モデルのオーサリングを続行する前に、テーブル間のリレーションシップが正しく作成されたかどうかを確認する必要があります。 このチュートリアルでは、3 つの新しいリレーションシップを追加します。  
  
#### <a name="to-review-existing-relationships"></a>既存のリレーションシップを確認するには  
  
1.  **[モデル]** メニュー > **[モデル ビュー]** > **[ダイアグラム ビュー]** をクリックします。  

    モデル デザイナーがダイアグラム ビューに表示されるようになります。これは、インポートしたすべてのテーブルを線で結んでグラフィカルな形式で表示します。 テーブル間の線は、データをインポートしたときに自動的に作成されたリレーションシップを示します。
    
    ![aas-lesson4-diagram](../tutorials/media/aas-lesson4-diagram.png)
  
    モデル デザイナーの右下隅にあるミニマップ コントロールを使用して、できるだけ多くのテーブルを含めるようにします。 テーブルをクリック、ドラッグして別の場所に移動したり、テーブル同士を近づけたり、特定の順序に並べたりすることができます。 テーブルの移動は、テーブル間の既存のリレーションシップには影響しません。 特定のテーブル内のすべての列を表示するには、テーブルの端をクリック、ドラッグして縮小または拡張します。  
  
2.  **DimCustomer** テーブルと **DimGeography** テーブル間の実線をクリックします。 これら 2 つのテーブル間の実線は、このリレーションシップがアクティブであることを示しています。つまり、DAX の数式を計算するときに既定で使用されるということです。  
  
    **DimCustomer** テーブル内の **GeographyKey** 列と **DimGeography** テーブル内の **GeographyKey** 列の両方が、それぞれボックス内に表示されています。 これらの列はリレーションシップで使用されます。 リレーションシップのプロパティも **[プロパティ]** ウィンドウに表示されるようになります。  
  
    > [!TIP]  
    > ダイアグラム ビューでモデル デザイナーを使用するだけでなく、[リレーションシップの管理] ダイアログ ボックスを使用して、テーブル形式ですべてのテーブル間のリレーションシップを表示することもできます。 表形式モデル エクスプローラーで **[リレーションシップ]** > **[リレーションシップの管理]**を右クリックします。
  
3.  AdventureWorksDW データベースから各テーブルがインポートされたときに、次のリレーションシップが作成されたことを確認します。  
  
    |アクティブ|テーブル|関連するルックアップ テーブル|  
    |----------|---------|------------------------|  
    |あり|**DimCustomer [GeographyKey]**|**DimGeography [GeographyKey]**|  
    |あり|**DimProduct [ProductSubcategoryKey]**|**DimProductSubcategory [ProductSubcategoryKey]**|  
    |あり|**DimProductSubcategory [ProductCategoryKey]**|**DimProductCategory [ProductCategoryKey]**|  
    |あり|**FactInternetSales [CustomerKey]**|**DimCustomer [CustomerKey]**|  
    |あり|**FactInternetSales [ProductKey]**|**DimProduct [ProductKey]**|  
  
    リレーションシップのいずれかが存在しない場合は、モデルに DimCustomer、DimDate、DimGeography、DimProduct、DimProductCategory、DimProductSubcategory、FactInternetSales の各表が含まれていることを確認します。 同じデータ ソース接続のテーブルが別々の時期にインポートされた場合、これらのテーブル間のリレーションシップは作成されないので手動で作成する必要があります。  

### <a name="take-a-closer-look"></a>詳細を見る
ダイアグラム ビューでは、テーブル間のリレーションシップを示す実線部分に矢印、アスタリスク、数字があります。

![aas-lesson4-line](../tutorials/media/aas-lesson4-line.png)

矢印は、フィルターの方向を示しています。 アスタリスクは、このテーブルがリレーションシップの基数の多い側であることを示し、1 はこのテーブルがリレーションシップの一方の側であることを示しています。 リレーションシップを編集する必要がある場合 (リレーションシップの方向や基数を変更するなど)、リレーションシップ線をダブルクリックして、[リレーションシップを編集] ダイアログを開きます。

![aas-lesson4-edit](../tutorials/media/aas-lesson4-edit.png)

これらの機能は、高度なデータ モデリング用であり、このチュートリアルの範囲外です。 詳細については、[Analysis Services における表形式モデルの双方向フィルタ](https://docs.microsoft.com/sql/analysis-services/tabular-models/bi-directional-cross-filters-tabular-models-analysis-services)を参照してください。

場合によっては、特定のビジネス ロジックをサポートするために、モデル内のテーブル間に追加でリレーションシップを作成する必要があります。 このチュートリアルでは、FactInternetSales テーブルと DimDate テーブル間に 3 つの追加のリレーションシップを作成する必要があります。  
  
#### <a name="to-add-new-relationships-between-tables"></a>テーブル間に新しいリレーションシップを追加するには  
  
1.  モデル デザイナーの **FactInternetSales** テーブルで、**OrderDate** 列をクリックし、そのままカーソルを **DimDate** テーブル内の **Date** 列にドラッグして離します。  

    **Internet Sales** テーブル内の **OrderDate** 列と **Date** テーブル内の **Date** 列の間に、アクティブなリレーションシップを作成したことを示す実線が現れます。 
  
      ![aas-lesson4-new](../tutorials/media/aas-lesson4-new.png) 
  
    > [!NOTE]  
    > リレーションシップを作成すると、主テーブルと関連するルックアップ テーブルの間の基数とフィルターの方向は自動的に選択されます。  
  
2.  **FactInternetSales** テーブルで、**DueDate** 列をクリックし、そのままカーソルを **DimDate** テーブル内の **Date** 列にドラッグして離します。  
  
    **FactInternetSales** テーブル内の **DueDate** 列と **DimDate** テーブル内の **Date** 列の間に、非アクティブなリレーションシップを作成したことを示す点線が現れます。 テーブル間には複数のリレーションシップを持つことができますが、一度にアクティブにできるリレーションシップは 1 つのみです。 非アクティブなリレーションシップは、カスタムの DAX 式で特別な集計を実行することで有効にすることができます。  
  
3.  最後に、もう 1 つリレーションシップを作成します。 **FactInternetSales** テーブルで、**ShipDate** 列をクリックし、そのままカーソルを **DimDate** テーブル内の **Date** 列にドラッグして離します。  
    
     ![aas-lesson4-newinactive](../tutorials/media/aas-lesson4-newinactive.png)
  
## <a name="whats-next"></a>次の手順
[レッスン 5: 計算列を作成する](../tutorials/aas-lesson-5-create-calculated-columns.md)
  
  
  

