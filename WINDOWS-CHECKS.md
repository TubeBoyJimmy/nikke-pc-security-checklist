# Windows 檢查指令

這頁的檢查指令**只讀取、不修改任何東西**，不需要系統管理員權限，也不需要連網。請用**當時玩遊戲的 Windows 帳號**執行。

開始前，先在右下角系統匣對 NIKKE 啟動器按右鍵，選結束。

## 一鍵檢查

1. 在開始按鈕上按右鍵，選「終端機」或「Windows PowerShell」。
2. 複製下面整段，在視窗裡按右鍵貼上，再按 Enter。跳出「多行貼上」的警告就選仍要貼上；貼上後畫面停頓幾秒是正常的。
3. 等幾秒到一兩分鐘。如果沒自動找到 NIKKE，它會請你貼上安裝資料夾的路徑。

```powershell
& {
$ErrorActionPreference = 'SilentlyContinue'
$clsid = 'Software\Classes\CLSID\{E77CC89B-7401-4C04-8CED-149DB35ADD04}'
$red = New-Object 'System.Collections.Generic.List[string]'
$yel = New-Object 'System.Collections.Generic.List[string]'
$note = New-Object 'System.Collections.Generic.List[string]'
$latin1 = [Text.Encoding]::GetEncoding(28591)
$userDir = $env:USERPROFILE
function Show-Path([string]$p) { if ($userDir) { return $p.Replace($userDir, '~') }; return $p }
function Get-Text([IO.FileInfo]$f) {
    if ($f.Length -gt 64MB) { return $null }
    try {
        $fs = [IO.File]::Open($f.FullName, 'Open', 'Read', 'ReadWrite, Delete')
        try {
            $buf = New-Object byte[] ([int]$fs.Length)
            $n = 0
            while ($n -lt $buf.Length) { $r = $fs.Read($buf, $n, $buf.Length - $n); if ($r -le 0) { break }; $n += $r }
            return $latin1.GetString($buf, 0, $n)
        } finally { $fs.Dispose() }
    } catch { return $null }
}
function Find-Mark([string]$text, [string[]]$marks) {
    foreach ($m in $marks) {
        if ($text.Contains($m)) { return $m }
        if ($text.Contains($latin1.GetString([Text.Encoding]::Unicode.GetBytes($m)))) { return $m }
    }
    return $null
}

# 1. 找 NIKKE 安裝位置（解除安裝清單 + 各磁碟前三層名稱含 NIKKE 的資料夾）
$roots = New-Object 'System.Collections.Generic.List[string]'
foreach ($k in 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*', 'HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*', 'HKLM:\Software\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*') {
    foreach ($e in @(Get-ItemProperty -Path $k)) {
        if ("$($e.DisplayName)" -like '*NIKKE*' -and $e.InstallLocation) { $roots.Add("$($e.InstallLocation)".Trim().Trim('"')) }
    }
}
$skip = @('Windows', '$Recycle.Bin', 'System Volume Information')
foreach ($drv in [IO.DriveInfo]::GetDrives()) {
    if ($drv.DriveType -ne [IO.DriveType]::Fixed -or -not $drv.IsReady) { continue }
    $level = @($drv.RootDirectory.FullName)
    for ($depth = 1; $depth -le 3; $depth++) {
        $next = New-Object 'System.Collections.Generic.List[string]'
        foreach ($dir in $level) {
            foreach ($c in @(Get-ChildItem -LiteralPath $dir -Directory -Force)) {
                if (($c.Attributes -band [IO.FileAttributes]::ReparsePoint) -or ($depth -eq 1 -and $skip -contains $c.Name)) { continue }
                if ($c.Name -like '*NIKKE*') { $roots.Add($c.FullName) }
                if ($depth -lt 3) { $next.Add($c.FullName) }
            }
        }
        $level = $next
    }
}
$launchers = New-Object 'System.Collections.Generic.List[string]'
function Add-Launcher([string]$root) {
    if (-not $root -or -not (Test-Path -LiteralPath $root -PathType Container)) { return }
    foreach ($c in @(Get-Item -LiteralPath $root -Force) + @(Get-ChildItem -LiteralPath $root -Directory -Recurse -Depth 2 -Force -Filter 'Launcher')) {
        $hasLog = Test-Path -LiteralPath (Join-Path $c.FullName 'intl_service')
        $hasDl = Test-Path -LiteralPath (Join-Path $c.FullName 'log')
        if (($hasLog -or $hasDl) -and -not ($launchers -contains $c.FullName)) { $launchers.Add($c.FullName) }
    }
}
foreach ($r in $roots) { Add-Launcher $r }
$dataDirs = @(Get-ChildItem -LiteralPath $env:APPDATA -Directory -Force -Filter 'nikke_launcher*')
if ($launchers.Count -eq 0) {
    $manual = ''
    try { $manual = Read-Host '沒自動找到 NIKKE 安裝位置。有安裝的話，請貼上安裝資料夾路徑後按 Enter（沒有就直接按 Enter）' } catch {}
    Add-Launcher "$manual".Trim().Trim('"')
}

# 2. 啟動器日誌、下載紀錄
$missingLog = $false
foreach ($L in $launchers) {
    $dbg = Join-Path $L 'intl_service\debug.log'
    if (Test-Path -LiteralPath $dbg -PathType Leaf) {
        $first = $null; $last = $null; $hits = [ordered]@{}; $weak = 0
        try {
            $sr = New-Object IO.StreamReader([IO.File]::Open($dbg, 'Open', 'Read', 'ReadWrite, Delete'))
            try {
                while ($null -ne ($line = $sr.ReadLine())) {
                    $stamp = $null
                    if ($line -match '^\[(?:\d+:\d+:)?(\d{4}/\d{6})') { $stamp = $Matches[1]; if (-not $first) { $first = $stamp }; $last = $stamp }
                    $m = $null
                    foreach ($x in 'k2f_chan', 'lua downloaded to', 'agent started as', 'analytics.insightjs.com', 'insight.bundle') { if ($line.Contains($x)) { $m = $x; break } }
                    if ($m) { if (-not $hits.Contains($m)) { $hits[$m] = $stamp } } elseif ($line.Contains('insightjs')) { $weak++ }
                }
            } finally { $sr.Dispose() }
            foreach ($m in $hits.Keys) { $red.Add("debug.log 出現「$m」（第一次：$($hits[$m])）：$(Show-Path $dbg)") }
            if ($weak) { $yel.Add("debug.log 有 $weak 行提到 insightjs：$(Show-Path $dbg)") }
            if ($first) { $note.Add("debug.log 涵蓋期間 $first ~ $last（月日/時分秒，沒有年份）：$(Show-Path $dbg)") } else { $note.Add("debug.log 沒有可辨識的時間戳：$(Show-Path $dbg)") }
        } catch { $missingLog = $true; $note.Add("debug.log 無法讀取：$(Show-Path $dbg)") }
    } else { $missingLog = $true; $note.Add("找不到 debug.log：$(Show-Path $dbg)") }
    foreach ($f in @(Get-ChildItem -LiteralPath (Join-Path $L 'log') -File -Recurse -Depth 8 -Force)) {
        if ($f.Name -eq 'k2f.lua') { $red.Add("下載資料夾有木馬檔 k2f.lua：$(Show-Path $f.FullName)"); continue }
        $t = Get-Text $f
        if ($null -eq $t) { $note.Add("檔案太大或無法讀取，未檢查：$(Show-Path $f.FullName)"); continue }
        if ($t.Length -ge 3 -and [int]$t[0] -eq 0x1B -and [int]$t[1] -eq 0x4C -and [int]$t[2] -eq 0x4A) { $red.Add("下載資料夾有 LuaJIT 木馬本體：$(Show-Path $f.FullName)"); continue }
        $m = Find-Mark $t @('insight.bundle', 'analytics.insightjs.com')
        if (-not $m -and $f.Name -like 'tinyget*' -and $t.Contains('-3214f"')) { $m = 'ETag -3214f（木馬本體的下載回應）' }
        if ($m) { $red.Add("下載紀錄出現「$m」：$(Show-Path $f.FullName)") } elseif (Find-Mark $t @('insightjs')) { $yel.Add("下載紀錄提到 insightjs：$(Show-Path $f.FullName)") }
    }
}

# 3. 啟動器資料與快取（%APPDATA%\nikke_launcher*）
foreach ($d in $dataDirs) {
    foreach ($f in @(Get-ChildItem -LiteralPath $d.FullName -File -Recurse -Depth 12 -Force)) {
        $t = Get-Text $f
        if ($null -eq $t) { $note.Add("檔案太大或無法讀取，未檢查：$(Show-Path $f.FullName)"); continue }
        $p = Show-Path $f.FullName
        $m = Find-Mark $t @('k2f_chan', 'k2f.agent')
        if ($m) { $red.Add("啟動器資料有木馬紀錄「$m」：$p"); continue }
        if ($f.Name -eq 'TransportSecurity' -and $t.Contains('wxL1BeG57AG6frvPqS7txqpE3NyN0/fSUZHCbEn9NSw=')) { $red.Add("連線紀錄有木馬伺服器 analytics.insightjs.com：$p"); continue }
        if ($f.Name -eq 'Network Persistent State' -and $t.Contains('analytics.insightjs.com')) { $red.Add("連線紀錄有木馬伺服器 analytics.insightjs.com：$p"); continue }
        if ($f.Name -eq 'TransportSecurity' -and $t.Contains('JeFWywdZy+3jPC93e3P7ACdJ8liQHB7rH+9VHn7kswg=')) { $yel.Add("連線紀錄有 insightjs.com：$p"); continue }
        if (Find-Mark $t @('insightjs')) { $yel.Add("啟動器快取載入過 insightjs 腳本：$(Show-Path $f.DirectoryName)") }
    }
}

# 4. 瀏覽器常駐 DLL、暫存檔、登錄檔
$userData = @("$env:LOCALAPPDATA\Google\Chrome\User Data", "$env:LOCALAPPDATA\Microsoft\Edge\User Data", "$env:LOCALAPPDATA\BraveSoftware\Brave-Browser\User Data", "$env:LOCALAPPDATA\Vivaldi\User Data", "$env:LOCALAPPDATA\Chromium\User Data", "$env:APPDATA\Opera Software\Opera Stable", "$env:APPDATA\Opera Software\Opera GX Stable")
foreach ($u in $userData) {
    foreach ($i in @(Get-ChildItem -LiteralPath (Join-Path $u 'PlatformExperienceHelper') -Recurse -Depth 4 -Force)) {
        if ($i.PSIsContainer -and $i.Name -match '^[0-9a-f]{64}$') { $red.Add("常駐用的 64 碼資料夾（裡面沒有 DLL 也算）：$(Show-Path $i.FullName)") }
        elseif (-not $i.PSIsContainer -and $i.Name -eq 'platform_experience_helper.dll') { $red.Add("惡意 DLL：$(Show-Path $i.FullName)") }
    }
}
foreach ($b in @($env:LOCALAPPDATA, $env:ProgramW6432, $env:ProgramFiles, ${env:ProgramFiles(x86)}) | Where-Object { $_ } | Select-Object -Unique) {
    foreach ($n in 'platform_experience_helper.dll', 'platform_experience_helper.dll.old', 'old-clsid.txt') {
        $p = Join-Path $b "Google\Chrome\Application\PlatformExperienceHelper\$n"
        if (Test-Path -LiteralPath $p -PathType Leaf) { $red.Add("舊版常駐檔案：$(Show-Path $p)") }
    }
}
foreach ($f in @(Get-ChildItem -LiteralPath $env:TEMP -Force -Filter 'k2f*')) { $red.Add("暫存資料夾有木馬檔案：$(Show-Path $f.FullName)") }
foreach ($view in 'Registry64', 'Registry32') {
    $base = [Microsoft.Win32.RegistryKey]::OpenBaseKey('CurrentUser', $view)
    $key = $base.OpenSubKey($clsid)
    if ($key) {
        $v = $null
        $ip = $key.OpenSubKey('InprocServer32')
        if ($ip) { $v = $ip.GetValue(''); $ip.Close() }
        $key.Close()
        if ($v) { $red.Add("登錄檔常駐：HKCU\$clsid 指向 $(Show-Path "$v")") } else { $red.Add("登錄檔殘留：HKCU\$clsid 存在但沒有內容，代表曾被裝過") }
    }
    $base.Close()
}

# 5. 結果
Write-Host ''
if ($launchers.Count) { $launchers | ForEach-Object { Write-Host "已檢查：$(Show-Path $_)" } }
$note | Select-Object -Unique | ForEach-Object { Write-Host "  [i] $_" -ForegroundColor Gray }
$red | Select-Object -Unique | ForEach-Object { Write-Host "  [X] $_" -ForegroundColor Red }
$yel | Select-Object -Unique | ForEach-Object { Write-Host "  [!] $_" -ForegroundColor Yellow }
Write-Host ''
if ($red.Count) { Write-Host '【結果：中毒】木馬在這台電腦跑過。先斷網，用手機改密碼並登出所有裝置，再照 README「中毒了怎麼辦」處理。' -ForegroundColor Red }
elseif ($yel.Count) { Write-Host '【結果：有接觸】啟動器載入過攻擊腳本，但沒找到木馬執行的痕跡。帳號請照中毒處理：用手機改密碼並登出所有裝置。' -ForegroundColor Yellow }
elseif ($launchers.Count -eq 0 -and $dataDirs.Count -eq 0) { Write-Host '【結果：找不到 NIKKE】這個 Windows 帳號下沒有 NIKKE PC 版的安裝或資料。裝在特別位置的話，重跑一次並貼上路徑。' -ForegroundColor Cyan }
elseif ($missingLog -or $launchers.Count -eq 0) { Write-Host '【結果：資料不足】缺少 debug.log 或安裝位置，無法判斷。帳號請照「有接觸」處理。' -ForegroundColor Cyan }
else { Write-Host '【結果：沒找到已知痕跡】請確認上面 debug.log 的涵蓋期間有包含你玩的日期。9/27 後開過啟動器的話，仍建議用手機改 NIKKE 密碼。' -ForegroundColor Green }
Write-Host '提醒：找不到不等於 100% 安全。官方確認修好之前，先不要開 PC 版啟動器。'
}
```

