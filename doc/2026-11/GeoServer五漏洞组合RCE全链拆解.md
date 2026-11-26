#  GeoServer五漏洞组合RCE全链拆解  
原创 予辉安全
                    予辉安全  予辉安全   2026-11-26 10:00  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/2K4EpvaFFClay5IHWNuY8h7ATvaibnex5580qxT68BjT8K3c9zP3aOBTWbBRRky1z9dJ3RrS6iaxIPgepnTKEO6g/640?wx_fmt=gif&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFR489vPHAamAqcSDJ3CPHw9hjiaWWpKlqVPUNiaLshSw4grRroibfbib7E8AyCIC8ZNDzBsooCYlAfPtFViblUZZnmictoKK3hst0qKM7c4oibfVg/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFR489vPHAbbMb1atCO66ib9b8qWb0yDT8ibbk3uZcO3gHSzplAhtz1LHs9uXhwx9mY3ZcicN4g7TL9zkB9BDuKxEj9X2wJn2NdEVjPjUFb0js/640?wx_fmt=png&from=appmsg "")  
  
**-漏洞描述-**  
  
  
  
单个 CVE 单独看，GeoServer 官方给它们的评分并不全是“触目惊心”的程度：路径泄露 6.9、安全规则绕过 8.8、XInclude 文件读取 8.6、remember-me 伪造 9.2、JDBC 反序列化 8.6。真正的问题是，当这五个缺陷串在一条链上时，**未认证攻击者可以在默认安装上从“什么都不知道”走到“命令执行”**  
 ，官方容器镜像默认以 root 运行，拿到的是最高权限  
。  
  
漏洞链的发现者是 Rapid7 研究员 Jonah Burgess（Crypto-Cat），2026 年 8 月 2 日报告，11 月 23 日 GeoServer 发布修复版本并公开五份安全公告。CNCERT 在 11 月 25 日收录了该漏洞链，综合评级“高危”，并确认 **漏洞技术细节与 PoC 已传播**  
。watchTowr 已报告在真实环境中观察到利用尝试  
  
