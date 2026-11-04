# Windows 手動檢查 NIKKE 啟動器痕跡

適用於 Windows PowerShell 5.1 或 PowerShell 7。請用當時玩遊戲的 Windows 帳號操作。若已看到 `k2f_chan registered` 或特定惡意 DLL，先斷開網路，從系統匣完整結束啟動器，直接依 [README 的處置說明](README.md) 處理。後面的檢查可以保留證據，不必等全部做完才處理帳號。

以下區塊可逐段貼進同一個 PowerShell 視窗。第 1～4 節都是本機唯讀查詢，無須下載工具或使用系統管理員權限。先保留日誌，避免為了檢查而重開、重裝遊戲或清快取。曾用過多個 Windows 帳號、裝過不同區服或搬過安裝目錄，就分別檢查。請勿公開上傳整份日誌或快取，裡面可能有主機名稱、識別碼及登入資料。

## 1. 啟動器是否執行過遠控程式

先在檔案總管找到 NIKKE 安裝目錄底下的 `Launcher` 資料夾，它裡面應有 `intl_service`。將 **Launcher 資料夾** 的完整路徑貼給下列指令。它只顯示日誌所涵蓋的首末時間、特徵出現次數與時間，略過主機名稱和識別碼。

```powershell
$launcher = (Read-Host '請輸入遊戲安裝目錄下 Launcher 資料夾的完整路徑').Trim().Trim('"')
$debugLog = Join-Path $launcher 'intl_service\debug.log'
if (Test-Path -LiteralPath $debugLog -PathType Leaf) {
    $firstStamp = $null
    $lastStamp = $null
    $hits = @(
        foreach ($line in [IO.File]::ReadLines($debugLog)) {
            $stamp = $null
            if ($line -match '^\[(?<stamp>\d{4}/\d{6}(?:\.\d+)?)[:\]]') {
                $stamp = $Matches.stamp
                if (-not $firstStamp) { $firstStamp = $stamp }
                $lastStamp = $stamp
            }
            if ($line.Contains('"k2f_chan registered"')) {
                if ($stamp) { $stamp } else { '時間格式無法辨識' }
            }
        }
    )
    [pscustomobject]@{
        FirstLogTime = $firstStamp
        LastLogTime  = $lastStamp
        Matches      = $hits.Count
        MatchTimes   = $hits -join ', '
    }
} else { '找不到 debug.log，這份安裝的歷史無法由第一步判讀。' }
```

