---
title: "管理對容器與 Blob 的匿名讀取權限 | Microsoft Docs"
description: "了解如何讓容器與 Blob 可供匿名存取，以及如何以程式設計方式存取。"
services: storage
documentationcenter: 
author: mmacy
manager: timlt
editor: tysonn
ms.assetid: a2cffee6-3224-4f2a-8183-66ca23b2d2d7
ms.service: storage
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/08/2016
ms.author: marsma
translationtype: Human Translation
ms.sourcegitcommit: 931503f56b32ce9d1b11283dff7224d7e2f015ae
ms.openlocfilehash: 4fe41c3aabf5e6d9ae899cea0b9f9b6c9c305cf0


---
# <a name="manage-anonymous-read-access-to-containers-and-blobs"></a>管理對容器與 Blob 的匿名讀取權限。
## <a name="overview"></a>Overview
根據預設，只有儲存體帳戶的擁有者可以存取該帳戶內的儲存體資源。 您可以僅針對 Blob 儲存體設定容器的權限，以允許對容器和 Blob 的匿名讀取權限，讓您能夠授予這些資源的存取權，而無需分享您的帳戶金鑰。

匿名存取適用於您想要某些 Blob 永遠可供匿名讀取存取的狀況。 如需更精密的控制，您可以建立共用存取簽章，讓您能夠在指定的時間間隔內，使用不同的權限來委派受限制的存取權。 如需建立共用存取簽章的詳細資訊，請參閱 [使用共用存取簽章 (SAS)](storage-dotnet-shared-access-signature-part-1.md)。

## <a name="grant-anonymous-users-permissions-to-containers-and-blobs"></a>授與容器和 Blob 的匿名使用者權限
根據預設，可能只有儲存體帳戶的擁有者能存取容器及其內部的任何 Blob。 若要為匿名使用者授與容器及其 Blob 的讀取權限，您可以設定容器權限以允許公用存取。 匿名使用者可以讀取可公開存取之容器內的 Blob，而不需驗證要求。

容器會提供下列選項來管理容器存取：

* **完整的公開讀取權限：** 可以透過匿名要求讀取容器和 Blob 資料。 用戶端可以透過匿名要求列舉容器內的 Blob，但無法列舉儲存體帳戶內的容器。
* **僅對 Blob 有公開讀取權限：** 您可以透過匿名要求讀取此容器內的 Blob 資料，但您無法使用容器資料。 用戶端無法透過匿名要求列舉容器內的 Blob。
* **沒有公開讀取權限：** 只有帳戶擁有者可以讀取容器和 Blob 資料。

您可以利用下列方式設定容器權限：