**-影响版本-**  
  
  
<table><thead><tr><th data-colwidth="165" style="border-bottom: 0.909091px solid rgba(0, 0, 0, 0.12);font: 500 15px / 25px quote-cjk-patch, Inter, system-ui, -apple-system, BlinkMacSystemFont, &#34;Segoe UI&#34;, Roboto, Oxygen, Ubuntu, Cantarell, &#34;Open Sans&#34;, &#34;Helvetica Neue&#34;, sans-serif;border-top-width: medium;border-top-style: none;border-top-color: currentcolor;min-width: 100px;max-width: min(30vw, 320px);padding: 10px 16px 10px 0px;text-align: left;"><span><span leaf="">分支</span></span></th><th style="border-bottom: 0.909091px solid rgba(0, 0, 0, 0.12);font: 500 15px / 25px quote-cjk-patch, Inter, system-ui, -apple-system, BlinkMacSystemFont, &#34;Segoe UI&#34;, Roboto, Oxygen, Ubuntu, Cantarell, &#34;Open Sans&#34;, &#34;Helvetica Neue&#34;, sans-serif;border-top-width: medium;border-top-style: none;border-top-color: currentcolor;min-width: 100px;max-width: min(30vw, 320px);padding: 10px 16px;text-align: left;"><span><span leaf="">受影响版本</span></span></th><th style="border-bottom: 0.909091px solid rgba(0, 0, 0, 0.12);font: 500 15px / 25px quote-cjk-patch, Inter, system-ui, -apple-system, BlinkMacSystemFont, &#34;Segoe UI&#34;, Roboto, Oxygen, Ubuntu, Cantarell, &#34;Open Sans&#34;, &#34;Helvetica Neue&#34;, sans-serif;border-top-width: medium;border-top-style: none;border-top-color: currentcolor;min-width: 100px;max-width: min(30vw, 320px);padding: 10px 16px;text-align: left;"><span><span leaf="">修复版本</span></span></th></tr></thead><tbody><tr><td data-colwidth="165" style="border-bottom: 0.909091px solid rgba(0, 0, 0, 0.1);font: 400 15px / 25px quote-cjk-patch, Inter, system-ui, -apple-system, BlinkMacSystemFont, &#34;Segoe UI&#34;, Roboto, Oxygen, Ubuntu, Cantarell, &#34;Open Sans&#34;, &#34;Helvetica Neue&#34;, sans-serif;min-width: 100px;max-width: min(30vw, 320px);padding: 10px 16px 10px 0px;"><span><span leaf="">3.0.x</span></span></td><td style="border-bottom: 0.909091px solid rgba(0, 0, 0, 0.1);font: 400 15px / 25px quote-cjk-patch, Inter, system-ui, -apple-system, BlinkMacSystemFont, &#34;Segoe UI&#34;, Roboto, Oxygen, Ubuntu, Cantarell, &#34;Open Sans&#34;, &#34;Helvetica Neue&#34;, sans-serif;min-width: 100px;max-width: min(30vw, 320px);padding: 10px 16px;"><span><span leaf="">3.0.0 – 3.0.1</span></span></td><td style="border-bottom: 0.909091px solid rgba(0, 0, 0, 0.1);font: 400 15px / 25px quote-cjk-patch, Inter, system-ui, -apple-system, BlinkMacSystemFont, &#34;Segoe UI&#34;, Roboto, Oxygen, Ubuntu, Cantarell, &#34;Open Sans&#34;, &#34;Helvetica Neue&#34;, sans-serif;min-width: 100px;max-width: min(30vw, 320px);padding: 10px 0px 10px 16px;"><strong style="font-weight: 600;"><span><span leaf="">3.0.2</span></span></strong></td></tr><tr><td data-colwidth="165" style="border-bottom: 0.909091px solid rgba(0, 0, 0, 0.1);font: 400 15px / 25px quote-cjk-patch, Inter, system-ui, -apple-system, BlinkMacSystemFont, &#34;Segoe UI&#34;, Roboto, Oxygen, Ubuntu, Cantarell, &#34;Open Sans&#34;, &#34;Helvetica Neue&#34;, sans-serif;min-width: 100px;max-width: min(30vw, 320px);padding: 10px 16px 10px 0px;"><span><span leaf="">2.28.x</span></span></td><td style="border-bottom: 0.909091px solid rgba(0, 0, 0, 0.1);font: 400 15px / 25px quote-cjk-patch, Inter, system-ui, -apple-system, BlinkMacSystemFont, &#34;Segoe UI&#34;, Roboto, Oxygen, Ubuntu, Cantarell, &#34;Open Sans&#34;, &#34;Helvetica Neue&#34;, sans-serif;min-width: 100px;max-width: min(30vw, 320px);padding: 10px 16px;"><span><span leaf="">2.28.0 – 2.28.5</span></span></td><td style="border-bottom: 0.909091px solid rgba(0, 0, 0, 0.1);font: 400 15px / 25px quote-cjk-patch, Inter, system-ui, -apple-system, BlinkMacSystemFont, &#34;Segoe UI&#34;, Roboto, Oxygen, Ubuntu, Cantarell, &#34;Open Sans&#34;, &#34;Helvetica Neue&#34;, sans-serif;min-width: 100px;max-width: min(30vw, 320px);padding: 10px 0px 10px 16px;"><strong style="font-weight: 600;"><span><span leaf="">2.28.6</span></span></strong></td></tr><tr><td data-colwidth="165" style="border-bottom: 0.909091px solid rgba(0, 0, 0, 0.1);font: 400 15px / 25px quote-cjk-patch, Inter, system-ui, -apple-system, BlinkMacSystemFont, &#34;Segoe UI&#34;, Roboto, Oxygen, Ubuntu, Cantarell, &#34;Open Sans&#34;, &#34;Helvetica Neue&#34;, sans-serif;min-width: 100px;max-width: min(30vw, 320px);padding: 10px 16px 10px 0px;"><span><span leaf="">2.27.x 及更早</span></span></td><td style="border-bottom: 0.909091px solid rgba(0, 0, 0, 0.1);font: 400 15px / 25px quote-cjk-patch, Inter, system-ui, -apple-system, BlinkMacSystemFont, &#34;Segoe UI&#34;, Roboto, Oxygen, Ubuntu, Cantarell, &#34;Open Sans&#34;, &#34;Helvetica Neue&#34;, sans-serif;min-width: 100px;max-width: min(30vw, 320px);padding: 10px 16px;"><span><span leaf="">≤ 2.27.6</span></span></td><td style="border-bottom: 0.909091px solid rgba(0, 0, 0, 0.1);font: 400 15px / 25px quote-cjk-patch, Inter, system-ui, -apple-system, BlinkMacSystemFont, &#34;Segoe UI&#34;, Roboto, Oxygen, Ubuntu, Cantarell, &#34;Open Sans&#34;, &#34;Helvetica Neue&#34;, sans-serif;min-width: 100px;max-width: min(30vw, 320px);padding: 10px 0px 10px 16px;"><strong style="font-weight: 600;"><span><span leaf="">2.27.7</span></span></strong></td></tr></tbody></table>  
**-利用链路-**  
  
  
### 第一环 · CVE-2026-129244  
  
