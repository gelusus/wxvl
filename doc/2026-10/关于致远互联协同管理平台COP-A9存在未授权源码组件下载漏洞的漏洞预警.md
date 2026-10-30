#  关于致远互联协同管理平台COP-A9存在未授权源码组件下载漏洞的漏洞预警  
原创 Vlan911
                        Vlan911  我不懂安全   2026-10-30 00:39  
  
**漏洞名称：**  
致远互联协同管理平台COP-A9存在源码泄露漏洞  
  
**漏洞描述：**  
致远互联协同管理平台COP-A9存在源码泄露漏洞，远程攻击者在无需授权认证的情况下即可下载平台内部 Maven 构件仓库的任意已知坐标构件  
  
**风险等级：中**  
  
**利用难度：**  
极低，需要知道受影响系统组件名称、组件对应版本号  
  
**漏洞影响版本：**  
软件版本：V5.3.0  受影响组件版本：boot-starter-file <= 5.*.200  
  
**漏洞细节：**  
  
平台的文件下载接口 /xxx/xxx/xxx/download 对 storageKey 参数采用 fail-open（失败即放行）的鉴权判定，未认证攻击者仅凭一条 URL 即可下载平台内部 Maven 构件仓库中的全部已知坐标构件，进而获取平台服务端全部组件的完整实现（Java 字节码可近乎无损还原为源码）、组件精确版本号与构建时间戳，以及打包在构件内的数据库建表脚本（库表结构）  
  
**漏洞复现：**  
  
复现要点：需要知道组件包版本  
  
获取组件包大致版本很简单，系统存在多处未授权访问接口，都能泄露系统组件的版本  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Pms46XiaGB2yTTPibrlo1mMO6oOiaU5N86NOypicqXCPElfO2TBlRDEgvjHOhb2bGr6XbUyYkibxaJmsianDNp4GWYmJ8xxDibuvjmLqicSnXbnqia0A/640?wx_fmt=png&from=appmsg "")  
  
直接访问即可下载jar包  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Pms46XiaGB2wr8CiamSuibOC2F1vFokV9rZwLUGdxmC0dCYgUaO2V9Bf6jVALmGo3cUmJnEMwtabzwSJVMxRt7Xsy1gez3AEozWKLN3Fvb8FEI/640?wx_fmt=png&from=appmsg "")  
  
下载pom  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Pms46XiaGB2zg2xs5CeV6YFvkJ8CWyiaWJXCoNqQmTd8wQeYic9yj7Yvg09Dleu2Te94cYOyaTjB9F1k8lS8eFzfaticMbbL0gibAJ6gR4R8M3Ww/640?wx_fmt=png&from=appmsg "")  
  
由于产品的后台实际上都是由后台组件、核心组件撑起来的，所以实际上下载组件包就相当于拿到了源代码，至于如何拿到那么多组件的组件名称、组件版本，实际上是有方法的，这里不具体透露  
  
  
**防护建议：**  
  
致远官方已经对未授权访问漏洞、源码下载漏洞进行了补丁更新，请联系维保进行补丁升级  
  
  
  
  
