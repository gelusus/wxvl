#  Android现死亡短信高危漏洞，隐形zero-click短信可阻断急救通话  
 FreeBuf   2026-11-02 10:04  
  
![FreeBuf](https://mmbiz.qpic.cn/sz_mmbiz_gif/icBE3OpK1IX27kUwzqxb2HUhJia2quk6hQbNTHU01h4FDhMXwBFQakM2xRSQZb6aia5fVtyiaz4qlQxZbrS79HZMYvVr3VqKeibb1o9yWBdr8X04/640?wx_fmt=gif "")  
  
  
![Android死亡短信漏洞](https://mmbiz.qpic.cn/sz_mmbiz_jpg/icBE3OpK1IX09056ewMOXHeicSiagW1j60Xx80XdF8poxy8I687FNYrVLIAwdz5Pic9nKG3sePO9ibTaz9MYo0iciaJGtTiaEG3PDNQV16oxgcRBmao/640?wx_fmt=jpeg "")  
  
  
攻击者只需发送一条经过特殊构造的短信，就能打挂Android的电话服务。这类攻击会中断正在进行的通话，导致智能手机暂时无法使用，全程无需受害者进行任何交互，甚至会阻断用户拨打急救电话的通路。  
  
  
研究人员将该漏洞命名为"死亡短信"，它再次引发了外界对Android智能手机消息系统安全的关注。独立研究员Mark Blaszczyk记录了该漏洞的细节：远程攻击者只需发送一条特制的WAP Push消息，就能中断目标设备的电话服务。  
  
  
WAP Push是通过移动网络传输的特殊消息，通常以二进制短信形式存在，包含结构化数据，无需用户干预即可由智能手机自动处理。与传统短信不同，WAP Push可以承载通知、互联网资源链接以及移动服务配置信息。  
  
  
Part  
01  
  
特制WAP Push消息触发漏洞  
  
  
该漏洞最值得警惕的是其触发方式。这是一种zero-click攻击，收件人无需打开消息、点击链接或授权任何操作。操作系统会自动处理消息内容，进而导致负责电话功能的进程崩溃。  
  
  
该漏洞影响Android框架组件WapPushOverSms，该组件位于frameworks/opt/telephony路径下，负责解析和管理设备收到的WAP Push消息。漏洞的技术成因是消息解析过程中对某个数值的处理存在错误。具体来说，decodeWapPdu()函数会读取以uintvar格式编码的WSP头长度，该值最初以64位数字形式返回，随后会在未做范围校验的情况下被转换为32位整数。  
  
  
这个转换操作可能会把看似合法的数值转为负数，而这个负数会被用来确定数组的大小。当Android尝试分配大小为负数的数组时，就会抛出NegativeArraySizeException异常。该异常在处理流程中未被正确捕获，最终导致负责电话功能的com.android.phone进程终止。  
  
  
研究人员验证，仅需5字节的WSP头就能生成值-2147483648，足以触发该漏洞，攻击者不需要发送体积很大的内容就能发动攻击。分析文档写道："5字节的uintvar值88 80 80 80 00解码后为0x80000000，转换为32位有符号int类型时值为-2147483648。系统尝试分配new byte[-2147483648]时，会在电话主线程抛出未捕获的NegativeArraySizeException异常，直接导致com.android.phone进程终止。"  
  
  
Part  
02  
  
攻击过程隐形无感知  
  
  
这类恶意消息通过2948端口的WAP Push通道以数据短信形式传输。与普通短信不同，它不会显示在短信收件箱中，也不会生成用户可识别的通知，发件人号码通常也不会在会话中显示。设备收到消息后，Android会自动开始处理内容。  
  
  
电话进程崩溃会导致设备暂时失去连接，中断正在进行的语音通话，包括拨打急救服务的电话。研究人员公布的验证结果显示，设备每处理一条恶意消息，电话服务就会中断1到2秒，期间相关组件会自动重启。但如果设备反复处理恶意消息，问题会严重得多。"死亡短信"最危险的一点在于，单次进程崩溃可能演变为持续性拒绝服务状态。恶意消息会残存在短信接收队列中，在后续投递尝试时被重复处理，运营商的重传机制也可能导致事件反复触发，进一步加剧服务中断。  
  
  
研究人员观察到，设备仅收到一条恶意消息后，间歇性崩溃状态最长可持续一周。发送约20条特制消息，就能让设备在很长一段时间内无法使用。更严重的情况下，电话进程会在系统启动阶段持续崩溃，引发类似启动循环的故障。  
  
  
要恢复设备正常使用，用户需要借助电脑上的Android调试桥（ADB）直接清理待处理消息队列中的恶意内容。对普通用户来说，这个操作门槛极高。  
  
  
Part  
03  
  
漏洞波及多版本Android设备  
  
  
"死亡短信"漏洞表明，网络攻击不一定非要窃取信息、执行代码或控制设备，才能造成严重后果。这次攻击的目标是电话服务可用性，而这项功能在用户拨打急救电话时尤为关键。这种无需任何用户交互就能中断通信的特性，让该漏洞的风险等级格外高。用户要降低风险，需检查设备的Android安全补丁级别，及时安装可用更新，尤其要关注那些已停止获得厂商支持的设备。  
  
  
Mark Blah开展的测试显示，该漏洞可在多款设备上复现，包括Google Pixel 9，以及三星Galaxy、Galaxy A系列智能手机。存在漏洞的代码存在于Android 12到Android 17的所有版本中，但不同系统版本获得补丁的情况并不统一。  
  
  
根据研究人员公布的文档，Google已经通过2026年10月的安全更新，为Android 14、15、16、17版本推送了漏洞修复补丁。运行Android 12、13等旧版本系统的设备如果无法获得对应的修复更新，将一直面临漏洞风险。  
  
  
GrapheneOS于2026年10月29日在X平台发文表示："该漏洞编号为CVE-2026-58882，在2026年11月Android安全公告中被评为严重级别。自2026072901版本起，我们就已将该漏洞补丁纳入安全预览版本，相关说明可见发行说明。"  
  
  
参考来源：  
  
SMS of Death: Android under attack, an invisible SMS can block calls and emergency services  
  
https://securityaffairs.com/202184/security/sms-of-death-android-under-attack-an-invisible-sms-can-block-calls-and-emergency-services.html  
  
  
### 推荐阅读  
  
  
[](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651349303&idx=1&sn=86458815b0d0b880c32e19864a449a98&scene=21#wechat_redirect)  
  
  
###   
  
### 电报讨论  
  
  
[]()  
  
  
  
![扫码加入AI安全交流群](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0Py7ibxdLKXia1pMziaic5vIE9XPXG9OGaeJDa07iaG10eicuzhW59nwpF5msHiaYZvfMqCNkx2aFDiaMzm3oAf4rTaHXU5UAI1mUYgts/640?wx_fmt=png "")  
  
  
  
![下载FreeBuf知识大陆APP](https://mmbiz.qpic.cn/mmbiz_png/icBE3OpK1IX0TIGzII2Hcmtzu7AJeZFicnqd1mXojVoawje2uLxYqwJbVgzJpmSXzVhrpOsLurRZ2lVa4vfgLBqg7uJKbrKg5F18VzZxVPicZU/640?wx_fmt=png "")  
  
  
  
  
