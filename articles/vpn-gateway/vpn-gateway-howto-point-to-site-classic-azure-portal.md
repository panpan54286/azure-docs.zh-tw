---
title: "使用點對站將電腦連接至 Azure 虛擬網路︰Azure 入口網站：傳統 | Microsoft Docs"
description: "使用 Azure 入口網站建立點對站 VPN 閘道連接，以安全地連接至您的傳統 Azure 虛擬網路。"
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: timlt
editor: 
tags: azure-service-management
ms.assetid: 65e14579-86cf-4d29-a6ac-547ccbd743bd
ms.service: vpn-gateway
ms.devlang: na
ms.topic: hero-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/20/2017
ms.author: cherylmc
translationtype: Human Translation
ms.sourcegitcommit: eeb56316b337c90cc83455be11917674eba898a3
ms.openlocfilehash: bc542cfbca3a27aec98b75e2b7ed05472419c3a7
ms.lasthandoff: 04/03/2017


---
# <a name="configure-a-point-to-site-connection-to-a-vnet-using-the-azure-portal-classic"></a>使用 Azure 入口網站設定 VNet 的點對站連線 (傳統)
> [!div class="op_single_selector"]
> * [Resource Manager - Azure 入口網站](vpn-gateway-howto-point-to-site-resource-manager-portal.md)
> * [Resource Manager - PowerShell](vpn-gateway-howto-point-to-site-rm-ps.md)
> * [傳統 - Azure 入口網站](vpn-gateway-howto-point-to-site-classic-azure-portal.md)
>
>

點對站 (P2S) 設定可讓您建立從個別的用戶端電腦到虛擬網路的安全連線。 P2S 是透過 SSTP (安全通訊端通道通訊協定) 的 VPN 連線。 當您想要從遠端位置 (例如從住家或會議) 連線至 VNet 時，或只有幾個需要連線至虛擬網路的用戶端時，點對站連線是很實用的解決方案。 P2S 連線不需要 VPN 裝置或公眾對應 IP 位址即可運作。 您可從用戶端電腦建立 VPN 連線。

本文逐步引導您使用 Azure 入口網站，在傳統部署模型中建立具有點對站連線的 VNet。 如需有關點對站連線的詳細資訊，請參閱本文結尾的[點對站常見問題集](#faq)。


### <a name="deployment-models-and-methods-for-p2s-connections"></a>P2S 連線的部署模型和方法
[!INCLUDE [deployment models](../../includes/vpn-gateway-deployment-models-include.md)]

下表顯示 P2S 組態的兩種部署模型和可用的部署方法。 當包含設定的文章推出時，我們會直接從此資料表連結至該文章。

[!INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-table-point-to-site-include.md)]

## <a name="basic-workflow"></a>基本工作流程
![Point-to-Site-diagram](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/point-to-site-connection-diagram.png)

以下小節將逐步引導您建立與虛擬網路的安全點對站連線。

1. 建立虛擬網路和 VPN 閘道
2. 產生憑證
3. 上傳 .cer 檔案
4. 產生 VPN 用戶端組態封裝
5. 設定用戶端電腦
6. 連接到 Azure

### <a name="example-settings"></a>範例設定
您可以使用下列範例設定：

* **名稱：VNet1**
* **位址空間：192.168.0.0/16**<br>在此範例中，我們只使用一個位址空間。 您可以針對 VNet 使用一個以上的位址空間。
* **子網路名稱：FrontEnd**
* **子網路位址範圍：192.168.1.0/24**
* **訂用帳戶：**如果您有一個以上的訂用帳戶，請確認您使用正確的訂用帳戶。
* **資源群組：TestRG**
* **位置：美國東部**
* **連線類型：點對站**
* **用戶端位址空間：172.16.201.0/24**。 使用這個點對站連線來連線到 VNet 的 VPN 用戶端，會收到來自指定集區的 IP 位址。
* **GatewaySubnet: 192.168.200.0/24**。 閘道子網路必須命名為 'GatewaySubnet'。
* **大小**：選取您想要使用的閘道 SKU。
* **路由類型：動態**



