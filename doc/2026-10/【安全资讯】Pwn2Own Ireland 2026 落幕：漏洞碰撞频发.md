#  【安全资讯】Pwn2Own Ireland 2026 落幕：漏洞碰撞频发  
原创 360漏洞研究院
                    360漏洞研究院  360漏洞研究院   2026-10-09 04:32  
  
2026年Pwn2Own爱尔兰站（Pwn2Own Ireland 2026）于10月6日至8日在爱尔兰科克举行。赛事覆盖智能手机、AI基础设施、智能家居等多个领域，三星Galaxy S26、Google Pixel 10、OpenAI Codex等多个目标被成功攻破。  
  
  
**大赛亮点**  
- 三星Galaxy S26三天累计遭遇7次成功利用  
  
- Google Pixel 10第三天三次被成功攻破，合计颁发562,500美元奖金  
  
- OpenAI Codex、Oracle Autonomous AI Database、NVIDIA Dynamo及LiteLLM等AI相关目标接连被攻破  
  
- Philips Hue Bridge Pro、Sonos Era 300等智能家居设备被成功攻破  
  
- Team MAMMOTH通过6个零日漏洞组成的攻击链攻破Home Assistant Green  
  
**奖金情况**  
  
第一天：$388,500（32 个独特 0day）  
  
第二天：多项奖金已公布，官方当日汇总待确认  
  
第三天：多项奖金已公布，官方当日汇总待确认  
  
总计：待 ZDI 官方公布  
  
**🏆 Master of Pwn总冠军：**  
Ikotas Labs, Inc.  
  
  
**01**  
  
**第一天战报（10月6日）：三星Galaxy S26、OpenAI Codex与Oracle AI数据库成为重点攻击目标**  
  
  
💰 当日奖金：$388,500 | 独特零日漏洞：32个 | 参赛项目：21项  
  
  
**📱 智能手机项目**  
  
⚠️ Nguyen Thanh Dat (Viettel Cyber Security)  
  
攻击目标：Samsung Galaxy S26 – Remote  
  
漏洞类型：组合4个漏洞，其中3个已被厂商知晓  
  
战绩：**成功 + 撞洞**  
  
奖金与积分：$31,250 | 3.25分  
  
⚠️ Interrupt Labs  
  
攻击目标：Samsung Galaxy S26 – Remote  
  
漏洞类型：4个漏洞组合，包括1个独特零日漏洞和3个碰撞漏洞  
  
战绩：**成功 + 撞洞**  
  
奖金与积分：$15,750 | 3.25分  
  
⚠️ Ikotas Labs, Inc.  
  
攻击目标：Samsung Galaxy S26 – Remote  
  
漏洞类型：组合4个漏洞，其中1个已被厂商知晓但尚未修复  
  
战绩：**成功 + 撞洞**  
  
奖金与积分：$11,000 | 4.5分  
  
❌ Mikhail Evdokimov, Polina Smirnova, Mate Zombor (White Noise Club)  
  
攻击目标：Google Pixel 10 – Remote  
  
战绩：**失败**  
（未能在规定时间内完成漏洞利用）  
  
  
**🤖 AI工具链 / 编码助手 / AI基础设施项目**  
  
✅ Taisic Yun (Xint)  
  
攻击目标：LiteLLM  
  
漏洞类型：输入验证不当与代码注入组合  
  
战绩：  
**成功**  
  
奖金与积分：$40,000 | 4分  
  
⚠️ HaeJung Yang, ByungYoung Yi (Out of Bounds)  
  
攻击目标：LiteLLM  
  
漏洞类型：组合4个漏洞，其中2个碰撞漏洞  
  
战绩：**成功 + 撞洞**  
  
奖金与积分：$15,000 | 3分  
  
✅ Nam Nguyen, Thanh Vu, Tin Huynh (VinSOC)  
  
攻击目标：Oracle Autonomous AI Database  
  
漏洞类型：5个漏洞链式组合  
  
战绩：  
**成功**  
  
奖金与积分：$40,000 | 4分  
  
✅ Ikotas Labs, Inc.  
  
攻击目标：OpenAI Codex  
  
漏洞类型：参数注入  
  
战绩：  
**成功**  
  
奖金与积分：$40,000 | 4分  
  
❌ Nam Nguyen, Thai Son Dinh, Hoang Tien Minh (VinSOC)  
  
攻击目标：Chroma  
  
战绩：  
**失败**  
（未能在规定时间内完成漏洞利用）  
  
  
**🏠 智能家居项目**  
  
✅ @_McCaulay  
  
攻击目标：Sonos Era 300  
  
