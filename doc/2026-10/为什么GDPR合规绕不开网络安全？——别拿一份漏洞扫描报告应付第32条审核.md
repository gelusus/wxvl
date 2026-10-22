#  为什么GDPR合规绕不开网络安全？——别拿一份漏洞扫描报告应付第32条审核  
原创 GTG-Hardy
                    GTG-Hardy  GTG网络安全实验室   2026-10-22 01:00  
  
> 很多做跨境业务的企业有个困惑：GDPR明明是数据保护法规，为什么欧盟客户做供应商尽职调查，张口就要漏洞扫描、渗透测试这类网络安全报告？   
> 不少厂商的做法很简单：直接甩出一份最新的漏洞扫描PDF，以为就完成GDPR第32条的举证。  
> 但往往会被客户反问：这份安全报告，怎么对应我们业务里的个人数据风险？  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/U5ROicHxZUzplmGHB7LlmC7Ff7L7F46EqM9I5w3nUdGBxsuwtmVyiapCUaqFaKyrw2OSguauWsn0KxpLjHWK3HickyG1iaEVrBEXrOlyIxfUO1I/640?wx_fmt=png&from=appmsg "")  
  
这里核心误区：  
**漏洞扫描报告只是安全措施的佐证材料，不等于安全措施本身，更不等于自动满足GDPR第32条。**  
  
 GDPR不是只管隐私政策、用户授权、数据留存这些“纸面流程”，网络安全技术手段是法律硬性义务。  
  
本文拆解GDPR第32条的核心要求、各类安全检测的实际举证价值，以及交付给海外客户的材料该补齐哪些关键内容。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/U5ROicHxZUzqGTyUEiaQBRX4gWAL6g38s75hbcBPaDnicnaTWvYIFl8bRj3FK4lFNoMAicfF8QEDgZtrUUaHdu1s7M8YlvQ2TLA21ibAjhNfJWmc/640?wx_fmt=png&from=appmsg "")  
  
  
  
**0****1**  
  
  
**GDPR第32条：数据保护，本质就是风险导向的网络安全义务**  
  
  
GDPR第32(1)条明文要求数据控制者与处理者，必须实施适配风险的技术与组织安全措施，原文核心逻辑有两层：  
> **Taking into account the state of the art, the costs of implementation and the nature, scope, context and purposes of processing as well as the risk of varying likelihood and severity for the rights and freedoms of natural persons, the controller and the processor shall implement appropriate technical and organisational measures to ensure a level of security appropriate to the risk.**  
  
1. **风险导向是底层逻辑**  
  
安全措施的强度不是统一标准，要匹配你处理个人数据的类型、范围、业务场景，以及数据泄露之后对自然人权益造成的危害。 同样是未授权访问漏洞：处理人脸、家庭影像的设备，和仅传输设备序列号、无个人信息的网关，二者需要的安全防护等级完全不一样。  
  
1. **法律明确划定四类安全措施方向，技术安全占大头**  
  
<table><thead><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border-top: 1px solid #dfe2e5;margin: 0px;padding: 0px;"><th style="box-sizing: border-box;border-width: 1px 1px 0px;border-bottom-style: initial;border-bottom-color: initial;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: #dfe2e5;border-right-color: #dfe2e5;border-left-color: #dfe2e5;border-image: initial;margin: 0px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">分项</span></span></th><th style="box-sizing: border-box;border-width: 1px 1px 0px;border-bottom-style: initial;border-bottom-color: initial;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: #dfe2e5;border-right-color: #dfe2e5;border-left-color: #dfe2e5;border-image: initial;margin: 0px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">原文关键词</span></span></th><th style="box-sizing: border-box;border-width: 1px 1px 0px;border-bottom-style: initial;border-bottom-color: initial;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: #dfe2e5;border-right-color: #dfe2e5;border-left-color: #dfe2e5;border-image: initial;margin: 0px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">中文要点</span></span></th></tr></thead><tbody><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border-top: 1px solid #dfe2e5;margin: 0px;padding: 0px;"><td style="box-sizing: border-box;border: 1px solid #dfe2e5;margin: 0px;min-width: 32px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">(a)</span></span></td><td style="box-sizing: border-box;border: 1px solid #dfe2e5;margin: 0px;min-width: 32px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">pseudonymisation and encryption</span></span></td><td style="box-sizing: border-box;border: 1px solid #dfe2e5;margin: 0px;min-width: 32px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">假名化与加密</span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border-top: 1px solid #dfe2e5;margin: 0px;padding: 0px;background-color: #f8f8f8;"><td style="box-sizing: border-box;border: 1px solid #dfe2e5;margin: 0px;min-width: 32px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">(b)</span></span></td><td style="box-sizing: border-box;border: 1px solid #dfe2e5;margin: 0px;min-width: 32px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">ongoing confidentiality, integrity, availability and resilience</span></span></td><td style="box-sizing: border-box;border: 1px solid #dfe2e5;margin: 0px;min-width: 32px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">持续保障保密性、完整性、可用性、系统韧性</span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border-top: 1px solid #dfe2e5;margin: 0px;padding: 0px;"><td style="box-sizing: border-box;border: 1px solid #dfe2e5;margin: 0px;min-width: 32px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">(c)</span></span></td><td style="box-sizing: border-box;border: 1px solid #dfe2e5;margin: 0px;min-width: 32px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">restore availability and access in a timely manner</span></span></td><td style="box-sizing: border-box;border: 1px solid #dfe2e5;margin: 0px;min-width: 32px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">发生安全事件后，及时恢复数据与系统访问能力</span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border-top: 1px solid #dfe2e5;margin: 0px;padding: 0px;background-color: #f8f8f8;"><td style="box-sizing: border-box;border: 1px solid #dfe2e5;margin: 0px;min-width: 32px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">(d)</span></span></td><td style="box-sizing: border-box;border: 1px solid #dfe2e5;margin: 0px;min-width: 32px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">a process for regularly testing, assessing and evaluating the effectiveness</span></span></td><td style="box-sizing: border-box;border: 1px solid #dfe2e5;margin: 0px;min-width: 32px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">定期测试、评估安全措施实际有效性</span></span></td></tr></tbody></table>  
很多企业最容易忽略第(d)项。重点两个关键词：  
**regularly（定期）、effectiveness（有效性）。**  
只做过一次漏洞扫描，不能证明“定期”；仅仅罗列一堆安全漏洞，无法证明你的防护措施真正生效。单次检测只是一个快照，法律要的是一套持续运转的流程。  
  
  
  
