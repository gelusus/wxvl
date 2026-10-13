#  AI在漏洞挖掘中的辅助作用  
 Z2O安全攻防   2026-10-13 12:40  
  
## 引言  
  
我理解的AI对漏洞挖掘的价值，主要是降低重复性工作的时间成本。漏洞挖掘通常经历攻击面测绘、漏洞点验证和危害链评估三个阶段。前端逆向、接口归类、用例整理、跨接口比对和证据整理会占用大量时间；AI 适合辅助处理这些信息密集型工作，使测试者把精力放在业务判断、风险边界和人工复核上。  
  
本文将 AI 的作用限定为三个层次：第一，理解和整理已获得的信息，例如前端代码、接口文档和请求响应；第二，根据已有线索提出可验证的测试假设，并协助组织用例；第三，归纳证据、记录已测状态和发现的关联。AI 输出的是线索、结构和候选用例，不是漏洞结论。任何结论都应在授权范围内，由测试者结合身份、对象归属、响应内容和实际影响复核。  
  
后文案例均来自原始记录。为避免把报告写成可直接用于未授权操作的说明，本文保留原有证据截图和技术事实，同时对高风险动作以“经授权验证”“风险影响”的方式概述。目标域名和敏感数据沿用原文的脱敏形式。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4kp359AaRqL6qx0AnnaoPJ6vAYAmxV3ic8e9WjKHwVVj5sk1Oib2PFWZTBicyfn8DekZEjwRbtq68RDg9XTrjLa402tFHzaVpjBNI/640?wx_fmt=png&from=appmsg "")  
  
❞## 一  AI 辅助漏洞挖掘的工作方式  
### 1 前端 JavaScript 解析与接口梳理  
  
现代 Web 系统常采用前后端分离架构，路由、接口调用、参数拼装、鉴权流程和部分加解密逻辑往往分散在压缩或混淆后的 JavaScript 文件中。前端代码不能保证覆盖全部后端能力，但它通常能提供高价值的入口线索，例如 API 前缀、接口路径、参数名、角色字段和请求拦截器。  
  
AI 可以帮助按模块阅读代码片段，提取接口、参数、调用位置与疑似权限判断，并整理为接口台账。台账应至少记录：接口路径、请求方法、身份要求、对象标识、敏感字段、来源文件、测试状态和证据编号。分段处理可以降低长文本遗漏，但仍应以搜索结果、浏览器网络请求和实际响应交叉验证。  
### 2 认证异常与权限验证的组织  
  
401、403、重定向或网关拦截并不直接等同于“存在绕过”。它们可能来自身份缺失、令牌过期、角色不足、来源限制、WAF 规则或业务状态。AI 的作用是根据请求上下文整理可能原因和最小化验证步骤，协助测试者区分认证、功能级授权和对象级授权问题。  
  
对象级授权关注“当前身份是否有权访问这一个对象”；功能级授权关注“当前身份是否有权调用这个功能”。例如，普通用户读取他人记录属于对象级问题；普通用户调用管理员接口属于功能级问题。两类问题都需要使用不同身份和不同归属对象进行对照，不能只凭单个状态码下结论。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4kpk0RxAe8jDR5DLQzh5RzKYibOwN3xjASWJxdOQOMYFzVqfp8NibJiaFzwYib8MhTdLPaaWbh1paRHIuibW8MLXaJNwicHYwlJNfV4k/640?wx_fmt=png&from=appmsg "")  
  
❞### 3 上下文记录与跨接口关联  
  
拿到用户 ID、商家 ID、工号、订单号或角色字段后，单点测试往往只能说明一个接口的情况。更有效的做法是将这些事实纳入结构化记录：参数来自哪里、属于哪个身份、在哪些接口出现、哪些验证已经完成、哪些结果仍待复核。AI 可以协助维护这类记录，生成待测清单和结果摘要，减少人工在多轮测试中的遗漏。  
  
自动化只应在明确的授权范围、目标清单、速率限制和操作白名单内运行。它不能“保证无遗漏”，也不应在遇到拦截时自行扩展测试强度。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/DJX1rNqJe4k5xftoHmV1ggaOo9Lxp4nO8zHDJ2nVP9epwkPJd0ib0L9SqKSXKhibiaWMLgUzgwGRafOL7bI2doj6hZ5sQIZhRceGesCicDRicCII/640?wx_fmt=png&from=appmsg "")  
  
❞### 4 角色与高影响功能分析  
  
账号名、roleId、菜单项或前端字段只能作为角色线索，实际权限必须以服务端行为为准。高权限功能通常涉及用户管理、配置管理、导出、文件管理和业务审批。AI 可以帮助把角色线索、接口定义和响应结果关联起来，并优先列出需要人工验证的高影响功能。风险评级取决于可达性、可操作范围、数据敏感度和实际影响，不能仅依据账号名称判断。  
### 5 基于 API 文档的测试设计  
  