### 結果怎麼看

- `[X]` 紅字：中毒的痕跡。
- `[!]` 黃字：有接觸的痕跡。
- `[i]` 灰字：補充資訊。**請看「debug.log 涵蓋期間」**，確認有包含你玩的日期。時間格式是「月日/時分秒」，例如 `1101/012037` 是 11/1 01:20:37。
- 最後一行是結論，對照 [README 的判讀表](README.md#結果怎麼判讀)。

如果跑出一堆紅色錯誤訊息（不是 `[X]` 開頭的），通常是複製時少了符號，請從頭重新複製一次。

結果要給別人看時，先把使用者名稱、電腦名稱遮掉。

### 這段指令查了哪些地方

| 位置 | 找什麼 |
|---|---|
| 安裝資料夾的 `Launcher\intl_service\debug.log` | `k2f_chan`、`lua downloaded to`、`agent started as`、`insight.bundle`、`analytics.insightjs.com`，以及紀錄涵蓋的期間 |
| 安裝資料夾的 `Launcher\log\`（包含 `tinyget.mem.log`） | `insight.bundle`、`analytics.insightjs.com`、木馬下載回應的 ETag（`-3214f`）、`k2f.lua`、LuaJIT 格式的檔案 |
| `%APPDATA%\nikke_launcher`、`%APPDATA%\nikke_launcher_hmt`（國際版、台港澳版） | `k2f_chan`、`k2f.agent`、`insightjs`，以及連線紀錄 `TransportSecurity`、`Network Persistent State` |
| Chrome、Edge、Brave、Vivaldi、Opera、Chromium 的 `PlatformExperienceHelper` | 64 碼的子資料夾、`platform_experience_helper.dll` |
| `Google\Chrome\Application\PlatformExperienceHelper`（舊版常駐位置） | `platform_experience_helper.dll`、`platform_experience_helper.dll.old`、`old-clsid.txt` |
| `%TEMP%` | `k2f` 開頭的檔案 |
| 登錄檔 `HKCU\Software\Classes\CLSID\{E77CC89B-7401-4C04-8CED-149DB35ADD04}`（64 位元與 32 位元） | 這個機碼存不存在、指向哪裡 |

- 找 NIKKE 的方式：Windows 的解除安裝清單，加上每顆硬碟前三層名稱含 `NIKKE` 的資料夾。
- 快取檔是二進位格式，指令會同時用 ASCII 和 UTF-16 兩種編碼搜尋。
- 單一檔案超過 64 MB 會略過，並在 `[i]` 列出來。

## 不想跑指令：手動看這幾個地方

1. **啟動器紀錄**：用記事本打開 `NIKKE 安裝資料夾\Launcher\intl_service\debug.log`，按 Ctrl+F 搜尋 `k2f_chan`。有找到就是中了。
2. **常駐資料夾**：在檔案總管的網址列貼上 `%LOCALAPPDATA%\Google\Chrome\User Data\PlatformExperienceHelper`，按 Enter。顯示「找不到」就沒事；裡面如果有 64 碼英數字的子資料夾，就是中了。
3. **暫存資料夾**：在網址列貼上 `%TEMP%`，按 Enter。有 `k2f` 開頭的檔案就是中了。
4. **登錄檔**：在 PowerShell 輸入下面兩行。兩行都顯示「找不到指定的登錄機碼或值」才是沒事；只要有查到東西（就算是空的），就是中了。

```powershell
reg query "HKCU\Software\Classes\CLSID\{E77CC89B-7401-4C04-8CED-149DB35ADD04}"
reg query "HKCU\Software\Classes\CLSID\{E77CC89B-7401-4C04-8CED-149DB35ADD04}" /reg:32
```

啟動器快取（`tbs_cache`）是二進位檔，用記事本搜不準，這部分請用一鍵檢查。

## Defender 保護歷程（選用）

打開「Windows 安全性 → 病毒與威脅防護 → 保護歷程記錄」，看有沒有相關的隔離或封鎖紀錄，有的話記下時間和名稱。沒有紀錄不代表沒事，Defender 曾經漏掉這顆 DLL。

## 封鎖攻擊用的網域和 IP

這一節會**修改設定**，是預防措施，不是檢查。

**AdGuard Home（全家一起擋）**：在「過濾器 → 自訂過濾規則」加上下面兩行，子網域也會一起擋掉。

```text
||insightjs.com^
||insightjs.org^
```

**單台電腦（hosts 檔）**：在開始按鈕上按右鍵，選「終端機（系統管理員）」或「Windows PowerShell（系統管理員）」，貼上：

```powershell
Add-Content -Path "$env:SystemRoot\System32\drivers\etc\hosts" -Value '', '0.0.0.0 insightjs.com', '0.0.0.0 analytics.insightjs.com', '0.0.0.0 insightjs.org'
ipconfig /flushdns
```

**封鎖 IP**（hosts 檔擋不了 IP）：在路由器設定裡封鎖 `89.127.233.127`、`89.126.250.172`，或在同一個系統管理員視窗貼上：

```powershell
New-NetFirewallRule -DisplayName 'NIKKE incident - block attacker IPs' -Direction Outbound -RemoteAddress '89.127.233.127', '89.126.250.172' -Action Block
```

要撤回這條規則：

```powershell
Remove-NetFirewallRule -DisplayName 'NIKKE incident - block attacker IPs'
```

## 選用：擋住 tbs_browser.exe 的對外連線

研究者發現，木馬所有的對外通訊都經過啟動器內建的瀏覽器 `tbs_browser.exe`。有玩家回報擋住它之後，被改過的檔案就載不進來；原作者也建議在官方確認修好之前保留這條規則。

先知道它的限制：

- 這只是降低風險，**不能清除已經中的木馬**，也擋不到 Chrome 裡常駐 DLL 的連線。
- 啟動器的網頁畫面（公告等）可能會打不開。
- 以前就設過這條規則，不等於一定沒中，還是要跑一鍵檢查。
- 規則是用完整路徑比對，重裝或搬移遊戲後要重新設定。

設定方式（需要系統管理員權限，公司電腦請先問 IT）：

1. 用檔案總管在 NIKKE 安裝資料夾裡搜尋 `tbs_browser.exe`，對它按右鍵，選「複製路徑」。
2. 以系統管理員身分開 PowerShell，貼上下面這行，把雙引號那一段（含雙引號）換成剛剛複製的路徑：

```powershell
New-NetFirewallRule -DisplayName 'NIKKE - Block tbs_browser outbound' -Direction Outbound -Action Block -Program "D:\NIKKE\Launcher\intl_service\tbs_browser.exe"
```

要撤回這條規則：

```powershell
Remove-NetFirewallRule -DisplayName 'NIKKE - Block tbs_browser outbound'
```

## 重新開啟動器前，先刪快取

等官方確認修好，而且一鍵檢查的結果已經截圖存好，再刪快取。刪掉之後，就查不到快取裡的痕跡了。

先從系統匣結束啟動器，再貼上：

```powershell
Remove-Item "$env:APPDATA\nikke_launcher*\tbs_cache" -Recurse -Force
```

沒有出現紅字就是刪好了。這只會刪啟動器的網頁快取，不會刪到遊戲和帳號資料。

---

檢查項目依據：[MotherSphere 檢查工具的項目說明](https://github.com/MotherSphere/NIKKE-Launcher-Check/blob/main/docs/checks.md)、[MotherSphere 事件調查](https://gist.github.com/MotherSphere/d6c049a09ba8ac3085c51ee9df24a4e5)、[巴哈姆特整理文](https://forum.gamer.com.tw/C.php?bsn=36390&snA=23304)。
