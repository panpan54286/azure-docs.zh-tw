---
title: "使用 ASP.NET Core 建立應用程式的 Web 前端 | Microsoft Docs"
description: "使用 ASP.NET Core Web API 專案對 Web 公開 Service Fabric 應用程式，以及透過 ServiceProxy 進行服務間通訊。"
services: service-fabric
documentationcenter: .net
author: seanmck
manager: timlt
editor: 
ms.assetid: 96176149-69bb-4b06-a72e-ebbfea84454b
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 10/29/2016
ms.author: seanmck
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: faa452d3ba407d5e1ffb3fc4b60b71b2bc64113b


---
# <a name="build-a-web-service-front-end-for-your-application-using-aspnet-core"></a>使用 ASP.NET Core 建置應用程式的 Web 服務前端
根據預設，Azure Service Fabric 服務不提供 Web 的公用介面。 若要對 HTTP 用戶端公開應用程式的功能，您必須建立 Web 專案來做為進入點，然後從該處與個別服務進行通訊。

在本教學課程中，我們將從 [在 Visual Studio 中建立第一個應用程式](service-fabric-create-your-first-application-in-visual-studio.md) 教學課程中斷處來開始講起，並在具狀態計數器服務前面新增 Web 服務。 如果您尚未這麼做，請返回並先逐步進行該教學課程。

## <a name="add-an-aspnet-core-service-to-your-application"></a>將 ASP.NET Core 服務新增至您的應用程式
ASP.NET Core 是輕量型、跨平台的 Web 開發架構，可供您用來建立新式 Web UI 和 Web API。 讓我們將 ASP.NET Web API 專案新增至現有的應用程式。

> [!NOTE]
> 若要完成本教學課程，您需要[安裝 .NET Core 1.0][dotnetcore-install]。
> 
> 

1. 在 [方案總管] 中，以滑鼠右鍵按一下應用程式專案中的 [服務]，然後選擇 [新增] > [新增 Service Fabric Explorer]。
   
    ![將新服務加入至現有的應用程式][vs-add-new-service]
2. 在 [建立服務] 頁面上，選擇 [ASP.NET Core] 並予以命名。
   
    ![在新服務對話方塊中選擇 ASP.NET Web 服務][vs-new-service-dialog]
3. 下一頁會提供一組 ASP.NET Core 專案範本。 請注意，這些都是您在 Service Fabric 應用程式外部建立 ASP.NET Core 專案時所會看到的相同範本。 在本教學課程中，我們會選擇 [Web API] 。 但您可以將相同的概念套用於建置完整的 Web 應用程式。
   
    ![選擇 ASP.NET 專案類型][vs-new-aspnet-project-dialog]
   
    建立 Web API 專案後，您的應用程式中會有兩個服務。 隨著您繼續建置應用程式，您將以完全相同的方式加入更多服務。 每個服務都可以獨立設定版本和升級。