WFS 的 storedQuery_id  
 参数在读取对应的查询描述文件失败时，会把**尝试打开的绝对路径原样写入 ows:ExceptionText 返回给匿名用户**  
。用一个超长的 storedQuery_id  
 触发文件系统路径超限错误，异常信息里就带出了数据目录的绝对路径  
。  
  
单独看，这只是一个信息泄露，CVSS 6.9。但没有它，后续环节不知道该把 XInclude 指向哪里、不知道该从哪个路径读 security/config.xml  
。它是整条链的“侦察兵”。  
### 第二环 · CVE-2026-129243：用 x/WFS 骗过门禁  
  
GeoServer 的 Dispatcher.findService  
 会把 service  
 参数中 /  
 之前的内容删掉来选择实际服务，所以 service=x/WFS  
 最终会被分派给 WFS。但安全检查回调拿到的仍然是原始的 x/WFS  
，它去和针对 wfs  
 写的规则比较，**没有规则匹配，角色检查直接被跳过**  
。  
  
默认配置把 CreateStoredQuery  
 和 DropStoredQuery  
 限制为仅管理员可用。绕过之后，匿名用户就可以创建 WFS 存储查询，为第三环准备好“容器”。  
### 第三环 · CVE-2026-120705：把“数据”重新变成“代码”  
  
内置的 GetFeatureById  
 存储查询用简单的字符串替换把调用者提供的 ID 塞进 XML 模板。模板里有 <fes:ResourceId rid='${ID}'/>  
，引号可以闭合属性、插入新元素。更关键的是，查询文本在**保存后重载时会被二次解析**  
，而重载用的解析器启用了 XInclude  
。  
  
攻击者在第二环创建的存储查询里放入转义后的 xi:include  
，指向第一环拿到的数据目录下的 security/config.xml  
 和 users.xml  
。调用 DescribeStoredQueries  
 时，XInclude 生效，**remember-me 签名密钥和管理员口令摘要被直接读出**  
。  
### 第四环 · CVE-2026-129245：用“默认密钥”伪造管理员  
  
GeoServer 发布的数据目录自带一个固定的 remember-me 签名密钥：geoserver  
。Cookie 的签名算法用 username + ":" + expiry + ":" + passwordDigest + ":" + key  
 计算摘要  
。第三环已经同时拿到了密钥和管理员口令摘要，攻击者不需要破解口令本身，**按算法算出有效签名，就能伪造一个被 GeoServer 接受的管理员 remember-me Cookie**  
，并换取管理员 JSESSIONID  
。  
  
这一步的 CVSS 4.0 评分是 9.2，五个环节中最高  
。  
### 第五环 · CVE-2026-129242：最后一步，数据库值变成 Java 对象  
  
