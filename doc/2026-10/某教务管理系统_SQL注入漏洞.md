#  某教务管理系统_SQL注入漏洞  
原创 深潜sec安全团队
                        深潜sec安全团队  深潜sec安全团队   2026-10-16 02:15  
  
## 免责声明  
  
免责声明：文中所有涉及的内容均不针对任何厂商或个人，同时由于传播、利用文中所发布的技术或工具造成的任何直接或者间接的后果及损失，均由使用者本人承担。  
  
关注公众号，输入“学习交流”加入交流群  
  
觉得不错的话，可以多点赞、分享、关注  
## 一、漏洞描述  
  
某教务管理系统存在SQL漏洞，攻击者可利用此漏洞获取数据库信息。经过审计发现参数未经过滤直接拼接到sql查询语句中，可能导致企业业务数据、用户账号密码、供应链信息等敏感数据泄露。该漏洞无需登录即可触发，利用门槛低，影响面广。  
## 二、漏洞复现  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3zcPYCnV3SjwKYw6yRUOCU3ibIVlwJy942tuDBnibgsPN2mJkVlJUmaAvNWxUeoDXpSMkSiciaFVye7VxmInwAffWUQc3jyJaco5OeQuPMZe3DU/640?wx_fmt=png&from=appmsg "null")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/3zcPYCnV3SjyCPkCgMYNe7BucFynwHQ6icOpqncIPFDLU34c5LorUjqGzfXF8qInW9r1IHTD2O2cZLpR7o1rpFLZtjruQM7iaQalIrPUg8QWA/640?wx_fmt=png&from=appmsg "null")  
## 三、poc获取方法  
  
网络安全漏洞N/0day分享、加入纷传请扫描下方二维码，更多精彩内容，敬请期待！  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/3zcPYCnV3ShBF9prvfQWsZxBLex7zLgWPP5NstQlicCSdJr4jEz8KuwthtIlveV3XXBtFEDnPUgGnDST15EvXK6z3495xmjvPaF0026QLII0/640?wx_fmt=png&from=appmsg "null")  
  
