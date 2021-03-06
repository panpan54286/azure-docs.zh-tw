---
title: "適用於行動電話的 Microsoft Authenticator 應用程式 | Microsoft Docs"
description: "了解如何升級至最新版的 Azure Authenticator。"
services: multi-factor-authentication
documentationcenter: 
author: kgremban
manager: femila
editor: curtland
ms.assetid: 3065a1ee-f253-41f0-a68d-2bd84af5ffba
ms.service: multi-factor-authentication
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/25/2016
ms.author: kgremban
ms.custom: H1Hack27Feb2017
translationtype: Human Translation
ms.sourcegitcommit: 6872b216ab2bfd5823b65b1bb70d503c7628ee13
ms.openlocfilehash: 26d9f04f9d3019ab5fcad48ef1e9821a4dd6a724
ms.lasthandoff: 03/01/2017

---
# <a name="get-started-with-the-microsoft-authenticator-app"></a>開始使用 Microsoft 驗證器應用程式
Microsoft 驗證器應用程式可在 Azure 帳戶 (例如 bsimon@contoso.onmicrosoft.com)、內部部署工作帳戶 (例如 bsimon@contoso.com) 或 Microsoft 帳戶 (例如 bsimon@outlook.com) 提供額外一層安全性。

此應用程式可以下列兩種方式之一運作︰

* **通知**。 此應用程式可協助防止未經授權即存取帳戶，並藉由推播通知到您的手機或平板電腦來停止詐騙交易。 只需檢視通知，如果合法，則選取 [驗證] 。 否則，您可以選取 [拒絕] 。 如需拒絕通知的相關資訊，請參閱＜如何使用 Multi-Factor Authentication 的拒絕和回報詐騙功能＞。
* **密碼與驗證碼**。 此應用程式可以作為軟體權杖來產生 OATH 驗證碼。 在您輸入使用者名稱和密碼後，必須在登入畫面出現提示時輸入應用程式提供的驗證碼。 驗證碼提供第二種形式的驗證。

Microsoft Authenticator 應用程式會取代 Azure Authenticator 應用程式。  Azure 驗證器應用程式仍可繼續運作，但如果您決定改用新的 Microsoft 驗證器應用程式，本文可協助您。  

## <a name="install-the-app"></a>安裝應用程式
Microsoft 驗證器應用程式適用於 [Windows Phone](http://go.microsoft.com/fwlink/?Linkid=825071)、[Android](http://go.microsoft.com/fwlink/?Linkid=825072) 和 [IOS](http://go.microsoft.com/fwlink/?Linkid=825073)。

## <a name="add-accounts-to-the-app"></a>將帳戶新增至應用程式
針對每個想要新增至 Microsoft 驗證器應用程式的帳戶，請使用下列其中一個程序。

### <a name="add-a-personal-microsoft-account-to-the-app"></a>將個人的 Microsoft 帳戶新增至應用程式

針對個人的 Microsoft 帳戶 (您用來登入 Outlook.com、Xbox、Skype 等的帳戶)，您只需要登入在 Microsoft 驗證器應用程式中的帳戶。

### <a name="add-a-work-or-school-account-to-the-app-using-the-qr-code-scanner"></a>使用 QR 代碼掃描器將工作或學校帳戶新增至應用程式
1. 移至安全性驗證設定畫面。  如需如何進入此畫面的詳細資訊，請參閱 [變更安全性設定](multi-factor-authentication-end-user-manage-settings.md#where-to-find-the-settings-page)。
2. 核取 **Azure Authenticator 應用程式**旁的方塊，然後選取 [設定]。

    ![安全性驗證設定畫面上的設定按鈕](./media/authenticator-app-how-to/azureauthe.png)

    這會顯示有 QR 代碼的畫面。

    ![提供 QR 代碼的畫面](./media/authenticator-app-how-to/barcode2.png)
3. 開啟 Microsoft 驗證器應用程式。 在 [帳戶] 畫面上，選取 [+]，然後指定您要新增工作帳戶或學校帳戶。
4. 使用相機來掃描 QR 代碼，然後選取 [完成]  關閉 QR 代碼畫面。

    如果相機運作不正常，您可以[手動輸入 QR 代碼和 URL](#add-an-account-to-the-app-manually)。

5. 當應用程式顯示您的帳戶名稱與下面的六位數代碼時，則您已完成。 

    ![帳戶畫面](./media/authenticator-app-how-to/accounts.png)

### <a name="add-an-account-to-the-app-manually"></a>手動新增帳戶至應用程式
1. 移至安全性驗證設定畫面。  如需如何進入此畫面的詳細資訊，請參閱 [變更安全性設定](multi-factor-authentication-end-user-manage-settings.md)。
2. 選取 [設定] 。

    ![安全性驗證設定畫面上的設定按鈕](./media/authenticator-app-how-to/azureauthe.png)

    這會顯示有 QR 代碼的畫面。  記下代碼和 URL。

    ![提供 QR 代碼與 URL 的畫面](./media/authenticator-app-how-to/barcode2.png)
3. 開啟 Microsoft 驗證器應用程式。 在 [帳戶] 畫面上，選取 [+]，然後指定您要新增工作帳戶或學校帳戶。

4. 在掃描器上，選取 [手動輸入代碼] 。

    ![掃描 QR 代碼的畫面](./media/multi-factor-authentication-end-user-first-time/scan2.png)
5. 在應用程式的適當方塊中輸入代碼和 URL，然後選取 [完成]。

    ![輸入代碼和 URL 的畫面](./media/authenticator-app-how-to/manual.png)

6. 當應用程式顯示您的帳戶名稱與下面的六位數代碼時，則您已完成。

    ![帳戶畫面](./media/authenticator-app-how-to/accounts.png)

### <a name="add-an-account-to-the-app-using-touch-id"></a>使用 Touch ID 將帳戶新增至應用程式
在 iOS 上的 Microsoft 驗證器應用程式支援 Touch ID。  Azure Multi-Factor Authentication 可讓組織要求裝置的 PIN 碼。 使用 Touch ID 時，iOS 使用者不需要輸入 PIN 碼。 相反地，他們可以掃描其指紋，並選取 [核准] 。

設定 Touch ID 搭配 Azure 驗證器很簡單。 您可以使用 PIN 碼完成一般驗證挑戰。 如果您的裝置支援 Touch ID，Microsoft 驗證器將會自動為該帳戶進行設定。

![Touch ID 安裝程式的驗證](./media/authenticator-app-how-to/touchid1.png)

往後當您被要求驗證登入時，您只要選取收到的推播通知並且掃描您的指紋即可，不需要輸入您的 PIN。

![推播通知](./media/authenticator-app-how-to/touchid2.png)