Swagger、YApi 或自研接口文档能够说明接口路径、请求方法、字段约束和数据模型，有助于减少调用猜测；它们本身并不等同于白盒代码审计，也未必反映完整业务逻辑。AI 可以依据文档生成边界值、类型差异、权限对照和异常处理的候选用例，再由测试者结合接口语义与响应进行筛选。  
### 6 基于语义的风险覆盖  
  
参数名和数据类型可以提供初始假设，例如 id 类字段常需要对象归属验证，sort 或 order 类字段需要检查输入约束，url 或 path 类字段需要检查服务端处理边界，file 类字段需要检查上传与引用链路。但参数命名不是漏洞证据。AI 的价值在于提醒可能被忽略的类别，并按接口语义生成可控、可记录的测试矩阵。  
### 7 安全边界与人工确认  
  
AI 辅助测试的前提是遵守授权范围和业务安全边界。HTTP 方法不能单独代表风险：GET 也可能触发昂贵计算或状态变化，POST、PUT、DELETE 也可能是必要但高风险的业务接口。因此应以操作的实际效果为准，默认使用最小数据集、只读验证和低频请求；涉及新增、修改、删除、上传、权限绑定、导出大量数据或可能影响生产业务的动作，应暂停并由人工确认。  
## 二  案例记录  
  
以下案例保留原始发现链路、截图和数据规模。文中“AI 参与”均指其对前端代码、接口、参数或响应的辅助整理；漏洞是否成立以截图所示的授权验证与返回结果为依据。  
### 案例 1 访客预约系统的对象级授权问题  
  
在尝试登录并获取站点前端资源后，将 JavaScript 文件交由 AI 协助梳理，得到访客详情和分页列表两个高价值接口。随后以已授权测试身份进行对象归属对照：详情接口需要 JWT，但原始记录显示，切换对象标识后仍可返回其他访客记录；列表接口支持分页并返回完整访客信息。  
  
这一案例的关键不在于接口是否要求登录，而在于服务端是否把“当前令牌代表的用户”与“请求对象”进行绑定。原始记录中的 ID 范围为 1 至 5465，列表接口返回总量为 5,354 条，两者应分别记录，避免将可枚举范围和实际记录总数混为一谈。返回字段包含姓名、手机号、身份证号、工作单位、车牌号和被访人姓名，影响为高敏感个人信息暴露。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4nsLcfF4gqqcA6iaVGAbDJhOZSkGIPW2yiaheESNSBkFU2kkoGrUMfMOGfGlHC5zgic7CF0x9j44hS0WJWciblPspdibkELs4GKL9qI/640?wx_fmt=png&from=appmsg "")  
  
图  
1 登录入口与前端资源获取线索  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4mWRdcarIlnpN8OnKS3V0ADibIia3ju4ss5qx0xq3C8B7xjFGxTuMZvva5nObsME1e5s6Tf0nRcFick95X0ULdh2RMWvSwfLR5IX8/640?wx_fmt=png&from=appmsg "")  
  
  
 图  
  
2  详情接口返回其他对象记录的原始证据  
  
  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/DJX1rNqJe4mYN8Rcct3mVZqv5dHvl8ecBXGfTdK2PnL7TT6nETKAhDpP7LsCIUdqvCEKxicibSER8O9dO4Ze9eOgNOicVWaYyTsQFLzzfMhAc8/640?wx_fmt=png&from=appmsg "")  
  
  
 图  
  
3  分页列表接口返回的访客数据示例  
  
❞### 案例 2 多端系统中的身份与权限边界问题  
  
该系统同时存在微信 H5 端和 PC 管理端，二者使用不同的 API 前缀。首先，以测试账号在个人身份页面进行对象归属对照，原始记录显示可通过修改 ID 查看其他个人信息，属于典型的对象级授权缺失线索。  
  
随后，对 PC 端身份验证逻辑、令牌处理和接口调用进行梳理。原始记录显示，H5 端账户可在跨端身份处理链路中获得更高权限，并在后续接口调用中读取工作人员、供应商、租户等信息，以及与摄像头部署相关的拓扑和分布信息。此类结论需要把“令牌可复用”“角色已变化”“高权限接口可访问”三类证据分别保存，避免只依据前端字段判断提权成立。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4nFcMfibpneiaq7QR8nv5Bl5VjpC0FAFG2QlmoFjjBuo5VRVVibMtkYFtzeNQicNZ0zuPg7cPbhyHjDyVvooicYyMZSKiacO3dRnNiavo/640?wx_fmt=png&from=appmsg "")  
  
