#  【水】Exchange邮件系统ProxyShell攻击链复现  
原创 黄豆安全实验室
                    黄豆安全实验室  黄豆安全实验室   2026-10-22 12:09  
  
    这段文字其实是24年发在年圣频道的，现在来看确实是古人的智慧，正好公众号没得发了，就搬一下。  
  
    最近又遇到了一个阿根廷军队域名(.mil.ar)的公网资产。我认为这个资产归属听起来还是很哈人的，军✌，为了复健红队能力 （为了装逼） 于是乎花了一个下午的时间战斗爽，最终也是成功的把这个资产捣鼓rce了  
  
    Exchange ProxyShell由三个Exchange的漏洞组成。CVE-2021-34473，SSRF漏洞;CVE-2021-34523 Exchange PowerShell BackEnd提权漏洞;最后CVE-2021-31207后台任意文件写入漏洞。有没有什么脚本能运行一下就把整个攻击链复现完呢？有的，你川哥就发送了给我一个，也是我最开始在GitHub上找到的（这个脚本丢了，感兴趣的师傅可以根据特征自行找一下）。  
  
    这个脚本复现漏洞需要邮服系统的账号密码才能完成攻击链，脚本user.txt里边有几个默认密码，运行了过后发现都不是，傻瓜式利用失败——毕竟是军队的邮服系统怎么可能这么若智，德国国防部幽默弱口令123456除外。  
  
第二个脚本，一个八百多行的硕大脚本  
  
    https://github.com/horizon3ai/proxyshell  
  
    运行后还是失败，报错超时，连接断开服务器无响应，很正常，因为脚本毕竟是死的，这种利用过程非常复杂的漏洞作者能写出一个一般环境能用的已经相当不容易了。那就硬着头皮看原理吧  
  
    SSFR很好理解，利用Exchange自身URI解析造成的路径混乱，构造恶意请求从内网访问到不允许访问到的PowerShell端点。正常从浏览器访问mapi/nspi路径是需要输入账号密码的，利用这个漏洞就能直接访问了。  
  
代码层面的分析：  
  
    https://blog.csdn.net/qq_41874930/article/details/120037619  
  
    POC里边也给得很详细。利用SSRF漏洞访问端点时候大部分身份是NT Authroity/System，高权限，这个漏洞反常之处是其他漏洞权限越高越好，这个漏洞权限高到system了，system用户是没有邮箱的，导致端点鉴权过后异常反而会失败。系统鉴权是检查的是X-CommonAccessToken，所以得搞到个普通用户的token值才能调用端点，普通用户的token值怎么搞到呢？你得有邮箱账号密码登录进去抓包，这也是为什么网上的脚本需要邮箱账密才能利用。现在就陷入一个幽默死循环，我得先有token我才能getshell看你的token，但是我又得先有个token我才能getshell，跑网上脚本默认密码跑完了都不对，爆破似乎也太难了，堂堂.mil的军队邮服正常来说不会有弱口令。咋办呢?方法一，打个电话到阿根廷国防部去，哥们借你邮箱账号密码一用，不白用，等我拿到shell了改配置文件写个新的账号密码给你有借有还咱俩扯平了。  
方法二，参考国外大佬的文章https://peterjson.medium.com/reproducing-the-proxyshell-pwn2own-exploit-49743a4ea9a1  
  
    从system提权到域管administrator的过程。原理是如果请求头里边不包含Token值，服务器会运行代码GET参数X-Rps-CAT的值，然后再进行反序列化来创建一个Token令牌并传入BackendRehydrationModule方法完成鉴权，这就是我们用于访问powershell接口的令牌。审计源码我们可以得到CommonAccessToken值生成的规则，  
