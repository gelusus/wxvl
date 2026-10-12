#  AI挖洞实战 02｜Workbuddy+Skill技能库，完整复现高危文件上传漏洞  
原创 网安学习室
                    网安学习室  网络安全学习室   2026-10-12 04:47  
  
做SRC挖洞，文件上传漏洞是永远的高频高危考点，也是新人最容易拿高分的漏洞类型。  
  
但绝大多数新人挖不到有效漏洞，核心问题不是不会测试，而是**流程混乱、找不到上传入口、测试无边界、报告不规范**  
。  
  
传统手动挖洞，需要反复抓包、改包、测试绕过、整理步骤，耗时费力还容易出错。  
  
今天给大家落地一套**标准化AI自动化挖洞全流程**  
：依托 **Workbuddy Agent + 全套Web漏洞Skill技能库**  
，分步完成「上传点定位→漏洞排查人工复现→自动生成SRC标准报告」，全程闭环可落地、可直接复刻用于SRC众测。  
## 01 核心工具：标准化AI挖洞体系  
  
本次实战全程依托成套自动化体系，摒弃无脑瞎扫的低效模式，采用**分技能、分步骤、可控化**  
测试思路。  
  
配套工具：**Workbuddy Agent**  
  
核心资源：**40套Web漏洞专项Skill技能库**  
  
整套技能库覆盖网安挖洞全链路：信息收集、接口探测、各类漏洞专项排查、合规报告生成。  
  
无需每次重复编写复杂提示词，根据测试场景直接调用对应Skill，测试标准统一、流程专业、结果精准，完美解决新人测试无思路、流程不规范的问题。  
## 02 实战前置：合规测试环境准备  
  
所有SRC挖洞的前提，一定是**合规授权**  
。  
  
优先注册补天、漏洞盒子等正规SRC平台账号，仅在平台授权的众测资产范围内开展测试，杜绝未授权扫描、渗透，规避法律风险。  
  
本次实操测试目标：**用户中心个人资料页面**  
  
测试地址：http://192\.168\.110\.148:9090/profile\.php  
  
核心测试点：页面头像文件上传功能（高频高危业务漏洞入口）  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Vs6KsYlvMyPibh1URTqfRmrsSLeDafN1RFpUWqsslCoKVcm2FciajJZCBfvSonYuibrQamUibpKdFedorKbxcx5vG2FjPMrYbbcgqKtibhJFOuoU/640?wx_fmt=png&from=appmsg "")  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Vs6KsYlvMyO4pBBaXqLw3MzODicYtCDHa7jCCiaG8rJiblAexaF0MDVqMI7IfhibjNyNZldtcoFhtXTjWWcibv04s9mNic862nsR05ydUNK7guonU/640?wx_fmt=png&from=appmsg "")  
![](https://mmbiz.qpic.cn/mmbiz_png/Vs6KsYlvMyNygG65PvkrgE6Mf5M44oYPJ829dzibzYI6ic3hlcbevScfXaKQjHGgHOgkxibpcBgwLcI2XB6HnJgJeqAPIdalLruQLsQpGu7rh4/640?wx_fmt=png&from=appmsg "")  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Vs6KsYlvMyNtoHlehVmeFoanh5N9xkz7nlkCUyrI9Uptx47GqaM8hg5DXVCb4J5etdPBPtoeu8TxJiaJJ5Cic6LlCa8xNFACwoDNc0Fm0aPE0/640?wx_fmt=png&from=appmsg "")  
  
## 03 第一步：AI精准定位文件上传入口  
  
很多新人测试上传漏洞最大的误区：上来就直接传文件测绕过。  
  
真实专业测试流程，第一步一定是**先定位所有合法上传入口**  
。  
  
很多上传功能隐藏在登录后的业务页面、前端JS接口中，并非肉眼可见，盲目测试只会遗漏漏洞、打乱测试节奏。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Vs6KsYlvMyM1YsCCJdZfvRd9PdZBJzuicGuU27jem7Jx0edw14rszzCZcEvNJgQqXx6wbjjMnaI31HGFLx5Ov5ldfy33DnJOK3PSPdX3PlgU/640?wx_fmt=png&from=appmsg "")  
  
### 本次调用Skill：04-前端JS与API信息收集  
  
我们通过精细化提示词约束AI行为，只做入口识别，不做漏洞测试，彻底区分「找入口」和「测漏洞」两个环节，避免结果混淆。  
  
AI通过登录授权账号，完整分析页面表单、前端JS代码、业务接口信息，最终精准定位核心漏洞入口：  
  
**目标页面profile.php存在头像上传功能，上传字段为avatar，前端仅限制图片格式，无后端强校验。**  
  
至此，我们得到唯一有效测试入口，为下一步漏洞排查锁定精准范围。  
## 04 第二步：AI自动化排查 + 人工复现漏洞  
  
