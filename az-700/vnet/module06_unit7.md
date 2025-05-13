# M06-單元 7：使用 Azure 入口網站部署與設定 Azure 防火牆

## 練習情境

你是 Contoso 公司網路安全小組的一員，負責建立防火牆規則來允許或拒絕對特定網站的存取。本練習將引導你完成環境建立（建立資源群組、虛擬網路與子網路、虛擬機器），接著部署 Azure Firewall 與防火牆原則，設定預設路由、應用程式規則、網路規則、目的NAT(DNAT) 規則，最後進行測試。

地區以**UK South**為例

## 架構圖 
![架構圖](./image/m6u7/7-exercise-deploy-configure-azure-firewall-using-azure-portal.png)

### 練習目標：
1. 建立資源群組  
2. 建立虛擬網路與子網路  
3. 建立虛擬機器  
4. 部署防火牆與防火牆原則  
5. 建立預設路由  
6. 設定應用程式規則  
7. 設定網路規則  
8. 設定目的地 NAT (DNAT) 規則  
9. 修改虛擬機網路介面的 DNS 設定  
10. 測試防火牆規則  

   >**注意**: 可以使用 **[互動式實驗室模擬](https://mslabs.cloudguides.com/guides/AZ-700%20Lab%20Simulation%20-%20Deploy%20and%20configure%20Azure%20Firewall%20using%20the%20Azure%20portal)** ，以便您按照自己的步調點擊實驗室。您可能會發現互動式模擬和託管實驗室之間存在細微的差別，但所展示的核心概念和想法是相同的。
---

## 任務 1：建立資源群組
**在此任務中，您將建立一個新的資源群組**

1. 登入 Azure 入口網站。  
2. 選擇「資源群組」>「建立」。  
   ![建立資源群組](./image/m6u7/1_建立資源群組.jpg)
3. 輸入資源群組名稱：`Test-FW-RG`，選擇區域 UK South。  
4. 點選「檢閱 + 建立」>「建立」。
   ![資源群組](./image/m6u7/2_建立資源群組內容.jpg)


## 任務 2：建立虛擬網路與子網路
**在此任務中，您將建立一個包含兩個子網路的虛擬網路。**

**建立名稱為 `Test-FW-VN` 的虛擬網路，地址空間為 `10.0.0.0/16`。** 
1. 在 Azure 入口網站主頁的搜尋框中，輸入  **虛擬網絡** ，然後在出現時選擇 **「虛擬網路」** 
1. 選擇 **建立**
   ![建立虛擬網路](./image/m6u7/3_建立虛擬網路.jpg)
1. 選擇您先前建立的**Test-FW-RG**
1. 在**名稱** 中, 輸入 **Test-FW-VN**
   ![建立虛擬網路](./image/m6u7/4_建立虛擬網路.jpg)

**修改預設子網路為 `AzureFirewallSubnet`，子網段 `10.0.1.0/26`**  
1. 選擇 **下一步: IP 位址**。 如果預設尚未輸入 IPv4 位址空間 10.0.0.0/16，請輸入該位址空間。
   ![建立虛擬子網路](./image/m6u7/5_建立虛擬網路.jpg)
1. **在子網路名稱**下，選擇**Azure Firewall**. **(這邊與微軟原先的選擇不一樣)**
1. **在「編輯子網路」** 對話方塊中，名稱會自動設定為**AzureFirewallSubnet**.
1. **將子網路位址範圍**變更為**10.0.1.0/26**.
1. 點選 **儲存**.
   ![建立虛擬子網路](./image/m6u7/7_建立子網路_AzureFirewallSubnet.jpg)

**新增子網路 `Workload-SN`，子網段 `10.0.2.0/24`**
1. 點選 **新增子網路**，建立另一個子網，將託管您即將建立的工作負載伺服器
   ![建立虛擬子網路](./image/m6u7/8_add_subnet.jpg)
1. 在 **編輯子網路** 對話方塊中，將名稱變更為 **Workload-SN**
1. **將子網路位址範圍** 變更為 **10.0.2.0/24**.
1. 點選 **新增**.
   ![建立虛擬子網路](./image/m6u7/9_add_subnet_Workload-SN.jpg)
1. 點選 **檢閱+ 建立**.
   ![建立虛擬子網路](./image/m6u7/10_review_create.jpg)
1. 點選 **建立**.
   ![建立虛擬子網路](./image/m6u7/11_create.jpg)


## 任務 3：建立虛擬機器
**在此任務中，您將建立工作負載虛擬機器並將其放置在先前建立的 Workload-SN 子網路中。**

**開啟 Cloud Shell，選擇 PowerShell**  
1. 在 Azure 入口網站中，選擇 Cloud Shell 圖示（右上角）。如果需要，配置 shell。  
    + 選擇 **PowerShell**.
    + 選擇 **不需要任何儲存體帳戶**和您的**訂閱**然後選擇 **套用**.
    + 等待終端機建立並顯示提示 
   ![powershell](./image/m6u7/12_powershell.jpg)
   ![powershell](./image/m6u7/13_powershell.jpg)

**上傳 `firewall.json` 與 `firewall.parameters.json` 檔案** 
1. 在 Cloud Shell 窗格的工具列中，選擇 **「管理檔案」** ，在下拉式選單中選擇 **「上傳」** ，並將下列檔案 **firewall.json** 和 **firewall.parameters.json從來源資料夾F:\Allfiles\Exercises\M06** 逐一上傳到 Cloud Shell 主目錄中
   ![upload](./image/m6u7/14_upload.jpg)
   ![upload](./image/m6u7/15_upload.jpg)

   >**注意**: 
   + 檔案下載網址: https://github.com/MicrosoftLearning/AZ-700-Designing-and-Implementing-Microsoft-Azure-Networking-Solutions/archive/master.zip
   
    
**執行下列命令部署 VM**
部署以下 ARM 範本來建立此練習所需的 VM： (複製以下code至powershell執行)
   >**注意**: 系統將提示您提供管理員密碼。 (需要 **設定大小寫特殊符號**之密碼，ex:Admin1234!)

   ```powershell
   $RGName = "Test-FW-RG"
   New-AzResourceGroupDeployment -ResourceGroupName $RGName -TemplateFile firewall.json -TemplateParameterFile firewall.parameters.json
   ```
   ![vm](./image/m6u7/17_admin.jpg)

   >**注意**: 
   + 執行前需修改微軟連結所提供的**firewall.json** 和 **firewall.parameters.json**內之**vmsize** ，不然後續執行會出現該區域/地區不支援之錯誤
   + 錯誤畫面
   ![error](./image/m6u7/18_error.jpg)
   + 確認地區可用虛擬機器大小
   ![vmsize](./image/m6u7/19_check_region_vmsize.jpg)
   + 修改內容位置
   ![vmsize](./image/m6u7/20_change_vmsize.jpg)

部署成功
   ![vm](./image/m6u7/21_deploy_vm_success.jpg)

**部署完成後記下虛擬機器私有 IP（例如：10.0.2.4）**
   ![vm_ip](./image/m6u7/22_vm_overview.jpg)
   ![vm_ip](./image/m6u7/23_srv-wrok_ip.jpg)

## 任務 4：部署防火牆與防火牆原則
**在此任務中，您將把防火牆部署到配置了防火牆策略的虛擬網路**

建立名稱為 `Test-FW01` 的 Azure Firewall，選擇 SKU 為 Standard。  
建立新的防火牆原則 `fw-test-pol`。  
指定虛擬網路為 `Test-FW-VN`，新增公用 IP：`fw-pip`。  
記下防火牆的私有 IP（例如：10.0.1.4）與公用 IP（例如：172.166.159.224）。

1. 在 Azure 入口網站首頁上，選擇 **「建立資源」**，然後在搜尋方塊中輸入  **「防火牆」** 並在出現時選擇 **「防火牆」** 

   ![create_rss](./image/m6u7/24_create_rss.jpg)
1. **在防火牆** 頁面上，選擇 **建立**

   ![FW](./image/m6u7/26_network_firewall.jpg)
1. **在「基本資訊」** 標籤上，使用下表中的資訊建立防火牆

   | **設定**          | **值**                                                    |
   | -------------------- | ------------------------------------------------------------ |
   | 訂閱         | 選擇你的訂閱                                     |
   | 資源群組       | **Test-FW-RG**                                               |
   | 防火牆名稱        | **Test-FW01**                                                |
   | 地區               | UK South                                                  |
   | 防火牆  SKU        | **標準**                                                 |
   | 防火牆管理  | **使用防火牆策略來管理此防火牆**            |
   | 防火牆原則      | 點選**新增**<br />名稱: **fw-test-pol**<br />地區: **UK South** |

   ![Create a new firewall policy](./image/m6u7/27_FW_setting.jpg)
   ![Create a new firewall policy](./image/m6u7/28_FW_setting.jpg)
1. 我們不使用防火牆管理器，因此 **取消** 選取 **啟用防火牆管理 NIC** 的方塊
 
   ![Cancel enable NIC](./image/m6u7/30.jpg)

   | 選擇虛擬網絡 | **使用現有的**                         |
   | ------------------------ | ---------------------------------------- |
   | 虛擬網絡          | **Test-FW-VN**                           |
   | 公用 IP 位址        | 點選**新增**<br />名稱: **fw-pip** |

   ![Add public IP address to firewall](./image/m6u7/29_FW_setting.jpg)
1. 檢查設定 

   ![Create a firewall - review settings](./image/m6u7/31_FW_all_settings.jpg)
1. 繼續 **檢閱 + 建立**，然後 **建立**.
   ![Create a firewall - review settings](./image/m6u7/32_create_FW.jpg)
1. 等待防火牆部署完成
1. 防火牆部署完成後，選擇  **「前往資源」**
1. 在 **Test-FW01** 的 **概覽** 頁面上，在頁面右側，記下此防火牆的 **防火牆私有 IP** (例如，**10.0.1.4**).
   ![Create a firewall - review settings](./image/m6u7/33_Test-FW01_ip.jpg)
1. 在左側選單中，在 **「設定」** 下，選擇 **「公用 IP 配置」**.
1. **記下fw-pip** 公用 IP 配置的 **IP 位址** (例如，**172.166.159.224**).
   ![Create a firewall - review settings](./image/m6u7/35_fw_pip.jpg)

## 任務 5：建立預設路由
**在此任務中，在 Workload-SN 子網路上，您將設定出站預設路由以穿過防火牆**

建立名稱為 `Firewall-route` 的路由表，並與 `Workload-SN` 子網路關聯。  
新增路由：  
   - 名稱：`fw-dg`  
   - 地址前綴：`0.0.0.0/0`  
   - 下一跳類型：虛擬設備  
   - 下一跳位址：防火牆私有 IP（10.0.1.4）

1. 在 Azure 入口網站首頁上，選擇 **「建立資源」**，然後在搜尋方塊中輸入**路由**並在出現時選擇 **「路由表」**
   ![Create rss](./image/m6u7/36_create_rss.jpg)
1. 在 **路由表** 頁面上，選擇 **建立**
   ![Create a route table](./image/m6u7/37_create_default_route.jpg)
1. **在「基本資訊」** 標籤上，使用下表中的資訊建立一個新的路由表

   | **設定**              | **值**                |
   | ------------------------ | ------------------------ |
   | 訂閱             | 選擇你的訂閱 |
   | 資源群組           | **Test-FW-RG**           |
   | 地區                   | UK South              |
   | 名稱                     | **Firewall-route**       |
   | 傳播閘道路由 | **Yes**                  |


   ![Create a route table](./image/m6u7/38_route_settings.jpg)
1. 點選 **檢閱 + 建立**
1. 點選 **建立**
   ![Create a route table](./image/m6u7/39_create_route.jpg)

1. 部署完成後，選擇 **「前往資源」**
   ![rss overview](./image/m6u7/40_rss_overview.jpg)
1. 在 **防火牆路由頁面的「設定」** 下，選擇 **「子網路」**，然後選擇 **「關聯」**
1. 在 **虛擬網路**上，選擇 **Test-FW-VN**
1. 在 **子網路上**，選擇**Workload-SN**。確保僅為該路由選擇 Workload-SN 子網，否則防火牆將無法正常運作
1. 點選 **確定**.
   ![FW-route subnet associate](./image/m6u7/41_FW_route_subnet_associate.jpg)
   ![FW-route subnet associate](./image/m6u7/42_result.jpg)

1. 在左邊欄位 **設定** 下，點選 **路由**，然後點選 **新增**
1. 在**路由名稱**欄位輸入 **fw-dg**
1. 在**目的地類型** 選擇 **IP位址**
1. 在**目的地IP位址/CIDR範圍(位址前綴目標)**，輸入**0.0.0.0/0**
1. 在**下一跳/下一個躍點類型**中，選擇 **虛擬設備**
1. 在**下一跳/下一個躍點位址**上，輸入您先前記下的防火牆的私人 IP 位址（例如，**10.0.1.4**)
1. 點選 **新增**
    ![Add firewall route](./image/m6u7/43_route.jpg)

