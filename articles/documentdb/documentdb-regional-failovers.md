---
title: "Azure DocumentDB 的區域性容錯移轉 | Microsoft Docs"
description: "了解 Azure DocumentDB 的手動和自動容錯移轉如何運作。"
services: documentdb
documentationcenter: 
author: arramac
manager: jhubbard
editor: 
ms.assetid: 446e2580-ff49-4485-8e53-ae34e08d997f
ms.service: documentdb
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/09/2017
ms.author: arramac
ms.custom: H1Hack27Feb2017
translationtype: Human Translation
ms.sourcegitcommit: 094729399070a64abc1aa05a9f585a0782142cbf
ms.openlocfilehash: 4af4d30a3378e1aea66309a1d757be1c1da2ea0d
ms.lasthandoff: 03/07/2017


---
# <a name="automatic-regional-failovers-for-business-continuity-in-documentdb"></a>DocumentDB 中商務持續性的自動區域容錯移轉
Azure DocumentDB 會簡化資料的全球發佈作業，方法是提供多個可完全管理的[多區域資料庫帳戶](documentdb-distribute-data-globally.md)，在一致性、可用性和效能之間進行明確取捨，這一切全都倚靠相對應的保證來完成。 DocumentDB 帳戶具備下列優點：高可用性、個位數的毫秒延遲、[定義完善的一致性層級](documentdb-consistency-levels.md)、利用多首頁 API 透明進行的區域性容錯移轉，以及全球輸送量及儲存體的彈性調整能力。 

Azure DocumentDB 支援明確和原則導向的容錯移轉，可讓您控制在失敗發生時的端對端系統行為。 我們將在本文中說明：

* DocumentDB 的手動容錯移轉如何運作？
* DocumentDB 的自動容錯移轉如何運作？
* 如何在應用程式架構中使用手動容錯移轉？

您也可以在這段 Azure Friday 影片中，和 Scott Hanselman 與工程總經理 Karthik Raman 一起了解區域容錯移轉。

>[!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Planet-Scale-NoSQL-with-DocumentDB/player]  

## <a id="ConfigureMultiRegionApplications"></a>設定多區域應用程式
在深入探討容錯移轉模式之前，我們會探討如何設定應用程式以發揮多區域可用性，並在區域性容錯移轉中兼具彈性。

