---
title: "適用於 IntelliJ 的 Azure 工具組新增功能 | Microsoft Docs"
description: "了解適用於 IntelliJ 的 Azure 工具組最新功能。"
services: 
documentationcenter: java
author: rmcmurray
manager: erikre
editor: 
ms.assetid: 46ed791f-df59-416a-809e-f52345ad973c
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: multiple
ms.devlang: Java
ms.topic: article
ms.date: 12/22/2016
ms.author: robmcm;asirveda;martinsawicki
translationtype: Human Translation
ms.sourcegitcommit: ff60ebaddd3a7888cee612f387bd0c50799496ac
ms.openlocfilehash: a2006dbcf0d63ef38651a0859dc654d531fd875a


---
# <a name="whats-new-in-the-azure-toolkit-for-intellij"></a>適用於 IntelliJ 的 Azure 工具組新增功能
## <a name="azure-toolkit-for-intellij-releases"></a>適用於 IntelliJ 的 Azure 工具組版本
本文包含適用於 IntelliJ 的 Azure 工具組的各種版本和最新更新相關資訊。

> [!NOTE]
> 另外還有適用於 Eclipse IDE 的 Azure 工具組。 如需詳細資訊，請參閱 [適用於 Eclipse 的 Azure 工具組]。
> 
> 

### <a name="august-26-2016"></a>2016 年 8 月 26 日
適用於 IntelliJ 的 Azure 工具組 - 2016 年 8 月版本包含下列增強功能：

* **自訂 JDK 散發套件**。 適用於 IntelliJ 的 Azure 工具組現在支援指定任意 JDK 版本，以及將其部署至 Azure WebApp 容器︰
  * 除了 Azure 提供的 JDK 之外，您也可以從 Azul Systems 在 Azure 上提供的多種 Zulu OpenJDK 版本中選擇。
  * 您也可以指定自己的 JDK 散發套件，前提是您必須以 ZIP 檔案的形式將它上傳到儲存體帳戶。
* **Azure Explorer 檢視的增強功能**：
  * 支援使用 Azure 的新 Resource Manager 模型管理虛擬機器︰您可以列出、建立和刪除以 Resource Manager 為基礎的虛擬機器，而不需要離開 IDE。
  * 支援使用 Azure 的 Resource Manager 管理儲存體帳戶 Blob，這項功能可輔助現有的「傳統」儲存體帳戶管理功能。
* **Microsoft JDBC Driver 6.0 for SQL Server**。 這項更新包含適用於 Microsoft SQL Server (6.0 版) 的最新 JDBC 驅動程式，該驅動程式現在以程式庫形式提供，可讓您輕鬆地加入專案中，因而取代舊有版本。

### <a name="june-29-2016"></a>2016 年 6 月 29 日
適用於 IntelliJ 的 Azure 工具組 - 2016 年 6 月版本包含下列增強功能：

* **Java 8 需求**。 適用於 IntelliJ 的 Azure 工具組現在需要 Java 8，雖然此需求僅適用於此工具組 - 您的應用程式可以繼續使用 Azure 所支援的所有 Java 版本。
* **支援最新的 Java JDK**。 適用於 IntelliJ 的 Azure 工具組現在支援最新版本的 Java JDK。
* **支援 Azure SDK v2.9.1**。 最新版的 Azure SDK 現在是適用於 IntelliJ 的 Azure 工具組的最低必要條件。
* **整合範例**。 適用於 IntelliJ 的 Azure 工具組目前精選了數個範例應用程式，可協助開發人員開始使用。
* **HDInsight 工具整合**。 Azure 的 HDInsight 工具現在隨附於適用於 IntelliJ 的 Azure 工具組。 如需詳細資訊，請參閱 [IntelliJ 的 HDInsight Tools 外掛程式]。
* **Java Web 應用程式的遠端偵錯**。 適用於 IntelliJ 的 Azure 工具組現在支援對 Azure App Service 上的 Java Web 應用程式進行遠端偵錯。

### <a name="april-12-2016"></a>2016 年 4 月 12 日
適用於 IntelliJ 的 Azure 工具組 - 2016 年 4 月版本包含下列增強功能：

* **支援 Azure SDK v2.9.0**。 最新版的 Azure SDK 現在是適用於 IntelliJ 的 Azure 工具組的最低必要條件。
* **與 Azure Web 應用程式支援有關的其他使用性、回應性和效能改進**。 工具組中多項與 Azure 通訊方式的效能最佳化會讓 UI 回應速度更快。
* **能夠在 IntelliJ 中刪除 Azure 現有的 Web 應用程式容器**。 適用於 IntelliJ 的 Azure 工具組現在可讓您刪除現有的 Azure Web 容器，不需要退出 IntelliJ。

## <a name="see-also"></a>另請參閱
如需適用於 Java IDE 的 Azure 套件組的詳細資訊，請參閱下列連結：

* [適用於 Eclipse 的 Azure 工具組]
  * [安裝 Azure Toolkit for Eclipse]
  * [Create a Hello World Web App for Azure in Eclipse (在 Eclipse 中建立 Azure Hello World Web 應用程式)]
  * [適用於 Eclipse 的 Azure 工具組的新功能]
* [Azure Toolkit for IntelliJ]
  * [安裝 Azure Toolkit for IntelliJ]
  * [在 IntelliJ 中建立 Azure Hello World Web 應用程式]
  * *適用於 IntelliJ 的 Azure 工具組新增功能 (本文)*

如需有關在 Azure 中使用 Java 的詳細資訊，請參閱 [Azure Java 開發人員中心]。

<!-- URL List -->

[適用於 Eclipse 的 Azure 工具組]: ./azure-toolkit-for-eclipse.md
[Azure Toolkit for IntelliJ]: ./azure-toolkit-for-intellij.md
[Create a Hello World Web App for Azure in Eclipse (在 Eclipse 中建立 Azure Hello World Web 應用程式)]: ./app-service-web/app-service-web-eclipse-create-hello-world-web-app.md
[在 IntelliJ 中建立 Azure Hello World Web 應用程式]: ./app-service-web/app-service-web-intellij-create-hello-world-web-app.md
[安裝 Azure Toolkit for Eclipse]: ./azure-toolkit-for-eclipse-installation.md
[安裝 Azure Toolkit for IntelliJ]: ./azure-toolkit-for-intellij-installation.md
[適用於 Eclipse 的 Azure 工具組的新功能]: ./azure-toolkit-for-eclipse-whats-new.md
[What's New in the Azure Toolkit for IntelliJ]: ./azure-toolkit-for-intellij-whats-new.md

[Azure Java 開發人員中心]: http://go.microsoft.com/fwlink/?LinkID=699547

[IntelliJ 的 HDInsight Tools 外掛程式]: ./hdinsight/hdinsight-apache-spark-intellij-tool-plugin.md



<!--HONumber=Feb17_HO2-->