图4  目标系统页面与分析入口  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/DJX1rNqJe4l3NsxOoPUXBoLYyceVrC0jt7G4JnfR14TtAoxicQlwKMVLS8B5KVqkdpJsJZ8ZcWoG1QdSQv9jKQbfE1sWicI1BCupKy3Rmb9kA/640?wx_fmt=png&from=appmsg "")  
  
图  
  
 5  不同对象的身份信息对照  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4luad8LTf0pPVSHhEKdn1Q4maxPCpVxgd5lmauP71oz46icWpvfZemveicF620yNp63PjOPqYK74CyIQ8VHzCgK4W4LzTQichyIibc/640?wx_fmt=png&from=appmsg "")  
  
   
图  
6  跨端身份处理与接口响应证据  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/DJX1rNqJe4niaSvEdPsqY7Csj1vjVbaHFTMiaphXLkBK9ic5kcMLF5RoeqTyw7lQDAqTjU3jwH4Q3Y2p4udd29mIK9PhBGiaiaFcH3S8RaPA9Rvg/640?wx_fmt=png&from=appmsg "")  
  
 图  
7  原始记录中的导出结果示例  
  
❞### 案例 3 地图系统中的接口暴露与配置风险  
  
本案例原始记录包含同类地图站点的多个独立发现。为避免把不同系统误认为一条利用链，以下按“教职工信息”“门户配置与角色”“接口文档与文件管理”“组件版本风险”四个子场景整理。  
#### 3.1 教职工信息接口与令牌用途异常  
  
登录请求中出现 OAuth 令牌响应后，AI 协助对前端 JavaScript 进行接口归类，提取出 /os/ 前缀下的教职工相关接口，并结合运行时配置确认服务端地址。原始记录显示：未携带令牌时接口返回 401；将登录响应中取得的 refresh_token 作为 Bearer 凭证提交给资源接口时可访问教职工数据。  
  
按 OAuth 设计，refresh token 应只用于向授权服务器换取 access token，不应被资源服务器当作访问凭证。因此，若原始对照请求可稳定复现，问题应表述为“资源服务错误接受 refresh token 或令牌校验边界不当”。由工号关联后返回的字段包括教师信息、身份证号、电话、姓名、政治面貌和职位，属于高敏感个人信息。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/DJX1rNqJe4nia0wSYhDibFQMic7Z1x9rAKx2DpegLJswknUve198ib1WGuic9pPTWscibXhiaQvafBFIq49Yibjuet1Z0kYSYiaMiczULMOauOQg7lP3w/640?wx_fmt=png&from=appmsg "")  
  
图  
8  地图站点登录页面  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/DJX1rNqJe4mUdEnGA8PvgBIPaK41bQIB3SkQ9B6rwCnvBYb6s1q65CBEYEI9FpIyo5cWnU7oYM3o7TMp451kt3n082KCNIOdRDdPiaXJmdfU/640?wx_fmt=png&from=appmsg "")  
  
  
图  
  
9  登录请求与令牌响应线索  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/DJX1rNqJe4lNcfUJzn9PVrO5icn5mzYUWbQoHgeL8yVHVsMD3Lc1x9iarlonACKfK1MtCCkGibVt7fbWaENZreDS9lgvEE08BXfpib7hibSLWMWc/640?wx_fmt=png&from=appmsg "")  
  
图  
  
10  前端代码中与令牌处理有关的调用线索  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/DJX1rNqJe4nv7yicyznIXC8bHp6iaKJXurn76niafoTaJlndrLRibzo3ofib1ySMn5xkWuKcVOfTMqgyz7k31DHG1YLpAeKic7LGacDQcod8j1L6g/640?wx_fmt=png&from=appmsg "")  
  
图  
  
11  JavaScript 中提取到的教职工接口路径  
  
![](https://mmbiz.qpic.cn/mmbiz_png/DJX1rNqJe4mlTeia2n2fUsS0BibGpCQVwTdMu4qHicGoS39ibwVicAPWd13WdJzeHlsickHbrCPWLWylUpaWicvqUOFuEuHYe9qIUCwSibvia31GAOgA/640?wx_fmt=png&from=appmsg "")  
  
图  
12  携带凭证后的教职工接口响应  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/DJX1rNqJe4lyM6NImzHJ97YQichJxu4Xrr38wiabQcwRsDwDibELibKCOjvhB5ASVSrRcqWXRG1vPL4JwaJFUtLiaCyDehricRTLw9BwOiacTvcNicM/640?wx_fmt=png&from=appmsg "")  
  
图  
13  工号与教职工信息的关联响应  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4k9zQ9d8KdCjBqkShdmiaEYRcKibZ9J6SCJclnsEs4vBXzR1NJSJgh7WZHiam0AdqKvtsLEWbztdKhQozJHmHlySeK3ibThXY0ZMss/640?wx_fmt=png&from=appmsg "")  
  
 图  