上传入口确认后，直接调用专项漏洞技能，针对性开展漏洞检测。  
### 本次调用Skill：08-文件上传  
  
依托预设成熟测试逻辑，AI自动对已定位的上传入口开展合规检测，同时严格约束测试边界：仅验证漏洞存在，不扩大危害、不横向渗透，完全贴合SRC测试规范。  
### 漏洞探测结果  
  
该头像上传功能存在**高危文件上传漏洞**  
：  
  
1. 前端仅做图片格式展示限制，后端无任何文件类型、文件内容校验；  
  
2. 上传文件保留原始文件名，支持直接访问解析；  
  
3. 可上传PHP脚本文件，实现服务器命令执行，危害等级高危。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Vs6KsYlvMyP4yVKSG90cXUajHpr09FfgGvCmlXG4TicwYKdaq78M0V4JwsEFCnwiaRnbu2mHhQibd6rho7RNGoFadticiaIoOClSG62N2q4OcICM/640?wx_fmt=png&from=appmsg "")  
  
### 100%可落地人工复现步骤  
  
为规避AI误判，所有漏洞结果必须人工复核，完整复现流程如下：  
  
1. 新建验证脚本文件：verify_cmd.php  
  
2. 文件写入测试代码：<?php echo shell_exec("whoami"); ?>  
  
3. 登录测试账号，进入个人中心profile.php头像上传位置，提交脚本文件  
  
4. 访问上传后的文件地址，页面成功返回服务器用户信息  
  
复现结果证明：**后端可正常解析恶意脚本，存在完整命令执行漏洞，漏洞真实有效。**  
## 05 第三步：AI一键生成SRC合规报告  
  
漏洞复现确认后，无需手动编写复杂报告，彻底解决新人「会挖洞、不会写报告」的痛点。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Vs6KsYlvMyN5ib3uT3BvhLibicD42HnalJHKoGCpoan0rUaCWKSMTyv9DTZb0svyHdGaNdsXt5jOkv3XEJ59YNnRmbbuH1LVicumhsl1xBIsdaY/640?wx_fmt=png&from=appmsg "")  
  
### 本次调用Skill：40-漏洞报告生成-VulnReportGen  
  
技能库内置补天、漏洞盒子等主流SRC平台标准报告模板，包含漏洞标题、资产信息、漏洞原理、复现步骤、危害分析、修复建议等完整字段。  
  
我们仅需明确核心规则：**仅基于人工验证的真实结果生成报告，不夸大危害、不杜撰未验证漏洞、不补充无关风险**  
。  
  
AI自动输出完整报告初稿后，仅需补充实操截图、核对资产信息、复现步骤、请求响应数据，确认内容真实准确，即可直接提交SRC平台等待审核结算。  
## 06 漏洞根因与风险总结  
  
**漏洞根因**  
：开发过度信任前端校验逻辑，仅在页面端限制上传文件格式，服务端未做二次校验，恶意用户可绕过前端限制上传脚本文件。  
  
**漏洞危害**  
：攻击者可上传恶意木马、脚本文件，实现服务器命令执行、文件读取等高危操作，接管服务器权限，造成企业核心数据泄露、业务瘫痪。  
  
**SRC定级**  
：高危业务漏洞  
  
本次全套干货已打包：**AI 自动化挖洞 Skill，也就是这套《Web 漏洞技能集》**  
。这套技能库一共整理了 40 个 Skill，从前期的信息收集、资产分析，到中间的漏洞排查，再到最后的漏洞报告生成，整个挖洞流程都已经拆分好了。  
  
后面实际测试的时候，只需要根据当前目标的情况，选择对应的 Skill，让 AI 按照对应流程执行即可，新手也可以快速上手，直接👀👉[全套自学资料](https://mp.weixin.qq.com/s?__biz=MzkzNzk5MjEyNg==&mid=2247483826&idx=1&sn=4ff03198e63ca6e68161a5b93eee17da&scene=21#wechat_redirect)  
  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/Vs6KsYlvMyNxf7qI2XIPEu06RvIuO0J4mUanPzCzseMd2Y4hpdU1RcNica9SMNYUdNf6ibR5Dru6PuEFAXsDtib5E9gMU8sDz3SgazXhibavwbw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1 "")  
  
⚠️ 合规声明 本文所有技术内容仅用于官方授权 SRC 合规测试与网络安全技术学习，严禁未授权探测、非法渗透测试，违规操作自行承担法律责任。  
  
[#网络安全]()  
 [#SRC挖洞]()  
 [#AI自动化挖洞]()  
 [#Hermes]()  
 [#业务逻辑漏洞]()  
 [#优惠券越权漏洞]()  
 [#网安副业]()  
 [#零基础挖洞]()  
  
  
