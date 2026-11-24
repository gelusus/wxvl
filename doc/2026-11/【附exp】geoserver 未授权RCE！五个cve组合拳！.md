#  【附exp】geoserver 未授权RCE！五个cve组合拳！  
原创 棉花糖糖糖
                    棉花糖糖糖  棉花糖fans   2026-11-24 16:00  
  
## 漏洞介绍（长亭）：  
  
GeoServer 多个版本存在未认证远程命令执行漏洞。攻击者可通过默认开放的 WFS 服务，利用超长存储查询 ID 泄露数据目录路径，通过 x/WFS 服务名绕过操作鉴权，并借助二次 XML 解析和 XInclude 读取 remember-me 密钥及管理员密码摘要。攻击者据此伪造管理员会话，再创建指向恶意 HSQLDB 数据库的 JDBC 用户服务，利用 OTHER 类型字段触发 Java 反序列化，最终以 GeoServer 进程权限执行系统命令。受影响版本包括 GeoServer 2.27.6 及更早版本、2.28.0–2.28.5、3.0.0–3.0.1，建议升级至 2.27.7、2.28.6 或 3.0.2 及以上版本。  
## 影响范围：  
  
GeoServer <= 2.27.6  GeoServer 2.28.0 - 2.28.5  GeoServer 3.0.0 - 3.0.1  
## 漏洞链路：flowchart LR    A["超长 storedQuery_id 泄露数据目录<br/>CVE-2026-129244"]    B["x/WFS 绕过操作鉴权<br/>CVE-2026-129243"]    C["XInclude 读取 key 和密码摘要<br/>CVE-2026-120705"]    D["伪造 remember-me 管理员会话<br/>CVE-2026-129245"]    E["HSQLDB 反序列化<br/>CVE-2026-129242"]    F["以 GeoServer 进程权限执行命令"]    A --> B --> C --> D --> E --> F  
### 1. WFS 异常泄露数据目录（CVE-2026-129244）  
  
GetFeature  
 的 storedQuery_id  
 进入 StoredQueryProvider.getStoredQuery()  
。toFilename()  
 只清理特殊字符并添加 .xml  
，没有限制长度。  
  
超长 ID 使 Files.readAttributes()  
 抛出文件系统异常。getStoredQuery()  
 保留底层异常继续抛出，OWS 将异常链写入公开响应。 响应中的绝对路径去掉 /wfs/query/<ID>.xml  
 后就是数据目录。  
### 2. 服务名不一致绕过鉴权（CVE-2026-129243）  
  
services.properties  
 只允许管理员执行 wfs.CreateStoredQuery  
。  
  
Dispatcher.findService()  
 将 x/WFS  
 截取为 WFS  
，找到真实 WFS 服务。 请求对象的 service  
 仍为 x/WFS  
。OperationSecurityCallback  
 使用原值匹配规则， 无法命中 wfs.CreateStoredQuery  
，因此匿名请求可以写入 stored query。  
### 3. 二次解析触发 XInclude 文件读取（CVE-2026-120705）  
  
stored query 包含转义后的 xi:include  
 文本。请求 XML 解码后，CopyingHandler.characters()  
 将 <  
 等字符直接写入缓冲区。  
  
QueryExpressionTextTypeBinding.encode()  
 使用 SAX 再次解析查询文本， 文本因此变为 xi:include  
 元素并写入磁盘。  
  
DescribeStoredQueries  
 调用 StoredQueryProvider.parseStoredQuery()  
 重新加载文件。 parser 开启 XInclude，实体解析器允许本地 file:  
，响应返回：  
- security/config.xml  
 中的 remember-me key。  
  
- security/usergroup/default/users.xml  
 中的 admin 密码摘要。  
  
### 4. 伪造 remember-me 会话（CVE-2026-129245）  
  
Spring Security 按以下内容计算 remember-me 签名：  
```
SHA256(username:expiry:password-digest:key)

```  
  
用户名为 admin@default  
。攻击者使用泄露的密码摘要和 key 计算签名， 将 username、expiry、算法、签名编码为 remember-me cookie。 认证过滤器接受 cookie 并创建管理员 JSESSIONID  
，不需要破解密码。  
### 5. JDBC 反序列化执行命令（CVE-2026-129242）  
  
管理员 REST API 将 HSQLDB 文件写入数据目录，并创建 JDBC 用户服务。  
  
查询用户组时，JDBCUserGroupService.getUserByUsername()  
 对 user_props.propvalue  
调用 ResultSet.getObject()  
。该列类型为 OTHER  
，HSQLDB 调用JavaObjectData.getObject()  
 和 InOutUtil.deserialize()  
，最终执行ObjectInputStream.readObject()  
。  
  
反序列化对象使用以下调用链：  
```
PriorityQueue.readObject
  -> BeanComparator.compare
  -> TemplatesImpl.getOutputProperties
  -> CommandTranslet.<init>
  -> ProcessBuilder("/bin/sh", "-c", command)

```  
  
命令以 GeoServer 进程权限运行。  
## exp使用条件  
- Linux 目标，可访问 WFS 与 REST。  
  
- default  
 用户服务中的 admin  
 已启用。  
  
exp仓库：  
```
https://gitee.com/mhtsec/geoserver-preauth-rce
```  
## 使用方法  
```
python3 -m pip install requests
python3 exp.py -u http://TARGET/geoserver -c id

```  
  
执行系统命令：  
```
python3 exp.py -u http://TARGET/geoserver -c 'id; whoami'

```  
  
保存脱敏请求响应和阶段结果：  
```
python3 exp.py --no-color --evidence-dir evidence \
  -u http://TARGET/geoserver -c id

```  
  
自签 HTTPS 证书使用 -k  
。HTTP 超时通过 --timeout  
 设置。  
```
python3 exp.py -k --timeout 60 -u https://TARGET/geoserver -c id

```  
  
-c  
 支持最多 512 字节可打印 ASCII。进度写入 stderr，命令输出写入 stdout。 脚本在结束后删除临时 stored query、JDBC 用户服务和 staging 文件。 若无法确认数据库已关闭，脚本保留文件并输出提示，不要并发攻击同一目标。  
## 复现：  
  
已在无境靶场  
复现该漏洞，成功执行命令。  
  
在线靶场地址：  
https://bdziyi.com/ulab/lab.html?page=target-detail&id=307  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/IicMcDFtTOlP3bVBwy0NDswtTvNWgxbKggPN4G9AU0wdxdy9KNTAW5icvzX3v1CibF1xib00nUUWbI9LibwO8PuIywDqq35LrArzcIvWlYmKKiarU/640?wx_fmt=png&from=appmsg "")  
  
image-20261122133  
  
## 广告时间：  
  
如果您对内网靶场、SRC靶场、WEB靶场、应急响应靶场、网安方案、资料文档、各种各样的网安工具（SRC资产监控平台、ICP备案批量查询、md5等hash解密）等等超级多实用的在线功能、教程资源感兴趣，欢迎看看我的网站广告，包您心动：[棉花糖会员站介绍(26年4月26日版本) ：在线内网靶场、网安资料方案、在线工具全能资源站](https://mp.weixin.qq.com/s?__biz=MzkyOTQzNjIwNw==&mid=2247493656&idx=1&sn=ef2aad19a122c739055604331f93f34c&scene=21#wechat_redirect)  
  
  
网络安全交流群请在公众号菜单栏点击找棉花糖，加棉花糖好友后发送“棉花糖fans”，邀请您进群，CISP全系列、国内外网安考证也可以在公众号菜单栏点击找棉花糖，微信联系哦，爱你😘，下篇文章再见  
  
