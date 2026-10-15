#  【紧急漏洞预警】GitLab整数溢出导致RCE漏洞(POC已公开)  
YGnight
                    YGnight  night安全   2026-10-15 00:15  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/LAQpgdWQSctVS8Ps0NsFTqMiasz8uDibcvoib0spt17ORFYGT7Lk8y0JElHpWukiczXboIicO8mrOUut0DwfE4PvMwpuReeibS0yQHqN8Tictdm4Z4/640?wx_fmt=png&from=appmsg "")  
  
## 一、漏洞描述  
  
问题出在 GitLab 处理 CI 配置里正则表达式的那段代码。  
  
GitLab CE 与 EE 的 CI/CD 正则编译器存在整数溢出，编号 CVE-2026-97229，CWE 归类 190。  
  
CVSS 打分 10.0，向量 CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H，由 GitLab 自己给出，NVD 按 Secondary 来源收录。  
  
受影响区间分三条线，19.2 到 19.2.8 之前、19.3 到 19.3.5 之前、19.4 到 19.4.2 之前，两个版本都在里面。  
  
GitLab.com 已经跑上修补后的版本，Dedicated 客户无需操作。  
  
触发入口是配置校验接口，GET /api/v4/projects/:id/ci/lint，跟着一个 content_ref 参数指向某个分支。  
  
公开情报把它描述成无需认证。实测那一方多补了一层，攻击者先要能往公开项目推分支，普通账号就够，此后那条触发请求不带任何凭据。  
<table><tbody><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">CVE 编号</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">CVE-2026-97229</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">漏洞类型</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">整数溢出导致远程代码执行，CWE-190</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">影响组件</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">Gitlab::Ci::Build::Rules::Rule::Clause 的正则编译与校验，rules 下 exists.regexp 与 changes.regexp 共用</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">受影响版本</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(220, 38, 38);font-weight: bold;"><section><span leaf="">19.2.0 ~ 19.2.7、19.3.0 ~ 19.3.4、19.4.0 ~ 19.4.1，CE 与 EE</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">修复版本</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(146, 64, 14);font-weight: bold;"><section><span leaf="">19.2.8、19.3.5、19.4.2</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">触发入口</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">GET /api/v4/projects/:id/ci/lint，content_ref 指定分支</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">触发前提</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">能往公开项目推分支，普通账号即可；项目可被未登录用户读取</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">实测情况</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">19.4.1 CE amd64 官方镜像，10 次全部成功，单次约 12 秒，实例需约 9 GiB 可用内存</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">落地身份</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">git 账号，实测回显 uid=998</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">CVSS</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">10.0，CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H，GitLab 给出，NVD 记为 Secondary</span></section></td></tr><tr><th style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);text-align: left;font-weight: bold;color: rgb(26, 26, 26);background-color: rgb(243, 244, 246);white-space: nowrap;"><section><span leaf="">在野利用</span></section></th><td style="padding: 10px 12px;border: 1px solid rgb(209, 213, 219);color: rgb(51, 51, 51);"><section><span leaf="">公开情报中暂无已观测到的在野利用记录，CISA KEV 未收录，验证脚本已公开</span></section></td></tr></tbody></table>## 二、漏洞原理  
  
入口在配置文件里。  
  
.gitlab-ci.yml 的 include 规则支持 exists 下面写 regexp。  
  
写进去的正则会被拿去和仓库里的文件路径逐个匹配，路径取自 repository.ls_files，也就是这个分支上的全部文件名。  
  
写正则的人和提供文件名的人，是同一个。  
  
// lib/gitlab/ci/build/rules/rule/clause/exists.rb v19.4.1-ee  
def regexp_match?(context)  
    matcher = regexp_matcher(context)  
    matcher.validate_pattern_length!  
    matcher.match?(worktree_paths(context, false))  
end  
def regexp_matcher(context)  
    Rules::Rule::Clause::RegexpMatcher.new(  
      raw_pattern: @regexp,  
      expanded_pattern: expand_value_nested(@regexp, context),  
      max_comparisons: MAX_PATTERN_COMPARISONS,  
      log_scope: 'rules:exists',  
      project_id: context.project&.id  
    )  
end  
  
正则交给 RegexpMatcher，编译动作落在 Clause.compile_regexp。  
  
GitLab 在那里放了两道体检。  
  
第一道查量词，上限 1000，扫描用的正则认花括号里只有数字或逗号的写法。  
  
第二道查体积，用 ObjectSpace.memsize_of 量编译出来的东西，上限 1 MB。字符长度另有一道 255 的坎，在变量展开之后重算一次。  
  
