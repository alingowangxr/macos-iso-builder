# macOS ISO Builder

直接從 Apple 官方伺服器製作可開機的 macOS 安裝 ISO 與 DMG 映像——無需 Mac。

[English](README.md) | 繁體中文

## 專案概述

本專案包含兩個部分：

1. **`mkmaciso` 腳本** — 僅使用 macOS 內建工具與指令，從 Apple 伺服器下載完整的 macOS 安裝程式至 **/Applications**，然後製作可開機的 ISO/DMG 映像。
2. **GitHub Actions 工作流程** — 如果你沒有 Mac，可以透過 GitHub Actions 在 Azure 資料中心託管的 Mac mini 上執行 `mkmaciso`。

## 開始之前

請先至 [Release 頁面](https://github.com/LongQT-sea/mkmaciso/releases/latest) 檢查——可能已經有人構建了你需要的映像。如果沒有，請參閱[使用方法](#使用方法)自行構建。

> [!Important]
> GitHub 托管的執行器是免費的公共資源——請合理使用。

## 磁碟映像格式比較

| | ISO | DMG |
|---|---|---|
| 最適合 | 虛擬機 | 可開機 USB 隨身碟 |
| 虛擬機支援 | 掛載為虛擬光碟機 | 掛載為虛擬硬碟 |
| 分割區配置 | 混合式 UDF/HFS | 原始 GPT 磁碟映像 |

**ISO 檔案** — 非常適合虛擬機使用（*Proxmox、QEMU、VirtualBox、VMware*）。只需將其掛載為虛擬光碟機即可。甚至可以在 Windows 中掛載以便查看內容。

**DMG 檔案** — 使用 [Rufus](https://rufus.ie/en/#download)（*Windows*）、`dd`（*Linux*）或 `asr`（*macOS*）將其寫入 USB 隨身碟，製作可開機的安裝媒體。若要在虛擬機中使用，可透過 `qemu-img` 轉換為 `.vhd`（*Hyper-V 用*）或 `.vmdk`（*VMware 用*）。QEMU/Proxmox 可直接使用原始磁碟映像，無需轉換。

> [!NOTE]
> DMG 檔案的副檔名為 `.img`（例如 `macOS_Sequoia.dmg.img`），這樣 Rufus 就能在檔案總管中直接辨識，無需切換為「所有檔案」模式。

## 支援的 macOS 版本

從 OS X Lion（10.7，2011 年）到最新的 macOS Tahoe（26，2025 年），幾乎涵蓋所有主要版本。

| 版本號 | 代號 | 發行年份 |
|---|---|---|
| 10.7 | Lion | 2011 |
| 10.8 | Mountain Lion | 2012 |
| 10.9 | Mavericks | 2013 |
| 10.10 | Yosemite | 2014 |
| 10.11 | El Capitan | 2015 |
| 10.12 | Sierra | 2016 |
| 10.13 | High Sierra | 2017 |
| 10.14 | Mojave | 2018 |
| 10.15 | Catalina | 2019 |
| 11 | Big Sur | 2020 |
| 12 | Monterey | 2021 |
| 13 | Ventura | 2022 |
| 14 | Sonoma | 2023 |
| 15 | Sequoia | 2024 |
| 26 | Tahoe | 2025 |

---

## 使用方法

### 沒有 Mac？使用 GitHub Actions

> [!TIP]
> <details>
> <summary>點此觀看視覺化操作指南（GIF）</summary>
>
> ![How to fork and run workflow](https://raw.githubusercontent.com/LongQT-sea/macos-iso-builder/main/.github/how_to_fork_and_run_workflow.gif)
>
> </details>

1. [點此](https://github.com/LongQT-sea/macos-iso-builder/fork) Fork 本儲存庫（需要 GitHub 帳號）。
2. 在你 Fork 的儲存庫中，前往 **Actions** 分頁。
3. 點選綠色的 **"I understand my workflows, go ahead and enable them"** 按鈕。
4. 從左側邊欄選擇工作流程：
   * **Recovery ISO**（*建議*）— 較小的還原映像，構建時間約 2-5 分鐘。適合虛擬機使用。
   * **Full Installer** — 完整的離線安裝程式，大小 5-18GB，構建時間約 5-60 分鐘。
5. 點選 **"Run workflow"** 按鈕並設定工作流程參數：
   * **macOS version** — 選擇版本（*Sequoia*、*Sonoma* 等）。
   * **Image format** — 選擇 `iso`（虛擬機用）或 `dmg`（可開機 USB 隨身碟用）。
6. 點選綠色的 **"Run workflow"** 按鈕開始構建，然後等待工作流程完成。
7. 完成後，重新載入頁面並向下捲動至 **Artifacts** 區域。點選產物連結開始下載（例如 `macOS_Sequoia_15.7.4.iso`）。
8. **Recovery ISO** 產物為壓縮檔——使用前請先解壓縮。

---

### 有 Mac？本地執行 `mkmaciso`

使用 Terminal.app 快速執行（將 `tahoe` 替換為你想要的版本）：
```bash
curl -s https://raw.githubusercontent.com/LongQT-sea/mkmaciso/main/mkmaciso | bash -s tahoe
```

或先下載腳本，再以參數執行：
```bash
curl -O https://raw.githubusercontent.com/LongQT-sea/mkmaciso/main/mkmaciso
chmod +x mkmaciso
./mkmaciso --help
```

執行 `./mkmaciso` 不帶參數會顯示互動式選單。

---

### 指令列參數

```
./mkmaciso [版本] [格式] [輸出路徑]
```

| 參數 | 說明 | 範例 |
|---|---|---|
| 版本 | macOS 版本號或代號 | `14`、`sonoma`、`10.13`、`highsierra` |
| 格式 | 輸出映像格式 | `iso`（預設）、`dmg` |
| 輸出路徑 | 自訂輸出檔案位置 | `~/Desktop/out.iso` |

**支援的版本參數：**

| 版本號 | 代號 |
|---|---|
| `10.7` | `lion` |
| `10.8` | `mountainlion` |
| `10.9` | `mavericks` |
| `10.10` | `yosemite` |
| `10.11` | `elcapitan` |
| `10.12` | `sierra` |
| `10.13` | `highsierra` |
| `10.14` | `mojave` |
| `10.15` | `catalina` |
| `11` | `bigsur` |
| `12` | `monterey` |
| `13` | `ventura` |
| `14` | `sonoma` |
| `15` | `sequoia` |
| `26` | `tahoe` |

**使用範例：**
```bash
./mkmaciso 14              # 建立 Sonoma ISO
./mkmaciso 14 dmg          # 建立 Sonoma DMG
./mkmaciso sonoma dmg ~/Desktop/out.dmg  # 自訂輸出路徑
./mkmaciso 10.13 iso       # 建立 High Sierra ISO
```

---

## 使用技巧與注意事項

### 虛擬機使用建議

- 只需將 ISO 掛載為虛擬光碟機即可。
- **Proxmox 用戶** — 若想要更好的效能，可考慮 GPU 直通。本作者的另一個儲存庫 [OpenCore-ISO](https://github.com/LongQT-sea/OpenCore-ISO) 可能對安裝過程有所幫助，另有專門用於 [Intel iGPU 直通](https://github.com/LongQT-sea/intel-igpu-passthru)的資源。
- QEMU/Proxmox 可直接使用 DMG 原始磁碟映像，無需轉換格式。

### 可開機 USB 隨身碟

- 將 DMG 寫入 USB 隨身碟後，磁碟上會有剩餘空間。如有需要，可用該空間建立 FAT32 分割區存放 EFI 資料夾。
- 在 Linux 上使用 `dd` 指令時，請務必再三確認目標裝置。`dd` 不會要求確認。

### 常見問題與故障排除

| 問題 | 可能原因 | 解決方法 |
|---|---|---|
| 下載失敗或中斷 | 網路不穩定或 Apple 伺服器問題 | 重新執行腳本，支援斷點續傳 |
| 磁碟空間不足 | 建構過程需要 20-40GB 空間 | 清理磁碟空間後重試 |
| Apple Silicon 上無法使用 `createinstallmedia` | 較舊版本的 macOS 不支援 ARM 架構 | 腳本會自動使用替代方法 |
| GitHub Actions 超時 | 完整安裝程式構建時間較長 | 考慮使用 Recovery ISO，或在本地執行 |
| DMG 檔案在 Rufus 中看不到 | 副檔名為 `.img` | 在 Rufus 中選擇「所有檔案」或直接使用 `.dmg.img` 副檔名 |
| ISO 掛載後無法在 Windows 中讀取 | 格式相容性問題 | 使用 7-Zip 或 HFS+ 讀取工具 |

### 磁碟空間注意事項

- **Recovery ISO**：構建過程約需 2-5GB 臨時空間
- **完整安裝程式 ISO/DMG**：構建過程約需 20-40GB 臨時空間
- 腳本會在開始前檢查可用空間，不足時會提示錯誤

---

## 系統需求

### 本地執行 `mkmaciso`

| 需求 | 說明 |
|---|---|
| 作業系統 | macOS 10.9（Mavericks）或更新版本；構建 10.13+ 映像建議使用 macOS 11（Big Sur）或更新版本 |
| 處理器架構 | Intel Mac 為推薦；Apple Silicon 可用但有限制（無法對較舊版本使用 `createinstallmedia`，腳本會自動使用替代方法） |
| 可用磁碟空間 | 建構過程中需 20-40GB |
| 網路連線 | 需要（從 Apple 伺服器下載） |
| 權限 | 需要 `sudo` 權限 |

### 使用的內建工具

本腳本僅使用 macOS 內建工具，無需安裝任何額外軟體：

- `hdiutil` — 磁碟映像操作
- `softwareupdate` — 下載現代版本的 macOS
- `curl` — 下載舊版安裝程式
- `createinstallmedia` — Apple 官方的安裝媒體建立工具（10.12+）
- `diskutil` — 磁碟管理
- `openssl` — Mavericks 下載認證
- `xxd`、`od`、`iconv` — 二進位/文字處理工具
- `pkgutil`、`cpio` — Apple Silicon 上的舊版套件解壓縮
- `codesign`、`xattr` — Apple Silicon 相容性處理

### GitHub Actions（無需本地環境）

- 只需一個 GitHub 帳號即可 Fork 並執行工作流程
- 所有構建在 GitHub 托管的 Mac 執行器上完成

---

## 技術細節

### 各版本下載策略

| 版本範圍 | 下載方式 |
|---|---|
| 10.7 - 10.8（Lion / Mountain Lion） | 直接從 Apple CDN 下載 DMG |
| 10.9（Mavericks） | 透過恢復伺服器協定下載，需要硬體認證 |
| 10.10 - 10.11（Yosemite / El Capitan） | 從 Apple 軟體更新目錄下載各元件 |
| 10.12 - 10.15（Sierra - Catalina） | 從 Apple 軟體更新目錄下載 BaseSystem.dmg 等元件 |
| 11+（Big Sur 及更新版本） | 使用 `softwareupdate --fetch-full-installer` 指令 |

### 映像建立策略

| 版本範圍 | ISO 建立方式 | DMG 建立方式 |
|---|---|---|
| 10.7 - 10.8 | `hdiutil makehybrid` | `hdiutil create` |
| 10.9 - 10.11 | `hdiutil makehybrid` | `hdiutil create` |
| 10.12+ | `hdiutil makehybrid` + `createinstallmedia` | `hdiutil create` + `createinstallmedia` |

### Apple Silicon 限制

在 Apple Silicon Mac 上：
- 無法對較舊版本（10.12 以前）使用 `createinstallmedia`
- 腳本會自動偵測架構並使用替代方法（如直接操作 BaseSystem.dmg）
- 部分舊版安裝程式可能需要額外的簽名與屬性設定

---

## 致謝

- [Apple](https://www.apple.com/) 提供 macOS 及其更新伺服器
- [Mavericks Forever](https://mavericksforever.com/) 記錄了 Mavericks 恢復協定
- [InsanelyMac 社群](https://www.insanelymac.com/forum/topic/338810-create-legit-copy-of-macos-from-apple-catalog/) 對於直接從 Apple 目錄下載 macOS 的研究

## 法律聲明

本工具直接從 Apple 官方伺服器下載 macOS 映像。使用者應自行負責遵守 [Apple 軟體授權協議](https://www.apple.com/legal/sla/)。macOS 是 Apple Inc. 的商標。

## 許可證

本專案採用 [GNU 通用公共授權條款 v3（GPLv3）](LICENSE) 授權。
