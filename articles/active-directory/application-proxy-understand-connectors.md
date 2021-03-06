---
title: "了解 Azure AD 應用程式 Proxy 連接器 | Microsoft Docs"
description: "涵蓋 Azure AD 應用程式 Proxy 連接器的基本概念。"
services: active-directory
documentationcenter: 
author: kgremban
manager: femila
ms.assetid: 
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/22/2017
ms.author: kgremban
translationtype: Human Translation
ms.sourcegitcommit: cea53acc33347b9e6178645f225770936788f807
ms.openlocfilehash: 41d6f678dba769cf7f949751da8cacf3df7f88c1
ms.lasthandoff: 03/03/2017


---

# <a name="understand-azure-ad-application-proxy-connectors"></a>了解 Azure AD 應用程式 Proxy 連接器

本文討論連接器，也就是 Azure AD 應用程式 Proxy 的神秘推手。 它們很簡單、容易部署及維護且超級強大。

> [!NOTE]
> 應用程式 Proxy 是您升級至 Premium 或 Basic 版本的 Azure Active Directory 時才能使用的功能。 如需詳細資訊，請參閱 [Azure Active Directory 版本](active-directory-editions.md)。

## <a name="what-are-azure-ad-application-proxy-connectors"></a>何謂 Azure AD 應用程式 Proxy 連接器？
須在您的網路上安裝稱為連接器的 Windows Server 服務之後，應用程式 Proxy 才會運作。 您可以根據您的高可用性和延展性需求安裝連接器。 以其中一種開始，並視需要新增更多。 每次安裝連接器時，它會新增至做為您租用戶的連接器集區。

我們建議您不要在應用程式伺服器本身上安裝連接器；雖然這是可行的，尤其是針對小型部署。

不需要刪除您不再使用的連接器。 當連接器執行時，它在連接到服務時會保持作用中。 未使用的連接器會標記為_非作用中_，且將在未作用 10 天之後移除。 