14  原始记录中的数据处理脚本界面  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/DJX1rNqJe4nhNia8heO4WQImdjV2CiaLJP6jQBd8lbC80D5kIqrysQ0oOx7gFSooRh633lX2nbQyqY0D5HLZcutZRiaNq2ic6MRFzQ9RmPcsjbI/640?wx_fmt=png&from=appmsg "")  
  
   
  
图  
  
15  原始记录中的导出结果示例  
  
❞#### 3.2 门户配置与角色绑定接口  
  
在登录状态下继续审计前端资源，可发现地图服务配置、后台管理接口和门户配置读取路径。地图服务 Key 出现  
在前端并不自动  
构成高危漏洞，仍需结合服务端限制、配额和实际数据范围评估。原始记录中的重点是配置查询、文件上传、配置保存与角色绑定等接口的服务端权限验证。  
  
对于门户 Logo 修改场景，风险应建立在“普通身份能够读取配置、上传资源并写回配置”的完整对照链上，而不是仅以能够看到图片为依据。对于 rule/bind 场景，原始记录显示普通用户的 role 为空、用户列表可读取角色信息，且绑定 ruleId=1 后可得到大量后台功能。该问题属于功能级授权缺失，需保留绑定前后身份与菜单、接口权限变化的证据。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4kN01cx2Rhsib5nVoc06TbBsf7SOgYTpO6vuEfKs47vPicgV0H4PjbjOXr6O9B4ej6khnMzCFtJ3ojtNibwWjykEas0BsWFp7z3vM/640?wx_fmt=png&from=appmsg "")  
  
图  
 16  前端中出现的地图服务 Key 线索  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4n5CVbTlgWNPMp2v6RcYF9XGFaUw0qvhKpSN09hAuL3O6adiaXQ2EWQUunOqA6KIsbYWia2bHeBj4yjKEiciciclggiakeXVB4BAWH88/640?wx_fmt=png&from=appmsg "")  
  
 图  
  
 17  后台管理接口路径的代码线索   
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4kXD7xNQX2qKx6ic3x8NojDGkOsCyloBOEelwibKYIpx7z2v1SCIZKBFSqRCZTvjXMXC4slBeib2FVhl2GCkE7LGMJVeLjQrHr6O8/640?wx_fmt=png&from=appmsg "")  
  
图  
 18  配置读取接口与 Logo 路径返回  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4mpLcwnMo05WlfaVBibL94CasIVGCTOeVjZMyic1cKR5FoQX8LpXCJpAiazjZt0Wq5nbVHVaia2H6NOEVbdX2COjGF9BcMgEEGBuog/640?wx_fmt=png&from=appmsg "")  
  
 图  
  
 19  门户标题图片的修改前对比  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4lodsSsdGYJia8LqiaLunUtOqzxLBDbaeMyrvv8mFmISCxyDickCzHrfZwC6fNRVeHJQV422l1RYz19GYFV1lp5pN9TKRwXP4IBcQ/640?wx_fmt=png&from=appmsg "")  
  
 图  
  
 20  门户标题图片的修改后对比  
![](https://mmbiz.qpic.cn/mmbiz_png/DJX1rNqJe4kdZYD0ytbMPUlShib07EMYMDq6Uwc2LZDaXibOsbmqejd3EjGibUNiatIe0cn8sfKBkgBJia4hOicQSmrC5vkEmkLSjGFELNmGajBCo/640?wx_fmt=png&from=appmsg "")  
  
 图  
  
 21  普通用户角色状态证据  
![](https://mmbiz.qpic.cn/mmbiz_png/DJX1rNqJe4kiccfwExTTxCDkmNQRxQFxQia5Zu1xyHEaadt24lnDEI8eFxooa836lkrOMrCnjq7ibIxhcUzuZdBwr1OQnl4lB3M5ibJtEic7lqpQ/640?wx_fmt=png&from=appmsg "")  
  
 图  
  
 22  用户列表返回的角色与用户标识  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4kTwDEyx8OK39CxBUiclYuMycJ8pAssJOvodLHlb96WwOLJYvvtI5PTCtRhSRNlial4dfjQNpTmS7gR8iaia26T8bWumk806rPEhbA/640?wx_fmt=png&from=appmsg "")  
  
 图  
 23  后台用户信息中的角色线索  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4nddVO5Ry6FMFjwz0JiaxGbsiaTH6EPBXFnp50yW3NTq0W5GdXV818iayZVoLPElZNCdKFWVbz3LsPS4cgGamvgaOsn5KfNCNkL8s/640?wx_fmt=png&from=appmsg "")  
  
 图  
  
 24  角色绑定请求与响应  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4nfMoPSdeS1o1kl5k7pGMIXYuBO1ibjdD8spicW1lAjXC2USfXtfFK66thYmVOeKpzUXwo1wHb1SW3lpBvSxoWNQIwoBNicibD4s7g/640?wx_fmt=png&from=appmsg "")  
  
 图  
 25  绑定后权限变化的验证界面  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/DJX1rNqJe4mcAjkiaz6zibWmaicRK7pr99ZnCVHmRjmaricQGMEttJX2mK9ll3tTdtNK0Pg11NLm1KanrKlsdEh1ozSQz8ich9gicepKRticqNhOic8/640?wx_fmt=png&from=appmsg "")  
  
  
 图  
  
 26  高权限功能调用线索  