## <a name="vnetvpn"></a>區段 1 - 建立虛擬網路和 VPN 閘道

在開始之前，請確認您有 Azure 訂用帳戶。 如果您還沒有 Azure 訂用帳戶，則可以啟用 [MSDN 訂戶權益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details)或註冊[免費帳戶](https://azure.microsoft.com/pricing/free-trial)。
### <a name="createvnet"></a>第 1 部分：建立虛擬網路
如果您還沒有虛擬網路，請建立一個。 已提供螢幕擷取畫面做為範例。 請務必將值取代為您自己的值。 若要使用 Azure 入口網站建立 VNet，請使用下列步驟：

1. 透過瀏覽器瀏覽至 [Azure 入口網站](http://portal.azure.com) ，並視需要使用您的 Azure 帳戶登入。
2. 按一下 [新增] 。 在 [搜尋 Marketplace] 欄位中，輸入「虛擬網路」。 在傳回的清單中找到 [虛擬網路]，並按一下以開啟 [虛擬網路] 刀鋒視窗。

    ![搜尋虛擬網路刀鋒視窗](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/newvnetportal700.png)
3. 從接近 [虛擬網路] 刀鋒視窗底部的 [選取部署模型] 清單中，選取 [傳統]，然後按一下 [建立]。

    ![選取部署模型](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/selectmodel.png)
4. 在 [建立虛擬網路]  刀鋒視窗中進行 VNet 設定。 在此刀鋒視窗中，您將新增您的第一個位址空間和單一子網路位址範圍。 完成 VNet 建立之後，您可以返回並新增其他子網路和位址空間。

    ![建立虛擬網路的刀鋒視窗](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/vnet125.png)
5. 確認 [訂用帳戶]  正確無誤。 您可以使用下拉式清單變更訂用帳戶。
6. 按一下 [資源群組]  並選取現有的資源群組，或輸入新的資源群組名稱以建立新的資源群組。 如果您要建立新的群組，請根據您計劃的組態值來命名資源群組。 如需資源群組的詳細資訊，請瀏覽 [Azure Resource Manager 概觀](../azure-resource-manager/resource-group-overview.md#resource-groups)。
7. 接著，選取 VNet 的 [位置]  設定。 此位置會決定您部署到此 VNet 的資源所在位置。
8. 如果想要能夠在儀表板上輕鬆地尋找您的 VNet，請選取 [釘選到儀表板]，然後按一下 [建立]。

    ![釘選到儀表板](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/pintodashboard150.png)
9. 按一下 [建立] 之後，您會看到儀表板上有一個圖格會反映 VNet 的進度。 建立 VNet 時，此圖格會變更。

    ![建立虛擬網路磚](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/deploying150.png)
10. 建立虛擬網路之後，您可以新增 DNS 伺服器的 IP 位址，以便處理名稱解析。 開啟您的虛擬網路設定，按一下 DNS 伺服器，並新增您想要使用的 DNS 伺服器 IP 位址。 此設定不會建立新的 DNS 伺服器。 務必新增您的資源可以與其通訊的 DNS 伺服器。

一旦建立虛擬網路之後，您將在 Azure 傳統入口網站的網路頁面上看見 [狀態] 下列出 [已建立]。

### <a name="gateway"></a>第 2 部分：建立閘道子網路和動態路由閘道
在此步驟中，您將建立一個閘道子網路和一個動態路由閘道。 在傳統部署模型的 Azure 入口網站中，建立閘道子網路和閘道可以透過相同的組態刀鋒視窗完成。

1. 在入口網站中，瀏覽至要建立閘道的虛擬網路。
2. 在虛擬網路的刀鋒視窗的 [概觀] 刀鋒視窗fe 的 [VPN 連線] 區段中，按一下 [閘道]。

    ![按一下以建立閘道](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/beforegw125.png)
3. 在 [新增 VPN 連線] 刀鋒視窗上，選取 [點對站]。

    ![點對站連線類型](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/newvpnconnect.png)
4. 針對 [用戶端位址空間]，新增 IP 位址範圍。 這是 VPN 用戶端在連接時將從其接收 IP 位址的範圍。 刪除自動填入的範圍，並新增您自己的範圍。

    ![用戶端位址空間](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/clientaddress.png)
5. 選取 [立即建立閘道] 核取方塊。

    ![立即建立閘道](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/creategwimm.png)
6. 按一下 [選擇性閘道組態] 可開啟 [閘道組態] 刀鋒視窗。

    ![按一下選擇性閘道組態](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/optsubnet125.png)
7. 按一下 [子網路設定所需設定] 可新增**閘道子網路**。 雖然您可以建立小至 /29 的閘道子網路，我們建議您選取至少 /28 或 /27，建立包含更多位址的較大子網路。 這將允許足夠的位址，以容納您未來可能需要的其他組態。

   > [!IMPORTANT]
   > 使用閘道子網路時，避免將網路安全性群組 (NSG) 與閘道子網路產生關聯。 將網路安全性群組與此子網路產生關聯，可能會導致您的 VPN 閘道如預期般停止運作。 如需有關網路安全性群組的詳細資訊，請參閱[什麼是網路安全性群組？](../virtual-network/virtual-networks-nsg.md)
   >
   >

    ![新增 GatewaySubnet](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/gwsubnet125.png)
8. 選取閘道**大小**。 這是您將用來建立虛擬網路閘道的閘道 SKU。 在入口網站中，預設的 SKU 是**基本**。 如需關於閘道 SKU 的資訊，請參閱[關於 VPN 閘道設定](vpn-gateway-about-vpn-gateway-settings.md#gwsku)。

    ![閘道大小](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/gwsize125.png)
9. 選取閘道的 [路由類型]。 P2S 組態需要**動態**路由類型。 完成此刀鋒視窗的設定時，按一下 [確定]。

    ![設定路由類型](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/routingtype125.png)
10. 在 [新增 VPN 連線] 刀鋒視窗中，按一下刀鋒視窗底部的 [確定]，來開始建立虛擬網路閘道。 這項作業可能需要 45 分鐘的時間才能完成。

## <a name="generatecerts"></a>第 2 節 - 建立憑證
憑證是 Azure 用於點對站 VPN 的 VPN 用戶端驗證。 建立根憑證之後，您要將公開憑證資料 (不是私密金鑰)，匯出為 Base-64 編碼的 X.509.cer 檔案。 接著將來自根憑證的公開憑證資料上傳至 Azure。

每個使用點對站連線至 VNet 的用戶端電腦都必須安裝用戶端憑證。 用戶端憑證是從根憑證產生，並安裝在每部用戶端電腦上。 如果未安裝有效的用戶端憑證，且用戶端嘗試連線至 VNet，驗證將會失敗。

### <a name="cer"></a>第 1 部分︰取得根憑證的公開金鑰 (.cer)

####<a name="enterprise-certificate"></a>企業憑證
 
如果您是使用企業解決方案，則可以使用現有的憑證鏈結。 取得您想要使用的根憑證 .cer 檔案。

####<a name="self-signed-root-certificate"></a>自我簽署根憑證

如果您未使用企業憑證解決方案，則必須建立自我簽署的根憑證。 若要建立包含 P2S 驗證的必要欄位之自我簽署憑證，您可以使用 PowerShell。 [使用 PowerShell 建立點對站連線的自我簽署憑證](vpn-gateway-certificates-point-to-site.md)將逐步引導您完成建立自我簽署根憑證的步驟。

> [!NOTE]
> 先前，makecert 是建立自我簽署根憑證，並產生用於點對站連線之用戶端憑證的建議方法。 您現在可以使用 PowerShell 來建立這些憑證。 使用 PowerShell 的一個優點是能夠建立 SHA-2 憑證。 請參閱[使用 PowerShell 建立點對站連線的自我簽署憑證](vpn-gateway-certificates-point-to-site.md)的必要值。
>
>

#### <a name="to-export-the-public-key-for-a-self-signed-root-certificate"></a>若要匯出自我簽署根憑證的公開金鑰

點對站連線需要公開金鑰 (.cer) 上傳至 Azure。 下列步驟可協助您匯出自我簽署根憑證的 .cer 檔案。

1. 若要取得憑證的 .cer 檔案，請開啟 [certmgr.msc] 。 找出自我簽署的根憑證，通常位於 '[憑證 - 目前的使用者]\[個人]\[憑證]' 中，然後按一下滑鼠右鍵。 按一下 [所有工作]，然後按一下 [匯出]。 這會開啟 [憑證匯出精靈] 。
2. 在精靈中，按 [下一步]。 選取 [否，不要匯出私密金鑰]，然後按 [下一步]。
3. 在 [匯出檔案格式] 頁面上，選取 [Base-64 編碼 X.509 (.CER)]，然後按 [下一步]。 
4. 在 [要匯出的檔案] 中，[瀏覽] 到您要匯出憑證的位置。 針對 [檔案名稱] ，請為憑證檔案命名。 然後按 [下一步] 。
5. 按一下 [完成]  以匯出憑證。 您將會看到 [匯出成功]。 按一下 [確定] 以關閉精靈。

### <a name="genclientcert"></a>第 2 部分：產生用戶端憑證

您可以為每個會進行連線的用戶端產生唯一的憑證，您也可以在多個用戶端上使用相同的憑證。 產生唯一的用戶端憑證的優點是能夠視需要撤銷單一憑證。 否則，如果每個人都使用相同的用戶端憑證，而您發現需要撤銷某一個用戶端的憑證時，則必須為所有使用該憑證進行驗證的用戶端產生並安裝新的憑證。

####<a name="enterprise-certificate"></a>企業憑證
- 如果您使用企業憑證解決方案，請以一般的名稱值格式 'name@yourdomain.com' 產生用戶端憑證，而不要使用 'domain name\username' 格式。
- 請確定您簽發的用戶端憑證所根據的憑證範本，是以「用戶端驗證」(而不是「智慧卡登入」等) 作為使用清單中第一個項目的「使用者」憑證範本。您可以按兩下用戶端憑證，然後檢視 [詳細資料] > [增強金鑰使用方法]，來檢查憑證。

####<a name="self-signed-root-certificate"></a>自我簽署根憑證 
如果您使用自我簽署根憑證，請參閱[使用 PowerShell 建立點對站連線的自我簽署憑證](vpn-gateway-certificates-point-to-site.md#clientcert)，以了解產生與點對站連線相容之用戶端憑證的步驟。

### <a name="exportclientcert"></a>第 3 部分：匯出用戶端憑證
如果您使用 [PowerShell](vpn-gateway-certificates-point-to-site.md#clientcert) 指示從自我簽署根憑證產生用戶端憑證，它會自動安裝在您用來產生它的電腦上。 如果您想要在另一部用戶端電腦上安裝用戶端憑證，您需要將它匯出。

1. 若要匯出用戶端憑證，請開啟 **certmgr.msc**。 以滑鼠右鍵按一下要匯出的用戶端憑證，然後依序按一下 [所有工作] 和 [匯出]。 這會開啟 [憑證匯出精靈] 。
2. 在精靈中，按一下 [下一步]，接著選取 [是，匯出私密金鑰]，然後按 [下一步]。
3. 在 [匯出檔案格式] 頁面上，保留選取預設值。 務必選取 [如果可能的話，包含憑證路徑中的所有憑證]。 然後按 [下一步] 。
4. 在 [安全性]  頁面上，您必須保護私密金鑰。 如果您選取要使用密碼，請務必記錄或牢記您為此憑證設定的密碼。 然後按 [下一步] 。
5. 在 [要匯出的檔案] 中，[瀏覽] 到您要匯出憑證的位置。 針對 [檔案名稱] ，請為憑證檔案命名。 然後按 [下一步] 。
6. 按一下 [完成]  以匯出憑證。

## <a name="upload"></a>第 3 節 - 上傳根憑證 .cer 檔案
建立閘道之後，您可以將信任的根憑證的 .cer 檔案上傳至 Azure。 您最多可上傳 20 個根憑證。 您並未將根憑證的私密金鑰上傳至 Azure。 一旦上傳 .cer 檔案，Azure 會使用它來驗證連接至虛擬網路的用戶端。

1. 在 VNet 刀鋒視窗的 [VPN 連線] 區段中，按一下**用戶端**圖形以開啟 [點對站 VPN 連線] 刀鋒視窗。

    ![用戶端](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/clients125.png)
2. 在 [點對站連線] 刀鋒視窗中，按一下 [管理憑證] 來開啟 [憑證] 刀鋒視窗。<br>

    ![憑證刀鋒視窗](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/ptsmanage.png)<br><br>
3. 在 [憑證] 刀鋒視窗中，按一下 [上傳] 來開啟 [上傳憑證] 刀鋒視窗。<br>

    ![上傳憑證刀鋒視窗](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/uploadcerts.png)<br>
4. 按一下資料夾圖片來瀏覽 .cer 檔案。 選取檔案，然後按一下 [確定]。 重新整理頁面，以在 [憑證] 刀鋒視窗上查看所上傳的憑證。

    ![Upload certificate](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/upload.png)<br>

## <a name="vpnclientconfig"></a>第 4 節 - 產生 VPN 用戶端組態封裝
若要連接到虛擬網路，您還需要設定 VPN 用戶端。 用戶端電腦需要具備用戶端憑證及適當的 VPN 用戶端組態封裝，才能順利連接。

VPN 用戶端套件包含的組態資訊可設定 Windows 內建的 VPN 用戶端軟體。 此套件不會安裝其他軟體。 這些是您所要連接之虛擬網路的專屬設定。 如需支援的用戶端作業系統清單，請參閱本文結尾的[點對站連線常見問題集](#faq)。

### <a name="to-generate-the-vpn-client-configuration-package"></a>產生 VPN 用戶端組態封裝
1. 在 Azure 入口網站中，於 VNet 的 [概觀] 刀鋒視窗的 [VPN 連線] 中，按一下用戶端圖形以開啟 [點對站 VPN 連線] 刀鋒視窗。
2. 在 [點對站 VPN 連線] 刀鋒視窗上方，按一下對應至即將安裝之目標用戶端作業系統的下載套件：

   * 若為 64 位元用戶端，請選取 [VPN 用戶端 (64 位元)]。
   * 若為 32 位元用戶端，請選取 [VPN 用戶端 (32 位元)]。

     ![下載 VPN 用戶端組態封裝](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/dlclient.png)<br>
3. 您會看到一則訊息，說明 Azure 正在產生虛擬網路的 VPN 用戶端組態封裝。 在幾分鐘之後，封裝會產生，而您會在本機電腦看到一則訊息，說明已下載封裝。 儲存組態封裝檔案。 您會在將使用 P2S 連線至虛擬網路的每個用戶端電腦上安裝它。

## <a name="clientconfiguration"></a>第 5 節 - 設定用戶端電腦
### <a name="part-1-install-an-exported-client-certificate"></a>第 1 部分 - 安裝匯出的用戶端憑證

如果您想要從不同於用來產生用戶端憑證的用戶端電腦建立 P2S 連線，您需要安裝用戶端憑證。 安裝用戶端憑證時，您需要匯出用戶端憑證時所建立的密碼。

1. 找出 *.pfx* 檔案並複製到用戶端電腦。 在用戶端電腦上，按兩下 *.pfx* 檔案以安裝。 [存放區位置] 保留 [目前使用者]，然後按 [下一步]。
2. 在 [要匯入的檔案]  頁面上，請勿進行任何變更。 按 [下一步] 。
3. 在 [私密金鑰保護] 頁面上，如果您已使用密碼，請輸入憑證的密碼，或確認安裝憑證的安全性主體是否正確，然後按 [下一步]。
4. 在 [憑證存放區] 頁面上，保留預設的位置，然後按 [下一步]。
5. 按一下 [完成] 。 在憑證安裝的 [安全性警告] 上，按一下 [是]。 您可以安心地按一下 [是]，因為您已經產生憑證。 現在已成功匯入憑證。

### <a name="part-2-install-the-vpn-client-configuration-package"></a>第 2 部分：安裝 VPN 用戶端組態套件
您可以在每個用戶端電腦上使用相同的 VPN 用戶端組態封裝，前提是版本符合用戶端的架構。

1. 在本機將組態檔複製到您要連線至虛擬網路的電腦上。 
2. 在用戶端電腦上按兩下 .exe 檔案以安裝套件。 組態套件尚未簽署，因為是由您建立。 這表示您可能會看到警告。 如果您看到 Windows SmartScreen 快顯視窗，請按一下 [更多資訊] (左側)，然後按一下 [仍要執行] 來安裝套件。
3. 在用戶端電腦上，瀏覽至 [網路設定]，然後按一下 [VPN]。 您會看到列出的連線。 它會顯示將要連接的虛擬網路名稱，如下所示：

    ![VPN 用戶端](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/vpn.png)

## <a name="connect"></a>第 6 節 - 連接到 Azure
### <a name="connect-to-your-vnet"></a>連接到您的 VNet
1. 若要連接至您的 VNet，在用戶端電腦上瀏覽到 VPN 連線，然後找出所建立的 VPN 連線。 其名稱會與虛擬網路相同。 按一下 [ **連接**]。 可能會出現與使用憑證有關的快顯訊息。 如果出現，按一下 [繼續]  以使用較高的權限。
2. 在 [連線] 狀態頁面上，按一下 [連線] 以便開始連線。 如果出現 [選取憑證]  畫面，請確認顯示的用戶端憑證是要用來連接的憑證。 如果沒有，請使用下拉箭頭來選取正確的憑證，然後按一下 [確定] 。

    ![VPN 用戶端連線](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/clientconnect.png)
3. 現在應該已建立您的連接。

    ![建立的連線](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/connected.png)

> [!NOTE]
> 如果您使用的是藉由企業 CA 解決方案簽發的憑證，而在驗證時發生問題，請檢查用戶端憑證上的驗證順序。 您可以按兩下用戶端憑證，然後移至 [詳細資料] > [增強金鑰使用方法]，來檢查驗證清單順序。 請確定清單所顯示的第一個項目是「用戶端驗證」。 如果不是，您就必須根據以「用戶端驗證」作為清單中第一個項目的「使用者」範本來簽發用戶端憑證。 
>
>

### <a name="verify-the-vpn-connection"></a>驗證 VPN 連線
1. 若要驗證您的 VPN 連線為作用中狀態，請開啟提升權限的命令提示字元，並執行 *ipconfig/all*。
2. 檢視結果。 請注意，您接收到的 IP 位址是建立 VNet 時所指定之點對站連線位址範圍內的其中一個位址。 結果應該類似下面的內容：

範例：

    PPP adapter VNet1:
        Connection-specific DNS Suffix .:
        Description.....................: VNet1
        Physical Address................:
        DHCP Enabled....................: No
        Autoconfiguration Enabled.......: Yes
        IPv4 Address....................: 192.168.130.2(Preferred)
        Subnet Mask.....................: 255.255.255.255
        Default Gateway.................:
        NetBIOS over Tcpip..............: Enabled

## <a name="add"></a>新增或移除受信任的根憑證

您可以從 Azure 新增和移除受信任的根憑證。 當您移除受信任的憑證時，從根憑證所產生的用戶端憑證將不再能夠透過點對站連線到 Azure。 若您希望用可以戶端連線，他們需要安裝在 Azure 中受信任的憑證所產生的新用戶端憑證。

### <a name="to-add-a-trusted-root-certificate"></a>若要新增受信任的根憑證

您最多可新增 20 個受信任的根憑證 .cer 檔案至 Azure。 如需指示，請參閱[第 3 節 - 上傳根憑證 .cer 檔案](#upload)。

### <a name="to-remove-a-trusted-root-certificate"></a>移除受信任的根憑證


1. 在 VNet 刀鋒視窗的 [VPN 連線] 區段中，按一下**用戶端**圖形以開啟 [點對站 VPN 連線] 刀鋒視窗。

    ![用戶端](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/clients125.png)
2. 在 [點對站連線] 刀鋒視窗中，按一下 [管理憑證] 來開啟 [憑證] 刀鋒視窗。<br>

    ![憑證刀鋒視窗](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/ptsmanage.png)<br><br>
3. 在 [憑證] 刀鋒視窗中，按一下您想要移除之憑證旁邊的省略符號，然後按一下 [刪除]。

     ![刪除根憑證](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/deleteroot.png)<br>


## <a name="revokeclient"></a>撤銷用戶端憑證
您可以撤銷用戶端憑證。 憑證撤銷清單可讓您選擇性地拒絕以個別的用戶端憑證為基礎的點對站連線。 這不同於移除受信任的根憑證。 若您從 Azure 移除受信任的根憑證 .cer，就會撤銷所有由撤銷的根憑證所產生/簽署的用戶端憑證之存取權。 撤銷用戶端憑證，而不是根憑證，可以讓從根憑證產生的憑證繼續用於點對站連線的驗證。

常見的做法是使用根憑證管理小組或組織層級的存取權，然後使用撤銷的用戶端憑證針對個別使用者進行細部的存取控制。

### <a name="to-revoke-a-client-certificate"></a>若要撤銷用戶端憑證

您可以藉由將指紋新增至撤銷清單來撤銷用戶端憑證。

1. 擷取用戶端憑證指紋。 如需詳細資訊，請參閱[做法：擷取憑證的指紋](https://msdn.microsoft.com/library/ms734695.aspx)。
2. 將資訊複製到文字編輯器，並移除所有的空格，讓它是連續字串。
3. 瀏覽至 [「傳統虛擬網路名稱」> 點對站 VPN 連線 > 憑證] 刀鋒視窗，然後按一下 [撤銷清單] 以開啟 [撤銷清單] 刀鋒視窗。 
4. 在 [撤銷清單] 刀鋒視窗中，按一下 [+新增憑證] 以開啟 [將憑證加入撤銷清單] 刀鋒視窗。
5. 在 [將憑證加入撤銷清單] 刀鋒視窗中，貼上連續一行文字且不含空格的憑證指紋。 按一下刀鋒視窗底部的 [確定]。
6. 更新完成之後，憑證無法再用於連線。 嘗試使用此憑證進行連線的用戶端會收到訊息，指出憑證不再有效。


## <a name="faq"></a>點對站常見問題集

[!INCLUDE [Point-to-Site FAQ](../../includes/vpn-gateway-point-to-site-faq-include.md)]

## <a name="next-steps"></a>後續步驟
一旦完成您的連接，就可以將虛擬機器加入您的虛擬網路。 如需詳細資訊，請參閱[虛擬機器](https://docs.microsoft.com/azure/#pivot=services&panel=Compute)。 若要了解網路與虛擬機器的詳細資訊，請參閱 [Azure 與 Linux VM 網路概觀](../virtual-machines/linux/azure-vm-network-overview.md)。

