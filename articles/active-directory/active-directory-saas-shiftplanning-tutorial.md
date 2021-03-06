---
title: "教學課程：Azure Active Directory 與 ShiftPlanning 整合 | Microsoft Docs"
description: "了解如何使用 ShiftPlanning 搭配 Azure Active Directory 來啟用單一登入、自動佈建和更多功能！"
services: active-directory
author: jeevansd
documentationcenter: na
manager: femila
ms.assetid: 6aa771e9-31c6-48d1-8dde-024bebc06943
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 01/03/2017
ms.author: jeedes
translationtype: Human Translation
ms.sourcegitcommit: 9a653ac435198e89a527070a0174a1adaf830dc3
ms.openlocfilehash: 6104bd7e22d855c4c8737ef1080bfd445b29eafb


---
# <a name="tutorial-azure-active-directory-integration-with-shiftplanning"></a>教學課程：Azure Active Directory 與 ShiftPlanning 整合
本教學課程的目的是要示範 Azure 與 ShiftPlanning 的整合。  
本教學課程中說明的案例假設您已經具有下列項目：

* 有效的 Azure 訂用帳戶
* 已啟用 ShiftPlanning 單一登入的訂用帳戶

完成本教學課程之後，您指派給 ShiftPlanning 的 Azure AD 使用者就能夠從您的 ShiftPlanning 公司網站 (服務提供者起始登入)，或使用 [存取面板](active-directory-saas-access-panel-introduction.md)來單一登入應用程式。

本教學課程中說明的案例由下列建置組塊組成：

1. 啟用 ShiftPlanning 的應用程式整合
2. 設定單一登入
3. 設定使用者佈建
4. 指派使用者

![案例](./media/active-directory-saas-shiftplanning-tutorial/IC786612.png "案例")

## <a name="enabling-the-application-integration-for-shiftplanning"></a>啟用 ShiftPlanning 的應用程式整合
本節的目的是要說明如何啟用 ShiftPlanning 的應用程式整合。

### <a name="to-enable-the-application-integration-for-shiftplanning-perform-the-following-steps"></a>若要啟用 ShiftPlanning 的應用程式整合，請執行下列步驟：
1. 在 Azure 傳統入口網站中，按一下左方瀏覽窗格的 [Active Directory] 。
   
    ![Active Directory](./media/active-directory-saas-shiftplanning-tutorial/IC700993.png "Active Directory")

2. 從 [目錄]  清單中，選取要啟用目錄整合的目錄。

3. 若要開啟應用程式檢視，請在目錄檢視中，按一下頂端功能表中的 [應用程式]  。
   
    ![應用程式](./media/active-directory-saas-shiftplanning-tutorial/IC700994.png "應用程式")

4. 按一下頁面底部的 [新增]  。
   
    ![新增應用程式](./media/active-directory-saas-shiftplanning-tutorial/IC749321.png "新增應用程式")

5. 在 [欲執行動作] 對話方塊上，按一下 [從資源庫中新增應用程式]。
   
    ![從資源庫新增應用程式](./media/active-directory-saas-shiftplanning-tutorial/IC749322.png "從資源庫新增應用程式")

6. 在 [搜尋方塊] 中，輸入 **ShiftPlanning**。
   
    ![應用程式資源庫](./media/active-directory-saas-shiftplanning-tutorial/IC786613.png "應用程式資源庫")

7. 在結果窗格中，選取 [ShiftPlanning]，然後按一下 [完成] 來新增應用程式。
   
    ![ShiftPlanning](./media/active-directory-saas-shiftplanning-tutorial/IC786614.png "ShiftPlanning")
   
## <a name="configuring-single-sign-on"></a>設定單一登入

