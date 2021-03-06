---
title: "教學課程：Azure Active Directory 與 Amazon Web Services (AWS) 整合 | Microsoft Docs"
description: "了解如何搭配 Azure Active Directory 使用 Amazon Web Service (AWS) 來啟用單一登入、自動化佈建和更多功能！"
services: active-directory
documentationcenter: 
author: jeevansd
manager: femila
editor: 
ms.assetid: 7561c20b-2325-4d97-887f-693aa383c7be
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 3/10/2017
ms.author: jeedes
translationtype: Human Translation
ms.sourcegitcommit: 07635b0eb4650f0c30898ea1600697dacb33477c
ms.openlocfilehash: 8ea8de916a36a1b5274f42513b0446f99707c179
ms.lasthandoff: 03/28/2017


---
# <a name="tutorial-azure-active-directory-integration-with-amazon-web-services-aws"></a>教學課程：Azure Active Directory 與 Amazon Web Services (AWS) 整合
本教學課程旨在說明如何整合 Amazon Web Services (AWS) 與 Azure Active Directory (Azure AD)。  

Amazon Web Services (AWS) 與 Azure AD 整合提供下列優點： 

* 您可以在 Azure AD 中控制可存取 Amazon Web Services (AWS) 的人員 
* 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Amazon Web Services (AWS) 單一登入 (SSO)
* 您可以在 Azure 傳統入口網站中集中管理您的帳戶

若您想了解 SaaS app 與 Azure AD 整合的更多詳細資訊，請參閱 [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](active-directory-appssoaccess-whatis.md)。

## <a name="prerequisites"></a>必要條件
若要設定 Azure AD 與 Amazon Web Services (AWS) 整合，您需要下列項目：

* Azure AD 訂用帳戶
* 已啟用 Amazon Web Services (AWS) SSO 的訂用帳戶

>[!NOTE]
>若要測試本教學課程中的步驟，我們不建議使用生產環境。 
> 

若要測試本教學課程中的步驟，您應該遵循這些建議：