有了管理员会话，攻击者可以通过 REST API 上传一个 HSQLDB 数据库文件，并将其注册为 JDBC 用户组服务。JDBC 用户服务在加载用户属性时，对 OTHER  
 列使用 ResultSet.getObject()  
，HSQLDB 驱动会用**没有对象过滤器的普通 ObjectInputStream**  
 反序列化存储的字节  
。  
  
POC 使用的是 PriorityQueue  
 / BeanComparator  
 / Spring proxy / TemplatesImpl  
 gadget 链，加载一个执行命令的 translet  
。用户属性查找被触发时，命令以 GeoServer 进程的操作系统权限执行。  
  
**-修复建议-**  
  
  
  
**1.升级到 3.0.2 / 2.28.6 / 2.27.7。**  
 五个 CVE 一次性修复。容器部署需要 docker pull  
 新标签后 **重建容器**  
，仅重启不会换镜像  
。集群环境需要在每个节点设置相同的 GEOSERVER_REMEMBERME_KEY  
，因为修复版本会在每次启动时生成随机密钥，旧 remember-me Cookie 会失效  
。  
  
**2**  
.在反向代理层拦截 service  
 参数中包含 /  
 的请求，并在安全设置中从 web 过滤器链移除 rememberme  
。但这只是临时措施，**不能修复 JDBC 反序列化环节，也不能替代升级**  
。  
  
**-自查脚本-**  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/CFR489vPHAZrF8H3RibOoNkrWlIBer0PrIPRIRnyubEb7oUTiaPr4oLAHuiciaahyx8yvCI4r9qAOwnEjo1lBcicFahAZyTJdPKTs67u1tias01fA/640?wx_fmt=png&from=appmsg "")  
  
后台回复：20261126  
  
  
  
  
**End**  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/2K4EpvaFFClay5IHWNuY8h7ATvaibnex5noCLyoyaxx7l2ChafzmKwtcOibIpFtuGcicwpdlpFLbHQhyqpnhTKwYg/640?wx_fmt=png&from=appmsg "")  
  
**文章都看完了，还不点个赞**  
  
  
  
  
**回顾往期内容**  
  
[CTF丨 PHPinfo信息泄露 Writeup 从环境变量中直接读取Flag](https://mp.weixin.qq.com/s?__biz=MzI4MTE5NTY2OA==&mid=2247484559&idx=1&sn=3951ea70c14c951f7a5ac62e34cf09b9&scene=21#wechat_redirect)  
  
  
[CTF丨目录穿越 Writeup 从目录列表到Flag文件](https://mp.weixin.qq.com/s?__biz=MzI4MTE5NTY2OA==&mid=2247484552&idx=1&sn=ca50597dbcc5000a8236f39401ebd5be&scene=21#wechat_redirect)  
  
  
[CTF题目类型全解析](https://mp.weixin.qq.com/s?__biz=MzI4MTE5NTY2OA==&mid=2247484542&idx=1&sn=2d1d8745a5a3abecc9177fa7d108fd47&scene=21#wechat_redirect)  
  
  
[SQL注入靶场显错注入（一）](https://mp.weixin.qq.com/s?__biz=MzI4MTE5NTY2OA==&mid=2247483818&idx=1&sn=41542d23e0391d2c390fd3bcadf9e179&scene=21#wechat_redirect)  
  
  
[Sunny网络中间件](https://mp.weixin.qq.com/s?__biz=MzI4MTE5NTY2OA==&mid=2247483798&idx=1&sn=2c9305a9af4c7bc3d31e0228d87f0ebb&scene=21#wechat_redirect)  
  
  
[JAVA代码审计：鉴权漏洞深度分析](https://mp.weixin.qq.com/s?__biz=MzI4MTE5NTY2OA==&mid=2247483756&idx=1&sn=57e00393f90e7753835dbb3fc6cf2941&scene=21#wechat_redirect)  
  
  
[你不知道的表网、深网与暗网世界](https://mp.weixin.qq.com/s?__biz=MzI4MTE5NTY2OA==&mid=2247483745&idx=1&sn=f2d316ae89039789bd168250bac5cb14&scene=21#wechat_redirect)  
  
  
  