❞#### 3.3 Swagger 暴露、匿名配置读取与文件删除风险  
  
另一地图站点的 JavaScript 审计显示，Swagger 文档和 API 描述可在未认证状态访问，原始记录中包含 273 个接口定义。接口文档暴露本身主要扩大了攻击面可见性；是否构成实际风险，仍要结合其中接口是否需要认证、是否返回敏感数据以及是否能执行高影响操作判断。  
  
原始记录显示，匿名访问可读取内部配置；同时发现文件删除接口。向无效 ID 发送请求返回 400 业务错误，而同站其他受保护接口返回 401。这说明请求可能进入了业务处理路径，是强线索，但单凭 400 不能证明任意文件一定可被删除。后续应在授权范围内，以可恢复的测试资源或经批准的对象完成“有效 ID、删除行为、实际影响”的闭环验证。原始记录中通过公开 POI 与文件引用接口取得文件 ID，并展示了网站图片资源删除造成页面缺图的影响。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4nWPVPVLhr0ZFo103ZmhKrFJ9RzEKzI3nUG9xQkaic4ly9zHpsic9HXEjG4YeMUW7TsGBNwYB4VSFaZFToDhlGDGkAnpRN4WYI0w/640?wx_fmt=png&from=appmsg "")  
  
图  
 27  前端代码中的地图接口定义  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4lelzvEdXEatEWA9xSqXmb25aOLBY7QTK7qlaRXicOyHujibfVMWSn2BLLudmOclTZsLU4sFY9LOoQ3H7iapZokoRPOp9AX9aUCfo/640?wx_fmt=png&from=appmsg "")  
  
 图  
  
 28  匿名访问 API 描述文档的原始页面  
![](https://mmbiz.qpic.cn/mmbiz_png/DJX1rNqJe4nbqV4GDp3POSxSDzIpsglzUqG7CWOtrPdSkDUayp2hrjwP5B39RU69zArWrkvCJRJr9TDD1bHWKb1drDdzu2dyn5ZOftATRd4/640?wx_fmt=png&from=appmsg "")  
  
 图  
  
 29  Swagger 页面公开可访问的证据  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4nC7qCJYLWxbD8EtoiacYobLUv8fRsDiaKSJlnRsTcm5fG5snFS0GrcDPNvAFC3k0KjWYnerjtwKpS9NN6uoA83rFc6rdy99fu0w/640?wx_fmt=png&from=appmsg "")  
  
 图  
  
 30  匿名读取配置的返回示例  
![](https://mmbiz.qpic.cn/mmbiz_png/DJX1rNqJe4lzu91n2MBickbLoBHCYy82lY6ibEVr9sOsD86ndeIwxzKxRlDy9MvibAupoNr0JN4h0icS3Hs0KiazXMibT2cLEHgDfhjT9gBrj2hdI/640?wx_fmt=png&from=appmsg "")  
  
图  
 31  删除接口返回业务错误的线索  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4m89Z39Pib1TGdABx1g0fITibdR8FMvUSibfmRe11RI8X36fiaPU50Iib3lWbQrCgKWWHySLJUuqz6X32tyL6N6GayYEuiblgqYa88mU/640?wx_fmt=png&from=appmsg "")  
  
 图  
  
 32  公开接口返回的对象关联信息  
