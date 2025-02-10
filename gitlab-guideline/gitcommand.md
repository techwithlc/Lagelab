### English Version

#### Step 1: Clone the Repository
1. Go to the GitLab project page.
2. Click the "Clone" button and copy the repository URL (HTTPS or SSH).
3. Open your terminal or command prompt and run:
   ```bash
   git clone <repository-url>
   Git clone https://gitlab.com/azure50402/Lagelab.git
   ```
   Replace `<repository-url>` with the URL you copied.

---

#### Step 2: Go to Branch
Before making changes, create a new branch to isolate your work:
```bash
git checkout your-branch-name
git checkout dev
```
Replace `your-branch-name` with a descriptive name for your branch.

---

*git checkout -b dev origin/dev
Switch local branch name to follow remote dev branch

*Git fetch
Get remote file to local on current branch

git branch -a
List local and remote branch
Git branch
List local branch


---

#### Step 4: Make Your Changes
1. Open the cloned repository in your code editor.
2. Make the necessary changes to the files.

---

#### Step 5: Stage Your Changes
After making changes, stage them for commit:
```bash
git add .
```
This stages all changes. You can also stage specific files by replacing `.` with the file name.

---

#### Step 6: Commit Your Changes
Commit your changes with a meaningful message:
```bash
git commit -m "Your commit message"
```
Replace `"Your commit message"` with a brief description of your changes.

---

#### Step 7: Push Your Changes to GitLab
Push your branch to the remote repository:
```bash
git push origin your-branch-name
```
Replace `your-branch-name` with the name of your branch.

---

#### Step 8: Create a Merge Request (MR)
1. Go to the GitLab project page.
2. Click on "Merge Requests" and then "New Merge Request."
3. Select your branch as the source and the main branch as the target.
4. Add a title and description for your MR, then submit it.

---

#### Step 9: Wait for Review
Your changes will be reviewed by the maintainers. Once approved, your contribution will be merged into the main branch.

---

### Chinese Version


#### Git 和 GitLab 新手指南：如何將貢獻推送到倉庫

Git 是一個強大的版本控制系統，幫助開發者追蹤程式碼庫的變更。GitLab 是一個基於 Web 的平台，使用 Git 進行版本控制，並提供問題追蹤、CI/CD 流水線等附加功能。以下是為初學者準備的逐步指南，教你如何將貢獻推送到 GitLab 倉庫。

---

#### 第一步：安裝 Git
如果你還沒有安裝 Git，請從官方網站下載並安裝：  
[https://git-scm.com/](https://git-scm.com/)

---

#### 第二步：克隆倉庫
1. 打開 GitLab 專案頁面。
2. 點擊 "Clone" 按鈕，複製倉庫的 URL（HTTPS 或 SSH）。
3. 打開終端或命令提示符，運行以下命令：
   ```bash
   git clone <repository-url>
   ```
   將 `<repository-url>` 替換為你複製的 URL。

---

#### 第三步：建立新分支
在開始修改之前，建立一個新分支來隔離你的工作：
```bash
git checkout -b your-branch-name
```
將 `your-branch-name` 替換為你的分支名稱。

---

#### 第四步：進行修改
1. 在程式碼編輯器中打開克隆的倉庫。
2. 對檔案進行必要的修改。

---

#### 第五步：暫存修改
完成修改後，將變更暫存：
```bash
git add .
```
這將暫存所有變更。你也可以通過將 `.` 替換為檔案名稱來暫存特定檔案。

---

#### 第六步：提交修改
使用有意義的提交訊息提交你的變更：
```bash
git commit -m "Your commit message"
```
將 `"Your commit message"` 替換為對你變更的簡要描述。

---

#### 第七步：將變更推送到 GitLab
將你的分支推送到遠端倉庫：
```bash
git push origin your-branch-name
```
將 `your-branch-name` 替換為你的分支名稱。

---

#### 第八步：建立合併請求 (MR)
1. 打開 GitLab 專案頁面。
2. 點擊 "Merge Requests"，然後點擊 "New Merge Request"。
3. 選擇你的分支作為來源分支，主分支作為目標分支。
4. 為你的 MR 添加標題和描述，然後提交。

---

#### 第九步：等待審核
你的變更將由維護者審核。審核通過後，你的貢獻將被合併到主分支。

---
