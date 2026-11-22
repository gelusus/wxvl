#  nginx2root漏洞排查  
原创 彭瑞
                    彭瑞  新蜂网络安全实验室   2026-11-21 04:00  
  
# 一、漏洞概述  
## （一）简介  
  
NGINX是一款开源的高性能Web服务器、反向代理服务器和邮件代理服务器，也常被用作TCP/UDP代理、负载均衡器和API网关，是全球部署量最大的开源Web服务器与反向代理软件之一。  
  
F5于2026年11月公告修复了NGINX的5个内存安全漏洞（nginx2root），分别涉及HTTP/2、HTTP/3、DNS解析和邮件代理。其中CVE-2026-123007无需认证即可远程执行代码，获得worker进程权限；条件满足时，攻击者还可串联一个Linux内核SCTP漏洞，实现宿主机root提权或容器逃逸。  
  
## （二）受影响版本  
<table><tbody><tr style="height:18.4500pt;"><td data-colwidth="143" width="204" valign="center" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: rgb(0, 0, 0);"><p style="text-align: left;vertical-align: middle;margin-bottom: 0px;line-height: 1.75em;"><b><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-weight: bold;font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">CVE</span></font></span></b><b><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-weight: bold;font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">编号</span></font></span></b></p></td><td data-colwidth="158" width="243" valign="center" style="padding: 0pt 5.4pt;border-width: 1pt 1pt 1pt medium;border-style: solid solid solid none;border-color: rgb(0, 0, 0) rgb(0, 0, 0) rgb(0, 0, 0) currentcolor;"><p style="text-align: left;vertical-align: middle;margin-bottom: 0px;line-height: 1.75em;"><b><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-weight: bold;font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">组件</span></font></span></b></p></td><td data-colwidth="139" width="200" valign="center" style="padding: 0pt 5.4pt;border-width: 1pt 1pt 1pt medium;border-style: solid solid solid none;border-color: rgb(0, 0, 0) rgb(0, 0, 0) rgb(0, 0, 0) currentcolor;"><p style="text-align: left;vertical-align: middle;margin-bottom: 0px;line-height: 1.75em;"><b><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-weight: bold;font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">受影响版本</span></font></span></b></p></td><td data-colwidth="324" width="447" valign="center" style="padding: 0pt 5.4pt;border-width: 1pt 1pt 1pt medium;border-style: solid solid solid none;border-color: rgb(0, 0, 0) rgb(0, 0, 0) rgb(0, 0, 0) currentcolor;"><p style="text-align: left;vertical-align: middle;margin-bottom: 0px;line-height: 1.75em;"><b><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-weight: bold;font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">触发条件</span></font></span></b></p></td></tr><tr style="height:18.4500pt;"><td data-colwidth="143" width="204" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(0, 0, 0) rgb(0, 0, 0);"><p style="text-align: left;vertical-align: middle;margin-bottom: 0px;line-height: 1.75em;"><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">CVE-2026-123007</span></font></span></p></td><td data-colwidth="158" width="243" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt medium;border-style: none solid solid none;border-color: currentcolor rgb(0, 0, 0) rgb(0, 0, 0) currentcolor;"><p style="text-align: left;vertical-align: middle;margin-bottom: 0px;line-height: 1.75em;"><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">HTTP/2流控内存破坏</span></font></span></p></td><td data-colwidth="139" width="200" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt medium;border-style: none solid solid none;border-color: currentcolor rgb(0, 0, 0) rgb(0, 0, 0) currentcolor;"><p style="text-align: left;vertical-align: middle;margin-bottom: 0px;line-height: 1.75em;"><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">1.9.5 </span></font></span><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">-</span></font></span><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><span style="font-family: &#34;AR PL UMing CN&#34;;"><span leaf=""> </span></span><font face="AR PL UMing CN"><span leaf="">1.31.8</span></font></span></p></td><td data-colwidth="324" width="447" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt medium;border-style: none solid solid none;border-color: currentcolor rgb(0, 0, 0) rgb(0, 0, 0) currentcolor;"><p style="text-align: left;vertical-align: middle;margin-bottom: 0px;line-height: 1.75em;"><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">任一监听器启用了</span></font><font face="AR PL UMing CN"><span leaf="">HTTP/2</span></font></span></p></td></tr><tr style="height:18.4500pt;"><td data-colwidth="143" width="204" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(0, 0, 0) rgb(0, 0, 0);"><p style="text-align: left;vertical-align: middle;margin-bottom: 0px;line-height: 1.75em;"><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">CVE-2026-123633</span></font></span></p></td><td data-colwidth="158" width="243" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt medium;border-style: none solid solid none;border-color: currentcolor rgb(0, 0, 0) rgb(0, 0, 0) currentcolor;"><p style="text-align: left;vertical-align: middle;margin-bottom: 0px;line-height: 1.75em;"><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">HTTP/2释放后使用</span></font></span></p></td><td data-colwidth="139" width="200" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt medium;border-style: none solid solid none;border-color: currentcolor rgb(0, 0, 0) rgb(0, 0, 0) currentcolor;"><p style="text-align: left;vertical-align: middle;margin-bottom: 0px;line-height: 1.75em;"><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">1.19.7 </span></font></span><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">-</span></font></span><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><span style="font-family: &#34;AR PL UMing CN&#34;;"><span leaf=""> </span></span><font face="AR PL UMing CN"><span leaf="">1.31.8</span></font></span></p></td><td data-colwidth="324" width="447" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt medium;border-style: none solid solid none;border-color: currentcolor rgb(0, 0, 0) rgb(0, 0, 0) currentcolor;"><p style="text-align: left;vertical-align: middle;margin-bottom: 0px;line-height: 1.75em;"><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">启用</span></font><font face="AR PL UMing CN"><span leaf="">HTTP/2，且请求被转发到上游</span></font></span></p></td></tr><tr style="height:18.4500pt;"><td data-colwidth="143" width="204" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(0, 0, 0) rgb(0, 0, 0);"><p style="text-align: left;vertical-align: middle;margin-bottom: 0px;line-height: 1.75em;"><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">CVE-2026-121810</span></font></span></p></td><td data-colwidth="158" width="243" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt medium;border-style: none solid solid none;border-color: currentcolor rgb(0, 0, 0) rgb(0, 0, 0) currentcolor;"><p style="text-align: left;vertical-align: middle;margin-bottom: 0px;line-height: 1.75em;"><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">HTTP/3释放后使用</span></font></span></p></td><td data-colwidth="139" width="200" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt medium;border-style: none solid solid none;border-color: currentcolor rgb(0, 0, 0) rgb(0, 0, 0) currentcolor;"><p style="text-align: left;vertical-align: middle;margin-bottom: 0px;line-height: 1.75em;"><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">1.25.0 </span></font></span><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">-</span></font></span><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><span style="font-family: &#34;AR PL UMing CN&#34;;"><span leaf=""> </span></span><font face="AR PL UMing CN"><span leaf="">1.31.8</span></font></span></p></td><td data-colwidth="324" width="447" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt medium;border-style: none solid solid none;border-color: currentcolor rgb(0, 0, 0) rgb(0, 0, 0) currentcolor;"><p style="text-align: left;vertical-align: middle;margin-bottom: 0px;line-height: 1.75em;"><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">有</span></font><font face="AR PL UMing CN"><span leaf="">quic监听，且请求被转发到上游</span></font></span></p></td></tr><tr style="height:18.4500pt;"><td data-colwidth="143" width="204" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(0, 0, 0) rgb(0, 0, 0);"><p style="text-align: left;vertical-align: middle;margin-bottom: 0px;line-height: 1.75em;"><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">CVE-2026-123634</span></font></span></p></td><td data-colwidth="158" width="243" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt medium;border-style: none solid solid none;border-color: currentcolor rgb(0, 0, 0) rgb(0, 0, 0) currentcolor;"><p style="text-align: left;vertical-align: middle;margin-bottom: 0px;line-height: 1.75em;"><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">Resolver释放后使用</span></font></span></p></td><td data-colwidth="139" width="200" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt medium;border-style: none solid solid none;border-color: currentcolor rgb(0, 0, 0) rgb(0, 0, 0) currentcolor;"><p style="text-align: left;vertical-align: middle;margin-bottom: 0px;line-height: 1.75em;"><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">0.6.18 </span></font></span><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">-</span></font></span><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><span style="font-family: &#34;AR PL UMing CN&#34;;"><span leaf=""> </span></span><font face="AR PL UMing CN"><span leaf="">1.31.8</span></font></span></p></td><td data-colwidth="324" width="447" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt medium;border-style: none solid solid none;border-color: currentcolor rgb(0, 0, 0) rgb(0, 0, 0) currentcolor;"><p style="text-align: left;vertical-align: middle;margin-bottom: 0px;line-height: 1.75em;"><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">resolver + 由请求派生变量的 proxy_pass + mirror（SSI 等子请求也可能触发）</span></font></span></p></td></tr><tr style="height:18.4500pt;"><td data-colwidth="143" width="204" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor rgb(0, 0, 0) rgb(0, 0, 0);"><p style="text-align: left;vertical-align: middle;margin-bottom: 0px;line-height: 1.75em;"><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">CVE-2026-113425</span></font></span></p></td><td data-colwidth="158" width="243" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt medium;border-style: none solid solid none;border-color: currentcolor rgb(0, 0, 0) rgb(0, 0, 0) currentcolor;"><p style="text-align: left;vertical-align: middle;margin-bottom: 0px;line-height: 1.75em;"><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">邮件代理释放后使用</span></font></span></p></td><td data-colwidth="139" width="200" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt medium;border-style: none solid solid none;border-color: currentcolor rgb(0, 0, 0) rgb(0, 0, 0) currentcolor;"><p style="text-align: left;vertical-align: middle;margin-bottom: 0px;line-height: 1.75em;"><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">0.5.33 </span></font></span><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">-</span></font></span><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><span style="font-family: &#34;AR PL UMing CN&#34;;"><span leaf=""> </span></span><font face="AR PL UMing CN"><span leaf="">1.31.8</span></font></span></p></td><td data-colwidth="324" width="447" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt medium;border-style: none solid solid none;border-color: currentcolor rgb(0, 0, 0) rgb(0, 0, 0) currentcolor;"><p style="text-align: left;vertical-align: middle;margin-bottom: 0px;line-height: 1.75em;"><span style="font-family: &#34;AR PL UMing CN&#34;;color: rgb(0, 0, 0);font-style: normal;font-size: 10.5pt;"><font face="AR PL UMing CN"><span leaf="">mail块里同时有proxy on和auth_http</span></font></span></p></td></tr></tbody></table>##   
## （三）已修复版本  
  
