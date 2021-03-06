---
title: "Service Fabric 和容器的概觀 | Microsoft Docs"
description: "Service Fabric 及使用容器來部署微服務應用程式的概觀。 本文提供如何使用容器及 Service Fabric 所提供之功能的概觀。"
services: service-fabric
documentationcenter: .net
author: msfussell
manager: timlt
editor: 
ms.assetid: c98b3fcb-c992-4dd9-b67d-2598a9bf8aab
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 2/6/2017
ms.author: msfussell
translationtype: Human Translation
ms.sourcegitcommit: 93e0493e6a62a70a10b8315142765a3c3892acd1
ms.openlocfilehash: abf5e4bc69aa32ca9af8998ef81de20baae24560


---
# <a name="preview-service-fabric-and-containers"></a>預覽︰Service Fabric 和容器
> [!NOTE]
> 這項功能在 Linux 和 Windows Server 2016 中處於預覽階段。 
>   

## <a name="introduction"></a>簡介
Azure Service Fabric 是跨機器叢集的服務的 [Orchestrator](service-fabric-cluster-resource-manager-introduction.md)，在 Microsoft 的大規模服務中已使用多年且經過最佳化。 開發服務有許多的方式，從 [Service Fabric 程式設計模型](service-fabric-choose-framework.md)，到部署[來賓可執行檔](service-fabric-deploy-existing-app.md)。 根據預設，Service Fabric 會以處理序形式部署和啟動這些服務。 處理序能以最快速度和最密集的程度使用叢集的資源。 Service Fabric 也可以在容器映像中部署服務。 重要的是，您可以在相同應用程式中混合容器中的服務和處理序中的服務。 

## <a name="containers-and-service-fabric-roadmap"></a>容器和 Service Fabric 藍圖
在未來的幾個版本，Service Fabric 會繼續增加對 Windows 和 Linux 上容器的大量支援，包括對聯網、資源條件約束、安全性、診斷、磁碟區驅動程式的增強，尤其是改進 Visual Studio 中的工具支援，如此一來，使用容器映像來部署服務可提供一流的使用體驗。 這種方式可讓您選擇使用容器來封裝現有程式碼 (例如 IIS MVC 應用程式) 或使用 Service Fabric 程式設計模型，而且，因為 Service Fabric 將它們視為同樣的東西，所以您可在應用程式中混用它們，讓您以彈性的方式來部署程式碼。 視您的情況而定，兩者都可兼得。

## <a name="what-are-containers"></a>什麼是容器？
容器是封裝成可個別部署的元件，在相同核心上以隔離的執行個體執行，以利用作業系統提供的虛擬化。 這表示，每個應用程式、其執行階段、相依性及系統程式庫在容器內執行時，在其各自於作業系統建構中的獨立範圍內，都具有容器的完整專屬存取權。 再加上可攜性，這種程度的安全性和資源隔離是容器與 Service Fabric 一起使用的主要優點 (否則是在處理序中執行服務)。

容器是從應用程式中將基礎作業系統虛擬化的一種虛擬化技術。 容器提供不變的環境讓應用程式以不同的隔離程度執行。 容器直接執行於核心之上，而且在檔案系統和其他資源中有其各自隔離的範圍。 相較於虛擬機器，容器的優點如下︰

* **小型**︰容器使用單一儲存空間，每一層的差異較小，以增加效率。
* **快速啟動**︰容器不需要啟動作業系統，因此，比虛擬機器更快啟動和運作，通常只在幾秒內。
* **可攜性**︰容器化應用程式映像可以移植到雲端或內部部署中，而在虛擬機器內或直接在實體機器上執行。
* **資源管理**：容器可以限制在其主機上可取用的實體資源。

## <a name="container-types"></a>容器類型
Service Fabric 支援下列類型的容器。

### <a name="docker-containers-on-linux"></a>Linux 上的 Docker 容器
Docker 提供高階 API 來建立和管理 Linux 核心容器頂端的容器。 Docker 中樞是儲存和擷取容器映像的中央儲存機制。

如需如何執行此操作的逐步解說，請閱讀[將 Docker 容器部署至 Service Fabric](service-fabric-deploy-container-linux.md)。

