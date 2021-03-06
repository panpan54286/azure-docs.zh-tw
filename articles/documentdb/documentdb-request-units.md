---
title: "要求單位和估計輸送量 - Azure DocumentDB | Microsoft Docs"
description: "了解如何在 DocumentDB 中了解、指定及估計要求單位需求。"
services: documentdb
author: syamkmsft
manager: jhubbard
editor: mimig
documentationcenter: 
ms.assetid: d0a3c310-eb63-4e45-8122-b7724095c32f
ms.service: documentdb
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/08/2017
ms.author: syamk
translationtype: Human Translation
ms.sourcegitcommit: a087df444c5c88ee1dbcf8eb18abf883549a9024
ms.openlocfilehash: b098e3087cb08528c5fbdc2d0d768ce40e7ffe0d
ms.lasthandoff: 03/15/2017


---
# <a name="request-units-in-documentdb"></a>DocumentDB 中的要求單位
現在可供使用︰DocumentDB [要求單位計算機](https://www.documentdb.com/capacityplanner)。 深入了解 [預估您的輸送量需求](documentdb-request-units.md#estimating-throughput-needs)。

![輸送量計算機][5]

## <a name="introduction"></a>簡介
[Azure DocumentDB](https://azure.microsoft.com/services/documentdb/) 是適用於 JSON 文件且受到完整管理、可調整的 NoSQL 資料庫服務。 有了 DocumentDB，您就不需要租用虛擬機器、部署軟體或監視資料庫。 Microsoft 工程師會負責操作並持續監視 DocumentDB，提供世界級的可用性、效能和資料保護能力。 DocumentDB 中的資料會儲存在集合內，而集合是具有彈性、高可用性的容器。 您可以保留輸送量 (以每秒要求數表示)，而不需考量及管理集合的硬體資源，例如 CPU、記憶體和 IOP。 DocumentDB 會自動管理集合的佈建、透明資料分割和調整，以便處理已佈建的要求數目。 

DocumentDB 支援數種 API，以便進行讀取、寫入、查詢和預存程序執行。 因為並非所有的要求都相等，所以系統會根據處理要求所需的計算量，指派標準化的**要求單位**數量。 作業的要求單位數具決定性，您可以在 DocumentDB 中透過回應標頭追蹤任何作業所取用的要求單位數。

DocumentDB 中的每個集合都可以經由輸送量 (也可以要求單位表示) 保留。 這是以每秒 100 個要求單位的區塊表示 (範圍從每秒數百個到數百萬個要求單位)。 佈建的輸送量可以在集合存留期期間進行調整，以適應多變的處理需求和應用程式的存取模式。 

閱讀本文後，您將能夠回答下列問題：  

* 什麼是要求單位和要求費用？
* 如何指定集合的要求單位容量？
* 如何估計應用程式的要求單位需求？
* 如果超過集合的要求單位容量，會發生什麼事？

## <a name="request-units-and-request-charges"></a>要求單位和要求費用
DocumentDB 和 API for MongoDB 可「保留」資源以滿足應用程式的輸送量需求，提供快速且可預測的效能。  因為應用程式會隨著時間載入和存取模式變化，所以 DocumentDB 可讓您輕鬆地增加或減少應用程式可用的保留輸送量。

有了 DocumentDB，保留的輸送量是根據每秒處理的要求單位來指定。  您可以將要求單位想像為輸送量貨幣，因此您每秒可「保留」  保證可供應用程式使用的要求單位數量。  DocumentDB 中的每個作業 (寫入文件、執行查詢、更新文件) 都會耗用 CPU、記憶體和 IOPS。  也就是說，每個作業都會產生「要求費用」，這是以「要求單位」來表示。  了解影響要求單位費用的因素，以及您應用程式的輸送量需求，讓您能夠以最經濟實惠的方式來執行應用程式。 

我們建議從觀看 Aravind Ramachandran 說明使用 DocumentDB 的要求單位和可預測效能的下列影片來開始。

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Predictable-Performance-with-DocumentDB/player]
> 
> 

## <a name="specifying-request-unit-capacity-in-documentdb"></a>在 DocumentDB 中指定要求單位容量
建立 DocumentDB 集合時，您可以指定想要保留給集合的每秒要求單位數 (每秒 RU)。 DocumentDB 會根據佈建的輸送量，配置實體資料分割來裝載您的集合，並隨著資料成長分割/再平衡所有資料分割的資料。

在集合中佈建了 10,000 個或更多要求單位時，DocumentDB 會要求指定資料分割索引鍵。 此外，也需要資料分割索引鍵，才能調整未來超過 10,000 個要求單位的集合輸送量。 因此，強烈建議在建立集合時設定[資料分割索引鍵](documentdb-partition-data.md) (不論初始輸送量是多少)。 因為您的資料可能必須分散於多個資料分割，所以必須挑選具有高基數 (數百個到數百萬個相異值) 的資料分割索引鍵，以便 DocumentDB 一致調整您的集合和要求。 

> [!NOTE]
> 資料分割索引鍵是一種邏輯界限，而不是實體界限。 因此，您不需要限制不同資料分割索引鍵值的數目。 事實上，最好能擁有較多不同的資料分割索引鍵值，DocumentDB 才有更多的負載平衡選項。

以下的程式碼片段使用 .NET SDK 建立包含每秒 3,000 個要求單位的集合：

```csharp
DocumentCollection myCollection = new DocumentCollection();
myCollection.Id = "coll";
myCollection.PartitionKey.Paths.Add("/deviceId");

await client.CreateDocumentCollectionAsync(
    UriFactory.CreateDatabaseUri("db"),
    myCollection,
    new RequestOptions { OfferThroughput = 3000 });
```

DocumentDB 會在輸送量的保留模型上運作。 也就是說，您需要針對為集合所「保留」的輸送量支付費用，而不論該輸送量中有多少數量是主動「使用」的。 當您應用程式的負載、資料及使用方式模式變更時，您可以透過 DocumentDB SDK 或使用 [Azure 入口網站](https://portal.azure.com)，輕鬆地相應增加和減少保留 RU 的數量。

每個集合會對應至 DocumentDB 中的 `Offer` 資源，其具有集合佈建輸送量的相關中繼資料。 藉由查閱集合的對應 offer 資源，然後使用新的輸送量值加以更新，即可變更已配置的輸送量。 以下的程式碼片段使用 .NET SDK 將集合的輸送量變更為每秒 5,000 個要求單位：

```csharp
// Fetch the resource to be updated
Offer offer = client.CreateOfferQuery()
                .Where(r => r.ResourceLink == collection.SelfLink)    
                .AsEnumerable()
                .SingleOrDefault();

// Set the throughput to 5000 request units per second
offer = new OfferV2(offer, 5000);

// Now persist these changes to the database by replacing the original resource
await client.ReplaceOfferAsync(offer);
```

當您變更輸送量時，不會影響集合的可用性。 新保留的輸送量通常會在套用新輸送量的數秒內於生效。

## <a name="specifying-request-unit-capacity-in-api-for-mongodb"></a>在 API for MongoDB 中指定要求單位容量
API for MongoDB 可讓您指定想要保留給集合的每秒要求單位數 (每秒 RU)。

API for MongoDB 是在與 DocumentDB 相同的保留模型上，根據輸送量而運作。 也就是說，您需要針對為集合所「保留」的輸送量支付費用，而不論該輸送量中有多少數量是主動「使用」的。 當應用程式的負載、資料及使用方式模式變更時，您可以透過 [Azure 入口網站](https://portal.azure.com)，輕鬆地相應增加和減少保留的 RU 數量。

當您變更輸送量時，不會影響集合的可用性。 新保留的輸送量通常會在套用新輸送量的數秒內於生效。

## <a name="request-unit-considerations"></a>要求單位的考量
在估計要保留給 DocumentDB 集合的要求單位數量時，請務必考量下列變數：

* **文件大小**。 文件大小增加時，也會增加讀取或寫入資料所耗用的單位。
* **文件屬性計數**。 假設預設會編製所有屬性的索引，撰寫文件所耗用的單位將會隨著屬性計數增加而增加。
* **資料一致性**。 當使用強式或繫結失效的資料一致性層級時，將會耗用額外單位來讀取文件。
* **已編製索引的屬性**。 每個集合上的索引原則會判定預設要編製哪些索引的屬性。 您可以限制已編製索引的屬性數目或讓索引延遲編製，以減少要求單位的耗用量。
* **編製文件索引**。 依預設會自動編製每份文件的索引，如果您選擇不要編製一些文件的索引，則會耗用較少的要求單位。
* **查詢模式**。 查詢的複雜性會影響針對作業所耗用的要求單位數量。 述詞數目、述詞性質、預測、UDF 數目，以及來源資料集的大小，全都會影響查詢作業的成本。
* **指令碼使用方式**。  查詢、預存程序和觸發程序會根據所執行作業的複雜度來耗用要求單位。 開發應用程式時，請檢查要求費用標頭，進一步了解每項作業如何耗用要求單位容量。

## <a name="estimating-throughput-needs"></a>估計輸送量需求
要求單位是要求處理成本的標準化量值。 單一要求單位代表讀取 (透過自我連結或識別碼) 由 10 個唯一屬性值 (不包括系統屬性) 所組成的單一 1KB JSON 文件所需的處理容量。 建立 (插入)、取代或刪除相同文件的要求將會耗用服務的更多處理，因此會耗用更多的要求單位。   

> [!NOTE]
> 1KB 文件 1 個要求單位的基準，會對應至文件自我連結或識別碼的簡單 GET。
> 
> 

例如，下表顯示要在三個不同的文件大小 (1KB、4KB 和 64KB)，以及兩個不同效能等級 (500 讀取/秒 + 100 寫入/秒和 500 讀取/秒 + 500 寫入/秒) 中佈建多少要求單位。 在工作階段中設定資料一致性，且索引編製原則已設定為 [無]。

<table border="0" cellspacing="0" cellpadding="0">
    <tbody>
        <tr>
            <td valign="top"><p><strong>文件大小</strong></p></td>
            <td valign="top"><p><strong>讀取/秒</strong></p></td>
            <td valign="top"><p><strong>寫入/秒</strong></p></td>
            <td valign="top"><p><strong>要求單位</strong></p></td>
        </tr>
        <tr>
            <td valign="top"><p>1 KB</p></td>
            <td valign="top"><p>500</p></td>
            <td valign="top"><p>100</p></td>
            <td valign="top"><p>(500 *1) + (100* 5) = 1,000 RU/s</p></td>
        </tr>
        <tr>
            <td valign="top"><p>1 KB</p></td>
            <td valign="top"><p>500</p></td>
            <td valign="top"><p>500</p></td>
            <td valign="top"><p>(500 *5) + (100* 5) = 3,000 RU/s</p></td>
        </tr>
        <tr>
            <td valign="top"><p>4 KB</p></td>
            <td valign="top"><p>500</p></td>
            <td valign="top"><p>100</p></td>
            <td valign="top"><p>(500 *1.3) + (100* 7) = 1,350 RU/s</p></td>
        </tr>
        <tr>
            <td valign="top"><p>4 KB</p></td>
            <td valign="top"><p>500</p></td>
            <td valign="top"><p>500</p></td>
            <td valign="top"><p>(500 *1.3) + (500* 7) = 4,150 RU/s</p></td>
        </tr>
        <tr>
            <td valign="top"><p>64 KB</p></td>
            <td valign="top"><p>500</p></td>
            <td valign="top"><p>100</p></td>
            <td valign="top"><p>(500 *10) + (100* 48) = 9,800 RU/s</p></td>
        </tr>
        <tr>
            <td valign="top"><p>64 KB</p></td>
            <td valign="top"><p>500</p></td>
            <td valign="top"><p>500</p></td>
            <td valign="top"><p>(500 *10) + (500* 48) = 29,000 RU/s</p></td>
        </tr>
    </tbody>
</table>

### <a name="use-the-request-unit-calculator"></a>使用要求單位計算機
若要協助客戶微調其輸送量預估，有 Web 架構 [要求單位計算機](https://www.documentdb.com/capacityplanner) 可以協助預估一般作業的要求單位需求，包括︰

* 文件建立 (寫入)
* 文件讀取
* 文件刪除
* 文件更新

此工具也支援根據您提供的範例文件預估資料儲存需求。

使用此工具很簡單︰

1. 上傳一或多個具代表性的 JSON 文件。
   
    ![將文件上傳至要求單位計算機][2]
2. 若要預估資料儲存需求，請輸入您預期要儲存的文件總數。
3. 輸入您需要的建立、讀取、更新和刪除作業的文件數 (以每秒為單位)。 若要預估文件更新作業的要求單位費用，請上傳一份上述步驟 1 中包含一般欄位更新的範例文件。  例如，如果文件更新通常會修改名為 lastLogin 和 userVisits 的兩個屬性，則只要複製範例文件、更新這兩個屬性的值，並上傳複製的文件。
   
    ![在要求單位計算機中輸入輸送量需求][3]
4. 按一下計算，並檢查結果。
   
    ![要求單位計算機結果][4]

> [!NOTE]
> 如果您的文件類型與已編製索引之屬性的大小與數目截然不同，則請將每個一般文件「類型」的範例上傳至工具，然後計算結果。
> 
> 

### <a name="use-the-documentdb-request-charge-response-header"></a>使用 DocumentDB 要求費用回應標頭
DocumentDB 服務的每個回應都會包括自訂標頭 (`x-ms-request-charge`)，其中包含要求所耗用的要求單位。 此標頭也可以透過 DocumentDB SDK 進行存取。 在 .NET SDK 中，RequestCharge 是 ResourceResponse 物件的屬性。  針對查詢， Azure 入口網站中的 DocumentDB 查詢總管會提供已執行查詢的要求費用資訊。

![在查詢總管中檢查 RU 費用][1]

將此銘記於心，有一個估計應用程式所需之保留輸送量的方法是，記錄與根據應用程式所使用之代表性文件執行一般作業相關聯的要求單位費用，然後估計您預期每秒執行的作業數目。  此外，請務必測量並包含一般查詢和 DocumentDB 指令碼使用方式。

> [!NOTE]
> 如果您的文件類型與已編製索引之屬性的大小與數目截然不同，則請記錄與每個標準文件「類型」相關聯的適用作業要求單位費用。
> 
> 

例如：

1. 記錄建立 (插入) 標準文件的要求單位費用。 
2. 記錄讀取標準文件的要求單位費用。
3. 記錄更新標準文件的要求單位費用。
4. 記錄標準且常見文件查詢的要求單位費用。
5. 記錄應用程式所使用的任何自訂指令碼 (預存程序、觸發程序、使用者定義的函式) 的要求單位費用
6. 根據您預期每秒執行的作業估計數目，來計算必要的要求單位數。

### <a id="GetLastRequestStatistics"></a>使用 API for MongoDB 的 GetLastRequestStatistics 命令
API for Mongodb 支援自訂命令 *getLastRequestStatistics*，可擷取指定之作業的要求費用。

例如，在 Mongo 殼層中，執行您想要驗證要求費用的作業。
```
> db.sample.find()
```

接著，執行命令 *getLastRequestStatistics*。
```
> db.runCommand({getLastRequestStatistics: 1})
{
    "_t": "GetRequestStatisticsResponse",
    "ok": 1,
    "CommandName": "OP_QUERY",
    "RequestCharge": 2.48,
    "RequestDurationInMilliSeconds" : 4.0048
}
```

將此銘記於心，有一個估計應用程式所需之保留輸送量的方法是，記錄與根據應用程式所使用之代表性文件執行一般作業相關聯的要求單位費用，然後估計您預期每秒執行的作業數目。

> [!NOTE]
> 如果您的文件類型與已編製索引之屬性的大小與數目截然不同，則請記錄與每個標準文件「類型」相關聯的適用作業要求單位費用。
> 
> 

## <a name="use-api-for-mongodbs-portal-metrics"></a>使用 API for MongoDB 的入口網站計量
若要準確估計 API for MongoDB 資料庫的要求單位費用，最簡單的方法就是使用 [Azure 入口網站](https://portal.azure.com)計量。 利用 [要求數目] 和 [要求費用] 圖表，您可以估計每個作業耗用多少要求單位，以及它們彼此之間相對耗用多少要求單位。

![API for MongoDB 入口網站計量][6]

## <a name="a-request-unit-estimation-example"></a>要求單位估計範例
請考量下列 ~1KB 文件：

```json
{
 "id": "08259",
  "description": "Cereals ready-to-eat, KELLOGG, KELLOGG'S CRISPIX",
  "tags": [
    {
      "name": "cereals ready-to-eat"
    },
    {
      "name": "kellogg"
    },
    {
      "name": "kellogg's crispix"
    }
  ],
  "version": 1,
  "commonName": "Includes USDA Commodity B855",
  "manufacturerName": "Kellogg, Co.",
  "isFromSurvey": false,
  "foodGroup": "Breakfast Cereals",
  "nutrients": [
    {
      "id": "262",
      "description": "Caffeine",
      "nutritionValue": 0,
      "units": "mg"
    },
    {
      "id": "307",
      "description": "Sodium, Na",
      "nutritionValue": 611,
      "units": "mg"
    },
    {
      "id": "309",
      "description": "Zinc, Zn",
      "nutritionValue": 5.2,
      "units": "mg"
    }
  ],
  "servings": [
    {
      "amount": 1,
      "description": "cup (1 NLEA serving)",
      "weightInGrams": 29
    }
  ]
}
```

> [!NOTE]
> 文件在 DocumentDB 中會變小，因此系統針對上述文件計算出的大小會稍微少於 1KB。
> 
> 

下表顯示本文件中標準操作的概略要求單位費用 (概略的要求單位費用假設帳戶一致性層級會設定為「工作階段」且所有文件都已自動編制索引)：

| 作業 | 要求單位費用 |
| --- | --- |
| 建立文件 |~15 RU |
| 讀取文件 |~1 RU |
| 依識別碼查詢文件 |~2.5 RU |

此外，下表顯示應用程式中所使用之標準查詢的概略要求單位費用︰

| 查詢 | 要求單位費用 | # 傳回的文件數目 |
| --- | --- | --- |
| 依識別碼選取食物 |~2.5 RU |1 |
| 依製造商選取食物 |~7 RU |7 |
| 依食物群組選取並依重量排序 |~70 RU |100 |
| 選取食物群組中的前 10 個食物 |~10 RU |10 |

> [!NOTE]
> RU 費用會根據傳回的文件數目而有所不同。
> 
> 

利用此資訊，我們可以根據預期每秒的作業和查詢數目，來估計此應用程式的 RU 需求︰

| 作業/查詢 | 每秒估計的數目 | 必要的 RU 數 |
| --- | --- | --- |
| 建立文件 |10 |150 |
| 讀取文件 |100 |100 |
| 依製造商選取食物 |25 |175 |
| 依食物群組選取 |10 |700 |
| 選取前 10 個 |15 |總共&150; 個 |

在此情況下，我們預期平均輸送量需求為 1,275 RU/秒。  四捨五入至最接近 100 的數目，我們會針對此應用程式的集合佈建 1,300 RU/秒。

## <a id="RequestRateTooLarge"></a> 超過 DocumentDB 中保留的輸送量限制
您應該記得，要求單位耗用量是以每秒的速率來評估。 對於超過集合上佈建的要求單位速率的應用程式，對於該集合的要求會受到節流控制，直到該速率降到預留層級以下。 當節流發生時，伺服器將預先使用 RequestRateTooLargeException (HTTP 狀態碼 429) 來結束要求，並傳回 x-ms-retry-after-ms 標頭，以指出使用者重試要求之前必須等候的時間量 (毫秒)。

    HTTP Status 429
    Status Line: RequestRateTooLarge
    x-ms-retry-after-ms :100

如果您使用 .NET 用戶端 SDK 和 LINQ 查詢，則大部分時間您都不再需要處理這個例外狀況，因為目前版本的 .NET 用戶端 SDK 會隱含地攔截這個回應、遵守伺服器指定的 retry-after 標頭，然後重試要求。 除非有多個用戶端同時存取您的帳戶，否則下次重試將會成功。

如果您有多個用戶端會以高於要求速率的方式累積運作，則預設的重試行為可能會不敷使用，而用戶端將擲回 DocumentClientException (狀態碼 429) 到應用程式。 在這類情況下，您可以考慮處理應用程式錯誤處理常式中的重試行為和邏輯，或為集合提高保留的輸送量。

## <a id="RequestRateTooLargeAPIforMongoDB"></a> 超過 API for MongoDB 中保留的輸送量限制
如果應用程式超過集合已佈建的要求單位，則會受到節流控制，直到速率降到保留的等級以下。 節流發生時，後端會提前結束要求，並傳回&16500; 錯誤碼 -「太多要求」。 根據預設，在傳回「太多要求」錯誤碼之前，API for MongoDB 會自動重試最多 10 次。 如果您收到很多「太多要求」錯誤碼，您可以考慮在應用程式的錯誤處理常式中新增重試行為，或[提高集合的保留輸送量](documentdb-set-throughput.md)。

## <a name="next-steps"></a>後續步驟
若要深入了解透過 Azure DocumentDB 資料庫保留輸送量的方式，請探索下列資源：

* [DocumentDB 價格](https://azure.microsoft.com/pricing/details/documentdb/)
* [在 DocumentDB 中模型化資料](documentdb-modeling-data.md)
* [DocumentDB 效能等級](documentdb-partition-data.md)

若要深入了解 DocumentDB，請參閱 Azure DocumentDB [文件](https://azure.microsoft.com/documentation/services/documentdb/)。 

若要開始使用 DocumentDB 的相關規模和效能測試，請參閱 [Azure DocumentDB 的相關效能和規模測試](documentdb-performance-testing.md)。

[1]: ./media/documentdb-request-units/queryexplorer.png 
[2]: ./media/documentdb-request-units/RUEstimatorUpload.png
[3]: ./media/documentdb-request-units/RUEstimatorDocuments.png
[4]: ./media/documentdb-request-units/RUEstimatorResults.png
[5]: ./media/documentdb-request-units/RUCalculator2.png
[6]: ./media/documentdb-request-units/api-for-mongodb-metrics.png