`Matches` 大於 0 是遠控程式在啟動器中執行過的強證據。這個結果本身無法告訴你哪些資料傳出。`Matches = 0` 要搭配首末時間判讀。`1024/094320.599` 代表該電腦當地時間 10 月 24 日 09 時 43 分 20.599 秒，沒有年份或時區。首末時間不能證明中間每次啟動都保有紀錄。若欄位空白、出現讀取錯誤、曾刪除或重裝，請視為資料不足。單獨的 `startup already in progress` 或 `FETCH_LIST suc success` 也不足以認定感染。[執行指標與日誌保存方式](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5#check-if-you-are-infected)

## 2. 啟動器是否請求過惡意下載

沿用上一節的 `$launcher`。下面只查 `log\tinyget` 中檔名有日期的 `tinyget.*.log`，列出提到特定下載網址的日誌檔名和行數。它略過 `tinyget.mem.log`。後者會重複使用同一個緩衝區，尾端有時留有舊工作階段的內容。

```powershell
$tiny = Join-Path $launcher 'log\tinyget'
if (Test-Path -LiteralPath $tiny -PathType Container) {
    $logs = @(Get-ChildItem -LiteralPath $tiny -File -ErrorAction Stop |
        Where-Object { $_.Name -match '^tinyget\.\d{8}[^\\/]*\.log$' })
    "找到 $($logs.Count) 份有日期的 tinyget 日誌。"
    foreach ($log in $logs) {
        $rows = @(Select-String -LiteralPath $log.FullName -SimpleMatch -ErrorAction Stop `
            -Pattern 'analytics.insightjs.com/comm/assets/insight.bundle')
        if ($rows.Count) {
            [pscustomobject]@{ File = $log.Name; MatchingLines = $rows.Count }
        }
    }
} else { '找不到 tinyget 資料夾，無法判讀下載紀錄。' }
```

有命中代表該批日誌記錄了惡意網址。`AddTask` 等請求紀錄本身無法證明下載完成。若需釐清，離線用記事本查看命中處附近的同一筆工作是否完成，再比對第 1 節的時間。早期個案記錄的檔名是 `k2f.lua`，其他個案的惡意內容使用 `.log` 副檔名；不要執行或嘗試載入這些檔案。若只有一筆下載請求，先停止使用啟動器、保留資料並查第 3、4 節。若確認已下載成功，應先按疑似主機遭入侵處理。[檔名、下載紀錄與早期樣本](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5#stage-2-payload-download)

## 3. 分別查啟動器的腳本快取與遠控狀態

這一步和系統有沒有裝 Google Chrome 無關。先關閉啟動器，再檢查 `%APPDATA%` 下的 `nikke_launcher` 與 `nikke_launcher_hmt`。`Code Cache\js` 裡的腳本網址是**接觸線索**。`Local Storage\leveldb` 裡的 `k2f.agent.state` 是**遠控程式曾執行的強證據**。兩種結果的意義不同，請分開記錄。[研究者列出的兩種痕跡](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5#check-if-you-are-infected)

下列程式以單位元組 ASCII 相容編碼及 UTF-16LE 兩種形式搜尋相同的特徵，僅輸出檔案數量。每個目錄最多讀 32 個檔、每檔最多 8 MiB。腳本快取優先選研究者驗過的四個檔名，再依修改時間由新到舊選檔；檔名相符本身不計入命中。LevelDB 則只查 `.log`、`.ldb`、`.sst`。

```powershell
function Test-NikkeCacheMarker ([byte[]]$bytes, [string[]]$markers) {
    # 每個位元組對應一個字元，UTF-16LE 特徵即使從奇數位移開始也能比對。
    $oneByte = [Text.Encoding]::GetEncoding(28591)
    $data = $oneByte.GetString($bytes)
    foreach ($marker in $markers) {
        $wide = $oneByte.GetString([Text.Encoding]::Unicode.GetBytes($marker))
        if ($data.Contains($marker) -or $data.Contains($wide)) { return $true }
    }
    return $false
}