稳定线  
1.30.x 需 ≥ 1.30.7，主线1.31.x需 ≥ 1.31.9。注意  
：  
1.31.0到1.31.8仍然受影响。同一批公告还修复了另外5个CVE（118899、125283、120967、123640、120993），升级到上述版本即一并覆盖。  
  
# 二、漏洞排查  
## （一）版本检查  
  
# 检查版本和编译选项  
  
nginx -v  
  
nginx -V 2>&1 | tr ' ' '\n' | grep -E 'http_v2|http_v3|mail|stream'  
  
  
# 检查是否已包含安全修复（changelog）  
  
zgrep -E 'CVE-2026-(123007|123633|123634|121810|113425)' /usr/share/doc/nginx*/changelog.Debian.gz  
  
rpm -q --changelog nginx | grep -E 'CVE-2026-(123007|123633|123634|121810|113425)'  
  
  
判断标准：  
  
稳定线1.30.x需 ≥ 1.30.7，主线1.31.x需 ≥ 1.31.9，或changelog确认包含相关CVE修复，则状态为安全。  
  
RPM变更日志可能不写CVE编号，查不到时以发行版公告为准。  
  
## （二）触发条件检查  
  
# 先导出完整配置（包含所有include的文件，并去掉注释），再逐项检查：  
  
