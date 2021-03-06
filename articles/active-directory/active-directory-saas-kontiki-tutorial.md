---
title: "教學課程：Azure Active Directory 與 Kontiki 整合 | Microsoft Docs"
description: "了解如何使用 Kontiki 搭配 Azure Active Directory 來啟用單一登入、自動化佈建和更多功能！"
services: active-directory
author: jeevansd
documentationcenter: na
manager: femila
ms.assetid: 8d5e5413-da4c-40d8-b1d0-f03ecfef030b
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 02/02/2017
ms.author: jeedes
translationtype: Human Translation
ms.sourcegitcommit: 230facec9f8f83e8a94eaaac5a1495195ff45cc7
ms.openlocfilehash: 3af125b186e3905572dd5d621c948a6639f9b023
ms.lasthandoff: 02/17/2017


---

# <a name="tutorial-azure-active-directory-integration-with-kontiki"></a>教學課程：Azure Active Directory 與 Kontiki 整合
本教學課程的目的是要示範 Azure 與 Kontiki 的整合。  
本教學課程中說明的案例假設您已經具有下列項目：

* 有效的 Azure 訂閱
* 啟用 Kontiki 單一登入 (SSO) 的訂用帳戶

完成本教學課程之後，您指派給 Kontiki 的 Azure AD 使用者就能夠單一登入您 Kontiki 公司網站 (服務提供者起始登入) 的應用程式，或是使用 [存取面板簡介](active-directory-saas-access-panel-introduction.md)。

本教學課程中說明的案例由下列建置組塊組成：

* 啟用 Kontiki 的應用程式整合
* 設定單一登入
* 設定使用者佈建
* 指派使用者

![案例](./media/active-directory-saas-kontiki-tutorial/IC790235.png "案例")

## <a name="enable-the-application-integration-for-kontiki"></a>啟用 Kontiki 的應用程式整合
本節的目的是要說明如何啟用 Kontiki 的應用程式整合。

**若要啟用 Kontiki 的應用程式整合，請執行下列步驟：**

1. 在 Azure 傳統入口網站中，按一下左方瀏覽窗格的 [Active Directory] 。
   
   ![Active Directory](./media/active-directory-saas-kontiki-tutorial/IC700993.png "Active Directory")
2. 從 [目錄]  清單中，選取要啟用目錄整合的目錄。
3. 若要開啟應用程式檢視，請在目錄檢視中，按一下頂端功能表中的 [應用程式]  。
   
   ![應用程式](./media/active-directory-saas-kontiki-tutorial/IC700994.png "應用程式")
4. 按一下頁面底部的 [新增]  。
   
   ![新增應用程式](./media/active-directory-saas-kontiki-tutorial/IC749321.png "新增應用程式")
5. 在 [欲執行動作] 對話方塊上，按一下 [從資源庫中新增應用程式]。
   
   ![從資源庫新增應用程式](./media/active-directory-saas-kontiki-tutorial/IC749322.png "從資源庫新增應用程式")
6. 在**搜尋方塊**中，輸入 **Kontiki**。
   
   ![應用程式資源庫](./media/active-directory-saas-kontiki-tutorial/IC790236.png "應用程式資源庫")
7. 在結果窗格中，選取 [Kontiki]，然後按一下 [完成] 以新增應用程式。
   
   ![Kontiki](./media/active-directory-saas-kontiki-tutorial/IC790237.png "Kontiki")
   
## <a name="configure-single-sign-on"></a>設定單一登入

本節的目的是要說明如何依據 SAML 通訊協定來使用同盟，讓使用者能夠用自己的 Azure AD 帳戶在 Kontiki 中進行驗證。

*若要設定單一登入，請執行下列步驟：**

1. 在 Azure 傳統入口網站的 [Kontiki] 應用程式整合頁面上，按一下 [設定單一登入] 來開啟 [設定單一登入] 對話方塊。
   
   ![設定單一登入](./media/active-directory-saas-kontiki-tutorial/IC790238.png "設定單一登入")
2. 在 [您希望使用者如何登入 Kontiki] 頁面上，選取 [Microsoft Azure AD 單一登入]，然後按一下 [下一步]。
   
   ![設定單一登入](./media/active-directory-saas-kontiki-tutorial/IC790239.png "設定單一登入")
3. 在 [設定應用程式 URL] 頁面的 [Kontiki 單一登入 URL] 文字方塊中，輸入使用者登入您的 Kontiki 時所使用的 URL (如：“*https://company.mc.eval.kontiki.com/*")，然後按一下 [下一步]。
   
   ![設定應用程式 URL](./media/active-directory-saas-kontiki-tutorial/IC790240.png "設定應用程式 URL")
4. 在 [設定在 Kontiki 單一登入] 頁面上，按一下 [下載中繼資料]，然後將中繼資料檔儲存在您的電腦中。
   
   ![設定單一登入](./media/active-directory-saas-kontiki-tutorial/IC790241.png "設定單一登入")
5. 將中繼資料檔傳送給 Kontiki 支援小組。
   
   >[!NOTE]
   >單一登入設定必須由 Kontiki 支援小組執行。 設定完成後，您將會收到通知。 
   > 
6. 在 Azure 傳統入口網站上，選取單一登入設定確認，然後按一下 [完成] 來關閉 [設定單一登入] 對話方塊。
   
  ![設定單一登入](./media/active-directory-saas-kontiki-tutorial/IC790242.png "設定單一登入")
   
## <a name="configure-user-provisioning"></a>設定使用者佈建

沒有動作項目可讓您設定 Kontiki 使用者佈建。 當指派的使用者嘗試使用存取面板登入 Kontiki 時，Kontiki 會檢查使用者是否存在。  

>!NOTE] 如果尚無可用的使用者帳戶，Kontiki 會自動予以建立。
>

## <a name="assign-users"></a>指派使用者
若要測試您的組態，則需指派您所允許使用您應用程式的 Azure AD 使用者，藉此授予其存取組態的權限。

**若要將使用者指派給 Kontiki，請執行下列步驟：**

1. 在 Azure 傳統入口網站中建立測試帳戶。
2. 在 [Kontiki] 應用程式整合頁面中，按一下 [指派使用者]。
   
   ![指派使用者](./media/active-directory-saas-kontiki-tutorial/IC790243.png "指派使用者")
3. 選取測試使用者，按一下 [指派]，然後按一下 [是] 以確認指派。
   
   ![是](./media/active-directory-saas-kontiki-tutorial/IC767830.png "是")

如果要測試您的單一登入設定，請開啟存取面板。 如需 [存取面板] 的詳細資訊，請參閱 [存取面板簡介](active-directory-saas-access-panel-introduction.md)。


