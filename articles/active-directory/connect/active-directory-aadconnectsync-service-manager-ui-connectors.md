---
title: "Azure AD Synchronization Service Manager UI 中的連接器 | Microsoft Docs&quot;"
description: "了解 Azure AD Connect 的 Synchronization Service Manager 中的 [連接器] 索引標籤。"
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
ms.date: 03/02/2017
ms.author: billmath
ms.custom: H1Hack27Feb2017
translationtype: Human Translation
ms.sourcegitcommit: 2f03ba60d81e97c7da9a9fe61ecd419096248763
ms.openlocfilehash: bd2240678fed44db748ae062bdf91e457159b4a2
ms.lasthandoff: 03/04/2017

---
# <a name="using-connectors-with-the-azure-ad-connect-sync-service-manager"></a>使用連接器搭配 Auzre AD Connect Sync Service Manager

![Sync Service Manager](./media/active-directory-aadconnectsync-service-manager-ui/connectors.png)

[連接器] 索引標籤可用來管理同步處理引擎連接的所有系統。

## <a name="connector-actions"></a>連接器動作
| 動作 | 註解 |
| --- | --- |
| 建立 |請勿使用。 若要連接到其他的 AD 樹系，請使用安裝精靈。 |
| 屬性 |用於網域和 OU 篩選。 |
| [刪除](#delete) |用來刪除連接器空間中的資料或刪除與樹系的連接。 |
| [更新執行設定檔](#configure-run-profiles) |除了網域篩選以外，不會在此處進行任何設定。 您可以使用此動作來查看已設定的執行設定檔。 |
| 執行 |用來啟動設定檔的一次性執行。 |
| 停止 |停止目前執行設定檔的連接器。 |
| 匯出連接器 |請勿使用。 |
| 匯入連接器 |請勿使用。 |
| 更新連接器 |請勿使用。 |
| 重新整理結構描述 |重新整理快取的結構描述。 最好是改為在安裝精靈中使用此選項，因為其也會更新同步處理規則。 |
| [搜尋連接器空間](#search-connector-space) |用來尋找物件，以及 [在整個系統中追隨物件及其資料](#follow-an-object-and-its-data-through-the-system)。 |

### <a name="delete"></a>刪除
刪除動作適用於兩個不同的用途。  
![Sync Service Manager](./media/active-directory-aadconnectsync-service-manager-ui/connectordelete.png)

[僅刪除連接器空間]  選項會移除所有資料，但保留組態。

[刪除連接器和連接器空間]  選項會移除資料及組態。 當您不想再連接到樹系時，可使用此選項。

這兩個選項會同步處理所有物件，並更新 metaverse 物件。 此動作是長時間執行的作業。

### <a name="configure-run-profiles"></a>更新執行設定檔
此選項可讓您查看連接器所設定的執行設定檔。

![Sync Service Manager](./media/active-directory-aadconnectsync-service-manager-ui/configurerunprofiles.png)

### <a name="search-connector-space"></a>搜尋連接器空間
尋找物件和疑難排解資料問題時，搜尋連接器空間動作非常有用。

![Sync Service Manager](./media/active-directory-aadconnectsync-service-manager-ui/cssearch.png)

先選取一個 [範圍] 。 您可以依據資料 (RDN、DN、錨點、子樹狀目錄) 或物件狀態 (所有其他選項) 進行搜尋。  
![Sync Service Manager](./media/active-directory-aadconnectsync-service-manager-ui/cssearchscope.png)  
例如，如果您進行樹狀子目錄的搜尋，將會取得某一個 OU 中的所有物件。  
![Sync Service Manager](./media/active-directory-aadconnectsync-service-manager-ui/cssearchsubtree.png)  
您可以從此格線選取物件、選取「屬性」，然後[跟隨物件](active-directory-aadconnectsync-troubleshoot-object-not-syncing.md)，從來源連接器空間、通過 Metaverse，再到目標連接器空間。

## <a name="next-steps"></a>後續步驟
深入了解 [Azure AD Connect 同步](active-directory-aadconnectsync-whatis.md) 組態。

深入了解 [整合內部部署身分識別與 Azure Active Directory](active-directory-aadconnect.md)。

