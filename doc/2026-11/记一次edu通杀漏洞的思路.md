#  记一次edu通杀漏洞的思路  
原创 小白安全圈
                    小白安全圈  小白安全圈   2026-11-03 08:16  
  
正常访问一个网站，通过熊猫头findsomething查看接口情况，发现很多接口，将接口全部提取出来保存后面留着拼接路径访问。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ChhM9MJ0Tz8KZRws9icP6V2zr3IZxUsR5uvGuNibvdwfxFmjM77rAZX2Yf8W8YEZ2f1ZFqAcC18gCvfRMRK0cnlqhw6zibOjePqliam0ZQEkr1Y/640?wx_fmt=png&from=appmsg "")  
  
这个时候发现接口/sys/file/download，看到这个就可以想到可以测试的漏洞有，ssrf，任意文件读取漏洞，目录遍历漏洞等等，逐步分析前端js代码文件寻找接口。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ChhM9MJ0TzicpZwVdrrIOk113Cr3Cw9xqwwn5JEUrHKdLofWAUb9VlA173GIgoZZvphs1fh0NtexcfQTnsibr6d8WbicqIL43BpK0wZh2ycuBY/640?wx_fmt=png&from=appmsg "")  
  
然后通过前端js代码分析，发现s代表的是url，get请求方法，以及t参数是path，那么实际上请求的url地址为https://xxxx/baseurl/comm  
on/file/download?path=  
  
说明，baseurl为  
axios 里的基础地址（base URL），相当于给所有请求 URL 加一个公共前缀。最终请求地址 = baseURL + 路径。  
  
接下来就是常规手法了，测试任意文件读取，读取发现有.env文件泄露  
  
https://xxxx/baseurl/comm  
on/file/download?path=../../../../../.env  
  
.env 是前端/后端项目用来存放环境变量的配置文件（如 API 地址、密钥、数据库密码、第三方 AppKey 等）。.env 文件泄露指这个本应只在服务器/本地存在、绝不该被浏览器下载的文件，被攻击者通过 URL 直接访问到了，从而暴露其中的敏感配置。  
  
最终在文件里面发  
现  
管理员的jwt密钥、redis密码、网站登录默认密码、appkey。  
  
然后导致可以通过账号和密码  
登录  
管理员  
网  
站后台  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ChhM9MJ0TzicOI1N8zzljboWZtmBW4wjsAibAmU4icZ1MS5GP5SO3YiacRiaVbfvzibqOLg2n70oxIr3K58qWAc6iccA88icP6N1NgFBgOJiayuL9b7c/640?wx_fmt=png&from=appmsg "")  
  
利用2：通过伪造管理员jwt获取其他用户信息，前端泄露算法，当然也可以直接登录管理员账号获取凭证。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ChhM9MJ0Tz83IsXxUVSmJ3UOFUuoTkiamYtn5D3ZfnG2BxHWylkBuhUqpe6DdXQUsUgwHibK4ddQ53cqV6M9tkb0WRnlDic9s8WgelmwFuzUFo/640?wx_fmt=png&from=appmsg "")  
  
这个时候回过头去拿着管理员的凭证权限去查看敏感接口  
，获取  
含工号、姓名、机构、性别、认证状态。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ChhM9MJ0Tz8YXpfldaPm4iac2OTOadETgJich4SxtB1JvXeUVzxhCxicrJ6YibUAcGjLfVDUwZmakhXdJicn3iawJvETOXLBjqCSnZZ7cicNveyPho/640?wx_fmt=png&from=appmsg "")  
  
利用3：根据上面的同理找到另外一个接口任意文件读取常见源码数据。  
  
app/Sxxxxxe/xxxuth/xxxuthxxx.php     
密钥派生规则  
  
app/Mxxxels/Bxxxxse/Gxxxs.php      
AES key/iv   
  
这个时候发现密钥生成的逻辑为  
 硬编码前缀 + 可预测时间戳。然后通过ai构造一个免认证登录的password。成功登录任意师生账号获取token凭证。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ChhM9MJ0Tzic0xobRP6RtdjDMFOvJsGbnKMeuNFZXII0lsYvcOkq9GFz4naBRLOicVGkIRtr3YgC9f3VncIDByOClHD2wwkaDxZHGFSdic72U0/640?wx_fmt=png&from=appmsg "")  
  
  
利用4：通过获取师生凭证测试看看能不能越权，发现普通的老师可以直接查看管理员的接口，实现垂直越权。接口也是前端获取得的，  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ChhM9MJ0TzibEswPDsic3YYkpgd3jibT34FuvxIx8ueHMibeuibibCn5y8hsLibdmZ2mq6Db1dMoCVqaiblqzgIicgAj8JQpo4avlic6BFBK0dgaF0t4U/640?wx_fmt=png&from=appmsg "")  
  
最终也是拿下了30+rank  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ChhM9MJ0Tz9bRIMNFibpUmRtRyXf9dEm3Jm40gzHX9MtLicdkenicgZYdSwr2HwydWn7icpb2Rcwql8E16y2ibJblibMxCbOa7MfDv19Y7OW9RVCQ/640?wx_fmt=png&from=appmsg "")  
  
