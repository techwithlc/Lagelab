# AZ700: M04-Unit 4 - 建立與配置 Azure 負載平衡器

## 創建和配置 Azure 負載均衡器

### 一、情境模擬

協助 Fictional Contoso Ltd organization 建立 Internal Load Balancer。


**預計時間：** 60 分鐘 (包含45分鐘部署)

### (一)架構圖 
![架構圖](./image\m2u4\1_Framework.png)
### (二)Load Balancer 的使用情境與重要性

| 用途 | 說明 | 實際應用案例（產業） | Lab 相關設定 |
|------|------|----------------|-------------|
| 流量分配 (負載平衡, Traffic distribution) | 將流量分配到多個 VM，避免單一 VM 過載 | 電商、串流、金融交易系統，如 Shopee、Lazada 透過 Load Balancer 分散大量購物請求 | 設定 Backend Pool，加入多台 VM |
| 提高高可用性 (High Availability) | 讓系統即使有 VM 當機，依然能運作 | SaaS 服務：Zoom、Office 365，確保即使某台伺服器故障，服務仍然可用 | 設定 Health Probe 來監測 VM 狀態 |
| 確保應用程序穩定 | 減少單點故障，提升系統穩定性 | 企業內部系統，如 SAP(ERP)、Salesforce (CRM)，使公司內部 API 或資料庫更穩定 | 設定 Load Balancer 規則來自動轉發流量 |
| 提升效能 (Scaling & Performance) | 讓請求平均分佈，減少延遲 | 全球型網路服務，如 Agoda、Booking.com 透過 Load Balancer 確保不同地區用戶連到最近的伺服器，降低延遲 | 透過 Load Balancer 來管理不同 VM 的流量 |
| API Gateway 與微服務架構 | 負載均衡 API 請求，提高請求處理效率 | 叫車平台，如 Uber、Grab 透過 Load Balancer 分配 API 請求給不同的微服務，確保叫車服務不中斷 | 將 API 請求導向不同的 VM |



 **目的**：這個 Lab 讓您實際體驗如何使用 Azure Load Balancer 提升應用程式的可用性與效能，這在高流量網站、企業內部應用和雲端服務中為常見的技術。

---

## **二、在此練習中，您將熟悉以下概念**

- **任務 1**: 建立 Virtual Network 虛擬網路
- **任務 2**: 建立 Backend Servers 後端伺服器
- **任務 3**: 建立 Load Balancer
- **任務 4**: 建立 Load Balancer 相關資源
- **任務 5**: 測試 Load Balancer 是否能成功使用

---

## **三、步驟**

### **Task 1: 建立 Virtual Network (VNet) 和 Subnet (子網路)**

1. 登入 Azure Portal。
2. 在 Azure Portal 首頁，使用全域搜尋搜尋 **Virtual Networks**，然後選擇 **Virtual Networks**。
3. 在 **虛擬網路** 頁面，選擇 **建立 (Create)** 來開啟 **建立虛擬網路**。
4. 在 **基本資訊 (Basics)** 標籤頁，輸入以下資料來建立虛擬網路。

**請插入圖片 1：Azure Portal 建立 Virtual Network 畫面**

5. 選擇 **下一步：IP Addresses**，然後設定 IPv4 位址空間為 `10.1.0.0/16`。
6. 點擊 **+ 新增子網路 (Add subnet)**，分別新增：
   - `myBackendSubnet`，地址範圍 `10.1.0.0/24`
   - `myFrontEndSubnet`，地址範圍 `10.1.2.0/24`

**請插入圖片 2：Subnet 設定畫面**

7. 選擇 **下一步：安全性 (Next: Security)**，啟用 **BastionHost**。
8. 點擊 **檢閱 + 建立 (Review + Create)**，再點擊 **建立 (Create)**。

**請插入圖片 3：Bastion 啟用畫面**

---

### **Task 2: 建立 Backend Servers (後端伺服器)**

1. 開啟 Azure Cloud Shell，選擇 **PowerShell**。
2. 上傳以下兩個檔案至 Cloud Shell：
   - `azuredeploy.json`
   - `azuredeploy.parameters.json`

3. 執行以下 PowerShell 命令來部署 ARM (Azure Resource Manager) 範本：

   ```powershell
   $RGName = "IntLB-RG"
   New-AzResourceGroupDeployment -ResourceGroupName $RGName -TemplateFile azuredeploy.json -TemplateParameterFile azuredeploy.parameters.json
   ```

