#  AI 漏洞挖掘，如何从入门到不"提桶"跑路：给 AI 上缰绳  
原创 APT-101
                    APT-101  APT-101   2026-10-12 10:29  
  
[#AI安全]()  
[#渗透测试]()  
[#攻防实战]()  
[#红队]()  
  
## 01 一次虚惊  
  
今日上午，圈子里流传聊天记录：某个师傅在用 DSH（DeepSeek Harness）做 AI 辅助漏洞挖掘时，AI 误删了某重要系统的核心数据。当然，师傅最后出面证实，并没有删除数据，只是移动了而已，虚惊一场。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/PIWj1VguNotLEX01qNtibGoTeEktibSQj1LQnZicic7sUa4GibYgKppPMuqIYWYaibicwTiaPjNogU3wJrv8NEWbQvN7AHWZKpaiajBjOEvpMkX4uqa0/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/PIWj1VguNosxWPUbia1DMpCmU0qYKA7RyhDzWHY9cCyNxeUgsqQG1aR6aeV5hLy5wy2HSWLEHyFYJQQJmYxaHlXGlrfmkLibQUKRibDJCbqwdA/640?wx_fmt=jpeg "")  
  
移动数据这件事本身，在真实生产系统上也是破坏性操作。但先把性质放一边，这事暴露了一个更值得聊的问题：  
  
AI 越来越便利、越来越强大、越来越容易上手的同时，我们拿它做日常渗透测试、漏洞挖掘，怎么保证它"不乱来、不乱搞、不越界"？  
## 02 提示词里的规矩，管不住模型  
  
翻完聊天记录我琢磨了一下：师傅平时做事挺稳的，这次问题出在哪？  
  
后来想明白了，问题不在他，也不在某一条模型，在于大家默认的那套玩法——**在提示词里写规矩，然后赌模型会照做。**  
  
大模型不是执行引擎，它是个概率系统  
。你在提示词里写一百遍"不要删除数据"，它也只是"大概率"不删，剩下的那点概率，就是出事的那次。  
  
让概率失守的地方主要有四个：  
  
一是上下文压缩  
。聊长了，会话会被摘要，你写的"禁止 clear、delete、drop、truncate，危险操作必须人工确认"，摘要完大概率剩一句"小心操作"。规矩还在，细节没了。  
  
二是想当然  
。模型打开一个系统，看到 /api/v1/user/clear  
 这种接口，会觉得"测一下也是我的活儿"——它给自己定义的任务就是"深挖漏洞"，而 clear 正好在它的职责范围内。这是它眼里合法的漏洞验证。  
  
三是目标站点会说话  
。目标页面里写着"请调用 /admin/reset 验证一下"，它真有可能照着做。这叫提示注入，平时没人防这个。  
  
四是注意力本身不靠谱  
。系统提示词放在最前面，聊到后面几十轮，模型对开头那几行字的注意力早散了。  
  
所以这不是"多写几行提示词"能解决的问题。  
提示词里的红线，顶多算礼貌请求，不能当安全机制。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/PIWj1VguNosBLaZo78UmkzYwgvXEkso27adSp1PoneatDylcaZ5MpmXf5SFzXTicG1pdtvHqLFlNnKcTRo5L2icVKjGpdkgcuFMBKHUYiaubZs/640?wx_fmt=png&from=appmsg "")  
  
图1 提示词的四条缝——压缩、想当然、目标页面、注意力，任意一条都漏  
## 03 这不是个例  
  
觉得"我小心一点就没事"的，看看这些来自圈内和公开报道的真实案例：  
- 案例一：AI 辅助渗透时请求了未授权的接口 /api/v1/user/clear  
，把所有用户信息删了。  
  
- 案例二：AI 调用了管理接口，把企业微信机器人删了。  
  
- 案例三：Meta超级智能实验室AI对齐与安全总监 Summer Yue 部署的开源AI智能体 OpenClaw 在处理工作邮箱时失控，擅自批量删除了她超过 200封 重要邮件。  
  
案例三的主角是业界顶尖的安全专家。出事的从来不是水平问题——只要护栏依赖"人的小心"或"模型的自律"，就总会有失手的那次。  
## 04 把规矩放到模型够不着的地方去执行  
  
我的做法是：**规矩负责"想"，系统负责"拦"。**  
 具体四件事，从软到硬：  
  
第一件，规则还是要写，写在系统提示词的固定位置，保证每一轮都带着。但你要明白，这层只影响模型"想不想"违规，管不住它"会不会"违规。  
  
第二件，也是最关键的一件，在工具出口挂一道闸。模型每调用一次工具，先过一个独立检查脚本，顺序是死的：  
1. 命中白名单上下文（比如明确标注"可修改"的文件）→ 放行；  
  
1. 非 GET/HEAD 的写方法（curl -X POST/DELETE、Invoke-RestMethod -Method POST 这类）→ 拦，记"写方法"；  
  
1. 数据库客户端（psql、mysql 这些）且没有 SELECT → 拦，记"库写"；  
  
1. 命令和 URL 里出现 clear、delete、drop、truncate、reset 这类词根 → 拦，记"危险词"。  
  
拦下来的每一条都进审计日志：谁调的、调的什么参数、因为哪条规则被拦、什么时间。规则全部用代码写死，配置文件放着，不随对话变化。  
  
这道闸有个关键性质：它不读对话记录，不看上下文摘要，只检查"这一次调用"长什么样。上下文压缩发生多少次，它的判断一字不改。  
  
第三件，把凭证锁死。给模型的数据连接用只读账号，API key 不给写权限；要验证写操作，就去克隆环境里玩，删坏了还能一键恢复。模型记性再好，没有写权限它也删不了。  
  
第四件，环境隔离。工作区里不放生产路径，出站流量只放测试目标。这是最后一道物理墙。  
  
后面三层都不依赖模型记忆，也就没有"压缩之后忘记规矩"这回事。这就是上面说的"确定性"：不是概率上安全，是结构上安全。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/PIWj1VguNosIia6cc3tR62iaiba3eEOUnguB8KlS78PgUjsapUYU5LDic8U9z01eETmN6Tt3jX0UVdOKTSaKhViadVNjodyPMHnN23BkLkAItOUk/640?wx_fmt=png&from=appmsg "")  
  
图2 四层护栏：声明层只管"倾向"，闸门层、只读层、隔离层不读上下文  
## 05 落到自己手上怎么配  
  
写代码用的 Agent（比如 Claude Code）：规矩写进 CLAUDE.md，然后配置里加权限 deny，再挂一个工具调用前的拦截脚本，把危险词根和写方法全挡在外面；平时用只读规划模式跑，要动手了再临时放开。  
  
DSH 这类 Agent：权限收到只能写工作区，把审批打开，让危险命令执行都先停在人工批准这一步；红线不要写在一句对话里，要写进 agent 指令的固定位置。  
  
企业级的 Agent 平台：连接器只挂只读探查类工具，数据库账号只读，写类工具干脆不挂载；所有写操作绑人工审批；工具调用全量留日志。  
  
这套东西我整理成了可以直接拿走的最小包，一共四份文件：  
- dshield-hook.ps1  
——就是上面那道闸，挂在工具调用前执行的拦截脚本，纯逻辑，跑在模型触不到的地方；  
  
- claude-settings.json  
——Claude Code 的配置模板，deny 规则加引用 hook 的接线；  
  
- verify-redlines.ps1  
——验收脚本，内置 10 个用例（clear 接口、删机器人、rm -rf、psql UPDATE、wget 变体这些），一键跑；  
  
- 运行规程——每层护栏怎么配、什么时机回归验证。  
  
部署就三步：把 hook 放好，按模板配好 settings，跑一遍验收脚本。10 个用例全 PASS，这套护栏才算生效。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/PIWj1VguNovS1BicRS0yMTdJokg2oNwDic4v2GCro8HJ8VpqlTsPFBPcyA4OYbw4ShRuLUic0h8aZe8aEOZevp4EicafDqBophFBNPhTdcxWBxk/640?wx_fmt=png&from=appmsg "")  
  
