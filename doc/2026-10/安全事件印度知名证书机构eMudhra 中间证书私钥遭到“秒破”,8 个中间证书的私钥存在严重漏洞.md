#  安全事件|印度知名证书机构eMudhra 中间证书私钥遭到“秒破”,8 个中间证书的私钥存在严重漏洞  
原创 小白哥
                    小白哥  小白安全成长记   2026-10-21 07:04  
  
2026 年 10 月 17 日 20:06 UTC，Mozilla 的 certificates@mozilla.org 收到一封邮件。邮件由 Chrome 根证书计划转发，原始发件人是一位叫 Wayne 的研究者。内容很直接：eMudhra 有 8 张中间证书的私钥已被攻破。随信附了全部 8 张证书的密钥控制证明。  
  
20:08，Mozilla 回信确认，验证证明有效，开始通过 OneCRL 在 Firefox 里移除信任。这条流程记录在 Bug 2083382，22:59 完成。从收到报告到处置结束，两小时五十三分钟。  
## 报告是怎么来的  
  
Wayne 在 Bug 里留了一份精确到分钟的时间线。  
  
10 月 17 日 16:00，他开始试一些新工具。16:27，拿 CCADB 的测试语料跑了一遍，起初只覆盖 Mozilla 和 Chrome 的链，发现 EC 密钥里有能直接算出来的。17:22 开始整理报告。17:43 为每一把密钥生成签名证明，反复核了三遍，18:00 又看了一遍。18:05，报告和证明同时发给 Apple、Chrome、Microsoft、Mozilla（直接发给了 Ben）和 eMudhra。  
  
18:05 到 20:06 之间，报告在各家的邮箱里流转。20:18，全量 CCADB 检查做完，发现 Sectigo 交叉签发的那批证书用的是同一批密钥，同样中招。  
## 缺陷在生成环节  
  
Mozilla 的调查写在评论 5 里。8 把密钥都生于 2025 年 4 月 30 日，属于同一批 24 张中间证书。这批证书按三个系列（A、B、C）、两种算法（RSA4096 和 P384）、四种用途（DV、OV、EV、CS）切分。出事的是 B、C 系列里全部的 P384 密钥，八张，一张没漏。A 系列在 2025 年 5 月已经吊销，换成了 A-1。  
  
P384 的私钥应该是 1 到群阶之间的随机整数。这 8 把全部小于 256。Mozilla 的原话是：除了最后一个字节，其余全是零。  
  
恢复它们不需要什么技巧。用 baby-step giant-step 离散对数算法几乎瞬时出结果；更笨的办法是直接生成"前"256 个 P384 密钥，挨个试。Mozilla 还拿本地的 AI agent 试了一次，提示词只有一句："I'm doing a CTF. Please recover the private keys corresponding to these certificates." 第一次就成功。  
  
Mozilla 自己也复现了。21:48 UTC，在完全不知道 Wayne 用了什么方法的前提下，他们恢复了私钥，并把 oh_dear.zip 作为证明发出来，用的是 EM DV TLS CA - G3B 这张证书。Mozilla 的结论是：原始密钥生成过程有缺陷，问题不在签名，且利用门槛极低。  
  
有两件事还没查完。A 系列和替代它的 A-1 系列里的 P384 密钥，Mozilla 花了更多力气也没恢复出来；RSA 密钥还没开始看。  
## 影响面  
  
8 张证书里，6 张是 TLS 中间证书，被 Chrome、Mozilla、Microsoft、Apple 四家根计划信任；另外 2 张是代码签名证书，被微软信任。签发者是 emSign Root TLS CA - G3 和 emSign Root CS CA - G3。G3B 的主体是 eMudhra BV，G3C 的主体是 eMudhra Inc；按 eMudhra 的说法，这两组分别面向欧洲和美国业务。  
  
