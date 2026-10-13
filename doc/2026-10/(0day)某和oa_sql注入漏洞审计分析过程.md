#  (0day)某和oa_sql注入漏洞审计分析过程  
原创 深潜sec安全团队
                        深潜sec安全团队  深潜sec安全团队   2026-10-13 07:24  
  
## 免责声明  
  
免责声明：文中所有涉及的内容均不针对任何厂商或个人，同时由于传播、利用文中所发布的技术或工具造成的任何直接或者间接的后果及损失，均由使用者本人承担。  
  
关注公众号，输入“学习交流”加入交流群  
  
觉得不错的话，可以多点赞、分享、关注  
## 一、鉴权分析  
  
1.  
首先找到对应的登录接口程序集，进行反编译  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3zcPYCnV3SjvoC63xnrg6LZlz3Ec8K1Yc657hzR5ciaNsPoiaMwHwvmdUxnUuGjwC1rXssUNek1YWDh3QiaqAkS76kF5JEiaibmFiaUhwGbPCF74c/640?wx_fmt=png&from=appmsg "null")  
  
2.  
然后可以跟踪AjaxForLogin类，发现一些关键调用的方法  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3zcPYCnV3ShsK0tnXFr5OMYbvcpc86Lum1HvW3nme90eFnicP6mQIibOuhiay796TaqLK2p589wWzb3kVPT73oIHoDs9gBnnYATHZkyDNnj7cg/640?wx_fmt=png&from=appmsg "null")  
  
3.  
在InitVariable方法请求中取得操作类型、登录名和口令，并做用户标识映射，还会检查 AD/邮件登录相关状态。这里看到参数被读取，不代表认证成功  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3zcPYCnV3SjSn8AedRxWqqNk2NU0mURTAynhxHlibdPgO1eHOQHiaricUSBcqd9FVWphjLHb4uCtJCiauouVG8WDicj6YDXr49qOqASaDW2Nia4VE/640?wx_fmt=png&from=appmsg "null")  
  
4.  
在回头去看Action方法，发现 OpType进入Login、Exit、IsLogin等分支。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3zcPYCnV3SiaZCt0KZqyhGfjQBlicW8j61mreaTefswUz5D0gbiaF6pj9tf6sAW2RrmsI3ibE7fib1zkg0LibjU8pMv8xibUKWKiapkmHR53NsbGyPA/640?wx_fmt=png&from=appmsg "null")  
  
5.  
跟进 Login，注意它不是总会调用LoginCheck该方法先计算登录策略结果，再检查Session["SMSValidate"]、Session["NewPass"]，在其他分支调用 LoginCheck()。此时不能定性为绕过  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3zcPYCnV3SjwwILAXS7DG5rL5JnoM2cGMficPygZhku2OYUCGpQyiasHGwZzlePvESWRuOv8jcgOtLFJeialuPnAPLZVMJcAChmf5a1acBe3UQ/640?wx_fmt=png&from=appmsg "null")  
  
6.  
沿常规分支继续分析LoginCheck方法，在该方法的UserValidate分支下进行口令检测  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3zcPYCnV3SjZd5OgmQkczqqZ2kEtibib6BFuJZyFl3b7SFjicAah56GNVCrF98LvW7san0UjHBNGqPyYNIVUjesRkyUkfZu4v9UxjPCGGibWHsg/640?wx_fmt=png&from=appmsg "null")  
  
7.  
接着跟踪UserValidate方法，可以看pw的声明是怎么来的  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/3zcPYCnV3SiaLptgn9ib9PicYict0enn2YN7bthribJicd8Sha5HLDhicTES7mxZgw3lufTf0IQv7KnXgmkgtGK6YGtg0n37ugXSbERbt31CTPjjr8/640?wx_fmt=png&from=appmsg "null")  
  
8.  
接着在pw.IsPassword实现先调用Resume.EncryptPassword处理口令，再查询用户及口令相关字段；无记录时返回ERROR，有记录时还结合口令策略设置返回状态。方法名不能证明具体加密算法，也不能把所有返回值都当作简单的 true/false。（并且可以看到该处也是sql直接拼接获取对应的值）  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/3zcPYCnV3SjxqicicqAfzJGicicOY3VbBUEFAQNuwVK8ugECrhkc5lWiaE5HQwicO3LJMVp0ZQYmH2vEUEeBtCsr18ZtfNVrKzFpF65tZibgiauoQsM/640?wx_fmt=png&from=appmsg "null")  
  
