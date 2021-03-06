---
title: "將 Azure 中 Linux VM 上的磁碟加密 | Microsoft Docs"
description: "如何使用 Azure CLI 2.0 將 Linux VM 上的虛擬磁碟加密以增強安全性"
services: virtual-machines-linux
documentationcenter: 
author: iainfoulds
manager: timlt
editor: 
tags: azure-resource-manager
ms.assetid: 2a23b6fa-6941-4998-9804-8efe93b647b3
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 03/23/2017
ms.author: iainfou
translationtype: Human Translation
ms.sourcegitcommit: 356de369ec5409e8e6e51a286a20af70a9420193
ms.openlocfilehash: 10e6ba839130d05dc6b3c1c31681478209a73ed6
ms.lasthandoff: 03/27/2017


---
# <a name="how-to-encrypt-virtual-disks-on-a-linux-vm"></a>如何將 Linux VM 上的虛擬磁碟加密
如需強化虛擬機器 (VM) 安全性與法規遵循，可以將 Azure 中的虛擬磁碟加密。 磁碟會使用 Azure 金鑰保存庫中受保護的密碼編譯金鑰進行加密。 您可控制這些密碼編譯金鑰，並可稽核其使用情況。 本文詳細說明如何使用 Azure CLI 2.0 將 Linux VM 上的虛擬磁碟加密。 您也可以使用 [Azure CLI 1.0](virtual-machines-linux-encrypt-disks-nodejs.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) 來執行這些步驟。

