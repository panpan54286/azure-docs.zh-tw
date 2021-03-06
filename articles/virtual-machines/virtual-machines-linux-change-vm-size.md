---
title: "如何使用 Azure CLI 2.0 來調整 Linux VM 大小 | Microsoft Docs"
description: "如何藉由變更 VM 的大小來相應增加或相應減少 Linux 虛擬機器。"
services: virtual-machines-linux
documentationcenter: na
author: mikewasson
manager: timlt
editor: 
tags: 
ms.assetid: e163f878-b919-45c5-9f5a-75a64f3b14a0
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/10/2017
ms.author: mwasson
ms.custom: H1Hack27Feb2017
translationtype: Human Translation
ms.sourcegitcommit: cea53acc33347b9e6178645f225770936788f807
ms.openlocfilehash: d149e51441dbf440732b1930245d27a6ff144970
ms.lasthandoff: 03/03/2017


---
# <a name="resize-a-linux-virtual-machine-using-cli-20"></a>使用 CLI 2.0 來調整 Linux 虛擬機器大小

部屬虛擬機器 (VM) 之後，您可以藉由變更 [VM 大小][vm-sizes]來相應增加或減少 VM。 在某些情況下，您必須先解除配置 VM。 如果無法在裝載 VM 的硬體叢集上取得所需的大小，您可能需要解除配置 VM。 本文詳述如何使用 Azure CLI 2.0 來調整 Linux VM 的大小。 您也可以使用 [Azure CLI 1.0](virtual-machines-linux-change-vm-size-nodejs.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) 來執行這些步驟。

## <a name="cli-versions-to-complete-the-task"></a>用以完成工作的 CLI 版本
您可以使用下列其中一個 CLI 版本來完成工作︰

- [Azure CLI 1.0](#resize-a-linux-vm) – 適用於傳統和資源管理部署模型的 CLI (本文章)
- [Azure CLI 2.0](virtual-machines-linux-change-vm-size.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) - 適用於資源管理部署模型的新一代 CLI



若要調整 VM 的大小，您需要安裝最新的 [Azure CLI 2.0](/cli/azure/install-az-cli2)，並使用 [az login](/cli/azure/#login) 來登入 Azure 帳戶。

1. 使用 [az vm list-vm-resize-options](/cli/azure/vm#list-vm-resize-options) 檢視裝載 VM 之硬體叢集上可用的 VM 大小清單。 下列範例會針對資源群組 `myResourceGroup` 區域中名為 `myVM` 的 VM 列出 VM 大小：
   
    ```azurecli
    az vm list-vm-resize-options --resource-group myResourceGroup --name myVM --output table
    ```

2. 如果已列出所需的大小，請使用 [az vm resize](/cli/azure/vm#resize) 來調整 VM 的大小。 下列範例會將名為 `myVM` 的 VM 大小調整為 `Standard_DS3_v2` 大小：
   
    ```azurecli
    az vm resize --resource-group myResourceGroup --name myVM --size Standard_DS3_v2
    ```
   
    VM 會在此程序中重新啟動。 重新啟動之後，會重新對應現有的 OS 和資料磁碟。 暫存磁碟的相關資料都會遺失。

3. 如果未列出所需的 VM 大小，您需要先使用 [az vm deallocate](/cli/azure/vm#deallocate) 解除配置 VM。 此程序可讓 VM 的大小調整為區域支援的任何可用大小，然後啟動。 下列步驟會解除配置、調整大小，然後啟動 `myResourceGroup` 資源群組中名為 `myVM` 的 VM：
   
    ```azurecli
    az vm deallocate --resource-group myResourceGroup --name myVM
    az vm resize --resource-group myResourceGroup --name myVM --size Standard_DS3_v2
    az vm start --resource-group myResourceGroup --name myVM
    ```
   
   > [!WARNING]
   > 將 VM 解除配置也會釋出指派給該 VM 的任何動態 IP 位址。 不會影響作業系統和資料磁碟。

## <a name="next-steps"></a>後續步驟
如需提高延展性，可執行多個 VM 執行個體並相應放大。 如需詳細資訊，請參閱[在虛擬機器擴展集中自動調整 Linux 機器][scale-set]。 

<!-- links -->
[boot-diagnostics]: https://azure.microsoft.com/en-us/blog/boot-diagnostics-for-virtual-machines-v2/
[scale-set]: ../virtual-machine-scale-sets/virtual-machine-scale-sets-linux-autoscale.md 
[vm-sizes]: virtual-machines-linux-sizes.md

