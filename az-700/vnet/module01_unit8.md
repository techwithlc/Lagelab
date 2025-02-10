### 小試身手 - 在 Azure 中設計和實施虛擬網路

> **辣個通知** 請先確保您有看過這個影片註冊 Azure Free Account - https://youtu.be/ZsfllNRuErk?si=3q2RHUsL9vrX3ltB

## 情境模擬

現在您已經準備好在 Azure 入口網站中部署虛擬網路。

考慮到不同區域公司分部即將開張，該公司正在將基礎設施和應用程序遷移到 Azure。你身為網絡工程師，你必須計劃並實施兩個虛擬網路進行peering對等連接


**預計時間：** 20 分鐘

**CoreServicesVnet** (10.20.0.0/16) 虛擬網路部署在 **美國東部** 區域，為公司總部區域。此虛擬網路內有TestVM1 (10.20.20.4)。

**ManufacturingVnet** (10.30.0.0/16) 虛擬網路部署在 **西歐** 區域，為新公司區域。此虛擬網路將包含用於製造設施運營的系統。該組織預計會有大量內部連接設備。此虛擬網路內有ManufactorungVM (10.30.10.4)。

## 架構圖

![架構圖](./image/8-exercise-connect-two-azure-virtual-networks-global.png)


### 公司的網路佈局
### 您將建立以下資源：
**TestVM1(美東)**、**ManufactoringVM(西歐)**、**Peering對等連接**

### 在此練習中，您將：

1. **任務 1：** 建立虛擬機器來測試配置
2. **任務 2：** 使用 RDP 連接到測試虛擬機
3. **任務 3：** 測試虛擬機器之間的連接
4. **任務 4：** 在 CoreServicesVnet 和 ManufacturingVnet 之間建立 VNet 對等互連
5. **任務 5：** 測試虛擬機器之間的連接

### 任務 1：建立虛擬機器來測試配置

### 建立 ManufacturingVM
1. 在 Azure 入口網站中，選擇 Cloud Shell 圖示（右上角）。 如果有必要，請配置 shell。 
    ![Cloud Shell](./image/unit8_1_cloud_shell_icon.jpg)
    + 選擇 **PowerShell**。
    ![Powershell](./image/unit8_2_select_powershell.jpg)
    + 選擇 **「不需要儲存帳戶」** 和您的 **訂閱**，然後選擇 **「套用」**。
    ![](./image/unit8_3_select_powershell.jpg)
    + 等待終端機建立並顯示提示。 

2. 在 Cloud Shell 窗格的工具列上，選擇「管理檔案」圖標，在下拉式選單中選擇 **「上傳」** ，並將下列檔案**ManufacturingVMazuredeploy.json**和**ManufacturingVMazuredeploy.parameters.json**從來源資料夾**F:\Allfiles\Exercises\M011**上傳至 Cloud Shell 主目錄。

   >**注意**: 
   + 檔案下載網址: https://github.com/MicrosoftLearning/AZ-700-Designing-and-Implementing-Microsoft-Azure-Networking-Solutions/archive/master.zip
   + 再上傳前需修改**ManufacturingVMazuredeploy.json**和**ManufacturingVMazuredeploy.parameters.json**內vmsize value為 **Standard_D2ls_v5**
    
3. 部署以下 ARM 範本來建立此練習所需的 VM： (複製以下code至powershell執行)

   >**注意**: 系統將提示您提供管理員密碼。
   ```powershell
   $RGName = "ContosoResourceGroup"
   New-AzResourceGroupDeployment -ResourceGroupName $RGName -TemplateFile ManufacturingVMazuredeploy.json -TemplateParameterFile ManufacturingVMazuredeploy.parameters.json
   ```

4. 部署完成後，請前往 Azure 入口網站主頁，然後選擇 **「虛擬機器」**。
5. 驗證虛擬機器是否已建立。


![Resourcegroup](./image/resoure.png)

### 任務 2：使用 RDP 連接到測試虛擬機

1. 在 Azure 入口網站首頁上，導航到 **全域搜尋欄** 並搜尋 **虛擬網路**，然後在服務下選擇 **虛擬網路**。

![Virtualnetwork](./image/virtual.png)

2. 在虛擬網路頁面上選擇 **建立**。
3. 使用以下表格中的信息來建立 **CoreServicesVnet** 虛擬網路。

| 標籤            | 選項               | 值                  |
|----------------|----------------------|------------------------|
| 基本         | 資源群組       | ContosoResourceGroup   |
|                | 名稱                 | CoreServicesVnet       |
|                | 區域               | (美國) 美國東部           |
| IP 位址   | IPv4 位址空間   | 10.20.0.0/16           |

![Virtualnettype](./image/virtual-type.png)

4. 使用以下表格中的信息來建立 **CoreServicesVnet** 子網路。


| 子網路                  | 選項               | 值                  |
|-------------------------|----------------------|------------------------|
| GatewaySubnet           | 子網路名稱          | GatewaySubnet          |
|                         | 子網路位址範圍 | 10.20.0.0/27           |
| SharedServicesSubnet    | 子網路名稱          | SharedServicesSubnet   |
|                         | 子網路位址範圍 | 10.20.10.0/24          |
| DatabaseSubnet          | 子網路名稱          | DatabaseSubnet         |
|                         | 子網路位址範圍 | 10.20.20.0/24          |
| PublicWebServiceSubnet  | 子網路名稱          | PublicWebServiceSubnet |
|                         | 子網路位址範圍 | 10.20.30.0/24          |