## 任務 6：設定應用程式規則
**在此任務中，您將新增允許出站存取 <www.google.com> 的應用程式規則。**

編輯防火牆原則 `fw-test-pol` > Application Rules > 新增規則集。  
規則名稱：`App-Coll01`，來源 `10.0.2.0/24`，協定 `http, https`，目的地：`www.google.com`。

1. 在 Azure 入口網站首頁上，選擇 **「所有資源」**
1. 在資源清單中，選擇您的防火牆原則 **fw-test-pol**
   ![Add an application rule collection](./image/m6u7/44_fw-test-pol.jpg)
1. 在**規則**下，選擇 **應用程式規則**
1. 點選 **“新增規則集合”**
   ![Add an application rule collection](./image/m6u7/45_fw-test-pol-rule.jpg)

1. **在「新增規則集合」** 頁面上，使用下表中的資訊建立一個新的應用程式規則

   | **設定**            | **值**                                 |
   | ---------------------- | ----------------------------------------- |
   | 名稱                   | **App-Coll01**                            |
   | 規則集合類型   | **應用程式**                           |
   | 優先順序               | **200**                                   |
   | 規則集合動作 | **允許**                                 |
   | 規則集合群組  | **DefaultApplicationRuleCollectionGroup** |
   | **規則部分**      |                                           |
   | 名稱                   | **Allow-Google**                          |
   | 來源類型            | **IP 位址**                            |
   | 來源                 | **10.0.2.0/24**                           |
   | 協定               | **http,https**                            |
   | 目的地類型       | **FQDN**                                  |
   | 目的地            | **<www.google.com>**                        |