漏洞类型：越界写入与格式化字符串漏洞组合  
  
战绩：  
**成功**  
  
奖金与积分：$50,000 | 5分  
  
⚠️ linhlhq, Son Dinh (VinSOC)  
  
攻击目标：Sonos Era 300  
  
漏洞类型：组合2个漏洞，其中1个已公开漏洞  
  
战绩：**成功 + 撞洞**  
  
奖金与积分：$17,500 | 3.5分  
  
✅ Vũ Chí Thành, Huỳnh Đức Tin (VinSOC)  
  
攻击目标：Philips Hue Bridge Pro  
  
漏洞类型：7个独特零日漏洞组合利用  
  
战绩：  
**成功**  
  
奖金与积分：$40,000 | 4分  
  
⚠️ ByungYoung Yi, KeunHo Kim (Out of Bounds)  
  
攻击目标：Philips Hue Bridge Pro  
  
漏洞类型：组合5个漏洞，其中4个碰撞漏洞  
  
战绩：**成功 + 撞洞**  
  
奖金与积分：$12,000 | 2.5分  
  
⚠️ Joohyun Park (Xint)  
  
攻击目标：Philips Hue Bridge Pro  
  
漏洞类型：组合5个漏洞，其中4个碰撞漏洞  
  
战绩：**成功 + 撞洞**  
  
奖金与积分：$6,000 | 2.5分  
  
  
**🖨️ 打印设备项目**  
  
✅ Thanh Do (Team Confused)  
  
攻击目标：Lexmark CX532adwe  
  
漏洞类型：释放后重用漏洞  
  
战绩：  
**成功**  
  
奖金与积分：$20,000 | 2分  
  
✅ Sina Kheirkhah (Summoning Team)  
  
攻击目标：Lexmark CX532adwe  
  
漏洞类型：单个漏洞  
  
战绩：  
**成功**  
  
奖金与积分：$10,000 | 2分  
  
❌ Ikotas Labs, Inc.  
  
攻击目标：Brother MFC-L8970CDW  
  
战绩：  
**失败**  
（未能在规定时间内完成漏洞利用）  
  
❌ Hyeop Jung, Hyunwoo Kim, Jinyoung Choi, Jihun Lee (Team T-X Lab)  
  
攻击目标：Lexmark CX532adwe  
  
战绩：  
**失败**  
（未能在规定时间内完成漏洞利用）  
  
  
**⌚ 健康设备项目**  
  
✅ Interrupt Labs  
  
攻击目标：Garmin Index BPM  
  
漏洞类型：越界读取与越界写入组合  
  
战绩：  
**成功**  
  
奖金与积分：$20,000 | 2分  
  
❌ Aaron Christophel (Summoning Team)  
  
攻击目标：Garmin Index BPM  
  
战绩：  
**失败**  
（未能在规定时间内完成漏洞利用）  
  
  
**02**  
  
**第二天战报（10月7日）：NVIDIA Dynamo与Oracle AI数据库接连被攻破，三星Galaxy S26再遭成功利用**  
  
  
当日奖金：官方汇总待确认 | 发现漏洞：官方汇总待确认 | 计划尝试次数：24次  
  
  
**📱 智能手机项目**  
  
✅ Dimitrios Valsamaras, Ken Gannon, Tenia Valsamara (Mobile Hacking Lab / CENSUS Labs相关研究人员)  
  
攻击目标：Samsung Galaxy S26 – Remote  
  
漏洞类型：混淆代理漏洞  
  
战绩：  
**成功**  
  
奖金与积分：官方赛果未列明  
  
⚠️ Kyeongmin Kim (KAIST Hacking Lab)  
  
攻击目标：Samsung Galaxy S26 – Remote  
  
漏洞类型：组合3个漏洞，其中1个独特零日漏洞和2个碰撞漏洞  
  
战绩：**成功 + 撞洞**  
  
奖金与积分：$8,500 | 3.5分  
  
⚠️ PetoWorks  
  
攻击目标：Samsung Galaxy S26 – Remote  
  
漏洞类型：3个碰撞漏洞  
  
战绩：**成功 + 撞洞**  
  
奖金与积分：$6,250 | 2.5分  
  
  
**🤖 AI基础设施 / AI数据库项目**  
  
✅ HaeJung Yang (Out of Bounds)  
  
攻击目标：NVIDIA Dynamo  
  
战绩：  
**成功**  
  
奖金与积分：$40,000 | 4分  
  
⚠️ Taisic Yun (Xint)  
  
攻击目标：Oracle Autonomous AI Database  
  
