# NIKKE PC 啟動器木馬檢查表（Windows）

資料更新：2026-11-07 09:54（台灣時間）。事件還在發展，請以官方後續公告為準。

## 先做這三件事

**9/27 之後在電腦上開過 NIKKE 啟動器的人，不管有沒有中，都建議先做：**

1. **先別更新或重開 PC 版啟動器；先保全日誌、快取與報告，完成安全檢查。** 官方公告的清除功能會在取得同意後刪除相關日誌、DLL 與登錄資料；另有玩家尚未獨立驗證的更新後日誌消失回報，詳見[官方最新措施與保全提醒](#官方說法)。
2. **用手機改 NIKKE（Level Infinite）帳號密碼，並登出所有裝置。** 同一組密碼也用在別處的，一起換掉。
3. **跑一次[一鍵檢查](WINDOWS-CHECKS.md#一鍵檢查)**，照結果決定下一步。

**11/5 假冒「Cleanup Tool」警告：不要執行留言要求的管理員遠端腳本。** [MotherSphere 警告](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5?permalink_comment_id=6459938)指出，一則 **11/5 21:56（台灣時間）**留言假借清除本事件的名義，誘導使用者以管理員權限下載並執行外站腳本，讓遠端腳本能以該權限修改電腦。請勿複製執行、下載或開啟該留言提供的工具。

**這是研究者指出的另一項可疑清除工具風險，與原 NIKKE 攻擊指標分開。** 本表於 **11/5 22:42（台灣時間）**核對警告，未取得或分析該腳本，不能確認其具體行為或受害範圍，也未把留言聲稱的檔名、IP、登錄項目或排程工作加入感染判定。若已執行，研究者建議把電腦視為遭該腳本入侵：先斷網、用另一台乾淨裝置保護帳號，並重裝 Windows。本表原有 NIKKE 檢查不能排除這個遠端腳本造成的風險。

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

**11/6 研究補充：NAS／分享資料夾與同步檔案也可能在讀取範圍內。** [MotherSphere 11/6 回覆](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5?permalink_comment_id=6461096)依所分析的 **11/3 構建**指出，任意讀檔能力不限於固定蒐集清單；執行帳號有權限存取的 UNC／NAS 分享，以及本機 OneDrive 等同步資料夾，也可能被讀取。這不是已證明能直接讀取所有雲端資料。**這是研究者對樣本能力的分析，本表未獨立核驗反編譯內容；不能套用到全部構建，也不代表任何特定分享或檔案已被讀取、外傳，更不是 NAS 本身已感染的證據。**

### 官方說法

**11/5 官方已發布[道歉聲明](https://nikke-en.com/newsdetail.html?content_id=3a94ebf0ab6c5a4703aa87aa57d0b8cd3d4b)、[後續措施](https://nikke-en.com/newsdetail.html?content_id=b4f23938a63a0a4e68aa577af395b1edb172)與[手動安全檢查指南](https://nikke-en.com/newsdetail.html?content_id=ecdccd40a55a5a46aaa9abda64d436082db7)。** 11/4 [首次公告](https://x.com/NIKKE_en/status/2117817602659770434)之後的新進展如下：

- **道歉與目前啟動器狀態：** 官方就未能提供安全服務道歉並承擔營運責任，表示已完成必要修復及額外檢查，**宣稱目前發行的啟動器版本可安全使用**。這是官方聲明，本表未獨立驗證整個啟動器，也不代表先前受影響的電腦已清乾淨或外洩資料已收回。
- **遊戲內補償：** 官方表示已為全體玩家準備遊戲內補償；**截至本次查核，這份道歉聲明未列品項、數量或發送時間**。這不是外部帳號或金錢損失的全面賠償承諾。
- **檢查對象：** 官方要求 **9/27 14:41（台灣時間；15:41 UTC+9）後開過 Windows PC 啟動器**的人檢查。本表原本採用整個 9/27 起的保守範圍，未縮短。官方同時表示，**手機版本未受這次事件影響**，只玩手機版的人不需為此事件額外檢查。
- **CDN 修復：** 官方稱 11/4 01:04–01:23 已替換三個異常腳本，**11/4 11:09 完成最後一個異常檔案的 CDN 快取刷新，之後逐步部署到相關節點**。這不是本表獨立驗證所有節點同時乾淨的結果。
- **官方清除功能：** 公告稱 **11/5 07:00（台灣時間；08:00 UTC+9）起**，PC 客戶端提供本事件惡意程式的掃描與移除功能（手動指南稱為 ACE）。檢出後會顯示視窗，**徵得同意才刪除相關日誌、DLL 與登錄資料**；未檢出則可能直接啟動遊戲而沒有額外提示。本表只核對公告，**未實測功能部署、涵蓋範圍或清除效果**。
- **帳號保護與申報：** 官方要求更改在受影響電腦使用過的帳號密碼，撤銷並重發曾使用或儲存的 API 金鑰與 access token；疑似外部帳號或金錢損失可向客服申報。本表仍建議用手機或另一台乾淨裝置處理帳號，不必等電腦清完。
- **後續防護承諾：** 官方將強化監控、建立外部資源完整性驗證，並檢討、降低啟動器權限。公告沒有表示這些措施都已完成，入侵根因及受影響程度仍在調查。

**任何啟動器更新、重開或清除之前，先保全證據。** 若要留存、交給 IT 或釐清個案，請先依[清理前備份](#4-清理電腦)保存日誌、快取與報告，不要等到清除同意視窗才備份。這是本表的證據保全提醒，不是官方已提供備份功能。已隔離的電腦維持離線，**不要只為了觸發掃描或確認有沒有提示而重開啟動器**。官方手動指南同樣要求先斷網、完全關閉遊戲、啟動器與 Chrome，檢查完成前不要重開。

**11/5 17:28 發表、17:29 更新（台灣時間）的[單一玩家回報](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/25#issuecomment-6654463139)：** 該玩家自述把現有安裝移到舊筆電測試更新後，`debug.log` 被刪除，且沒有顯示結果。**本表未獨立重現；留言未交代確切版本、刪除機制或是否曾同意清除，不能推論所有更新都如此，也不能宣稱 ACE 未經同意刪除。** 這項未驗證回報補強了「更新或重開之前就先備份」的保守建議，沒有改變官方公告所述的同意流程。

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
| **11/5 19:22–19:23** | 本表唯讀複核：Verisign RDAP 顯示 `insightjs.com` 為 `client hold`，Google DNS 對主網域、`www` 與 `analytics` 的 A 查詢均回覆 NXDOMAIN；只是當時的網域狀態，不代表永久失效或主機安全 |

**11/4 13:52 本表抽查已有改善，但只是有限觀察。** 當時的檢查只讀取這四個檔案並計算雜湊，沒有執行 JavaScript。`FeedList` 這次收到 5,099 bytes，SHA-256 為 `49b625b30b34dc1fa16edba30bdf63c18b6171a0fc4cf030e21ea4e1263ae18a`，與[已知乾淨版對照](https://github.com/WormW/nikke_malware_analysis/blob/a2cb84787d334ef6027df45a216eb929c8c79149/docs/en/attack-path.md#211-official-asset-remediation-status-2026-11-04-re-check)相符。回應的 `Date` 是 03:12 UTC，並非本次取得時間，因此不能當作即時源站或全節點驗證。

上午取得污染副本的紀錄仍保留為歷史觀察；上述單一路徑抽查也不能證明所有地方都已安全，更不會清掉玩家電腦上的舊快取或既有感染。11/5 官方已公布上述處理方式，但完整影響範圍與入侵根因仍待釐清。

**15:14–15:20 的複查仍有範圍限制。** [WormW 最新複查（固定版本）](https://github.com/WormW/nikke_malware_analysis/blob/07f1bf4b2b9f16a83dd101a33c0eabf81cd2fb8a/docs/attack-path.md#211-官方资产当前状态2026-11-04-复核)確認上述四個物件在本次請求中已沒有已知注入特徵，但外部回應不能完全區分源站與中間快取，也不能排除其他 CDN 邊緣節點或玩家本機仍有舊檔。攻擊者如何取得寫入權限、是否還有其他受污染檔案，都尚未釐清。**這不是全面安全或感染電腦已清理的證明。**

**11/4 13:53 研究補充（發布時間）：單機疑似早期中斷。** WormW 根據一台電腦的兩次日誌截圖，記錄到 `k2f_chan registered` 後出現 `invalid UTF-8` 錯誤；截圖範圍內未見 `adopted` 或 `agent started as`，因此完整遠控會話是否建立仍未證實。原始完整日誌未提供，截圖未公開入庫，研究者也未直接檢查該主機。「中文使用者路徑導致中斷」只是未證實的假說。**這個案例仍已開始執行木馬，不能當作安全或沒有外洩的證明，也不要為了驗證而重開啟動器。** [WormW 單機觀察（固定版本）](https://github.com/WormW/nikke_malware_analysis/blob/1b7a2ca164654fd1a10464e95df56c082cb026dd/docs/field-observations.md)

**11/5 06:41 研究者補充：疑似使用者名稱編碼問題。** MotherSphere 在 [Issue #16 回覆](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/16#issuecomment-6644006257)中，依反編譯內容提出更具體的解釋：`sysinfo` 以 `os.getenv("USERNAME")` 讀取 Windows 系統碼頁的使用者名稱；若位元組不符合 UTF-8，JSON 編碼器就會報錯，再由 `k2f_chan` 回傳 `EIO`。作者因此將原因指向使用者名稱的編碼，而非中文路徑本身。作者另在 [Issue #21 回覆](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/21#issuecomment-6643556395)指出，處理函式捕捉錯誤後仍維持註冊，不會因此自動解除或刪除自身。**這是作者的程式碼分析，本表未獨立核驗反編譯內容、個案完整日誌或實際載荷版本；原始 `l.js` 仍未取得，錯誤後的行為無法確認。** 不能推論所有非 ASCII 使用者名稱都有此效果，更不能當作防護或沒有外洩的證明；既有感染判定與處置不變。

**11/7 07:10（台灣時間）[作者再澄清 #16](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/16#issuecomment-6698091916)：後來的 EIO 或檔案消失，不能推翻先前已載入本體。** 作者依所取得的個案日誌說明順序：下載完成，`l.js` 記錄 `lua downloaded to` 並載入 agent，之後才出現 `k2f_chan registered`；該玩家的 EIO 在註冊標記後約 87 毫秒才發生。因此，不能用該次錯誤或事後找不到 205,135-byte 檔案，推論下載從未完成或本體從未載入。

作者引用另兩例「下載後 207 毫秒」與「agent 啟動後約 4 秒」檔案消失的回報；這些仍是個案觀察。他推測 #16 的檔案較可能由啟動器或 `l.js` 刪除，**刪除者及觸發方式未確認**。官方 ACE 的實際檢查條件仍未公開，不能把玩家猜測的「只因日誌字串觸發」寫成事實。**本表未獨立核驗原始日誌、樣本或刪除過程；既有 `k2f_chan` 感染判定與處置不變，也不由此認定個案資料已外洩。**

**19:20 的複查範圍擴及四個 CDN 節點。** [MotherSphere 晚間更新（固定版本）](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5/4d9b02aa437755f376edb96403042c94898bda0a)回報，四個已知 JS 在四個 CDN IP、兩個主機名上均為乾淨內容，也檢查了 gzip 與帶查詢參數的回應。研究者因此建議：重新開啟前，先檢查或保全日誌、快取，再清除 `tbs_cache`，並持續封鎖已知攻擊網域與 IP。**這不是官方全節點安全確認**；未抽查節點、入侵根因仍有疑問，本機舊快取及既有感染也不會自動清除。封鎖目前的網域與 IP，亦不能防止攻擊者改用新位置。

**11/5 19:22–19:23（台灣時間）網域狀態複核。** MotherSphere 的[最新回覆](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5?permalink_comment_id=6459160)回報 `insightjs.com` 已遭註冊商暫停解析（registrar hold）。本表隨後直接讀取 [Verisign RDAP](https://rdap.verisign.com/com/v1/domain/insightjs.com)，確認狀態含 `client hold`，名稱伺服器為 `NS1.SUSPENDED-DOMAIN.COM`、`NS2.SUSPENDED-DOMAIN.COM`；同時透過 Google Public DNS 查詢 [`insightjs.com`](https://dns.google/resolve?name=insightjs.com&type=A)、[`www.insightjs.com`](https://dns.google/resolve?name=www.insightjs.com&type=A)、[`analytics.insightjs.com`](https://dns.google/resolve?name=analytics.insightjs.com&type=A) 的 A 紀錄，三者均回覆 `Status: 3`（NXDOMAIN），沒有位址答案。RDAP 的 `last changed` 為 **11/5 10:05:09（台灣時間）**，但不是專門記錄 hold 生效時間的欄位，不能單憑它確定精確停用起點。

**這不是永久失效、其他途徑已排除或感染電腦安全的證明。** [ICANN 對 `clientHold` 的說明](https://www.icann.org/resources/pages/epp-status-codes-2014-06-16-en)指出此狀態會停止網域的 DNS 解析，也可以解除。以上只反映該時段的註冊資料及一個公共解析器的查詢，不涵蓋所有 DNS 快取、`insightjs.org`、直接 IP 或其他投遞／遠控位置，也不會清除既有感染或撤回外洩資料。本次沒有造訪可疑站點、連線已知攻擊 IP 或執行載荷；**既有網域／IP 封鎖、證據保全、隔離及帳號保護建議不變，感染判定與腳本未修改**。

**11/6 痕跡歸屬補充。** [MotherSphere 同日回覆](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5?permalink_comment_id=6461096)將 `k2f.agent.state` 的寫入歸於 `l.js`，而非 Lua 本體，並稱其收到的此項命中案例都伴隨本體執行。**這是研究者歸屬及個案觀察，原始惡意 `l.js` 尚未取得，本表未獨立核驗；不能據此推導第一階段全部能力。** 既有感染判定與檢查腳本不變。

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

**11/6 18:51（台灣時間）工具判讀補充：下載或檔案存在，不等於 Lua 木馬本體已執行。** 作者在 [Issue #33 回覆](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/33#issuecomment-6682943728)（18:40 發表）承認，v0.5.2 只有下載及檔案跡證時，仍會把「Did the malware run on this computer?」回答為 Yes，這個執行結論過強；並表示將改為「Cannot tell」、補列 `startup failed: lua download timed out`。**截至本次查核，最新正式版仍是 v0.5.2，不能把修正承諾當作已發布。** 看上游報告時應核對 Details 的具體證據，不要只憑 Infected 或 Yes 標題認定本體執行；本表既有腳本與感染判定不變。

**該 timeout 個案仍未獨立核驗，不能當作安全證明。** 作者依玩家提供內容推測該次下載完成時，第一階段已停止等待；這不同於已出現 `k2f_chan` 後才啟動失敗，也不同於下載始終為 0 bytes。應核對同次下載結果、時間順序與完整 `debug.log`，搜尋正確字串 `k2f_chan`（不是 `k2f_registered`）。**缺少標記的解讀受日誌完整性、刪除／覆寫與掃描範圍限制；單憑 timeout、缺標記、檔案留下或沒有 DLL，都不能排除其他次執行或資料外洩。** 原始惡意 `l.js` 仍未取得，第一階段完整能力未明；保留證據與帳號防護，不以這則個案進一步降級。

**11/6 21:20（台灣時間）[作者續覆 #33](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/33#issuecomment-6685803432)：補上單一個案的時間差，仍不是安全結論。** 作者依該玩家日誌（當地 UTC+11）列出：23:35:41.522 加入下載任務、23:35:42.802 發出 GET、23:35:56.518 記錄 timeout，至 23:35:57.625 才完成 205,135 bytes 下載。即加入任務約 15 秒後停止等待，比下載完成早約 1.1 秒；實際下載約 14.8 秒，無法由日誌判定是伺服器或連線緩慢。作者據此及未見 `k2f_chan`，認為本體在**這台電腦最可能未載入**；單次紀錄不足以確定通用 timeout 設定，也不能套用到其他玩家。`l.js` 已在啟動器頁面執行，其原始惡意版本仍未取得，完整能力未明。**本表未獨立核驗原檔，不改變既有感染判定或帳號處置。** 作者同時明說，前述報告修正尚未完成；本次複核最新正式發布仍為 v0.5.2。

作者另表示，玩家提供的 SHA-256 與已公布樣本相同（`cc0ad05bfe85940a6067b3a72c182d3953339129206634141926d5d0b35973ec`），把 ETag `6ae90538-3214f` 與 `6ae9f628-3214f` 對應到相同位元組。這是**研究者依玩家雜湊作出的構建關聯，本表未重新取得樣本驗證**；中間的 `6ae9d408` 尚無公開雜湊，是否相同仍未知。ETag／`Last-Modified` 不能直接等同上線時間；玩家電腦時鐘與同段 HTTP `Date` 也待核對，不能據此擴大或縮短感染期間。

**11/7 07:10（台灣時間）[作者澄清 #33](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/33#issuecomment-6698091445)：擬修正的是「本體是否執行」，不是把整體 Infected 降級。** 對上述僅有下載跡證、未見 `k2f_chan` 的 timeout 個案，作者明確表示，nikke-check 的整體判定仍為 **Infected**，保留報告全部處置，包括建議重灌 Windows；只有「Did the malware run on this computer?」擬改為 **Cannot tell**。理由是無法排除已執行的 `l.js` 在啟動器頁面做過什麼，這是研究者的保守處置立場，**不是新增證據證明第一階段已獨立竊密或該個案已外洩**。

**截至 11/7 07:17（台灣時間）複核，這仍是尚未發布的修正。** 作者表示新版發布後會再通知，最新正式發布仍為 [v0.5.2](https://github.com/MotherSphere/NIKKE-Launcher-Check/releases/tag/v0.5.2)。上游工具的 Infected 與本表「中毒／高風險／有接觸」不是同一套分類，**本次僅更新研究說明，不改本表既有腳本、感染判定或處置步驟，不以此降低風險，也不把作者的個案處置擴套成所有僅有接觸者均已感染。** 此例「下載最終完成但等待逾時」仍須與「下載終結為 0 bytes」分開，不能互相套用；本表未獨立核驗其原檔，原始惡意 `l.js` 仍未取得。

**分享報告前先去識別化，不要上傳完整快取、下載檔或 DLL。** 作者提醒 `tbs_cache` 含 Cookie／Local Storage、`tinyget.mem.log` 含 Cookie／登入票證；偽裝為 tinyget 日誌的下載檔可能是可運作的木馬。需要求助時使用工具的「Hide my info」（`--redact`），再人工檢查最小必要節錄；只提供必要時間、錯誤行或 SHA-256，不公開原檔，也不要為了辨識而開啟或執行可疑檔案。

### Google Play Games 版：已公開命中節錄，完整原檔未核驗

**11/5 03:39（台灣時間）發表、03:41 更新的[上游 Issue #11 留言](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/11#issuecomment-6640267816)**，一名使用 Google Play Games 安裝 NIKKE PC 版的玩家貼出 PowerShell 搜尋摘要：`C:\NIKKE_GOOGLE\Launcher\intl_service\debug.log` 命中 `k2f_chan`，兩個 tinyget 日誌命中 `insight.bundle`。玩家另回報執行檔名為 `nikke_launcher_google.exe`，與一般版不同。

**作者於 11/5 06:41 [回覆](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/11#issuecomment-6644005418)，依當時的搜尋摘要判定木馬曾在該電腦執行，Google Play Games 版也載入了污染檔案，並建議按感染處理。** 玩家於 **11/5 10:43:31（台灣時間）[補充公開命中節錄](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/11#issuecomment-6648246141)**，共有 7 組 `lua downloaded` 與 `k2f_chan registered`，日期標記為 11/1、11/2（該電腦當地時間，未換算為台灣時間）。這比先前只有檔案路徑與關鍵字的摘要更具體，但仍是**玩家提供的公開節錄，完整原始日誌及下載任務結果尚未獨立核驗**，不能據此認定哪些資料外洩，或推論所有 Google Play Games 版玩家都受影響。

**節錄沒有其他字串，不代表原始日誌沒有其他遠控紀錄。** 玩家使用的指令只篩選 `lua downloaded`、`k2f_chan registered`，並未搜尋 `adopted` 或 `agent started as`；不能從這份輸出推定遠控會話未建立、木馬早期中斷或沒有外洩。玩家另回報指定 DLL 與登錄鍵檢查未命中，並嘗試對照 NetworkUsageView／SRUM 時間；這些仍是個案回報，不能排除其他常駐，也不能把時間相關或流量紀錄直接認定為外洩內容。

**tinyget 檔名日期不能直接當成請求日期。** 作者於 **11/5 17:14（台灣時間）[更正先前說法](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/11#issuecomment-6654206829)**：檔名反映建立時間，文字日誌可能繼續記錄同次啟動中較晚的請求；需核對命中行的時間戳及檔案實際內容。不能只憑 `tinyget.20261028-103204-028.log` 的名稱認定 10/28 已下載木馬，也不能把這項更正當作其他較早研究紀錄已被推翻。本表尚未取得該檔案的補充核對結果。

該玩家先前沒有找到 `tbs_cache`，但自述可能曾手動刪除；10:43 的留言又回報，重開後在 `%APPDATA%\nikke_launcher_google` 產生新快取。這是其個案的路徑觀察，缺少舊快取仍不能當作未受影響的證明。**不要為了重現或確認資料夾而重開啟動器。** 使用此版本的玩家，請先保留現有日誌與資料，核對檢查報告是否涵蓋實際安裝位置（該回報為 `C:\NIKKE_GOOGLE\Launcher`）與 `%APPDATA%\nikke_launcher_google`，不要只因資料夾沒有命中就排除。**本次只補充回報與限制，不改動既有腳本或感染判定。**

### 11/5 檢查限制與誤報修正

- **舊版 Temp 目錄誤報已獲作者確認。** [Issue #19 作者回覆（11/5 06:41）](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/19#issuecomment-6644006713)確認，兩位玩家的舊 `k2f` 開頭目錄是因名稱被誤列 critical；v0.5.2 已改成只比對 `k2f-`、`k2f_` 開頭的一般檔案，目錄不再算。**本表腳本尚未同步修正**，仍可能命中這類項目。若舊報告只有這一項，先保留資料、核對完整報告，再用新版重查；不要只憑模糊名稱認定感染，也不能因這項誤報被排除就證明整台電腦安全。實際惡意檔案、`k2f_chan` 等其他跡證不受此修正影響。
- **gzip HTTP 快取漏看已獲作者實機觀察支持，v0.5.2 仍未修正。** 先前[玩家回報](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5#gistcomment-6457040)解壓後才找到污染檔案；[MotherSphere 隨後回覆](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5?permalink_comment_id=6457471)，自己的快取也有 gzip 內容，解壓後找到污染的 `images` 與 `aegis-fix`，並承認工具有相同盲點。靜態核對 [v0.5.2 快取掃描（固定版本）](https://github.com/MotherSphere/NIKKE-Launcher-Check/blob/07cf01ca36ee4504983bb610d2dd5f5dec9daf76/src/scan.rs#L661-L705)，只有 `.ldb` 會先解析，其餘仍搜尋原始位元組。**這支持漏檢機制，並非已獨立核驗每位玩家的原始快取**；「純文字沒命中」或「已支援 LevelDB 解壓」都不能證明 HTTP 快取乾淨。

先保存快取與報告，不要為了驗證而重新開啟啟動器。**以上限制不會推翻實際日誌中的 `k2f_chan registered`、已確認的惡意 DLL 等具體跡證。** 本次只補充上游修正與證據限制，既有腳本與判定未修改。

**11/7 07:11（台灣時間）[作者澄清更新後防毒命中](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5?permalink_comment_id=6462582)：先分清舊痕跡與新執行。** 作者指出，已辨識的 Loader code cache 項目只含載入位址與時間，不含腳本；更新後的 `Trojan.Loader.K2F` 命中可能是留存的舊痕跡，**不能僅憑此認定再次感染或補丁失敗，也不能反過來當作電腦安全的證據**。仍須核對完整日誌與其他跡證；先保存證據，不要為了驗證而重開啟動器。

同一回覆分析的 Norton 警示，只表示它回報攔截了該次 Chrome 儲存密碼讀取，**不能證明 Cookie、其他瀏覽器或其他指令也被攔下**，既有改密碼與撤銷工作階段建議不變。這些是研究者對玩家貼文／截圖的解讀，部分路徑被截斷；**本表未獨立核驗原檔、實機或防毒紀錄，不據偵測名稱或檔案日期新增本事件 IOC，也不改感染判定**。

### 早期投遞腳本的證據界限

**11/5 補充研究限制，並非新增感染案例。** [MotherSphere 原留言](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5?permalink_comment_id=6456552)指出，該討論中尚無人公開早期負責選擇下載位置、投遞木馬的 `l.js`。[WormW 固定版本分析](https://github.com/WormW/nikke_malware_analysis/blob/07f1bf4b2b9f16a83dd101a33c0eabf81cd2fb8a/docs/attack-path.md#0-证据来源与结论边界)取得的 9,145 bytes 副本，是 **11/4 01:30（台灣時間）後被換成的 `/speed` 分析版本，不是早期惡意投遞版**。不能用這份事後副本證明先前腳本沒有其他功能。

MotherSphere 表示，該討論已貼出的 `insight.bundle` 下載網址都指向 `analytics.insightjs.com`，尚無其他主機或 IP 投遞的回報；但缺少當時的 `l.js`，**仍不能完整排除其他投遞路徑**。已知紀錄支持透過 `Srv_DownloadFile` 下載後，木馬隨即在啟動器內執行；具體由哪個原生橋呼叫觸發執行，仍未確認。

**11/5 17:14（台灣時間），作者補充了 Lua 樣本的觸發界限。** [Issue #17 回覆](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/17#issuecomment-6654207517)稱，所分析的 11/3 構建最上層只註冊 `k2f_chan`，包括 `keyscan` 在內的命令由指定命令名稱的 `open` 請求觸發；伺服器回應則由 `l.js` 處理。**本表未獨立核驗反編譯內容，不能套用到所有構建；因缺少當時的 `l.js`，仍無法判定 `keyscan` 是腳本啟動時自動要求，還是遠端指令觸發。** 不能把「Lua 不自行發起命令」解讀成初始腳本沒有其他能力或沒有外洩。

**同日 17:14 的 [Issue #25 作者回覆](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/25#issuecomment-6654208278)僅支持兩台電腦的個案觀察。** 作者表示，兩例都在啟動器頁面載入時執行 loader，之後保持開啟數日而未重載頁面，紀錄未見要求下載木馬；作者明說兩台不足以成為通則，也不知道所有會重載頁面的情境。本表未獨立核驗完整原檔，**不能據此建議保持啟動器開啟，或保證其他僅有接觸紀錄的人安全**。

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

拔網路線或關 Wi-Fi。在右下角系統匣對啟動器按右鍵，選結束。之後不要再用這台電腦登入任何服務。如果是公司電腦，或電腦裡有公司帳號、憑證，先找公司 IT。若受影響電腦可讀取公司分享資料夾或同步文件，也把這些可能暴露的資料範圍交給 IT 評估；不要自行刪除共用資料或把它們上傳公開查核。

### 2. 用手機或另一台乾淨的電腦處理帳號

照這個順序：

1. **主要信箱**（Google、Outlook 等）：它能拿來重設其他帳號，最優先。
2. **NIKKE／Level Infinite 帳號**。
3. **這台電腦瀏覽器存過密碼的帳號**：Discord、Steam、網路銀行等。
4. **受影響電腦可存取的 NAS／檔案分享與雲端同步帳號**：從乾淨裝置檢視存取、登入紀錄及共用權限；若相關密碼、憑證或工作階段可能暴露，更新或撤銷它們。公司帳號交由 IT 協調，避免擅自中斷共用服務。這是依讀檔能力提出的保守處置，不是已確認這些服務被入侵。

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

**11/6 21:32（台灣時間）[作者補充取證方式與工具涵蓋限制](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/32#issuecomment-6686028635)。** 若需要釐清已確認常駐的電腦或交給 IT，請在清除、重開或重灌之前，由具備經驗的人員保全：

- **先記錄原位置的資料夾／DLL 路徑、大小、UTC 建立與修改時間及 SHA-256，再複製。** 副本可能產生新的建立時間，不能拿副本時間反推感染時點。記下擷取時間與時區；輸出明確指定到另一顆磁碟或離線隨身碟，不要只給檔名，避免管理員終端機把檔案寫進即將清除的 `C:\Windows\System32`。
- **登錄取證須對準受影響的 Windows 使用者。** 作者建議用 `reg save` 保全已知 HKCU CLSID 鍵的 hive 與最後寫入時間，而不是會漏掉這項時間的 `reg export`。若提權時換成另一個管理員帳號，`HKCU` 可能指向錯的人；不能把該帳號的空結果當成受影響帳號陰性。BAM 程序活動則在 **`HKLM\SYSTEM` 的 SYSTEM hive**，不是 HKCU；Prefetch 與 SYSTEM hive 可交給 IT 交叉分析。作者稱 BAM 記錄約保留最近一週，實際可用範圍受系統與保留狀況影響。這些活動痕跡本身不能證明由木馬 `exec` 觸發，缺少痕跡也不能排除執行。
- **清理程式提到的位置，只能作待核對線索。** 作者指出所分析 agent 的清理碼會刪除 Chrome `Application\PlatformExperienceHelper`（`Program Files`、`Program Files (x86)` 與 `%LOCALAPPDATA%\Google\Chrome\Application` 下）中的 `platform_experience_helper.dll`、`platform_experience_helper.dll.old`、`old-clsid.txt`，以及工作目錄中的 `service_util.dll`；工作目錄「最可能是 NIKKE `Launcher`」仍是推測。這暗示可能有較早版本使用這些名稱，**不是已核實各位置、各同名檔案皆具惡意用途，也不是新增投遞路徑或可直接刪除的 IOC 清單**。若有發現，保留原件資訊與離線副本供分析，不要執行、公開上傳或只憑名稱刪除。
- **不要把不同工具的涵蓋範圍混在一起。** 作者稱 nikke-check 未查上述清理線索位置；本表既有 PowerShell 已列三類 Chrome `Application\PlatformExperienceHelper` 位置與前三個檔名，但未新增針對 `Launcher\service_util.dll` 的專門檢查。這次僅補充文件，不修改腳本、既有判定或擴大刪除清單；陰性結果不能用來保證所有位置都已查過。

**11/7 07:10（台灣時間）[作者補充 USN 的能力與盲點](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/32#issuecomment-6698090450)。** NTFS USN journal 可留下檔案建立、更名、刪除等變更紀錄，協助查找下載、暫存副本或截圖痕跡；它不是完整的檔案存取或網路傳輸紀錄，**單純讀取、外傳未必留下可辨識的紀錄，沒有命中不能證明沒有讀取或外洩**。須先核對最早保留紀錄是否涵蓋事件時間，舊紀錄可能已被覆蓋；另核對 **NIKKE 實際安裝磁碟**與 `%TEMP%` 所在磁碟，不要只查 `C:` 就認為完整。作者所述 `tinyget` 下載痕跡在遊戲安裝磁碟；個案時序及檔案名稱仍需交叉核對，不能直接推定執行或資料外洩。

USN 的一般用途亦可參考 [Microsoft 文件](https://learn.microsoft.com/en-us/windows/win32/fileio/change-journals)（2021-01-07 更新；本次核對）：它按磁碟區記錄檔案系統變更。**本表未獨立核驗上述玩家的原始 USN 或樣本，不新增取證腳本或感染判定。** 如需保全，請由有經驗的人員離線處理，輸出到另一顆磁碟；完整 journal 可能包含私人檔名，勿公開上傳。

登錄 hive、Prefetch、日誌與 DLL 都可能含私人資訊；僅離線保存並交給獲授權的協助者，不上傳公開 repo、議題或樣本服務。這些是研究者提供的保全方向，本表未在玩家電腦驗證。

**最保險的做法是重灌 Windows。** 木馬能執行任何指令，沒人知道攻擊者當時還做了什麼。先備份個人檔案（照片、文件），不要備份舊的程式、整個瀏覽器設定或啟動器快取，再用可信的安裝媒體重灌。

**11/6 18:51（台灣時間）常駐範圍提醒：不要把 DLL 的載入程序限定為 Chrome。** [單一玩家回報](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/32#issuecomment-6682079803)自述 `steamwebhelper.exe` 與 `runeapps.alt1.exe` 載入同一個 Chrome 目錄內的 DLL；[作者回覆](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/32#issuecomment-6682943355)（18:40 發表、18:45 更新）以所分析構建的 **HKCU COM 劫持**提出解釋；[21:32 續覆](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/32#issuecomment-6686028635)明確保留為**最可能的推測，尚未檢查 Steam／Alt1 構建**：同一使用者下會要求該 Windows Parental Controls COM 物件的其他相容程序，也可能載入已落地的 DLL。**本表未獨立核驗該玩家實機或 DLL，作者也未核驗 Steam／Alt1 的構建；其他 Chromium 程式尚未經此回報驗證。這不是 Steam、Alt1 或其他遊戲成為新投遞來源的證據，也不證明其中資料已外洩。**

若報告已列出 DLL 被程序載入，維持離線，從系統匣完整結束相關主程式；只關視窗或結束 `steamwebhelper.exe`，主程式可能再次啟動它。仍按下列順序**先移除已確認的 HKCU 常駐鍵、保留正常 HKLM 的 `wpc.dll`，再重開 Windows 使各程序卸載 DLL**，刪除已確認的惡意子資料夾並複查；不要為了測試載入而開啟其他程式。從乾淨裝置保護可能在受影響程序輸入或顯示過的帳號資料；例如使用過 Steam／Alt1 者，檢視相關帳號並撤銷工作階段。這是保守處置，不是宣告每位玩家都受此路徑影響。

作者另提醒 DLL 可能嵌入受害者 agent ID 與 token；**不要將 DLL 上傳公開議題或公開樣本服務**。程序掃描也可能因權限跳過部分程序，沒有列出載入項目不等於排除常駐。

暫時不重灌的話，至少要移除已知的常駐（全程保持離線）：

1. 刪除登錄機碼 `HKCU\Software\Classes\CLSID\{E77CC89B-7401-4C04-8CED-149DB35ADD04}`（整個機碼）。`HKCU\Software\Classes\WOW6432Node\CLSID\` 底下如果也有同一個，一起刪掉。**`HKLM` 底下同一個 CLSID 是 Windows 自己的（指向 `wpc.dll`），不要動。**
2. 重新開機。DLL 使用中會被鎖住，重開才刪得掉。
3. 刪除 `PlatformExperienceHelper` 底下那個 64 碼的子資料夾。**只刪那個子資料夾**，不要刪整個 `PlatformExperienceHelper`；裡面的 `platform_experience_helper.exe`、`LoadFeatures` 是 Chrome 正常的檔案。一鍵檢查如果有列出「舊版常駐檔案」，也一起刪掉。
4. 再跑一次一鍵檢查，確認都清掉了。

Defender（包括離線掃描）曾經漏掉這顆 DLL，**掃毒顯示乾淨不代表沒事**。手動移除也只處理已知的部分；攻擊者如果另外裝了別的東西，只有重灌才能確定清掉。

### 5. 重新使用啟動器之前

- 先完成安全檢查與必要處置。官方已公告修復與清除功能，但本表未實測其部署或效果；不要把沒有跳出警示當作電腦乾淨的證明。需要留證時，在任何啟動器更新、重開或同意清除前先備份。
- 先刪掉啟動器快取 `%APPDATA%\nikke_launcher*\tbs_cache`，裡面可能還留著被改過的檔案。需要留存資料的話，先照第 4 步複製。[刪除指令在這裡](WINDOWS-CHECKS.md#重新開啟動器前先刪快取)。
- 之後幾週多留意帳號的登入紀錄和金融交易。

## 沒中也建議做的預防

**11/6 防火牆提醒：已建立封鎖規則，不代表正在生效。** 請另外[唯讀核對活動網路、適用 profile 與政策](WINDOWS-CHECKS.md#先確認-windows-防火牆規則是否適用唯讀)；第三方或公司防火牆接管時先找管理者，不要逕自更改設定。新增提醒源於一則未獨立驗證的玩家回報，不能歸因於惡意程式，也不改變感染判定。

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

目前未見可靠公開證據顯示其他遊戲啟動器遭同一路徑入侵。網路上流傳「Level Infinite 旗下 PC 遊戲都要警戒」的清單，以及「王者榮耀、三角洲行動也確認中了」的說法，本表尚未找到支持來源。

**11/6 騰訊立場（媒體轉述，11/7 09:54 台灣時間核對）：** [Inven Global 報導](https://labs.invenglobal.com/articles/28258/tencent-says-nikke-hack-does-not-apply-to-other-games)稱，騰訊依截至當時的調查及技術評估，未發現其他旗下遊戲受影響的跡象，並以各遊戲啟動器架構不同為由，表示本次已識別的攻擊路徑不適用於其他遊戲。**本表尚未找到騰訊原始聲明，也未獨立核驗其調查或各遊戲架構；這是媒體轉述的官方立場，不是所有遊戲或玩家電腦都安全的證明。**

- 研究者 MotherSphere、WormW 與先前 Inven Global 報導所確認的初始入侵範圍仍是 NIKKE，不能把發行商或投資關係當成其他遊戲受害的證據。
- 已知初始投遞流程依賴 NIKKE 啟動器內建瀏覽器的本機功能；這不代表已落地的常駐 DLL 只能在 NIKKE 程序內載入，詳見[常駐範圍提醒](#4-清理電腦)。
- 那份清單把騰訊投資的工作室和 Level Infinite 發行的遊戲混在一起。例如 Valorant 是 Riot 的遊戲，《鳴潮》是庫洛自己營運。

仍須保留以下限制：

- 4 個被改的檔案裡，有 2 個放在共用路徑 `/comm/assets/lib/` 底下，研究者提醒同一個儲存空間的其他檔案也可能被改過。上述轉述沒有提供攻擊者取得 CDN 權限的根因、完整受查檔案或逐遊戲驗證細節，不能據此獨立排除所有其他投遞路徑。
- 木馬會偷 NIKKE 的登入資料。其他遊戲如果也用同一個 Level Infinite 帳號，那些遊戲的帳號一樣有風險，改 NIKKE 密碼時就一起處理了。**其他遊戲未見同路徑入侵，不會清除已受感染的電腦，也不能排除其中帳密、Cookie 或其他資料已暴露。**

目前沒有依據要求所有玩家僅因共用發行商就停玩其他遊戲；**已受影響的電腦仍須依既有隔離、保全與清理步驟處理，不要用它登入其他遊戲來測試。** 從乾淨裝置保護共用帳號並留意後續公告；本次不改腳本、感染判定或處置步驟。

### 手機版、Linux／Steam Deck 呢？

**11/5 [官方手動指南](https://nikke-en.com/newsdetail.html?content_id=ecdccd40a55a5a46aaa9abda64d436082db7)明確表示手機版本未受這次事件影響，只玩手機版的人不需為此事件額外檢查。** 這不涵蓋同時在 PC 版使用過的帳號。用 Linux 或 Steam Deck（Proton）跑 PC 版仍有研究者記錄的風險；官方這份 Windows 指南不是 Proton 的安全確認。本表指令只適用 Windows，其他系統請看 [MotherSphere 的檢查工具說明](https://github.com/MotherSphere/NIKKE-Launcher-Check)。

## 資料來源

- [11/6 騰訊稱其他遊戲未見受影響（Inven Global 媒體轉述）](https://labs.invenglobal.com/articles/28258/tencent-says-nikke-hack-does-not-apply-to-other-games)：11/7 09:54（台灣時間）核對；英文頁首標示 11/5、內文稱聲明在 11/6，本文按轉述標明聲明日期；尚未找到騰訊原文，調查與其他遊戲架構未獨立核驗，不能當作主機或共用帳號安全證明

- [11/7 更新後 Loader 快取與防毒攔截判讀（MotherSphere 回覆）](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5?permalink_comment_id=6462582)：07:11:18 發表、11/7 08:37（台灣時間）核對；舊痕跡不等於再次感染，單次攔截不涵蓋其他存取；玩家原檔與實機未獨立核驗
- [11/7 USN 能力、保留範圍與安裝磁碟提醒（Issue #32 作者續覆）](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/32#issuecomment-6698090450)：07:10:47 發表、11/7 08:37（台灣時間）核對；變更紀錄不能排除純讀取／外傳，個案原始 journal 未獨立核驗

- [11/7 timeout 判讀與處置澄清（Issue #33 作者續覆）](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/33#issuecomment-6698091445)：07:10:50 發表、07:17（台灣時間）核對；Infected 與完整處置不降級，僅本體執行問答擬改 Cannot tell，最新正式版仍 v0.5.2；個案原檔及原始惡意 `l.js` 未獨立核驗
- [11/7 註冊標記、EIO 與檔案消失澄清（Issue #16 作者續覆）](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/16#issuecomment-6698091916)：07:10:52 發表、07:17（台灣時間）核對；作者依個案日誌判斷本體先載入，刪除機制及 ACE 觸發條件仍未確認，本表未獨立核驗原檔或實機

- [11/6 取證修正與清理碼線索（Issue #32 作者續覆）](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/32#issuecomment-6686028635)：21:32 發表、21:51（台灣時間）核對；原檔時間、使用者 HKCU 與 SYSTEM hive 須分清；位置尚非已證實惡意用途的新 IOC，Steam／Alt1 COM 解釋為最可能推測
- [11/6 timeout 時序與構建關聯（Issue #33 作者續覆）](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/33#issuecomment-6685803432)：21:20 發表、21:51（台灣時間）核對；單例原檔未獨立核驗，中間構建與原始惡意 `l.js` 仍未知，報告修正尚未完成

- [11/6 COM 常駐機制與處置補充（Issue #32 作者回覆）](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/32#issuecomment-6682943355)：18:51（台灣時間）核對；Steam／Alt1 為單一玩家自述，實機及 DLL 未獨立核驗，不是新投遞路徑
- [11/6 下載與本體執行區分（Issue #33 作者回覆）](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/33#issuecomment-6682943728)：18:51（台灣時間）核對；timeout 個案未獨立核驗、原始惡意 `l.js` 未取得，v0.5.2 報告修正仍為承諾，最新正式發布頁仍為 v0.5.2

- [MotherSphere 11/6 研究回覆](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5?permalink_comment_id=6461096)：11/6 11:43（台灣時間）核對；補充 11/3 構建的 UNC／NAS、本機同步檔案讀取範圍及 `k2f.agent.state` 歸屬，未獨立核驗樣本或原始惡意 `l.js`，不代表個案已外洩

- [MotherSphere 假冒「Cleanup Tool」警告](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5?permalink_comment_id=6459938)：11/5 22:42（台灣時間）核對；研究者警告留言誘導管理員執行遠端腳本，本表未取得或分析腳本，與原事件 IOC 分開
- [註冊商 hold 研究者回覆](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5?permalink_comment_id=6459160)、[Verisign RDAP](https://rdap.verisign.com/com/v1/domain/insightjs.com)與 Google Public DNS（[主網域](https://dns.google/resolve?name=insightjs.com&type=A)、[`www`](https://dns.google/resolve?name=www.insightjs.com&type=A)、[`analytics`](https://dns.google/resolve?name=analytics.insightjs.com&type=A)）：11/5 19:22–19:23（台灣時間）獨立唯讀複核；查詢結果可隨時間改變，不代表主機安全
- [MotherSphere 事件調查](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5)：痕跡、時間線、處置建議
- MotherSphere [NIKKE-Launcher-Check](https://github.com/MotherSphere/NIKKE-Launcher-Check)：[事件細節](https://github.com/MotherSphere/NIKKE-Launcher-Check/blob/main/docs/incident.md)、[處置方式](https://github.com/MotherSphere/NIKKE-Launcher-Check/blob/main/docs/what-to-do.md)、[檢查項目](https://github.com/MotherSphere/NIKKE-Launcher-Check/blob/main/docs/checks.md)
- [v0.5.2 正式發布（11/5 06:40，台灣時間）](https://github.com/MotherSphere/NIKKE-Launcher-Check/releases/tag/v0.5.2)：Temp 誤報修正、Google Play Games 版辨識與啟動失敗報告
- [Google Play Games PC 版玩家摘要](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/11#issuecomment-6640267816)、[作者回覆（11/5 06:41，台灣時間）](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/11#issuecomment-6644005418)與[玩家公開命中節錄（11/5 10:43:31，台灣時間）](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/11#issuecomment-6648246141)：查核至 11/5 10:53；已公開 7 組兩類字串節錄，完整原始日誌尚未獨立核驗
- [v0.5.1 被 Defender 隔離的玩家回報](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/22)與[作者更新（11/5 08:14，台灣時間）](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/22#issuecomment-6643556806)：作者主張誤報並表示已提交 Microsoft 申訴，議題仍開啟，未見廠商裁定
- [Temp 誤報確認（11/5 06:41，台灣時間）](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/19#issuecomment-6644006713)、[gzip 快取漏看作者回覆](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5?permalink_comment_id=6457471)
- [更新後日誌消失的單一玩家回報（11/5 17:28 發表、17:29 更新，台灣時間）](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/25#issuecomment-6654463139)：未獨立驗證，版本、刪除機制與同意流程不明
- [tinyget 日期更正](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/11#issuecomment-6654206829)、[Lua 命令觸發界限](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/17#issuecomment-6654207517)、[兩個 loader-only 個案的限制](https://github.com/MotherSphere/NIKKE-Launcher-Check/issues/25#issuecomment-6654208278)：作者均於 11/5 17:14（台灣時間）回覆；本表未獨立核驗個案原檔或反編譯內容
- [WormW 樣本分析](https://github.com/WormW/nikke_malware_analysis)：攻擊路徑、[11/4 CDN 複查](https://github.com/WormW/nikke_malware_analysis/blob/a2cb84787d334ef6027df45a216eb929c8c79149/docs/en/attack-path.md#211-official-asset-remediation-status-2026-11-04-re-check)
- [巴哈姆特整理文（11/4 更新）](https://forum.gamer.com.tw/C.php?bsn=36390&snA=23304)
- [11/5 官方道歉聲明](https://nikke-en.com/newsdetail.html?content_id=3a94ebf0ab6c5a4703aa87aa57d0b8cd3d4b)：11/5 08:22（台灣時間）讀取全文；目前啟動器安全為官方聲明，遊戲內補償細節未在此頁公布
- [11/5 官方後續措施](https://nikke-en.com/newsdetail.html?content_id=b4f23938a63a0a4e68aa577af395b1edb172)、[11/5 官方手動安全檢查指南](https://nikke-en.com/newsdetail.html?content_id=ecdccd40a55a5a46aaa9abda64d436082db7)：後續措施查核至 11/5 07:24、手動指南複核至 08:22（台灣時間），均已讀取全文；未實測 ACE 部署或清除效果
- [11/4 國際版官方首次 X 公告](https://x.com/NIKKE_en/status/2117817602659770434)、[台港澳官方公告頁](https://nikke.hotcool.tw/news.html)、[國際版官方公告頁](https://nikke-en.com/news.html)
- [Inven Global 報導](https://www.invenglobal.com/articles/28122/is-my-pc-infected-nikke-launcher-tampering-concerns-from-attack-vector-to-remediation)
