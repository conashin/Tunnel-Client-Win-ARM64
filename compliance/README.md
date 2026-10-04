# compliance/

這個目錄存放「授權閘門」使用的基準值。每次建置時，workflow 會把上游原始碼的授權文件與這裡的基準值比對，不符就**不發布**，並開出 `license-review` issue 等待人工判斷。

## 檔案

| 檔案 | 用途 |
|---|---|
| `upstream-LICENSE.sha256` | 上游 repo 根目錄 `LICENSE`（Apache License 2.0 全文）的 SHA256 基準值，格式同 `sha256sum` 輸出。 |
| `upstream-NOTICE.sha256` | 上游 `NOTICE` 檔的 SHA256 基準值。**目前上游沒有 NOTICE，所以這個檔案刻意不存在。** 上游新增 NOTICE 後，須經人工確認內容才能建立。 |

## 比對規則

- 上游 `LICENSE` 的 SHA256 與 `upstream-LICENSE.sha256` 不符 → 閘門失敗。
- 上游出現 `NOTICE*` 檔，但 `upstream-NOTICE.sha256` 不存在或雜湊不符 → 閘門失敗（視為「NOTICE 變更」）。
- 依賴套件出現 forbidden、restricted 或無法辨識的授權（`go-licenses check`）→ 閘門失敗。**不得以放寬 `--disallowed_types` 的方式讓檢查通過。**

## 更新基準值的流程

1. 由 `license-review` issue 觸發，並在 issue 中取得維護者的明確同意。
2. 取得新舊內容的 diff（例如 `diff <舊 LICENSE> <新 LICENSE>`），並確認新內容仍是未修改的 Apache-2.0 全文（可與 <https://www.apache.org/licenses/LICENSE-2.0.txt> 比對）。若是 NOTICE，則須確認其內容並同步更新本 repo 的 `NOTICE`。
3. **一定要透過 Pull Request** 更新基準檔，並在 PR 描述附上新舊內容的 diff 與比對依據。
4. 經維護者核准並 merge 後，重新觸發建置。
