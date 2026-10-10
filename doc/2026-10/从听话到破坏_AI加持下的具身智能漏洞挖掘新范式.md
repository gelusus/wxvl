#  从听话到破坏_AI加持下的具身智能漏洞挖掘新范式  
蓝星安全
                    蓝星安全  蓝星安全   2026-10-09 22:00  
  
         
**点击上方****蓝星安全****关注我**  
  
****  
  
**免责声明：本公众号分享的任何资料仅限用于安全学习，严禁用于其他用途，请严格遵守中华人民共和国法律法规，对因不遵守国家法律法规而产生的任何后果，均由个人自行承担，本公众号不承担任何责任！**  
  
****  
**获取资料，请扫码下方二维码加入知识星球**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/opKHaHXLxcmxg12rU33pwcICNRkiaof5YSUAGfWPApU7M1BfdsTTOyvREw0gKD42g4U8eefG4n0XuYETEtbxSIN2drACnNVz3UeicCcldM5AA/640?wx_fmt=png&from=appmsg "")  
  
**《从「听话」到「破坏」——AI 加持下的具身智能漏洞挖掘新范式》**  
聚焦四足机器人、人形机器人、酒店机器人与智能汽车等具身智能设备的漏洞挖掘实践。文档披露了团队在 3 家厂商产品中发现 20+ 个漏洞（其中 10+ 高危）的成果，围绕「APP 应用 → 云控平台 → 本地通信层 → 硬件控制层」四层架构系统梳理了攻击面：硬件层的 UART/JTAG/SWD/USB/ADB 调试接口可被物理接触后获取 Shell 与固件，固件层存在 SSH 后门公钥、WiFi PSK 与 AES 密钥硬编码、U-Boot 默认凭证等 7 个 CRITICAL 供应链级漏洞（影响所有同型号设备），通信层的 BLE/WiFi/DDS/CAN/MAVLink/MQTT/WebRTC 协议普遍缺乏认证；标题中的「听话」指可控制机器人运动行为，「破坏」则指可突破其避障机制造成物理伤害、撞人、开舱门等现实风险。文档的核心方法论是「古法挖洞 vs AI 挖洞」的对比——以 CVE-2025-35027（蓝牙命令注入）等历史漏洞为起点挖掘出 0day（如 CVE-2025-60250/60251 密钥硬编码与认证绕过），并借助 Trae+jadx-ai-mcp、Claude+frida-mcp、BurpMCP-Ultra、ida-pro-mcp 等 MCP 工具链，把原本需要 1 周的蓝牙配网脚本压缩到 1.5 小时、WebRTC 控制脚本压缩到半天；文档同时强调 AI 是「效率倍增器」而非替代者——AI 负责重复性分析，研究员专注创造性突破与 PoC 验证，最终给出了 P0~P3 分级修复建议与网络/设备/应用/云端四层防护框架。完整版文档已上传蓝星安全知识星球，以下是部分内容：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/opKHaHXLxcmm0iaKhm4dPuBxJldPfEa0ZCfg0DFO2uhZWoI2J5IUhwb929UrD1YI2TD2qDoRS2dgcXdnRhzd1kuhLqQiauoREshGibvbDOTJyU/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/opKHaHXLxcnwicibAu3tQmXffgUob5mZumXoe6JC1Ciaz3v2zFqhISvj16aWAKib7GOoMOcDN1r3hdEulCsyc3XhIcfM9BjjwS69NCmxJSXzOgU/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/opKHaHXLxcn1L5jLVAMr2e6KDjQXenHXuub6AJhOCiaa97o5g4WU5icIcVyHLQBPZ68X7HokZ4KKAELYx3KTPNeXZTNdIBvT8G1wEPePaSyk0/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/opKHaHXLxckBcK1TpkdickERKX2EshmalhP1aicnxB6pR3tyLstLMlEBICdVLU5afibNsptu68A4yFOjicMMlAufI2ZZpDiamN4khjFv12nKpDPY/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/opKHaHXLxckCArEk0hgVcZhfAfYN66wRHBmFHxLGZM5LKuAHYxHOE6EktpBGW6b6aZmSezKibcL6wJtawYdBaZZpoYWHACicnFRIuXB2clzdk/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/opKHaHXLxcldnokrSJrLHZyOedETd45xwrc72xYUNN8ic6So2uVS4xkzdh4ibibvNgTHQOZCBdsecjs05qdLqdyF7pibVx3jUcaicyDpft9z8P9Y/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/opKHaHXLxckSyPOlicrCiaWGQ7lFWgE9pNpodsqsIq1hvKR1SfuhsNsZYWF8w01CsgBs6SpUYbzEFk13iaQPNMQBotXf4s9s93icqdNb2Oek2bo/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/opKHaHXLxcmjbcicA174zmsdEP7nfxZaQwNHxKv35E7DFwAibuL354nyFVRuuhiaa78VszpLjzqNgOz0EzuQIr24ozWl8M3EC9fdIZlq81maGU/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/opKHaHXLxcmuKAoyvSXHPzkmw2WHfDFSjhyME08fvTdSg62emLQC5mZBzpYLHm61hC9IXFfJBwJa1ricfkObGGKIDyOEAGGrmojnszmSkJps/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/opKHaHXLxclMddo8Goicje5duXDocWRsRs9fR86VCTibNtSnIbG0jJrlb3f9nY8NxcFLu0vW99GQ1B59DB6OykIdMLibpTL0NQZvgFoC0n40aI/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/opKHaHXLxckmxoPkQaljUkNEtyKl4iaM0qFQegnul0BD16Y7U3cePRuqsCH5icmicAKXbwupAtGACSMej1eHWDKfpnv2sz3euOosgPHvnJCvXU/640?wx_fmt=png&from=appmsg "")  
  
  
  
  