## <a name="quick-commands"></a>快速命令
如果您需要快速完成工作，下列章節詳細說明將 VM 上的虛擬磁碟加密的基本命令。 每個步驟的詳細資訊和內容可在文件其他地方找到，[從這裡開始](#overview-of-disk-encryption)。

您需要安裝最新的 [Azure CLI 2.0](/cli/azure/install-az-cli2)，並使用 [az login](/cli/azure/#login) 來登入 Azure 帳戶。 在下列範例中，請以您自己的值取代範例參數名稱。 範例參數名稱包含 `myResourceGroup`、`myKey` 和 `myVM`。

首先，使用 [az provider register](/cli/azure/provider#register) 啟用您的 Azure 訂用帳戶中的 Azure Key Vault 提供者，以及使用 [az group create](/cli/azure/group#create) 建立資源群組。 下列範例會在 `WestUS` 位置建立名為 `myResourceGroup` 的資源群組：

```azurecli
az provider register -n Microsoft.KeyVault
az group create --name myResourceGroup --location WestUS
```

使用 [az keyvault create](/cli/azure/keyvault#create) 建立 Azure Key Vault 並啟用 Key Vault 以便搭配磁碟加密使用。 針對 `keyvault_name` 指定唯一的 Key Vault 名稱，如下所示︰

```azurecli
keyvault_name=myUniqueKeyVaultName
az keyvault create --name $keyvault_name --resource-group myResourceGroup \
  --location WestUS --enabled-for-disk-encryption True
```

使用 [az keyvault key create](/cli/azure/keyvault/key#create) 在 Key Vault 中建立密碼編譯金鑰。 下列範例會建立名為 `myKey` 的金鑰：

```azurecli
az keyvault key create --vault-name $keyvault_name --name myKey --protection software
```

使用 Azure Active Directory 搭配 [az ad sp create-for-rbac](/cli/azure/ad/sp#create-for-rbac) 建立服務主體。 服務主體會處理 Key Vault 中密碼編譯金鑰的驗證與交換。 下列範例會讀取服務主體識別碼和密碼的值，以便使用於後續命令︰

```azurecli
read sp_id sp_password <<< $(az ad sp create-for-rbac --query [appId,password] -o tsv)
```

密碼只會在您建立服務主體時輸出。 如有需要，檢視及記錄密碼 (`echo $sp_password`)。 您可以使用 [az ad sp list](/cli/azure/ad/sp#list) 列出服務主體，以及使用 [az ad sp show](/cli/azure/ad/sp#show) 檢視特定服務主體的其他相關資訊。

使用 [az keyvault set-policy](/cli/azure/keyvault#set-policy) 設定 Key Vault 的權限。 下列範例中的服務主體識別碼是由先前命令所提供︰

```azurecli
az keyvault set-policy --name $keyvault_name --spn $sp_id \
  --key-permissions all \
  --secret-permissions all
```

使用 [az vm create](/cli/azure/vm#create) 建立 VM 並連結 5GB 資料磁碟。 只有某些 Marketplace 映像支援磁碟加密。 下列範例會使用 **CentOS 7.2n** 映像建立名為 `myVM` 的 VM︰

```azurecli
az vm create -g myResourceGroup -n myVM --image OpenLogic:CentOS:7.2n:7.2.20160629 \
  --admin-username azureuser --ssh-key-value ~/.ssh/id_rsa.pub \
  --data-disk-sizes-gb 5
```

SSH 連線到您的 VM。 建立磁碟分割和檔案系統，然後掛接資料磁碟。 如需詳細資訊，請參閱[連線到 Linux VM 以掛接新磁碟](virtual-machines-linux-add-disk.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json#connect-to-the-linux-vm-to-mount-the-new-disk)。 關閉 SSH 工作階段。

使用 [az vm encryption enable](/cli/azure/vm/encryption#enable) 將您的 VM 加密。 下列範例使用先前 `ad sp create-for-rbac` 命令中的 `$sp_id` 和 `$sp_password` 變數︰

```azurecli
az vm encryption enable --resource-group myResourceGroup --name myVM \
  --aad-client-id $sp_id \
  --aad-client-secret $sp_password \
  --disk-encryption-keyvault $keyvault_name \
  --key-encryption-key myKey \
  --volume-type all
```

磁碟加密程序需花一些時間完成。 使用 [az vm encryption show](/cli/azure/vm/encryption#show) 監視此程序的狀態：

```azurecli
az vm encryption show --resource-group myResourceGroup --name myVM
```

狀態顯示 **EncryptionInProgress**。 等到 OS 磁碟的狀態回報 **VMRestartPending**，然後使用 [az vm restart](/cli/azure/vm#restart) 重新啟動您的 VM：

```azurecli
az vm restart --resource-group myResourceGroup --name myVM
```

磁碟加密程序會在開機過程中完成，因此請等候幾分鐘，再使用 **az vm encryption show** 檢查加密的狀態：

```azurecli
az vm encryption show --resource-group myResourceGroup --name myVM
```

 狀態現在應會將 OS 磁碟和資料磁碟回報為 [已加密]。

## <a name="overview-of-disk-encryption"></a>磁碟加密概觀
Linux VM 上的待用虛擬磁碟會使用 [dm-crypt](https://wikipedia.org/wiki/Dm-crypt) 進行加密。 將 Azure 中的虛擬磁碟加密完全免費。 密碼編譯金鑰會使用軟體保護功能儲存在 Azure 金鑰保存庫中，或者您可以在 FIPS 140-2 第 2 級標準認證的硬體安全性模組 (HSM) 中匯入或產生金鑰。 您可保留這些密碼編譯金鑰的控制權，並可稽核其使用情況。 這些密碼編譯金鑰用來加密及解密連接到 VM 的虛擬磁碟。 Azure Active Directory 服務主體會提供一個安全機制，以便在 VM 開機或關機時發出這些密碼編譯金鑰。

將 VM 加密的程序如下所示：

1. 在 Azure 金鑰保存庫中建立密碼編譯金鑰。
2. 將密碼編譯金鑰設定為可用於磁碟加密。
3. 若要讀取 Azure Key Vault 中的密碼編譯金鑰，請使用適當的權限建立 Azure Active Directory 服務主體。
4. 發出命令以將您的虛擬磁碟加密，並指定要使用的 Azure Active Directory 服務主體和適當密碼編譯金鑰。
5. Azure Active Directory 服務主體會向 Azure Key Vault 要求所需的密碼編譯金鑰。
6. 虛擬磁碟會使用所提供的密碼編譯金鑰進行加密。

## <a name="encryption-process"></a>加密程序
磁碟加密依賴下列其他元件︰

* **Azure 金鑰保存庫** - 用來保護磁碟加密/解密程序所使用的密碼編譯金鑰和密碼。 
  * 如果有的話，您可以使用現有的 Azure 金鑰保存庫。 您沒有專門用來加密磁碟的金鑰保存庫。
  * 若要區分系統管理界限和金鑰可視性，您可以建立專用的金鑰保存庫。
* **Azure Active Directory** - 處理必要密碼編譯金鑰的安全交換以及所要求動作的驗證。 
  * 您通常使用現有的 Azure Active Directory 執行個體來裝載您的應用程式。
  * 服務主體會提供一個安全機制來要求並取得適當的密碼編譯金鑰。 您並未開發與 Azure Active Directory 整合的實際應用程式。

## <a name="requirements-and-limitations"></a>需求和限制
磁碟加密支援的案例和需求：

* 下列 Linux 伺服器 SKU - Ubuntu、CentOS、SUSE、SUSE Linux Enterprise Server (SLES) 和 Red Hat Enterprise Linux。
* 所有資源 (例如金鑰保存庫、儲存體帳戶、VM) 必須位於相同的 Azure 區域和訂用帳戶。
* 標準 A、D、DS、G 和 GS 系列 VM。

下列案例目前不支援磁碟加密︰

* 基本層 VM。
* 使用傳統部署模型建立的 VM。
* 在 Linux VM 上停用作業系統磁碟加密。
* 在已經加密的 Linux VM 上更新密碼編譯金鑰。

## <a name="create-azure-key-vault-and-keys"></a>建立 Azure Key Vault 和金鑰
您需要安裝最新的 [Azure CLI 2.0](/cli/azure/install-az-cli2)，並使用 [az login](/cli/azure/#login) 來登入 Azure 帳戶。 在整個命令範例中，以您自己的名稱、位置和金鑰值取代所有的範例參數。 下列範例使用 `myResourceGroup`、`myKeyVault`、`myAADApp` 等的慣例。

在整個命令範例中，以您自己的名稱、位置和金鑰值取代所有的範例參數。 下列範例使用 `myResourceGroup`、`myKey`、`myVM` 等的慣例。

建立 Azure 金鑰保存庫的第一個步驟是儲存您的密碼編譯金鑰。 Azure 金鑰保存庫儲存可讓您安全地在應用程式和服務中實作的金鑰和密碼 (Secret 或 Password)。 如需虛擬磁碟加密，您可使用金鑰保存庫來儲存用來加密或解密虛擬磁碟的密碼編譯金鑰。 

使用 [az provider register](/cli/azure/provider#register) 啟用您的 Azure 訂用帳戶中的 Azure Key Vault 提供者，以及使用 [az group create](/cli/azure/group#create) 建立資源群組。 下列範例會在 `WestUS` 位置建立名為 `myResourceGroup` 的資源群組：

```azurecli
az provider register -n Microsoft.KeyVault
az group create --name myResourceGroup --location WestUS
```

包含密碼編譯金鑰和相關聯計算資源 (例如儲存體和 VM) 的 Azure 金鑰保存庫本身必須位於相同的區域中。 使用 [az keyvault create](/cli/azure/keyvault#create) 建立 Azure Key Vault 並啟用 Key Vault 以便搭配磁碟加密使用。 針對 `keyvault_name` 指定唯一的 Key Vault 名稱，如下所示︰

```azurecli
keyvault_name=myUniqueKeyVaultName
az keyvault create --name $keyvault_name --resource-group myResourceGroup \
  --location WestUS --enabled-for-disk-encryption True
```

您可以使用軟體或硬體安全性模型 (HSM) 保護功能來儲存密碼編譯金鑰。 使用 HSM 時需要進階金鑰保存庫。 建立進階金鑰保存庫 (而非用來儲存軟體保護金鑰的標準金鑰保存庫) 會有額外的成本。 若要建立進階金鑰保存庫，在前一個步驟中將 `--sku Premium` 新增至命令。 下列範例會使用軟體保護的金鑰，因為我們建立了標準金鑰保存庫。 

在兩種保護模型中，Azure 平台都必須獲得存取權，才能在 VM 開機時要求密碼編譯金鑰來將虛擬磁碟解密。 使用 [az keyvault key create](/cli/azure/keyvault/key#create) 在 Key Vault 中建立密碼編譯金鑰。 下列範例會建立名為 `myKey` 的金鑰：

```azurecli
az keyvault key create --vault-name $keyvault_name --name myKey --protection software
```


## <a name="create-the-azure-active-directory-service-principal"></a>建立 Azure Active Directory 服務主體
將虛擬磁碟加密或解密時，您可以指定帳戶以處理 Key Vault 中密碼編譯金鑰的驗證和交換。 此帳戶 (Azure Active Directory 服務主體) 可讓 Azure 平台代表 VM 要求適當的密碼編譯金鑰。 雖然許多組織都有專用的 Azure Active Directory 目錄，但您的訂用帳戶中會有預設 Azure Active Directory 執行個體。

使用 Azure Active Directory 搭配 [az ad sp create-for-rbac](/cli/azure/ad/sp#create-for-rbac) 建立服務主體。 下列範例會讀取服務主體識別碼和密碼的值，以便使用於後續命令︰

```azurecli
read sp_id sp_password <<< $(az ad sp create-for-rbac --query [appId,password] -o tsv)
```

密碼只會在您建立服務主體時顯示。 如有需要，檢視及記錄密碼 (`echo $sp_password`)。 您可以使用 [az ad sp list](/cli/azure/ad/sp#list) 列出服務主體，以及使用 [az ad sp show](/cli/azure/ad/sp#show) 檢視特定服務主體的其他相關資訊。

若要成功加密或解密虛擬磁碟，Key Vault 中儲存之密碼編譯金鑰的權限必須設定為允許 Azure Active Directory 服務主體讀取金鑰。 使用 [az keyvault set-policy](/cli/azure/keyvault#set-policy) 設定 Key Vault 的權限。 下列範例中的服務主體識別碼是由先前命令所提供︰

```azurecli
az keyvault set-policy --name $keyvault_name --spn $sp_id \
  --key-permissions all \
  --secret-permissions all
```


## <a name="create-virtual-machine"></a>Create virtual machine
若要實際加密某些虛擬磁碟，請建立 VM 並新增資料磁碟。 使用 [az vm create](/cli/azure/vm#create) 建立要加密的 VM 並連結 5GB 資料磁碟。 只有某些 Marketplace 映像支援磁碟加密。 下列範例會使用 **CentOS 7.2n** 映像建立名為 `myVM` 的 VM︰

```azurecli
az vm create -g myResourceGroup -n myVM --image OpenLogic:CentOS:7.2n:7.2.20160629 \
  --data-disk-sizes-gb 5
```

透過 SSH 連線至 VM 以建立磁碟分割和檔案系統，然後掛接資料磁碟。 如需詳細資訊，請參閱[連線到 Linux VM 以掛接新磁碟](virtual-machines-linux-add-disk.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json#connect-to-the-linux-vm-to-mount-the-new-disk)。 關閉 SSH 工作階段。


## <a name="encrypt-virtual-machine"></a>將虛擬機器加密
若要將虛擬磁碟加密，您可結合所有先前的元件︰

1. 指定 Azure Active Directory 服務主體和密碼。
2. 指定金鑰保存庫以儲存已加密磁碟的中繼資料。
3. 指定要用於實際加密和解密的密碼編譯金鑰。
4. 指定是否要將作業系統磁碟、資料磁碟或所有磁碟加密。

使用 [az vm encryption enable](/cli/azure/vm/encryption#enable) 將您的 VM 加密。 下列範例使用先前 `ad sp create-for-rbac` 命令中的 `$sp_id` 和 `$sp_password` 變數︰

```azurecli
az vm encryption enable --resource-group myResourceGroup --name myVM \
  --aad-client-id $sp_id \
  --aad-client-secret $sp_password \
  --disk-encryption-keyvault $keyvault_name \
  --key-encryption-key myKey \
  --volume-type all
```

磁碟加密程序需花一些時間完成。 使用 [az vm encryption show](/cli/azure/vm/encryption#show) 監視此程序的狀態：

```azurecli
az vm encryption show --resource-group myResourceGroup --name myVM
```

輸出類似於下列截斷的範例：

```json
[
  "dataDisk": "EncryptionInProgress",
  "osDisk": "EncryptionInProgress",
]
```

等到 OS 磁碟的狀態回報 **VMRestartPending**，然後使用 [az vm restart](/cli/azure/vm#restart) 重新啟動您的 VM：

```azurecli
az vm restart --resource-group myResourceGroup --name myVM
```

磁碟加密程序會在開機過程中完成，因此請等候幾分鐘，再使用 **az vm encryption show** 檢查加密的狀態：

```azurecli
az vm encryption show --resource-group myResourceGroup --name myVM
```

狀態現在應會將 OS 磁碟和資料磁碟回報為 [已加密]。


## <a name="add-additional-data-disks"></a>新增其他資料磁碟
一旦將資料磁碟加密，您稍後即可將其他虛擬磁碟新增至您的 VM，而且予以加密。 當您執行 `az vm encryption enable` 命令時，請使用 `--sequence-version` 參數遞增順序版本。 此順序版本參數可讓您在相同的 VM 上執行重複的作業。

例如，將第二個虛擬磁碟新增至您的 VM，如下所示︰

```azurecli
az vm disk attach-new --resource-group myResourceGroup --vm-name myVM --size-in-gb 5
```

重新執行命令來加密虛擬磁碟，這次新增 `--sequence-version` 參數，並從第一次執行遞增此值，如下所示︰

```azurecli
az vm encryption enable --resource-group myResourceGroup --name myVM \
  --aad-client-id $sp_id \
  --aad-client-secret $sp_password \
  --disk-encryption-keyvault $keyvault_name \
  --key-encryption-key myKey \
  --volume-type all \
  --sequence-version 2
```


## <a name="next-steps"></a>後續步驟
* 如需有關管理 Azure 金鑰保存庫 (包括刪除密碼編譯金鑰和保存庫) 的詳細資訊，請參閱[使用 CLI 管理金鑰保存庫](../key-vault/key-vault-manage-with-cli.md)。
* 如需有關磁碟加密 (例如準備加密的自訂 VM 以上傳至 Azure) 的詳細資訊，請參閱 [Azure 磁碟加密](../security/azure-security-disk-encryption.md)。


