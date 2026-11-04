#  原创漏洞 | 微步XGPT发现 Apple macOS 多个漏洞  
微步情报局
                    微步情报局  微步在线研究响应中心   2026-11-04 09:51  
  
漏洞概况  
  
  
近日，Apple 官方发布 macOS Golden Gate 27、macOS Sequoia 15.8 和 macOS Tahoe 26.7 安全更新，修  
复了 SMB、HFS  
 等多个组件中的安全漏洞。其中CVE-2026-84537、  
 CVE-2026-84536、  
CVE-2026-84581   
由微步XG  
PT发现并报送A  
pple官方。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/T4OSm0sXdEOTdElEno80SbX5gWSPZSYMkmhfCds8SbRDWKibMC8stzjPf4cEB77eX4BkRRCficmpvEFGpX4yhWf3XsqWbIDOcGtmUn1WKOsus/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/T4OSm0sXdEO15DlYN2sR8l8x2xwGVyuE2O6Rib0QsarJXkkK0j17AtbcnGgl3yxyqIAaynTwp9vJIicIoMy54tlnaFQZJyGSyupuhZwpAEhicQ/640?wx_fmt=png&from=appmsg "")  
  
CVE-2026-84537 是 macOS SMB 客户端文件读写处理逻辑中的内核内存破坏漏洞。当实际读写长度小于请求长度时，内核释放内存的大小与分配时不一致。本机应用对已建立的 SMB 连接发起读写，或本机作为客户端连接到返回过短数据的恶意 SMB 服务器时，可造成系统异常终止或内核内存破坏。  
  
CVE-2026-84536 是 macOS SMB 客户端符号链接处理逻辑中的整数下溢漏洞。攻击者诱导用户连接恶意 SMB 服务器后，可导致系统异常终止。  
  
CVE-2026-84581 是 macOS HFS 内核扩展中的堆缓冲区溢出漏洞。已登录的普通用户挂载恶意构造的 HFS+ 磁盘映像并读取其中文件后，可造成系统异常终止或内核内存损坏；满足特定利用条件时，可在内核中执行代码。  
  
漏洞影响范围  
  
<table><tbody><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 12px;text-align: left;vertical-align: top;font-weight: bold;background-color: rgb(248, 249, 250);"><section><span leaf="">受影响版本</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">macOS Sequoia &lt; 15.8、macOS Tahoe &lt; 26.7、macOS Golden Gate &lt; 27</span></section></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 12px;text-align: left;vertical-align: top;font-weight: bold;background-color: rgb(248, 249, 250);"><section><span leaf="">有无修复补丁</span></section></td><td style="border:1px solid #ddd;padding:12px;text-align:left;vertical-align:top;"><section><span leaf="">有</span></section></td></tr></tbody></table>  
修复方案  
  
### 官方修复方案  
  
Apple 建议所有用户尽快通过设置 > 通用 > 软件更新进行升级。  
  
官方修复通告：  
  
https://support.apple.com/en-us/149035  
  
https://support.apple.com/en-us/149042  
  
https://support.apple.com/en-us/149043  
  
微步产品支撑  
  
  
微步漏洞情报于  
   
2  
026-09-15  
   
收录上述漏洞。  
  
微步下一  
代威胁情报平台NGTIP及X情报中心已于漏洞收录时向漏洞订阅用户推送该漏洞情报，  
并将持续推  
送后续更新；对于已经录入资产的用户，支持实时自动化排查受影响资产。  
  
  