4. 等待 5-10 分鐘完成 VM 建立。

**請插入圖片 4：Azure Cloud Shell 執行畫面**

### **Task 3: 建立 Load Balancer**

1. 在 Azure Portal，選擇 **建立資源 (Create a resource)**。
2. 搜尋 **Load Balancer**，並選擇 **建立 (Create)**。
3. 在 **基本資訊 (Basics)** 頁面，選擇 **Standard SKU** 負載平衡器。
4. 在 **前端 IP 配置 (Frontend IP Configuration)** 頁面，新增前端 IP。
5. 點選 **檢閱 + 建立 (Review + Create)**，然後點選 **建立 (Create)**。

**請插入圖片 4：Azure Load Balancer 設定畫面**

---
### **Task4: 建立 Load Balancer 相關資源**
#### **步驟**
1. 在 Azure Portal，開啟 **myIntLoadBalancer**。
2. 在 **Settings** 選單中，選擇 **Backend pools**，點選「新增」。
3. 設定名稱為 myBackendPool，並選擇所屬的虛擬網路 IntLB-VNet。
4. 新增虛擬機，將 myVM1、myVM2、myVM3 加入 Backend Pool。
5. 儲存設定。

**請插入圖片 5：Load Balancer Backend Pool 設定**

---

### **Task5: 測試 Load Balancer 是否能成功使用**
#### **步驟**
1. 在 Azure Portal，選擇 **「Create a resource」**，然後建立 **虛擬機（Virtual Machine, VM）**。
2. 設定 VM 參數：
    - Password: LageLab1234567890
3. 選擇 **Next : Disks**，再選擇 **Next : Networking**。
4. 在 Networking 的頁面填入下面資訊：
    - 選擇「Review+Create」>「Create」，並等待 VM 確實部署再繼續下一個 task。
5. 在 Azure 入口網站首頁，選擇 所有資源 (All resources)，然後從資源列表中選擇 myIntLoadBalancer。
6. 在 Overview 頁面，記下 私有 IP 位址 (Private IP address)，或將其複製到剪貼簿。
7. 返回 首頁 (Home)，然後在 Azure 入口網站首頁，選擇 所有資源 (All resources)，再選擇剛剛建立的 myTestVM 虛擬機 (virtual machine)。
8. 在 概覽 (Overview) 頁面，選擇 連線 (Connect)，然後選擇 Bastion。
9. 選擇 使用 Bastion (Use Bastion)。
10. 在使用者名稱 (Username) 欄位輸入 TestUser，在密碼 (Password) 欄位輸入你設定的密碼，然後選擇 連線 (Connect)。
11. 在工作列 (Taskbar) 上選擇 Internet Explorer 圖示以開啟網頁瀏覽器。
12. 在瀏覽器的網址列中輸入（或貼上）先前記錄的私有 IP 位址 (Private IP address)（例如 10.1.0.4），然後按下 Enter。
13. 瀏覽器視窗將顯示 IIS Web 伺服器的預設網頁，表示內部負載平衡器 (Internal Load Balancer) 的其中一台虛擬機已經回應。

---

## **清理資源**
注意： 請記得刪除所有不再使用的 Azure 資源，以確保不會產生意外的費用。
1.	在 Azure 入口網站 中，開啟 Cloud Shell 面板 內的 PowerShell 會話。
2.	執行以下命令，刪除你在本模組的實驗過程中建立的所有資源群組：
Remove-AzResourceGroup -Name 'IntLB-RG' -Force -AsJob
3.	注意： 此命令使用 -AsJob 參數，表示會異步執行（非同步處理）。
- 你可以立即執行其他 PowerShell 命令，而不需要等候刪除完成。
- 但實際刪除資源群組仍需要數分鐘才能完成。


執行以下命令刪除資源群組：

```powershell
Remove-AzResourceGroup -Name 'IntLB-RG' -Force -AsJob
```

## **Takeaways**

✅ **負載平衡 (Load Balancing)**：高效地分配流量到後端伺服器。

✅ **Azure Load Balancer** 透過 **負載平衡規則** 和 **健康探測** 來管理流量。

✅ **Public vs. Private Load Balancer**：
   - **Public Load Balancer**：適用於對外應用，如網頁服務。
   - **Private Load Balancer**：適用於內部應用，如內部 API、資料庫。