1. 點選 **新增**
   ![Add an application rule collection](./image/m6u7/46_add_rule_collection.jpg)

## 任務 7：設定網路規則
**在此任務中，您將新增一個網路規則，允許對連接埠 53（DNS）的兩個 IP 位址進行出站存取。**

編輯防火牆原則 `fw-test-pol` > Network Rules > 新增規則集。  
規則名稱：`Net-Coll01`，來源 `10.0.2.0/24`，目的地 `209.244.0.3, 209.244.0.4`，協定 UDP 53。

1. 在 **fw-test-pol頁面的「規則」** 下，選擇 **「網路規則」**.
1. 點選 **新增規則集合**.
  ![Add a network rule collection](./image/m6u7/47_network_ruleset.jpg)

1. **在「新增規則集合」** 頁面上，使用下表中的資訊建立新的網路規則

   | **設定**            | **值**                                                    |
   | ---------------------- | ------------------------------------------------------------ |
   | 名稱                   | **Net-Coll01**                                               |
   | 規則集合類型   | **網路**                                                  |
   | 優先順序               | **200**                                                      |
   | 規則集合動作 | **允許**                                                    |
   | 規則集合群組  | **DefaultNetworkRuleCollectionGroup**                        |
   | **規則部分**      |                                                              |
   | 名稱                   | **Allow-DNS**                                                |
   | 來源類型            | **IP 位址**                                               |
   | 來源                 | **10.0.2.0/24**                                              |
   | 協定               | **UDP**                                                      |
   | 目的地連接埠      | **53**                                                       |
   | 目的地類型       | **IP 位址**                                               |
   | 目的地            | **209.244.0.3, 209.244.0.4**<br />這些是由 Century Link 營運的公共 DNS 伺服器 |

