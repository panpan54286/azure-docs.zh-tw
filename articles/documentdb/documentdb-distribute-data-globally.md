---
title: "使用 DocumentDB 全球發佈資料 | Microsoft Docs"
description: "了解如何從 Azure DocumentDB (可完全管理的 NoSQL 資料庫服務)，使用全域資料庫進行全球規模的異地複寫、容錯移轉及資料復原。"
services: documentdb
documentationcenter: 
author: arramac
manager: jhubbard
editor: 
ms.assetid: ba5ad0cc-aa1f-4f40-aee9-3364af070725
ms.service: documentdb
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/14/2017
ms.author: arramac
translationtype: Human Translation
ms.sourcegitcommit: a087df444c5c88ee1dbcf8eb18abf883549a9024
ms.openlocfilehash: 8e1fccf953579beb138d47d1897bf702461fc39a
ms.lasthandoff: 03/15/2017


---
# <a name="documentdb---a-globally-distributed-database-service-on-azure"></a>DocumentDB - Azure 上分散在世界各地的資料庫服務
Azure 無所不在 - 跨 30 多個地理區域，遍佈全球並持續擴充中。 遍及全球的 Azure 提供給開發人員的其中一項差異化功能，就是能夠輕鬆地建置、部署及管理分散在世界各地的應用程式。 DocumentDB 是 Microsoft 分散在世界各地的多租用戶資料庫系統，目的是讓開發人員建置全球規模的應用程式。 DocumentDB 可讓您跨數目不拘的地理區域有彈性地調整輸送量和儲存體。 此服務可確保在 P99 低延遲、可用性高達 99.99%、可預測的輸送量，以及[多個定義完善的一致性模型](documentdb-consistency-levels.md) – 全部由全方位 SLA 支援。 憑藉其[結構描述無從驗證和寫入最佳化的資料庫引擎 (英文)](http://www.vldb.org/pvldb/vol8/p1668-shukla.pdf)，DocumentDB 預設能自動對其以獨立調整方式內嵌和提供給 [SQL](documentdb-sql-query.md)、[MongoDB](documentdb-protocol-mongodb.md) 及 [JavaScript 語言整合查詢](documentdb-programming.md#javascript-language-integrated-query-api)的所有資料編製索引。 DocumentDB 是經過仔細的全新工程設計的雲端服務，可供多租用戶使用和提供全域散發功能。

**分割並散發在三個 Azure 區域的單一 DocumentDB 集合**

![分割並散發在三個區域的 Azure DocumentDB 集合](./media/documentdb-distribute-data-globally/documentdb-global-apps.png)

如我們在建置 DocumentDB 時所知，新增全域散發功能是當務之急 - 它不能只放在「單一網站」資料庫系統上。 全域分散式資料庫提供的功能遠遠超越「單一網站」資料庫提供的傳統地理災害復原 (Geo-DR) 功能。 提供 Geo-DR 功能的單一網站資料庫是分散在世界各地的資料庫嚴格子集。 

使用 DocumentDB 周全的全域散發功能，藉由在資料庫記錄檔採用 Lambda 模式 (例如，[AWS DynamoDB 複寫](https://github.com/awslabs/dynamodb-cross-region-library/blob/master/README.md))，或跨多個區域執行「重複寫入」，開發人員就不必建置自己的複寫結構 。 由於無法確保這類方法的正確性，並提供健全的 SLA，因此我們不建議使用這些方法。 

在本文中，我們提供 DocumentDB 的全域散發功能概觀。 我們也會說明 DocumentDB 唯一能提供完整 SLA 的方法。 

## <a id="EnableGlobalDistribution"></a>啟用周全的全球發散功能
DocumentDB 提供下列功能，讓您輕鬆地撰寫全球規模的應用程式。 這些功能都可以透過 DocumentDB 以資源提供者為主的 [REST API](https://docs.microsoft.com/rest/api/documentdbresourceprovider/) 和 Azure 入口網站來取得。

### <a id="RegionalPresence"></a>遍及各區，無所不在 
Azure 透過不斷連上新的地理區域，而日益普及。 根據預設，新的 Azure 區域全部可使用 DocumentDB。 這可讓您的地理區域與 DocumentDB 資料庫帳戶建立關聯，因為 Azure 會為企業開啟新的區域。

**根據預設，Azure 區域全部可使用 DocumentDB**

![Azure 區域全部可使用 DocumentDB](./media/documentdb-distribute-data-globally/azure-regions.png)

### <a id="UnlimitedRegionsPerAccount"></a>將數目不拘的區域與 DocumentDB 資料庫帳戶產生關聯
DocumentDB 可讓您將任何數目的 Azure 區域與 DocumentDB 資料庫帳戶產生關聯。 除了異地隔離限制 (例如中國、德國) 之外，可和DocumentDB 資料庫帳戶產生關聯的區域數沒有限制。 下圖顯示設定為橫跨 21 個 Azure 區域的資料庫帳戶。  

**橫跨 21 個 Azure 區域的租用戶 DocumentDB 資料庫帳戶**

![橫跨 21 個 Azure 區域的 DocumentDB 資料庫帳戶](./media/documentdb-distribute-data-globally/documentdb-spanning-regions.png)

### <a id="PolicyBasedGeoFencing"></a>原則式異地隔離限制
DocumentDB 的設計可取得原則式異地隔離限制功能。 異地隔離這個重要元件可確保資料管理和法規遵循限制，並防止特定地區與您的帳戶產生關聯。 異地隔離範例包括 (但不限於) 不超過主權雲端 (例如中國和德國) 或政府稅務界限 (例如澳洲) 界定各區域的全域散發範疇。 使用 Azure 訂用帳戶的中繼資料控制原則。

### <a id="DynamicallyAddRegions"></a>動態新增和移除區域
DocumentDB 可讓您隨時對資料庫帳戶新增 (關聯) 或移除 (中斷關聯) 區域 (請參閱[上圖](#UnlimitedRegionsPerAccount))。 藉由以平行方式跨資料分割複寫資料，DocumentDB 可確保當新區域上線時，30 分鐘以內就能在世界各地以高達 100 TB 的速度使用 DocumentDB。 

### <a id="FailoverPriorities"></a>容錯移轉優先順序
若要在有數個區域發生中斷時完全控制區域容錯移轉的順序，DocumentDB 可讓您將優先順序和資料庫帳戶相關的各個區域 (請參閱下圖)。 DocumentDB 可確保會以您所指定的優先順序自動進行容錯移轉。

**DocumentDB 的租用戶能為和資料庫帳戶相關的區域設定容錯移轉優先順序 (右窗格)**

![使用 Azure DocumentDB 設定容錯移轉優先順序](./media/documentdb-distribute-data-globally/documentdb-failover-priorities.png)

### <a id="OfflineRegions"></a>讓區域動態「離線」
DocumentDB 可讓特定區域的資料庫帳戶離線，稍後再讓它重新上線。 標示為離線的區域不會主動參與複寫，也不是容錯移轉順序的一部分。 這可讓您在對應用程式推出可能有風險的升級時，將上一個已知良好的資料庫映像凍結在其中一個讀取區域。

### <a id="ConsistencyLevels"></a>多個定義完善的一致性模型可供全球複寫資料庫使用
DocumentDB 公開由 SLA 所支援的[多個定義完善的一致性層級](documentdb-consistency-levels.md)。 您可以根據工作負載/案例選擇特定的一致性模型 (從可用的選項清單中)。 

### <a id="TunableConsistency"></a>可微調全球複寫資料庫不一致之處
DocumentDB 可讓您在執行階段以程式設計方式覆寫和放寬個別要求的預設一致性選擇。 

### <a id="DynamicallyConfigurableReadWriteRegions"></a>可動態設定讀取和寫入區域
DocumentDB 可讓您將區域 (和資料庫相關聯) 設定為「讀取」、「寫入」或「讀取/寫入」區域。 

### <a id="ElasticallyScaleThroughput"></a>在 Azure 區域，來彈性調整的輸送量
您能以程式設計方式佈建輸送量，彈性地調整 DocumentDB 集合。 散發集合的所有區域都會套用該輸送量。

### <a id="GeoLocalReadsAndWrites"></a>異地本機讀取和寫入
資料庫分散在世界各地的主要優點是世界各地資料都提供低延遲存取。 DocumentDB 可在 P99 進行各種資料庫作業時確保低延遲。 它可確保將所有讀取都路由到最接近的本機讀取區域。 使用區域本機的仲裁發出讀取，來提供讀取要求。這同樣適用於寫入。 只有大多數複本在本機永久認可寫入，而不限在遠端複本上認可寫入之後，寫入才會受到認可。 換句話說，DocumentDB 的複寫通訊協定運作時，假設讀取和寫入仲裁永遠都在發出要求的個別讀取和寫入區域本機。

### <a id="ManualFailover"></a>手動起始區域的容錯移轉
DocumentDB 可讓您觸發資料庫帳戶的容錯移轉，驗證整個應用程式的*端對端*可用性屬性 (除資料庫之外)。 由於可確保失敗偵測和選出領導者的安全性與作用中屬性，因此 DocumentDB 可確保租用戶起始的手動容錯移轉作業*零資料遺失*。

### <a id="AutomaticFailover"></a>自動容錯移轉
萬一有一或多個區域中斷，DocumentDB 支援自動容錯移轉。 進行區域容錯移轉時，DocumentDB 會維持其讀取延遲時間、執行時間可用性、一致性及輸送量的 SLA。 DocumentDB 提供完成自動容錯移轉作業的持續時間上限。 資料很可能會在區域中斷時的這段時間遺失。

### <a id="GranularFailover"></a>專為不同的容錯移轉資料粒度設計
目前是在資料庫帳戶的資料粒度公開自動和手動容錯移轉功能。 注意，在內部 DocumentDB 的設計是以更精細的資料庫、集合，或甚至是 (擁有一系列索引鍵的集合) 資料分割的資料粒度*自動*容錯移轉。 

### <a id="MultiHomingAPIs"></a>DocumentDB 中的多路連接 API
DocumentDB 可讓您使用邏輯 (區域無從驗證) 或實體 (區域特定) 端點來和資料庫互動。 使用邏輯端點，萬一進行容錯移轉時，可確保以透明的方式多路連接應用程式。 後者實體端點可微調控制應用程式，將讀取和寫入將重新導向特定區域。

### <a id="ReadPreferencesAPIforMongoDB"></a> API for MongoDB 中可設定的 讀取喜好設定
API for MongoDB 可讓您針對全球分散式資料庫，指定集合的讀取喜好設定。 為了兼顧低延遲讀取和全球高可用性，建議將集合的讀取喜好設定設為 [最接近]。 [最接近] 讀取喜好設定會設定為從最近的區域讀取。

```csharp
var collection = database.GetCollection<BsonDocument>(collectionName);
collection = collection.WithReadPreference(new ReadPreference(ReadPreferenceMode.Nearest));
```

如果應用程式具有主要讀取/寫入區域和次要地區來因應災害復原 (DR) 情況，我們建議將集合的讀取喜好設定設為 [慣用次要]。 [慣用次要] 讀取喜好設定會設定當主要區域無法使用時，從次要地區讀取。

```csharp
var collection = database.GetCollection<BsonDocument>(collectionName);
collection = collection.WithReadPreference(new ReadPreference(ReadPreferenceMode.SecondaryPreferred));
```

最後，如果您想要手動指定讀取區域， 您可以在讀取喜好設定內設定區域標記。

```csharp
var collection = database.GetCollection<BsonDocument>(collectionName);
var tag = new Tag("region", "Southeast Asia");
collection = collection.WithReadPreference(new ReadPreference(ReadPreferenceMode.Secondary, new[] { new TagSet(new[] { tag }) }));
```

### <a id="TransparentSchemaMigration"></a>透明且一致的資料庫結構描述與索引移轉 
DocumentDB 完全[無從驗證結構描述 (schema agnostic)](http://www.vldb.org/pvldb/vol8/p1668-shukla.pdf)。 資料庫引擎的獨特設計能自動且同步對內嵌的所有資料編製索引，不需要您提供任何結構描述或次要索引。 這可讓您快速地反覆查看分散在世界各地的應用程式，而不必擔心資料庫結構描述和索引移轉，或協調多階段應用程式推出結構描述變更。 DocumentDB 可確保明確由您對編製索引原則所做的變更，不會造成效能或可用性降低。  

### <a id="ComprehensiveSLAs"></a>全方位 SLA (不光只有高可用性)
DocumentDB 是分散在世界各地的資料庫服務，還提供定義完善的 SLA 將資料庫視為一體，全面處理「資料遺失」、「可用性」、「在 P99 的延遲」、「輸送量」及「一致性」，無論與資料庫相關聯的區域數目為何。  

## <a id="LatencyGuarantees"></a>延遲保證
像是 DocumentDB 資料庫服務分散在世界各地的主要優點是您在世界各地的資料都提供低延遲存取。 DocumentDB 可在 P99 進行各種資料庫作業時確保低延遲。 DocumentDB 採用的複寫通訊協定可確保一律在用戶端所在區域執行資料庫作業 (理想的情況是，讀取和寫入)。 DocumentDB 的延遲 SLA 包括 P99，用於各種要求和回應大小的讀取、(同步) 編製索引的寫入和查詢。 寫入延遲保證，包含持久的多數仲裁在本機資料中心內認可。

### <a id="LatencyAndConsistency"></a>延遲和一致性的關聯性 
要在世界各地安裝分散在世界各地的服務提供絕佳一致性，需要同時複寫，然後寫入，或同時執行跨區域讀取 – 光線速度和廣域網路網路可靠性決定強式一致性會有高延遲和低可用性的資料庫作業。 因此，為了保證在 P99 提供低延遲和 99.99 可用性，服務必須採用非同步複寫。 這轉而要求服務還必須提供[定義完善、寬鬆的一致性選項](documentdb-consistency-levels.md) – 比強式稍差 (以保證提供低延遲與高可用性)，且最好比「最終」一致性更強 (提供直覺式程式設計模型)。

DocumentDB 確保不需要讀取作業連絡跨多個區域的複本，提供特定一致性層級保證。 同樣地，它可確保 跨所有區域複寫資料時，不會封鎖寫入作業 (也就是寫入會以非同步的方式跨區域複寫)。 對於多重區域資料庫帳戶，可以提供多個寬鬆的一致性層級。 

### <a id="LatencyAndAvailability"></a>延遲和可用性的關聯性 
延遲和可用性就像錢幣的一體兩面。 我們會討論穩定狀態下的作業延遲和可面對失敗時的可用性。 從應用程式的觀點而言，執行緩慢的資料庫作業和無法使用的資料庫，並無法辨別。 

DocumentDB 因此就各種資料庫作業的延遲提供絕對的時間上限，來區分高延遲和無法使用的情況。 如果要完成資料庫作業花費的時間超過上限，DocumentDB 就會傳回逾時錯誤。 DocumentDB 可用性 SLA 可確保針對可用性 SLA 計算逾時。 

### <a id="LatencyAndThroughput"></a>延遲和輸送量的關聯性
DocumentDB 不會要您在延遲和輸送量之間做出選擇。 而是接受 SLA，用於在 P99 的延遲並傳遞您佈建的輸送量。 

## <a id="ConsistencyGuarantees"></a>一致性保證
雖然[強式一致性模型](http://cs.brown.edu/~mph/HerlihyW90/p463-herlihy.pdf)是可程式性的黃金標準，但要付出的很大代價是高延遲 (穩定狀態下) 和喪失可用性 (面對失敗)。 

DocumentDB 會提供定義完善的程式設計模型，讓您理解有關複寫資料的一致性。 為了讓您建立多路連接的應用程式，DocumentDB 所公開的一致性模型目的是為了讓區域無從驗證，還有不依存於提供讀取和寫入的區域。 

DocumentDB 的一致性 SLA 保證 100% 的讀取要求都將符合您要求的一致性層級 (資料庫帳戶上的預設一致性層級或要求上的覆寫值)。 如果和一致性層級相關聯的所有一致性保證全都滿足，就會將讀取要求視為已符合一致性 SLA。 下表擷取的一致性保證對應至 DocumentDB 提供的特定一致性層級。

**和 DocumentDB 中所指定一致性層級相關聯的一致性保證**

<table>
    <tr>
        <td><strong>一致性層級</strong></td>
        <td><strong>一致性特性</strong></td>
        <td><strong>SLA</strong></td>
    </tr>
    <tr>
        <td rowspan="3">工作階段</td>
        <td>讀取自己的寫入</td>
        <td>100%</td>
    </tr>
    <tr>
        <td>單純讀取</td>
        <td>100%</td>
    </tr>
    <tr>
        <td>一致的前置詞</td>
        <td>100%</td>
    </tr>
    <tr>
        <td rowspan="3">界限-陳舊</td>
        <td>單純讀取 (區域內)</td>
        <td>100%</td>
    </tr>
    <tr>
        <td>一致的前置詞</td>
        <td>100%</td>
    </tr>
    <tr>
        <td>過期限定 &lt; K、T</td>
        <td>100%</td>
    </tr>
    <tr>
        <td>最終</td>
        <td>一致的前置詞</td>
        <td>100%</td>
    </tr>
    <tr>
        <td>重要事項</td>
        <td>線性</td>
        <td>100%</td>
    </tr>
</table>

### <a id="ConsistencyAndAvailability"></a>一致性和可用性的關聯性
[CAP 定理 (CAP theorem)](https://people.eecs.berkeley.edu/~brewer/cs262b-2004/PODC-keynote.pdf) [不可達的結果 (impossibility result)](http://www.glassbeam.com/sites/all/themes/glassbeam/images/blog/10.1.1.67.6951.pdf) 證明系統面對失敗時，確實無法維持可用，同時提供線性一致性。 資料庫服務必須選擇 CP 或 AP - CP 系統會為了線性一致性而放棄可用性，而 AP 系統則會為了可用性而放棄[線性一致性 (英文)](http://cs.brown.edu/~mph/HerlihyW90/p463-herlihy.pdf)。 DocumentDB 永遠不會違反要求的一致性層級，因此使其正式成為 CP 系統。 然而實際上，一致性並不主張孤注一擲 – 除了線性和最終一致性之間的一致性範圍，還有數個定義完善的一致性模型。 在 DocumentDB 中，我們試著利用真實世界的適用性和直覺式程式設計模型，來識別數個寬鬆的一致性模型。 DocumentDB 透過提供 99.99 可用性 SLA 以及[多個寬鬆但定義完善的一致性層級](documentdb-consistency-levels.md)，瀏覽一致性可用性的權衡取捨。 

### <a id="ConsistencyAndAvailability"></a>一致性和延遲的關聯性
Daniel Abadi 教授提出更全面的 CAP 變化稱為 [PACELC](http://cs-www.cs.yale.edu/homes/dna/papers/abadi-pacelc.pdf)，也能解釋在穩定狀態下對延遲和一致性的權衡取捨。 它指出在穩定狀態下，資料庫系統必須在一致性和延遲之間做出選擇。 使用多個寬鬆的一致性模型 (受非同步覆寫和本機讀取、寫入仲裁支援)，DocumentDB 可確保所有讀取和寫入都是分別在讀取和寫入區域本機進行。  這可讓 DocumentDB 在一致性層級的區域內提供低延遲保證。  

### <a id="ConsistencyAndThroughput"></a>一致性和輸送量的關聯性
由於實作特定一致性模型，取決於選擇的[仲裁類型](http://cs.brown.edu/~mph/HerlihyW90/p463-herlihy.pdf)，而輸送量也根據一致性選擇而不同。 例如，在 DocumentDB 中，進行強式一致的讀取時，輸送量大約是進行最終一致讀取的一半。 
 
**DocumentDB 中特定一致性層級的讀取產能的關聯性**

![一致性和輸送量之間的關聯性](./media/documentdb-distribute-data-globally/consistency-and-throughput.png)

## <a id="ThroughputGuarantees"></a>輸送量保證 
DocumentDB 可讓您視需求彈性地跨不同的區域調整輸送量 (以及儲存體)。 

**單一 DocumentDB 集合經過分割 (跨三個分區)，然後散發在三個 Azure 區域**

![Azure DocumentDB 已散發並分割集合](./media/documentdb-distribute-data-globally/documentdb-throughput.png)

DocumentDB 集合可使用兩個維度來散發 – 區域內和跨區域。 方式如下： 

* 在單一區域內，以資源資料分割的方式擴充 DocumentDB 集合。 藉由在一組複本間複寫狀態機器，每個資源資料分割都會管理一組索引鍵且極為一致並高度可用。 DocumentDB 是全面管理資源的系統，其中的資源資料分割負責依照系統資源配置的預算，傳遞自己的輸送量。 調整 DocumentDB 集合的大小完全透明 – DocumentDB 會管理資源的資料分割，並視需要予以分割及合併。 
* 接著將每個資源資料分割散發在多個區域。 各個區域表單分割集的資源資料分割都有同一組索引鍵 (請參閱[上圖](#ThroughputGuarantees))。  使用狀態機器複寫，跨多個區域協調分割集內的資源資料分割。 根據設定的一致性層級，分割集內的資源資料分割設定為動態使用不同的拓撲 (例如，星形、菊輪鍊、樹狀目錄等)。 

透過回應相當靈敏的資料分割管理、負載平衡及嚴格的資源管理，DocumentDB 可讓您彈性地調整 DocumentDB 集合上數個 Azure 區域的輸送量。 變更集合上的輸送量是 DocumentDB 中的執行階段作業 - 和其他資料庫作業一樣，DocumentDB 可為變更輸送量的要求保證其延遲絕對時間上限。 例如，下圖顯示的客戶集合會根據需求彈性地佈建輸送量 (在兩個區域有每秒 1M-10M 個要求)。
 
**彈性地佈建輸送量的客戶集合 (每秒&1;M-10M 個要求)**

![Azure DocumentDB 可彈性佈建輸送量](./media/documentdb-distribute-data-globally/documentdb-elastic-throughput.png)

### <a id="ThroughputAndConsistency"></a>輸送量和一致性的關聯性 
與[一致性和輸送量的關聯性](#ConsistencyAndThroughput)相同。

### <a id="ThroughputAndAvailability"></a>輸送量和可用性的關聯性
DocumentDB 會在變更輸送量時維持其可用性。 DocumentDB 會明確管理資料分割 (例如，分割、合併、複製作業)，並在應用程式彈性增加或減少輸送量時，確保這些作業不會降低效能或可用性。 

## <a id="AvailabilityGuarantees"></a>可用性保證
DocumentDB 針對每個資料面和控制面作業提供 99.99%的執行時間可用性 SLA。 如先前所述，DocumentDB 的可用性保證包含每個資料面和控制面作業的延遲絕對上限。 可用性保證堅定不移，而且不會隨區域數目或區域之間的地理距離變化。 可用性保證適用於手動以及自動容錯移轉。 DocumentDB 提供透明的多路連接的 API，可以確保您的應用程式在進行容錯移轉時，能操作邏輯端點，並明確地將要求路由至新的區域。 換句話說，進行區域性容錯移轉並維持可用性 SLA 的情況下，不需要重新部署您的應用程式。

### <a id="AvailabilityAndConsistency"></a>可用性和一致性、延遲及輸送量的關聯性
可用性和一致性、延遲及輸送量的關聯性，如[一致性和可用性的關聯性](#ConsistencyAndAvailability)、[延遲和可用性的關聯性](#LatencyAndAvailability)及[輸送量和可用性的關聯性](#ThroughputAndAvailability)中所述。 

## <a id="GuaranteesAgainstDataLoss"></a>「資料遺失」的保證和系統行為
在 DocumentDB 中，每個資料分割 (集合) 都有數個複本散佈在至少 10-20 個容錯網域，藉以提供高可用性。 所有寫入在對用戶端認可之前，均會由複本的多數仲裁同步且永久地認可。 協調分散在多個區域的資料分割，需要套用非同步複寫。 DocumentDB 保證租用戶起始的手動容錯移轉零資料遺失。 在自動容錯移轉期間，DocumentDB 在其 SLA 中保證資料遺失時間的上限為已設定的限定過期。

## <a id="CustomerFacingSLAMetrics"></a>客戶面向 SLA 計量
DocumentDB 會直接公開輸送量、延遲、一致性和可用性計量。 這些計量可以程式設計方式或透過 Azure 入口網站存取 (請見下圖)。 您也可以設定各種使用 Azure Application Insights 的臨界值警示。
 
**每個租用戶都能直接取得一致性、延遲、輸送量和可用性計量資訊**

![Azure DocumentDB 客戶可見 SLA 計量](./media/documentdb-distribute-data-globally/documentdb-customer-slas.png)

## <a id="Next Steps"></a>後續步驟
* 若要使用 Azure 入口網站對 DocumentDB 帳戶實作全球散發，請參閱[如何使用 Azure 入口網站執行 DocumentDB 全球資料庫複寫](documentdb-portal-global-replication.md)。
* 若要了解如何使用 DocumentDB 實作多重主機架構，請參閱[使用 Azure DocumentDB 的多重主機資料庫架構](documentdb-multi-region-writers.md)。
* 若要深入了解自動和手動容錯移轉如何在 DocumentDB 中運作，請參閱[Azure DocumentDB 的區域性容錯移轉](documentdb-regional-failovers.md)。

## <a id="References"></a>參考
1. Eric Brewer。 [面對健全的分散式系統 (英文)](https://people.eecs.berkeley.edu/~brewer/cs262b-2004/PODC-keynote.pdf)
2. Eric Brewer。 [CAP&12; 年後 - 規則如何變化(英文)](http://informatik.unibas.ch/fileadmin/Lectures/HS2012/CS341/workshops/reportsAndSlides/PresentationKevinUrban.pdf)
3. Gilbert、Lynch。 - [一致、可用、資料分割容錯 Web 服務的布魯爾推測和可行性 (英文)](http://www.glassbeam.com/sites/all/themes/glassbeam/images/blog/10.1.1.67.6951.pdf)
4. Daniel Abadi。 [現代分散式資料庫系統設計的一致性取捨 (英文)](http://cs-www.cs.yale.edu/homes/dna/papers/abadi-pacelc.pdf)
5. Martin Kleppmann。 [不要再以 CP 或 AP 來稱呼資料庫 (英文)](https://martin.kleppmann.com/2015/05/11/please-stop-calling-databases-cp-or-ap.html)
6. Peter Bailis et al。[實際部分仲裁的隨機限定過期 (PBS) (英文)](http://vldb.org/pvldb/vol5/p776_peterbailis_vldb2012.pdf)
7. Naor 和 Wool。 [仲裁系統的負載、容量及可用性 (英文)](http://www.cs.utexas.edu/~lorenzo/corsi/cs395t/04S/notes/naor98load.pdf)
8. Herlihy 和 Wing。 [線性：並行物件的正確性條件 (英文)](http://cs.brown.edu/~mph/HerlihyW90/p463-herlihy.pdf)
9. Azure DocumentDB SLA (上次於 2016 年 12 月更新)

