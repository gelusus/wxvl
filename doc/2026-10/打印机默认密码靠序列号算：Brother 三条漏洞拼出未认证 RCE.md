#  打印机默认密码靠序列号算：Brother 三条漏洞拼出未认证 RCE  
原创 黑卷
                    黑卷  黑卷的IoT攻防日记   2026-10-22 23:44  
  
先说结论：STAR Labs 盯上了一台最不起眼的办公设备——Brother MFC-J1010DW 多功能一体机，花 150 刀就能买到的那种。结果拆开一看，三个小毛病串起来，能在内网里不用任何账号密码，直接把它打穿。  
  
最后他们让打印机的小屏幕上打出了「STAR LABS!」几个字。听着像恶作剧，但能上屏就等于能执行任意代码，真要干坏事空间大得很。  
  
![关键数字一览](https://mmbiz.qpic.cn/sz_mmbiz_jpg/JdicK53hX84XcPSQT9tvpWY0cY4Le2xKBL1YPzdo6Y5iaWj2HhC5GjddGJA7DIZsVk2JxvXibxJV4AC58q3qH0Yichg0k2oicXvaOibPNXttKYBkA/640?from=appmsg "")  
## 三条漏洞，一条链  
  
这台机器固件 1.18 及以下都中招（厂商在 1.19 修了）。三个问题单看都不致命，拼一起就要命：  
  
一是 SNMP 空口能读序列号；二是默认管理员密码能从序列号直接算出来；三是固件能被未认证回滚，再配上一个 Referer 头的栈溢出。顺着这条线走一遍，全程不需要登录。  
  
![攻击链：SNMP → 算密码 → 回滚 → 溢出 → 控屏](https://mmbiz.qpic.cn/mmbiz_jpg/JdicK53hX84XcbdkKR81kvkl4QDic4jh3OokTicwhWHOxONibsVoUbk05Tic2Gib10KNC6AeF35ugAxuWUzrsvnEviaEyngxG76eWSZhWS6DvUyjYg/640?from=appmsg "")  
## 第一刀：SNMP 这把万能钥匙  
  
Brother 打印机出厂默认开着 SNMP，这本是给网管远程看设备状态用的。问题是，同一个网里任何人都能查，不用认证——community 字段填个 public 就行。  
  
查出来的东西里，有一项是打印机的序列号。序列号本来没啥大不了，坏就坏在下一步。  
## 第二刀：密码是算出来的  
  
Brother 的默认管理员密码，是拿序列号推出来的。  
  
研究员把算法扒清楚了：序列号前 16 位，拼上一张内置盐表里取出的几个字节（还做了减一和倒序），一起丢进 SHA-256，取哈希的前 8 个字符，再按一张固定映射表把某些字母换成符号，得到的就是后台密码。  
  
你品一下这个逻辑：序列号空口白拿，算法又是写死在固件里的。那所谓「默认密码」，其实约等于没有密码。公开信息喂进固定算法，算出来的东西就不配叫秘密。  
  
更现实的是，很多人买回打印机根本不会改这个后台密码。也就是说，攻击者只要跟这台机器在同一个网里——办公室、打印店、酒店大堂都算——先 SNMP 捞序列号，再本地跑一遍算法，几秒钟就能登进管理后台，剩下的就随他折腾了。  
## 第三刀：给打印机开时光机  
  
很多厂商靠固件更新来补漏洞。可要是攻击者能把固件「倒回去」呢？  
  
这台机器正好允许未认证的固件回滚：用户明明升级到了打好补丁的新版，攻击者从厂商官网存档里下个旧的有漏洞版本，不用任何凭证就能把打印机降级回去。补丁白打了，漏洞又活过来了。  
## 拆机：先把板子掏出来  
  
真正的硬菜是第三个漏洞——栈溢出。但要分析它，得先把固件搞到手。研究员把机器拆开，主板藏在顶部扫描组件下面。  
  
![拆开顶盖，主板在扫描头下面](https://mmbiz.qpic.cn/sz_mmbiz_jpg/JdicK53hX84UITg6s2eGdt4bxkPDK1cDWyEyOOkzbiaoqk9p4UXZ3QdEW4uTgKZKU4R8ibs0eEjOOG4BkbhBcFg9gnnGGbMhVibcRCpXQcaWsEQ/640?from=appmsg "")  
  
![主板正面全貌，中间那颗就是主控 GELATO](https://mmbiz.qpic.cn/mmbiz_jpg/JdicK53hX84UicPKIpdsM3ch8VojSL5yDEdgSCngl4fWCbYRVibc9A2Od1ssGEHtUSLVqKZibd9rahKLickKIjOicVjQJDIeCuL4EBDoO85cuia6AQ/640?from=appmsg "")  
  
主控印着「GELATO SCG111A」，这是个定制料号。研究员顺着「日本品牌」这条线，判断它其实是瑞萨 Renesas 的 RZ/A1，Cortex-A9 内核。旁边那颗是 Zentel 的 DRAM。  
  
![主控与内存特写：GELATO SCG111A + Zentel DRAM](https://mmbiz.qpic.cn/sz_mmbiz_jpg/JdicK53hX84VFvohMM7CBUeKpMeCVpUDIWcysCDDVoH9cqw3zJjjXwmPYUdEEWrslhoEg6EWaKyT8fZjK3iarOLzvtia7Hv4iaQPMu8JDoSjNLw/640?from=appmsg "")  
  
板子背面干净得多，只有一颗印着「2113 E001H」的小芯片，看周边元件，多半是个运放或光耦，跟安全没太大关系。  
  
![主板背面，只有一颗不认识的小芯片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/JdicK53hX84UVicVCfTsh6I8rtOQLuFtQxGWrvErv44JdRjz4fpIsD6jGTu5cejG8c7rPUASdeIhMib3yEBxZThzVqnbzyWYZlx8mjwPEA3TKY/640?from=appmsg "")  
  
![板上关键芯片对照卡](https://mmbiz.qpic.cn/sz_mmbiz_jpg/JdicK53hX84UngjLgSvZVrk59ReblTN96qZAhOhQQw68xWD0UHREdl2arQeBoBE7xZgaeSXlic4MsRLwUd0sjfZo6IWpWU0Yysa0hscetyVoo/640?from=appmsg "")  
## 抽固件：CH341 夹子直接读  
  
固件存在一颗 GigaDevice GD25Q127C 的 SPI NOR Flash 里，16MB。研究员用改成 3.3V 的 CH341 编程器夹上去，flashrom 一把梭就把整块 flash 读了出来。  
  
![CH341 编程器夹住 NOR Flash](https://mmbiz.qpic.cn/sz_mmbiz_jpg/JdicK53hX84XAKHqbAiaVH3Uibn6yic9vYiboxroKkv0HmA86ibhIvjF6kc0plWxE56HokK7YiaVM3ytr6ySIKuz07kutl5uibeR0OZaSKZrQIjyJsI/640?from=appmsg "")  
  
旁边还有颗 Puya 的 EEPROM，抽出来一看是空的，估计早就不用了。  
  
![EEPROM 抽出来是空的](https://mmbiz.qpic.cn/mmbiz_jpg/JdicK53hX84WsUM1AFVoWnPib9C8SQXNvT9dqrUJP3d1pKDqBsrlXpjJqFBP0XcwmdleoNFPISrBj6JjCgfRJsC9w6QfyrdNMkib2VuXT7Ahpg/640?from=appmsg "")  
## SWD：能连上，却走不动  
  
板上没找到能用的 UART 控制台，倒是有个丝印写着「JTAG」、实际是 SWD 的口。研究员拿 Tigard 板子 3.3V 接上，openocd 认出来是 Cortex-A9。  
  
![Tigard 接上丝印写 JTAG 的 SWD 口](https://mmbiz.qpic.cn/sz_mmbiz_jpg/JdicK53hX84W4Vo3DHre1O7w7VjWujtRfQoic3a4iaZEJtHkWqsicKicSScCpFZStLtjAbcutFcHfL1sNzvI0mhLKd31m8byHU8XauvNWHKibOpkQ/640?from=appmsg "")  
  
但想单步调试没那么顺：T-Kernel 这套 RTOS 启动后开了看门狗，一旦把 CPU 停住，看门狗超时就把芯片重启了。好在上电头 2 秒左右能提前 halt 住，趁这个窗口粗略追了下地址流，还把 RAM 内容 dump 了出来，留给 IDA 慢慢看。  
## 拆固件：zlib 一段段抠出来  
  
拿到 flash.bin，先做熵分析——整块都是高熵，说明里面一堆压缩或加密的段。  
  
![熵分析：整块高熵，典型固件结构](https://mmbiz.qpic.cn/mmbiz_jpg/JdicK53hX84VibLu2L9WZJ5SqGtLdcwjicfa1hicqPzJHfbLibQ1bpqVqTI5htm3N0rzibFl077oBVyibJBRqGTaOBFEwmRo7lnLFBV1MDaBTjmyHs/640?from=appmsg "")  
  
接着用 ImHex 翻二进制，发现一串像内存地址的字符串。写个 pattern 解析器一套，固件的内存布局就清楚了：一堆带名字、带加载地址的段。  
  
![翻出像内存地址的字符串表](https://mmbiz.qpic.cn/mmbiz_jpg/JdicK53hX84UvyCwusOsXJbnkVKoBmXib1ic2PRXX3DDdDtTt3NBY8CkKoKXpF6NQ4fRdH37bsMq6Vywogtse2icTtLktf0fPAH0GJzVia5iaEoSQ/640?from=appmsg "")  
  
关键发现是：zlibRo、zlibRw、zlib2Rw 这三段其实是首尾拼在一起的，用 Python 的 zlib 顺着解压就能一段段抠出来。  
  
![写脚本按段抽取并解压](https://mmbiz.qpic.cn/mmbiz_jpg/JdicK53hX84XFw9ECx6N4MVNC2Jiadj0EEWcBQMUUtUjnibJUQrKlSaYfeONjqhNTH6MVEQ5pqibsZKsoX9rnhkMkErb3ZuGs4nV1UCHyibU8S7U/640?from=appmsg "")  
## 丢进 IDA：把六段拼成一张图  
  
把抽出来的段按正确的加载地址丢进 IDA Pro，选 ARM 小端、32 位，再把六个段一个个加好、设对读/写/执行权限，重新分析加跑一遍 Find Functions，固件的函数就都浮出来了。  
  
![IDA 里拼好六个段的内存布局](https://mmbiz.qpic.cn/mmbiz_jpg/JdicK53hX84WgibtsWTU9db4jmRzW3D09wHTXPZJXeRQsPqXgfEjibBVb4m3ql7sYNlGKRScOia6G297cEOSFIyK8nopzjOcLt5VSNmq7ia5oLNQ/640?from=appmsg "")  
## 那个要命的 memcpy  
  
研究员参考了 Rapid7 之前那个 CVE-2024-51979（CSRFToken 头的栈溢出）找同类问题，果然在处理 Referer 头的函数里逮到一个。  
  
逻辑大概是：它先把 Referer 头的内容取出来，找到「//」之后的那段，一路 strcpy、再 memcpy 进一个栈上的小缓冲区——全程没做长度检查。只要 Referer 里塞超过 256 个字符，就把缓冲区打爆了。  
  
顺着调用链看，这个溢出发生在 handle_http_request → decode_params → decode_csrftoken 这一路的最里层。参数解码失败时，代码会去调一个回调函数来返回错误，而这个回调指针恰好就保存在被打爆的栈帧里。  
  
打爆之后，栈上保存的那个回调指针（落在 R9 里）被覆盖成研究员指定的地址，调用时的第一个参数（在 R6 里）也能一起控住。思路就清楚了：把回调指针指到一个能往屏幕上写字的函数，再让参数指向自己想显示的字符串，等它触发错误回调，代码就按攻击者的剧本跑了。  
  
（出于安全考虑，这里只讲原理，不给可直接照搬去打别人设备的完整利用代码。）  
## 收尾：让屏幕打出 STAR LABS!  
  
研究员在固件里翻到一个专门往 LCD 上写字的函数，地址 0x4054B380，正好有个 16 字符的上限——塞「STAR LABS!」刚刚好。  
  
把超长 Referer 发过去，打印机屏幕就乖乖显示出了这行字。全程没登录、没认证，这就是实打实的未认证远程代码执行。  
  
![打印机屏幕打出 STAR LABS!](https://mmbiz.qpic.cn/mmbiz_jpg/JdicK53hX84WQeD8icTwaGNLRowKuJck7oicQdcEpxtVY3xmGVCezl7dNfibwt2nTSA9voY7jeQGoiaoGVIyHAmxeMM5DdeZb0DRbBK8cQ0Bw6jk/640?from=appmsg "")  
  
研究员也说清楚了：弹个字只是概念验证。真遇上坏人，一台能跑任意代码、又常年挂在内网里的打印机，可以拿来做内网跳板、截留打印任务里的敏感文件、或者长期潜伏当后门——比弹一行字难看得多。  
## 怎么堵  
  
![用户先堵、厂商根治的防护清单](https://mmbiz.qpic.cn/sz_mmbiz_jpg/JdicK53hX84W4Q5Ls72Q3C5QIeO4EQy8bmKSrQnD0g7Gz4xRlgKdldJXXwn4VKEXEtWZDmh3kA2ntP9znLTYZNTib22t2R24Wn0j89alYqwCk/640?from=appmsg "")  
  
用户这边：固件升到 1.19 以上；用不到就把 SNMP 关掉；默认密码一定改；有条件把打印机单独扔一个 VLAN。厂商那边改在根上：SNMP 要认证、别拿序列号算密码、固件更新必须鉴权还要防回滚、字符串操作老老实实做边界检查。  
## 黑卷点评  
  
![能算出来的密码就不叫密码](https://mmbiz.qpic.cn/mmbiz_jpg/JdicK53hX84VWr3Mzve7UawMicS9el8mlW5XcnWQT2S3xjsjGg1ydUOke0T9xVZHpYP196PFXvh4Oh7lfw4icT1kcciaFmAicqs2g8axrYyIia9WA/640?from=appmsg "")  
  
这篇最值得记的，不是那个栈溢出有多精巧，而是「默认密码能从序列号算出来」这种设计。序列号又能空口白拿，整条链最软的一环压根不在代码里，在产品设计的脑子里。  
  
办公室里那台安静吃灰、谁都不管的打印机，常常就是内网里最好打的一个点。它有网口、有固件、有 Web 后台，却几乎没人给它做安全。下次做内网评估，别跳过打印机。  
  
 ⚠️ 免责声明  
  
 本人所有文章均为技术分享，均用于防御为目的的记录，请勿用于其他用途，否则后果自负。   
  
![星球「黑卷的IoT攻防日记」](https://mmbiz.qpic.cn/mmbiz_jpg/JdicK53hX84USqCKOkqNXicJsbTCibicakCTreZVhf6EXQgibw4oNRHmyfD7dL7ZxWhBPZicmFTR7CKuIbw1IqIHe8m74yqa1AvczW12icR2BqDicSI/640?from=appmsg "")  
  
👇 扫码进星球「黑卷的IoT攻防日记」· 🎟 早鸟价 ¥98/年  
  
**往期推荐**  
- [从调试口抽到改固件：Sonoff ZigBee 网关的芯片到云端两条 CVE](https://mp.weixin.qq.com/s?__biz=Mzg5MjY0MzU0Nw==&mid=2247486121&idx=1&sn=fa00d6550883f102a66c5308fc9197e8&scene=21#wechat_redirect)  
  
  
- [EOL 路由照样打穿：Netgear WGR614v9 UART + Bitdefender Box SPI 降级 RCE](https://mp.weixin.qq.com/s?__biz=Mzg5MjY0MzU0Nw==&mid=2247485429&idx=1&sn=5a4cdfd6982657765522258b8310892d&scene=21#wechat_redirect)  
  
  
- [从 UART 焊到未认证后门：ANJIA PTZ 摄像头（CVE-2026-31077）](https://mp.weixin.qq.com/s?__biz=Mzg5MjY0MzU0Nw==&mid=2247486026&idx=1&sn=3786d4d0ca929f9d35b3913c6a6d6c1a&scene=21#wechat_redirect)  
  
  