本節的目的是要說明如何依據 SAML 通訊協定來使用同盟，讓使用者能夠用自己在 Azure AD 中的帳戶在 ShiftPlanning 中進行驗證。  
在此程序中，您必須建立 Base-64 編碼的憑證檔案。  
如果您不熟悉這個程序，請參閱 [如何將二進位憑證轉換成文字檔](http://youtu.be/PlgrzUZ-Y1o)

### <a name="to-configure-single-sign-on-perform-the-following-steps"></a>若要設定單一登入，請執行下列步驟：
1. 在 Azure 傳統入口網站中的 [ShiftPlanning] 應用程式整合頁面上，按一下 [設定單一登入] 來開啟 [設定單一登入] 對話方塊。
   
    ![設定單一登入](./media/active-directory-saas-shiftplanning-tutorial/IC786615.png "設定單一登入")

2. 在 [要如何讓使用者登入 ShiftPlanning] 頁面上，選取 [Microsoft Azure AD 單一登入]，然後按 [下一步]。
   
    ![設定單一登入](./media/active-directory-saas-shiftplanning-tutorial/IC786616.png "設定單一登入")

3. 在 [設定應用程式 URL] 頁面的 [ShiftPlanning 登入 URL] 文字方塊中，使用下列模式輸入您的 URL："https://company.shiftplanning.com/includes/saml/"，然後按 [下一步]。
   
    ![設定應用程式 URL](./media/active-directory-saas-shiftplanning-tutorial/IC786617.png "設定應用程式 URL")

4. 在 [設定在 ShiftPlanning 單一登入] 頁面上，按一下 [下載憑證] 來下載您的憑證，然後將憑證檔案儲存在您的電腦上。
   
    ![設定單一登入](./media/active-directory-saas-shiftplanning-tutorial/IC786618.png "設定單一登入")

5. 在不同的 Web 瀏覽器視窗中，以系統管理員身分登入您的 **ShiftPlanning** 公司網站。
6. 在頂端的功能表中，按一下 [系統管理員] 。
   
    ![管理](./media/active-directory-saas-shiftplanning-tutorial/IC786619.png "管理")

7. 在 [整合] 下方，按一下 [單一登入]。
   
    ![單一登入](./media/active-directory-saas-shiftplanning-tutorial/IC786620.png "單一登入")

8. 在 [單一登入]  區段中，執行下列步驟：
   
    ![單一登入](./media/active-directory-saas-shiftplanning-tutorial/IC786905.png "單一登入")
   
    1. 選取 [已啟用 SAML] 。

    2. 選取 [允許密碼登入] 

    3. 在 Azure 傳統入口網站中的 [設定在 ShiftPlanning 單一登入] 對話頁面上，複製 [遠端登入 URL] 值，然後將它貼至 [SAML 簽發者 URL] 文字方塊中。

    4. 在 Azure 傳統入口網站中的 [設定在 ShiftPlanning 單一登入] 對話頁面上，複製 [遠端登出 URL] 值，然後將它貼至 [遠端登出 URL] 文字方塊中。

    5. 從您下載的憑證建立「Base-64 編碼」  檔案。  
       
    > [!TIP]
    > 如需詳細資訊，請參閱 [如何將二進位憑證轉換成文字檔](http://youtu.be/PlgrzUZ-Y1o)
    > 
    > 

    6. 在記事本中開啟您的 base-64 編碼的憑證，將其內容複製到您的剪貼簿，然後貼到 [X.509 憑證]  文字方塊中

    7. 按一下 [儲存設定] 。

9. 在 Azure 傳統入口網站上，選取單一登入設定確認，然後按一下 [完成] 來關閉 [設定單一登入] 對話方塊。
   
    ![設定單一登入](./media/active-directory-saas-shiftplanning-tutorial/IC786621.png "設定單一登入")
   
## <a name="configuring-user-provisioning"></a>設定使用者佈建

若要讓 Azure AD 使用者可以登入 ShiftPlanning，則必須將他們佈建至 ShiftPlanning。  
ShiftPlanning 需以手動的方式佈建。

### <a name="to-provision-a-user-accounts-perform-the-following-steps"></a>若要佈建使用者帳戶，請執行下列步驟：
1. 以系統管理員身分登入您的 **ShiftPlanning** 公司網站。

2. 按一下 Admin 。
   
    ![管理](./media/active-directory-saas-shiftplanning-tutorial/IC786619.png "管理")

3. 按一下 [職員] 。
   
    ![職員](./media/active-directory-saas-shiftplanning-tutorial/IC786623.png "職員")

4. 在 [動作] 下方，按一下 [新增員工]。
   
    ![新增員工](./media/active-directory-saas-shiftplanning-tutorial/IC786624.png "新增員工")

5. 在 [新增員工]  區段中，執行下列步驟：
   
    ![儲存員工](./media/active-directory-saas-shiftplanning-tutorial/IC786625.png "儲存員工")
   
    1. 在相關的文字方塊中，輸入您想要佈建之有效 AAD 帳戶的 [名字]、[姓氏] 和 [電子郵件]。

    2. 按一下 [儲存員工] 。

> [!NOTE]
> 您可以使用 ShiftPlanning 提供的任何其他 ShiftPlanning 使用者帳戶建立工具或 API 來佈建 AAD 使用者帳戶。
> 
> 

## <a name="assigning-users"></a>指派使用者
若要測試您的組態，則需指派您所允許使用您應用程式的 Azure AD 使用者，藉此授予其存取組態的權限。

### <a name="to-assign-users-to-shiftplanning-perform-the-following-steps"></a>若要將使用者指派給 ShiftPlanning，請執行下列步驟：
1. 在 Azure 傳統入口網站中建立測試帳戶。

2. 在 [ShiftPlanning] 應用程式整合頁面上，按一下 [指派使用者]。
   
    ![指派使用者](./media/active-directory-saas-shiftplanning-tutorial/IC786626.png "指派使用者")

3. 選取測試使用者，按一下 [指派]，然後按一下 [是] 以確認指派。
   
    ![是](./media/active-directory-saas-shiftplanning-tutorial/IC767830.png "是")
 
如果要測試您的單一登入設定，請開啟存取面板。 如需 [存取面板] 的詳細資訊，請參閱 [存取面板簡介](active-directory-saas-access-panel-introduction.md)。




<!--HONumber=Jan17_HO1-->


