---
title: "Azure CLI 指令碼範例 - 建立具有 OMS 監視的 Windows Server 2016 VM | Microsoft Docs"
description: "Azure CLI 指令碼範例 - 建立具有 OMS 監視的 Windows Server 2016 VM"
services: virtual-machines-Windows
documentationcenter: virtual-machines
author: rickstercdn
manager: timlt
editor: tysonn
tags: 
ms.assetid: 
ms.service: virtual-machines-Windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-Windows
ms.workload: infrastructure
ms.date: 02/23/2017
ms.author: rclaus
translationtype: Human Translation
ms.sourcegitcommit: 0d8472cb3b0d891d2b184621d62830d1ccd5e2e7
ms.openlocfilehash: 70df053ba31974e753830487f1320d0d2edc4403
ms.lasthandoff: 03/21/2017

---

# <a name="monitor-a-vm-with-operations-management-suite"></a>使用 Operations Management Suite 監視 VM

此指令碼會建立 Azure 虛擬機器、安裝 Operations Management Suite (OMS) 代理程式，並向 OMS 工作區註冊系統。 執行此指令碼後，就能在 OMS 主控台中看到虛擬機器。

您可以視需要使用 [Azure CLI 安裝指南 (英文)](https://docs.microsoft.com/cli/azure/install-azure-cli) 中的指示來安裝 Azure CLI，然後執行 `az login` 來建立與 Azure 的連線。 此外，您需要更新 OMS 工作區識別碼和工作區金鑰。 最後，您應該將指令碼開頭的 $AdminPassword 變數變更為唯一值，並符合密碼複雜性需求。

這個範例適用於 Bash 殼層。 如需在 Windows 上執行 Azure CLI 指令碼的選項，請參閱[在 Windows 中執行 Azure CLI](../virtual-machines-windows-cli-options.md)。

## <a name="sample-script"></a>範例指令碼

[!code-azurecli[主要](../../../cli_scripts/virtual-machine/create-vm-monitor-oms/create-windows-vm-monitor-oms.sh "快速建立 VM")]

## <a name="clean-up-deployment"></a>清除部署 

執行下列命令來移除資源群組、VM 和所有相關資源。

```azurecli
az group delete --name myResourceGroup --yes
```

## <a name="script-explanation"></a>指令碼說明

此指令碼使用下列命令來建立資源群組、虛擬機器和所有相關資源。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意事項 |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group#create) | 建立用來存放所有資源的資源群組。 |
| [az vm create](https://docs.microsoft.com/cli/azure/vm#create) | 建立虛擬機器，並將它連線到網路卡、虛擬網路、子網路及 NSG。 此命令也會指定要使用的虛擬機器映像和管理認證。  |
| [azure vm extension set](https://docs.microsoft.com/cli/azure/vm/extension#set) | 對虛擬機器執行 VM 擴充功能。 在此情況下，會使用 Operations Management Suite 代理程式擴充功能來安裝 OMS 代理程式，並在 OMS 工作區中註冊 VM。 |
| [az group delete](https://docs.microsoft.com/cli/azure/vm/extension#set) | 刪除資源群組，包括所有的巢狀資源。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure/overview)。

您可以在 [Azure Windows VM 文件](../virtual-machines-windows-cli-samples.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)中找到其他的虛擬機器 CLI 指令碼範例。