* 除非必要，否則您不應使用生產環境，。
* 如果您沒有 Azure AD 試用環境，您可以取得[一個月試用](https://azure.microsoft.com/pricing/free-trial/)。 

## <a name="scenario-description"></a>案例描述
此教學課程的目標是讓您在測試環境中測試 Azure AD SSO。  

本教學課程中說明的案例由三個主要建置組塊組成：

1. 從資源庫新增 Amazon Web Services (AWS) 
2. 設定並測試 Azure AD SSO

## <a name="add-amazon-web-services-aws-from-the-gallery"></a>從資源庫新增 Amazon Web Services (AWS)
若要設定 Amazon Web Services (AWS) 與 Azure AD 整合，您需要從資源庫將 Amazon Web Services (AWS) 新增到受管理的 SaaS App 清單。

**若要從資源庫新增 Amazon Web Services (AWS)，請執行下列步驟：**

1. 在 **Azure 傳統入口網站**中，按一下左方瀏覽窗格的 [Active Directory]。 
   
    ![Active Directory][1] 

2. 從 [目錄]  清單中，選取要啟用目錄整合的目錄。

3. 若要開啟應用程式檢視，請在目錄檢視中，按一下頂端功能表中的 [應用程式]  。 
   
    ![應用程式][2]

4. 按一下頁面底部的 [新增]  。 
   
    ![應用程式][3]

5. 在 [欲執行動作] 對話方塊上，按一下 [從資源庫中新增應用程式]。 
   
    ![應用程式][4]

6. 在搜尋方塊中，輸入 **Amazon Web Services (AWS)**。
   
    ![應用程式][5]

7. 在結果窗格中，選取 [Amazon Web Services (AWS)]，然後按一下 [完成] 以新增應用程式。
   
    ![應用程式][6]

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入
本節目標是說明如何以名為 "Britta Simon" 的測試使用者為基礎，使用 Amazon Web Services (AWS) 來設定及測試 Azure AD SSO。

若要讓 SSO 運作，Azure AD 必須知道 Amazon Web Services (AWS) 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 Amazon Web Services (AWS) 中的相關使用者之間建立連結關聯性。  

建立此連結關聯性的方法是將 Azure AD 中**使用者名稱**的值指定為 Amazon Web Services (AWS) 中 **Username** 的值。

若要使用 Amazon Web Services (AWS) 來設定並測試 Azure AD SSO，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-single-sign-on)** - 讓您的使用者能夠使用此功能。
2. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
3. **[建立 Amazon Web Services (AWS) 測試使用者](#creating-a-halogen-software-test-user)** - 使 Amazon Web Services (AWS) 中對應的 Britta Simon 連結到她在 Azure AD 中的代表項目。
4. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. **[測試單一登入](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入
本節目標是在 Azure 傳統入口網站啟用 Azure AD SSO，並在您的 Amazon Web Services (AWS) 應用程式中設定單一登入。  

您的 Amazon Web Services (AWS) 應用程式需要特定格式的 SAML 判斷提示，這需要您將自訂屬性對應新增到您的 **saml token 屬性** 設定。 

以下螢幕擷取畫面顯示上述的範例。

![設定單一登入][27]

**若要使用 Amazon Web Services (AWS) 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 傳統入口網站的 [Amazon Web Services (AWS)] 應用程式整合頁面上，按一下 [設定單一登入] 來開啟 [設定單一登入] 對話方塊。
   
    ![設定單一登入][7]

2. 在 [您希望使用者如何登入 Amazon Web Services (AWS)] 頁面上，選取 [Azure AD 單一登入]，然後按 [下一步]。
   
    ![設定單一登入][8]

3. 在 [設定應用程式設定] 對話方塊頁面上，按 [下一步]。 
   
    ![設定 App 設定][9]

4. 在 [設定在 Amazon Web Services (AWS) 單一登入] 頁面上，按 [下載中繼資料]，然後將中繼資料檔儲存在您的本機電腦中。
   
    ![設定單一登入][10]

5. 在不同的瀏覽器視窗中，以系統管理員身分登入您的 Amazon Web Services (AWS) 公司網站。

6. 按一下 [主控台首頁] 。
   
    ![設定單一登入][11]

7. 按一下 [身分識別與存取管理] 。 
   
    ![設定單一登入][12]

8. 按一下 [識別提供者]，然後按一下 [建立提供者]。 
   
    ![設定單一登入][13]

9. 在 [設定提供者]  對話方塊頁面上，執行下列步驟： 
   
    ![設定單一登入][14]   
  1. 針對 [提供者類型]，選取 [SAML]。
  2. 在 [提供者名稱] 文字方塊中，輸入提供者名稱 (例如：*WAAD*)。
  3. 若要上傳您下載的中繼資料檔，請按一下 [選擇檔案]。
  4. 按一下頁面底部的 [新增] 。

10. 在 [驗證提供者資訊] 對話方塊頁面上，按一下 [建立]。 
    
    ![設定單一登入][15]

11. 按一下 [角色]，然後按一下 [建立新角色]。 
    
    ![設定單一登入][16]

12. 在 [設定角色名稱]  對話方塊上，執行下列步驟： 
    
    ![設定單一登入][17] 
  1. 在 [角色名稱] 文字方塊中，輸入角色名稱 (例如：*TestUser*)。 
  2. 按一下頁面底部的 [新增] 。

13. 在 [選取角色類型]  對話方塊上，執行下列步驟： 
    
    ![設定單一登入][18] 
  1. 選取 [識別提供者存取的角色]。 
  2. 在 [授與 SAML 提供者的 Web 單一登入 (WebSSO) 存取] 區段中，按一下 [選取]。

14. 在 [建立信任]  對話方塊上，執行下列步驟：  
    
    ![設定單一登入][19] 
  1. 針對 SAML 提供者，請選取您先前建立的 SAML 提供者 (例如：*WAAD*)   
  2. 按一下頁面底部的 [新增] 。

15. 在 [確認角色信任] 對話方塊上，按 [下一步]。 
    
    ![設定單一登入][32]

16. 在 [附加原則] 對話方塊上，按 [下一步]。  
    
    ![設定單一登入][33]

17. 在 [檢閱]  對話方塊上，執行下列步驟：   
    
    ![設定單一登入][34] 
  1. 複製**角色 ARN** 值。  
  2. 複製**信任實體** ARN 值。 
  3. 按一下 [建立角色]。 

18. 在 Azure 傳統入口網站中，選取單一登入設定確認，然後按 [下一步] 。
    
    ![何謂 Azure AD Connect][20]

19. 在 [單一登入確認] 頁面上，按一下 [完成] 以關閉 [設定單一登入] 對話方塊。
    
    ![何謂 Azure AD Connect][22]

20. 在頂端的功能表中，按一下 [屬性] **屬性** to open the **SAML Token 屬性** 對話方塊。 
    
    ![設定單一登入][21]

21. 按一下 [加入使用者屬性] 。 
    
    ![設定單一登入][23]

22. 在 [加入使用者屬性] 對話方塊上，執行下列步驟。 
    
    ![設定單一登入][24]  
  1. 在 [屬性名稱] 文字方塊中，輸入 **https://aws.amazon.com/SAML/Attributes/Role**。  
  2. 在 [屬性值] 文字方塊中，輸入 [角色 ARN 值], [受信任實體 ARN 值]。
    
     >[!TIP]
     >建立您的角色後，會有一些您已從 [檢閱] 對話方塊中複製的值。 
     > 
     
  3. 按一下 [完成] 以關閉 [加入使用者屬性] 對話方塊。

23. 按一下 [加入使用者屬性] 。 
    
    ![設定單一登入][23]

24. 在 [加入使用者屬性] 對話方塊上，執行後續步驟，然後按一下 [套用變更]。 
    
    ![設定單一登入][25]
 1. 在 [屬性名稱] 文字方塊中，輸入 **https://aws.amazon.com/SAML/Attributes/RoleSessionName**。
 2. 在 [屬性值] 文字方塊中，輸入或從下拉式清單中選取 **user.userprincipalname**。

     ![設定單一登入][35]
 3. 按一下 [完成] 以關閉 [加入使用者屬性] 對話方塊。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 傳統入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 測試使用者](./media/active-directory-saas-amazon-web-service-tutorial/create_aaduser_01.png)

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 傳統入口網站**中，按一下左方瀏覽窗格的 [Active Directory]。
   
    ![建立 Azure AD 測試使用者](./media/active-directory-saas-amazon-web-service-tutorial/create_aaduser_02.png) 

2. 從 [目錄]  清單中，選取要啟用目錄整合的目錄。

3. 若要顯示使用者清單，請按一下頂端功能表的 [使用者] 。
   
    ![建立 Azure AD 測試使用者](./media/active-directory-saas-amazon-web-service-tutorial/create_aaduser_03.png) 

4. 若要開啟 [新增使用者] 對話方塊，請按一下底部工具列上的 [新增使用者]。 
   
    ![建立 Azure AD 測試使用者](./media/active-directory-saas-amazon-web-service-tutorial/create_aaduser_04.png) 

5. 在 [告訴我們這位使用者]  對話方塊頁面上，執行下列步驟： 
   
    ![建立 Azure AD 測試使用者](./media/active-directory-saas-amazon-web-service-tutorial/create_aaduser_05.png) 
 1. 針對 [使用者類型]，選取 [您組織中的新使用者]。
 2. 在 [使用者名稱] 文字方塊中，輸入 **BrittaSimon**。
 3. 按 [下一步]。

6. 在 [使用者設定檔]  對話方塊頁面上，執行下列步驟： 
   
    ![建立 Azure AD 測試使用者](./media/active-directory-saas-amazon-web-service-tutorial/create_aaduser_06.png)  
 1. 在 [名字] 文字方塊中，輸入 **Britta**。   
 2. 在 [姓氏] 文字方塊中輸入 **Simon**。 
 3. 在 [顯示名稱] 文字方塊中，輸入 **Britta Simon**。 
 4. 在 [角色] 清單中選取 [使用者]。 
 5. 按 [下一步] 。

7. 在 [取得暫時密碼] 對話方塊頁面上，按一下 [建立]。
   
    ![建立 Azure AD 測試使用者](./media/active-directory-saas-amazon-web-service-tutorial/create_aaduser_07.png) 

8. 在 [取得暫時密碼]  對話方塊頁面上，執行下列步驟：
   
    ![建立 Azure AD 測試使用者](./media/active-directory-saas-amazon-web-service-tutorial/create_aaduser_08.png)  
 1. 記下 [新密碼] 的值。  
 2. 按一下 [完成]。   

### <a name="create-a-amazon-web-services-aws-test-user"></a>建立 Amazon Web Services (AWS) 測試使用者
本節目標是在 Amazon Web Services (AWS) 中建立名為 Britta Simon 的使用者。

**若要在 Amazon Web Services (AWS) 中建立名為 Britta Simon 的使用者，請執行以下步驟：**

1. 以系統管理員身分登入您的 **Amazon Web Services (AWS)** 公司網站。

2. 按一下 [主控台首頁]  圖示。 
   
    ![設定單一登入][11]

3. 按一下 [身分識別與存取管理]。 
   
    ![設定單一登入][28]

4. 在儀表板中，按一下 [使用者]，然後按一下 [建立新使用者]。 
   
    ![設定單一登入][29]

5. 在 [建立使用者] 對話方塊中，執行下列步驟： 
   
    ![設定單一登入][30]   
 1. 在 [輸入使用者名稱] 文字方塊中，輸入 Brita Simon 在 Azure AD 中的使用者名稱 (userprincipalname)。 
 2. 按一下 [建立] 。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者
本節目標是授與 Britta Simon 對 Amazon Web Services (AWS) 的存取權，讓她能夠使用 Azure SSO。

![指派使用者][31]

**若要將 Britta Simon 指派給 Amazon Web Services (AWS)，請執行下列步驟：**

1. 在 Azure 傳統入口網站中，若要開啟應用程式檢視，請在目錄檢視中，按一下頂端功能表中的 [應用程式]  。
   
    ![指派使用者][26]

2. 在應用程式清單中，選取 [Amazon Web Services (AWS)] 。
   
    ![指派使用者][27]

3. 在頂端的功能表中，按一下 [使用者] 。
   
    ![指派使用者][25]

4. 在 [使用者] 清單中，選取 [Britta Simon] 。

5. 在底部的工具列中，按一下 [指派] 。
   
    ![指派使用者][29]

### <a name="test-single-sign-on"></a>測試單一登入
本節的目標是要使用「存取面板」來測試您的 Azure AD SSO 組態。  

當您在存取面板中按一下 [Amazon Web Services (AWS)] 圖格時，應該會自動登入您的 Amazon Web 服務 (AWS) 應用程式。

## <a name="additional-resources"></a>其他資源
* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](active-directory-saas-tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](active-directory-appssoaccess-whatis.md)