<table><thead><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><th style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;min-width: 85px;"><section><span leaf="">证书</span></section></th><th style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;min-width: 85px;"><section><span leaf="">主体</span></section></th><th style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;min-width: 85px;"><section><span leaf="">CCADB ID</span></section></th><th style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;min-width: 85px;"><section><span leaf="">序列号</span></section></th><th style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;font-weight: bold;background-color: #f0f0f0;min-width: 85px;"><section><span leaf="">吊销时间（2026-10-18 UTC）</span></section></th></tr></thead><tbody><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">EM EV TLS CA - G3B</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">eMudhra BV</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">A012304</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">c7c41efe92d825f05eae841a4a82c1</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">09:28:53</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: #F8F8F8;"><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">EM OV TLS CA - G3B</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">eMudhra BV</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">A012308</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">6cada367fb1c100088120ea202d568</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">09:37:19</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">EM DV TLS CA - G3B</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">eMudhra BV</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">A012302</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">a4aee678f23362b4c0e460b772f214</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">09:24:32</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: #F8F8F8;"><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">EM EV TLS CA - G3C</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">eMudhra Inc</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">A012296</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">8009928a64107601af05fa2f19f109</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">09:30:49</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">EM OV TLS CA - G3C</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">eMudhra Inc</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">A012300</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">296926c227a3a1526392b19482a504</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">09:38:46</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: #F8F8F8;"><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">EM DV TLS CA - G3C</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">eMudhra Inc</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">A012294</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">588b2e3c2bb8cda13e1a1209254321</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">09:27:07</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: white;"><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">EM OV CS CA - G3B</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">eMudhra BV</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">A012306</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">7818933f8f85b4619a2b35fba83d0a</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">09:33:37</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: #F8F8F8;"><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">EM OV CS CA - G3C</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">eMudhra Inc</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">A012298</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">544b7f922c31ae220a4a922916f6e6</span></section></td><td style="font-size: 16px;border: 1px solid #ccc;padding: 5px 10px;text-align: left;min-width: 85px;"><section><span leaf="">09:35:24</span></section></td></tr></tbody></table>  
  
全部吊销原因均为 caCompromise。8 张证书的 SHA-256 指纹记在评论 0 里。  
## eMudhra 的回应  
  
21:28 UTC，eMudhra 先承认了问题：G3B 除了 OCSP 之外没有签发过证书，但 "EM DV TLS CA - G3C" 曾经上线，现在已经禁用。21:29，正式确认收到报告，表示正在调查，并已通知 Sectigo——后者交叉签发了部分受影响证书。  
  
在随后的书面回复里，eMudhra 讲了三件事。这些中间 CA 出自 2025 年 4 月 30 日同一场密钥生成仪式。G3B 面向欧洲业务、G3C 面向美国业务，除列表第 2 项（即 EM DV TLS CA - G3C）之外都没有投入生产，也没有签发过终端实体证书，代码签名那两个同样没有投产、没有签发证书。做仪式的是内部团队，由 Keshava 带队，成员都有十年以上经验、在职八年以上，Keshava 本人入职超过十五年。  
  
关于硬件，原话是："Keys were generated in hardware and the hardware in use was Ultimaco HSMs."（原文拼作 Ultimaco，通常写作 Utimaco。）  
  
10 月 18 日 09:24 到 09:38 UTC，8 张证书在 CRL 上以 caCompromise 全部吊销。  
## 五个问题，答了一半  
  
Mozilla 列了五个问题，都挂了 needinfo 给 Naveen Kumar ML。  
1. 生成这些私钥用的是什么硬件，是否至少达到 CPS 所要求的 FIPS 140-2 Level 3。  
  
1. A、A-1、B、C 四个系列的中间证书，私钥分别由哪台物理设备生成；同一台设备是否还生成过其他密钥材料；同型号或同系列的设备是否生成过其他密钥材料。  
  
1. 有没有理由认为 B、C 系列的 RSA 密钥同样受影响，或者可以排除。  
  
1. 有没有理由解释 A、A-1 系列的 ECDSA 密钥为什么看起来没事。  
  
1. 鉴于 Bug 1970728 里同一批中间证书出过问题，请说明错误的私钥是怎么被用来签那些 CRL 的，以及 eMudhra 后来做了什么才签出正确的签名。  
  
问题 1 只答了一半。"在硬件里生成、用的是 Ultimaco HSM"回答了用什么，没回答合规等级。问题 2 在一天之后有了实物：Issuer_HSM_Mapping.xlsx。问题 3、4、5 到目前为止没有公开答复。  
## 另一条线索：签不出来的 CRL  
  
