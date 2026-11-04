# 給 AI 助手的提示詞

適用於**能讀本機檔案、執行指令**的 AI 工具，例如 Claude Code、Claude 桌面版（Cowork）、OpenAI Codex、GitHub Copilot 的 Agent 模式。網頁版或手機 App 的聊天視窗看不到你的電腦，只能教你手動查。

使用前先看：

- **已經確定中毒的電腦（例如看到 `k2f_chan`），不要為了用 AI 而重新連網。** 請直接用[一鍵檢查](WINDOWS-CHECKS.md#一鍵檢查)，它不用連網。
- 檢查結果是中毒的話，你在這台電腦用過的 AI 工具也要**登出並換掉 API 金鑰**，木馬會偷這些東西。
- AI 也可能判斷錯。結果最好再用一鍵檢查對照一次。
- 不要把 AI 給的完整報告貼到網路上，裡面有你的使用者名稱和電腦名稱。
- 公司電腦，先問公司 IT。

提示詞一只做檢查，提示詞二才協助處理。兩段都可以單獨使用。

## 提示詞一：唯讀檢查

```text
請在這台 Windows 電腦上，用目前登入的使用者身分，唯讀檢查「2026 年 NIKKE PC 啟動器木馬事件」的痕跡。

規則：
- 只能讀取。不要修改、刪除、移動任何檔案或登錄檔，不要提權，不要下載或執行網路上的腳本。
- 不要開啟 NIKKE 啟動器、遊戲或任何可疑檔案（包括 k2f.lua 和 tinyget 資料夾裡的檔案）。
- 不要讀取瀏覽器的密碼、Cookie、錢包或私鑰內容。
- 篩選要在本機指令裡完成，只回報命中的項目、檔名和時間。不要把整份日誌、快取或登錄檔內容印到對話裡。
- 回報時把使用者名稱、電腦名稱、UUID、token 遮掉。
- 如果你沒辦法存取本機，請改教我手動怎麼查，並說明你沒有實際檢查。

步驟：
1. 先問我大概哪些日期開過啟動器。再找出 NIKKE 的安裝位置（解除安裝清單、捷徑，或問我）。國際版和台港澳版分開裝的話，兩份都要查。
2. 讀 <安裝資料夾>\Launcher\intl_service\debug.log：
   - 回報最早和最晚的時間戳（格式是 [MMDD/HHMMSS.mmm:...]，沒有年份），確認有涵蓋我玩的日期。
   - 搜尋 k2f_chan、lua downloaded to、agent started as、insight.bundle、analytics.insightjs.com。任何一個命中 = 中毒。
   - 只出現 insightjs = 有接觸。
3. 搜尋 <安裝資料夾>\Launcher\log\ 底下所有檔案（包含 tinyget.mem.log）：
   - 出現 insight.bundle、analytics.insightjs.com，或 ETag 以 -3214f" 結尾 = 中毒。
   - 有 k2f.lua，或檔案開頭是位元組 1B 4C 4A（LuaJIT）= 中毒。不要執行這些檔案。
4. 搜尋 %APPDATA%\nikke_launcher 和 %APPDATA%\nikke_launcher_hmt 底下所有檔案。這些多是二進位檔，請用位元組比對，同時搜尋 ASCII 和 UTF-16LE 兩種編碼。不要因為一般文字搜尋沒結果，就判定沒有。
   - 出現 k2f_chan 或 k2f.agent = 中毒。
   - 名為 TransportSecurity 的檔案含 wxL1BeG57AG6frvPqS7txqpE3NyN0/fSUZHCbEn9NSw=，或名為 Network Persistent State 的檔案含 analytics.insightjs.com = 中毒。
   - 其他檔案含 insightjs = 有接觸。
5. 檢查瀏覽器常駐（沒裝 Chrome 也要查）：
   - 下列資料夾底下的 PlatformExperienceHelper：%LOCALAPPDATA%\Google\Chrome\User Data、%LOCALAPPDATA%\Microsoft\Edge\User Data、%LOCALAPPDATA%\BraveSoftware\Brave-Browser\User Data、%LOCALAPPDATA%\Vivaldi\User Data、%LOCALAPPDATA%\Chromium\User Data、%APPDATA%\Opera Software\Opera Stable、%APPDATA%\Opera Software\Opera GX Stable。有 64 碼十六進位名稱的子資料夾（就算是空的），或有 platform_experience_helper.dll = 中毒。platform_experience_helper.exe 和 LoadFeatures 是正常檔案。
   - %LOCALAPPDATA%、%ProgramFiles%、%ProgramFiles(x86)% 底下的 Google\Chrome\Application\PlatformExperienceHelper，有 platform_experience_helper.dll、platform_experience_helper.dll.old 或 old-clsid.txt = 中毒。
6. 看 %TEMP% 有沒有 k2f 開頭的檔案（只看檔名，不要開啟）。有 = 中毒。
7. 讀登錄檔 HKCU\Software\Classes\CLSID\{E77CC89B-7401-4C04-8CED-149DB35ADD04}，64 位元和 32 位元檢視都要查（reg query 分別加 /reg:64、/reg:32）。機碼存在 = 中毒，就算預設值是空的也算。回報 InprocServer32 的預設值指向哪裡。HKLM 底下同一個 CLSID 指向 wpc.dll 是 Windows 的正常項目，不算。

報告格式：
- 第一行給結論：中毒／有接觸／資料不足（找不到 debug.log、重裝過，或紀錄沒涵蓋我玩的日期）／沒找到已知痕跡。
- 接著用表格列出：檢查項目｜查了哪裡｜結果與時間。
- 列出沒查到或查不了的項目。
- 做到報告為止，不要開始處理。
```

## 提示詞二：依結果協助處理

```text
請根據剛才（或我貼上的）NIKKE 唯讀檢查報告，協助我處理這台電腦。不要跟我要密碼、Token、助記詞或驗證碼。

規則：
- 每個會改變電腦的動作，先告訴我：根據哪一項證據、要動哪個確切的路徑或機碼、會有什麼影響、能不能復原。等我說「可以」才做，一次只做一個。
- 不要刪整個 PlatformExperienceHelper 資料夾，不要用萬用字元批次刪除，不要動 HKLM，不要動 Windows 的 wpc.dll。
- 不要下載或執行網路上的清除工具，不要上傳任何檔案。
- 如果我是遠端連線，斷網前先提醒我會中斷連線。不要自己斷網或重新開機。

處理順序：
1. 判斷範圍：
   - 中毒：提醒我拔網路線或關 Wi-Fi，並改用手機處理帳號（第 4 步）。
   - 有接觸／資料不足：電腦先不用清理，但帳號照中毒處理（第 4 步）。
   - 沒找到已知痕跡：提醒我用手機改 NIKKE 密碼並登出所有裝置，然後跳到第 5 步。
2. 動手前，建議我把檢查結果截圖，或存到本機（不要存到雲端同步資料夾）。
3. 中毒時的電腦處理：
   - 先建議最保險的方式：備份個人檔案後，用可信的安裝媒體重灌 Windows。
   - 如果我選擇先手動移除已知常駐：(a) 刪除 HKCU\Software\Classes\CLSID\{E77CC89B-7401-4C04-8CED-149DB35ADD04} 整個機碼（32 位元檢視也有的話，一起刪）→ (b) 請我保持離線並重新開機 → (c) 只刪除報告列出的 64 碼子資料夾和舊版常駐檔案 → (d) 重新跑一次唯讀檢查確認。
   - 提醒我：掃毒顯示乾淨或手動移除，都不能證明沒有其他後門。
4. 帳號由我自己在手機或另一台電腦操作，你只要列清單：主要信箱 → NIKKE／Level Infinite → 瀏覽器存過密碼的帳號。每個都要換新密碼、登出所有裝置、開兩步驟驗證、檢查登入紀錄和信箱轉寄規則。開發用的 SSH／Git／API 金鑰要撤銷重發；在這台電腦用過的 AI 工具要登出並換金鑰；有加密錢包的話，要在乾淨裝置建立新錢包並轉移資產。
5. 預防：協助我用 hosts 檔封鎖 insightjs.com、analytics.insightjs.com、insightjs.org，並用防火牆封鎖 IP 89.127.233.127、89.126.250.172。這需要系統管理員權限，一樣先取得我的同意。
6. 重新使用前：等官方確認修好，再刪除 %APPDATA%\nikke_launcher*\tbs_cache（先確認檢查結果已經保存）。

最後分成「已完成並確認」「需要我自己做」「無法確認」三類回報。
```

---

提示詞依據：[MotherSphere 事件調查](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5)、[MotherSphere 檢查項目](https://github.com/MotherSphere/NIKKE-Launcher-Check/blob/main/docs/checks.md)、[巴哈姆特整理文](https://forum.gamer.com.tw/C.php?bsn=36390&snA=23304)、[WormW 樣本分析](https://github.com/WormW/nikke_malware_analysis)。
