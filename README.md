# NIKKE PC 啟動器木馬檢查表（Windows）

資料更新：2026-11-04 13:00（台灣時間）。事件還在發展，請以官方後續公告為準。

## 先做這三件事

**9/27 之後在電腦上開過 NIKKE 啟動器的人，不管有沒有中，都建議先做：**

1. **官方確認修好之前，不要開 PC 版啟動器。**
2. **用手機改 NIKKE（Level Infinite）帳號密碼，並登出所有裝置。** 同一組密碼也用在別處的，一起換掉。
3. **跑一次[一鍵檢查](WINDOWS-CHECKS.md#一鍵檢查)**，照結果決定下一步。

> 為什麼沒中也要改密碼？木馬只在「部分啟動」時才送來，而且會刪掉自己的暫存檔，檢查不出來不代表 100% 沒事，而改密碼的成本很低。研究者 MotherSphere 建議 10/23 之後開過啟動器就改；巴哈整理文更保守，從 9/27 算起。這份表採用 9/27。

**在 `Launcher\intl_service\debug.log` 裡看到 `k2f_chan`，就是中了。** 日期在 10 月、沒裝 Chrome、找不到 DLL，都一樣算。直接看[中毒了怎麼辦](#中毒了怎麼辦)。

| 我想… | 看這裡 |
|---|---|
| 自己檢查 | [WINDOWS-CHECKS.md](WINDOWS-CHECKS.md)：一鍵檢查、手動檢查、封鎖網域 |
| 請 AI 助手幫忙 | [AGENT-PROMPT.md](AGENT-PROMPT.md)：可直接複製的提示詞 |
| 已經確定中了 | [中毒了怎麼辦](#中毒了怎麼辦) |

## 發生了什麼事

NIKKE PC 啟動器裡有一個內建的網頁畫面，每次開啟都會從 Level Infinite 的 CDN 載入 JavaScript。9/27 有人改了 CDN 上的 4 個 JS 檔（檔名沒變），讓啟動器多去載入 `insightjs[.]com` 的腳本。有些啟動時，攻擊者會再送來一個遠端控制木馬，用啟動器的下載功能下載並執行。

- **只要開啟動器就可能中。** 不用點廣告、不用登入遊戲，跟有沒有裝 Chrome 也無關。
- **不是每次都中。** 有人開了約 19 次，中了 3 次。所以同樣有開，有人中、有人沒中。
- **木馬在啟動器開著時能做的事：** 執行任何指令、讀檔、截圖，偷 NIKKE 帳密、各家瀏覽器（Chrome、Edge、Firefox 等）存的密碼和登入 Cookie、SSH 與 API 金鑰、加密錢包資料。部分電腦還被裝上寄生在 Chrome 的常駐 DLL。
- **紀錄只能證明木馬跑過**，看不出哪些資料被拿走了。

### 官方說法

11/4 官方承認啟動器腳本「可能被第三方竄改」，說已在 **11/4 01:30（台灣時間）左右** 完成客戶端修復與更新，影響範圍還在確認，後續處理方式會另外公告。國際版發在 [X](https://x.com/NIKKE_en/status/2117817602659770434)，台港澳官方粉絲團同日也有發布（[巴哈整理文](https://forum.gamer.com.tw/C.php?bsn=36390&snA=23304)有引用全文）。

官方修好的是「以後」不會再被感染。**之前已經中的電腦，不會因為更新就變乾淨。**

### 時間線（台灣時間）

| 日期 | 發生什麼 |
|---|---|
| 9/27 | `insightjs[.]com` 被註冊；同一天 CDN 上 4 個 JS 檔被改 |
| 10/24 00:40 | 目前公開最早的木馬執行紀錄。另有玩家回報 9/30、10/15 的 DNS 紀錄就出現這個網域，實際開始時間可能更早 |
| 10/28 | `FeedList.feb647c9.js` 在源站被清掉，但部分 CDN 節點還在送舊檔 |
| 11/4 01:04–01:21 | 另外 3 個被改的檔案換回乾淨版本 |
| 11/4 02:31 | 攻擊者的木馬伺服器停止回應 |
| 11/4 06:26、10:01–10:07 | 研究者仍從部分 CDN 節點拿到被改過的 `FeedList.feb647c9.js` |
| 11/4 11:13 | 國際版官方在 X 發布公告 |

木馬伺服器停了，現在的風險比前幾天低。但 CDN 還沒完全清乾淨，`insightjs[.]com` 也不在官方手上，**還是先別開啟動器**。

## 結果怎麼判讀

[一鍵檢查](WINDOWS-CHECKS.md#一鍵檢查)最後一行會顯示結果：

| 結果 | 代表什麼 | 怎麼做 |
|---|---|---|
| **中毒** | 木馬在這台電腦跑過 | 照下面的[中毒了怎麼辦](#中毒了怎麼辦)處理 |
| **有接觸** | 啟動器載入過攻擊腳本，但沒找到木馬執行的痕跡 | **帳號照中毒處理**（第 2、3 步）。電腦本身沒有中毒證據，不用急著重灌 |
| **資料不足** | 找不到 `debug.log`、重裝過，或紀錄沒涵蓋你玩的日期 | 無法判斷，帳號照「有接觸」處理 |
| **沒找到已知痕跡** | 已知的痕跡都沒出現 | 做完[先做這三件事](#先做這三件事)，等官方公告 |

**以下任何一項出現，就算中毒：**

- `debug.log` 裡有 `k2f_chan`、`lua downloaded to` 或 `agent started as`
- 安裝資料夾的 `Launcher\log\` 裡出現 `insight.bundle`、`analytics.insightjs.com`，或有 `k2f.lua`、LuaJIT 格式的檔案
- 啟動器資料 `%APPDATA%\nikke_launcher*` 裡出現 `k2f_chan` 或 `k2f.agent`（例如 `k2f.agent.state`），或連線紀錄裡有 `analytics.insightjs.com`
- 登錄檔有 `HKCU\Software\Classes\CLSID\{E77CC89B-7401-4C04-8CED-149DB35ADD04}`。**就算是空的也算**，因為木馬自己的清除指令只會清空、不會刪掉
- 瀏覽器資料夾的 `PlatformExperienceHelper` 底下有 64 碼英數字的子資料夾（**裡面沒有 DLL 也算**），或有 `platform_experience_helper.dll`
- `%TEMP%` 裡有 `k2f` 開頭的檔案

**只有下面這些，算「有接觸」：** 啟動器快取裡有 `insightjs`、DNS 紀錄查過 `insightjs[.]com`。

## 中毒了怎麼辦

帳號和電腦可以同時處理，不用等電腦清完才改密碼。

### 1. 隔離電腦

拔網路線或關 Wi-Fi。在右下角系統匣對啟動器按右鍵，選結束。之後不要再用這台電腦登入任何服務。如果是公司電腦，或電腦裡有公司帳號、憑證，先找公司 IT。

### 2. 用手機或另一台乾淨的電腦處理帳號

照這個順序：

1. **主要信箱**（Google、Outlook 等）：它能拿來重設其他帳號，最優先。
2. **NIKKE／Level Infinite 帳號**。
3. **這台電腦瀏覽器存過密碼的帳號**：Discord、Steam、網路銀行等。

每個帳號都要：

- 換一組新的、沒在別處用過的密碼
- **登出所有裝置**。偷走的登入 Cookie 不用密碼也能登入，只改密碼不夠
- 開啟兩步驟驗證
- 看一下最近的登入紀錄，以及信箱有沒有被加上自動轉寄

### 3. 換掉金鑰和錢包

- **有寫程式的人**：撤銷並重新產生 SSH 金鑰、Git Token 和各種 API 金鑰（OpenAI、Anthropic、AWS、GitHub 等）。
- **在這台電腦用過 AI 工具的人**（Claude、ChatGPT、Codex 等）：登出那個工具，並換掉它的 API 金鑰。
- **有加密貨幣的人**：在乾淨的裝置建立新錢包（新的助記詞），把資產轉過去。

### 4. 清理電腦

動手前，先把一鍵檢查的結果截圖存下來。

**最保險的做法是重灌 Windows。** 木馬能執行任何指令，沒人知道攻擊者當時還做了什麼。先備份個人檔案（照片、文件），不要備份舊的程式、整個瀏覽器設定或啟動器快取，再用可信的安裝媒體重灌。

暫時不重灌的話，至少要移除已知的常駐（全程保持離線）：

1. 刪除登錄機碼 `HKCU\Software\Classes\CLSID\{E77CC89B-7401-4C04-8CED-149DB35ADD04}`（整個機碼）。`HKCU\Software\Classes\WOW6432Node\CLSID\` 底下如果也有同一個，一起刪掉。**`HKLM` 底下同一個 CLSID 是 Windows 自己的（指向 `wpc.dll`），不要動。**
2. 重新開機。DLL 使用中會被鎖住，重開才刪得掉。
3. 刪除 `PlatformExperienceHelper` 底下那個 64 碼的子資料夾。**只刪那個子資料夾**，不要刪整個 `PlatformExperienceHelper`；裡面的 `platform_experience_helper.exe`、`LoadFeatures` 是 Chrome 正常的檔案。一鍵檢查如果有列出「舊版常駐檔案」，也一起刪掉。
4. 再跑一次一鍵檢查，確認都清掉了。

Defender（包括離線掃描）曾經漏掉這顆 DLL，**掃毒顯示乾淨不代表沒事**。手動移除也只處理已知的部分；攻擊者如果另外裝了別的東西，只有重灌才能確定清掉。

### 5. 重新使用啟動器之前

- 等官方公告確認修好，並說明影響範圍。
- 先刪掉啟動器快取 `%APPDATA%\nikke_launcher*\tbs_cache`，裡面可能還留著被改過的檔案。[刪除指令在這裡](WINDOWS-CHECKS.md#重新開啟動器前先刪快取)。
- 之後幾週多留意帳號的登入紀錄和金融交易。

## 沒中也建議做的預防

- 做完[先做這三件事](#先做這三件事)。
- **封鎖攻擊用的網域和 IP**：網域 `insightjs.com`、`analytics.insightjs.com`、`insightjs.org`，IP `89.127.233.127`、`89.126.250.172`。用 hosts 檔、AdGuard Home 或路由器都可以，[做法在這裡](WINDOWS-CHECKS.md#封鎖攻擊用的網域和-ip)。
- **重新開啟動器之前，先刪 `tbs_cache`。**
- 選用：用 Windows 防火牆擋住 `tbs_browser.exe` 的對外連線，[做法在這裡](WINDOWS-CHECKS.md#選用擋住-tbs_browserexe-的對外連線)。

## 常見問題

### 我的 `k2f_chan` 是 10 月的，比大爆發還早，也算嗎？

算。目前公開最早的執行紀錄是 10/24，實際可能更早。

`debug.log` 的紀錄長這樣：

```text
[1101/012037.043:INFO:CONSOLE(1)] "k2f_chan registered", source: (1)
```

`1101/012037` 就是 11 月 1 日 01:20:37（電腦的當地時間），紀錄裡沒有年份。如果看到比 10/24 還早的紀錄，請保留原檔，對照系統時間確認年份。

### 我沒裝 Chrome，是不是就沒事？

不是。木馬是在 NIKKE 啟動器自己內建的瀏覽器裡執行，跟你有沒有裝 Chrome 無關。有玩家只裝了 Edge 和 Firefox，`debug.log` 照樣有兩次執行紀錄。

大家說 Chrome 是重災區，是因為木馬有專門偷 Chrome 密碼和錢包的功能，常駐 DLL 也是寄生在 Chrome 上。但它一樣會偷 Edge、Firefox 的資料，也會自己建立放 DLL 的 Chrome 資料夾。沒用 Chrome，只是少了一部分損失。

### 只有 `k2f_chan`，沒有 DLL 和登錄檔，算比較輕嗎？

不算。偷資料在啟動器裡就能做完，DLL 只是後續用來常駐的手段。有沒有裝上 DLL，帳號都一樣要處理。

### 我家用 AdGuard Home，看得出是不是它擋下來的嗎？

打開 AdGuard Home 的「查詢紀錄」，選那台電腦，對準你開遊戲的時間，搜尋 `insightjs`：

| 查到什麼 | 代表什麼 |
|---|---|
| `insightjs.com` 被封鎖 | 那次啟動時，攻擊腳本的來源被擋住了。每次遊玩的時間都吻合的話，更有把握 |
| `insightjs.com` 放行，`analytics.insightjs.com` 被封鎖 | 攻擊腳本可能已經載入，但下載木馬那一步被擋了 |
| 回應了真實 IP（放行） | 那次 DNS 沒擋，要看電腦上的痕跡 |
| 完全沒紀錄 | 無法判斷。查詢可能走了別的 DNS（例如路由器發的 IPv6 DNS）、用了快取，或紀錄已經過期 |

有兩件事要知道：

- **DNS 沒紀錄，不能推翻電腦上的中毒痕跡。** 有一台已經中毒的電腦，Pi-hole 上完全沒有 `analytics.insightjs.com` 的紀錄，後來才發現它同時在用路由器發的 IPv6 DNS。
- **沒中不一定是 DNS 的功勞。** 有人 11/1 用 AdGuard Home，DNS 放行了攻擊網域，結果也沒中，可能只是伺服器那幾次沒送木馬。

另外，可以在 AdGuard Home 的「自訂過濾規則」加上 `||insightjs.com^` 和 `||insightjs.org^`，連子網域一起擋掉。

### 可以請 AI 幫我查嗎？

可以，用 [AGENT-PROMPT.md](AGENT-PROMPT.md) 的提示詞。但要注意：

- **已經確定中毒的電腦，不要為了用 AI 而重新連網。** 改用一鍵檢查，不用連網也能跑。
- 在中毒電腦上用過的 AI 工具，要登出並換掉金鑰。
- 不要把完整的紀錄或 AI 的報告貼到網路上，裡面有你的使用者名稱和電腦名稱。

### 其他 Level Infinite 遊戲也會中嗎？

目前沒有證據。網路上流傳「Level Infinite 旗下 PC 遊戲都要警戒」的清單，以及「王者榮耀、三角洲行動也確認中了」的說法，都沒有附來源。

- 研究者 MotherSphere、WormW 和 Inven Global 的報導，都只發現 NIKKE 受影響。Inven 的說法是：目前沒有證據顯示其他遊戲的啟動器被用同樣的方式入侵。
- 這次的木馬要靠 NIKKE 啟動器內建瀏覽器的本機功能才能下載、執行，研究者確認它只在 NIKKE 啟動器裡運作。
- 那份清單把騰訊投資的工作室和 Level Infinite 發行的遊戲混在一起。例如 Valorant 是 Riot 的遊戲，《鳴潮》是庫洛自己營運。

不過有兩點還不確定：

- 4 個被改的檔案裡，有 2 個放在共用路徑 `/comm/assets/lib/` 底下，研究者提醒同一個儲存空間的其他檔案也可能被改過。攻擊者怎麼拿到 CDN 權限、範圍是不是只有 NIKKE，官方都還沒說明。
- 木馬會偷 NIKKE 的登入資料。其他遊戲如果也用同一個 Level Infinite 帳號，那些遊戲的帳號一樣有風險，改 NIKKE 密碼時就一起處理了。

所以目前不用跟著停玩其他遊戲，但 Level Infinite 帳號的密碼要改，並留意官方後續公告。

### 手機版、Linux／Steam Deck 呢？

目前沒有手機版出事的回報，但官方也還沒確認安全。用 Linux 或 Steam Deck（Proton）跑 PC 版的一樣會中，不過這份表的指令只適用 Windows，請看 [MotherSphere 的檢查工具說明](https://github.com/MotherSphere/NIKKE-Launcher-Check)。

## 資料來源

- [MotherSphere 事件調查](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5)：痕跡、時間線、處置建議
- MotherSphere [NIKKE-Launcher-Check](https://github.com/MotherSphere/NIKKE-Launcher-Check)：[事件細節](https://github.com/MotherSphere/NIKKE-Launcher-Check/blob/main/docs/incident.md)、[處置方式](https://github.com/MotherSphere/NIKKE-Launcher-Check/blob/main/docs/what-to-do.md)、[檢查項目](https://github.com/MotherSphere/NIKKE-Launcher-Check/blob/main/docs/checks.md)
- [WormW 樣本分析](https://github.com/WormW/nikke_malware_analysis)：攻擊路徑、[11/4 CDN 複查](https://github.com/WormW/nikke_malware_analysis/blob/a2cb84787d334ef6027df45a216eb929c8c79149/docs/en/attack-path.md#211-official-asset-remediation-status-2026-11-04-re-check)
- [巴哈姆特整理文（11/4 更新）](https://forum.gamer.com.tw/C.php?bsn=36390&snA=23304)
- [國際版官方 X 公告](https://x.com/NIKKE_en/status/2117817602659770434)、[台港澳官方公告頁](https://nikke.hotcool.tw/news.html)、[國際版官方公告頁](https://nikke-en.com/news.html)
- [Inven Global 報導](https://www.invenglobal.com/articles/28122/is-my-pc-infected-nikke-launcher-tampering-concerns-from-attack-vector-to-remediation)
