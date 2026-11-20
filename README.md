# DeepSeek Harness 0.2.1-rc.2（Windows x64）官方下载与安装指南

---

## 0. 先读：两个最常见的坑（同事装不上的大概率原因）

1. **GitHub Releases 上没有安装包**。`github.com/deepseek-ai/deepseek-harness` 的所有
   release 的 assets 都是空的，官方安装包只在 DeepSeek 自有 CDN 分发（见下）。
2. **网上流传的 246 MB 单文件 `DeepSeek Harness.exe` 不是安装器**，是从安装目录里
   抠出来的主程序。它缺少 `icudtl.dat`、`resources\` 等配套文件，单独运行只会
   静默退出（报错 `icu_util.cc:237 Invalid file descriptor to ICU data received`），
   **任何方式都装不起来**。真正的安装器是 **289 MB**，文件名格式为
   `deepseek-harness-<版本>-win-x64.exe`。

---

## 1. 下载（官方 CDN 直链）

```
https://download.deepseek.com/dsh-desk/bin/win-x64/deepseek-harness-0.2.1-rc.2-win-x64.exe
```

命令行下载（PowerShell 或 cmd 均可）：

```powershell
curl.exe -L -o "$env:TEMP\dsh-0.2.1-rc.2-installer.exe" "https://download.deepseek.com/dsh-desk/bin/win-x64/deepseek-harness-0.2.1-rc.2-win-x64.exe"
```

- 约 **289 MB**；断点续传加 `-C -` 参数。
- URL 规律（换版本只改版本号）：
  `https://download.deepseek.com/dsh-desk/bin/win-x64/deepseek-harness-<版本>-win-x64.exe`
- 查最新版本 feed：`https://download.deepseek.com/dsh-desk/feeds/win-x64/nightly.yml`

## 2. 校验（运行前必做，一项不过就删除勿运行）

| 项目              | 期望值                                                       |
| ----------------- | ------------------------------------------------------------ |
| 大小              | `289,385,456` 字节                                           |
| MD5（= CDN ETag） | `f8fa3ed15a9519683e16ae8f0810e1c7`                           |
| SHA-256           | `72E4A8579F11B5428DD0A1506532777924ADBA7ECAACC710A7E429085796537F` |
| 数字签名          | **Valid**，签名者含 `Hangzhou DeepSeek Artificial Intelligence Co., Ltd.` |
| 版本信息          | ProductName `DeepSeek Harness`，FileVersion `0.2.1-rc.2`     |

一条 PowerShell 完成全部校验：

```powershell
$f = "$env:TEMP\dsh-0.2.1-rc.2-installer.exe"
"Size : {0:N0}" -f (Get-Item $f).Length
"MD5  : " + (Get-FileHash $f -Algorithm MD5).Hash.ToLower()
"SHA256: " + (Get-FileHash $f -Algorithm SHA256).Hash
$s = Get-AuthenticodeSignature $f
"Sig  : " + $s.Status + "  |  " + ($s.SignerCertificate.Subject.Split(',')[0])
"Ver  : " + (Get-Item $f).VersionInfo.FileVersion
```

⚠️ 网上存在大量假冒 DeepSeek 名义的安装包（参见
[McAfee 报告](https://www.mcafee.com/blogs/other-blogs/mcafee-labs/bogus-deepseek-ai-installers-are-infecting-devices-with-malware-research-finds)、
[Kaspersky 报告](https://www.kaspersky.com/blog/trojans-disguised-as-deepseek-grok-clients/53116)）。
**签名无效 / 签名者不是杭州深度求索 / 哈希不符 → 一律删除，不要尝试运行。**

## 3. 安装

**静默安装（推荐，ZCode 可直接执行）：**

```powershell
Start-Process -FilePath "$env:TEMP\dsh-0.2.1-rc.2-installer.exe" -ArgumentList '/S','/D=D:\DeepSeekHarness' -Wait -PassThru
```

规则（NSIS 约定，违反会装失败或装错位置）：

- `/S` 静默；`/D=` 指定目录；**`/D=` 必须是最后一个参数，不加引号，路径不能含空格**
- 安装过程可能弹 UAC，需要用户点"是"
- 装完目录里应出现 `DeepSeek Harness.exe`、`icudtl.dat`、`resources\`、`locales\` 等

**图形安装**：直接双击安装器，按向导走（默认装 C 盘，自定义可改）。

## 4. 安装后验证

```powershell
$exe = 'D:\DeepSeekHarness\DeepSeek Harness.exe'
# 主程序 SHA-256 必须等于：
'9C789407850773CE7D914A3EC0476852BF1CEEA69789D8CCF4E39BE953F4BF52'
(Get-FileHash $exe -Algorithm SHA256).Hash
# 启动验证（12 秒后应输出 RUNNING + 窗口标题）
$p = Start-Process $exe -WorkingDirectory 'D:\DeepSeekHarness' -PassThru
Start-Sleep 12; $p.Refresh()
if ($p.HasExited) { "EXITED code " + $p.ExitCode } else { "RUNNING [" + $p.MainWindowTitle + "]" }
```

## 5. 保持 0.2.1-rc.2：拒绝自动更新

官方 feed 已对全量用户推送 `0.2.2-rc.5`（stagingPercentage: 100）。
应用内弹出更新提示时**一律拒绝/跳过**，否则会离开 0.2.1-rc.2。

## 6. 故障排查

| 症状                                           | 原因与处理                                                |
| ---------------------------------------------- | --------------------------------------------------------- |
| 双击 exe 无反应，日志有 `icu_util.cc` ICU 报错 | 你在跑 246 MB 的裸主程序，不是安装器 → 回到第 1 节        |
| CDN 404                                        | 核对 URL 拼写；官方可能在未来下架旧版 → 用第 7 节离线分发 |
| 下载中断                                       | `curl.exe -C - -L -o <同一路径> <URL>` 续传               |
| 签名校验 Invalid                               | 文件被篡改或不完整，删除重下；仍 Invalid → 假包，勿运行   |
| 装完启动闪退                                   | 检查杀软是否拦截了 `resources\` 下文件；加白名单后重装    |

## 7. 离线分发（推荐做法）

旧版本随时可能从 CDN 下架，**最稳的做法是留存安装器本体**：

1. 拷贝 `deepseek-harness-0.2.1-rc.2-win-x64.exe`（289 MB）给同事（内网共享/U 盘）
2. 连同本文档一起给（文档里有全部校验值）
3. 对方从第 2 节"校验"开始执行即可，跳过下载

---