9.  
然后在回到Login方法，继续点GetUserInfo，它调用pt_GetUserInfoForLogin；再回到 CreateSession，就能看到身份落到会话里的位置  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/3zcPYCnV3SgcSrhA0jo4YclY97Wia2alwXQ9Scuq0p2pibcZTtoM6ahc4Q1hiahf3jtt6wtLEqgubGQ20VIupzQcBl16wibPCW9IVibZ7JROLicp4/640?wx_fmt=png&from=appmsg "null")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3zcPYCnV3SjG0zElaSubXWBn0Lg9gh5LwyIHgSibnBOm6aicJqcBBBl8HVfmWYO9ND7ZiaW1Uhye2ibPlmfYK42bKIAwdoqaXzsEKOduWqWlPf8/640?wx_fmt=png&from=appmsg "null")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/3zcPYCnV3ShQNwkQoNtvTnDLNWCtzZ5RU441MTiaJBnbJufX9r95AzIEZ2AAFmMsZ0sBK4Yqxiccw1ewoTMMxMAKQsSmS1EZ1kmRYfUPu0xrI/640?wx_fmt=png&from=appmsg "null")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/3zcPYCnV3Sia0ypqP19SSdsDkvfibHBfvibITbV0ttribMv5onVRRSrpC9stsGAZm6aw81J365AyEvTPA2ykXEcpCodGfozFqtXjBl0tjLKrn2Y/640?wx_fmt=png&from=appmsg "null")  
  
10.  
身份存在之后，再向下追踪角色检查，然后可以确认用户ID和角色代码进入角色判断存储过程，且返回1被当作成功  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/3zcPYCnV3ShhepFwnAXzlpgLG8URBicOhevCkSlhe2JZfPbQncDzb2hnzmFH4DfDH3qTicugJqkHa5QZK4UPU9X7tdibaIUbIhlaOYE9ggq7Dg/640?wx_fmt=png&from=appmsg "null")  
![](https://mmbiz.qpic.cn/mmbiz_png/3zcPYCnV3ShMXjfayLJdicyFPxibffGwAUA3qDl03y3qH7csrUeW8uaBE2oia41D56sqb13zhh7nrMqxic9duL0FGAp5W9epVzQmlz2ar8vNqes/640?wx_fmt=png&from=appmsg "null")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/3zcPYCnV3SgS7Nblmv9xfjoObYaYvfaeGiaiaxGWYjTsPPV0AuJOsWNqbWJZrVRIuyOfShria3Tbl2M7xVzXO5MzlUfV5FHS1E1CPBicrBcvDO8/640?wx_fmt=png&from=appmsg "null")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/3zcPYCnV3Sjcnd7CfsbfZAicOxxTDkUzHFJwUwYhLuS4Rxzd9iaRPXeianlAAuCuBPiaLrju9CAvvbcl18xkJg0NVlagylVG8JAK5HSA5HeEnvA/640?wx_fmt=png&from=appmsg "null")  
## 二、审计过程  
  
搜索sql漏洞关键字，发现在该处直接拼接的。(根据前面鉴权分析，发现它没有调用RoleCtrl，也没有传入用户或角色信息给档案查询，实际也就是未授权就可以进行sql注入）  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/3zcPYCnV3Siax2hKzusicUBsic8QAgCXBeHnGPR1qHusJj96RaAgurB7gMsDuELUODrXjR4Mq5ZFpkrrSTptvDZZOymNRwxCWxfUOibOUeB420c/640?wx_fmt=png&from=appmsg "null")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3zcPYCnV3SiaK6XTVblqOXsRWG27icSgFAoOPwHImKkZfs2s5oibmWWNovAG0TLibTYQ6Onpz8srOQ7WSaCVPsNCyY2zqN3iaqeibQQ0eweO2ibEQQ/640?wx_fmt=png&from=appmsg "null")  
## 三、poc获取方法  
  
1.  
圈主平时需要参与项目、比赛、上课及其他相关工作，时间相对有限，因此审计内容及漏洞更新频率可能不会太高，部分内容的更新周期也会相对较长。如对更新速度有较高要求，还请谨慎加入。  
2.  
圈内大部分漏洞及相关技术内容均为未公开内容，请勿擅自转载、传播或对外泄露。文中所涉及的技术、工具及相关内容仅供安全研究与学习交流使用。任何个人或组织因传播、使用或不当利用相关内容所造成的直接或间接后果及损失，均由使用者自行承担。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3zcPYCnV3ShbsBVFsQNPjOXabJeE3IOkKZRNVVRNZibeqwNaN0Z4CD1TUibic7rkhF06bdpXPv7R8ztz4qnGBlyQPriab1Tic50FqxUnsEVSeKjU/640?wx_fmt=png&from=appmsg "null")  
  