![](https://mmbiz.qpic.cn/mmbiz_png/DJX1rNqJe4lz5A2K4KLB1X8iczVxy5z3xyqpBxCQebcANFDyQaFZfaRSlo6h1xEMWd2Nlw29tXIShcVNxmCFtx4sHYQVyb2ySJD7iaprhWCjc/640?wx_fmt=png&from=appmsg "")  
  
 图  
  
 33  文件查询接口返回的文件标识  
![](https://mmbiz.qpic.cn/mmbiz_png/DJX1rNqJe4l5wDbRlIyPDmvKcawGqaj5TcoBHEBia8ibmUOVMDWQ7PrjPx7V5ZnoLqX761hp7UdxIn8pAbND4ibNm8NwNNjMHCmzy0BDtrXa7A/640?wx_fmt=png&from=appmsg "")  
  
 图  
 34  原始记录中的页面资源影响示例  
  
❞#### 3.4 GeoServer 组件版本与远程执行风险验证  
  
前端配置和瓦片 URL 表明站点部署了同域 GeoServer。原始记录通过管理入口、能力文档和页面指纹识别到 GeoServer 2.22.5，并关联到 CVE-2024-36401。版本匹配只说明需要优先排查，不能单独证明目标可利用；漏洞是否成立还需在授权范围内进行最小化、无破坏验证，并确认真实执行上下文。  
  
原始记录还展示了 WAF 拦截差异、受控验证请求和 /proc/self/status 的回显。该部分保留为风险证据，不展开可直接复用的绕过构造。回显中 UID 为 0，说明当时记录的进程以 root 身份运行，风险不仅来自组件漏洞，也来自服务进程权限过高。修复建议应包括升级或应用官方补丁、限制对相关服务的访问、以低权限用户运行服务，并复核 WAF 规则不能替代版本修复。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4m7PxOUtSib4UvEfdtzrrQjia34uFrTIUJic1uTu6cdpJgVQPr5AbV5bovr9DEy78SDPyZ7sictyDnMop7nRHUTYVgRZ94mEY362TI/640?wx_fmt=png&from=appmsg "")  
  
图 35  JavaScript 中发现 GeoServer 服务地址的线索  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/DJX1rNqJe4kSNicQxRT9JunfjiabgvrRz0nC1u0cPib0iaRLL2xxPicTBD8iankOUZlJlXkusRKP7Zpia58hTlr5HOpPtxaT4VJJaBd6DncrccXRbw/640?wx_fmt=png&from=appmsg "")  
  
 图  
  
 36  GeoServer 管理界面与版本指纹  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4kuZdKPu15FRhvXE6SibibyvTCFqj0o2qG8HAtdnjS8XStCl2ENq5rNDb5CoXzaE9Wicicibic5s5ESNAvpRD7aNrUUPzVc9icl1HOhNE/640?wx_fmt=png&from=appmsg "")  
  
图  
 37  受控验证中的拦截响应示例  
  
![](https://mmbiz.qpic.cn/mmbiz_png/DJX1rNqJe4nzMDInQicribbAnTAiaOJy8b64esARjY7UG0njicjIXC4KHgo94PfY9wz4zTOLhzAJMLgWsfjxsjhBz02TRsLN02Dh2D2zxur83X4/640?wx_fmt=png&from=appmsg "")  
  
图  
 38  不同输入处理结果的原始记录  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4mOjambOB9w2aCDD47Giaoc6khicbfz8EXshzSCRicy2JISDwXCy6icaV7KHSLbCSKcwkgrK32Y3sYzRTAEx0JhGTfJKVV9jthpGcs/640?wx_fmt=png&from=appmsg "")  
  
图  
 39  授权验证成功的响应证据  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4lXVgwhCR99iaV1Kew5QNH6n6LBn1HH3eibSS0xyr3iajUEr5IQt8A1LOTwxcY0fAT3TzmNB3Ml4JGurSNzI4baomguh2egvSBako/640?wx_fmt=png&from=appmsg "")  
  
 图  
  
 40  受控回显写入与读取的原始记录  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4lHia2Szuo7So7DeTJb83DKHNYV887uES6dKFX9RKr57mic243819GxRHAQ2nEAzCYGyhXjxNUTrWw7DgvzZLkEun8aBtcuFvEnI/640?wx_fmt=png&from=appmsg "")  
  
图  
 41  进程身份回显，显示 UID 为 0  
  
  
❞### 案例 4 人力资源平台的未授权接口与注入风险  
  
在审计前端 JavaScript 并梳理 .mob 端点时，原始记录发现 MobileInterface 模块的多个接口。以 getStaffEduInfoDetail 为例，正常 staffId=1 时返回空数据；改变 staffId 输入后，接口返回约 2.18 MB JSON 数据，其中包含 4,878 条员工教育信息。返回字段包括毕业院校、教育经历和操作人相关信息，截图中还显示身份证号字段。  
  
从黑盒现象看，异常输入导致返回集显著扩大，可作为 SQL 注入的有力证据。关于“用户输入被直接拼接 SQL”“未使用预编译”“AuthenticationTokenFilter 在 web.xml 中被注释”等根因表述，只有在获得源码或配置证据时才能作为确定结论；若仅来自响应和前端分析，应明确为待确认的根因假设。另一个相似系统中，原始记录列出 getStaffWorkInfoDetail、getStaffFamInfoDetail、getStaffContract 等可控接口，应逐一验证身份要求、输入处理和返回范围。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/DJX1rNqJe4mEPbian09Lsp2HoYgBe43fpSj4mxx7RcJmWTa2qRQID1aEbVCt719sGdHJl63fayQAeXcarkjkSTSwylLkicMusM2IbNW3qZB2A/640?wx_fmt=png&from=appmsg "")  
  