**0****2**  
  
  
**三类安全检测，分别能为GDPR证明什么？**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/U5ROicHxZUzqJGLCzfCd5WLAKuH1wdsqtF3tI2v5zCNiaU1TmE2KfRicYdJ3KHg4LHq9SBvmjJxrmltaicxvLrGyIkl90M7GxdSnwGqI11vB0Cg/640?wx_fmt=png&from=appmsg "")  
  
漏洞扫描、渗透测试、固件/APP安全检测，各有边界，不能互相替代。海外客户审核，要看的是你组合使用，并且清楚每一项检测的能力局限。  
  
<table><thead><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border-top: 1px solid #dfe2e5;margin: 0px;padding: 0px;"><th style="box-sizing: border-box;border-width: 1px 1px 0px;border-bottom-style: initial;border-bottom-color: initial;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: #dfe2e5;border-right-color: #dfe2e5;border-left-color: #dfe2e5;border-image: initial;margin: 0px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">检测类型</span></span></th><th style="box-sizing: border-box;border-width: 1px 1px 0px;border-bottom-style: initial;border-bottom-color: initial;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: #dfe2e5;border-right-color: #dfe2e5;border-left-color: #dfe2e5;border-image: initial;margin: 0px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">主要对应第32条分项</span></span></th><th style="box-sizing: border-box;border-width: 1px 1px 0px;border-bottom-style: initial;border-bottom-color: initial;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: #dfe2e5;border-right-color: #dfe2e5;border-left-color: #dfe2e5;border-image: initial;margin: 0px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">可以证明的内容</span></span></th><th style="box-sizing: border-box;border-width: 1px 1px 0px;border-bottom-style: initial;border-bottom-color: initial;border-top-style: solid;border-right-style: solid;border-left-style: solid;border-top-color: #dfe2e5;border-right-color: #dfe2e5;border-left-color: #dfe2e5;border-image: initial;margin: 0px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">单独拿出来举证的短板</span></span></th></tr></thead><tbody><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border-top: 1px solid #dfe2e5;margin: 0px;padding: 0px;"><td style="box-sizing: border-box;border: 1px solid #dfe2e5;margin: 0px;min-width: 32px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">漏洞扫描</span></span></td><td style="box-sizing: border-box;border: 1px solid #dfe2e5;margin: 0px;min-width: 32px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">(b)</span></span></td><td style="box-sizing: border-box;border: 1px solid #dfe2e5;margin: 0px;min-width: 32px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">系统已知漏洞、错误配置的暴露面；持续扫描可以证明安全检查执行频率</span></span></td><td style="box-sizing: border-box;border: 1px solid #dfe2e5;margin: 0px;min-width: 32px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">无法发现业务逻辑漏洞、权限越权等业务层面风险</span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border-top: 1px solid #dfe2e5;margin: 0px;padding: 0px;background-color: #f8f8f8;"><td style="box-sizing: border-box;border: 1px solid #dfe2e5;margin: 0px;min-width: 32px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">渗透测试</span></span></td><td style="box-sizing: border-box;border: 1px solid #dfe2e5;margin: 0px;min-width: 32px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">(a)、(b)</span></span></td><td style="box-sizing: border-box;border: 1px solid #dfe2e5;margin: 0px;min-width: 32px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">身份认证、权限控制、传输加密在真实攻击路径下是否可靠</span></span></td><td style="box-sizing: border-box;border: 1px solid #dfe2e5;margin: 0px;min-width: 32px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">测试时间、覆盖范围有限，报告必须写明测试边界</span></span></td></tr><tr style="box-sizing: border-box;break-inside: avoid;break-after: auto;border-top: 1px solid #dfe2e5;margin: 0px;padding: 0px;"><td style="box-sizing: border-box;border: 1px solid #dfe2e5;margin: 0px;min-width: 32px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">固件与APP检测</span></span></td><td style="box-sizing: border-box;border: 1px solid #dfe2e5;margin: 0px;min-width: 32px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">(a)、(b)、(c)</span></span></td><td style="box-sizing: border-box;border: 1px solid #dfe2e5;margin: 0px;min-width: 32px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">本地数据加密、通信加密、固件签名、个人数据残留情况</span></span></td><td style="box-sizing: border-box;border: 1px solid #dfe2e5;margin: 0px;min-width: 32px;padding:5px 10px;"><span style="color: #3e3e3e;font-size: 14px;letter-spacing: normal;caret-color: red;background-color: transparent;font-family:mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;"><span leaf="">缺少真实攻击者视角，风险优先级容易判断失真</span></span></td></tr></tbody></table>  
简单理解：漏扫广而不深，渗透深而不广，固件/APP检测聚焦终端设备本身。单一报告不足以支撑第32条合规举证。  
  
  
  