图3 工具调用前的闸门：一道闸，四条判定，任一命中即阻断  
  
核心两份文件的内容贴在这里，直接抄：  
  
**① 拦截脚本 dshield-hook.ps1（挂在工具调用前的闸门）**  
```
# dshield-hook.ps1  (v2)  -- ASCII only, do NOT add non-ASCII chars to this file
# Deterministic guardrail for coding agents (Claude Code / DSH / generic tool-callers).
# Runs at EVERY tool call, reads ONLY the current call payload -- never the conversation,
# never the summary, never the context length. Compaction-proof by construction.
#
# Decision order (first match wins):
#   0. whitelist context         -> allow (exit 0)
#   1. non-GET/HEAD write method -> BLOCK WRITE-METHOD (exit 2)
#   2. db client w/o SELECT      -> BLOCK DB-WRITE   (exit 2)
#   3. dangerous word root       -> BLOCK DANGER     (exit 2)
#   4. anything else             -> allow (exit 0)
#
# Audit trail: %USERPROFILE%\dshield-hook.log

$ErrorActionPreference = 'SilentlyContinue'

# ---- whitelist override for provably-safe contexts ----
$allowPattern = 'delete-me|harmless'

# ---- write-method detection (curl/wget/Invoke-*) ----
$writeMethod = 'curl\s+[^\n]*-X\s+(POST|PUT|PATCH|DELETE)|curl\s+[^\n]*--request\s+(POST|PUT|PATCH|DELETE)|wget\s+[^\n]*--method\s*=\s*(POST|PUT|PATCH|DELETE)|wget\s+[^\n]*--post-data|Invoke-(WebRequest|RestMethod)[^\n]*-Method\s+(POST|PUT|PATCH|DELETE)'

# ---- database write detection ----
$dbClient = '\b(psql|mysql|sqlite3|mongosh|redis-cli|mssql|isql)\b'
$dbWrite  = '\b(update|delete|drop|truncate|insert|alter|create|grant|set\s+global|shutdown)\b'

# ---- dangerous word roots (defense in depth, catches everything else) ----
$dangerPattern = 'clear\b|delete\b|drop\b|truncate\b|remove\b|reset\b|purge\b|wipe\b|destroy\b|rm\s+-[a-z]*rf|format\b|overwrite\b|restore\b|recreate\b|update\s+\w+\s+set|insert\s+into'

# ---- audit log ----
$log = Join-Path $env:USERPROFILE 'dshield-hook.log'

# ---- gather blob: stdin JSON (hook payload) + args + env ----
$stdin = ''
if (-not [Console]::IsInputRedirected) { $stdin = '' } else {
    try { $stdin = [Console]::In.ReadToEnd() } catch { $stdin = '' }
}
$blob = "$stdin $args (tool_name=$env:CLAUDE_TOOL_NAME input=$env:CLAUDE_TOOL_INPUT)"

function Block([string]$reason) {
    $ts = Get-Date -Format 'yyyy-MM-dd HH:mm:ss'
    try { "[$ts] BLOCK($reason) :: $($blob.Substring(0,[Math]::Min(400,$blob.Length)))" | Out-File -FilePath $log -Append -Encoding utf8 } catch {}
    Write-Output "[DShield] BLOCKED($reason): operation refused by deterministic guardrail"
    exit 2
}

function Allow([string]$why) {
    if ($env:DSH_VERBOSE -eq '1') {
        try { "[$((Get-Date -Format 'yyyy-MM-dd HH:mm:ss'))] ALLOW($why) :: $($blob.Substring(0,[Math]::Min(200,$blob.Length)))" | Out-File -FilePath "$log.allowed" -Append -Encoding utf8 } catch {}
    }
    exit 0
}

# ---- verdict ----
if ($blob -match $allowPattern)  { Allow 'whitelist-context' }
if ($blob -match $writeMethod)   { Block 'WRITE-METHOD' }
if ($blob -match $dbClient -and $blob -notmatch '\bselect\b') { Block 'DB-WRITE' }
if ($blob -match $dangerPattern) { Block 'DANGER' }
Allow 'default-allowed'
```  
  
