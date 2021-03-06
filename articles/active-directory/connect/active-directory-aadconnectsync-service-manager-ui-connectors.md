---
title: "Azure AD の Synchronization Service Manager UI のコネクタ | Microsoft Docs"
description: "Azure AD Connect の Synchronization Service Manager の [コネクタ] タブについて"
services: active-directory
documentationcenter: 
author: andkjell
manager: femila
editor: 
ms.assetid: 60f1d979-8e6d-4460-aaab-747fffedfc1e
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/13/2017
ms.author: billmath
ms.custom: H1Hack27Feb2017
ms.translationtype: Human Translation
ms.sourcegitcommit: eeb56316b337c90cc83455be11917674eba898a3
ms.openlocfilehash: 6d893efd775ff6b55524ba3a621d8248adbdd432
ms.contentlocale: ja-jp
ms.lasthandoff: 04/03/2017

---
# <a name="using-connectors-with-the-azure-ad-connect-sync-service-manager"></a>Azure AD Connect の Sync Service Manager でコネクタを使用する

![Sync Service Manager](./media/active-directory-aadconnectsync-service-manager-ui/connectors.png)

[コネクタ] タブを利用し、同期エンジンが接続されているすべてのシステムを管理します。

## <a name="connector-actions"></a>コネクタのアクション
| アクション | コメント |
| --- | --- |
| 作成 |使用しないでください。 その他の AD フォレストに接続するには、インストール ウィザードを使用します。 |
| プロパティ |ドメインと OU フィルター処理に使用されます。 |
| [削除](#delete) |コネクタ スペースのデータを削除するか、フォレストへの接続を削除するために使用されます。 |
| [実行プロファイルの構成](#configure-run-profiles) |ドメイン フィルター処理を除き、ここで構成するものはありません。 このアクションは、既に構成されている実行プロファイルを確認する場合に使用します。 |
| 実行 |プロファイルの 1 回限りの実行を開始するために使用されます。 |
| 停止 |現在プロファイルを実行しているコネクタを停止します。 |
| コネクタのエクスポート |使用しないでください。 |
| コネクタのインポート |使用しないでください。 |
| コネクタの更新 |使用しないでください。 |
| スキーマの更新 |キャッシュされたスキーマを更新します。 このアクション代わりに、インストール ウィザードのオプションを使用することをお勧めします。インストール ウィザードのオプションであれば、同期規則も更新されます。 |
| [コネクタ スペースの検索](#search-connector-space) |オブジェクトを検索し、 [オブジェクトとそのデータをシステム全体でフォローする](#follow-an-object-and-its-data-through-the-system)ために使用されます。 |

### <a name="delete"></a>削除
削除アクションには次の 2 つがあります。  
![Sync Service Manager](./media/active-directory-aadconnectsync-service-manager-ui/connectordelete.png)

**[Delete connector space only (コネクタ スペースのみを削除する)]** オプションを選択すると、すべてのデータが削除されますが、構成は維持されます。

**[Delete Connector and connector space (コネクタとコネクタ スペースを削除する)]** オプションを選択すると、データと構成が削除されます。 このオプションは、フォレストに今後接続しない場合に使用します。

いずれのオプションでも、すべてのオブジェクトが同期され、メタバース オブジェクトが更新されます。 このアクションは、実行時間の長い操作となります。

### <a name="configure-run-profiles"></a>実行プロファイルの構成
このオプションを使用すると、コネクタ用に構成された実行プロファイルを表示できます。

![Sync Service Manager](./media/active-directory-aadconnectsync-service-manager-ui/configurerunprofiles.png)

### <a name="search-connector-space"></a>コネクタ スペースの検索
コネクタ スペースの検索アクションは、オブジェクトを検索し、データ問題を解決するのに役立ちます。

![Sync Service Manager](./media/active-directory-aadconnectsync-service-manager-ui/cssearch.png)

まず、 **[scope]**(範囲) を選択します。 データ (RDN、DN、アンカー、サブツリー) またはオブジェクトの状態 (その他すべてのオプション) に基づいて検索できます。  
![Sync Service Manager](./media/active-directory-aadconnectsync-service-manager-ui/cssearchscope.png)  
たとえば、サブツリー検索を行うと、1 つの OU のすべてのオブジェクトが取得されます。  
![Sync Service Manager](./media/active-directory-aadconnectsync-service-manager-ui/cssearchsubtree.png)  
このグリッドからオブジェクトを選択し、**[プロパティ]** を選択して、ソース コネクタ スペースからメタバースを経てターゲット コネクタ スペースまで[フォロー](active-directory-aadconnectsync-troubleshoot-object-not-syncing.md)できます。

### <a name="changing-the-ad-ds-account-password"></a>AD DS アカウント パスワードの変更
アカウントのパスワードを変更すると、Synchronization Service でオンプレミスの AD に変更をインポートまたはエクスポートできなくなります。   次のように表示されます。

- AD コネクタのインポートまたはエクスポート手順が失敗し、"no-start-credentials (開始資格情報なし)" エラーが表示されます。
- Windows イベント ビューアーのアプリケーション イベント ログに、イベント ID 6000 のエラーと「資格情報が有効でないため、管理エージェント "contoso.com" は実行できませんでした。」というメッセージが表示されます。

この問題を解決するには、次の手順で AD DS ユーザー アカウントを更新します。


1. Synchronization Service Manager を起動します ([スタート]、[同期サービス] の順に移動します)。
</br>![Sync Service Manager](./media/active-directory-aadconnectsync-service-manager-ui/startmenu.png)
2. **[コネクタ]** タブに移動します。
3. AD DS アカウントを使用して構成されている AD コネクタを選択します。
4. [アクション] の **[プロパティ]** を選択します。
5. ポップアップ ダイアログで、[Active Directory フォレストに接続] を選択します。
6. フォレスト名は、対応するオンプレミスの AD を示します。
7. ユーザー名は、同期に使用する AD DS アカウントを示します。
8. ![[Azure AD Connect Sync 暗号化キー ユーティリティ]](media/active-directory-aadconnectsync-encryption-key/key6.png) というパスワード テキスト ボックスに、AD DS アカウントの新しいパスワードを入力します。
9. [OK] をクリックして、新しいパスワードを保存し、Synchronization Service を再起動して、メモリ キャッシュから古いパスワードを削除します。



## <a name="next-steps"></a>次のステップ
[Azure AD Connect Sync](active-directory-aadconnectsync-whatis.md) の構成に関するページをご覧ください。

「 [オンプレミス ID と Azure Active Directory の統合](active-directory-aadconnect.md)」をご覧ください。

