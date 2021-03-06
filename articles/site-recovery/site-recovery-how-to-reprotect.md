---
title: "如何從 Azure 重新保護到內部部署網站 | Microsoft Docs"
description: "將 VM 容錯移轉到 Azure 之後，您可以起始將 VM 容錯回復到內部部署的作業。 了解在容錯回復之前進行重新保護的步驟。"
services: site-recovery
documentationcenter: 
author: ruturaj
manager: gauravd
editor: 
ms.assetid: 44813a48-c680-4581-a92e-cecc57cc3b1e
ms.service: site-recovery
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: 
ms.date: 02/13/2017
ms.author: ruturajd
translationtype: Human Translation
ms.sourcegitcommit: 0bec803e4b49f3ae53f2cc3be6b9cb2d256fe5ea
ms.openlocfilehash: 7b7177faa9fa571d3a62ee15b4a0fbfdab3a097f
ms.lasthandoff: 03/24/2017


---
# <a name="reprotect-from-azure-to-an-on-premises-site"></a>從 Azure 重新保護至內部部署網站

## <a name="overview"></a>概觀
此文章說明如何將 Azure 虛擬機器從 Azure 重新保護到內部部署網站。 當您準備好使用此[使用 Azure Site Recovery 將 VMWare 虛擬機器和實體伺服器複寫至 Azure](site-recovery-failover.md)，將已從內部部署網站容錯移轉至 Azure 的 VMware 虛擬機器或 Windows/Linux 實體伺服器容錯回復時，請依照此文章中的指示執行。

完成重新保護且受保護的虛擬機器開始複寫之後，您可以在虛擬機器上起始容錯回復，以將它們回復到內部部署網站。