```
V + version + T + type + C + compress + data
if compress => decompress then if type is Windows
```  
  
    可以看到其他都是格式，这里最重要的自变量就是用户SUID和组SUID。利用之前的SSRF漏洞从/autodiscover/autodiscover.xml路径到/mapi/emsmdb路径可以得到SUID。组SUID就有点难了，这里也是为什么第二个脚本运行失败的原因。第二个脚本(地址:https://github.com/horizon3ai/proxyshell)不仅大且复杂，最生猛的地方在于在GitHub的md文件中作者就说了“No email address needs to be supplied  
”，我搜了一下全网各种exp脚本中过程中似乎就他一个人做到了这点。这里脚本查找组SUID和之前提过的老外那篇文章的思路是一样的，审计系统的cmdlet内容，最后发现能往RPC接口发送指定内容能够回显LegacyDn，再通过LegacyDn获取SUID。  
```
payload:/o=exchange/ou=Exchange Administrative Group (FYDIBOHF23SPDLT)/cn=Recipients/cn=970e9f8578c04ef98b1d451ef016fcc8-Administrator
```  
  
    第二个脚本也是这么做的的，为什么失败了呢。经过一整个下午半猜半查的各种尝试终于找到了原因。首先脚本是没错的，其他有这个漏洞的资产都能复现成功，问题出在阿根廷这个邮服本身。虽然ProxyShell这个链他没修复，但是他补了CVE-2018-8581这个漏洞。18年这个day是知道某个用户的sid，那么就可以模拟这个用户使用SOAP通过ews的api进行各种操作。这个漏洞的补丁正好打断了攻击链获取LegacyDn这一步。18年那个补丁是这么打的  
  
https://i.blackhat.com/USA21/Wednesday-Handouts/us-21-ProxyLogon-Is-Just-The-Tip-Of-The-Iceberg-A-New-Attack-Surface-On-Microsoft-Exchange-Server.pdf?fbclid=IwAR2V0-4k2yb8dmPP5Mksd8iHYTOfE6sBwygMt4wjq3M9be8Tw6TlH0andhA  
  
    所以最后的利用还得根据网站环境的不同(参考网上的资料，我个人总结的影响因素包括三个，RFC接口版本/autodiscover服务配置情况/SOAP版本)。最常见的绕过方法是在使用LegacyDn获取sid发包的那一步发包的包体后面，burp打开hex选项卡第21行填充空字节，修改了这里第二个脚本就能运行成功了。拿到sid后使用脚本就能计算出token值  
  
token计算器地址  
  
    https://github.com/yisaditatimi/ProxyShell  
  
    最后认证后任意文件写入是在端点用了New-MailboxExportRequest这个命令，将用户的邮箱导出到指定路径，导出的格式是PST格式，这个格式的文件用了微软的置换编码，所以写明文进去是没用的，webshell还要先处理一下，webshell编码器也在yisaditatimi的那个仓库里，原理就不解释了，编码的脚本都是现成的所以我也没花功夫去搞懂这个细节。总之你可以按着我这个文章的思路一步一步去手动复现，也可以最简单的，拿我周一频道发那个脚本一键傻瓜式利用就行。如果有补丁你就加空字节/改Content-Type/soap模板参数，都不用搞懂补丁的代码，就挨个试网上的绕过方法，手动发包绕过成功后问gpt怎么用python实现，然后在原有脚本的基础上复制粘贴修改一下就行了  
    最后这毕竟是个军✌️的东西我也不好挂txt我是真特么不想找/找不到绝对路径  
，find命令一运行就卡死 这里就跑脚本-c 执行whoami一下证明吧  
  
完整复现视频 https://www.youtube.com/watch?v=LbIYPFrltdA    
  
不得不感叹一句审出这个day的老外是真牛  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/SbtGVeibYWqt6KibJA97E6l54fRtTz8QickEOPgLQbK9UEl9ULOd01ACYOm00ey80mx0c8Hdpia8ibk7sO2heXRVyHObQUDPDHbG7jz9etWXwAzo/640?wx_fmt=png&from=appmsg "")  
  
  