> [!TIP]
> 若要深入了解如何建置 ASP.NET Core 服務，請參閱 [ASP.NET Core 文件](https://docs.asp.net)。
> 
> 

## <a name="run-the-application"></a>執行應用程式
若要了解我們所做的事情，就讓我們部署新的應用程式並看看 ASP.NET Core Web API 範本所提供的預設行為。

1. 在 Visual Studio 按 F5 以進行應用程式偵錯。
2. 部署完成時，Visual Studio 會啟動瀏覽器並瀏覽至 ASP.NET Web API 服務的根目錄，類似 http://localhost:33003 。 連接埠號碼會隨機進行指派，因此可能與您電腦上的連接埠號碼不同。 ASP.NET Core Web API 範本不根提供根目錄的預設行為，因此您將在瀏覽器中收到錯誤。
3. 將 `/api/values` 新增至瀏覽器中的位置。 這將會叫用 Web API 範本中 ValuesController 上的 `Get` 方法。 它會傳回範本所提供的預設回應，也就是包含兩個字串的 JSON 陣列：
   
    ![從 ASP.NET Core Web API 範本傳回的預設值][browser-aspnet-template-values]
   
    在本教學課程結束前，我們會以具狀態服務的最新計數器值取代這些預設值。

## <a name="connect-the-services"></a>連接服務
對於您與可靠服務通訊的方式，Service Fabric 會提供完整的彈性。 在單一應用程式中，您可能擁有可透過 TCP 存取的服務、可透過 HTTP REST API 存取的其他服務，並且還有其他可透過 Web 通訊端存取的服務。 如需可用選項和相關權衡取捨的背景，請參閱 [與服務進行通訊](service-fabric-connect-and-communicate-with-services.md)。 在本教學課程中，我們會遵循下列其中一種更簡單的方法並使用 SDK 中提供的 `ServiceProxy`/`ServiceRemotingListener` 類別。

在 `ServiceProxy` 方法 (模仿遠端程序呼叫或 RPC) 中，您會定義一個介面以做為服務的公用合約。 然後使用該介面來產生 Proxy 類別，以便與服務進行互動。

### <a name="create-the-interface"></a>建立介面
我們會先建立做為具狀態服務與其用戶端之間合約的介面，包括 ASP.NET Core 專案。

1. 在 [方案總管] 中，以滑鼠右鍵按一下您的方案並選擇 [加入] **將** > 。
2. 在左側導覽窗格中選擇 [Visual C#] 項目，然後選取 [類別庫] 範本。 確定 .NET Framework 版本已設定為 **4.5.2**。
   
    ![為具狀態服務建立介面專案][vs-add-class-library-project]
3. 為了讓介面可供 `ServiceProxy`使用，它必須衍生自 IService 介面。 這個介面會包含在其中一個 Service Fabric NuGet 封裝中。 若要新增封裝，請以滑鼠右鍵按一下新的類別庫專案，然後選擇 [管理 NuGet 封裝] 。
4. 搜尋 **Microsoft.ServiceFabric.Services** 封裝並加以安裝。
   
    ![新增服務 NuGet 封裝][vs-services-nuget-package]
5. 在類別庫中，透過單一方法 `GetCountAsync`建立介面，並從 IService 擴充介面。
   
    ```c#
    namespace MyStatefulService.Interfaces
    {
        using Microsoft.ServiceFabric.Services.Remoting;
   
        public interface ICounter: IService
        {
            Task<long> GetCountAsync();
        }
    }
    ```

### <a name="implement-the-interface-in-your-stateful-service"></a>在具狀態服務中實作介面
既然我們已定義介面，我們必須在具狀態服務中實作該介面。

1. 在具狀態服務中，新增含有此介面之類別庫專案的參考。
   
    ![新增具狀態服務中的類別庫專案的參考][vs-add-class-library-reference]
2. 找出繼承自 `StatefulService` 的類別 (例如 `MyStatefulService`)，然後加以擴充以實作 `ICounter` 介面。
   
    ```c#
    using MyStatefulService.Interfaces;
   
    ...
   
    public class MyStatefulService : StatefulService, ICounter
    {        
          // ...
    }
    ```
3. 現在實作 `ICounter` 介面中所定義的單一方法，即 `GetCountAsync`。
   
    ```c#
    public async Task<long> GetCountAsync()
    {
      var myDictionary =
        await this.StateManager.GetOrAddAsync<IReliableDictionary<string, long>>("myDictionary");
   
        using (var tx = this.StateManager.CreateTransaction())
        {          
            var result = await myDictionary.TryGetValueAsync(tx, "Counter");
            return result.HasValue ? result.Value : 0;
        }
    }
    ```

### <a name="expose-the-stateful-service-using-a-service-remoting-listener"></a>使用服務遠端處理接聽程式公開具狀態服務
實作 `ICounter` 介面後，讓具狀態服務可從其他服務呼叫的最後一個步驟是開啟通訊通道。 對於具狀態服務，Service Fabric 會提供稱為 `CreateServiceReplicaListeners`的可覆寫方法。 透過此方法，您可以根據想要為服務啟用的通訊類型，指定一或多個通訊接聽程式。

> [!NOTE]
> 用於開啟無狀態服務之通訊通道的對等方法稱為 `CreateServiceInstanceListeners`。
> 
> 

在此情況下，我們將取代現有的 `CreateServiceReplicaListeners` 方法，並提供 `ServiceRemotingListener` 的執行個體，其會透過 `ServiceProxy` 來建立可從用戶端呼叫的 RPC 端點。  

```c#
using Microsoft.ServiceFabric.Services.Remoting.Runtime;

...

protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
{
    return new List<ServiceReplicaListener>()
    {
        new ServiceReplicaListener(
            (context) =>
                this.CreateServiceRemotingListener(context))
    };
}
```


### <a name="use-the-serviceproxy-class-to-interact-with-the-service"></a>使用 ServiceProxy 類別來與服務互動
現在，具狀態服務已經準備好接收來自其他服務的流量。 因此，剩下的工作便是從 ASP.NET Web 服務加入程式碼來與其通訊。

1. 在 ASP.NET 專案中，新增對含有 `ICounter` 介面之類別庫的參考。
2. 從 [建置] 功能表中，開啟 [組態管理員]。 您應該會看到如下的結果：
   
    ![類別庫顯示為 AnyCPU 的組態管理員][vs-configuration-manager]
   
    請注意，類別庫專案 **MyStatefulService.Interface**是設定為針對 [任何 CPU] 來建置。 若要正確使用 Service Fabric，它的目標必須明確鎖定 x64。 按一下 [平台] 下拉式清單、選擇 [新增] ，然後建立 x64 平台組態。
   
    ![為類別庫建立新平台][vs-create-platform]
3. 將 Microsoft.ServiceFabric.Services 封裝新增至 ASP.NET 專案，就如同先前對類別庫專案所做的一樣。 這會提供 `ServiceProxy` 類別。
4. 在 **Controllers** 資料夾中，開啟 `ValuesController` 類別。 請注意， `Get` 方法目前只會傳回 "value1" 和 "value2" 的硬式編碼字串陣列，這符合我們稍早在瀏覽器中所見的內容。 使用下列程式碼來取代此實作：
   
    ```c#
    using MyStatefulService.Interfaces;
    using Microsoft.ServiceFabric.Services.Remoting.Client;
   
    ...
   
    public async Task<IEnumerable<string>> Get()
    {
        ICounter counter =
            ServiceProxy.Create<ICounter>(new Uri("fabric:/MyApplication/MyStatefulService"), new ServicePartitionKey(0));
   
        long count = await counter.GetCountAsync();
   
        return new string[] { count.ToString() };
    }
    ```
   
    第一行程式碼是關鍵程式碼。 若要建立具狀態服務的 ICounter Proxy，您必須提供兩項資訊：資料分割識別碼和服務名稱。
   
    您可以使用資料分割，根據您定義的索引鍵 (例如客戶識別碼或郵遞區號) 將具狀態服務的狀態劃分為不同的值區，藉此調整具狀態服務。 在我們的簡單應用程式中，具狀態服務只有一個資料分割，所以索引鍵並不重要。 您所提供的任何索引鍵將會造成相同的資料分割。 如深入了解分割服務，請參閱 [如何分割 Service Fabric 可靠的服務](service-fabric-concepts-partitioning.md)。
   
    服務名稱是 fabric:/&lt;application_name&gt;/&lt;service_name&gt; 形式的 URI。
   
    利用這兩項資訊，Service Fabric 即可唯一識別要求應傳送至的電腦。 `ServiceProxy` 類別也會順暢地處理裝載具狀態服務分割區的電腦發生失敗的情況，而另一部電腦則必須進行升級才能取而代之。 此概念讓撰寫用戶端程式碼來處理其他服務變得簡單許多。
   
    一旦擁有 Proxy，我們只需叫用 `GetCountAsync` 方法並傳回其結果。
5. 再次按 F5 以執行修改過的應用程式。 像之前一樣，Visual Studio 會自動啟動瀏覽器並瀏覽至 Web 專案的根目錄。 新增 "api/values" 路徑，您應該會看到傳回的目前計數器值。
   
    ![在瀏覽器中顯示的具狀態計數器值][browser-aspnet-counter-value]
   
    定期重新整理瀏覽器，以查看計數器值更新。

> [!WARNING]
> 範本中提供的 ASP.NET Core 網頁伺服器，稱為 Kestrel， [目前不支援處理直接的網際網路流量](https://docs.asp.net/en/latest/fundamentals/servers.html#kestrel)。 如果是生產案例，請考慮在 [API Management][api-management-landing-page] 或另一個網際網路閘道後方裝載 ASP.NET Core 端點。 請注意，Service Fabric 不支援在 IIS 中的部署。
> 
> 

## <a name="what-about-actors"></a>動作項目呢？
本教學課程著重於新增會與具狀態服務通訊的 Web 前端。 但是您可以依照非常類似的模型來與動作項目交談。 事實上，這比較簡單。

當您建立動作項目專案時，Visual Studio 自動替您產生介面專案。 您可以使用該介面在 Web 專案中產生動作項目 Proxy 來與動作項目進行通訊。 系統會自動提供通訊通道。 因此您不需要如同在本教學課程中處理具狀態服務一樣，建立 `ServiceRemotingListener` 。

## <a name="how-web-services-work-on-your-local-cluster"></a>Web 服務在本機叢集上的運作方式
一般而言，您可以將完全相同的 Service Fabric 應用程式部署到您在本機叢集上部署之多部電腦的叢集，而且極度相信它會如預期般運作。 這是因為本機叢集只是摺疊成單一機器的五個節點的組態。

不過，提到 Web 服務，有一個重要細微差別。 當您的叢集位於負載平衡器後方時，如同在 Azure 中一樣，您必須確定 Web 服務已部署在每一部機器上，因為負載平衡器只會將流量循環配置於各電腦。 您可以將服務的 `InstanceCount` 設定為特殊值 "-1" 來達到此目的。

相反地，當您在本機執行 Web 服務時，您必須確定服務只有一個執行個體正在執行。 否則，您將會與正在相同路徑和連接埠上進行接聽的多個處理序發生衝突。 因此，本機部署的 Web 服務執行個體計數應該設為 "1"。

若要了解如何針對不同環境設定不同的值，請參閱 [管理多個環境的應用程式參數](service-fabric-manage-multiple-environment-app-configuration.md)。

## <a name="next-steps"></a>後續步驟
* [在 Azure 中建立叢集以將您的應用程式部署至雲端](service-fabric-cluster-creation-via-portal.md)
* [深入了解如何與服務進行通訊](service-fabric-connect-and-communicate-with-services.md)
* [深入了解如何分割具狀態服務](service-fabric-concepts-partitioning.md)

<!-- Image References -->

[vs-add-new-service]: ./media/service-fabric-add-a-web-frontend/vs-add-new-service.png
[vs-new-service-dialog]: ./media/service-fabric-add-a-web-frontend/vs-new-service-dialog.png
[vs-new-aspnet-project-dialog]: ./media/service-fabric-add-a-web-frontend/vs-new-aspnet-project-dialog.png
[browser-aspnet-template-values]: ./media/service-fabric-add-a-web-frontend/browser-aspnet-template-values.png
[vs-add-class-library-project]: ./media/service-fabric-add-a-web-frontend/vs-add-class-library-project.png
[vs-add-class-library-reference]: ./media/service-fabric-add-a-web-frontend/vs-add-class-library-reference.png
[vs-services-nuget-package]: ./media/service-fabric-add-a-web-frontend/vs-services-nuget-package.png
[browser-aspnet-counter-value]: ./media/service-fabric-add-a-web-frontend/browser-aspnet-counter-value.png
[vs-configuration-manager]: ./media/service-fabric-add-a-web-frontend/vs-configuration-manager.png
[vs-create-platform]: ./media/service-fabric-add-a-web-frontend/vs-create-platform.png


<!-- external links -->
[dotnetcore-install]: https://www.microsoft.com/net/core#windows
[api-management-landing-page]: https://azure.microsoft.com/en-us/services/api-management/



<!--HONumber=Nov16_HO3-->