sudo nginx -T 2>/dev/null | sed 's/#.*//' > /tmp/ng.conf  
  
  
# 第一项：检查HTTP/2（123007、123633）  
  
grep -nE 'listen[[:space:]].*[[:space:]]http2|http2[[:space:]]+on' /tmp/ng.conf  
  
有输出即启用  
  
  
# 第二项：检查是否转发到上游（123633、121810）  
  
grep -nE '(proxy|grpc|fastcgi|uwsgi|scgi)_pass[[:space:]]' /tmp/ng.conf  
  
  
# 第三项：检查HTTP/3（121810）：编译选项、quic监听、UDP监听  
  
nginx -V 2>&1 | grep -o 'http_v3_module'  
  
grep -nE 'listen[[:space:]].*[[:space:]]quic' /tmp/ng.conf  
  
ss -lunp | grep nginx  
  
  
# 第四项：检查Resolver（123634）：resolver、带变量的proxy_pass、mirror或ssi  
  
grep -nE '^[[:space:]]*resolver[[:space:]]' /tmp/ng.conf  
  
grep -nE 'proxy_pass[[:space:]]+[^;]*\$' /tmp/ng.conf  
  
grep -nE '^[[:space:]]*(mirror[[:space:]]+[/@]|ssi[[:space:]]+on)' /tmp/ng.conf  
  
  
# 第五项：邮件代理（113425）：mail块、proxy on、auth_http、对外端口  
  