漏洞类型：组合5个漏洞，其中2个独特零日漏洞和3个碰撞漏洞  
  
战绩：**成功 + 撞洞**  
  
奖金与积分：$14,000 | 3分  
  
✅ Ikotas Labs, Inc.  
  
攻击目标：Oracle Autonomous AI Database  
  
漏洞类型：7个漏洞链式组合，攻击链末端涉及释放后重用漏洞、类型混淆漏洞  
  
战绩：  
**成功**  
  
奖金与积分：$10,000 | 4分  
  
⚠️ Mingeun Kim, Chulhan Park, SeokHun Lee, Inhyung Lee, Sehyun Baek, Nayeon Lee, Junseok Kim (Team MAMMOTH)  
  
攻击目标：Chroma  
  
漏洞类型：组合3个漏洞，其中1个独特零日漏洞和2个碰撞漏洞  
  
战绩：**成功 + 撞洞**  
  
奖金与积分：$12,000 | 1.25分  
  
⚠️ Alessandro Fanio Gonzalez  
  
攻击目标：Chroma  
  
漏洞类型：组合2个N-day漏洞及1个漏洞碰撞  
  
战绩：**成功 + 撞洞**  
  
奖金与积分：$4,500 | 1分  
  
❌ Eugene (k3vg3n)  
  
攻击目标：Chroma  
  
战绩：  
**失败**  
（未能在规定时间内完成漏洞利用）  
  
  
**🏠 智能家居项目**  
  
✅ Yves Bieri (Xint)  
  
攻击目标：Home Assistant Green  
  
战绩：  
**成功**  
  
奖金与积分：$30,000 | 3分  
  
⚠️ Yassine Bengana, Maxence Schmitt (Doyensec)  
  
攻击目标：Home Assistant Green  
  
漏洞类型：组合3个漏洞，均为碰撞漏洞  
  
战绩：**成功 + 撞洞**  
  
奖金与积分：$7,500 | 1.5分  
  
⚠️ PetoWorks  
  
攻击目标：Home Assistant Green  
  
漏洞类型：组合5个漏洞，其中3个独特漏洞和2个碰撞漏洞  
  
战绩：**成功 + 撞洞**  
  
奖金与积分：$12,000 | 2.5分  
  
⚠️ Kyeongmin Kim (KAIST Hacking Lab)  
  
攻击目标：Home Assistant Green  
  
漏洞类型：组合4个漏洞，其中1个独特零日漏洞和3个碰撞漏洞  
  
战绩：**成功 + 撞洞**  
  
奖金与积分：$4,750 | 2分  
  
⚠️ @_McCaulay  
  
攻击目标：Home Assistant Green  
  
漏洞类型：组合6个漏洞，其中5个独特零日漏洞和1个碰撞漏洞  
  
战绩：**成功 + 撞洞**  
  
奖金与积分：$7,000 | 2.75分  
  
⚠️ Jack Dates (RET2 Systems)  
  
攻击目标：Sonos Era 300  
  
漏洞类型：组合2个漏洞，其中1个独特零日漏洞和1个碰撞漏洞  
  
战绩：**成功 + 撞洞**  
  
奖金与积分：$9,500 | 3.75分  
  
⚠️ Nguyen Thanh Dat, dungnm (Viettel Cyber Security)  
  
攻击目标：Sonos Era 300  
  
漏洞类型：组合3个漏洞，其中2个独特零日漏洞和1个碰撞漏洞  
  
战绩：**成功 + 撞洞**  
  
奖金与积分：$10,500 | 4.25分  
  
✅ Sina Kheirkhah (Summoning Team)  
  
攻击目标：Sonos Era 300  
  
漏洞类型：组合2个漏洞  
  
战绩：  
**成功**  
  
奖金与积分：$12,500 | 5分  
  
❌ Shio Kudo (GMO Flatt Security)  
  
攻击目标：Home Assistant Green  
  
战绩：  
**失败**  
  
  
**🖨️ 打印设备项目**  
  
✅ @_McCaulay  
  
攻击目标：Canon imageFORCE 1643F  
  
漏洞类型：组合3个漏洞，包括硬编码凭据、关键功能缺少身份验证和命令注入  
  
战绩：  
**成功**  
  
奖金与积分：$10,000 | 2分  
  
✅ Interrupt Labs  
  
攻击目标：Lexmark CX532adwe  
  
战绩：  
**成功**  
  
奖金与积分：$5,000 | 2分  
  
✅ Rick de Jager, Filippo Cremonese (Zellic / v12sec)  
  
攻击目标：Lexmark CX532adwe  
  
战绩：  
**成功**  
  
