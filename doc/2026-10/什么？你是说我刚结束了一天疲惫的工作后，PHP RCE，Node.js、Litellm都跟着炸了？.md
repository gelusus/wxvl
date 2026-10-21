#  什么？你是说我刚结束了一天疲惫的工作后，PHP RCE，Node.js、Litellm都跟着炸了？  
xiachuchunmo
                    xiachuchunmo  银遁安全团队   2026-10-21 16:39  
  
    在PHP曝出RCE漏洞之后，  
Node.js也  
发布全版本线安全更新，26.x/ 24.x/ 22.x（LTS）全活跃分支均受高危漏洞影响  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/14zTrTiaCfEVEyWdxop86eAhcoC75ZotNzSIkOBM62SlnarNAq91vmChJoHxiaaeAUuDIRR5vMx7Prze5x05GplTNQ98Ae3SSRAhicoCzGPU2k/640?wx_fmt=jpeg&from=appmsg "")  
  
那真是很糟糕的一天了..  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/14zTrTiaCfEWWFuKHV5ul9YqGVAE09Sic29oLIPwOkMV4WhehUIzmEPOBicibShO7QdMc5K2iccYUSnmNQ0Vy8581ib1icuZ37YMDhC5Ue0H79iavKk/640?wx_fmt=png&from=appmsg "")  
  
什么？你是说我刚结束了一天疲惫的工作后，PHP RCE，Node.js、Litellm都炸了？  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/14zTrTiaCfEX9KOFnowSpCtghVR7roZUT9lH5icDCicytf8NfMIqNyp9YBsicEB9MvCIQuWDPHswicCC9253OcB4hoib8BVoj47a2ylj8pC5G1cRI/640?wx_fmt=jpeg&from=appmsg "")  
  
  
      
想象一下你的架构是 nginx 反向代理，后面挂着 PHP 8.1 主站、Node.js 24.x Brotli 网关、Go 微服务、Rust TLS 校验层、Python + LiteLLM 1.107.0 AI 中台，全部跑在 Linux 内核 c6e50df8 之前，HTTPS 证书由 eMudhra 签发并通过 emSign 交叉认证，根证书库用 Go crypto/x509 + Rust webpki-roots + Python certifi，CI 昨晚刚从 PyPI 拉了新包。  
  
    真是美妙的一天......  
  
