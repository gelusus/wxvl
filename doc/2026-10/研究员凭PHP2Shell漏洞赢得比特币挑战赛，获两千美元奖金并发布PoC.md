#  研究员凭PHP2Shell漏洞赢得比特币挑战赛，获两千美元奖金并发布PoC  
e安在线
                    e安在线  e安在线   2026-10-27 01:47  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Hxdb7gjfn9mI4VMxLUNdBZzoc5nTXiallRPGUTgQ37vQpUzRSj72OtfqNsbZkCqXsgVFLobxDeoHelhY6FWZmlrgCLTiaGhoDKYjr5WploNuU/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Hxdb7gjfn9kM1AUiaC1o2F9PjUGrrTh6uXANNpL392gaYYMK7RpoGhfrspeI6LYjl0WXK3KfIqC6UbhOGvCMvsWRNdXSj1dAlJN7cgUQibdN0/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Hxdb7gjfn9knnYF3mrJzg8ibViaSW1N0WX2uXUiaer5Ud3ZD968PIxOar80ElgxKgMamAAtp8iahnuoK9GkRLrqdicFyibFT0ZIHCj9wzEbT9QoHM/640?wx_fmt=png&from=appmsg "")  
  
  
**安全研究员killa借助PHP2Shell(CVE-2026109618、CVE-2026-109619,CVSS9.2)漏洞拿下比特币挑战赛，耗时19.28小时，斩获价值约2021美元的BTC奖金，并在GitHub放出PoC。**  
  
  
**该PoC依靠两次内存泄漏获取地址，构造数据触发RCE，本地成功率约70%。早在官方10月21日推送补丁前4天，就已有攻击探测，攻击者借此投放Mirai变种，安全人员需尽快升级PHP并重启服务。**  
  
  
  
网名为killa的安全研究员利用两个PHP严重漏洞攻破测试服务器，赢下公开比特币挑战赛。整个攻击过程耗时19.28小时。2026年10月25日协调世界时15:47挑战赛决出胜负，当时奖金总额为0.02511679 BTC，价值2021.22美元。随后该研究员在GitHub上发布了PoC。  
  
  
本次挑战赛部署在stealmybitcoin.com站点上。站点运行未打补丁的PHP版本，RCE测试关卡后存放着钱包密钥，参赛者的目标就是从外部在服务器上执行命令。  
  
  
挑战赛进行期间，外界捐款将奖金从500美元推高至2000美元以上。挑战赛发起者表示，早期攻击尝试全部失败，因为服务器最初通过PHP命令行模式提供静态文件，无法触发漏洞。直到站点切换至真实PHP页面后，有效攻击才成为可能。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Hxdb7gjfn9mxsjhSu3iaIrVdfrZdyHicec8ESxgPEj78bZXpv5SGZ70zHAE75C4Td8T4wddeNGXXZM8T8u36hNMRz7b0B5PCadhrSdlM5icadc/640?wx_fmt=png&from=appmsg "")  
  
  
**PHP2Shell两漏洞获严重评级**  
  
  
这两个漏洞编号分别为CVE-2026-109618、CVE-2026-109619，统称为PHP2Shell漏洞组。两个漏洞的CVSS v4.0评分均为9.2，属于严重级。第一个漏洞会在内存溢出后的清理过程中破坏内存，第二个漏洞存在数据复用问题：共享工作进程处理前序请求后残留的数据，可被后续请求调用。  
  
  
PHP官方于2026年10月21日发布安全补丁，覆盖8.5.12、8.4.27、8.3.36、8.2.35四个版本。Google发布的安全指引显示，受影响版本范围为PHP 5.4至8.5.11。8.1及更早的旧版本分支已停止官方补丁支持，运行这些版本的站点必须升级到仍在维护的版本。  
  
  
**公开PoC利用内存泄漏实现RCE**  
  
  
这份公开的利用脚本借助两个内存泄漏问题完成利用。第一个内存泄漏可泄露堆地址，第二个可泄露PHP代码地址。拿到两个地址后，攻击者只需发送最终请求，在内存中植入伪造的数据结构。该结构会诱导PHP调用system函数，执行系统命令。脚本执行成功后，会输出随机测试标记、用户ID以及flag程序的哈希值。  
  
  
随PoC发布的技术说明也明确了该利用脚本的局限性。作者仅在本地实验环境中搭建和测试该脚本，并未针对线上站点调试，实验环境下的利用成功率约为70%。PoC作者坦言，本次攻击并未捕获完整的线上请求流量。最终恢复出的密钥与主办方公布的奖金地址不匹配，作者认为这是主办方的配置失误导致。  
  
  
**补丁发布前已现攻击尝试**  
  
  
目前公开利用代码已对外发布，相关攻击活动在补丁上线前就已出现。亚马逊方面观测到，最早的攻击尝试可追溯至10月17日，比官方补丁发布时间早4天。Sophos表示，已拦截多起利用该漏洞投递Potassium的攻击，该恶意程序疑似为Mirai僵尸网络变种。目前尚未发现利用该漏洞成功入侵的案例。  
  
  
升级到已打补丁的PHP版本，是唯一能彻底修复漏洞的方案。配置防火墙规则、调整相关设置或许能降低风险，但无法从根源上封堵漏洞。升级完成后，必须重启相关服务，确认当前运行的工作进程已加载新版本代码。这是因为捆绑运行时、容器环境可能仍在使用旧版本代码，导致补丁不生效。  
  
  
  
  
  
  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Hxdb7gjfn9kne0SWYOox32JeicVfibvUSDruVLyfRm7w6yU4tTktQ9pCFshLof9AUA8fTvylyYUrXOibA3lkciamhq2vTIlrUJia4BXW267JVIW4/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Hxdb7gjfn9lTFneLj3wsib8sLFWp1J3uiaMC5yvwnP18uw9IAosO5Lu6eBJVwauVJsNKPnQx7AmicV1eESXjiarvCcCrsa3RQ1TciblUVK78osaw/640?wx_fmt=png&from=appmsg "")  
  
  
声明：除发布的文章无法追溯到作者并获得授权外，我们均会注明作者和文章来源。如涉及版权问题请及时联系我们，我们会在第一时间删改，谢谢！文章来源：FreeBuf  
  
  
  
  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Hxdb7gjfn9lxTbGkW9tS6XIZakvNTXFxL7od3gCr6JhVJvPJfLwoPmbvDkibrpmjcINQicEUmO8iaDSAhZtdrLIxXn69z5ia2z1Y5gagzzheM1s/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Hxdb7gjfn9nqa5UfXb9BlTnNko7SKjjT8NcL9EMRHziaXCnEqoiaCCKF1BDkjhtDWZKqItKMYomUJRtqPoEhUTBIRU11Ks6TNIDqWcicqStAiaE/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Hxdb7gjfn9mxqJKboEDQ646XJ0rpg7NojrqZUsuEdvxP9YE3vaJElX5GlsI8RysgbiaVVSC5EnWLWKuL5kTEMavt0KRF8meEcjkribibMrjVy8/640?wx_fmt=png&from=appmsg "")  
  
  
  