奖金与积分：$5,000 | 2分  
  
⚠️ @BoredPentester  
  
攻击目标：Lexmark CX532adwe  
  
漏洞类型：组合3个漏洞，其中2个独特漏洞和1个碰撞漏洞  
  
战绩：**成功 + 撞洞**  
  
奖金与积分：$4,250 | 1.25分  
  
❌ Thanh Do (Team Confused)  
  
攻击目标：Brother MFC-L8970CDW  
  
战绩：  
**失败**  
  
  
**⌚ 健康设备项目**  
  
⚠️ @_McCaulay  
  
攻击目标：Garmin Index BPM  
  
漏洞类型：组合2个漏洞，其中1个独特漏洞和1个碰撞漏洞  
  
战绩：**成功 + 撞洞**  
  
奖金与积分：$6,750 | 1.5分  
  
  
**03**  
  
**第三天战报（10月8日）：Google Pixel 10三次被成功攻破，Ikotas Labs斩获30万美元夺冠**  
  
  
💰 当日奖金：官方未公布汇总 | 发现漏洞：官方未公布汇总 | 计划尝试次数：17次  
  
  
**📱 智能手机项目**  
  
⚠️ Tim Becker, Yves Bieri (Xint)  
  
攻击目标：Google Pixel 10 – Remote  
  
漏洞类型：单个碰撞漏洞  
  
战绩：**成功 + 撞洞**  
  
奖金与积分：$150,000 | 15分  
  
⚠️ Ikotas Labs, Inc.  
  
攻击目标：Google Pixel 10 – Remote  
  
漏洞类型：多个漏洞组合利用  
  
战绩：**成功 + 撞洞**  
  
奖金与积分：$300,000 | 30分  
  
特别战绩：夺得Master of Pwn总冠军  
  
⚠️ BunkyoWesterns  
  
攻击目标：Samsung Galaxy S26 – Remote  
  
漏洞类型：组合2个漏洞，包括1个独特漏洞和1个碰撞漏洞  
  
战绩：**成功 + 撞洞**  
  
奖金与积分：$8,250 | 3.75分  
  
⚠️ Dimitrios Valsamaras, Ken Gannon, Tenia Valsamara (Mobile Hacking Lab / CENSUS Labs)  
  
攻击目标：Google Pixel 10 – Remote  
  
漏洞类型：组合2个漏洞，包括1个独特零日漏洞和1个碰撞漏洞  
  
战绩：**成功 + 撞洞**  
  
奖金与积分：$112,500 | 22.5分  
  
  
**🤖 AI工具链 / 编码助手 / AI基础设施项目**  
  
⚠️ Nikolaos Mourousias, Bruno Halltari (OtterSec)  
  
攻击目标：Oracle Autonomous AI Database  
  
漏洞类型：组合4个漏洞，包括1个独特零日漏洞和3个碰撞漏洞  
  
战绩：**成功 + 撞洞**  
  
奖金与积分：$6,250 | 2.5分  
  
⚠️ Connor Laidlaw, Matthew Keeley (Platform Security)  
  
攻击目标：Oracle Autonomous AI Database  
  
漏洞类型：组合5个漏洞，包括1个独特漏洞和4个碰撞漏洞  
  
战绩：**成功 + 撞洞**  
  
奖金与积分：$6,000 | 2.5分  
  
  
**🏠 智能家居项目**  
  
⚠️ Sina Kheirkhah, Aaron Christophel (Summoning Team)  
  
攻击目标：Philips Hue Bridge Pro  
  
漏洞类型：组合5个漏洞，全部为碰撞漏洞  
  
战绩：**成功 + 撞洞**  
  
奖金与积分：$5,000 | 2.5分  
  
⚠️ Ben Koo, Evangelos Daravigkas (Team DDOS)  
  
攻击目标：Home Assistant Green  
  
漏洞类型：组合5个漏洞，其中包含1个零日漏洞  
  
战绩：**成功 + 撞洞**  
  
奖金与积分：$4,500 | 2分  
  
⚠️ @_McCaulay  
  
攻击目标：Philips Hue Bridge Pro  
  
漏洞类型：组合4个漏洞，全部为碰撞漏洞  
  
战绩：**成功 + 撞洞**  
  
奖金与积分：$5,000 | 2分  
  
✅ Chulhan Park, Mingeun Kim, SeokHun Lee, Inhyung Lee, Sehyun Baek, Nayeon Lee, Junseok Kim (Team MAMMOTH)  
  
攻击目标：Home Assistant Green  
  
漏洞类型：6个独特零日漏洞组合利用  
  
战绩：  
**成功**  
  
