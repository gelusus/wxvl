#  已复现 | Windows Shell 0day漏洞  
原创 陆安予
                    陆安予  白帽子安全笔记2.0   2026-10-19 17:31  
  
   
  
## 一、这是什么  
  
这是 2026 年10月微软补丁修复的 Windows Shell 中一个已被在野利用的 0Day 命令注入漏洞，允许攻击者绕过 Mark of the Web (MOTW) 安全机制执行任意命令。编号：CVE-2026-96951。  
  
![CVE-2026-96951](https://mmbiz.qpic.cn/mmbiz_png/ibXL9MCj2GqPp2Vqjo507HNymZ92EhCbGuZlEsLNHnFMEUOlc4YbASeDnPjwyQMk76XPPrMuia4Ieibo8RDb4icpFR7acibgCNsFzTmyy8EiasnIM/640?wx_fmt=png&from=appmsg "null")  
  
CVE-2026-96951  
## 二、技术原理  
  
从捕获的在野攻击样本中看，攻击者投递了一个特制的XML格式的文件来触发漏洞，其中包含了嵌入PDF数据以绕过检测，它在CVE-2018-8414基础上变形而来，但存在多处修改。  
## 三、复现情况  
  
在未打10月补丁的WIN11-25H2机器上，打开可以弹出计算器，对文件施加一个MOTW，打开依然绕过。这表明存在MOTW绕过，意味着从互联网下载打开不会存在任何警告:  
```
Set-Content -Path poc.SettingContent-ms -Stream Zone.Identifier -Value '[ZoneTransfer]','ZoneId=3','HostUrl=https://example.com' -Encoding ASCII
```  
  
![未打补丁](https://mmbiz.qpic.cn/mmbiz_gif/ibXL9MCj2GqP7bcfRMy3IqXLLFMP1qPkd5eFmlAic0UgHav8hl2RibwdtzKNRodj2p7v4TZq0xbTeRIr6Kv4WpeMryCZkMngibrUTP6kHia8TSlw/640?wx_fmt=gif&from=appmsg "null")  
  
未打补丁  
  
在已打最新补丁的系统上，打开则触发拦截，一旦点击允许，依然弹出计算器。十分危险。  
  
![已打补丁](https://mmbiz.qpic.cn/mmbiz_gif/ibXL9MCj2GqNWT6Diby5gfSx30hUx39f5XPjMdfw8QPCxib03PQkWlvicEbLbqhiaZVuU8V7J4xIoT4J5VTGnUYjRu9xF2HhhSkW7sGwdicInEjC4/640?wx_fmt=gif&from=appmsg "null")  
  
已打补丁  
  
这是 2026 年第三个被在野利用的 Windows Shell MOTW 绕过漏洞（前两个是 2 月的 CVE-2026-21510 和 4 月的 CVE-2026-32202）。  
  
或许由于传统的LNK，以及其它格式均被MOTW防线阻止，这表明攻击者今年明显在集中火力攻击 Windows 的 MOTW 机制。‌  
## 四、处置建议  
  
立即将windows更新至10月份版本，直到最新。并将此文件纳入信息安全意识培训，以防止钓鱼攻击。  
  
   
  