1. 點選 **新增**
  ​ ![Add a network rule collection](./image/m6u7/48_add_network_rule_collection.jpg)

## 任務 8：設定目的 NAT (DNAT) 規則
**在此任務中，您將新增一條 DNAT 規則，讓您可以透過防火牆將遠端桌面連接到 Srv-Work 虛擬機器。**

編輯防火牆原則 `fw-test-pol` > DNAT Rules > 新增規則集。  
將公用 IP (如 172.166.159.224) 的 TCP 3389 導向內部虛擬機的 IP (如 10.0.2.4) 的 TCP 3389。

1. 在 **fw-test-pol頁的「規則」** 下，選擇 **「DNAT 規則」**
1. 點選 **新增規則集合**
​  ![Add a DNAT rule collection](./image/m6u7/49_DNAT_rule_collection.jpg)

1. **在「新增規則集合」** 頁面上，使用下表中的資訊建立新的 DNAT 規則

   | **設定**           | **值**                                                    |
   | --------------------- | ------------------------------------------------------------ |
   | 名稱                  | **rdp**                                                      |
   | 規則集合類型  | **DNAT**                                                     |
   | 優先順序              | **200**                                                      |
   | 規則集合群組 | **DefaultDnatRuleCollectionGroup**                           |
   | **規則部分**     |                                                              |
   | 名稱                  | **rdp-nat**                                                  |
   | 來源類型           | **IP 位址**                                               |
   | 來源                | *                                                            |
   | 協定              | **TCP**                                                      |
   | 目的地連接埠     | **3389**                                                     |
   | 目的地類型      | **IP 位址**                                               |
   | 目的地(防火牆IP位址)           | 輸入您之前記下的 **fw-pip** 防火牆公用 IP 位址。<br />**例如 - 172.166.159.224** |
   | 已轉譯的位址或 FQDN    | 輸入您之前記下的 **Srv-Work** 的私人 IP 位址。<br />**例如 - 10.0.2.4** |
   | 已轉譯連接埠       | **3389**                                                     |

