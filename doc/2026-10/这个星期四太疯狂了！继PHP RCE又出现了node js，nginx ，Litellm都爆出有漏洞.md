#  这个星期四太疯狂了！继PHP RCE又出现了node js，nginx ，Litellm都爆出有漏洞  
湘南第一深情
                    湘南第一深情  湘安无事   2026-10-21 17:49  
  
继有学员再群里面问  
php最新爆出来的  
CVE-2026-109619  
```
官方地址：https://www.php.net/index.php#2026-10-20-1
网传poc:https://github.com/9bie/php2shell-bypass-aslr-exp
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9EtBPS5uuicKoaddzlsSDHHvVUg5wz7yTLhg5s91EiadlqhkAEfIBBicVoInSyiaDHXq2BUyt9cexZG86rJGa57JgCt5FL87bKrrd5I/640?wx_fmt=png&from=appmsg "")  
  
刚开始这个漏洞影响力很广呀，百度都上热搜了，微步&长亭&sxf都发了通稿  
  
![](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9Ev4fic5FCIK4Fej7ZiagFjuekMs6lah0iblHu6aZKbF5A5abPsgqhucicM4PmIlDQzopAEkLOUWn62qtZiaNYPOibI2tV70J2xiaUI8vs/640?wx_fmt=png&from=appmsg "")  
  
发现国外早就传开了，如图  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/tlibgKYKL9EuVcNMPpFiacZdRMPFYVa4icJpgoicvI10phhuWWMicicmcYS21iawVpRxW2QwfVsuvRibxOibn8TZ7zv5gcRwXIhwmLyIc6zX5buLuuxo/640?wx_fmt=jpeg&from=appmsg "")  
  
网传修复方式  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9Espy6rTGGSku2byHJPeWd0r8f9o2xwtTaXO8Phg58B4jYmGT7Ft7jzbhAqbgaueFibGnaibJ1y1MRoyzQrNzJHVmSKS8QceysJicY/640?wx_fmt=png&from=appmsg "")  
  
  
突然紧接着爆出了更多漏洞比如node js，nginx ，Litellm都爆出有漏洞，还让不让sqg睡觉了  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9EvyKwZTSZ8GymiaibwQOUqCBU2ZxwK9iax38BNjAeSGwkk71shIdeys3fdI9rl4X7zBhiboNmelUbtuJqqEfrjmd6wic8tpXFcuKtJQ/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9EstErp9CaZhnE4ttwr7lh48JVx5USmj97O0swS5Gq8VxZMFia5GiciaJ6KxulHMPhIVhOOpu5uetmO1E45JqBzHV6PsBfuhGLoDZM/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/tlibgKYKL9EsAzPSQXibNibFYU9oNBtD0zRypJOD2icfkoyiaAaOrxlbgDhmsE5hOBdv7PLOwPOn1Auj9GPFqVPbLicP464CEzOjDibPOnEDUoia9cI/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/tlibgKYKL9EuxbbPtzoJma5VvEC3QkWhUBiaHVibPaia1yY3iaWNswNfPOOicSXYyM0TP3R8m7aRMKialKB5Cboo2QKqWVoibu4ibH0ZUfiakrALouwLI/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/tlibgKYKL9Euo3d87foicwkGuZ4cKynp4ibXlGicvrFAibAaRE7YS3QajFXjSwcQfFapibfur1LibZKKTsT5OOux4g1vJf8ndOaqdTiczUmL6q7dAvI/640?wx_fmt=jpeg&from=appmsg "")  
  
就想知道php rce漏洞能不能无条件rce或者能找打内存地址，然后又突然一堆漏洞爆出来，这是蹭热度还是啥呀，有点懵了。谁来告诉深情哥，算了睡觉就当疯狂星期四过年了  
# 最后总结感兴趣的可以联系深情哥进群，有公开课会在群里面通知，包括审计和src。edu邀请码获取，咨询问题，hvv渠道推荐，nisp和cisp考证都可以联系深情哥。内部edu+src培训，包括ai渗透，src挖掘，edu挖掘，小程序逆向，js逆向，app渗透，导师是挖洞过50w的奥特曼深情哥，edu上千分的带头大哥！！！联系深情哥即可。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/tlibgKYKL9EsuyO0E5LibkFmuuFv4259LqOcfH30CyI9zwb9LB2boPBFcTLOM6G53z68PBC7CFzCuhkta10xSbxWOhxxNp5QziboaT461qSDKY/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1&retryload=1#imgIndex=19 "")  
  
