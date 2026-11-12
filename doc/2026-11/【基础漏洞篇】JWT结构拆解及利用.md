#  【基础漏洞篇】JWT结构拆解及利用  
原创 念三千
                    念三千  三千网安   2026-11-12 10:00  
  
## 1.什么是JWT？  
  
JWT全名JSON Web Token，是一种登录凭据，类似于cookie，目前来说主流网站大部分已经不再使用cookie进行身份校验了，使用JWT进行校验。在请求头中位于Token，Authorization，X-Access-Token等字段处。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jk8WT81UN1AVOIqRt1H9ibxQXBCfUvVKyJ6JFGSxdVicqSOKiaQuKCmLz4bsYd4IicylLTlA6FQebC0amiaKaGqCnSBmA9JibNT9BjKMXBsH8niaf4/640?wx_fmt=png&from=appmsg "")  
  
## 2.JWT结构拆解  
  
JWT字段分为三部分，第一部分为数据头，第二部分数据体，最后是签名。每一段都用点号隔开。数据用base64进行加密。  
  
数据头一般情况下不需要关注，主要是数据体和签名。  
### 2.1、数据体  
  
数据体中包含认证用户的信息，后端通过该信息来辨别用户。如果能够篡改的话，即可伪造任意用户身份登陆系统。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jk8WT81UN1CXSRtb9aRyPjbJuic1JvmHtAs4iaicIjuZEmdj6xEPy6UibLOZLE9nErakrlqpW1WzrvAht6IRUYo3e8VFuicsB2OsibysmpgLSdD8g/640?wx_fmt=png&from=appmsg "")  
  
### 2.2、签名  
  
前面所说如果数据体能够进行篡改，就能伪造用户，但JWT自身也有自己的安全机制，仅靠base64进行加密的话并不安全。因此签名主要是保证数据的完整性。签名通过对前面数据，使用后端的密钥进行加密，服务端获取token后便会进行比对。使用密钥对数据体进行重新加密，和签名进行对比，如果不一样则判断数据体被修改过，判定凭据无效。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jk8WT81UN1BwicDPwkkGicW13I34YiaibF1961iaWGPPodmMXOlX6M0wONnqicahricqxia9MGJiaCnRF9Yb0Diaf6T2PgmN9Cxo4jfAMZfgQT3GX20vg/640?wx_fmt=png&from=appmsg "")  
## 3.JWT漏洞利用  
  
以下是较为常见的JWT认证存在缺陷的利用方式  
### 3.1、签名未校验  
  
在部分情况下，后端因为代码缺陷导致没有校验签名，因此无论签名存在与否，都会通过校验。所以直接修改数据体即可。但是要保证JWT的格式完整。  
  
加解密网站：https://www.jwt.io/[#debugger]()  
  
  
不删除签名直接修改数据体  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jk8WT81UN1ApAqXJIB6JfvicvsAH14jzic7GL5v7wHib5TuLSo0qO3Uun4ChWUIGOtFicmXRXrUPiaeXtPG1zMVkpcicCMrpUC3PRqxPbuLLbxulQ/640?wx_fmt=png&from=appmsg "")  
  
  
删除签名修改请求体（末尾点号需要保留，以保证格式正确）  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/jk8WT81UN1BEzDia2YPiaTTCwiaWj59XAUNzlyMErrbZKSPm8Hb8u60NhROdTuyhAN8Z9F9WiarIwpR2l2lwrdMqZdnKhsIssicjm3x3Czan6z7g/640?wx_fmt=png&from=appmsg "")  
  
删除签名后变成  
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InNhbnFpYW5zZWN0ZXN0Iiwicm9sZSI6InVzZXIiLCJjbGllbnRUeXBlIjoiUEMiLCJpYXQiOjE3OTQzOTg3NDYsImV4cCI6MTc5NDQwMjM0Nn0.
```  
### 3.2、密钥爆破  
  
如果后端密钥为弱密钥，比如：abcdefghijk，abcdefg123456这样的可通过字典进行爆破（密钥在字典中存在才能爆破出来）  
  
使用Tscan中的JWT工具可进行爆破，设置下字典即可点击开始  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jk8WT81UN1C8rFibhiadk6Ufg9TpVpXdJBG5NYkc4iaoAWbupQ2F8hk240CphleZhYcktpWKMiaOlWAibt6j8xP3ZPDwpXW1mic1hInV0TkiblaBYs/640?wx_fmt=png&from=appmsg "")  
  
  
### 3.3、密钥硬编码  
  
一些通用框架，加密的密钥是写死的，因此后端如果部署系统没有修改相关配置的话即可直接利用默认密钥伪造凭据。最经典的就是Bladex的历史漏洞。  
  
bladex系统后端密钥为：  
```
bladexisapowerfulmicroservicearchitectureupgradedandoptimizedfromacommercialproject
```  
  
系统运维用户如果没有进行修改即可直接使用该密钥在网站（https://www.jwt.io/）上进行加密，获取凭据登录系统。  
  
引用文章：https://mp.weixin.qq.com/s/S9QsCMT6OEsVQ_FQgWQNfg  
  
![](https://mmbiz.qpic.cn/mmbiz_png/jk8WT81UN1A92ALP6FGQJ3P22ef5lq9bI0y3MXzb8qAUpQAo9smRIYZxOUstOECbqWQDpFVMms8icqJuOdx1MWj3frKpTyrcUPFmicZ0cZbPM/640?wx_fmt=png&from=appmsg "")  
  
## 4.最后  
  
创了个新群，如果你对网安感兴趣可以加入交流  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/jk8WT81UN1BH1cc5pnNrWBgH1EWBvqxZYrRDu83PfIzOZC1jKZiaWxkVXuCqxsWibjI9wJk2oVEyb9ZOZxVm13kicBCeKwNbyfFJXn6gbAvROo/640?wx_fmt=png&from=appmsg "")  
  
  
  
  
