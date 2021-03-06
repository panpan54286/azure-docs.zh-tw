---
title: "Azure 媒體服務的分散 MP4 即時內嵌規格 | Microsoft Docs"
description: "此規格說明以分散 MP4 為基礎的即時資料流內嵌通訊協定和格式 (Microsoft Azure 媒體服務適用)。 Microsoft Azure 媒體服務提供即時資料流服務，讓客戶使用 Microsoft Azure 作為雲端平台來即時串流即時事件和廣播內容。 本文也討論建置高度備援且穩固即時內嵌機制的最佳作法。"
services: media-services
documentationcenter: 
author: cenkdin
manager: erikre
editor: 
ms.assetid: 43fac263-a5ea-44af-8dd5-cc88e423b4de
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/07/2016
ms.author: cenkd;juliako
translationtype: Human Translation
ms.sourcegitcommit: e126076717eac275914cb438ffe14667aad6f7c8
ms.openlocfilehash: 307c9a377fce32c056a54d35f173efd1bafc4df5
ms.lasthandoff: 02/16/2017


---
# <a name="azure-media-services-fragmented-mp4-live-ingest-specification"></a>Azure 媒體服務的分散 MP4 即時內嵌規格
此規格說明以分散 MP4 為基礎的即時資料流內嵌通訊協定和格式 (Microsoft Azure 媒體服務適用)。 Microsoft Azure 媒體服務提供即時資料流服務，讓客戶使用 Microsoft Azure 作為雲端平台來即時串流即時事件和廣播內容。 本文也討論建置高度備援且穩固即時內嵌機制的最佳作法。

## <a name="1-conformance-notation"></a>1.一致性標記法
本文中「必須」、「不得」、「必要」、「可」、「不可」、「應該」、「不應」、「建議」、「可能」及「選擇性」等關鍵字的解釋如 RFC 2119 中所述。

## <a name="2-service-diagram"></a>2.服務圖表
下圖顯示 Microsoft Azure 媒體服務中即時資料流服務的概況架構：

1. 即時編碼器會將即時摘要推送到透過 Microsoft Azure 媒體服務 SDK 建立並佈建的通道。
2. Microsoft Azure 媒體服務中的通道、程式與資料流端點會處理所有的即時資料流功能，包括內嵌、格式化、雲端 DVR、安全性、延展性和備援能力。
3. 客戶可以選擇性地在資料流端點與用戶端端點之間選擇部署 CDN 層。
4. 用戶端會使用 HTTP 彈性資料流通訊協定 (例如 Smooth Streaming、DASH 或 HLS) 從資料流端點開始串流。

![image1][image1]