// lib/gitlab/ci/build/rules/rule/clause.rb v19.4.1-ee  
REGEXP_MAX_LENGTH = 255  
REGEXP_MAX_COMPILED_BYTES = 1.megabyte  
REPEAT_MAX = 1_000  
REPEAT_BOUND = /(?<!\\)(?:\\\\)*\{\s*(\d+)\s*(?:,\s*(\d+)?\s*)?\}/  
def self.compile_regexp(pattern, timeout: nil)  
    reject_large_repeats!(pattern) // 第一道，扫量词  
    alternation_free = pattern.gsub(ESCAPE_OR_ALTERNATION) { |t| t == '|' ? '\|' : t }  
    probe = Regexp.new(alternation_free)  
    check_compiled_size!(probe) // 第二道，先量副本  
    check_compiled_size!(Regexp.new(pattern, timeout: timeout))  
end  
def self.check_compiled_size!(regexp)  
    compiled_bytes = ObjectSpace.memsize_of(regexp)  
    return regexp if compiled_bytes <= REGEXP_MAX_COMPILED_BYTES  
    raise RegexpError, "compiles to #{compiled_bytes} bytes, over the limit"  
end  
  
第一道，靠一条注释。  
  
量词在配置里写成 {5(?#)7805}。  
  
Onigmo 把 (?#...) 当注释跳过，读到的量词是 57805。  
  
GitLab 那道扫描正则认的是花括号之间只有数字或逗号的写法，遇到左括号就匹配不上，整段被跳过。  
  
体检看到的是一个 5，引擎跑的是 57805。  
  
// 写进配置的量词  
{5(?#)7805}  
// REPEAT_BOUND 匹配不到，数字 5 后面跟的是左括号  
// Onigmo 跳过注释后拿到的量词  
{57805}  
  
第二道，靠编码。  
  
体检量的是先编出来的那份。先编一个去掉竖线的副本当探针，量完再编真身。  
  
两份模式串都是纯 ASCII，Ruby 按单字节编码编译，体积压在 1 MB 以下，检查放行。  
  
真身去匹配路径时才起变化。路径里带 UTF-8 字符，Ruby 要按 UTF-8 重新编一遍，这一遍的体积，体检够不着。  
  
溢出就发生在这一遍。  
  
每轮展开的字节码约 37,151 字节，乘上 57,805 次重复得 2,147,513,555，比 32 位有符号整数的上限多出 29,908。  
  
引擎生成跳转指令时偏移量绕回，多出来的字节落进字符类的位图。那块区域本来记的是哪些字符算命中，现在被当成了指令。  
  
// poc.py 重复次数按整数上限反算  
TARGET_BYTECODE_SIZE = 37_151  
REPEAT_COUNT = (0x7FFFFFFF + TARGET_BYTECODE_SIZE - 1) // TARGET_BYTECODE_SIZE // 57805  
// 37151 * 57805 = 2,147,513,555 INT_MAX = 2,147,483,647  
// 多出的 29,908 让生成的跳转偏移绕回  
digits = str(REPEAT_COUNT)  
pattern = f"\\A(?>{prefix}(?:{target}){{{digits[0]}(?#){digits[1:]}}}|)\\b\\B"  
  
后面是反着算的活。  
  
想执行的指令先拆成字节，按位摊成字符码点，再写成字符类，方括号里那串字符就是指令本身。  
  
绕回的偏移把执行流带到这些字节上，正则引擎开始跑攻击者排好的那段。  
  
这段指令把引擎的栈挪到路径字符串上，路径本来就是攻击者自己的文件名。接着复用解释器里已有的指针绕开地址随机化，最后跳到 libruby 里的 system 调用点。  
  
两个位置在脚本里写成常量，LIBRUBY_SYSTEM_GADGET 记 0x31E809，基准是 LIBRUBY_FINISH_CODE。  
  
文件名里塞的是 28 字节的命令，curl 拉一段脚本交给 sh，真正的命令再从监听端取回，标准输出和标准错误原路送回。  
  
靶子限定得比较死。给出的偏移只对应 19.4.1 CE amd64 这一套 Ruby 布局，换个版本要重新算。  
  
同一份正则和路径在 19.4.2 上返回 false，没有回连。  
## 三、修复建议  
  
三条线各自升到 19.4.2、19.3.5、19.2.8。  
## 近期值得重点关注的漏洞预警  
  
[【紧急漏洞预警】nacos爆出权限绕过漏洞，可以创建管理员账户，经过分析有点出处（已复现，exp脚本已编写）](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486343&idx=1&sn=b67c4eb02c03af9c3b206b1241ff7e9c&scene=21#wechat_redirect)  
  
  
[【漏洞预警】AnyDesk Linux存在堆缓冲区溢出漏洞(POC已公开)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486500&idx=1&sn=fb8fdf3dd59e252bf04360dfabac5a3c&scene=21#wechat_redirect)  
  
  
[【漏洞预警】macOS爆出路径穿越漏洞CVE-2026-84568(POC已公开)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486490&idx=1&sn=1009bcc6cb5416474ffa0a4f499bf67a&scene=21#wechat_redirect)  
  
  
[PostgreSQL爆出远程代码执行漏洞(POC已公开)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486480&idx=1&sn=31416416494b366dc85cf4d20e0ab3ee&scene=21#wechat_redirect)  
  
  
[【漏洞预警】Atlassian八款产品爆任意文件读取漏洞(附POC)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486475&idx=1&sn=3c669c88388236c2af996ceb9782e928&scene=21#wechat_redirect)  
  
  
[【漏洞预警】Joomla视频画廊扩展存在SQL注入漏洞(POC已公开)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486469&idx=1&sn=6146142b1b0095532a2d87dcbcf738df&scene=21#wechat_redirect)  
  
  
[【漏洞预警】Roundcube网页邮件客户端存在SQL注入漏洞(附POC)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486464&idx=1&sn=93e0df392b292c72408f2c33cfb868ca&scene=21#wechat_redirect)  
  
  
[WordPress页面模板存在文件包含漏洞(POC已公开)CVE-2026-87902](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486459&idx=1&sn=5dc658a83a772526773f3e87b1ac8ad0&scene=21#wechat_redirect)  
  
  
[WordPress插件Forminator未认证短代码执行漏洞(poc已公开)CVE-2026-92229](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486454&idx=1&sn=64894c978fbfac86c914945580147bd9&scene=21#wechat_redirect)  
  
  
[WordPress插件Authorizer存在权限提升漏洞(POC已公开)CVE-2026-81294](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486449&idx=1&sn=20a4f9a399ae34e07a27c7b6d2049e45&scene=21#wechat_redirect)  
  
  
[WordPress插件MStoreAPI存在认证绕过漏洞(POC已公开)CVE-2026-13447](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486444&idx=1&sn=a22077edeab35852d6791825ebba8b10&scene=21#wechat_redirect)  
  
  
[CakePHP爆出SQL注入漏洞(POC已公开)CVE-2026-77635](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486439&idx=1&sn=5030655c1d40e179369ab972ddd5dfc9&scene=21#wechat_redirect)  
  
  
[Node.js模块proxy-addr爆出严重漏洞CVE-2026-90711](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486429&idx=1&sn=fc1772f8703bdd4128ac829c1ae8b371&scene=21#wechat_redirect)  
  
  
[Langflow爆出代码执行漏洞CVE-2026-17633  (POC已公开)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486424&idx=1&sn=0fec55e718cb9fb60eb91ca24307fd1a&scene=21#wechat_redirect)  
  
  
[Apache CVE-2026-42536，补它只用一行，洞龄十四年](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486419&idx=1&sn=393b6baeea4f624ffa8440058c0d324e&scene=21#wechat_redirect)  
  
  
[【紧急】搜狗输入法爆出远程代码执行漏洞CVE-2026-51990  ，已被在野利用](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486413&idx=1&sn=0b2c6a470a165f288873fa6bace1d77b&scene=21#wechat_redirect)  
  
  
[【紧急】无需登录就能获取 GitLab 的任意文件数据(附POC)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486408&idx=1&sn=06f47231a649e03866f9e9524b5469a4&scene=21#wechat_redirect)  
  
  
[【紧急】JumpServer堡垒机被公布Access Key泄露漏洞，可以实现提权](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486403&idx=1&sn=8e4dc6a684e95b9dc5871b1aca487a57&scene=21#wechat_redirect)  
  
  
[Spring Cloud Azure公布严重漏洞 无需任何权限即可取得登录会话](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486398&idx=1&sn=c64f6f82f75da584d032b99509ffc726&scene=21#wechat_redirect)  
  
  
[【漏洞POC已公开】Linux RDS 爆出提权漏洞 一块内存放两次，root就到手了](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486393&idx=1&sn=629bab1cc81fbee25e8e6dd558ee155b&scene=21#wechat_redirect)  
  
  
[Tenda光猫爆出严重漏洞，目前无补丁(POC已公布](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486388&idx=1&sn=eb210feb27b85439b53b10e595f7667e&scene=21#wechat_redirect)  
  
  
[13 行补丁，堵住Linux能拿 root 的内核洞(EXP已公开)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486383&idx=1&sn=a87059fbbca283a7bd64ad78280ed00a&scene=21#wechat_redirect)  
  
  
[48 字节，换走一台 Windows 的 SYSTEM(CVE-2026-62735漏洞POC已公开)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486378&idx=1&sn=f9e17fe32b6a3c67249fb4f037a3f1c6&scene=21#wechat_redirect)  
  
  
[【漏洞预警】linux删个网桥就能提权root？目前没有补丁(POC已公开)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486373&idx=1&sn=5f8bb9b0ff7c9fad75a8071c026b9f93&scene=21#wechat_redirect)  
  
  
[【漏洞预警】Linux三年前补上的洞，又一次被挖开了](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486368&idx=1&sn=337dc5fa148d0b24e7dbd352a71e184f&scene=21#wechat_redirect)  
  
  
[【情报】支付宝8.2亿数据泄露？给同事说一声，别自己吓自己](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486362&idx=1&sn=927fdf3d81c79a2ff8871086f0e78a8b&scene=21#wechat_redirect)  
  
  
[【漏洞预警】自己注册个号，就能搬空别人的 Magento 订单和支付卡（POC已公开）](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486363&idx=1&sn=344a1f2e1a64b3b3be1509610edd2a0d&scene=21#wechat_redirect)  
  
  
[【漏洞预警】Next.js 爆出新漏洞，不用登录实现命令执行(POC已公开)](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486348&idx=1&sn=350cb257c3bfc3372e2dd50f2eb13738&scene=21#wechat_redirect)  
  
  
[【漏洞补丁绕过预警】Redis CVE-2026-23479变种高危漏洞，EXP已公开](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486299&idx=1&sn=7d1d8a95dad56cb021c2aa4ff549ee62&scene=21#wechat_redirect)  
  
  
[【漏洞补丁绕过】超 30 万 WordPress 站点受影响，Forminator 文件上传漏洞（附POC）](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486304&idx=1&sn=a22d6632366218b72fd68b3a5cb91557&scene=21#wechat_redirect)  
  
  
[【漏洞预警】Exchange爆出RCE漏洞，利用Exchange 帮另一台 Exchange 开门（POC已公开）](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486323&idx=1&sn=124dd54f863f64371f18c668d52e1fab&scene=21#wechat_redirect)  
  
  
[【漏洞预警】不用登录，一个请求就能让 GeoServer 乖乖执行命令（POC已公开）](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486318&idx=1&sn=b478f775998b89579c53d4e0fbdc8c6e&scene=21#wechat_redirect)  
  
  
[【漏洞预警】Keycloak不登录就能接管任意账户，双重验证也拦不住](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486309&idx=1&sn=d5c1a812f00448d397d3e3a8e78cdd45&scene=21#wechat_redirect)  
  
  
[【漏洞预警】 8月WebLogic公布了9个漏洞，有一个需要重点关注](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486294&idx=1&sn=eb26bc5f8fcafc9ab402b77acdb1b2a4&scene=21#wechat_redirect)  
  
  
[【漏洞预警】近期MongoDB 公布了5个漏洞，最严重的一个可以不认证、远程就能拿下你电脑](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486289&idx=1&sn=10023bdd570d9f011eef242a23f83bb6&scene=21#wechat_redirect)  
  
  
[【0DAY漏洞】 全球5G手机芯片厂商之一‌的紫光展锐SOC存在RCE漏洞，影响数十家手机厂商，还未发布补丁](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486279&idx=1&sn=5e1452f6984bf3d4b4d35a2240676df5&scene=21#wechat_redirect)  
  
  
[【漏洞告警】安卓 14 到 16 全中招，通讯录 SQL 注入漏洞正在抄家（POC已公开）](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486274&idx=1&sn=4bacf56f2045fca36b7b3de4be3ae591&scene=21#wechat_redirect)  
  
  
[【漏洞预警】紧急排查，SMB服务再曝新漏洞，利用脚本已公开](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486251&idx=1&sn=633ed2b7d916bcf98404a7b3a9a8c4b4&scene=21#wechat_redirect)  
  
  
[【漏洞预警】不用输密码，对方已经接管你的Mac：官方补丁公布macOS存在未鉴权RCE漏洞](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486246&idx=1&sn=eaf0c51686e92e89826098fcf51fb959&scene=21#wechat_redirect)  
  
  
[【0day漏洞预警】MariaDB 13 爆 9.8 分 RCE 链：最低 USAGE 权限就能远程执行命令](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486225&idx=1&sn=bd13754fd9378e01a088cf996fa5b557&scene=21#wechat_redirect)  
  
  
[VMware 爆 5 个洞，两个 9.8 不用密码就能打穿 vCente](https://mp.weixin.qq.com/s?__biz=MzU5MTc1NTE0Ng==&mid=2247486204&idx=1&sn=2590e283b0981541da74647ddc817282&scene=21#wechat_redirect)  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/LAQpgdWQSctTWicwicwovsVtMLAjYPfEB9lMRmJJiaIozWIuicwrbneXZ3pGly4lRNAd1WrP6AKYA925Iz4c1EbnMLCOqWzkCmdCLXDQibsG4VQw/640?wx_fmt=png&from=appmsg "")  
  