1. 點選 **新增**
​  ![Add a DNAT rule collection](./image/m6u7/50_DNAT_rule_collection_1.jpg)
​  ![Add a DNAT rule collection](./image/m6u7/51_DNAT_rule_collection_2.jpg)

## 任務 9：修改虛擬機器 DNS 設定(主 DNS 位址和輔助 DNS 位址)
**為了在本練習中進行測試，在此任務中，您將設定 Srv-Work 伺服器的主 DNS 位址和輔助 DNS 位址。但是，這不是 Azure 防火牆的一般要求。**

選擇 VM 的網路介面，在「DNS 伺服器」中選擇「自訂」。  
主 DNS：209.244.0.3，次 DNS：209.244.0.4  
儲存後重啟虛擬機器。

1. 在 Azure 入口網站首頁上，選擇 **「資源群組」**

1. 在資源組清單中，選擇您的資源群組 **Test-FW-RG**

1. 在此資源組的資源清單中，選擇 **Srv-Work** 虛擬機器的網路介面 (例如， **srv-work350**)

   ![Select NIC in resource group](./image/m6u7/52_srv-work-nic.jpg)

1. **在「設定」** 下，選擇 **DNS 伺服器**
1. **伺服器**  下，選擇 **自訂**
1. **在新增 DNS 伺服器** 文字方塊中輸入 **209.244.0.3** ，在下一個文字方塊中輸入 **209.244.0.4** 
1. 點選 **儲存**
   ![Change DNS servers on NIC](./image/m6u7/53_nic_dns_settings.jpg)