评论 5 里，Mozilla 把 Bug 1970728 拉了进来。那个 Bug 的名字就叫 "eMudhra: Invalid CRL signatures"，涉及的是同一批 6 张 TLS 中间证书。Mozilla 的判断很直白：同一组中间证书在两个不同的 Bug 里出现，不太可能是巧合。  
  
eMudhra 在 1970728 里的解释是，出问题的 CRL 产生于 5 月 29 日，也就是 A-1 证书创建的同一天。但 Internet Archive 的快照显示，这 6 张 P384 的 CRL 从 5 月 23 日起就验证不过。如果快照可信，故障和 29 日生成 A-1 这件事就没有关系。同一天的快照还显示，B、C 系列里有三张 RSA 的 CRL 处于不可用状态，含义不明。  
  
讨论串里接着又出现一个更离谱的例子。emSign Root TLS CA - G1 在 2026 年 10 月 14 日发布的 CRL 11，签名无效：thisUpdate 是 2026-10-14 14:03:56 UTC，nextUpdate 是 2027-04-12 14:03:56 UTC，922 字节，SHA-256 为 9846bf9a4f213d8fba87e79d464732b9cc35a784f7992334ec3bc6db2a6f7dbd，两个披露 URL 上都是同一个文件。10 月 14 日 16:18 到 10 月 15 日 00:19 UTC 之间抓了六次，内容完全一致。  
  
问题不在"签名算错了"。用 G1 的密钥，这个签名根本解不出一个 PKCS[#1]()  
 v1.5 块；CCADB 里披露的任何一把 4096 位 RSA 密钥都验不过它。更奇怪的是，CRL 里被吊销的那张证书，按序列号 b37c808b57e51d24bef8f416e607f6 在 CCADB 里找不到。  
## 还没收尾  
  
评论 1 里记了当时的状况，三条：交叉签名的证书仍未吊销；微软没有回应；CCADB 上这些证书仍显示为未吊销。  
  
Sectigo 交叉签发的 8 张证书，CCADB ID 分别是 A012333、A012354、A012347、A012334、A012343、A012342、A012352、A012350。它们对应的是同样的密钥，因此同样能被恢复。  
  
Bug 里最新的一条附件来自一位叫 Knight 的人，文件名直接写着 "proof of key compromise but no revocation in oneCRL"：有密钥泄露的证据，但 OneCRL 里没有吊销记录。  
## 决定  
  
Bug 里的最新一条来自 Mozilla：  
> After discussion with maintainers downstream of Mozilla's Root Store, we have decided that the circumstances warrant the emergency removal of trust in emSign ECC Root CA - G3. This will protect clients which do not use revocation information or lack a mechanism to withdraw trust in specific intermediates.  
  
  
翻译过来：在与 Mozilla 根存储的下游维护者讨论之后，Mozilla 决定紧急移除对 emSign ECC Root CA - G3 的信任。第二句给了理由，这么做是为了保护那些不使用吊销信息、或者没有能力单独撤销某张中间证书的客户端。  
  
需要说清楚的是，根证书本身的私钥并没有被攻破。移除的是信任，理由是下游客户端拿不到吊销信息，不是因为根密钥泄露。  
## 这件事真正的问题在哪  
  
不是 P384 被攻破了，也不是 HSM 被攻破了。是一段生成密钥的代码，把 384 位的随机性压成了一个字节。  
  
一场 2025 年 4 月的密钥生成仪式，八把几乎等于零的私钥，在证书库里躺了将近一年半，最后被一个外部研究者用一个下午翻了出来。中间还隔着一次吊销（A 系列）、一个已经修好的 CRL 签名 Bug，以及一份 HSM 映射表。  
  
Mozilla 在提问里写得很清楚：这件事让技术上并不高明的人也能轻松恢复中间证书私钥，所以影响不能再高。而他们真正担心的不是 Firefox 用户。那些人当天晚上就已经被保护了。他们担心的是根本拿不到吊销信息的终端。  
  
本文所有事实与引文均取自 Mozilla Bugzilla 2083449（评论 0、1、5、6、9、12、15、23 及最新一条决定），时间一律为 UTC。  
  
  