有關解決 Azure AD 連線問題的詳細資訊，請參閱[如何疑難排解 Azure AD 應用程式 Proxy 連線問題](https://blogs.technet.microsoft.com/applicationproxyblog/2015/03/24/how-to-troubleshoot-azure-ad-application-proxy-connectivity-problems)。 

## <a name="what-are-the-cloud-rules-for-connectors"></a>什麼是連接器的雲端規則？
連接器和服務會負責所有高可用性的工作。 它們可以動態新增或移除。 每當新要求抵達時，它將會路由至其中一個目前可用的連接器。 如果連接器暫時無法使用，它會因此不回應此流量。

除了服務設定和驗證此連接器之憑證的連線之外，連接器是無狀態且在電腦上沒有組態資料。 當它們連線至服務時，會提取所有必要的組態資料，且每隔幾分鐘會重新整理。
它們也會輪詢伺服器以尋找是否有較新版的連接器。 如果找到，連接器會自行更新。

您可以使用事件記錄檔和效能計數器，或從雲端使用 [連接器狀態] 頁面，從它們所執行的電腦上監視連接器，如下所示。

 ![Azure AD 應用程式 Proxy 連接器](./media/application-proxy-understand-connectors/app-proxy-connectors.png)

## <a name="all-networking-is-outbound"></a>所有的網路為輸出
連接器只會傳送輸出要求，因此連接一律由連接器啟動。 不需要開啟輸入連接埠，因為一旦建立工作階段後，流量會以兩種方式流動。

輸出流量會傳送到應用程式 Proxy 服務和已發佈應用程式。 服務的流量會傳送至 Azure 資料中心的多個不同通訊埠編號。 如需詳細資訊，請參閱[在 Azure 入口網站中啟用應用程式 Proxy](active-directory-application-proxy-enable.md)。

由於僅有輸出流量，不需要設定連接器之間的負載平衡，或透過您的防火牆設定內部存取。

如需設定輸出防火牆規則的相關資訊，請參閱[使用現有的內部部署 Proxy 伺服器](application-proxy-working-with-proxy-servers.md)。

請使用 [Azure AD 應用程式 Proxy 連接器連接埠測試工具](https://aadap-portcheck.connectorporttest.msappproxy.net/)，來確認您的連接器是否能夠連線到「應用程式 Proxy」服務。 至少，請確定「美國中部」區域及離您最近的區域都具有綠色勾選記號。 除此之外，綠色勾選記號越多代表恢復能力越佳。 

## <a name="network-security"></a>網路安全性

連接器可以安裝在允許它們將要求傳送至服務和後端應用程式的網路上任何地方。 如果您將它們安裝在公司網路內部、DMZ (非軍事區域) 內，或甚至在可存取應用程式之雲端中執行的虛擬機器上，它們將會正常運作。

DMZ 部署通常較為複雜。 但是，在 DMZ 中部署連接器的原因之一，是使用可供在該處執行之元件使用的其他基礎結構，例如後端應用程式負載平衡器和/或安全性控制 (例如入侵偵測系統)。

## <a name="domain-joining"></a>加入網域

連接器可以在未加入網域的電腦上執行。 不過，如果您選擇使用運用整合式 Windows 驗證 (IWA) 的應用程式使用單一登入 (SSO)，則需要已加入網域的電腦。 

在此情況下，連接器電腦必須代表已發佈應用程式的相關使用者加入可以執行 [Kerberos](https://web.mit.edu/kerberos) 限制委派的網域。

連接器也可以加入網域，或具有部分信任的樹系或唯讀網域控制器 (RODC)。

## <a name="connectors-on-hardened-environments"></a>強化環境上的連接器

在大部分情況下，連接器部署非常直截了當，且不需要特殊組態。 但是，有數個特別的條件應該列入考量︰

* 限制傳出流量的組織必須遵循此處的指示來開啟必要的連接埠。
* 可能需要 FIPS 相容的電腦來變更其設定，以允許連接器服務、連接器更新程式服務及其安裝程式來產生及儲存該電腦上的憑證。
* 根據發出網路要求的處理程序鎖定其環境的組織，必須確定已啟用兩個連接器服務才可存取所有必要的連接埠和 IP。
* 在某些情況下，輸出的正向 Proxy 可能會中斷雙向驗證憑證，並造成失敗的通訊。

## <a name="all-connectors-are-created-almost-equal"></a>所有連接器幾乎為相同方式建立

所有連接器會假設彼此完全相同，且每個傳入要求會抵達每一個連接器。 這表示它們全部都應該有相同的網路連線和 [Kerberos](https://web.mit.edu/kerberos) 設定。

所有的連接器對服務通訊都會受到所發行用戶端憑證的保護，然後安裝在連接器電腦上。 如需更新連接器憑證的資訊，請參閱[在 Azure 入口網站中啟用應用程式 Proxy](active-directory-application-proxy-enable.md)。

## <a name="connector-authentication"></a>連接器驗證

為了提供安全服務，連接器必須向服務驗證，且服務必須向連接器驗證。 可在連接器初始化連線時，使用用戶端和伺服器憑證來完成此動作。 如此一來，系統管理員的使用者名稱和密碼不會儲存在連接器電腦上。

使用的憑證是特定於應用程式 Proxy 服務。 它們會在初始註冊期間建立，且每隔幾個月會由連接器自動更新。 

如果連接器在數個月內無法連接到服務，則它可能具有已過期的憑證。 在此情況下就必須註冊，因此您必須解除安裝並重新安裝連接器以觸發程序註冊。 您可以執行下列 PowerShell 命令：

```
* Import-module AppProxyPSModule
* Register-AppProxyConnector
```

## <a name="performance-and-scalability"></a>效能和延展性

雖然線上服務的級別是透明的，但當涉及連接器時，級別便是因素。 您需要有足夠的連接器，以處理尖峰流量。 因為連接器是無狀態，它們並非依使用者或工作階段的數目而定。 相反地，它們是依要求的數目和其承載大小而定。 針對標準 Web 流量，我們看到電腦平均每秒處理幾千個要求。 這是依確切的電腦特性而定。

連接器效能受限於 CPU 和網路。 CPU 效能需要 SSL 加密和解密，而網路對於取得應用程式和 Azure 線上服務的快速連線很重要。 相反地，連接器發行需要較少的記憶體。

針對連接器服務，我們盡可能卸載連接器。 線上服務可以處理大部分的處理程序，及所有未經驗證的流量。 所有可在雲端中完成的內容都是在雲端中完成。

關於效能的另一個因素是連接器之間的網路品質，包括︰

* _線上服務︰_連接是否緩慢或具有高延遲。 它會影響服務等級。 您的組織最好是透過 Express Route 連線到 Azure。 否則，請確定網路小組可確保與 Azure 的連線是以有效的方式處理。

* _後端應用程式︰_在某些情況下，連接器和後端應用程式之間有其他 Proxy。 疑難排解這個問題很容易，方法為從連接器電腦開啟瀏覽器，並存取這些應用程式。 如果您在 Azure 中執行連接器，且應用程式在內部部署，則體驗可能無法如您的使用者所預期。
* _網域控制器︰_如果連接器使用 Kerberos 限制委派 (KCD) 來執行 SSO，它們會先連絡網域控制器後，才傳送要求至後端。 連接器有 Kerberos 票證的快取 (但是在忙碌環境中)，網域控制器的回應速度可能會減緩體驗。 當網域控制器為內部部署時，這在 Azure 中執行的連接器很常見。

## <a name="automatic-updates-to-the-connector"></a>自動更新至連接器

利用連接器更新程式服務，我們提供自動化的方式來維持最新狀態。 如此一來，您可持續擁有所有新功能的優點，以及安全性和效能增強功能。

Azure AD 支援您部署之所有連接器的自動更新。 只要應用程式 Proxy 連接器更新程式服務正在執行，您的連接器便會自動更新，無停機時間且不需要手動步驟。 如果您在伺服器上沒有看到連接器更新程式服務，則需要重新安裝您的連接器以取得更新。 如果您想要了解安裝連接器的詳細資訊，請參閱[設定文件](https://azure.microsoft.com/en-us/documentation/articles/active-directory-application-proxy-enable.md)。

### <a name="updater-impact"></a>更新程式的影響

_擁有一個連接器的租用戶︰_如果您只有一個連接器，則該連接器將更新為最新群組的一部分。 因為沒有任何其他連接器可重新路由流量通過，所以服務在更新期間無法使用。 若要避免這種停機情形並更輕鬆地確保高可用性，建議您安裝第二個連接器，然後建立連接器群組。 如需有關如何執行這項操作的詳細資訊，請參閱[有關連接器群組的文件](https://azure.microsoft.com/en-us/documentation/articlesactive-directory-application-proxy-connectors.md)。

_其他租用戶︰_在連接器更新期間，流量會重新路由至其他連接器以將中斷時間降至最低。 不過，更新啟動時，正在進行中的任何交易可能會中斷。 您的瀏覽器應會自動重試作業，使您可以看到此縮減。 否則，您可能必須重新整理頁面來解決這個問題。

我們知道即使是一分鐘的停機時間都很痛苦，但這些更新能讓我們為您提供更佳的連接器與許多增強功能，我們認為是值得的。

如果您想要關於我們最新連接器更新的變更詳細資訊，請參閱[最新更新](https://azure.microsoft.com/en-us/updates/app-proxy-connector-12sept2016)。 我們會修改該頁面，以及每個更新。

## <a name="under-the-hood"></a>幕後

我們在 Azure 中為您提供許多有用的工具。 特別是連接器，有許多有用的功能。 因為連接器是根據 Windows Server Web 應用程式 Proxy，因此具有大部分相同的管理工具；包括一組豐富的 Windows 事件記錄檔和 Windows 效能計數器，如下事件檢視器中所示︰

 ![AzureAD 事件檢視器](./media/application-proxy-understand-connectors/event-view-window.png)

和效能監視器：

 ![AzureAD 效能監視器](./media/application-proxy-understand-connectors/performance-monitor.png)

連接器有系統管理員和工作階段記錄檔。 系統管理記錄檔包含索引鍵事件和其錯誤。 這些工作階段記錄檔包含所有交易，以及它們的處理詳細資料。 

若要查看它們，您必須啟用事件檢視器 [檢視] 功能表上的 [顯示分析與偵錯記錄檔]。 接著，您必須啟用它們以開始收集事件。 這些記錄不會出現在 Windows Server 2012 R2 中的 Web 應用程式 Proxy，因為連接器會根據較新的版本。

 ![AzureAD 事件檢視器工作階段](./media/application-proxy-understand-connectors/event-view-window-session.png)

您可以檢查 [服務] 視窗中的服務狀態。 連接器包含兩個 Windows 服務；其中一個是實際的連接器，另一個負責更新。 必須具有這兩個才能隨時執行。

 ![AzureAD 服務本機](./media/application-proxy-understand-connectors/aad-connector-services.png)

如需有關解決應用程式 Proxy 連接器錯誤的詳細資訊，請參閱[疑難排解應用程式 Proxy](https://azure.microsoft.com/en-us/documentation/articles/active-directory-application-proxy-troubleshoot)。

## <a name="next-steps"></a>後續步驟
[使用現有的內部部署 Proxy 伺服器](application-proxy-working-with-proxy-servers.md)

[如何以無訊息方式安裝 Azure AD 應用程式 Proxy 連接器](active-directory-application-proxy-silent-installation.md)