1. 重新啟動 **Srv-Work** 虛擬機器，目的:讓設定生效
   ![Change DNS servers on NIC](./image/m6u7/54_srv-work_restart.jpg)

## 任務 10：測試防火牆規則
**在此最後任務中，您將測試防火牆以驗證規則是否正確配置並按預期工作。此設定將使您能夠透過防火牆的公用 IP 位址將遠端桌面連線穿過防火牆連線至 Srv-Work 虛擬機器。**

使用遠端桌面連線至防火牆的公用 IP（如 172.166.159.224:3389）。  
登入 `Srv-Work`，開啟瀏覽器瀏覽 `https://www.google.com`（應可開啟）。  
嘗試瀏覽 `https://www.microsoft.com`（應被封鎖）。

1. 在您的電腦上開啟 **遠端桌面連線(Remote Desktop Connection)** (可以在Windows開始搜尋欄位搜尋rdp)
   ![RDP](./image/m6u7/55_rdp.jpg)
   ![RDP](./image/m6u7/56_rdp.jpg)

1. 在電腦框中，輸入防火牆的公共 IP 位址（例如， **20.90.136.51**) ，後面接著 **:3389** (例如， **20.90.136.51:3389**).
1. 在 **使用者名稱** 中，輸入 **TestUser**
1. 點選 **連線**

   ![RDP connection to firewall's public IP address](./image/m6u7/57_rdp.jpg)

1. 在 **輸入您的認證** 中，使用您在部署期間提供的密碼登入 **Srv-Work** 伺服器虛擬機器。
`(密碼:Admin1234!)` ，點選確定

   ![RDP](./image/m6u7/58_rdp.jpg)

1. 在憑證訊息上選擇 **「是」**
   ![RDP](./image/m6u7/59_rdp.jpg)

1. 遠端桌面連線登入後，開啟 Internet Explorer 並瀏覽至 **<https://www.google.com>**
1. **在「安全警報」** 對話方塊中，選擇 **確定**
1. 在可能彈出的 Internet Explorer 安全性警報上選擇 **「關閉」**
1. 應該會看到 Google 主頁。

    ![RDP session on Srv-work server - browser on google.com](./image/m6u7/60_google.jpg)

1. 瀏覽至 **<https://www.microsoft.com>**
1. 應該會看到被防火牆封鎖了
    ![RDP session on Srv-work server - browser blocked on microsoft.com](./image/m6u7/61_microsoft_block.jpg)

## 清除資源

   >**注意** : 請記得刪除任何不再使用的新建立的 Azure 資源。刪除未使用的資源可確保您不會看到意外的費用。

1. 在 Azure 入口網站上，開啟**Cloud Shell**窗格中的**PowerShell**工作階段。
1. 透過執行以下命令刪除您在本模組的實驗中所建立的所有資源組：

```powershell
Remove-AzResourceGroup -Name 'Test-FW-RG' -Force -AsJob
```
   >**注意**: 此命令非同步執行（由 -AsJob 參數決定），因此雖然您可以在同一個 PowerShell 會話中立即執行另一個 PowerShell 命令，但實際刪除資源組之前需要幾分鐘的時間。

## Copilot 牛刀小試
Copilot 可以幫助您學習如何使用 Azure 腳本工具。 Copilot 還可以協助您完成實驗室未涉及的領域或需要更多資訊的領域。開啟 Edge 瀏覽器並選擇 Copilot（右上）或導覽至copilot.microsoft.com。花幾分鐘試試這些提示。

- Firewall 常見使用情境有哪些？  
- Azure Firewall 各 SKU 的比較表  
- Azure Firewall 三種規則差異（Application、Network、DNAT）

### - Firewall 常見使用情境有哪些？

防火牆作為網路安全的基本組成部分，其常見的使用情境包括：

* **邊界安全 (Perimeter Security)**：這是最經典的用途，防火牆部署在組織內部網路與外部不信任網路（如網際網路）之間，用於控制進出流量，防止未經授權的存取和惡意攻擊。
* **內部網路區隔 (Internal Network Segmentation)**：大型組織會將內部網路劃分為不同的區域（例如，伺服器區、使用者區、開發區等），並在這些區域之間部署防火牆，以限制東西向流量。這樣做可以防止一旦某個區域被入侵，攻擊者可以輕易橫向移動到其他重要區域，實現「微切分」的安全策略。
* **應用程式安全 (Application Security)**：現代防火牆（如應用程式防火牆）能夠理解特定應用程式層級的流量，例如 HTTP/HTTPS。這使得它們可以提供更精細的控制，例如基於網址 (URL)、FQDN (完全合格網域名稱) 或應用程式類型來允許或拒絕流量。
* **遠端存取安全 (Secure Remote Access)**：通過設定 VPN (虛擬私人網路) 與防火牆整合，可以為遠端使用者或分支機構提供安全的連線回公司內部網路，並對這些連線的流量進行安全檢查和控制。
* **保護伺服器/服務 (Protecting Servers/Services)**：將提供特定服務（如網站伺服器、郵件伺服器）的伺服器放置在一個稱為「非軍事區」(DMZ) 的隔離網路中，並使用防火牆嚴格控制進出 DMZ 的流量，只允許必要的通訊協定和埠，以降低風險。
* **記錄與監控 (Logging and Monitoring)**：防火牆可以記錄所有通過或被拒絕的流量，這些日誌對於安全事件分析、入侵偵測和合規性審計至關重要。

### - Azure Firewall 各 SKU 的比較表

Azure Firewall 提供不同 SKU (Standard, Premium, Basic) 以滿足不同規模和安全需求的組織。以下是它們主要功能的比較表：

| 功能 / SKU           | Basic                                       | Standard                                       | Premium                                         |
| :------------------- | :------------------------------------------ | :--------------------------------------------- | :---------------------------------------------- |
| **目標使用情境** | 中小型企業、開發/測試環境                  | 大部分企業工作負載、需要基本防火牆功能       | 高安全性、需要進階威脅防護的企業環境          |
| **定價** | 最低                                        | 中等                                           | 最高                                            |
| **擴充性** | 高 (可根據流量自動擴充)                   | 高 (可根據流量自動擴充)                      | 高 (可根據流量自動擴充)                       |
| **可用性** | 內建高可用性                                | 內建高可用性                                   | 內建高可用性                                    |
| **網路流量篩選** | 支援網路規則 (基於 IP, Port, Protocol)       | 支援網路規則 (基於 IP, Port, Protocol)         | 支援網路規則 (基於 IP, Port, Protocol)          |
| **應用程式流量篩選** | 支援應用程式規則 (基於 FQDN, 應用程式類型)     | 支援應用程式規則 (基於 FQDN, 應用程式類型)     | 支援應用程式規則 (基於 FQDN, 應用程式類型)      |
| **DNAT (目的地 NAT)**| 支援                                        | 支援                                           | 支援                                            |
| **SNAT (來源 NAT)** | 支援                                        | 支援                                           | 支援                                            |
| **威脅情報** | 支援 (警示模式)                           | 支援 (警示模式)                              | 支援 (警示和拒絕模式)                         |
| **TLS 檢查 (TLS Inspection)** | 不支援                                      | 不支援                                         | 支援 (解密和重新加密 TLS 流量以進行檢查)      |
| **入侵偵測與防護 (IDPS)** | 不支援                                      | 不支援                                         | 支援 (根據已知惡意活動的簽章進行偵測與阻擋)   |
| **網址篩選 (URL Filtering)** | 不支援                                      | 不支援                                         | 支援 (篩選 HTTP/HTTPS 流量中的網址)            |
| **Web 分類 (Web Categories)** | 不支援                                      | 不支援                                         | 支援 (允許或拒絕存取特定類別的網站，如賭博網站) |
| **集中管理** | 可透過 Azure Firewall Manager 管理          | 可透過 Azure Firewall Manager 管理             | 可透過 Azure Firewall Manager 管理              |

請注意：這是一個簡化的比較表，實際功能和限制請參考 Azure 官方文件以獲得最準確的資訊。

### - Azure Firewall 三種規則差異（Application、Network、DNAT）

Azure Firewall 主要使用三種類型的規則來控制網路流量，它們在檢查和過濾流量的層級和目的上有所不同：

**應用程式規則 (Application Rules)**：

* **作用層級**：基於應用程式層 (L7)，例如 HTTP, HTTPS, MSSQL。
* **過濾依據**：允許您根據完全合格網域名稱 (FQDN)（例如 www.google.com）、URL 或應用程式類型（例如 Azure Update, Windows Updates）來允許或拒絕流量。
* **目的**：用於控制應用程式層級的輸出流量存取。例如，允許虛擬機器存取特定的網站或雲端服務端點。
* **NAT**：應用程式規則執行 SNAT (來源 NAT)，將來自您內部網路的私人 IP 位址轉換為防火牆的公用 IP 位址，以便流量能夠路由到網際網路。DNAT 不適用於應用程式規則。

**網路規則 (Network Rules)**：

* **作用層級**：基於網路層 (L3) 和傳輸層 (L4)，例如 IP, TCP, UDP。
* **過濾依據**：允許您根據來源 IP 位址/範圍、目的地 IP 位址/範圍、通訊協定 (如 TCP, UDP, ICMP, Any) 和目的地埠來允許或拒絕流量。
* **目的**：用於控制網路層級的流量，包括輸出、輸入以及 VNet 之間的流量。例如，允許特定子網路的機器通過 SSH (TCP 埠 22) 連線到另一個子網路的伺服器，或允許從內部網路向外部特定 IP 位址進行 DNS 查詢 (UDP 埠 53)。
* **NAT**：網路規則通常執行 SNAT，類似於應用程式規則。

**DNAT 規則 (Destination NAT Rules)**：

* **作用層級**：基於網路層 (L3) 和傳輸層 (L4)。
* **過濾依據**：允許您根據來源 IP 位址/範圍、通訊協定 (如 TCP, UDP)、目的地埠和目的地 IP 位址（防火牆的公用 IP）來匹配傳入流量。
* **目的**：用於將傳入到防火牆公用 IP 位址和特定埠的流量，轉換其目的地 IP 位址和埠，重新導向到內部虛擬網路中的特定資源（例如，將傳入到防火牆公用 IP 的 RDP 流量導向到內部伺服器的私人 IP）。
* **NAT**：DNAT 規則執行目的地 NAT，將流量的目的地 IP 位址和埠從防火牆的公用 IP 和埠轉換為內部資源的私人 IP 和埠。DNAT 規則也隱含地執行 SNAT，以便內部資源的響應流量能夠正確地返回到原始的外部來源。

**總結來說：**

* **應用程式規則**：用於基於 FQDN/URL/應用程式類型的**輸出**流量控制。
* **網路規則**：用於基於 IP/Port/Protocol 的**輸出、輸入和 VNet 間**流量控制。
* **DNAT 規則**：用於將傳入到防火牆**公用 IP** 的流量**重新導向**到內部資源。

Azure Firewall 會按照優先順序處理規則，首先檢查 DNAT 規則，然後是網路規則，最後是應用程式規則。流量匹配到允許規則後會被放行，如果沒有匹配到任何允許規則且沒有明確的拒絕規則，則會根據預設行為（通常是拒絕）來處理。

## 延伸學習

- [Azure Firewall 簡介](https://learn.microsoft.com/training/modules/introduction-azure-firewall/)  
- [Azure Firewall Manager 簡介](https://learn.microsoft.com/training/modules/intro-to-azure-firewall-manager/)

## 重點整理

- Azure Firewall 是雲端防火牆服務，常與 Hub-Spoke 網路架構搭配使用。  
- 提供 Application、Network、NAT 三種規則。  
- 支援 Standard、Premium、Basic 三種 SKU。
