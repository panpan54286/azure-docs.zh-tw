---
title: "Azure CLI 指令碼範例 - 將 Web 應用程式連線至 redis 快取 | Microsoft Docs"
description: "Azure CLI 指令碼範例 - 將 Web 應用程式連線至 redis 快取"
services: appservice
documentationcenter: appservice
author: syntaxc4
manager: erikre
editor: 
tags: azure-service-management
ms.assetid: bc8345b2-8487-40c6-a91f-77414e8688e6
ms.service: app-service
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: web
ms.date: 03/20/2017
ms.author: cfowler
translationtype: Human Translation
ms.sourcegitcommit: 0d8472cb3b0d891d2b184621d62830d1ccd5e2e7
ms.openlocfilehash: 043b1e90085a63c852e428ec25fa96e37eccb203
ms.lasthandoff: 03/21/2017

---

# <a name="connect-a-web-app-to-a-redis-cache"></a>將 Web 應用程式連線至 redis 快取

在此案例中，您會了解如何建立 Azure redis 快取和 Azure Web 應用程式。 然後，您會使用應用程式設定將 redis 快取連結至 Web 應用程式。

您可以視需要使用 [Azure CLI 安裝指南 (英文)](https://docs.microsoft.com/cli/azure/install-azure-cli) 中的指示來安裝 Azure CLI，然後執行 `az login` 來建立與 Azure 的連線。

這個範例適用於 Bash 殼層。 如需在 Windows 用戶端上執行 Azure CLI 指令碼的選項，請參閱[在 Windows 中執行 Azure CLI](../../virtual-machines/virtual-machines-windows-cli-options.md)。

## <a name="sample-script"></a>範例指令碼

[!code-azurecli[主要](../../../cli_scripts/app-service/connect-to-redis/connect-to-redis.sh "Azure Redis 快取")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>指令碼說明

此指令碼使用下列命令來建立資源群組、Web 應用程式、redis 快取和所有相關資源。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意事項 |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group#create) | 建立用來存放所有資源的資源群組。 |
| [az appservice plan create](https://docs.microsoft.com/cli/azure/appservice/plan#create) | 建立 App Service 方案。 這就像是 Azure Web 應用程式的伺服器陣列。 |
| [az appservice web create](https://docs.microsoft.com/cli/azure/appservice/web#create) | 在 App Service 方案內建立 Azure Web 應用程式。 |
| [az redis create](https://docs.microsoft.com/en-us/cli/azure/redis#create) | 建立新的 Redis 快取執行個體。 這是儲存資料的位置。 |
| [az redis list-keys](https://docs.microsoft.com/en-us/cli/azure/redis#list-keys) | 列出 redis 快取執行個體的存取金鑰。 |
| [az appservice web config appsetings update](https://docs.microsoft.com/cli/azure/appservice/web/config/appsettings#update) | 建立或更新 Azure Web 應用程式的應用程式設定。 應用程式設定會顯示為應用程式的環境變數。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure/overview)。

您可以在 [Azure App Service 文件](../app-service-cli-samples.md)中找到其他的 App Service CLI 指令碼範例。
