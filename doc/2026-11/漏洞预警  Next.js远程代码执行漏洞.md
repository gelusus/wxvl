#  漏洞预警 | Next.js远程代码执行漏洞  
浅安
                    浅安  浅安安全   2026-11-17 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-126187  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
Next.js是Node.js生态中基于React的开源Web框架，其通过Server Actions功能提供了后端开发能力。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/7stTqD182SXuxF6aeAAAcJpby4ECRt8pVzjgiaOP6LdpyObdiaUPjyEbpZGFArqpsIRJFreSTW9NOahpf6QNRnbw/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-126187**  
  
**漏洞类型：**  
远程代码执行  
  
**影响：**  
执行任意命令  
  
**简述：**  
Next.js存在远程代码执行漏洞，由于其Flight序列化器在处理多个TypedArray视图共享同一个ArrayBuffer的场景时存在逻辑缺陷，未经验证的攻击者可通过构造单个未认证请求触发服务端代码执行，进而执行任意代码。  
  
**0x04 影响版本**  
- 16.0.0 <= Next.js <= 16.4.0  
  
**0x05****POC状态**  
- 已公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://nextjs.org/  
  
  
  