grep -nE '^[[:space:]]*mail[[:space:]]*\{|auth_http[[:space:]]|^[[:space:]]*proxy[[:space:]]+on' /tmp/ng.conf  
  
ss -tlnp | grep nginx | grep -E ':(25|110|143|465|587|993|995)\b'  
  
  
判定规则：  
  
CVE-2026-123007：第1项有输出。明文监听（如listen 80 http2;）也要算在内，不只是HTTPS。  
  
CVE-2026-123633：第1项和第2项都有输出。  
  
CVE-2026-121810：第3项有quic监听（或nginx在监听UDP），且第2项有输出。  
  
CVE-2026-123634：第4项三类同时出现（resolver、带变量的proxy_pass、mirror或ssi）。只有前两类时按疑似处理。  
  
CVE-2026-113425：mail块里同时有proxy on和auth_http。  
  
## （三）运行态确认  
  
“nginx -T”显示的是磁盘上的配置，不一定是正在运行的master已加载的配置。如果前面有CDN或负载均衡终止HTTP/2，要直接测nginx自己的端口。容器和ingress-nginx需要进入容器或对镜像单独检查。  
  
ls -l /proc/$(pgrep -f 'nginx: master' | head -1)/exe   
  
带(deleted)：仍在运行旧二进制  
  
  
pgrep -af 'worker process is shutting down'   
  
有输出：旧worker还在服务  
  
  
openssl s_client -connect 127.0.0.1:443 -alpn h2 </dev/null 2>&1 | grep ALPN   
  
显示h2表示HTTP/2实际生效  
  
# 三、修复方案  
## （一）根治  
  
升级到1.30.7（稳定线）或1.31.9（主线）及以上，或安装发行版已修复的包。升级后必须restart，不能只reload。容器镜像需要重新构建并重新部署。  
##   
## （二）临时缓解  
### 1、临时禁用HTTP/2  
  
关闭受影响服务的HTTP/2功能，移除监听配置中的http2参数，并在支持的版本中设置http2 off;，阻断CVE-2026-123007和CVE-2026-123633的已知攻击路径。实施前应评估对gRPC等依赖HTTP/2业务的影响。  
###   
### 2、临时禁用HTTP/3 / QUIC  
  
针对CVE-2026-121810，移除QUIC监听配置，设置http3 off;，停止发布HTTP/3服务通告，必要时通过防火墙封禁对应UDP端口。  
###   
### 3、调整动态域名解析及子请求配置  
  
针对CVE-2026-123634，避免使用SSI、mirror等相关子请求配置，同时避免代理目标域名包含变量；可改用指向固定后端IP的上游配置，消除相关请求期间的动态域名解析。  
###   
### 4、限制邮件代理服务的访问范围  
  
针对CVE-2026-113425，通过防火墙、安全组或访问白名单，仅允许可信来源访问IMAP、POP3、SMTP等邮件代理端口；不需要对外提供服务时，应关闭相关监听或绑定回环地址。  
###   
### 5、网络层缓解  
  
限制能连到nginx HTTP/2端口的源地址，或让前端CDN和负载均衡终止HTTP/2，而nginx自身不再接受HTTP/2。  
  
若可能，仅对可信客户端开放HTTPS。  
  
针对内核SCTP链式风险：可考虑临时卸载/黑名单SCTP模块（modprobe -r sctp或install sctp /bin/false），但需评估业务是否使用SCTP。  
  
  
