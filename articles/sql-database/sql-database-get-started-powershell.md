---
title: "Azure PowerShell︰建立 SQL Database | Microsoft Docs"
description: "了解如何在 Azure 入口網站中，建立 SQL Database 邏輯伺服器、伺服器層級防火牆規則和資料庫。"
keywords: "sql database 教學課程, 建立 sql database"
services: sql-database
documentationcenter: 
author: CarlRabeler
manager: jhubbard
editor: 
ms.assetid: 
ms.service: sql-database
ms.custom: quick start
ms.workload: data-management
ms.tgt_pltfrm: na
ms.devlang: PowerShell
ms.topic: hero-article
ms.date: 03/13/2017
ms.author: carlrab
translationtype: Human Translation
ms.sourcegitcommit: 4f2230ea0cc5b3e258a1a26a39e99433b04ffe18
ms.openlocfilehash: 70cf89a5832aee2d0c303e0d40e104d84837b50c
ms.lasthandoff: 03/25/2017

---

# <a name="create-a-single-azure-sql-database-using-powershell"></a>使用 PowerShell 建立單一 Azure SQL Database

PowerShell 可用來從命令列或在指令碼中建立和管理 Azure 資源。 本指南詳述如何使用 PowerShell 在 [Azure SQL Database 邏輯伺服器](sql-database-features.md)的 [Azure 資源群組](../azure-resource-manager/resource-group-overview.md)中部署 Azure SQL Database。

開始之前，請確定已安裝最新版 PowerShell。 如需詳細資訊，請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azureps-cmdlets-docs)。 

## <a name="log-in-to-azure"></a>登入 Azure

使用 [Add-AzureRmAccount](https://docs.microsoft.com/powershell/resourcemanager/azurerm.profile/v2.5.0/add-azurermaccount) 命令登入 Azure 訂用帳戶並遵循畫面上的指示。

```powershell
Add-AzureRmAccount
```

## <a name="create-a-resource-group"></a>建立資源群組

使用 [New-AzureRmResourceGroup](https://docs.microsoft.com/powershell/resourcemanager/azurerm.resources/v3.5.0/new-azurermresourcegroup) 命令建立 [Azure 資源群組](../azure-resource-manager/resource-group-overview.md)。 資源群組是在其中以群組方式部署與管理 Azure 資源的邏輯容器。 下列範例會在 `westeurope` 位置建立名為 `myResourceGroup` 的資源群組。

```powershell
New-AzureRmResourceGroup -Name "myResourceGroup" -Location "westeurope"
```
## <a name="create-a-logical-server"></a>建立邏輯伺服器

使用 [New-AzureRmSqlServer](https://docs.microsoft.com/powershell/resourcemanager/azurerm.sql/v2.5.0/new-azurermsqlserver) 命令建立 [Azure SQL Database 邏輯伺服器](sql-database-features.md)。 邏輯伺服器包含一組當作群組管理的資料庫。 下列範例會使用名為 `ServerAdmin` 的系統管理員登入和密碼 `ChangeYourAdminPassword1` 在資源群組中建立隨機命名的伺服器。 視需要取代這些預先定義的值。

```powershell
$servername = "server-$(Get-Random)"
New-AzureRmSqlServer -ResourceGroupName "myResourceGroup" `
    -ServerName $servername `
    -Location "westeurope" `
    -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList "ServerAdmin", $(ConvertTo-SecureString -String "ChangeYourAdminPassword1" -AsPlainText -Force))
```

## <a name="configure-a-server-firewall-rule"></a>設定伺服器防火牆規則

使用 [New-AzureRmSqlServerFirewallRule](https://docs.microsoft.com/powershell/resourcemanager/azurerm.sql/v2.5.0/new-azurermsqlserverfirewallrule) 命令建立 [Azure SQL Database 伺服器層級防火牆規則](sql-database-firewall-configure.md)。 伺服器層級防火牆規則可讓外部應用程式 (例如 SQL Server Management Studio 或 SQLCMD 公用程式) 穿過 SQL Database 服務防火牆連線到 SQL Database。 下列範例會建立適用於預先定義之位址範圍的防火牆規則，在此範例中，這是整個可能的 IP 位址範圍。 以外部 IP 位址或 IP 位址範圍的值取代這些預先定義的值。 

```powershell
New-AzureRmSqlServerFirewallRule -ResourceGroupName "myResourceGroup" `
    -ServerName $servername `
    -FirewallRuleName "AllowSome" -StartIpAddress "0.0.0.0" -EndIpAddress "255.255.255.255"
```

## <a name="create-a-blank-database"></a>建立空白資料庫

在伺服器中使用 [New-AzureRmSqlDatabase](https://docs.microsoft.com/powershell/resourcemanager/azurerm.sql/v2.5.0/new-azurermsqldatabase) 命令建立具有 [S0 效能等級](sql-database-service-tiers.md)的空白 SQL Database。 下列範例會建立名為 `mySampleDatabase` 的資料庫。 視需要取代此預先定義的值。

```powershell
New-AzureRmSqlDatabase  -ResourceGroupName "myResourceGroup" `
    -ServerName $servername `
    -DatabaseName "MySampleDatabase" `
    -RequestedServiceObjectiveName "S0"
```

## <a name="clean-up-resources"></a>清除資源

此集合中的其他快速入門會建置在本快速入門。 如果您打算繼續進行後續的快速入門或教學課程，請勿清除在此快速入門中建立的資源。 如果您不打算繼續，請使用下列命令來刪除本快速入門建立的所有資源。

```powershell
Remove-AzureRmResourceGroup -ResourceGroupName "myResourceGroup"
```

## <a name="next-steps"></a>後續步驟

- 若要使用 SQL Server Management Studio 來連線和查詢，請參閱[使用 SSMS 連線及查詢](sql-database-connect-query-ssms.md)
- 若要使用 Visual Studio 來連線，請參閱[使用 Visual Studio 連線及查詢](sql-database-connect-query.md)。
* 如需 SQL Database 的技術概觀，請參閱[關於 SQL Database 服務](sql-database-technical-overview.md)。

