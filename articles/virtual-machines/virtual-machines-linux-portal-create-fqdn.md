---
title: "在 Azure 入口網站中建立 Linux VM 的 FQDN | Microsoft Docs"
description: "了解如何在 Azure 入口網站中為基於 Resource Manager 的虛擬機器建立完整格式的網域名稱或 FQDN。"
services: virtual-machines-linux
documentationcenter: 
author: iainfoulds
manager: timlt
editor: tysonn
tags: azure-resource-manager
ms.assetid: 2cd6c249-a737-4a0a-b5ba-e1c09e551b30
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 03/14/2017
ms.author: iainfou
ms.custom: H1Hack27Feb2017
translationtype: Human Translation
ms.sourcegitcommit: cea53acc33347b9e6178645f225770936788f807
ms.openlocfilehash: b35d2886c1319623a2a56851e52fa25228bd0592
ms.lasthandoff: 03/03/2017


---
# <a name="create-a-fully-qualified-domain-name-in-the-azure-portal-for-a-linux-vm"></a>在 Azure 入口網站中建立 Linux VM 的完整網域名稱

在 [Azure 入口網站](https://portal.azure.com) 中使用 Resource Manager 部署模型來建立虛擬機器 (VM) 時，會自動建立虛擬機器的公用 IP 資源。 您可以使用此 IP 位址從遠端存取 VM。 雖然入口網站預設不會建立 [fully qualified domain name (完整網域名稱)](https://en.wikipedia.org/wiki/Fully_qualified_domain_name)(或稱 FQDN)，但是您可以在 VM 建立之後新增一個。 這篇文章示範建立 DNS 名稱或 FQDN 的步驟。

[!INCLUDE [virtual-machines-common-portal-create-fqdn](../../includes/virtual-machines-common-portal-create-fqdn.md)]

您現在可以使用這個 DNS 名稱 (例如搭配 `ssh ops@mydns.westus.cloudapp.azure.com`) 從遠端連接到 VM。

## <a name="next-steps"></a>後續步驟
既然您的 VM 已經有公用 IP 和 DNS 名稱，您便可以部署通用應用程式架構或服務，例如 nginx、MongoDB、Docker 等等。

您也可以進一步閱讀[使用 Resource Manager](../azure-resource-manager/resource-group-overview.md)，以取得建置 Azure 部署的相關秘訣。