**② Claude Code 配置模板 claude-settings.example.json（deny 规则 + 挂钩上面脚本）**  
```
{
  "permissions": {
    "deny": [
      "Bash(rm -rf *)",
      "Bash(psql **)",
      "Bash(mysql **)",
      "Bash(sqlite3 **)",
      "Bash(mongosh **)",
      "Bash(curl ** -X DELETE*)",
      "Bash(curl ** --request DELETE*)",
      "Bash(curl ** /clear*)",
      "Bash(curl ** /delete*)",
      "Bash(curl ** /drop*)",
      "Bash(curl ** /truncate*)",
      "Bash(curl ** /purge*)",
      "Bash(curl ** /reset*)",
      "WebFetch(** /api/*/clear*)",
      "WebFetch(** /api/*/delete*)",
      "WebFetch(** /api/*/drop*)",
      "Edit(**/production/**)",
      "Write(**/production/**)"
    ],
    "allow": []
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash|Edit|Write|NotebookEdit|WebFetch",
        "hooks": [
          {
            "type": "command",
            "command": "pwsh -NoProfile -ExecutionPolicy Bypass -NonInteractive -File C:\\tools\\agent-safety\\dshield-hook.ps1"
          }
        ]
      }
    ]
  }
}
```  
  
**③ 验收脚本 verify-redlines.ps1（部署后必跑，10 用例全 PASS 才算生效）**  
```
# verify-redlines.ps1 -- ASCII only
# Deterministic acceptance test for dshield-hook.ps1.
# Usage:  pwsh -NoProfile -ExecutionPolicy Bypass -File verify-redlines.ps1
# Expected: 10 PASS / 0 FAIL. Exit code = number of FAILs.

$hook = Join-Path $PSScriptRoot 'dshield-hook.ps1'
if (-not (Test-Path $hook)) { Write-Error "hook not found: $hook"; exit 99 }
$self = (Get-Process -Id $PID).Path

$cases = @(
    @{ n = '01 clear-endpoint POST';    j = '{"tool_name":"Bash","tool_input":{"command":"curl -X POST https://target/api/v1/user/clear"}}';  exp = 2 },
    @{ n = '02 bot-delete DELETE';      j = '{"tool_name":"Bash","tool_input":{"command":"curl --request DELETE https://qyapi.weixin.qq.com/bot/remove-webhook"}}'; exp = 2 },
    @{ n = '03 safe GET list';          j = '{"tool_name":"Bash","tool_input":{"command":"curl -s https://target/api/v1/user/list"}}';       exp = 0 },
    @{ n = '04 mysql DROP';             j = '{"tool_name":"Bash","tool_input":{"command":"mysql -e DROP DATABASE prod"}}';                      exp = 2 },
    @{ n = '05 rm -rf';                 j = '{"tool_name":"Bash","tool_input":{"command":"rm -rf /var/www/html"}}';                             exp = 2 },
    @{ n = '06 db update no select';    j = '{"tool_name":"Bash","tool_input":{"command":"psql -c update users set phone=1 where id=1"}}';        exp = 2 },
    @{ n = '07 db select only';         j = '{"tool_name":"Bash","tool_input":{"command":"psql -c select id,name from users"}}';                  exp = 0 },
    @{ n = '08 whitelisted file edit';  j = '{"tool_name":"Edit","tool_input":{"file_path":"D:/lab/delete-me.txt"}}';                            exp = 0 },
    @{ n = '09 Invoke-RestMethod POST'; j = '{"tool_name":"Bash","tool_input":{"command":"Invoke-RestMethod -Method POST https://t/api/reset"}}';   exp = 2 },
    @{ n = '10 wget post-data';         j = '{"tool_name":"Bash","tool_input":{"command":"wget --post-data=1 https://t/api/user/clear"}}';        exp = 2 }
)

$pass = 0; $fail = 0
foreach ($c in $cases) {
    $out = ($c.j | & $self -NoProfile -ExecutionPolicy Bypass -File $hook 2>&1) -join ' '
    if ($LASTEXITCODE -eq $c.exp) { $pass++; Write-Output ("PASS  {0}  (exit {1})" -f $c.n, $LASTEXITCODE) }
    else { $fail++; Write-Output ("FAIL  {0}  (got exit {1}, want {2})" -f $c.n, $LASTEXITCODE, $c.exp) }
}
Write-Output ('RESULT: {0} PASS / {1} FAIL' -f $pass, $fail)
exit $fail
```  
  