图  
 42  目标人力资源平台入口  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4m1YWf33ibW4o5gQ4SVw54aHa3gxNk4JZAGibyheU9rk9yXAIsAIIho3XSIynfSDKgIicia1IYzKlExXTW7jeh3xdpfcnYrXvPpxVk/640?wx_fmt=png&from=appmsg "")  
  
图  
 43  异常输入后接口返回的大量员工教育信息  
  
![](https://mmbiz.qpic.cn/mmbiz_png/DJX1rNqJe4lSTzbUBmmJUKCn6TWeCzS3e9mU7xT1wIE6KWZ5Vg9hoNnHbZCayVk1cfPJEAibrdU5ALdkfdkETCEU7PDuRJTn7pcCDhkFojws/640?wx_fmt=png&from=appmsg "")  
  
 图  
  
 44  返回数据中身份证号等敏感字段的证据  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4nXpFxopunaLOMDx9paKVaW7tDXZeVlhb7tplrbqSk9ApUuuXmJ6GXaiaNrsTv1B5wjYPzFU2FGDSaNCjrBOPfCqib7dFlv4x3vY/640?wx_fmt=png&from=appmsg "")  
  
 图  
  
 45  相似系统的登录入口  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4notdaQgNR4x7CJRnBuBYNicQfBB8mBVEicEBj5h8FiaYavNmYE0VgCAoTT9HAqKUpa2rJzMCqvicxZTXPdwD1Ay77tNJ86pFaqzd8/640?wx_fmt=png&from=appmsg "")  
  
图  
 46  相似系统中员工信息接口的响应  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4mzsDUjQYyb7m8FG9eWYBicTtGXf6zL5cK8zoaGcEhX3SCymia7h8m6LAUjfj21ctSeGV5I83ykFKJicuUy918q4FwedNzV5bZzBY/640?wx_fmt=png&from=appmsg "")  
  
 图  
  
 47  相似系统中合同信息接口的响应  
❞### 案例 5 证书查询系统的对象枚举与移动端访问控制问题  
  
扫描职业等级认定证书二维码后，可得到带有证书编号的查询 URL。原始记录显示，轻微修改编号即可返回其他证书信息，且编号中包含机构、年份、等级和流水号等规律。AI 在此处的作用是辅助归纳编号结构、整理机构范围并生成受控的枚举计划；它不能替代对单条记录归属关系和速率边界的人工验证。  
  
原始记录先确认约 3 万条姓名和身份证号泄露，并在后续按编号规律扩大覆盖范围，识别出 80 余家机构。20 万以上的规模属于基于机构数量、年份和序列分布的粗略估算，应与已实际验证的数据量分开陈述。  
  
修复后复测发现，PC 浏览器显示证书不存在，而移动端 User-Agent 可返回更完整的结构化数据。若服务端仅依据 User-Agent 关键字跳过 certUid 校验，则属于访问控制缺失：UA 是可伪造的客户端标识，不能作为授权依据。原始记录中的响应包含身份证号、证件照、准考证号、监考人员、考场地址及答题详情等大量敏感字段，影响需按实际可访问范围和已验证样本确定。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4nxhNE9uwrDibP9SdyVia9zWo4fA6jwUddIOnG6eNQdZxhQibYW4NrL8OYPpso9q7PHOfp3gaf3GUUwXvo71JHsSdiaEaRFtDQ99O4/640?wx_fmt=png&from=appmsg "")  
  
图  
 48  职业等级认定证书二维码  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4kdiclYaBmmfs3Gze8FlOneWHJsrGa8T4YooPKibO9BctxpRxvenal8h5kmicAlQicv54csH3ibwuBaxucyl4NYWYnw9gmpgibLltoCc/640?wx_fmt=png&from=appmsg "")  
  
 图  
  
 49  证书查询页面返回的个人信息示例  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4l3iaLM4dywFYXsd9CCnLaPqwiaopS8O8GK9pcAAiboSH7BTFiaZ2XsvRjxRAch3IrGm5UdbWTyUicZyWZu0hh4bVZMicx69WTcjbjibc/640?wx_fmt=png&from=appmsg "")  
  
 图  
  
 50  证书编号组成与可变字段说明  
