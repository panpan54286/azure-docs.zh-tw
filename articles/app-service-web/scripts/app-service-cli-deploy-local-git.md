---
title: "Azure CLI 指令碼範例 - 建立 Web 應用程式並從本機 Git 存放庫部署程式碼 | Microsoft Docs"
description: "Azure CLI 指令碼範例 - 建立 Web 應用程式並從本機 Git 存放庫部署程式碼"
services: app-service\web
documentationcenter: 
author: cephalin
manager: erikre
editor: 
tags: azure-service-management
ms.assetid: 048f98aa-f708-44cb-9b9e-953f67dc6da8
ms.service: app-service-web
ms.workload: web
ms.devlang: na
ms.topic: article
ms.date: 03/20/2017
ms.author: cephalin
translationtype: Human Translation
ms.sourcegitcommit: 0d8472cb3b0d891d2b184621d62830d1ccd5e2e7
ms.openlocfilehash: 10404f2814b167dfd54869f4b6e58f911adbdc8d
ms.lasthandoff: 03/21/2017

---

# <a name="create-a-web-app-and-deploy-code-from-a-local-git-repository"></a>建立 Web 應用程式並從本機 Git 存放庫部署程式碼

此範例指令碼會在 App Service 中建立 Web 應用程式及其相關資源，然後在本機 Git 存放庫中部署 Web 應用程式程式碼。

您可以視需要使用 [Azure CLI 安裝指南 (英文)](https://docs.microsoft.com/cli/azure/install-azure-cli) 中的指示來安裝 Azure CLI，然後執行 `az login` 來建立與 Azure 的連線。 此外，應用程式程式碼必須認可到本機 Git 儲存機制。

這個範例適用於 Bash 殼層。 如需在 Windows 用戶端上執行 Azure CLI 指令碼的選項，請參閱[在 Windows 中執行 Azure CLI](../../virtual-machines/virtual-machines-windows-cli-options.md)。

## <a name="sample-script"></a>範例指令碼

[!code-azurecli[主要](../../../cli_scripts/app-service/deploy-local-git/deploy-local-git.sh?highlight=3-5 "建立 Web 應用程式並從本機 Git 存放庫部署程式碼")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意事項 |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group#create) | 建立用來存放所有資源的資源群組。 |
| [az appservice plan create](https://docs.microsoft.com/cli/azure/appservice/plan#create) | 建立 App Service 方案。 |
| [az appservice web create](https://docs.microsoft.com/cli/azure/appservice/web#delete) | 建立 Azure Web 應用程式。 |
| [az appservice web deployment user set](https://docs.microsoft.com/cli/azure/appservice/web/deployment/user#set) | 設定 App Service 的帳戶層級部署認證。 |
| [az appservice web source-control config-local-git](https://docs.microsoft.com/cli/azure/appservice/web/source-control#config-local-git) | 建立本機 Git 存放庫的原始檔控制組態。 |
| [az appservice web browse](https://docs.microsoft.com/cli/azure/appservice/web#browse) | 在瀏覽器中開啟 Azure Web 應用程式。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure/overview)。

您可以在 [Azure App Service 文件](../app-service-cli-samples.md)中找到其他的 App Service CLI 指令碼範例。