### <a name="windows-server-containers"></a>Windows Server 容器
Windows Server 2016 提供兩種不同的容器，所提供的隔離程度有所不同。 Windows Server 容器類似於 Docker 容器，因為都有命名空間和檔案系統隔離，但與它們執行所在的主機共用核心。 在 Linux 上，傳統上由控制群組和命名空間提供這種隔離，而 Windows Server 容器具有類似的行為。

Windows Hyper-V 容器提供更高程度的隔離和安全性，因為每個容器彼此之間或與主機之間並不共用作業系統核心。 由於有如此高程度的安全性隔離，Hyper-V 容器在惡意多租用戶的情況下是重點部分。

如需如何執行此操作的逐步解說，請閱讀[將 Windows 容器部署至 Service Fabric](service-fabric-deploy-container.md)。

下圖顯示作業系統中可用的各種不同的虛擬化及隔離等級。
![Service Fabric 平台][Image1]

## <a name="scenarios-for-using-containers"></a>容器使用案例
以下是典型範例，容器是很好的選擇︰

* **IIS 提起然後平移**︰如果您有想要繼續使用的現有 [ASP.NET MVC](https://www.asp.net/mvc) 應用程式，將它們放在一個容器，而不是移轉到 ASP.NET 核心。 這些 ASP.NET MVC 應用程式相依於網際網路資訊服務 (IIS)。 您可以從預先建立的 IIS 映像將這些應用程式封裝成容器映像，然後與 Service Fabric 一起部署。 請參閱 [Windows Server 上的容器映像](https://msdn.microsoft.com/virtualization/windowscontainers/quick_start/quick_start_images)，以取得如何建立 IIS 映像的詳細資訊。
* **混合容器和 Service Fabric 微服務**︰針對您的應用程式的一部分使用現有的容器映像。 例如，對於應用程式的 Web 前端系統，您可以使用 [NGINX 容器](https://hub.docker.com/_/nginx/)，而對於更密集的後端運算，則可以使用具狀態服務。
* **減少「壟斷」服務的影響**︰您可以使用容器的資源控管能力來限制服務在主機上使用的資源。 如果服務可能會耗用大量資源，因而影響其他服務的效能 (例如，像作業一樣長時間執行的查詢)，請考慮將這些服務放到可控管資源的容器中。

## <a name="service-fabric-support-for-containers"></a>Service Fabric 的容器支援
目前，Service Fabric 支援將 Docker 容器部署在 Linux，也支援將 Windows Server 容器部署在 Windows Server 2016。 未來版本將增加支援 Hyper-V 容器。

在 Service Fabric [應用程式模型](service-fabric-application-model.md)中，容器代表多個服務複本所在的應用程式主機。 支援下列容器案例︰

* **來賓容器**︰此案例和[來賓可執行檔案例](service-fabric-deploy-existing-app.md)相同，可讓您部署容器中的現有應用程式。 範例包括 Node.js、JavaScript 或任何程式碼 (可執行檔)。
* **容器內的無狀態服務**︰這些是使用 Reliable Services 和 Reliable Actors 程式設計模型的無狀態服務。 目前只有 Linux 上支援此案例。 未來版本中將增加支援 Windows 容器中的無狀態服務。
* **容器內的具狀態服務**︰這些是 Linux 上使用 Reliable Actors 程式設計模型的具狀態服務。 Linux 上目前不支援 Reliable Services。  未來版本中將增加支援 Windows 容器中的具狀態服務。

Service Fabric 有數個容器功能可協助您建置由容器化微服務組成的應用程式。 這些稱為容器化服務。 功能包括：

* 容器映像部署和啟用。
* 資源管理。
* 儲存機制驗證。
* 容器連接埠至主機連接埠的對應。
* 容器至容器的探索及通訊。
* 能夠設定環境變數。

## <a name="next-steps"></a>後續步驟
在本文中，您已了解容器、Service Fabric 是容器協調者，以及 Service Fabric 具有支援容器的功能。 接下來，我們將示範每一項功能來展示其用法。

[將 Windows 容器部署至 Windows Server 2016 上的 Service Fabric](service-fabric-deploy-container.md)

[將 Docker 容器部署至 Linux 上的 Service Fabric](service-fabric-deploy-container-linux.md)

[Image1]: media/service-fabric-containers/Service-Fabric-Types-of-Isolation.png



<!--HONumber=Feb17_HO2-->