在本文末尾或 [Azure Recovery Services Forum (Azure 復原服務論壇)](https://social.msdn.microsoft.com/forums/azure/home?forum=hypervrecovmgr) 中張貼意見或問題。

如需快速概觀，請觀看下列有關如何從 Azure 容錯移轉到內部部署網站的影片。
> [!VIDEO https://channel9.msdn.com/Series/Azure-Site-Recovery/VMware-to-Azure-with-ASR-Video5-Failback-from-Azure-to-On-premises/player]


## <a name="prerequisites"></a>必要條件
以下是準備重新保護之前必須採取的先決條件步驟或考量事項。

* 如果您想要容錯回復到的目標虛擬機器是由 vCenter 伺服器進行管理，您必須確定已擁有在 vCenter 伺服器上探索虛擬機器的必要權限。 [閱讀更多資訊](site-recovery-vmware-to-azure-classic.md#vmware-permissions-for-vcenter-access)。
* 如果內部部署虛擬機器上有快照，重新保護程序將會失敗。 您可以先刪除快照，再進行重新保護。
* 在容錯回復之前，您將需要先建立兩個其他元件：
  * **建立處理伺服器**。 處理序伺服器會從在 Azure 中受保護的虛擬機器接收資料，並將資料傳送至內部部署網站。 低延遲網路必須位於處理伺服器與受保護虛擬機器之間。 因此，如果您使用 Azure ExpressRoute 連線則可擁有內部部署處理序伺服器，或如果使用 VPN 則為 Azure 處理序伺服器。
  * **建立主要目標伺服器**：主要目標伺服器會接收容錯回復資料。 您建立的內部部署管理伺服器預設會安裝主要目標伺服器。 不過，您可能必須根據容錯回復流量的大小，另外建立一部用來容錯回復的主要目標伺服器。
        * [Linux 虛擬機器需要 Linux 主要目標伺服器](site-recovery-how-to-install-linux-master-target.md)。
        * Windows 虛擬機器需要 Windows 主要目標伺服器。 您可以再次使用內部部署處理序伺服器和主要目標電腦。
* 執行容錯回復時，組態伺服器必須為內部部署。 在容錯回復期間，虛擬機器必須存在於組態伺服器資料庫中。 否則，將無法成功容錯回復。 請確定您有排定來定期備份伺服器。 發生災害時，您需要使用相同的 IP 位址還原伺服器，讓容錯回復運作。
* 確定您在 VMware 中將主要目標虛擬機器的組態參數設定設為 disk.EnableUUID=true。 如果此列不存在，請新增它。 需要此設定才能提供一致的 UUID 給虛擬機器磁碟 (VMDK)，使其可正確掛接。
* 您無法使用主要目標伺服器的儲存體 vMaster。 這會造成容錯回復失敗。 虛擬機器不會啟動，因為磁碟不可供其使用。
* 您需要將新的磁碟機新增至主要目標伺服器。 此磁碟機稱為保留磁碟機。 新增磁碟並格式化磁碟機。
* [重新保護之前在主要目標上檢查的常見項目](site-recovery-how-to-reprotect.md#common-things-to-check-after-completing-installation-of-the-master-target-server)中，列出主要目標的其他必要條件。


### <a name="why-do-i-need-a-s2s-vpn-or-an-expressroute-connection-to-replicate-data-back-to-the-on-premises-site"></a>為什麼需要 S2S VPN 或 ExpressRoute 連線，才能將資料複寫回到內部部署網站？
從內部部署複寫到 Azure 的動作可以透過網際網路或具有公用對等互連的 ExpressRoute 連線來執行，重新保護和容錯回復需要站台對站台 (S2S) VPN 以複寫資料。 必須提供網路，讓 Azure 中已容錯移轉的虛擬機器可以觸達 (ping) 內部部署組態伺服器。 您也可能會在容錯移轉虛擬機器的 Azure 網路中部署處理序伺服器。 此處理序伺服器也應該能夠與內部部署組態設定伺服器進行通訊。

### <a name="when-should-i-install-a-process-server-in-azure"></a>何時應該在 Azure 中安裝處理序伺服器？


您要在 Azure 上重新保護的虛擬機器會將複寫資料傳送至處理序伺服器。 您應該設定網路，以便 Azure 上的虛擬機器可以連線到處理序伺服器。

您可以在 Azure 中部署處理伺服器，或使用在容錯移轉期間使用的現有處理伺服器。 要考量的重點是將資料從 Azure 上的虛擬機器傳送到處理伺服器的延遲。

如果您已設定 ExpressRoute 連線，可使用內部部署處理序伺服器來傳送資料，因為虛擬機器和處理序伺服器之間的延遲很低。

 ![Expressroute 的架構圖](./media/site-recovery-failback-azure-to-vmware-classic/architecture.png)



然而，若您只有 S2S VPN，則我們建議您在 Azure 中部署處理伺服器。

 ![VPN 的架構圖](./media/site-recovery-failback-azure-to-vmware-classic/architecture2.png)


請記住，只會在 S2S VPN 或您 ExpressRoute 網路的私人對等互連上進行複寫。 請確定該網路通道上有足夠的可用頻寬。

深入了解如何安裝 [Azure 處理序伺服器](site-recovery-vmware-setup-azure-ps-resource-manager.md)。

### <a name="what-are-the-ports-to-be-open-on-different-components-so-that-reprotect-can-work"></a>在不同的元件上要開啟哪些連接埠，重新保護才能運作？

![容錯移轉-容錯回復所有的連接埠](./media/site-recovery-failback-azure-to-vmware-classic/Failover-Failback.png)

### <a name="which-master-target-server-should-be-used-for-reprotect"></a>哪一個主要目標伺服器應用於重新保護？
必須有內部部署主要目標伺服器才可接收來自處理伺服器的資料，並接著將資料寫入到內部部署內部部署的 VMDK。 如果您要保護 Windows 虛擬機器，您需要 Windows 主要目標伺服器。 您可以再次使用內部部署處理序伺服器和主要目標電腦 <!-- !todo component -->。 針對 Linux 虛擬機器，您需要設定其他內部部署 Linux 主要目標。


按一下下列連結以了解如何安裝主要目標伺服器：

* [如何安裝 Windows 主要目標伺服器](site-recovery-vmware-to-azure.md#run-site-recovery-unified-setup)
* [如何安裝 Linux 主要目標伺服器](site-recovery-how-to-install-linux-master-target.md)


#### <a name="common-things-to-check-after-completing-installation-of-the-master-target-server"></a>完成主要目標伺服器安裝後必須檢查的常見項目

* 若虛擬機器存在於內部部署 vCenter 伺服器上，主要目標伺服器必須能夠存取內部部署虛擬機器的 VMDK。 需要存取才可將複寫的資料寫入虛擬機器的磁碟。 請確定內部部署內部部署的資料存放區已掛接在主要目標的主機上並已設定讀寫權限。

* 如果虛擬機器不是在 vCenter Server 上的內部部署中出現，您需要在重新保護期間建立新的虛擬機器。 您必須在建立主要目標的 ESX 主機上建立此虛擬機器。 請謹慎選擇 ESX 主機，以便在您要的主機上建立容錯回復虛擬機器。

* 您無法使用主要目標伺服器的儲存體 vMotion。 這會造成容錯回復失敗。 虛擬機器不會啟動，因為磁碟不可供其使用。

* 您必須將新的磁碟機新增至現有的 Windows 主要目標伺服器。 此磁碟機稱為保留磁碟機。 新增磁碟並格式化磁碟機。 保留磁碟機可用來停止虛擬機器複寫回到內部部署站台時的時間點。 以下為保留磁碟機的一些準則，如果沒有這些準則，磁碟機將不會針對主要目標伺服器列出。

   * 磁碟區不能使用於其他用途 (例如做為複寫目標等)。

   * 磁碟區不能處於鎖定模式。

   * 磁碟區不能是快取磁碟區。 該磁碟區上不能有主要目標安裝。 處理伺服器和主要目標自訂安裝磁碟區不符合做為保留磁碟區的資格。 當處理序伺服器和主要目標安裝在磁碟區時，磁碟區是主要目標的快取磁碟區。

   * 磁碟區的檔案系統類型不能是 FAT 或 FAT32。

   * 磁碟區容量不能為零。

   * Windows 的預設保留磁碟區為 R 磁碟區。

   * Linux 的預設保留磁碟區為 /mnt/retention。

* Linux 容錯移轉虛擬機器需要 Linux 主要目標伺服器。 Windows 容錯移轉虛擬機器需要 Windows 主要目標伺服器。

* 在主要目標伺服器上安裝 VMware 工具。 若沒有 VMware 工具，將無法偵測主要目標之 ESXi 主機上的資料存放區。

* 使用 vCenter 屬性在主要目標虛擬機器上啟用 disk.EnableUUID=true 參數。 <!-- !todo Needs link. -->

* 主要目標應該至少連結一個虛擬機器檔案系統 (VMFS) 資料存放區。 如果沒有的話，重新保護頁面上的**資料存放區**輸入會空白，您將無法繼續。

* 主要目標伺服器在磁碟上不能有快照集。 如果有快照集，重新保護和容錯回復會失敗。

* 主要目標不能有 Paravirtual SCSI 控制器。 控制器只能是 LSI Logic 控制器。 如果沒有 LSI Logic 控制器，重新保護會失敗。

<!--
### Failback policy
To replicate back to on-premises, you will need a failback policy. This policy get automatically created when you create a forward direction policy. Note that

1. This policy gets auto associated with the configuration server during creation.
2. This policy is not editable.
3. The set values of the policy are (RPO Threshold = 15 Mins, Recovery Point Retention = 24 Hours, App Consistency Snapshot Frequency = 60 Mins)
   ![](./media/site-recovery-failback-azure-to-vmware-new/failback-policy.png)

-->

## <a name="steps-to-reprotect"></a>重新保護的步驟

重新保護之前，請確定您已在 Azure 和內部部署 Windows 或 [Linux 主要目標](site-recovery-how-to-install-linux-master-target.md)中安裝[處理序伺服器](site-recovery-vmware-setup-azure-ps-resource-manager.md)。

> [!NOTE]
> 在 Azure 虛擬機器啟動之後，需要經過一些時間，代理程式才會註冊回到組態伺服器中 (最多 15 分鐘)。 在這段時間，重新保護會失敗，且有錯誤訊息指出未安裝代理程式。 請等候幾分鐘，然後再次嘗試重新保護。



1. 在 [保存庫] > [已複寫的項目] 中，以滑鼠右鍵按一下已容錯移轉的虛擬機器，然後選取 [重新保護]。 您也可以按一下機器，並從命令按鈕中選取**重新保護**。
2. 在刀鋒視窗中，您可以看到已選取保護方向 [Azure 至內部部署]。
3. 在 [主要目標伺服器] 和 [處理伺服器] 中，選取內部部署主要目標伺服器和處理伺服器。
4. 選取您想要將內部部署磁碟復原至該位置的 [資料存放區]。 當內部部署虛擬機器已被刪除且您需要建立新的磁碟時，請使用此選項。 如果磁碟已存在，則會忽略此選項，但您仍然需要指定一個值。
5. 選擇保留磁碟機。
6. 將會自動選取容錯回復原則。
7. 在您按一下 [確定] 以開始重新保護之後，就會開始執行將內部部署從 Azure 複寫到內部部署網站的作業。 您可以在 [作業]  索引標籤上追蹤進度。

如果您想要復原到替代位置 (當內部部署虛擬機器已被刪除時)，請選取為主要目標伺服器所設定的保留磁碟機和資料存放區。 當您容錯回復到內部部署網站時，容錯回復保護計畫中的 VMware 虛擬機器會使用與主要目標伺服器相同的資料存放區。 在 vCenter 中會建立新的虛擬機器。
若要將 Azure 上的虛擬機器復原到現有的內部部署虛擬機器，則必須在主要目標伺服器的 ESXi 主機上掛接內部部署虛擬機器的資料存放區並設定讀寫權限。
    ![重新保護對話方塊](./media/site-recovery-failback-azure-to-vmware-new/reprotectinputs.png)

您也可以在復原計劃層級重新保護。 一個複寫群組只能使用復原計畫來重新保護它。 當您使用復原方案來重新保護時，您必須為每部受保護機器提供值。

> [!NOTE]
> 複寫群組應該使用相同的主要目標重新保護。 如果使用不同的主要目標伺服器重新保護，則伺服器無法提供一個共同的點的時間。


重新保護成功之後，虛擬機器將進入受保護狀態。

## <a name="next-steps"></a>後續步驟

在虛擬機器進入受保護狀態後，您就可以起始容錯回復。 容錯回復會關閉 Azure 中的虛擬機器，並啟動內部部署虛擬機器。 因此，應用程式有一小段停機時間。 因此，請選擇應用程式允許停機時的容錯回復時間。

[起始虛擬機器容錯回復的步驟](site-recovery-how-to-failback-azure-to-vmware.md#steps-to-fail-back)

## <a name="common-issues"></a>常見問題

* 如果您使用範本來建立虛擬機器，請確定每部虛擬機器有磁碟的唯一 UUID。 如果內部部署虛擬機器和主要目標的 UUID 衝突 (因為兩者都是從相同的範本建立)，則重新保護會失敗。 您必須部署另一個不是從相同範本建立的主要目標。
* 如果您執行唯讀的使用者 vCenter 探索並保護虛擬機器，保護會成功且容錯回復可作用。 在重新保護期間，容錯回復會失敗，因為無法探索資料存放區。 徵兆是重新保護期間不會列出資料存放區。 若要解決此問題，您可以使用有適當權限的帳戶更新 vCenter 認證並重試作業。 如需詳細資訊，請參閱[使用 Azure Site Recovery 將 VMWare 虛擬機器和實體伺服器複寫至 Azure](site-recovery-vmware-to-azure-classic.md#vmware-permissions-for-vcenter-access)。
* 當您將 Linux 虛擬機器容錯回復並在內部部署執行它時，您會看到 Network Manager 套件已從該機器解除安裝。 發生此解除安裝的原因是，當在 Azure 中復原虛擬機器時，Network Manager 套件已被移除。
* 當 Linux 虛擬機器已設定靜態 IP 位址且容錯移轉至 Azure 時，就會從 DHCP 取得 IP 位址。 當您容錯移轉至內部部署時，該虛擬機器會繼續使用 DHCP 取得 IP 位址。 視需要手動登入機器，並將 IP 位址設定回靜態位址。 Windows 虛擬機器可以再次取得其靜態 IP。
* 如果您是使用 ESXi 5.5 免費版或 vSphere 6 Hypervisor 免費版，則容錯移轉會成功，但容錯回復不會成功。 若要啟用容錯回復，請升級到上述程式的評估授權。
* 若無法從處理伺服器連線到組態伺服器，請使用 Telnet 檢查連線到組態伺服器連接埠 443 的連線。 您也可以嘗試從處理伺服器 Ping 組態伺服器。 當處理伺服器已連線到組態伺服器時，應該會有活動訊號。
* 如果您要嘗試容錯回復至替代的 vCenter，請確定您的新 vCenter 已被探索到，而且主要目標伺服器也已被探索到。 一般的徵兆是，您會發現在 [重新保護] 對話方塊中無法存取/看到資料存放區。
* 做為實體內部部署伺服器保護的 Windows Server 2008 R2 SP1 伺服器無法從 Azure 容錯回復到內部部署網站。