![Subnet](./image/subnet.png)

> 記得先把 default subnet 刪除掉，不然等等會有 10.20.0.0 CIDR 衝突

1. 要完成 **CoreServicesVnet** 及其相關子網路的建立，請選擇 **檢閱 + 建立**。
2. 確認您的配置通過驗證，然後選擇 **建立**。

![addsubnet](./image/add-subnet.png)

> GatewaySubnet 比較特殊，記得點選 Add Subnet 旁邊的 button，另外選擇 IP Size 是在後面選擇，而非輸入

![sharedsubnet](./image/shared-subnet.png)

順利的話，你應該要看到以下的子網在你的 **CoreServicesVNnet**

![corevnet](./image/corevnet.png)

3. 根據以下表格，重複步驟 1 - 8 為每個 VNet 建立。

### 任務 3：測試虛擬機器之間的連接

| 標籤            | 選項               | 值                  |
|----------------|----------------------|------------------------|
| 基本         | 資源群組       | ContosoResourceGroup   |
|                | 名稱                 | ManufacturingVnet      |
|                | 區域               | (歐洲) 西歐   |
| IP 位址   | IPv4 位址空間   | 10.30.0.0/16           |

| 子網路                  | 選項               | 值                  |
|-------------------------|----------------------|------------------------|
| ManufacturingSystemSubnet | 子網路名稱          | ManufacturingSystemSubnet |
|                         | 子網路位址範圍 | 10.30.10.0/24          |
| SensorSubnet1           | 子網路名稱          | SensorSubnet1          |
|                         | 子網路位址範圍 | 10.30.20.0/24          |
| SensorSubnet2           | 子網路名稱          | SensorSubnet2          |
|                         | 子網路位址範圍 | 10.30.21.0/24          |
| SensorSubnet3           | 子網路名稱          | SensorSubnet3          |
|                         | 子網路位址範圍 | 10.30.22.0/24          |


### 任務 4：在 CoreServicesVnet 和 ManufacturingVnet 之間建立 VNet 對等互連

| 標籤            | 選項               | 值                  |
|----------------|----------------------|------------------------|
| 基本         | 資源群組       | ContosoResourceGroup   |
|                | 名稱                 | ResearchVnet           |
|                | 區域               | 東南亞         |
| IP 位址   | IPv4 位址空間   | 10.40.0.0/16           |

| 子網路                  | 選項               | 值                  |
|-------------------------|----------------------|------------------------|
| ResearchSystemSubnet    | 子網路名稱          | ResearchSystemSubnet   |
|                         | 子網路位址範圍 | 10.40.0.0/24           |

### 任務 5：測試虛擬機器之間的連接

1. 在 Azure 入口網站首頁上，選擇 **所有資源**。
2. 確認 **CoreServicesVnet**、**ManufacturingVnet** 和 **ResearchVnet** 已列出。

![allvirtualnet](./image/allvirtualnet.png)

3. 在所有的 Vnet 中，於 **設定** 下選擇 **子網路**，確認您建立的子網路已列出，並且 IP 位址範圍正確。
4. 重複步驟 3 - 5 為每個 VNet 進行驗證。

## 使用 Copilot 牛刀小試

Copilot 可以幫助您學習如何使用 Azure 腳本工具。Copilot 還可以協助處理實驗室未涵蓋的領域或您需要更多信息的地方。打開 Edge 瀏覽器並選擇 **Copilot**（右上角）或導航到 [copilot.microsoft.com](https://copilot.microsoft.com)。花幾分鐘嘗試以下提示：

- 您能否提供一個實際場景中使用 10.30.0.0/16 IP 位址的示例？
- 在美國東部區域建立一個名為 **CoreServicesVnet** 的虛擬網路的 Azure PowerShell 命令是什麼？該虛擬網路應使用 10.20.0.0/16 IP 位址空間，可以嘗試一下 Copilot 回的 Powershell command 能不能用

![copilot](./image/copilot.png)

- 在西歐區域建立一個名為 **ManufacturingVnet** 的虛擬網路的 Azure CLI 命令是什麼？該虛擬網路應使用 10.30.0.0/16 IP 位址空間。


## 關鍵要點

- **Azure 虛擬網路** 是一項服務，為您在 Azure 中的私有網路提供基礎構建塊。該服務的實例（虛擬網路）使許多類型的 Azure 資源能夠安全地相互通信、與互聯網通信以及與本地網路通信。確保不重疊的位址空間。確保您的虛擬網路位址空間（CIDR 塊）不與您組織的其他網路範圍重疊。
- 虛擬網路中的所有 Azure 資源都部署到虛擬網路內的子網路中。子網路使您能夠將虛擬網路分割成一個或多個子網路，並為每個子網路分配虛擬網路位址空間的一部分。您的子網路不應覆蓋虛擬網路的整個位址空間。提前計劃並為未來保留一些位址空間。


## 技術學習補充

* [什麼是 Azure Virtual Network?](https://learn.microsoft.com/en-us/training/modules/introduction-to-azure-virtual-networks/2-explore-azure-virtual-networks)
* [什麼是子網路遮罩？](https://aws.amazon.com/tw/what-is/cidr/)