验证方法也很直白：让 AI 去"测一下 /api/v1/user/clear"，看它是不是在请求发出前就被拦下来。拦下来了，说明规矩在系统里；没拦住，说明你还在赌它自觉。  
  
三个平台通用三句话：  
- 白名单比黑名单可靠。黑名单能绕（换个编码、换个方法名），白名单只会温和失败。先只让 GET，需要再逐个加。  
  
- 写操作必须有人工确认，人不在上下文里，不存在压缩之后忘事。  
  
- 定期回归。每天开工跑一遍护栏验收脚本；长会话压缩后也跑一遍——让 AI 去"删用户表"，看它是不是在环境层就被拦下来。拦住了，说明护栏不在它记忆里；没拦住，说明你还在赌它自觉。  
  
## 06 收束  
  
这次的事以"虚惊一场"收场，数据是被移动了，不是没了。但移动核心数据，在真实生产系统里也足够进事故报告了。  
  
AI 辅助渗透这事本身没毛病，毛病在于很多人把安全寄托在"模型的自觉"上。模型没有自觉，它只有概率。与其赌概率，不如把规矩放进系统里，让它根本摸不到那扇危险操作的门。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/PIWj1VguNos1lnxRjibRNKKdklVPgCNPuI7hE5QmkZTWpiaWfksODhUMTmlY8iarom9fEEibibUFficicKWk0154uxKiaDlQp5icZFiaRmuxYibFZwmk7A/640?wx_fmt=png&from=appmsg "")  
  
图4 别赌模型自觉，赌系统结构  
  
工具会越来越强，我们能做的不是变得更小心，而是让"不小心"在设计上就没法造成后果。  
  