<!--Image references-->
[1]: ./media/active-directory-saas-amazon-web-service-tutorial/tutorial_general_01.png
[2]: ./media/active-directory-saas-amazon-web-service-tutorial/tutorial_general_02.png
[3]: ./media/active-directory-saas-amazon-web-service-tutorial/tutorial_general_03.png
[4]: ./media/active-directory-saas-amazon-web-service-tutorial/tutorial_general_04.png
[5]: ./media/active-directory-saas-amazon-web-service-tutorial/ic795019.png
[6]: ./media/active-directory-saas-amazon-web-service-tutorial/ic795020.png
[7]: ./media/active-directory-saas-amazon-web-service-tutorial/ic795027.png
[8]: ./media/active-directory-saas-amazon-web-service-tutorial/ic795028.png
[9]: ./media/active-directory-saas-amazon-web-service-tutorial/capture23.png
[10]: ./media/active-directory-saas-amazon-web-service-tutorial/capture24.png
[11]: ./media/active-directory-saas-amazon-web-service-tutorial/ic795031.png
[12]: ./media/active-directory-saas-amazon-web-service-tutorial/ic795032.png
[13]: ./media/active-directory-saas-amazon-web-service-tutorial/ic795033.png
[14]: ./media/active-directory-saas-amazon-web-service-tutorial/ic795034.png
[15]: ./media/active-directory-saas-amazon-web-service-tutorial/ic795035.png
[16]: ./media/active-directory-saas-amazon-web-service-tutorial/ic795022.png
[17]: ./media/active-directory-saas-amazon-web-service-tutorial/ic795023.png
[18]: ./media/active-directory-saas-amazon-web-service-tutorial/ic795024.png
[19]: ./media/active-directory-saas-amazon-web-service-tutorial/ic795025.png
[20]: ./media/active-directory-saas-amazon-web-service-tutorial/ic7950351.png
[21]: ./media/active-directory-saas-amazon-web-service-tutorial/tutorial_general_80.png
[22]: ./media/active-directory-saas-amazon-web-service-tutorial/ic7950352.png
[23]: ./media/active-directory-saas-amazon-web-service-tutorial/tutorial_general_81.png
[24]: ./media/active-directory-saas-amazon-web-service-tutorial/ic7950353.png
[25]: ./media/active-directory-saas-amazon-web-service-tutorial/tutorial_general_15.png
[26]: ./media/active-directory-saas-amazon-web-service-tutorial/tutorial_general_18.png
[27]: ./media/active-directory-saas-amazon-web-service-tutorial/ic7950357.png
[28]: ./media/active-directory-saas-amazon-web-service-tutorial/ic7950321.png
[29]: ./media/active-directory-saas-amazon-web-service-tutorial/tutorial_general_16.png
[30]: ./media/active-directory-saas-amazon-web-service-tutorial/ic795038.png
[31]: ./media/active-directory-saas-amazon-web-service-tutorial/tutorial_general_17.png
[32]: ./media/active-directory-saas-amazon-web-service-tutorial/ic7950251.png
[33]: ./media/active-directory-saas-amazon-web-service-tutorial/ic7950252.png
[34]: ./media/active-directory-saas-amazon-web-service-tutorial/ic7950253.png
[35]: ./media/active-directory-saas-amazon-web-service-tutorial/user_attributes_01.png