* 首先，在多個區域中部署您的應用程式
* 為了確保從您部署應用程式的每個區域能夠低延遲存取，使用支援的 SDK之一來設定每個區域的對應[慣用區域清單](https://msdn.microsoft.com/library/microsoft.azure.documents.client.connectionpolicy.preferredlocations.aspx#P:Microsoft.Azure.Documents.Client.ConnectionPolicy.PreferredLocations)。

下列程式碼片段示範如何初始化多區域的應用程式。 在這裡，我們替 DocumentDB 帳戶 `contoso.documents.azure.com` 設定兩個區域：美國西部和北歐。 

* 應用程式是部署在美國西部區域 (舉例來說，使用 Azure App Service) 
* 以 `West US` 設定為低延遲讀取的第一個慣用區域
* 以 `North Europe` 設定第二個慣用區域 (為了在區域失敗時有高可用性)

在 .NET 中，此設定看起來像下列程式碼片段︰

    ConnectionPolicy usConnectionPolicy = new ConnectionPolicy 
    { 
        ConnectionMode = ConnectionMode.Direct,
        ConnectionProtocol = Protocol.Tcp
    };

    usConnectionPolicy.PreferredLocations.Add(LocationNames.WestUS);
    usConnectionPolicy.PreferredLocations.Add(LocationNames.NorthEurope);

    DocumentClient usClient = new DocumentClient(
        new Uri("https://contosodb.documents.azure.com"),
        "memf7qfF89n6KL9vcb7rIQl6tfgZsRt5gY5dh3BIjesarJanYIcg2Edn9uPOUIVwgkAugOb2zUdCR2h0PTtMrA==",
        usConnectionPolicy);

應用程式也會部署在北歐區域，但是慣用區域順序相反。 也就是說，北歐是指定的第一個低延遲讀取區域。 然後，美國西部則指定為第二個慣用區域，以便在區域失敗時能有高可用性。

以下架構圖中顯示多區域應用程式的部署，其中的 DocumentDB 和應用程式設定為可在四個 Azure 地理區域使用。  

![使用 Azure DocumentDB 進行全球發佈應用程式的部署](./media/documentdb-regional-failovers/app-deployment.png)

現在，我們來看一下 DocumentDB 服務如何透過自動容錯移轉處理區域失敗。 

## <a id="AutomaticFailovers"></a>自動容錯移轉
發生 Azure 區域中斷這類罕見事件時，DocumentDB 會自動觸發受影響區域中出現的所有 DocumentDB 帳戶的容錯移轉。 

**如果讀取區域中斷會發生什麼事？**

DocumentDB 帳戶的讀取區域若在其中一個受影響區域中，會自動與其寫入區域中斷連線，並標示為離線。 DocumentDB SDK 實作區域探索通訊協定，當區域可供使用時它們會自動偵測，並將讀取呼叫重新導向至慣用區域清單中下一個可用的區域。 如果慣用區域清單中的區域都無法使用，呼叫會自動切換回目前的寫入區域。 不需要對應用程式的程式碼做任何變更來處理區域容錯移轉。 整個過程中，DocumentDB 的一致性保證絲毫無損。

![Azure DocumentDB 中的讀取區域失敗](./media/documentdb-regional-failovers/read-region-failures.png)

一旦受影響區域從中斷復原，服務會自動回復該區域中所有受影響的 DocumentDB 帳戶。 讀取區域在受影響區域中的 DocumentDB 帳戶，則會自動與目前寫入區域同步，並開啟上線。 DocumentDB SDK 會探索新區域的可用性，並根據應用程式設定的慣用區域清單，評估是否應該選取該區域做為目前讀取區域。 後續的讀取會被重新導向至復原的區域，不需要對應用程式的程式碼做任何變更。

**如果寫入區域中斷會發生什麼事？**

如果受影響的區域是 Azure DocumentDB 帳戶的目前寫入區域，則該區域會自動標示為離線。 然後，每個受影響 DocumentDB 帳戶的替代區域會升級為寫入區域。 您可以透過 Azure 入口網站或[以程式設計方式](https://docs.microsoft.com/rest/api/documentdbresourceprovider/databaseaccounts#DatabaseAccounts_FailoverPriorityChange)，完全控制 DocumentDB 帳戶的區域選擇順序。 

![Azure DocumentDB 的容錯移轉優先順序](./media/documentdb-regional-failovers/failover-priorities.png)

在自動容錯移轉期間，DocumentDB 會根據 Azure DocumentDB 帳戶指定的優先順序，自動選擇下一個寫入區域。 

![Azure DocumentDB 中的寫入區域失敗](./media/documentdb-regional-failovers/write-region-failures.png)

一旦受影響區域從中斷復原，服務會自動回復該區域中所有受影響的 DocumentDB 帳戶。 

* 原先寫入區域在受影響區域中的 DocumentDB 帳戶，即使在區域復原後，仍會保持具可讀取性的離線模式。 
* 在中斷期間，您可以與目前寫入區域中的可用資料比較，藉此查詢此區域來計算任何未複寫的寫入。 根據您的應用程式需求，您可以執行合併和/或衝突解決，並將最後一組變更寫回目前的寫入區域。 
* 完成合併變更後，您可以在 DocumentDB 帳戶中移除再重新新增區域，將受影響的區域帶回線上。 一旦將區域新增回來，您就可以透過 Azure 入口網站或[以程式設計方式](https://docs.microsoft.com/rest/api/documentdbresourceprovider/databaseaccounts#DatabaseAccounts_CreateOrUpdate)執行手動容錯移轉，將它設回寫入區域。

## <a id="ManualFailovers"></a>手動容錯移轉

除了自動容錯移轉，可以手動將 DocumentDB 帳戶的目前寫入區域動態變更為現有讀取區域之一。 手動容錯移轉可透過 Azure 入口網站或[以程式設計方式](https://docs.microsoft.com/rest/api/documentdbresourceprovider/databaseaccounts#DatabaseAccounts_CreateOrUpdate)起始。 

手動容錯移轉可確保**資料零遺失**和**可用性零遺失**，並優雅地將指定 DocumentDB 帳戶的寫入狀態從舊的寫入區域傳輸到新的。 像自動容錯移轉一樣，Azure DocumentDB SDK 會在手動容錯移轉期間自動處理寫入區域的變更，並確保呼叫被自動重新導向至新的寫入區域。 不需要對應用程式的程式碼或設定做任何變更來管理容錯移轉。 

![Azure DocumentDB 中的手動容錯移轉](./media/documentdb-regional-failovers/manual-failovers.png)

手動容錯移轉很實用的常見案例有︰

**遵循時鐘模型**︰如果您的應用程式可根據一天中的時間預測流量模式，您可以根據當日的時間定期將寫入狀態變更為最常使用的地理區域。

**服務更新**︰某些分散在世界各地的應用程式部署，可能會在其規劃的服務更新期間透過流量管理員將流量重新路由至其他區域。 現在，這類應用程式部署可以使用手動容錯移轉，將寫入狀態保持為在服務更新期間即將有活躍流量的區域。

**商務持續性和災害復原 (BCDR) 鑽研**︰大部分的企業應用程式會將商務持續性測試納入其開發和發行程序。 BCDR 測試通常是合規性認證以及區域性中斷時服務可用性保證的重要步驟。 您可以針對使用 DocumentDB 做為儲存體的應用程式測試 BCDR 完備性，做法是觸發 DocumentDB 帳戶的手動容錯移轉，及/或動態新增並移除區域。

在本文中，我們看了在 Azure DocumentDB 中手動和自動容錯移轉如何運作，以及如何將 DocumentDB 帳戶和應用程式設定為可全域使用。 使用 Azure DocumentDB 的全域複寫支援，可以改善端對端延遲，並確保其即使是在區域失敗時仍有高可用性。 

## <a id="NextSteps"></a>後續步驟
* 了解 DocumentDB 如何支援[全域發佈](documentdb-distribute-data-globally.md)
* 了解 [DocumentDB 的全域一致性](documentdb-consistency-levels.md)
* 使用 [Azure DocumentDB SDK](documentdb-developing-with-multiple-regions.md) 進行多區域開發
* 了解如何使用 Azure DocumentDB 建置[多區域寫入器架構](documentdb-multi-region-writers.md)


