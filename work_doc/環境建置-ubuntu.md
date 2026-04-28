# Ubuntu 環境建置紀錄

實戰整理：在 Ubuntu 22.04 從零安裝本論文模板編譯環境的完整流程，含已知坑點與處理方式。

## 系統需求

- Ubuntu 22.04（其他版本步驟類似，套件名稱相同）
- 約 6 GB 磁碟空間（`texlive-full` 解壓後約 5 GB）
- 網路連線（下載 TeX Live 與微軟字型）

確認版本：

```bash
lsb_release -a
```

## 一、安裝 TeX Live 與 make

```bash
sudo apt update
sudo apt install -y texlive-full make
```

下載量大、耗時 10–30 分鐘。安裝過程結尾會跑 `Pregenerating ConTeXt MarkIV format` — 詳見下一節。

### 驗證

```bash
which xelatex bibtex make
xelatex --version
```

預期：`/usr/bin/xelatex`、`/usr/bin/bibtex`、`/usr/bin/make` 皆存在。

## 二、處理 ConTeXt postinst 卡死

### 症狀

`apt install texlive-full` 進到最後階段時可能卡在：

```
Pregenerating ConTeXt MarkIV format. This may take some time...
Progress: [ 99%] [#########################################################.]
```

進度條停在 99% 不動超過 5 分鐘，且 `luatex` 程序 CPU 為 0%（用 `ps` 檢查）。

### 原因

`context` 套件的 post-install 腳本在某些環境下會 deadlock。**ConTeXt 是另一個排版系統，本論文模板用不到它**，可以直接跳過。

### 處理步驟

1. 找出卡住的 luatex 程序：

   ```bash
   pgrep -af 'mtxrun|context|luatex'
   ```

2. 砍掉相關程序（PID 換成上一步看到的）：

   ```bash
   sudo kill <luatex_pid> <mtxrun_pid> <context.postinst_pid>
   # 30 秒內若無反應再加 -9
   sudo kill -9 <pids>
   ```

3. 此時 apt 視窗會回報 `context` 設定失敗（連帶 `context-modules`、`texlive-full` meta-package 也標記未設定），但**其他 TeX 元件都已安裝完成**。

4. 移除 ConTeXt 並收尾 dpkg 狀態：

   ```bash
   sudo apt-get remove --purge -y context context-modules
   sudo dpkg --configure -a
   ```

   這一步也會重建一次 format 檔（不含 ConTeXt）。

> 注意：`texlive-full` 是 meta-package，被一起移除不影響實際使用 — 真正裝起來的子套件（`texlive-xetex`、`texlive-lang-chinese` 等）都還在。

## 三、安裝 Times New Roman

`main.tex` 指定英文字型為 `Times New Roman`，Linux 預設沒有此字型。

```bash
sudo apt install -y ttf-mscorefonts-installer
```

安裝過程會跳出微軟 EULA 接受畫面，按 Tab 移到「OK / 同意」後 Enter。需要網路下載字型檔。

### 驗證

```bash
fc-match "Times New Roman"
# 預期輸出：Times_New_Roman.ttf: "Times New Roman" "Regular"
```

若 `fc-match` 回傳的是其他字型（如 `DejaVu Serif`），表示 EULA 未同意或下載失敗，重新跑 `sudo dpkg-reconfigure ttf-mscorefonts-installer`。

## 四、標楷體（DFKai-SB）

**不需另外安裝** — `DFKai-SB.ttf` 已包在專案根目錄。XeLaTeX 透過 `\setCJKmainfont{DFKai-SB.ttf}` 從當前工作目錄讀取。

確認檔案存在：

```bash
ls -la DFKai-SB.ttf
```

## 五、編譯

```bash
make            # 等同 ./build.sh：xelatex → bibtex → xelatex
make            # 再跑一次，讓目錄頁碼與交叉參照收斂
```

### 為什麼要跑兩次

`build.sh` 內建 3 步流程（xelatex → bibtex → xelatex），第一次完成後仍會出現：

- `Label(s) may have changed. Rerun to get cross-references right.`
- `File 'main.out' has changed. Rerun to get outlines right`

再執行一次 `make`（多跑一輪 xelatex）即可收斂。

### 驗證成品

```bash
pdfinfo main.pdf | grep Pages
ls -la main.pdf
```

預期：60–61 頁、約 2.3 MB。

### 已知無害警告

下列訊息可忽略，PDF 內容仍正確：

- `Package biblatex Warning: Please (re)run BibTeX` — biblatex 用 BibTeX 後端的循環提示
- `Package biblatex Warning: Using fall-back BibTeX(8) backend` — 同上
- `Package caption Warning: Unused \captionsetup[sub]` — 未使用的 subcaption 設定

## 故障排除速查

| 症狀 | 原因 | 處理 |
|------|------|------|
| `xelatex: command not found` | TeX Live 未安裝完成 | 重跑步驟一 |
| `! Package fontspec Error: The font "Times New Roman" cannot be found` | 微軟字型未安裝 | 步驟三 |
| `! Package fontspec Error: The font "DFKai-SB.ttf" cannot be found` | 不在專案根目錄編譯 | `cd` 到 repo 根目錄再跑 |
| 目錄頁碼錯一頁 | 只跑了一次 `make` | 再跑一次 `make` |
| 安裝卡在 ConTeXt 99% | postinst 死結 | 步驟二 |