## <a name="3-bit-stream-format--iso-14496-12-fragmented-mp4"></a>3.位元資料流格式 – ISO 14496-12 分散 MP4
本文中所討論即時資料流內嵌的電傳格式是根據 [ISO-14496-12]。 請參閱 [[MS-SSTR]](http://msdn.microsoft.com/library/ff469518.aspx) 中有關分散 MP4 格式、點播視訊及即時串流內嵌的詳細說明。

### <a name="live-ingest-format-definitions"></a>即時內嵌格式定義
以下的特殊格式定義清單適用於 Microsoft Azure 媒體服務的即時內嵌：

1. ‘ftyp’、LiveServerManifestBox 及 ‘moov’ 方塊「必須」連同每個要求 (HTTP POST) 一起傳送。  「必須」在資料流的開頭傳送，並且若要繼續資料流內嵌時，編碼器必須每次都重新連線。  如需詳細資訊，請參閱 [1] 中的第 6 節。
2. [1] 的第 3.3.2 節為即時內嵌定義名為 StreamManifestBox 的選用方塊。 Microsoft Azure 負載平衡器的路由邏輯使得此方塊的使用者方式已被取代，當內嵌到 Microsoft Azure 媒體服務時「不應該」有此方塊存在。 如果有此方塊，則 Azure 媒體服務會無訊息地將其忽略。
3. 每個片段必須有在 [1] 的 3.2.3.2 中定義的 TrackFragmentExtendedHeaderBox。
4. 「應該」使用第 2 版的 TrackFragmentExtendedHeaderBox，才能在資料中心中使用相同的 URL 產生媒體區段。 如需跨資料中心容錯移轉以索引為基礎的資料流格式，例如 Apple HTTP 即時資料流 (HLS) 和以索引為基礎的 MPEG DASH，片段索引欄位是「必要」的。  若要啟用跨資料中心容錯移轉，多個編碼器之間的片段索引「必須」同步處理，後續的每個媒體片段會增加 1，即使是跨編碼器重新啟動或失敗。
5. [1] 中的 3.3.6 節定義名為 MovieFragmentRandomAccessBox ('mfra') 的方塊，此方塊「可能」會在即時內嵌結束時傳送，以表示通道 EOS (結束資料流)。 Azure 媒體服務的摘要邏輯使得 EOS (結束資料流) 的使用方式已被取代，「不應該」傳送即時內嵌的 ‘mfra’ 方塊。 如果已傳送，Azure 媒體服務會無訊息地將其忽略。 建議使用[重設通道](https://docs.microsoft.com/rest/api/media/operations/channel#reset_channels)來重設內點的狀態，也建議使用[程式停止](https://msdn.microsoft.com/library/azure/dn783463.aspx#stop_programs)結束簡報與資料流。
6. MP4 片段持續時間「應該」是固定的，才能減少用戶端資訊清單的大小，並透過使用重複標記來改善用戶端下載啟發學習法。  為了補償非整數的畫面播放速率，持續時間「可能」會變動。
7. MP4 片段持續期間「應該」大約在 2 到 6 秒之間。
8. 「應該」以遞增順序送達 MP4 片段時間戳記和索引 (TrackFragmentExtendedHeaderBox fragment_ absolute_ time 和 fragment_index)。  雖然 Azure 媒體服務在複製片段方面很有彈性，但是其在根據媒體時間軸將片段重新排列的功能非常有限。

## <a name="4-protocol-format--http"></a>4.通訊協定格式 – HTTP
Microsoft Azure 媒體服務的 ISO 分散 MP4 即時內嵌使用標準的長時間執行 HTTP POST 要求，將以分散 MP4 格式封裝的編碼媒體資料傳輸到服務。 每個 HTTP POST 會傳送完整的分散 MP4 位元資料流 ("Stream")，其開頭為標頭方塊 ('ftyp'、"Live Server Manifest Box" 及 'moov' 方塊)，接著連續的一連串片段 (‘moof’ 與 ‘mdat’ 方塊)。 請參閱 [1] 的第 9.2 節中有關 HTTP POST 要求的 URL 語法說明。 以下是 POST URL 的範例： 

    http://customer.channel.mediaservices.windows.net/ingest.isml/streams(720p)

### <a name="requirements"></a>需求
詳細需求如下：

1. 編碼器「應該」使用相同的內嵌 URL 來傳送空白 “body” (零內容長度) 的 HTTP POST 要求，藉此開始廣播。 這可協助快速偵測即時內嵌端點是否有效，以及是否需要任何的驗證或其他條件。 伺服器無法對每個 HTTP 通訊協定傳回 HTTP 回應，直到收到整個要求為止，包括 POST 內文。 由於即時事件其長時間執行的性質，若無此步驟，編碼器在完成傳送所有的資料之前，無法偵測到任何錯誤。
2. 編碼器「必須」處理任何因為 (1) 而造成的錯誤或驗證挑戰。 如果 (1) 後續的回應為 200，則繼續進行。
3. 編碼器「必須」以分散 MP4 資料流開始新的 HTTP POST 要求。  裝載的開頭「必須」是標頭方塊，後面加上片段。  請注意，由於上一個要求在資料流結束前終止，因此即使編碼器必須重新連線，‘ftyp’、“Live Server Manifest Box” 及 ‘moov’ 方塊 (依此順序) 仍「必須」連同每個要求傳送。 
4. 因為無法預測即時事件的整個內容長度，編碼器「必須」使用區塊傳送編碼上傳。
5. 當事件結束時，在傳送最後一個片段之後，編碼器「必須」依正常程序結束區塊傳輸編碼的訊息序列 (大部分的 HTTP 用戶端堆疊會自動處理)。 編碼器「必須」等候服務傳回最後的回應碼，然後終止連線。 
6. 如 [1] 中第 9.2 節所述，編碼器「不得」使用 Events() 名詞，才能即時內嵌至 Microsoft Azure 媒體服務。
7. 如果 HTTP POST 要求在資料流結束時終止或逾時，並發生 TCP 錯誤，則編碼器「必須」使用新連線發出新的 POST 要求，並遵循上述的需求及下列額外的需求：編碼器「必須」為資料流中的每個資料軌重新傳送前兩個 MP4 片段並繼續進行，但不在媒體時間軸上造成不連續。  為每個資料軌重新傳送最後兩個 MP4 片段，可確保不會遺失資料。  換句話說，如果資料流包含音訊和視訊播放軌，而且目前的 POST 要求失敗，則編碼器必須重新連線，然後為音訊播放軌重新傳送最後兩個片段 (先前已成功傳送)，為視訊播放軌重新傳送最後兩個片段 (先前已成功傳送)，以確保不會遺失任何資料。  編碼器「必須」維護媒體片段的 “forward” 緩衝區，當重新連線時會重新傳送此緩衝區。

## <a name="5-timescale"></a>5.時幅
[[MS-SSTR]](https://msdn.microsoft.com/library/ff469518.aspx) 描述 SmoothStreamingMedia (2.2.2.1 節)、StreamElement (2.2.2.3 節)、StreamFragmentElement (2.2.2.6) 和 LiveSMIL (2.2.7.3.1 節) 的「時幅」使用方式。 如果沒有時幅值，則使用的預設值為 10,000,000 (10 MHz)。 雖然 Smooth Streaming 格式規格不會封鎖使用其他的時幅值，但大部分的編碼器實作會使用此預設值 (10 MHz) 來產生 Smooth Streaming 內嵌資料。 由於 [Azure 媒體動態封裝](media-services-dynamic-packaging-overview.md) 功能之故，建議視訊資料流採用 90 kHz 時幅，音訊資料流則採用 44.1 或 48.1 kHz。 如果不同的資料流採用不同的時幅值，則「必須」傳送資料流層級時幅。 請參閱 [[MS-SSTR]](https://msdn.microsoft.com/library/ff469518.aspx)。     

## <a name="6-definition-of-stream"></a>6.「資料流」的定義。
「資料流」是指在撰寫即時簡報、處理資料流容錯移轉和備援案例的即時內嵌中，作業的基本單位。 「資料流」定義為一個唯一的分散 MP4 位元資料流，其中可能包含單一資料軌或多個資料軌。 完整即時簡報可包含一或多個資料流，視即時編碼器組態而定。 以下範例說明各種使用資料流撰寫完整即時簡報的選項。

**範例：** 

客戶想要建立一個即時資料流簡報，其中包含下列音訊/視訊位元速率：

視訊 – 3000kbps、1500kbps、750kbps

音訊 – 128 kbps

### <a name="option-1-all-tracks-in-one-stream"></a>選項 1：一個資料流中所有的資料軌
在此選項中，單一編碼器會產生所有的音訊/視訊資料軌，並將其結合成一個分散 MP4 位元資料流，系統會透過單一 HTTP POST 連線傳送此資料流。 此範例中有此即時簡報的唯一資料流：

![image2][image2]

### <a name="option-2-each-track-in-a-separate-stream"></a>選項 2：每個資料軌分在不同的資料流中
在此選項中，編碼器在每個分散 MP4 位元資料流中只有放一個資料軌，並透過多個分開的 HTTP 連線張貼所有的資料流。 這可透過一或多個編碼器完成。 從即時內嵌的觀點來看，此即時簡報由四個資料流組成。

![image3][image3]

### <a name="option-3-bundle-audio-track-with-the-lowest-bitrate-video-track-into-one-stream"></a>選項 3：將音訊資料軌與位元速率最低的視訊資料軌組合一個資料流
在此選項中，客戶選擇將音訊資料軌與位元速率最低的視訊資料軌組合成一個分散 MP4 位元資料流，並讓另外兩個視訊資料軌分別留在自己的資料流中。 

![image4][image4]

### <a name="summary"></a>摘要
以上「並非」詳盡清單，僅列出此範例中部分可用的內嵌選項。 事實上，即時內嵌支援將資料軌以任何組合分組到資料流。 客戶和編碼器廠商可以根據工程的複雜性、編碼器的能力，以及備援與容錯移轉考量，來選擇自己的實作。 不過請注意，在大部分情況下整個即時簡報只有一個音訊資料軌，因此請務必確保包含音訊資料軌的內嵌資料流是正常的。 這項考量通常可讓音訊資料軌放入自己的資料流 (如選項 2)，或以最低位元速率視訊資料軌建立音訊資料軌 (如選項 3)。 此外若要有更佳的備援與容錯移轉成效，強烈建議針對嵌入到 Microsoft Azure 媒體服務的即時內嵌，將同一個音訊資料軌傳送到兩個不同的資料流 (選項 2 中採用備援音訊資料軌)，或以至少兩個最低位元速率視訊資料軌建置音訊資料軌 (選項 3 中以至少兩個視訊資料流建置)。

## <a name="7-service-failover"></a>7.服務容錯移轉
基於即時資料流的性質，良好的容錯移轉支援是確保服務可用性的關鍵。 Microsoft Azure 媒體服務的設計可處理各種類型的故障，包括網路錯誤、伺服器錯誤、儲存體問題等。當從即時編碼器端搭配適當的容錯移轉邏輯使用時，客戶便可以從雲端達成高度可靠的即時資料流服務。

在本節中，我們將討論服務容錯移轉的案例。 在此案例中，服務的某一處發生故障，且故障將自己記錄成資訊清單中的網路錯誤。 以下是如何實施編碼器以處理服務容錯移轉的一些建議：

1. 建立逾時值為 10 秒的 TCP 連線。  如果嘗試建立連線超過 10 秒，便會中止作業並再試一次。 
2. 傳送 HTTP 要求訊息區塊的逾時值短。  如果目標 MP4 片段持續期間為 N 秒，請使用介於 N 到 2N 秒之間的傳送逾時；例如，如果 MP4 片段持續時間是 6 秒，則使用 6 到 12 的逾時。  如果發生逾時，請重設連線、開啟新連接，並以新的連線繼續資料流內嵌。 
3. 為每個資料軌維護一個循環緩衝區，其中包含最後兩個已成功完整地傳送到服務的片段。  如果資料流的 HTTP POST 要求在資料流結束前終止或逾時，則開啟新的連線，然後開始另一個 HTTP POST 要求、重新傳送資料流標頭、為每個資料軌重新傳送最後兩個片段，但不在媒體時間軸上造成不連續。  這會減少資料遺失的可能性。
4. 建議編碼器「不」要限制在發生 TCP 錯誤後，重新嘗試建立連線或繼續串流的次數。
5. 發生 TCP 錯誤後：
   1. 「必須」關閉目前的連線，且「必須」為新的 HTTP POST 要求建立新的連線。
   2. 新的 HTTP POST URL 「必須」與初始 POST URL 相同。
   3. 新的 HTTP POST 「必須」包括與初始 POST 相同的資料流標頭 (‘ftyp’、“Live Server Manifest Box” 及 ‘moov’ 方塊)。
   4. 「必須」重新傳送為每個資料軌傳送的最後兩個片段，並繼續資料流，但不在媒體時間軸上造成不連續。  MP4 片段時間戳記必須連續增加，甚至可橫跨 HTTP POST 要求。
6. 如果未以可與 MP4 片段持續時間相比的速率傳送資料，則編碼器「應該」終止 HTTP POST 要求。  不傳送資料的 HTTP POST 要求可以防止 Azure 媒體服務在服務更新時，快速地與編碼器中斷連線。  基於這個理由，疏鬆 (廣告訊號) 資料軌的 HTTP POST 「應該」短暫存留，一傳送疏鬆片段便立即終止。

## <a name="8-encoder-failover"></a>8.編碼器容錯移轉
編碼器容錯移轉是第二種容錯移轉案例，此案例必須處理，才能進行端對端即時資料流傳遞。 在此案例中，會在編碼器端發生錯誤狀況。 

![image5][image5]

發生編碼器容錯移轉時，預期在即時內嵌端點上會：

1. 「應該」建立新的編碼器執行個體，以繼續資料流，如上圖所示 (以虛線表示的 3000k 視訊資料流)。
2. 新編碼器「必須」使用與故障執行個體相同的 HTTP POST 要求 URL。
3. 新編碼器的 POST 要求「必須」包含與故障執行個體相同的分散 MP4 標頭方塊。
4. 新編碼器「必須」與所有其他的執行中編碼器正確地同步化，相同的即時簡報才能產生與符合的片段界限同步化的音訊/視訊範例。
5. 新的資料流「必須」在語意上等同上一個資料流，並可在標頭與片段層級互換。
6. 新的編碼器「應該」嘗試減少資料遺失。  媒體片段的 fragment_absolute_time 與 fragment_index「應該」從編碼器上次停止的點開始增加。  fragment_absolute_time 與 fragment_index「應該」連續增加，但允許視需要造成不連續。  Azure 媒體服務將忽略已收到並處理的片段，因此因重新傳送片段端而發生錯誤，會勝過在媒體時間軸上造成不連續。 

## <a name="9-encoder-redundancy"></a>9.編碼器備援
針對某些需要更高可用性與高品質經驗的重要即時事件，建議使用主動對主動備援編碼器，以達成順暢容錯移轉，而不會遺失資料。

![image6][image6]

如上圖所示，有兩組編碼器同時將每個資料流的兩個複本推送到即時服務。 此設定之所以受到支援，是因為 Microsoft Azure 媒體服務能夠根據資料流 ID 與片段時間戳記篩選出重複的片段。 如此所產生的即時資料流與封存將會是所有資料流的單一複本，此複本最有可能從兩個來源彙總而成。 例如，在極端的假設狀況下，只要有一個編碼器 (不一定要同一個) 在任何一個指定的時間點上對每個資料流執行，則自服務產生的即時資料流就會是連續的，不會遺失資料。 

此案例的需求與編碼器容錯移轉的需求幾乎相同，唯一不同處是第二組編碼器會與主要編碼器同時執行。

## <a name="10-service-redundancy"></a>10.服務備援
對於高度備援的全域散佈，有時需要跨區域備份以處理區域災難。 透過展開「編碼器備援」拓樸，客戶可以選擇讓備援服務部署在不同的區域，且該區域與第 2 組編碼器連線。 客戶也可以與 CDN 提供者合作，將 GTM (全域流量管理員) 放在兩個服務部署之前，藉此順暢地路由用戶端流量。 編碼器的需求與「編碼器備援」個案相同，唯一不同處為第二組的編碼器必須指向不同的即時內嵌端點。 下圖顯示這項設定：

![image7][image7]

## <a name="11-special-types-of-ingestion-formats"></a>11.特殊類型的內嵌格式
本節討論某些特殊類型的即時內嵌格式，其設計用來處理某些特定案例。

### <a name="sparse-track"></a>疏鬆資料軌
當以豐富的用戶端經驗傳遞即時資料流簡報時，通常需要傳輸與主要媒體資料時間同步化的事件或頻內訊號。 動態即時廣告插入是其中一個例子。 此類型的事件訊號不同於一般音訊/視訊資料流，因為其疏鬆的本質之故。 換句話說，訊號資料通常不會連續發生，其間隔難以預測。 疏鬆資料軌的概念經過特別設計，可內嵌並廣播頻內訊號資料。

以下為內嵌疏鬆資料軌的建議實作：

1. 建立個別的分散 MP4 位元資料流，其中只包含疏鬆資料軌，但不包含音訊/視訊資料軌。
2. 在如 [1] 的第 6 節定義的 “Live Server Manifest Box” 中，使用 “parentTrackName” 參數指定上層資料軌的名稱。 如需詳細資訊，請參閱 [1] 中的第 4.2.1.2.1.2 節。
3. 在 “Live Server Manifest Box” 中，manifestOutput「必須」設為 “true”。
4. 基於訊號事件的疏鬆特性，建議：
   1. 即時事件開始時，編碼器會將初始標頭方塊傳送給服務，這可讓該服務在用戶端資訊清單中註冊疏鬆資料軌。
   2. 未傳送資料時，編碼器「應該」終止 HTTP POST 要求。  不傳送資料且長期執行的 HTTP POST 可在服務更新或伺服器重新開機時，防止 Azure 媒體服務快速地與編碼器中斷連線，因為會在通訊端上的接收作業中暫時封鎖媒體伺服器。 
   3. 在此期間若沒有可用的訊號資料時，編碼器「應該」關閉 HTTP POST 要求。  POST 要求為作用中時，編碼器「應該」傳送資料 
   4. 傳送疏鬆片段時，編碼器可以設定明確的 Content-Length 標頭 (若有的話)。
   5. 透過新連線傳送疏鬆片段時，編碼器「應該」從標頭方塊開始傳送，接著傳送新片段。 這會處理在中間發生容錯移轉的狀況，並會一直對先前從未看到該疏鬆資料庫的新伺服器建立新的疏鬆連線。
   6. 當讓時間戳記值相等或更大的對應上層資料軌片段可供用戶端使用時，疏鬆資料軌片段便可供用戶端使用。 例如，如果疏鬆片段的時間戳記 t=1000，則預期在用戶端看見視訊 (假設上層資料軌名稱為視訊) 片段時間戳記為 1000 或以上後，便可下載 t=1000 的疏鬆片段。 請注意，實際訊號能夠在簡報時間軸上的不同位置上，非常好地運用在其指定用途。 在上述範例中，t=1000 的疏鬆片段具有 XML 承載，可將廣告插入到數秒後的位置上。
   7. 疏鬆資料軌片段的裝載可以有各種不同的格式 (例如 XML、文字或二進位等)，取決於不同的案例。 

### <a name="redundant-audio-track"></a>備援音訊資料軌
在一般的 HTTP 彈性資料流案例中 (例如 Smooth Streaming 或 DASH)，通常在整個簡報中只有一個音訊資料軌。 具有多個品質層級的視訊資料軌可讓用戶端在錯誤條件中選擇，而音訊資料軌不同於這類資料軌，當內嵌的資料流中有損壞的音訊資料軌時，音訊資料軌會是唯一的故障點。 

為解決此問題，Microsoft Azure 媒體服務支援即時內嵌多餘的音訊資料軌。 其概念與可透過不同的資料流多次傳送音訊資料軌相同。 雖然服務只會在用戶端資訊清單中註冊音訊資料軌一次，但它能夠使用多餘的音訊資料軌作為備份，在主要音訊資料軌發生問題時能夠擷取音訊片段。 為了內嵌多餘的音訊資料軌，編碼器必須：

1. 在多個分散 MP4 位元資料流中建立相同的音訊資料軌。 多餘的音訊資料軌「必須」在語意上與片段時間戳記完全相同，並可在標頭與片段層級互換。
2. 確定所有多餘音訊資料軌在即時伺服器資訊清單中的 “audio” 項目 ([1] 的第 6 節) 全部相同。

以下是多餘音訊資料軌的建議實作：

1. 讓資料流獨自傳送每個唯一的音訊資料軌。  為這些音訊資料軌資料流一一傳送多餘資料流，其中第 2 個資料流與第 1 個資料流唯一的不同處是 HTTP POST URL 中的識別碼：{protocol}://{server address}/{publishing point path}/Streams({identifier})。
2. 使用個別的資料流傳送兩個最低的視訊位元速率。 這些資料流皆「應該」包含每個唯一音訊資料軌的複本。  例如，當支援多國語言時，這些資料流「應該」包含每種語言的音訊資料軌。
3. 使用不同的伺服器 (編碼器) 執行個體來編碼和傳送 (1) 和 (2) 中所提到的多餘資料流。 

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]

[image1]: ./media/media-services-fmp4-live-ingest-overview/media-services-image1.png
[image2]: ./media/media-services-fmp4-live-ingest-overview/media-services-image2.png
[image3]: ./media/media-services-fmp4-live-ingest-overview/media-services-image3.png
[image4]: ./media/media-services-fmp4-live-ingest-overview/media-services-image4.png
[image5]: ./media/media-services-fmp4-live-ingest-overview/media-services-image5.png
[image6]: ./media/media-services-fmp4-live-ingest-overview/media-services-image6.png
[image7]: ./media/media-services-fmp4-live-ingest-overview/media-services-image7.png


