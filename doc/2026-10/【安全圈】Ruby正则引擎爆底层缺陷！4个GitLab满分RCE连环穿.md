#  【安全圈】Ruby正则引擎爆底层缺陷！4个GitLab满分RCE连环穿  
安全圈
                    安全圈  安全圈   2026-10-24 11:00  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/aBHpjnrGylgOvEXHviaXu1fO2nLov9bZ055v7s8F6w1DD1I0bx2h3zaOx0Mibd5CngBwwj2nTeEbupw7xpBsx27Q/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
**关键词**  
  
  
  
漏洞  
  
  
**核心事实**  
：Ruby 官方紧急发布 4.0.9、3.4.12 与 3.3.13 版本，修复语言内置正则表达式引擎 Onigmo 中的三处底层内存损坏缺陷。此前一个月内令全球 DevOps 运维彻夜难眠的 GitLab 连续 4 个致命远程代码执行漏洞（含两个 CVSS 10.0 满分漏洞），真正元凶正是由于 Ruby 引擎底座这一致命隐患  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/sbq02iadgfyFVM4xhJ23b1eUZcw9iaogQ7eJzl4GyLYjiaeVhbIV1MDTpEia0k8tHKhxBTY7hxdJpOAdylWNyd6YmvYVcibOI3klNjlicnOY3MxEI/640?wx_fmt=other&from=appmsg "")  
  
  
## 🔍 连环引爆GitLab：从CI配置到服务器接管  
  
过去数周，开源代码托管巨头 GitLab 密集发布补丁，修复了一连串最高危害评级的 RCE 漏洞。白帽研究员 joaxcar 发现，攻击者只需提交特制的正则字符串，即可在 Ruby 编译阶段诱发内存破坏：  
- **CVE-2026-97229（CVSS 10.0）**  
：无需任何账号登录权限，即可通过特定接口注入畸形模式直接拿下服务器  
  
- **CVE-2026-110264（CVSS 10.0）**  
：危害范围覆盖 GitLab 7.0 至今的所有历史发行版，同样支持未认证远程代码执行  
  
- **前期两处 9.9 分高危缺陷**  
：允许具有低权限普通账户的恶意用户打通系统控制流  
  
此前每次 GitLab 紧急发版，背后的补救措施实质上都是临时打上特殊构建的 Ruby 定制补丁。直至本轮，Ruby 核心团队终于将引擎修复正式合入官方代码树  
## ⚠️ 不止GitLab：所有动态编译正则的应用皆在射程  
  
Ruby 官方在通告中发出明确技术警告：**该漏洞绝不仅仅局限于 GitLab**  
。任何在业务代码中利用外部用户输入动态构造正则表达式的 Ruby 应用程序，全部处于被攻击暴露面中  
  
具体高危调用包含以下几类常见模式：  
- 直接将用户输入传入 Regexp.new(params[:query])  
 或 Regexp.compile()  
  
- 调用 Regexp.union()  
 合并包含不受信任变量的规则集合  
  
- 在正则表达式字面量中进行字符串插值，如 /#{untrusted_input}/  
  
若应用仅使用研发人员在源码中硬编码编写的静态正则表达式，则不会受到该底层漏洞影响  
## ❌ 两种常见“土办法”完全无效  
  
针对很多团队试图通过常规防护手段进行缓解的尝试，官方特别明确指出了两项常见误区：  
- **设置 Regexp.timeout 毫无用处**  
：超时机制仅用于防范执行匹配阶段的 ReDoS 拒绝服务，而当前漏洞发生在**正则语法编译构建阶段**  
，根本不会触发超时中断  
  
- **限制正则字符串长度无法阻断**  
：研究证实，极短的构造模式同样能够击穿 Onigmo 的状态校验并触发内存崩溃  
  
## 🚨 扫描器盲区与紧急处置路线  
  
值得安全团队极度重视的是，**Ruby 官方暂未为自身申请独立的 CVE 编号**  
，而是建议企业参照 GitLab 的 CVE 编号进行追踪。这导致目前市面上大多数依赖 CVE 匹配的自动化资产扫描器，在扫描宿主机 Ruby 环境时会出现无报警漏报  
  
安全团队必须立即展开人工排查，落地以下处置规范：  
- **全量版本升级**  
：Ruby 4.0 必须升级至 4.0.9  
（同周发布的 4.0.8 遗漏了 CVE-2026-110264，切勿混淆），Ruby 3.4 升至 3.4.12  
，Ruby 3.3 升至 3.3.13  
  
- **淘汰老旧环境**  
：Ruby 3.2 及以下分支已于 2026 年 4 月 1 日彻底停止技术支持（EOL），官方不会提供任何补丁，建议即刻规划迁移  
  
- **代码层止血**  
：在升级完成前，业务系统应全面停用从外部不可信参数动态生成正则表达式的危险逻辑  
  
  
  
  
   END    
  
  
阅读推荐  
  
  
[【安全圈】PHP2Shell 遭百万次攻击，竟无一得手](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079470&idx=1&sn=1f84c3759d5f7bf34fcbd56949ea72a3&scene=21#wechat_redirect)  
  
  
  
[【安全圈】9.0 分 Linux 内核 RCE，被一封邮件捅穿](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079470&idx=2&sn=f760b577c0f307d90d16107d46ad7e9a&scene=21#wechat_redirect)  
  
  
  
[【安全圈】1177 万条！卡普空确认账号库被偷](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079470&idx=3&sn=4ed4671e35da838dae890d616c227665&scene=21#wechat_redirect)  
  
  
  
[【安全圈】老乡鸡小程序又崩了](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079459&idx=1&sn=9bbd43e57a372693618b7ed16c3c2af4&scene=21#wechat_redirect)  
  
  
  
[【安全圈】单连接打崩服务器！HTTP3曝4000倍内存放大漏洞](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079445&idx=1&sn=faa0a86fd9d2ee022af099b2286cf0d2&scene=21#wechat_redirect)  
  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCEft6M27yliapIdNjlcdMaZ4UR4XxnQprGlCg8NH2Hz5Oib5aPIOiaqUicDQ/640?wx_fmt=gif "")  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCEDQIyPYpjfp0XDaaKjeaU6YdFae1iagIvFmFb4djeiahnUy2jBnxkMbaw/640?wx_fmt=png "")  
  
**安全圈**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCEft6M27yliapIdNjlcdMaZ4UR4XxnQprGlCg8NH2Hz5Oib5aPIOiaqUicDQ/640?wx_fmt=gif "")  
  
  
←扫码关注我们  
  
**网罗圈内热点 专注网络安全**  
  
**实时资讯一手掌握！**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCE3vpzhuku5s1qibibQjHnY68iciaIGB4zYw1Zbl05GQ3H4hadeLdBpQ9wEA/640?wx_fmt=gif "")  
  
**好看你就分享 有用就点个赞**  
  
**支持「****安全圈」就点个三连吧！**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCE3vpzhuku5s1qibibQjHnY68iciaIGB4zYw1Zbl05GQ3H4hadeLdBpQ9wEA/640?wx_fmt=gif "")  
  
  
  
  