* 從 [Azure 入口網站](https://portal.azure.com)。
* 使用儲存體用戶端程式庫或 REST API 以程式設計方式設定。
* 使用 PowerShell 設定。 若要深入了解如何從 Azure PowerShell 設定容器權限，請參閱 [使用 Azure PowerShell 與 Azure 儲存體](storage-powershell-guide-full.md#how-to-manage-azure-blobs)。

### <a name="setting-container-permissions-from-the-azure-portal"></a>從 Azure 入口網站設定容器權限
若要從 [Azure 入口網站](https://portal.azure.com)設定容器權限，請遵循下列步驟：

1. 瀏覽至儲存體帳戶的儀表板。
2. 從清單中選取容器名稱。 按一下名稱將會公開所選容器中的 Blob
3. 從工具列選取 [存取原則]  。
4. 在 [存取類型]  欄位中，選取您所需的權限層級，如以下螢幕擷取畫面所示。

    ![編輯容器中繼資料對話方塊](./media/storage-manage-access-to-resources/storage-manage-access-to-resources-0.png)

### <a name="setting-container-permissions-programmatically-using-net"></a>使用 .NET 以程式設計方式設定容器權限
若要使用 .NET 用戶端程式庫來設定容器的權限，請先呼叫 **GetPermissions** 方法來擷取容器的現有權限。 接著為由 **GetPermissions** 方法傳回的 **BlobContainerPermissions** 物件設定 **PublicAccess** 屬性。 最後，使用更新的權限呼叫 **SetPermissions** 方法。

下列範例將容器的權限設為完整公用讀取權限。 若只要將 Blob 的權限設為公用讀取權限，請將 **PublicAccess** 屬性設為 **BlobContainerPublicAccessType.Blob**。 若要移除匿名使用者的所有權限，請將屬性設為 **BlobContainerPublicAccessType.Off**。

```csharp
public static void SetPublicContainerPermissions(CloudBlobContainer container)
{
    BlobContainerPermissions permissions = container.GetPermissions();
    permissions.PublicAccess = BlobContainerPublicAccessType.Container;
    container.SetPermissions(permissions);
}
```

## <a name="access-containers-and-blobs-anonymously"></a>匿名存取容器與 Blob
匿名存取容器與 Blob 的用戶端可使用不需要認證的建構函式。 以下範例顯示可匿名參考 Blob 服務資源的不同方法。

### <a name="create-an-anonymous-client-object"></a>建立匿名用戶端物件
您可以為帳戶提供 Blob 服務端點，來建立用於匿名存取的新服務用戶端物件。 不同，您也必須知道可供匿名存取的帳戶中的容器名稱。

```csharp
public static void CreateAnonymousBlobClient()
{
    // Create the client object using the Blob service endpoint.
    CloudBlobClient blobClient = new CloudBlobClient(new Uri(@"https://storagesample.blob.core.windows.net"));

    // Get a reference to a container that's available for anonymous access.
    CloudBlobContainer container = blobClient.GetContainerReference("sample-container");

    // Read the container's properties. Note this is only possible when the container supports full public read access.
    container.FetchAttributes();
    Console.WriteLine(container.Properties.LastModified);
    Console.WriteLine(container.Properties.ETag);
}
```

### <a name="reference-a-container-anonymously"></a>匿名參考容器
如果您有可供匿名使用之容器的 URL，您可以使用該 URL 直接參考容器。

```csharp
public static void ListBlobsAnonymously()
{
    // Get a reference to a container that's available for anonymous access.
    CloudBlobContainer container = new CloudBlobContainer(new Uri(@"https://storagesample.blob.core.windows.net/sample-container"));

    // List blobs in the container.
    foreach (IListBlobItem blobItem in container.ListBlobs())
    {
        Console.WriteLine(blobItem.Uri);
    }
}
```


### <a name="reference-a-blob-anonymously"></a>匿名參考 Blob
如果您有可供匿名存取之 Blob 的 URL，您可以使用該 URL 直接參考 Blob：

```csharp
public static void DownloadBlobAnonymously()
{
    CloudBlockBlob blob = new CloudBlockBlob(new Uri(@"https://storagesample.blob.core.windows.net/sample-container/logfile.txt"));
    blob.DownloadToFile(@"C:\Temp\logfile.txt", System.IO.FileMode.Create);
}
```

## <a name="features-available-to-anonymous-users"></a>匿名使用者可使用的功能
下表顯示當容器的 ACL 設為允許公用存取時，匿名使用者可能會呼叫哪些作業。

| REST 作業 | 具有完整公開讀取權限 | 僅對 Blob 有公開讀取權限 |
| --- | --- | --- |
| 列出容器 |只有擁有者 |只有擁有者 |
| 建立容器 |只有擁有者 |只有擁有者 |
| 取得容器屬性 |全部 |只有擁有者 |
| 取得容器中繼資料 |全部 |只有擁有者 |
| 設定容器中繼資料 |只有擁有者 |只有擁有者 |
| 取得容器 ACL |只有擁有者 |只有擁有者 |
| 設定容器 ACL |只有擁有者 |只有擁有者 |
| 刪除容器 |只有擁有者 |只有擁有者 |
| 列出 Blob |全部 |只有擁有者 |
| 放置 Blob |只有擁有者 |只有擁有者 |
| 取得 Blob |全部 |全部 |
| 取得 Blob 屬性 |全部 |全部 |
| 設定 Blob 屬性 |只有擁有者 |只有擁有者 |
| 取得 Blob 中繼資料 |全部 |全部 |
| 設定 Blob 中繼資料 |只有擁有者 |只有擁有者 |
| 放置區塊 |只有擁有者 |只有擁有者 |
| 取得區塊清單 (僅限認可區塊) |全部 |全部 |
| 取得區塊清單 (僅限未認可的區塊或所有區塊) |只有擁有者 |只有擁有者 |
| 放置區塊清單 |只有擁有者 |只有擁有者 |
| 刪除 Blob |只有擁有者 |只有擁有者 |
| 複製 Blob |只有擁有者 |只有擁有者 |
| 快照 Blob |只有擁有者 |只有擁有者 |
| 租用 Blob |只有擁有者 |只有擁有者 |
| 放置頁面 |只有擁有者 |只有擁有者 |
| 取得頁面範圍 |全部 |全部 |
| 附加 Blob |只有擁有者 |只有擁有者 |

## <a name="see-also"></a>另請參閱
* [Azure 儲存體服務的驗證](https://msdn.microsoft.com/library/azure/dd179428.aspx)
* [使用共用存取簽章 (SAS)](storage-dotnet-shared-access-signature-part-1.md)
* [使用共用存取簽章來委派存取權](https://msdn.microsoft.com/library/azure/ee395415.aspx)




<!--HONumber=Dec16_HO2-->