$knownCacheFiles = 'dd65fa4d61079e7e_0','ce2fd3f1007a9fe9_0','1416c638b3830103_0','fd520c7c0f65c2d7_0'
foreach ($profileName in 'nikke_launcher', 'nikke_launcher_hmt') {
    foreach ($area in 'Code Cache\js', 'Local Storage\leveldb') {
        $store = Join-Path $env:APPDATA "$profileName\tbs_cache\$area"
        try {
            $files = @(Get-ChildItem -LiteralPath $store -File -Force -ErrorAction Stop)
        } catch {
            "$profileName / $area：目錄不存在或無法讀取，未檢查。"
            continue
        }
        if ($area -eq 'Code Cache\js') {
            $markers = 'insightjs.com/l.js','insightjs.org/l.js'
        } else {
            $markers = @('k2f.agent.state')
            $files = @($files | Where-Object { $_.Extension -in '.log','.ldb','.sst' })
        }
        $chosen = @($files | Where-Object { $_.Length -le 8MB } |
            Sort-Object @{ Expression = { $_.Name -in $knownCacheFiles } }, LastWriteTimeUtc -Descending |
            Select-Object -First 32)
        $matched = 0
        $checked = 0
        foreach ($file in $chosen) {
            try {
                $reader = [IO.BinaryReader]::new([IO.File]::OpenRead($file.FullName))
                try {
                    $length = $reader.BaseStream.Length
                    if ($length -gt 8MB) { continue }
                    $bytes = $reader.ReadBytes([int]$length)
                    if ($bytes.Length -ne $length -or $reader.BaseStream.Length -ne $length) { continue }
                    if (Test-NikkeCacheMarker $bytes $markers) { $matched++ }
                    $checked++
                } finally { $reader.Dispose() }
            } catch { Write-Warning '有檔案無法讀取，已計入未檢查。' }
        }
        [pscustomobject]@{
            Profile = $profileName; Area = $area; FilesRead = $checked
            FilesSkipped = $files.Count - $checked; FilesWithMarker = $matched
        }
    }
}
```

請依 `Area` 判讀 `FilesWithMarker`。只有 `Code Cache\js` 命中，就補查下載與執行跡象。`Local Storage\leveldb` 命中，依高風險處理。`FilesRead = 0` 代表該組沒有讀到候選檔。`FilesSkipped` 包含超出上限、讀取中長度改變或讀取失敗的候選檔，數值大於 0 就有未檢查部分。即使全讀且未命中，也只排除現存檔案裡的這兩種明文編碼，無法排除已清除、汰換或壓縮的歷史紀錄。[本機狀態特徵及適用範圍](https://github.com/MotherSphere/NIKKE-Launcher-Check/blob/main/docs/checks.md)

## 4. Windows 是否留有 DLL 和登錄設定

在當時玩遊戲的 Windows 帳號下執行。先看特定路徑，不需安裝或打開 Chrome。程式只讀取「64 個十六進位字元的子資料夾」下名為 `platform_experience_helper.dll` 的檔案並計算 SHA-256。正常的父資料夾、`platform_experience_helper.exe` 或 `LoadFeatures` 都不是本步的命中項目。

```powershell
$helper = Join-Path $env:LOCALAPPDATA 'Google\Chrome\User Data\PlatformExperienceHelper'
$dlls = @(
    if (Test-Path -LiteralPath $helper -PathType Container) {
        foreach ($folder in (Get-ChildItem -LiteralPath $helper -Directory -Force -ErrorAction Stop)) {
            if ($folder.Name -notmatch '^[0-9a-f]{64}$') { continue }
            $dll = Join-Path $folder.FullName 'platform_experience_helper.dll'
            if (Test-Path -LiteralPath $dll -PathType Leaf) {
                Get-FileHash -LiteralPath $dll -Algorithm SHA256 -ErrorAction Stop |
                    Select-Object -ExpandProperty Hash
            }
        }
    }
)
"符合路徑及檔名的 DLL 數量：$($dlls.Count)"
$dlls
```

再查目前使用者 `HKCU` 的登錄機碼。`/ve` 讀取預設值，`/reg:64` 和 `/reg:32` 分別查兩種 Windows 登錄檢視。輸出可能包含 Windows 使用者路徑，請留作本機比對。不要修改或刪除 `HKLM` 中指向 Windows `wpc.dll` 的正常項目。

```powershell
$key = 'HKCU\Software\Classes\CLSID\{E77CC89B-7401-4C04-8CED-149DB35ADD04}\InprocServer32'
if ([Environment]::Is64BitOperatingSystem) { reg.exe query $key /ve /reg:64 }
reg.exe query $key /ve /reg:32
```

若預設值指向上述可疑 DLL，且檔案存在，已找到持久化的強證據。若只留檔案、預設值空白，或指向已不存在的檔案，可能是安裝或清除後的殘跡，無法據此判定現在仍在執行。若顯示機碼不存在，僅代表該檢視沒找到這項設定；「存取被拒」則是無法查核。惡意 DLL 的雜湊可能每台都不同，不能用單一公布雜湊排除感染。[DLL 路徑與已知清除殘跡](https://github.com/MotherSphere/NIKKE-Launcher-Check/blob/main/docs/incident.md#persistence-on-windows) · [Microsoft 登錄查詢說明](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/reg-query)

## 5. 選用 Microsoft Defender 掃描

先開「Windows 安全性 → 病毒與威脅防護 → 保護歷程記錄」，查看是否早有相關隔離或封鎖紀錄，記下時間與名稱。部分 Windows 版本譯為「保護歷程」或顯示 **Protection history**。檔案若已隔離，留在隔離區即可。沒有歷程紀錄也無法排除入侵。[Microsoft 保護歷程說明](https://support.microsoft.com/en-us/windows/protection-history-f1e5fd95-09b4-46d1-b8c7-1059a1e09708)

完成證據保全後，可在「Windows 安全性 → 病毒與威脅防護 → 掃描選項」執行完整掃描。若選擇 Microsoft Defender 離線掃描，請先儲存工作，電腦會重新啟動。防毒軟體可能隔離檔案，建議先記下上面查到的時間和雜湊。曾有使用者回報 Defender 離線掃描未偵測到與本事件相符的 DLL。若已找到執行或持久化證據，掃描結果不影響斷網、從乾淨裝置處理帳號，以及以可信安裝媒體重建 Windows 的處置優先順序。[Microsoft Defender 離線掃描](https://learn.microsoft.com/en-us/defender-endpoint/microsoft-defender-offline) · [事件研究者的處置建議](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5#what-to-do)

## 6. 選用額外防護：封鎖 tbs_browser.exe 的出站連線

本節是可逆的網路限制，**不是停用程式、感染檢測、木馬清除或安全復玩許可**。目前仍以暫停開啟 PC 啟動器為優先；已有執行或常駐跡證時，依 README 隔離與處理帳號，不要只加這條規則。研究者分析的已知 agent 通訊經過啟動器內嵌瀏覽器，但不能推論所有版本、其他惡意程式或所有網路路徑都受這條規則控制。[研究分析](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5#agent-analysis)

### 不開啟啟動器的設定步驟

1. 完整結束啟動器；先保全前述檢查資料，不要刪快取或 EXE。以檔案總管找到實際安裝位置的 `Launcher\intl_service\tbs_browser.exe`。不同區服、不同安裝副本需分別確認完整路徑。
2. 開啟「Windows 安全性 → 防火牆與網路保護 → 進階設定」，選「輸出規則 → 新增規則 → 程式」。
3. 選「這個程式路徑」，瀏覽選取上一步的實際 EXE；選「封鎖連線」。
4. 勾選網域、私人、公用三種設定檔，給規則一個可辨識的名稱，例如 `NIKKE - Block tbs_browser outbound`。這會修改本機防火牆設定，需具備管理權限；公司管理的設備先遵循 IT 政策。
5. 在規則內容確認：已啟用、方向為輸出、動作為封鎖、程式完整路徑正確、設定檔全選，且沒有額外縮限到特定遠端位址或連接埠。確認適用的 Windows 防火牆設定檔已啟用。若由其他防火牆產品或組織原則管理，不能假設這份本機規則已生效。

以上只是設定查核，**不要為測試而重新開啟可疑啟動器**。沒有進行本機封包或受控環境驗證時，不能宣稱已證實阻斷本事件。Windows 程式規則依完整路徑比對，不支援路徑萬用字元；搬移、重裝或更換執行路徑後須重新核對。明確封鎖規則優先於衝突的允許規則，但組織的本機原則合併設定仍可能影響規則是否納入有效政策。[Microsoft 防火牆規則](https://learn.microsoft.com/en-us/windows/security/operating-system-security/network-security/windows-firewall/rules)

### 效果、限制與撤回

- 被封鎖的程式仍可能執行或使用 CPU／RAM；不能把文章標題「永遠關閉」當成實際效果。啟動器的網頁、公告或其他功能也可能受影響。
- 事前正確且持續生效的封鎖，能降低已知內嵌瀏覽器通訊路徑的曝險；**「以前設過規則」不等於 100% 未感染**。仍需核對當時是否使用同一路徑、規則與防火牆是否啟用，以及其他執行／常駐跡證。不要為排除感染而刪除日誌。
- 規則不能撤回已外洩的資料，也不能清除已植入的 DLL 或限制其他程序。它不是恢復使用 PC 啟動器的充分條件。
- 需要撤回時，在「輸出規則」找到自己新增的精確名稱，先停用該條即可；不要重設整個防火牆或刪除不相關規則。停用會解除這項額外限制，應在釐清風險及有明確需要後才做。

本節依 Microsoft 規則機制與公開研究整理，尚未在各區服／版本進行實機效果驗證。玩家舊文是操作參考，不是本事件無感染的證明：[巴哈原文](https://forum.gamer.com.tw/C.php?bsn=36390&snA=16463)。
