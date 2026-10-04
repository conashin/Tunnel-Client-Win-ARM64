# Windows ARM64 Tunnel Client Builds（非官方）

Unofficial Windows ARM64 builds of the cloudflared client, for use with Cloudflare Tunnel™.

本專案為個人維護的非官方建置，與 Cloudflare, Inc. 無關，亦未經其認可、贊助或背書。

This is an unofficial, personally maintained project. It is not affiliated with, endorsed by, or sponsored by Cloudflare, Inc.

## 用途

上游官方 release 只提供 `windows-386` 與 `windows-amd64`，沒有 `windows-arm64`。本專案提供原生 `windows/arm64` 的 `cloudflared.exe`，讓 Windows ARM64 裝置可以使用 cloudflared client，例如在 SSH 的 `ProxyCommand` 中執行 `cloudflared access ssh`。

## 運作方式

- 每天自動檢查上游 [`cloudflare/cloudflared`](https://github.com/cloudflare/cloudflared) 是否有新的正式版。
- 發現新版時，從該版 tag 的**上游原始碼原封不動**交叉編譯（`GOOS=windows GOARCH=arm64 CGO_ENABLED=0`）。本專案不修改任何上游原始碼，也不是 fork，repo 內只有 workflow 與文件。
- 編譯後在原生 Windows ARM64 runner（`windows-11-arm`）上做 smoke test：確認 PE 檔頭為 ARM64、`--version` 與 `access --help` 可執行、自動更新已停用。
- 通過後，把執行檔連同授權文件打包成 zip，發布為 GitHub Release，並附上建置來源證明（artifact attestation）。
- 發布前會做授權檢查：上游 `LICENSE` 與 `NOTICE`、以及所有被編入的第三方套件授權，任一項異常就不發布，改開 issue 等待人工判斷。

## 下載與使用

1. 到 [Releases](../../releases) 下載 `winarm64-build-<版本>.zip`。
2. 解壓縮，使用其中的 `cloudflared.exe`（保留上游原名，因為 SSH 設定會依賴這個名稱）。
3. 在 SSH 設定（`%USERPROFILE%\.ssh\config`）中以**完整路徑**指定：

   ```
   Host example
     HostName ssh.example.com
     ProxyCommand C:\Tools\winarm64-build\cloudflared.exe access ssh --hostname %h
   ```

   若路徑含空白，需加上雙引號。

zip 內容：

```
cloudflared.exe
LICENSE                      # 上游 Apache-2.0 全文
NOTICE                       # 本專案的 NOTICE
UPSTREAM-NOTICE              # 上游 NOTICE（僅在上游有時才會出現）
third-party-licenses.csv
THIRD_PARTY_LICENSES/        # 第三方套件授權全文，含 go/LICENSE
```

## 驗證

每個 release 都附有 `.sha256` 檔，也有 GitHub artifact attestation。

```powershell
# SHA256（應與 .sha256 檔及 release notes 一致）
Get-FileHash -Algorithm SHA256 .\winarm64-build-<版本>.zip

# 建置來源證明（需要 GitHub CLI）
gh attestation verify winarm64-build-<版本>.zip --repo conashin/Tunnel-Client-Win-ARM64
```

## 已知差異

與官方版本相比：

- **Windows ICMP proxy 停用。** 該功能需要 cgo，本專案以 `CGO_ENABLED=0` 編譯（官方 `windows-386` 版本也是如此）。這只影響「在 Windows 上作為 tunnel connector 時代理 ICMP」；client 端的 `cloudflared access ssh` 不受影響。
- **自動更新停用。** 內建更新器會以錯誤的架構覆蓋執行檔，所以已透過 ldflags 停用。請從本專案的 Releases 取得新版。
- **執行檔沒有程式碼簽章。** Windows SmartScreen 可能跳出警告，請自行以上述方式驗證檔案後再決定是否執行。

## 手動觸發重建

到 Actions 頁面選擇 `winarm64-build`，按下 **Run workflow**。可在 `tag` 欄位輸入要建置的上游版本（格式 `YYYY.M.P`，例如 `2026.8.0`），留空代表最新版。已存在同名 release 的 tag 不會被覆寫，也不會重複發布。

## 相關的上游 issue

- [cloudflare/cloudflared#1172](https://github.com/cloudflare/cloudflared/issues/1172)
- [cloudflare/cloudflared#1515](https://github.com/cloudflare/cloudflared/issues/1515)

若上游日後官方提供 `windows-arm64`，本專案將不再需要，屆時會封存。

## 關於本專案的編寫

本 repo 的 workflow、文件與合規檔案由 **Claude**（Anthropic 開發的 AI 助理）編寫。所有變更都經由 Pull Request 提出，並由維護者審閱、核准後才合併；Claude 不會自行 merge 或發布。

This repository's workflows, documentation and compliance files were written by Claude, an AI assistant made by Anthropic. Every change goes through a pull request and is reviewed and approved by the maintainer before it is merged.

## 授權

- **本 repo 內的檔案**（workflow、文件等）以 [Apache License 2.0](LICENSE) 授權，Copyright 2026 conashin。
- **release 中的 `cloudflared.exe`** 是由上游未經修改的原始碼編譯而來，依上游的 Apache License 2.0 散布（Copyright Cloudflare, Inc.）。
- **第三方套件**：執行檔靜態連結了 Go 標準函式庫與多個第三方套件。其授權全文（含 Go 的 BSD-3-Clause `LICENSE`）收錄在每個 release zip 內的 `THIRD_PARTY_LICENSES/`，套件清單見 `third-party-licenses.csv`。
- Apache-2.0 第 6 條不授予商標權。詳見 [NOTICE](NOTICE)。

---

Cloudflare, Cloudflare Tunnel, Cloudflare One and WARP are trademarks and/or registered trademarks of Cloudflare, Inc. in the United States and other jurisdictions. The cloudflared source code is distributed under the Apache License 2.0.
