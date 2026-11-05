# NIKKE PC 啟動器木馬檢查表（Windows）

資料更新：2026-11-05 10:53（台灣時間）。事件還在發展，請以官方後續公告為準。

## 先做這三件事

**9/27 之後在電腦上開過 NIKKE 啟動器的人，不管有沒有中，都建議先做：**

1. **先別重開 PC 版啟動器；先保全日誌與快取，完成安全檢查。** 官方已公告清除功能，但它在取得同意後會刪除相關日誌、DLL 與登錄資料，詳見[官方最新措施](#官方說法)。
2. **用手機改 NIKKE（Level Infinite）帳號密碼，並登出所有裝置。** 同一組密碼也用在別處的，一起換掉。
3. **跑一次[一鍵檢查](WINDOWS-CHECKS.md#一鍵檢查)**，照結果決定下一步。

**11/5 檢查工具更新：先前沒找到痕跡的人，請看[重新檢查建議](#舊的陰性結果建議重查)。** 上游已發布 v0.5.2，修正部分 Temp 誤報並補強 Google Play Games 版辨識；**gzip HTTP 快取仍可能漏看**，更新工具後沒找到痕跡也不能保證安全。

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

**11/5 官方已發布[道歉聲明](https://nikke-en.com/newsdetail.html?content_id=3a94ebf0ab6c5a4703aa87aa57d0b8cd3d4b)、[後續措施](https://nikke-en.com/newsdetail.html?content_id=b4f23938a63a0a4e68aa577af395b1edb172)與[手動安全檢查指南](https://nikke-en.com/newsdetail.html?content_id=ecdccd40a55a5a46aaa9abda64d436082db7)。** 11/4 [首次公告](https://x.com/NIKKE_en/status/2117817602659770434)之後的新進展如下：

- **道歉與目前啟動器狀態：** 官方就未能提供安全服務道歉並承擔營運責任，表示已完成必要修復及額外檢查，**宣稱目前發行的啟動器版本可安全使用**。這是官方聲明，本表未獨立驗證整個啟動器，也不代表先前受影響的電腦已清乾淨或外洩資料已收回。
- **遊戲內補償：** 官方表示已為全體玩家準備遊戲內補償；**截至本次查核，這份道歉聲明未列品項、數量或發送時間**。這不是外部帳號或金錢損失的全面賠償承諾。
- **檢查對象：** 官方要求 **9/27 14:41（台灣時間；15:41 UTC+9）後開過 Windows PC 啟動器**的人檢查。本表原本採用整個 9/27 起的保守範圍，未縮短。官方同時表示，**手機版本未受這次事件影響**，只玩手機版的人不需為此事件額外檢查。
- **CDN 修復：** 官方稱 11/4 01:04–01:23 已替換三個異常腳本，**11/4 11:09 完成最後一個異常檔案的 CDN 快取刷新，之後逐步部署到相關節點**。這不是本表獨立驗證所有節點同時乾淨的結果。
- **官方清除功能：** 公告稱 **11/5 07:00（台灣時間；08:00 UTC+9）起**，PC 客戶端提供本事件惡意程式的掃描與移除功能（手動指南稱為 ACE）。檢出後會顯示視窗，**徵得同意才刪除相關日誌、DLL 與登錄資料**；未檢出則可能直接啟動遊戲而沒有額外提示。本表只核對公告，**未實測功能部署、涵蓋範圍或清除效果**。
- **帳號保護與申報：** 官方要求更改在受影響電腦使用過的帳號密碼，撤銷並重發曾使用或儲存的 API 金鑰與 access token；疑似外部帳號或金錢損失可向客服申報。本表仍建議用手機或另一台乾淨裝置處理帳號，不必等電腦清完。
- **後續防護承諾：** 官方將強化監控、建立外部資源完整性驗證，並檢討、降低啟動器權限。公告沒有表示這些措施都已完成，入侵根因及受影響程度仍在調查。

**清除前先保全證據。** 官方功能可能刪掉相關日誌；若要留存、交給 IT 或釐清個案，請先依[清理前備份](#4-清理電腦)保存日誌、快取與報告，再決定是否同意清除。這是本表的證據保全提醒，不是官方已提供備份功能。已隔離的電腦維持離線，**不要只為了觸發掃描或確認有沒有提示而重開啟動器**。官方手動指南同樣要求先斷網、完全關閉遊戲、啟動器與 Chrome，檢查完成前不要重開。

**官方手動檢查也不是完整排除。** 指南以 `debug.log` 的 `k2f_chan registered` 為主要搜尋項目，並明說沒找到仍不能排除風險。它列出一般版與 `NIKKE_GOOGLE` 日誌位置，但快取清理只明列 `%APPDATA%\nikke_launcher\tbs_cache`；沒有補齊本表下述 CEF、LevelDB、gzip 等掃描限制，也未公開 ACE 的完整檢查清單、版本或雜湊。官方的已知惡意程式移除，不能當作已排除任意指令留下的其他變更，或已撤回外洩資料的證明。

**手動移除條件有範圍限制。** 官方只列出 `Launcher\log\tinyget` 內 **205,135 bytes、符合 `tinyget.<日期>-<時間>.log` 格式**的下載檔；[WormW 構建紀錄（固定版本）](https://github.com/WormW/nikke_malware_analysis/blob/07f1bf4b2b9f16a83dd101a33c0eabf81cd2fb8a/docs/attack-path.md#212-已知-payload-构建演进)另記錄 **165,108 bytes 的早期構建**，屬第三方報告、樣本未獨立核驗。單一大小條件不能涵蓋所有已報告構建。官方 DLL 步驟也只刪指定 Chrome 路徑中，同時符合「64 碼十六進位子目錄」及「內含 `platform_experience_helper.dll`」的項目。**沒有符合這些條件，不等於沒有執行過木馬或沒有其他常駐。** 請先保全證據，不要只憑檔案大小擴大刪除範圍；本表感染判定未改動。

**官方投遞時間與研究紀錄仍有差異。** 新公告把開始投遞惡意程式的時間列為 **10/30 02:52（台灣時間；03:52 UTC+9）**；本表引用的研究紀錄早至 **10/24 00:40（台灣時間；10/23 16:40 UTC）**。兩者尚未獲解釋，本表保留並列，**不據此縮短風險期間或推翻較早的執行紀錄**。

### 時間線（台灣時間）

| 日期 | 發生什麼 |
|---|---|
| 9/27 | `insightjs[.]com` 被註冊；同一天 CDN 上 4 個 JS 檔被改 |
| 10/24 00:40 | 目前公開最早的木馬執行紀錄。另有玩家回報 9/30、10/15 的 DNS 紀錄就出現這個網域，實際開始時間可能更早 |
| 10/28 | 先前證據包稱 `FeedList.feb647c9.js` 在源站被清掉、部分 CDN 節點仍送舊檔；最新研究未裁定源站與中間快取的差異，不能把此解釋當成已確認事實 |
| **10/30 02:52** | 11/5 官方公告所列的惡意程式投遞起點；與上列較早的研究執行紀錄不一致，尚待釐清 |
| 11/4 01:04–01:21 | 另外 3 個被改的檔案換回乾淨版本 |
| 11/4 02:31 | 攻擊者的木馬伺服器停止回應 |
| 11/4 06:26、10:01–10:07 | 研究者仍從部分 CDN 節點拿到被改過的 `FeedList.feb647c9.js` |
| **11/4 11:09** | 11/5 官方追述：最後一個異常檔案完成 CDN 快取刷新，之後逐步部署至相關節點；非全節點同時驗證 |
| 11/4 11:13 | 國際版官方在 X 發布首次公告 |
| **11/4 13:52** | 本表補充抽查：本次網路路徑取得 `FeedList.feb647c9.js`、`aegis-fix.min.js`、`vue-bucket.min.js`、`images.7fec4c32.js` 的內容，SHA-256 均符合研究者公布的乾淨版本。**不代表全球 CDN 節點都已清除** |
| **11/4 15:14–15:20** | WormW 重新 GET 四個已知官方 JS，均取得乾淨內容；`FeedList` 原 URL 與加上查詢參數的請求均為 5,099 bytes。這是研究者抽查結果，**不是官方全節點清除確認** |
| **11/4 19:20** | MotherSphere 複查四個 CDN 節點、兩個主機名的四個已知 JS，回報均為乾淨內容；這仍是有限範圍的研究者抽查 |
| **11/4 19:57** | MotherSphere 發布 v0.4.1，修正壓縮 LevelDB／共用前綴造成的漏檢；舊的陰性結果建議重新檢查，詳見下方說明 |
| **11/4 21:02** | MotherSphere 原始碼新增 CEF 登入資料夾檢查，並修正僅因啟動器執行中而判感染；當時尚未發布，後續已納入 v0.5.1 |
| **11/4 21:39** | MotherSphere 正式發布 v0.5.1，包含上述修正，並新增 Google Play Games 版的 `nikke_launcher_google` 資料夾檢查 |
| **11/5 01:13–01:34** | 玩家新增 gzip 快取漏檢及 Temp 名稱疑似誤報回報；個案尚待核實，詳見下方檢查限制 |
| **11/5 03:39–03:41** | 一名 Google Play Games PC 版玩家回報日誌搜尋命中 `k2f_chan`、`insight.bundle`；當時只附搜尋摘要，後於 10:43 公開命中節錄，完整原檔仍未獨立核驗 |
| **11/5 06:40** | MotherSphere 正式發布 v0.5.2：縮小 Temp 檔名比對、排除目錄，補強 Google Play Games 版辨識，並區分木馬載入後啟動失敗的報告；後者仍列為 critical |
| **11/5 06:41** | 作者確認 Issue #19 的 Temp 目錄命中是工具誤報；另依 Issue #11 當時的搜尋摘要判定該 Google Play Games 版個案有木馬執行 |
| **11/5 07:00 起（官方公告時間）** | 官方公告 PC 客戶端提供本事件掃描與經同意移除功能；本表未實測部署或效果，清除前應先保全證據 |
| **11/5 10:43** | Issue #11 玩家公開 7 組 `lua downloaded`／`k2f_chan registered` 節錄；完整原始日誌仍未獨立核驗，僅搜尋這兩類字串不能排除其他遠控紀錄或資料外洩 |

**11/4 13:52 本表抽查已有改善，但只是有限觀察。** 當時的檢查只讀取這四個檔案並計算雜湊，沒有執行 JavaScript。`FeedList` 這次收到 5,099 bytes，SHA-256 為 `49b625b30b34dc1fa16edba30bdf63c18b6171a0fc4cf030e21ea4e1263ae18a`，與[已知乾淨版對照](https://github.com/WormW/nikke_malware_analysis/blob/a2cb84787d334ef6027df45a216eb929c8c79149/docs/en/attack-path.md#211-official-asset-remediation-status-2026-11-04-re-check)相符。回應的 `Date` 是 03:12 UTC，並非本次取得時間，因此不能當作即時源站或全節點驗證。

上午取得污染副本的紀錄仍保留為歷史觀察；上述單一路徑抽查也不能證明所有地方都已安全，更不會清掉玩家電腦上的舊快取或既有感染。11/5 官方已公布上述處理方式，但完整影響範圍與入侵根因仍待釐清。

**15:14–15:20 的複查仍有範圍限制。** [WormW 最新複查（固定版本）](https://github.com/WormW/nikke_malware_analysis/blob/07f1bf4b2b9f16a83dd101a33c0eabf81cd2fb8a/docs/attack-path.md#211-官方资产当前状态2026-11-04-复核)確認上述四個物件在本次請求中已沒有已知注入特徵，但外部回應不能完全區分源站與中間快取，也不能排除其他 CDN 邊緣節點或玩家本機仍有舊檔。攻擊者如何取得寫入權限、是否還有其他受污染檔案，都尚未釐清。**這不是全面安全或感染電腦已清理的證明。**

**11/4 13:53 研究補充（發布時間）：單機疑似早期中斷。** WormW 根據一台電腦的兩次日誌截圖，記錄到 `k2f_chan registered` 後出現 `invalid UTF-8` 錯誤；截圖範圍內未見 `adopted` 或 `agent started as`，因此完整遠控會話是否建立仍未證實。原始完整日誌未提供，截圖未公開入庫，研究者也未直接檢查該主機。「中文使用者路徑導致中斷」只是未證實的假說。**這個案例仍已開始執行木馬，不能當作安全或沒有外洩的證明，也不要為了驗證而重開啟動器。** [WormW 單機觀察（固定版本）](https://github.com/WormW/nikke_malware_analysis/blob/1b7a2ca164654fd1a10464e95df56c082cb026dd/docs/field-observations.md)

**19:20 的複查範圍擴及四個 CDN 節點。** [MotherSphere 晚間更新（固定版本）](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5/4d9b02aa437755f376edb96403042c94898bda0a)回報，四個已知 JS 在四個 CDN IP、兩個主機名上均為乾淨內容，也檢查了 gzip 與帶查詢參數的回應。研究者因此建議：重新開啟前，先檢查或保全日誌、快取，再清除 `tbs_cache`，並持續封鎖已知攻擊網域與 IP。**這不是官方全節點安全確認**；未抽查節點、入侵根因仍有疑問，本機舊快取及既有感染也不會自動清除。封鎖目前的網域與 IP，亦不能防止攻擊者改用新位置。

### 舊的陰性結果建議重查

**11/4 19:57（台灣時間），MotherSphere 發布 [NIKKE-Launcher-Check v0.4.1](https://github.com/MotherSphere/NIKKE-Launcher-Check/releases/tag/v0.4.1)。** [這次修正](https://github.com/MotherSphere/NIKKE-Launcher-Check/commit/8a823d59e4d32358173197e64ea0b209650feb32)指出，啟動器 Local Storage 的 LevelDB 會把紀錄移到 `.ldb` 表，資料可能經 Snappy 壓縮，鍵名也可能共用前綴；直接搜尋原始文字可能漏掉 `k2f.agent.state`。新版會先解壓並還原完整鍵名再搜尋。

**本表的一鍵 PowerShell 檢查尚未加入這項解析。** 若先前只用本表指令、純文字搜尋，或 MotherSphere 舊版工具，得到「沒找到已知痕跡」或只有接觸、未見木馬的結果，建議用已正式發布的 [v0.5.2](https://github.com/MotherSphere/NIKKE-Launcher-Check/releases/tag/v0.5.2) 重新檢查，尤其是 `debug.log` 已遺失或被覆寫時：

**11/5 05:16（台灣時間）新增防毒警示回報。** 一名玩家在[上游 Issue #22](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/22)回報，執行 v0.5.1 掃描時，Windows Defender 立即隔離檢查工具，偵測名稱為 `Trojan:Win32/Bearfoos.A!ml`。回報未附被隔離檔案的 SHA-256、Defender 引擎／安全性情報版本或完整偵測紀錄。作者於 **11/5 06:17** 回覆，並在 **11/5 08:14** [更新同一則留言](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/22#issuecomment-6643556806)，主張這是誤報，**表示已將 v0.5.1 提交 Microsoft 申訴**，並把還原檔案的建議放在 Microsoft 確認排除偵測、更新 Defender 之後。這是作者的說明與申訴狀態，**議題仍開啟，未見 Microsoft 裁定；不能寫成已證實誤報、已解決，也不能據此斷言工具惡意。** v0.5.2 的發布亦不等於這個防毒問題已排除。

**若防毒攔截或隔離檢查工具，先停止使用並保持隔離，不要為了執行它而關閉防毒、加入排除或還原放行。** 保留偵測名稱、時間與報告，等待工具作者或防毒廠商核實；版本與雜湊核對不能取代安全審核。

1. 先保留現有日誌、快取與舊報告，**不要為了重查而開啟 NIKKE 啟動器，也不要先刪 `tbs_cache`**。
2. 從 [v0.5.2 發布頁](https://github.com/MotherSphere/NIKKE-Launcher-Check/releases/tag/v0.5.2)取得 Windows 版 `nikke-check-windows-x86_64.exe` 和**同版本**的 `SHA256SUMS.txt`，依[下載核對說明](https://github.com/MotherSphere/NIKKE-Launcher-Check/blob/v0.5.2/docs/usage.md#check-the-download)比對 SHA-256。這是未簽章的第三方檢查工具，不是遊戲官方程式；雜湊相符只確認下載一致，不等於完整安全審核。
3. 已隔離的電腦保持離線，由乾淨裝置下載後帶入。檢查工具可離線掃描，視窗上方應顯示 0.5.2。新版支援繁體中文；下列按鈕以英文名稱對照。先選「Scan this computer」；未找到木馬跡證時依提示做「Full scan」。特殊安裝位置或未涵蓋的資料夾，再用「Scan a folder...」，並核對「Details」中的掃描位置與讀取失敗項目。[使用說明](https://github.com/MotherSphere/NIKKE-Launcher-Check/blob/v0.5.2/docs/usage.md#the-window)

**新版沒有找到痕跡，也不能保證沒感染或沒外洩。** 已刪除的紀錄不會因更新工具而恢復，掃描仍受位置、檔案可讀性等限制。先保存新報告，再依既有處置步驟處理。

### 上游已發布的檢查範圍更新

**11/4 21:39（台灣時間），MotherSphere 正式發布 [v0.5.1](https://github.com/MotherSphere/NIKKE-Launcher-Check/releases/tag/v0.5.1)。** 先前 21:02 的[原始碼更新](https://github.com/MotherSphere/NIKKE-Launcher-Check/commit/570d8a0cd42f00d2c11a507f2cb494c2d137cb4d)已包含在這個正式版，**不必再等待發布**。[版本紀錄（固定版本）](https://github.com/MotherSphere/NIKKE-Launcher-Check/blob/0e09c7c422c48103856a933a3b910b51ba7b83da/CHANGELOG.md)列出：

- **擴大檢查範圍：** 新增啟動器登入視窗使用的 `%LOCALAPPDATA%\CEF\User Data`，搜尋相同的木馬跡證，也解析壓縮的 LevelDB。其他 CEF 程式可能共用這個位置，**資料夾存在本身不代表感染，不要整個直接刪除**。v0.4.1 未包含此範圍，本表腳本也仍未加入這項專門檢查；先保留可能相關的資料，再用目前的 v0.5.2 補查。
- **修正上游工具誤報：** 不再只因 NIKKE 啟動器正在執行就列為感染，而是同次掃描另有木馬跡證時才把執行中的啟動器列為 critical。若舊工具的感染結果只有這個理由，先核對報告的「Details」；**這項修正不會推翻已找到的實際木馬跡證**，也不是重新開啟啟動器就安全的保證。
- **補上 Google Play Games 版資料夾：** v0.5.1 新增 `%APPDATA%\nikke_launcher_google` 的檢查。使用這個版本的玩家，重查時請核對報告是否涵蓋該資料夾；這是掃描範圍補充，不是新增感染規模的證據。

以上是上游正式工具的更新，本表既有腳本與感染判定未隨之修改。v0.5.1 也包含先前的 LevelDB 漏檢修正；仍須先保存證據，不能把一次沒有找到痕跡當作全面安全的證明。

### v0.5.2：Temp 誤報與報告細節修正

**11/5 06:40:34（台灣時間；11/4 22:40:34 UTC），MotherSphere 正式發布 [v0.5.2](https://github.com/MotherSphere/NIKKE-Launcher-Check/releases/tag/v0.5.2)。** 這是研究者維護的第三方工具更新，不是 NIKKE 官方安全認證。主要修正：

- **Temp 不再把所有 `k2f` 開頭項目都算進去。** 現在只收錄 `k2f-` 或 `k2f_` 開頭的一般檔案，排除目錄及 `k2fkuxjm.qu0` 這類隨機名稱。作者已確認 Issue #19 的舊目錄命中是工具誤報，詳見下方限制。[修正與測試](https://github.com/MotherSphere/NIKKE-Launcher-Check/commit/435ea38ada82ea6325b4a789fb32423bb91fbeb7)
- **補強 Google Play Games 版辨識。** 除 v0.5.1 已加入的資料夾外，現在也能透過 `nikke_launcher_google.exe` 找到安裝位置、辨識執行中的程序，並在關閉建議列出正確名稱。[修正](https://github.com/MotherSphere/NIKKE-Launcher-Check/commit/2aba14446b6ba089447fff6c6bfb8ba3fdec0e48)
- **區分「木馬已載入，但啟動失敗」。** 若日誌裡每次 `k2f_chan` 後都在 `adopted`、`resuming` 或 `agent started as` 之前記錄 `startup failed`，新版會另用標題說明，**仍列 critical，仍適用感染後處置**。日誌無法保證攻擊者從未建立會話或沒有資料外洩；不能因錯誤訊息就改判安全。另已排除其他頁面、木馬啟動範圍外的同名字串，避免錯引證據與時間。[啟動失敗說明](https://github.com/MotherSphere/NIKKE-Launcher-Check/commit/107ae396a7b851acf0ad5ae0e7c2d085c593293a)、[引用範圍修正](https://github.com/MotherSphere/NIKKE-Launcher-Check/commit/4fa775de9627d09da9d90636ba1c76c8c5de8dc7)

本次只更新上游狀態與使用說明，**本表腳本與感染判定未修改**。v0.5.2 仍未補上 gzip HTTP 快取解壓檢查，不能把版本更新當成所有漏檢都已修復。

### Google Play Games 版：已公開命中節錄，完整原檔未核驗

**11/5 03:39（台灣時間）發表、03:41 更新的[上游 Issue #11 留言](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/11#issuecomment-6640267816)**，一名使用 Google Play Games 安裝 NIKKE PC 版的玩家貼出 PowerShell 搜尋摘要：`C:\NIKKE_GOOGLE\Launcher\intl_service\debug.log` 命中 `k2f_chan`，兩個 tinyget 日誌命中 `insight.bundle`。玩家另回報執行檔名為 `nikke_launcher_google.exe`，與一般版不同。

**作者於 11/5 06:41 [回覆](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/11#issuecomment-6644005418)，依當時的搜尋摘要判定木馬曾在該電腦執行，Google Play Games 版也載入了污染檔案，並建議按感染處理。** 玩家於 **11/5 10:43:31（台灣時間）[補充公開命中節錄](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/11#issuecomment-6648246141)**，共有 7 組 `lua downloaded` 與 `k2f_chan registered`，日期標記為 11/1、11/2（該電腦當地時間，未換算為台灣時間）。這比先前只有檔案路徑與關鍵字的摘要更具體，但仍是**玩家提供的公開節錄，完整原始日誌及下載任務結果尚未獨立核驗**，不能據此認定哪些資料外洩，或推論所有 Google Play Games 版玩家都受影響。

**節錄沒有其他字串，不代表原始日誌沒有其他遠控紀錄。** 玩家使用的指令只篩選 `lua downloaded`、`k2f_chan registered`，並未搜尋 `adopted` 或 `agent started as`；不能從這份輸出推定遠控會話未建立、木馬早期中斷或沒有外洩。玩家另回報指定 DLL 與登錄鍵檢查未命中，並嘗試對照 NetworkUsageView／SRUM 時間；這些仍是個案回報，不能排除其他常駐，也不能把時間相關或流量紀錄直接認定為外洩內容。

該玩家先前沒有找到 `tbs_cache`，但自述可能曾手動刪除；10:43 的留言又回報，重開後在 `%APPDATA%\nikke_launcher_google` 產生新快取。這是其個案的路徑觀察，缺少舊快取仍不能當作未受影響的證明。**不要為了重現或確認資料夾而重開啟動器。** 使用此版本的玩家，請先保留現有日誌與資料，核對檢查報告是否涵蓋實際安裝位置（該回報為 `C:\NIKKE_GOOGLE\Launcher`）與 `%APPDATA%\nikke_launcher_google`，不要只因資料夾沒有命中就排除。**本次只補充回報與限制，不改動既有腳本或感染判定。**

### 11/5 檢查限制與誤報修正

- **舊版 Temp 目錄誤報已獲作者確認。** [Issue #19 作者回覆（11/5 06:41）](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/19#issuecomment-6644006713)確認，兩位玩家的舊 `k2f` 開頭目錄是因名稱被誤列 critical；v0.5.2 已改成只比對 `k2f-`、`k2f_` 開頭的一般檔案，目錄不再算。**本表腳本尚未同步修正**，仍可能命中這類項目。若舊報告只有這一項，先保留資料、核對完整報告，再用新版重查；不要只憑模糊名稱認定感染，也不能因這項誤報被排除就證明整台電腦安全。實際惡意檔案、`k2f_chan` 等其他跡證不受此修正影響。
- **gzip HTTP 快取漏看已獲作者實機觀察支持，v0.5.2 仍未修正。** 先前[玩家回報](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5#gistcomment-6457040)解壓後才找到污染檔案；[MotherSphere 隨後回覆](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5?permalink_comment_id=6457471)，自己的快取也有 gzip 內容，解壓後找到污染的 `images` 與 `aegis-fix`，並承認工具有相同盲點。靜態核對 [v0.5.2 快取掃描（固定版本）](https://github.com/MotherSphere/NIKKE-Launcher-Check/blob/07cf01ca36ee4504983bb610d2dd5f5dec9daf76/src/scan.rs#L661-L705)，只有 `.ldb` 會先解析，其餘仍搜尋原始位元組。**這支持漏檢機制，並非已獨立核驗每位玩家的原始快取**；「純文字沒命中」或「已支援 LevelDB 解壓」都不能證明 HTTP 快取乾淨。

先保存快取與報告，不要為了驗證而重新開啟啟動器。**以上限制不會推翻實際日誌中的 `k2f_chan registered`、已確認的惡意 DLL 等具體跡證。** 本次只補充上游修正與證據限制，既有腳本與判定未修改。

### 早期投遞腳本的證據界限

**11/5 補充研究限制，並非新增感染案例。** [MotherSphere 原留言](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5?permalink_comment_id=6456552)指出，該討論中尚無人公開早期負責選擇下載位置、投遞木馬的 `l.js`。[WormW 固定版本分析](https://github.com/WormW/nikke_malware_analysis/blob/07f1bf4b2b9f16a83dd101a33c0eabf81cd2fb8a/docs/attack-path.md#0-证据来源与结论边界)取得的 9,145 bytes 副本，是 **11/4 01:30（台灣時間）後被換成的 `/speed` 分析版本，不是早期惡意投遞版**。不能用這份事後副本證明先前腳本沒有其他功能。

MotherSphere 表示，該討論已貼出的 `insight.bundle` 下載網址都指向 `analytics.insightjs.com`，尚無其他主機或 IP 投遞的回報；但缺少當時的 `l.js`，**仍不能完整排除其他投遞路徑**。已知紀錄支持透過 `Srv_DownloadFile` 下載後，木馬隨即在啟動器內執行；具體由哪個原生橋呼叫觸發執行，仍未確認。

這是公開證據的缺口，**不是已證實初始腳本能獨立竊密，也不能只憑接觸紀錄改判感染**。請保留現有日誌與快取；本次不改動既有腳本、感染判定或處置步驟。

## 結果怎麼判讀

[一鍵檢查](WINDOWS-CHECKS.md#一鍵檢查)最後一行會顯示結果：

| 結果 | 代表什麼 | 怎麼做 |
|---|---|---|
| **中毒** | 木馬在這台電腦跑過 | 照下面的[中毒了怎麼辦](#中毒了怎麼辦)處理 |
| **高風險** | 發現啟動器向木馬伺服器請求或下載的跡證，成功與否待核對，也沒找到執行紀錄 | 先隔離電腦，**帳號照中毒處理**（第 1–3 步）。再[核對那筆紀錄](WINDOWS-CHECKS.md#結果怎麼看)是成功還是失敗，並確認 `debug.log` 涵蓋你玩的日期。找到執行痕跡就照中毒處理；都沒有的話不一定要重灌，但重灌仍是最保險的做法 |
| **有接觸** | 啟動器載入過攻擊腳本，但沒找到木馬執行的痕跡 | **帳號照中毒處理**（第 2、3 步）。電腦本身沒有中毒證據，不用急著重灌 |
| **資料不足** | 找不到 `debug.log`、重裝過，或紀錄沒涵蓋你玩的日期 | 無法判斷，帳號照「有接觸」處理 |
| **沒找到已知痕跡** | 已知的痕跡都沒出現 | 做完[先做這三件事](#先做這三件事)，並留意[官方最新措施與限制](#官方說法) |

**以下任何一項出現，就算中毒（證明木馬跑過）：**

- `debug.log` 裡有 `k2f_chan` 或 `agent started as`
- 啟動器資料 `%APPDATA%\nikke_launcher*` 裡出現 `k2f_chan` 或 `k2f.agent`（例如 `k2f.agent.state`）
- 登錄檔有 `HKCU\Software\Classes\CLSID\{E77CC89B-7401-4C04-8CED-149DB35ADD04}`。**就算是空的也算**，因為木馬自己的清除指令只會清空、不會刪掉
- 瀏覽器資料夾的 `PlatformExperienceHelper` 底下有 64 碼英數字的子資料夾（**裡面沒有 DLL 也算**），或有 `platform_experience_helper.dll`
- `%TEMP%` 裡有 `k2f` 開頭的檔案

**以下這些算「高風險」（發現向木馬伺服器請求或下載的跡證，成功與否待核對，也不能單獨證明木馬跑過）：**

- `debug.log` 或安裝資料夾的 `Launcher\log\` 裡出現 `insight.bundle`、`analytics.insightjs.com` 或 `lua downloaded to`
- `Launcher\log\` 裡有 `k2f.lua`、LuaJIT 格式的檔案，或下載回應的 ETag 是 `-3214f` 結尾
- 啟動器的連線紀錄（`TransportSecurity`、`Network Persistent State`）裡有 `analytics.insightjs.com`

下載紀錄如果顯示 `http_code=0`、0 bytes，代表那次下載沒成功。但木馬不是每次啟動都會送來，其他次可能成功了，所以帳號一樣照中毒處理。研究者 MotherSphere 也把「啟動器連到木馬伺服器」算成中毒。

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

動手前，先把一鍵檢查的結果截圖存下來。有疑點、檢查不完整，或要交給 IT 的話，先把 `Launcher\intl_service\debug.log`、`Launcher\log\`、`%APPDATA%\nikke_launcher*\tbs_cache` 複製到隨身碟或離線資料夾（不要放雲端同步，裡面有主機名稱和登入資料），再清理。

**最保險的做法是重灌 Windows。** 木馬能執行任何指令，沒人知道攻擊者當時還做了什麼。先備份個人檔案（照片、文件），不要備份舊的程式、整個瀏覽器設定或啟動器快取，再用可信的安裝媒體重灌。

暫時不重灌的話，至少要移除已知的常駐（全程保持離線）：

1. 刪除登錄機碼 `HKCU\Software\Classes\CLSID\{E77CC89B-7401-4C04-8CED-149DB35ADD04}`（整個機碼）。`HKCU\Software\Classes\WOW6432Node\CLSID\` 底下如果也有同一個，一起刪掉。**`HKLM` 底下同一個 CLSID 是 Windows 自己的（指向 `wpc.dll`），不要動。**
2. 重新開機。DLL 使用中會被鎖住，重開才刪得掉。
3. 刪除 `PlatformExperienceHelper` 底下那個 64 碼的子資料夾。**只刪那個子資料夾**，不要刪整個 `PlatformExperienceHelper`；裡面的 `platform_experience_helper.exe`、`LoadFeatures` 是 Chrome 正常的檔案。一鍵檢查如果有列出「舊版常駐檔案」，也一起刪掉。
4. 再跑一次一鍵檢查，確認都清掉了。

Defender（包括離線掃描）曾經漏掉這顆 DLL，**掃毒顯示乾淨不代表沒事**。手動移除也只處理已知的部分；攻擊者如果另外裝了別的東西，只有重灌才能確定清掉。

### 5. 重新使用啟動器之前

- 先完成安全檢查與必要處置。官方已公告修復與清除功能，但本表未實測其部署或效果；不要把沒有跳出警示當作電腦乾淨的證明。需要留證時，在同意任何清除前先備份。
- 先刪掉啟動器快取 `%APPDATA%\nikke_launcher*\tbs_cache`，裡面可能還留著被改過的檔案。需要留存資料的話，先照第 4 步複製。[刪除指令在這裡](WINDOWS-CHECKS.md#重新開啟動器前先刪快取)。
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

**11/5 [官方手動指南](https://nikke-en.com/newsdetail.html?content_id=ecdccd40a55a5a46aaa9abda64d436082db7)明確表示手機版本未受這次事件影響，只玩手機版的人不需為此事件額外檢查。** 這不涵蓋同時在 PC 版使用過的帳號。用 Linux 或 Steam Deck（Proton）跑 PC 版仍有研究者記錄的風險；官方這份 Windows 指南不是 Proton 的安全確認。本表指令只適用 Windows，其他系統請看 [MotherSphere 的檢查工具說明](https://github.com/MotherSphere/NIKKE-Launcher-Check)。

## 資料來源

- [MotherSphere 事件調查](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5)：痕跡、時間線、處置建議
- MotherSphere [NIKKE-Launcher-Check](https://github.com/MotherSphere/NIKKE-Launcher-Check)：[事件細節](https://github.com/MotherSphere/NIKKE-Launcher-Check/blob/main/docs/incident.md)、[處置方式](https://github.com/MotherSphere/NIKKE-Launcher-Check/blob/main/docs/what-to-do.md)、[檢查項目](https://github.com/MotherSphere/NIKKE-Launcher-Check/blob/main/docs/checks.md)
- [v0.5.2 正式發布（11/5 06:40，台灣時間）](https://github.com/MotherSphere/NIKKE-Launcher-Check/releases/tag/v0.5.2)：Temp 誤報修正、Google Play Games 版辨識與啟動失敗報告
- [Google Play Games PC 版玩家摘要](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/11#issuecomment-6640267816)、[作者回覆（11/5 06:41，台灣時間）](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/11#issuecomment-6644005418)與[玩家公開命中節錄（11/5 10:43:31，台灣時間）](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/11#issuecomment-6648246141)：查核至 11/5 10:53；已公開 7 組兩類字串節錄，完整原始日誌尚未獨立核驗
- [v0.5.1 被 Defender 隔離的玩家回報](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/22)與[作者更新（11/5 08:14，台灣時間）](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/22#issuecomment-6643556806)：作者主張誤報並表示已提交 Microsoft 申訴，議題仍開啟，未見廠商裁定
- [Temp 誤報確認（11/5 06:41，台灣時間）](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/19#issuecomment-6644006713)、[gzip 快取漏看作者回覆](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5?permalink_comment_id=6457471)
- [WormW 樣本分析](https://github.com/WormW/nikke_malware_analysis)：攻擊路徑、[11/4 CDN 複查](https://github.com/WormW/nikke_malware_analysis/blob/a2cb84787d334ef6027df45a216eb929c8c79149/docs/en/attack-path.md#211-official-asset-remediation-status-2026-11-04-re-check)
- [巴哈姆特整理文（11/4 更新）](https://forum.gamer.com.tw/C.php?bsn=36390&snA=23304)
- [11/5 官方道歉聲明](https://nikke-en.com/newsdetail.html?content_id=3a94ebf0ab6c5a4703aa87aa57d0b8cd3d4b)：11/5 08:22（台灣時間）讀取全文；目前啟動器安全為官方聲明，遊戲內補償細節未在此頁公布
- [11/5 官方後續措施](https://nikke-en.com/newsdetail.html?content_id=b4f23938a63a0a4e68aa577af395b1edb172)、[11/5 官方手動安全檢查指南](https://nikke-en.com/newsdetail.html?content_id=ecdccd40a55a5a46aaa9abda64d436082db7)：後續措施查核至 11/5 07:24、手動指南複核至 08:22（台灣時間），均已讀取全文；未實測 ACE 部署或清除效果
- [11/4 國際版官方首次 X 公告](https://x.com/NIKKE_en/status/2117817602659770434)、[台港澳官方公告頁](https://nikke.hotcool.tw/news.html)、[國際版官方公告頁](https://nikke-en.com/news.html)
- [Inven Global 報導](https://www.invenglobal.com/articles/28122/is-my-pc-infected-nikke-launcher-tampering-concerns-from-attack-vector-to-remediation)
