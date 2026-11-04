# NIKKE PC 啟動器木馬檢查表｜Windows 繁體中文版

**如果你在自己那份 NIKKE 啟動器的 `Launcher\intl_service\debug.log` 裡看到 `k2f_chan registered`，先讓那台電腦斷網，改用乾淨的手機或電腦保護帳號。** 這行是已知遠端控制程式執行過的證據。即使日期在 10 月、電腦沒裝 Chrome、沒有找到 DLL，也適用。日誌無法告訴你哪筆密碼真的被取走。[執行指標與樣本分析](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5#check-if-you-are-infected)

| 你想做什麼 | 從這裡開始 |
|---|---|
| 自己查 Windows | [圖形介面位置與可複製的唯讀 PowerShell 指令](WINDOWS-CHECKS.md) |
| 請有本機工具權限的 AI 助手協助 | [提示詞一：唯讀檢查；提示詞二：依報告處置](AGENT-PROMPT.md) |
| 已確定執行過 | 先看下方「受害後的處置順序」。帳號止損和主機復原可以同時進行。 |

本表涵蓋 2026 年 NIKKE **PC 啟動器**事件，指令適用於 Windows。資料核對至 **2026-11-04 上午（台灣時間，UTC+8）**。依據玩家保留的紀錄、研究者取得的快取與樣本整理。截至核對時，未查到營運方公布完整受影響區服、入侵根因或全面恢復使用的公告。若官方後續提供修復通知，請核對公告時間及適用版本。[台港澳官方公告](https://nikke.hotcool.tw/news.html) · [國際版官方公告](https://nikke-en.com/news.html)

**為什麼只開啟動器就可能中？** 正版啟動器內建網頁介面，會自動取得官方配送的 JavaScript。研究者發現其中的檔案被加入程式碼，會再向外部網域取腳本，利用啟動器提供的本機下載功能取得 Lua 遠控程式。後續是否下載、啟動或設定常駐，可以分別留下不同的跡證；看到其中一項時，就依該項能證明的範圍處理。[攻擊路徑與證據來源](https://github.com/WormW/nikke_malware_analysis/blob/main/docs/attack-path.md)

## 10 月的紀錄也算嗎？

| 時間（台灣） | 已知事件 | 怎麼用來判讀 |
|---|---|---|
| 9 月 27 日 | 研究者將 Level Infinite 官方 CDN 上四份被改動的 JavaScript 追溯至這天 | 能說明攻擊腳本何時開始配送，不能據此認定所有玩家當天就執行木馬。 |
| **10 月 24 日 00:40** | 研究者彙整到的最早木馬啟動回報，原時間為 **10 月 23 日 16:40 UTC** | 這是目前公開回報的下限。另有 Windows 玩家貼出相符的 10 月執行序列。 |
| 11 月初 | 更多玩家發現跡證。研究者比對出不同時間投遞的程式版本；部分開啟啟動器的時段沒有收到木馬 | 11 月才被廣泛注意，無法用來排除之前的感染。 |
| 11 月 4 日 06:26、10:01–10:07 | MotherSphere 於 06:26 仍見兩個 CDN 節點送出舊 `FeedList`；WormW 於 10:01–10:07 複核時，仍從官方網址取得相同惡意檔案 | 這些是研究者當時的取樣結果，不能當作全區服／全節點的狀態。不要重開啟動器測試。 |

**11 月 4 日上午的供應端複核。** WormW 在 **02:01–02:07 UTC（台灣 10:01–10:07）** 重取三份官方檔案，所取的 `aegis-fix.min.js`、`images.7fec4c32.js` 未見已知載入器特徵，但 `FeedList.feb647c9.js` 仍與已知惡意版本的 SHA-256 相符，且不同請求曾取得不同版本。這是研究者的觀察，並非營運方宣布全面修復；也不能據此判定其他檔案、CDN 節點或既有感染已處理。保留證據、暫停使用啟動器與復用前處理本機舊快取的原則不變。[WormW 複核紀錄（固定版本）](https://github.com/WormW/nikke_malware_analysis/blob/a2cb84787d334ef6027df45a216eb929c8c79149/docs/en/attack-path.md#211-official-asset-remediation-status-2026-11-04-re-check)

例如，在確認是 2026 年的日誌裡，`[1030/…] "k2f_chan registered"` 表示 **10 月 30 日當次已執行**。前四碼通常是月日，行首多半沒有年份或時區。若看到比 10 月 24 日更早的命中，請保留原始紀錄、核對系統時間與年份，並用同時段的下載紀錄或木馬狀態交叉驗證。不要因為早於已公布案例就忽略，也不要只用單一時間戳改寫整體事件時間線。[事件時間線](https://github.com/MotherSphere/NIKKE-Launcher-Check/blob/main/docs/incident.md#timeline) · [10 月 Windows 個案](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5?permalink_comment_id=6455530#gistcomment-6455530)

## 五項檢查怎麼看

- [ ] **① 啟動器日誌。** 用當時玩遊戲的 Windows 帳戶及實際安裝目錄。看 `Launcher\intl_service\debug.log` 是否涵蓋遊玩日期，找完整的 `k2f_chan registered`。沒有後續 `adopted` 也算；單獨的 `startup already in progress` 不夠。台港澳版和國際版若分開安裝，要分別查。
- [ ] **② 特徵檔案。** 在 `%LOCALAPPDATA%\Google\Chrome\User Data\PlatformExperienceHelper\` 下找「**64 位十六進位字元子目錄**內含 **platform_experience_helper.dll**」。父目錄、名稱相近的 Metrics 資料夾、正常的 EXE 都可能合法。木馬可自行建立目錄，沒裝 Chrome 一樣要查。
- [ ] **③ 目前使用者的登錄檔。** 查 `HKCU\Software\Classes\CLSID\{E77CC89B-7401-4C04-8CED-149DB35ADD04}\InprocServer32` 的預設值。若指向②的實際檔案，代表已設定已知常駐方式。64 位元 Windows 查兩個登錄檢視。保留指向系統 `wpc.dll` 的正常 HKLM 項目。
- [ ] **④ 雜湊比對（選做）。** 只對精確路徑的候選 DLL 計算 SHA-256，保留於本機。不同受害機的 DLL 可能含不同識別資料，雜湊不必相同。單一公開樣本對不上，仍要依檔案位置、登錄指向和執行紀錄判讀。
- [ ] **⑤ DNS 與防毒歷史。** 在實際開啟遊戲的時段，核對查詢來自哪台主機、命中哪條規則、回覆內容，以及 Defender 當時對哪個檔案採取何種動作。解析成功無法證明木馬已執行；目前快取空白或防毒無告警也不能排除。

**第一步沒命中或日誌不完整時，再補查啟動器自己的資料。** 國際版從 `%APPDATA%\nikke_launcher\tbs_cache` 找起，台港澳版通常用 `nikke_launcher_hmt`。在 `Code Cache\js\` 內找到本案外部腳本 URL 是接觸跡象；在 `Local Storage\leveldb\` 內找到真正的 `k2f.agent.state` 狀態紀錄，則是木馬運行過的強證據。遊戲目錄的 `Launcher\log\tinyget\` 可用來分辨「嘗試下載」和「成功下載」。**先保留證據再清快取**，自行移除遊戲後也值得查 AppData。[巴哈 11/4 修訂版](https://forum.gamer.com.tw/C.php?bsn=36390&snA=23304) · [逐項操作與限制](WINDOWS-CHECKS.md)

## 查完後，依最強的證據採取行動

| 結果 | 能確定什麼 | 現在做什麼 |
|---|---|---|
| 日誌缺失、重裝後才有、查錯帳戶或無權讀取 | 關鍵時段的狀態不明 | 保留其他來源並補查。如果當時用過高價值帳號，優先從乾淨裝置檢查登入活動。 |
| 日誌有涵蓋，且多項交叉檢查均無跡證 | 目前未見這支木馬的已知跡證 | 依官方後續公告復用。不因單純玩過 PC 版就要求重灌。 |
| 只有腳本快取、DNS 查詢或一次攔截 | 有接觸或嘗試連線的跡象；DNS 攔截只證明該次查詢被擋 | 暫停使用 PC 啟動器，核對有沒有下載成功、執行或常駐。 |
| `insight.bundle` 已從本案網址成功下載，尚未找到啟動證據 | 惡意程式已送達，執行狀態未明 | 按高風險處理，先隔離、保留證據，補查同時段日誌及 LevelDB。涉及工作憑證時直接通知資安窗口。 |
| `k2f_chan registered` 或實際木馬狀態紀錄 | 遠控程式已在這台電腦運行 | 立即斷網，依下一節保護主機及帳號。DLL 缺席不影響這個判定。 |
| 精確 DLL 與 HKCU 指向吻合 | 已設定已知常駐機制；單憑設定仍不能斷言哪個程序載入了它 | 同上，並保留登錄檢視、檔案位置與雜湊。只有 DLL 或殘留指向時也應當作強烈異常查證。 |

**沒有 Chrome 的情況。** 攻擊從 NIKKE 內建瀏覽器進入，第一階段已有讀檔、截圖和執行指令的能力。樣本能讀取 Edge、Firefox 等瀏覽器資料。有 Windows 玩家僅安裝 Edge 與 Firefox，啟動器留下兩次木馬運行紀錄，卻無特徵 DLL 或對應登錄項目。上表的分級不取決於你預設用哪個瀏覽器。[樣本逆向分析](https://github.com/WormW/nikke_malware_analysis/blob/main/docs/attack-path.md) · [無 Chrome 的實機回報](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5?permalink_comment_id=6455805#gistcomment-6455805)

## 受害後的處置順序

1. **隔離。** 手動拔網路線或關 Wi-Fi，停止使用該機登入服務。若電腦歸公司管理或存有公司憑證，先聯絡 IT／資安，讓負責人決定如何保存證據與重啟。已離線的機器可以照著手動清單檢查；不要為了讓雲端助手連上而讓受害機恢復連網。
2. **在另一台可信裝置保護帳號。** 先保護主要信箱與帳號復原方式，其次是 NIKKE／Level Infinite、瀏覽器儲存過的密碼及重要服務。使用全新且不重複的密碼，逐一登出所有裝置／撤銷工作階段，檢查近期登入、陌生裝置、信箱轉寄規則及 MFA 設定。被複製的登入 Cookie 可能繼續有效。
3. **依這台電腦原本放過什麼來輪替。** 開發者撤銷舊 Git 權杖、API 金鑰與 SSH 公鑰授權，再核發新憑證並檢查使用紀錄。如果錢包助記詞或私鑰可能曝光，應由本人在可信裝置產生新錢包並處理資產。任何人或助手都不需要取得你的密碼、助記詞或新金鑰。
4. **保全與復原。** 記下命中來源、當地時間與時區、檔案雜湊及相關 HKCU 檢視。原始日誌和 LevelDB 可能含主機名、UUID 和敏感狀態，請留在本機或私有離線備份，分享時只給去識別摘要。木馬能執行任意指令，**確認執行後，要重新信任主機，較穩妥的方案是用可信安裝媒體全新安裝 Windows**。先備份必要個人文件，確認可讀，避免還原舊執行檔、整個瀏覽器設定檔或啟動器快取。[Microsoft 復原選項](https://support.microsoft.com/zh-tw/windows/experience/backup-recovery/recovery-options-in-windows)
5. **若先選擇手動移除已知常駐。** 記錄並核實 HKCU 預設值和 DLL 互相吻合 → 移除該使用者檢視的惡意登錄指向 → 保持離線重新開機 → 只移除含特徵 DLL 的精確 64 碼子目錄 → 重新查兩個登錄檢視和檔案。保留 Chrome 正常父目錄及 Windows 的 HKLM 項目。此流程只處理已知機制。Defender 完整／離線掃描可輔助複查，但單次掃描不能驗證歷史外傳或排除其他程式被植入。
6. **重新啟用前。** 查適用區服的官方修復公告，保留證據後再清除該版本的啟動器快取。追蹤帳號活動、金鑰使用與未授權交易。重灌無法使已被複製的密碼、Cookie 或私鑰失效。[事件處置依據](https://github.com/MotherSphere/NIKKE-Launcher-Check/blob/main/docs/what-to-do.md)

## AdGuard Home 是否曾擋住攻擊？

開啟 AdGuard Home 的 **Query Log／查詢紀錄**，選當時遊戲電腦的用戶端 IP 或名稱，將時間對準啟動器紀錄，搜尋 `insightjs`。若路由器代全家轉送 DNS，先確認日誌能否區分是哪台機器。只需本機比對，請勿造訪可疑網域或為了驗證再開遊戲。

| 歷史查詢結果 | 判讀 |
|---|---|
| `insightjs[.]com` 明確顯示已封鎖並有命中規則；如曾查備援 `insightjs[.]org` 也被擋 | 支持當次外部腳本的解析遭阻擋。多個開啟時段都吻合，歸因更有把握。 |
| 主網域放行，`analytics.insightjs[.]com` 被擋 | 腳本可能已載入，但已知下載或後續通訊的網域在那次查詢被擋。 |
| 回傳當時的真實 IP | 該次 DNS 放行。若顯示 `NOERROR`，還要看實際回覆與封鎖原因，因為阻擋頁或 `0.0.0.0` 也可能使用這個回應碼。 |
| 沒紀錄、過期、只查了今天的規則 | 無法據此重建當天。其他 DNS 伺服器（包括路由器提供的 IPv6 DNS）可能讓查詢不經 AdGuard Home；本機已快取的 DNS 回覆也未必產生新查詢。 |

**DNS 紀錄有盲點。** MotherSphere 彙整的一台已出現 `k2f_chan registered` 的 PC，Pi-hole 沒有 `analytics.insightjs[.]com` 紀錄；玩家後來發現系統同時列有路由器的 IPv6 DNS。研究者提醒，Pi-hole、AdGuard Home 或 NextDNS 只看得到送到它們的查詢；若路由器再轉送，也可能記在路由器名下。核對當時的 DNS 路徑及用戶端歸屬，**不要用網域未命中來否定本機的執行證據**。目前設定只能輔助，不能還原整段歷史。[研究者對 DNS 盲點的說明](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5?permalink_comment_id=6455933#gistcomment-6455933)

訂閱多份惡意網域及釣魚清單確實可能擋下其中一段。已有人回報 11/1 使用 AdGuard Home 時，根網域成功解析到當時攻擊伺服器，啟動器有腳本快取卻未見木馬執行指標。要確認**是 DNS 保住哪一次啟動**，看那台機器當時的命中規則和回覆；要判斷**木馬有沒有跑過**，看主機跡證。[AdGuard Home 設定與記錄保留](https://github.com/AdguardTeam/AdGuardHome/wiki/Configuration) · [玩家的 AdGuard Home 紀錄](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5?permalink_comment_id=6455812#gistcomment-6455812)
