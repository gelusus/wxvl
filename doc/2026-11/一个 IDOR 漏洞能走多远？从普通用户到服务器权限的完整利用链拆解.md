#  一个 IDOR 漏洞能走多远？从普通用户到服务器权限的完整利用链拆解  
原创 暗夜铭少
                    暗夜铭少  黑帽渗透技术   2026-10-31 23:41  
  
IDOR  
越权链  
JWT伪造  
GetShell  
权限维持  
付费合集 · 第 07 篇  
  
很多师傅挖到越权，交个 SRC，拿个低危，就结束了。  
  
但如果你手上那个 user_id=1002  
 的越权，背后连着的是一整套 **JWT 伪造、后台 GetShell、权限维持**  
 的完整链路呢？  
  
今天这篇文章，我拿一个**完整的、可复现的靶场级种子**  
来拆。不是讲“IDOR 是什么”，而是讲**一个低危越权如何被放大成服务器沦陷**  
。这篇是收费合集的其中一篇，我会把每一步的**判断逻辑**  
和**踩坑点**  
都写清楚，你能直接拿去改自己的利用链。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/VqJrJwib7Fo3biaorR5B6Nz2VKbEh7uibfVCjtDMS6vYFPm4uTib2GTPlnoKycBnnrPnoEDy8piaJfXs6bkbfxWoOYYWQ1mxEpFd3rWe0Pd3XH50/640?wx_fmt=png&from=appmsg "")  
  