**0****3**  
  
  
**提交给欧盟客户的安全报告，必须补上3块关键内容**  
  
  
直接输出工具原始扫描报告，技术CVSS评分和GDPR个人数据风险之间是断裂的，审计方看不懂技术漏洞会带来多大的数据保护风险，交付前要做三层加工：  
1. **把技术漏洞翻译成业务层面的个人数据风险**  
  
扫描工具输出的是CVSS技术危险等级。你需要额外说明：这个漏洞会不会造成个人数据泄露、篡改、非法访问。 举例：调试接口的命令注入漏洞，对比日志过度留存配置缺陷，二者对于个人数据保护的影响完全不同，不能只靠分数一概而论。  
  
1. **提供完整的整改闭环证据**  
  
只写“漏洞已修复”没有举证效力。 需要写明：修复版本号、上线时间、受影响业务范围，以及漏洞从存在到修复的暴露时长。证明你确实处理了会威胁个人数据的安全隐患。  
  
1. **保留复测对比结论**  
  
复测的意义是验证漏洞被真正修复，不是只核对文档修改。 建议留存原始扫描结果和复测结果的对比表格，审计的时候，对比表的说服力远高于简单的文字结论页。  
  
  
  
  
  
**0****4**  
  
  
**怎么落地“定期测试”，满足第32条(d)的要求？**  
  
  
第32条(d)要求的是一套制度流程，不是一次性做一次安全检测。可以结合产品生命周期落地可执行节奏：  
- 重要产品版本发布前，执行一轮漏洞扫描；  
  
- 每年执行一次完整渗透测试；  
  
- 第三方SDK、云服务发生重大变更，补充专项安全评估。  
  
测试频率不能主观拍脑袋，参考两个维度：一是处理个人数据的敏感等级；二是过往检测问题的收敛情况。 如果连续两轮检测反复出现同类高危风险，代表检测间隔不合理，或是漏洞整改没有真正纳入研发发布流程。  
  
  
  
**0****5**  
  
  
**三个常见错误，会直接让你的安全报告在GDPR审计失效**  
  
  
**① 一份报告用多年**  
  
报告自带时间戳，审计人员第一眼看的就是测试日期与测试范围。几年前的快照，无法证明当前系统的安全状态。  
  
**② 报告缺失测试范围说明**  
  
没有明确测试边界，审计方无法判断哪些处理个人数据的模块被覆盖，哪些不在检测之内，大概率直接要求重新检测。  
  
**③ 安全检测结果和RoPA、DPIA脱节**  
  
GDPR的安全防护强度必须匹配业务风险，风险评估来源于RoPA（处理活动记录）、DPIA（数据保护影响评估）。 如果安全报告发现的风险，没有回流更新到这两份文档，那么安全风险就孤立存在，无法证明“安全措施适配业务风险”这个核心合规要求。  
  
  
  
**0****6**  
  
  
**小结**  
  
  
GDPR不是只有隐私告知、用户权利这些纸面工作，  
**网络安全是法律强制义务，第32条就是技术安全的法律落脚点。**  
  
 一份可以用于GDPR举证的安全材料，核心做到三点：漏洞按个人数据风险分级、全部隐患有完整整改记录、修复问题留有复测对比证据。  
  
  
  
**更多相关内容**  
  
**尽在视频号“GTG网络安全实验室”**  
  
  
  
  