奖金与积分：$7,500 | 3分  
  
  
**🖨️ 打印设备项目**  
  
✅ Lucas Van Haaren, Hugo Leclercq (FuzzingLabs)  
  
攻击目标：Brother MFC-L8970CDW  
  
漏洞类型：单个独特零日漏洞  
  
战绩：  
**成功**  
  
奖金与积分：$20,000 | 2分  
  
✅ Sina Kheirkhah, Aaron Christophel (Summoning Team)  
  
攻击目标：Canon imageFORCE 1643F Multifunction Copier  
  
漏洞类型：4个独特漏洞组合利用  
  
战绩：  
**成功**  
  
奖金与积分：$5,000 | 2分  
  
✅ Cong Thanh, Duc Hieu, Nam Dung  
  
攻击目标：Lexmark CX532adwe  
  
漏洞类型：2个独特漏洞组合利用  
  
战绩：  
**成功**  
  
奖金与积分：$5,000 | 2分  
  
  
**⌚ 健康设备项目**  
  
❌ Polina Smirnova, Mikhail Evdokimov, Mate Zombor (White Noise Club)  
  
攻击目标：Garmin Index BPM  
  
战绩：  
**失败**  
（未能在规定时间内完成漏洞利用）  
  
  
**04**  
  
**最终排名**  
  
  
**🏆 Master of Pwn总冠军：Ikotas Labs, Inc.**  
  
安全团队Ikotas Labs凭借多项高价值目标的成功利用，最终夺得本届Pwn2Own Ireland 2026赛事的“Master of Pwn（黑客大师）”冠军！  
  
  
本届赛事覆盖智能手机、AI基础设施、智能家居等多个领域，Google Pixel 10、Samsung Galaxy S26、OpenAI Codex等多个重要目标被成功攻破。大量零日漏洞及复杂漏洞利用链的披露，进一步凸显了AI基础设施与智能终端面临的安全风险，也将推动相关厂商加快漏洞修复与安全防护升级。  
  
  
参考来源：  
  
**[1] Pwn2Own Ireland 2026 - Day One Results**  
  
https://www.zerodayinitiative.com/blog/2026/10/6/pwn2own-ireland-2026-day-one-results  
  
**[2] Pwn2Own Ireland 2026 - Day Two Results**  
  
https://www.zerodayinitiative.com/blog/2026/10/7/pwn2own-ireland-2026-day-two-results  
  
**[3] Pwn2Own Ireland 2026 - Day Three Results & Master of Pwn**  
  
https://www.zerodayinitiative.com/blog/2026/10/8/pwn2own-ireland-2026-day-three-results-amp-master-of-pwn  
  
**[4] Pwn2Own Ireland 2026 - The Full Schedule**  
  
https://www.zerodayinitiative.com/blog/2026/10/5/pwn2own-ireland-2026-the-full-schedule  
  
  
  
“扫描下方二维码，进入公众号粉丝交流群。更多一手网安资讯、漏洞预警、技术干货和技术交流等您参与！”  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/dZ7ia5iaWFzz8YToicKab1BicPnEdr7jiatvQUVWSMnYTBeG5ibibgxkGAG1rF4pUdpowPcCmokOO5tp4UjjhUsos4Zf4VwE1aM9NTUz3ogfgdwwFw/640?wx_fmt=gif&from=appmsg "")  
  
  
建议您订阅360漏洞情报服务，获取更多漏洞情报详情以及处置建议，让您的企业远离漏洞威胁。  
  
  
邮箱：360VRI@360.cn  
  
网址：https://vi.loudongyun.360.net  
  
  
  
“洞”悉网络威胁，守护数字安全  
  
  
**关于我们**  
  
  
360漏洞研究院，隶属于360安全能力中心。其成员常年入选谷歌、微软、华为等厂商的安全精英排行榜, 并获得谷歌、微软、苹果史上最高漏洞奖励。研究院是中国首个荣膺Pwnie Awards“史诗级成就奖”，并获得多个Pwnie Awards提名的组织。累计发现并协助修复谷歌、苹果、微软、华为、高通等全球顶级厂商CVE漏洞3000多个，收获诸多官方公开致谢。研究院也屡次受邀在BlackHat，Usenix Security，Defcon等极具影响力的工业安全峰会和顶级学术会议上分享研究成果，并多次斩获信创挑战赛、天府杯等顶级黑客大赛总冠军和单项冠军。研究院将凭借其在漏洞挖掘和安全攻防方面的强大技术实力，帮助各大企业厂商不断完善系统安全，为数字安全保驾护航，筑造数字时代的安全堡垒。  
  
  