![](https://mmbiz.qpic.cn/mmbiz_png/DJX1rNqJe4kqAPy50656ezzPeNE9v2UZAQiags2WW8PsfBs5UP00E4Sr1VlWBWE369W4vBL4zIq5AuEtygSCgyQic9cgSrEMTXia5hL8T2Fmia8/640?wx_fmt=png&from=appmsg "")  
  
 图  
  
 51  机构编号列表的原始记录  
![](https://mmbiz.qpic.cn/mmbiz_png/DJX1rNqJe4mOXELff2LIkV8qIsXxqX7ic5u80U7cyYyRLibB6dxjdjaAupJXkogbu71wKsCnibMTuPhPsECYY3INVvAbGlPFjMAaf6jROK1fkc/640?wx_fmt=png&from=appmsg "")  
  
 图  
  
 52  跨机构与跨年份访问证据汇总  
![](https://mmbiz.qpic.cn/mmbiz_png/DJX1rNqJe4mcyr4qHt9aUzCPUuO1OX6np3dJr26pjziab2FTRTYcQSjE9zblmc9TZ9BlFho9Gw7NKlaicr3Ee16hGrBM8548C6nEQKaTh5Os0/640?wx_fmt=png&from=appmsg "")  
  
 图  
  
 53  已确认机构范围的原始列表  
![](https://mmbiz.qpic.cn/mmbiz_png/DJX1rNqJe4mHS8jOm86qodribY4WY9ia9hJicwdP01mAOHmlhicyg2UCMbkRnPicE0XqicUttM9xtp1FhmlpPQ33N3ocPRhuNiaOEEQxCQva6QNsLU/640?wx_fmt=png&from=appmsg "")  
  
  
 图  
  
 54  不同机构与年份的样本记录  
![](https://mmbiz.qpic.cn/mmbiz_png/DJX1rNqJe4lSuic2bt6WRCoAnYOHFwNkM9aFlDXEUpHN3icpwfljUJAibtFGaX6kSpaIJLUx6ZFJ586lM1luA8HqPFuNJmHKsrSZWWjLTjhcFM/640?wx_fmt=png&from=appmsg "")  
  
 图  
  
 55  移动端页面返回的半加密证书信息  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4l054qZ4vn2K6FOycib5Yn8JofeHhj45OczC2O7GjAJnUqSKZKFjydqtRaLUKbPZicAiaf8TiaUvOXjlJesdYwp0cbBrAZGNDJibBwE/640?wx_fmt=png&from=appmsg "")  
  
 图  
  
 56  请求中 User-Agent 与返回字段的对照  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4nznqLg2QyXiat1gTBw1UUhDOQjS8IE0qF5tgPz5uGshPF52su6aDTcsRczoDXGreibCicEFVdYwqQrgTuas2L6zibHN1DbbiaK1Ebs/640?wx_fmt=png&from=appmsg "")  
  
 图  
  
 57  移动端查询接口的结构化响应  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4m1Gq7X2YmONMkyKqGXH8gy2PU3MwPBbSJubDZEaCd9ghAlkZB9Ks4QVCs1apk4JOJZicMC6PbXAptYGibzTJFEfmdDeTqWKfib6A/640?wx_fmt=png&from=appmsg "")  
  
  
 图  
  
 58  原始记录中的数据规模与覆盖范围汇总   
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DJX1rNqJe4nTC01vwQr7ldCGKtBstBbQeYYGwRhqwmuF8B1hs5cn3h4HhVawq1nsJUEAOlzY1J2PSVlBgaoScgMPmqbdWZfQqljqeq1uIso/640?wx_fmt=png&from=appmsg "")  
  
图  
 59  浏览器端显示的大量结构化返回数据  
  
❞## 三  从案例中得到的实践原则  
  
这些案例共同说明，AI 最适合承担“信息整理和测试编排”的角色：从大量前端文件中找接口线索，从响应中整理身份、对象与角色关系，从长链路中保留上下文，并帮助形成可复核的测试记录。AI 的输出必须经过人工验证，尤其是有关鉴权、注入根因、版本可利用性、数据规模和业务影响的结论。  
  
一份可复核的漏洞记录至少应回答五个问题：测试身份是谁；目标对象是否属于该身份；发送了什么最小化请求；返回了什么可验证结果；在授权范围内造成或可证明的实际影响是什么。将这些事实与 AI 提出的猜测分开记录，才能既提高效率，也避免因模型误读、接口遗漏或过度推断导致误判。  
  
最后，任何自动化都应有明确边界：目标和时间窗口经过授权，操作白名单和速率限制可追踪，涉及写入、删除、上传、权限变化或批量导出的行为由人工确认。这样使用 AI，才能把效率提升建立在可靠证据和业务安全之上。  
## 参考资料  
  
OWASP API Security Top 10 2023：API1 Broken Object Level Authorization；API5 Broken Function Level Authorization。  
  
OpenAPI Specification v3.1.1。  
  
RFC 6749 The OAuth 2.0 Authorization Framework。  
  
GeoServer 关于 CVE-2024-36401 的官方安全公告。  
## 建了个src专项圈子，内容包含src漏洞知识库、src挖掘技巧、src视频教程等，一起学习赚赏金技巧，以及专属微信群一起挖洞圈子专注于更新src相关：1、维护更新src专项漏洞知识库，包含原理、挖掘技巧、实战案例2、分享src优质视频课程3、分享src挖掘技巧tips4、小群一起挖洞图片图片图片  
  
  
