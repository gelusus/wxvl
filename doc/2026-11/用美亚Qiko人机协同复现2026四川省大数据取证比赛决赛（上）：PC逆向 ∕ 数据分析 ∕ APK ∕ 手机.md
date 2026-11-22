#  用美亚Qiko人机协同复现2026四川省大数据取证比赛决赛（上）：PC逆向 / 数据分析 / APK / 手机  
原创 ljnljn
                    ljnljn  ljnljn学习之路   2026-11-21 17:48  
  
容器密码：努力一把AI赋能取证@400DBW/sc   
  
容器下载地址：  
https://pan.baidu.com/s/1Q_AG_AE1qUuW8ajApJhwWQ?pwd=dch7  
  
（没有人觉得这个密码很诡异吗）  
  
今天也是完成了四川取证比赛的决赛，这里先使用Qiko来复现，夹杂着手搓和Qiko自动分析  
  
用Qiko加载检材时就发现了RAID5，不过似乎RAID自动重组有点问题，还需要手动导出一下   
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/3Bkxl9ud7XNy9NxPmmZX7z1g3yVlEMpOtt6jtWqgw5b3VZ7c7ibptHw4upa9XOYFlYuh4FeBguwJA9Wmiaacia5rOiclFU0gnSUL0InweUXCW84/640?wx_fmt=png "")  
### PC逆向 (8个小题, 共92分)  
#### 1. [填空题]通过对检材-PC逆向中“环球财源用户管理中心v2.rar”分析，可执行程序文件SHA256校验值的前8位是(大写字母与数字组合，格式： F34F21F3) (4分)  
  
BD0DBC02   
  
这个应该没啥说的吧，直接对exe进行哈希校验即可  
#### 2. [填空题]对可执行程序文件取证分析，本地用户信息的文件加密密钥是(按实际值填写) (12分)  
  
对于逆向来说直接交给Qiko自己做就行，本人实在是不太会（） ![](https://mmbiz.qpic.cn/mmbiz_png/3Bkxl9ud7XNJdHr8IoXrMtedaia8kGgxX5mxm8IQxG5chnTYxCBSIQicjK8hgZ0BezYT33JuED2Bmgak9GnRs14dicEeVBy23V9W78UOyaYIYg/640?wx_fmt=png "")  
  
#### 3. [填空题]对可执行程序文件取证分析，可直接用于解密本地用户信息文件的解密工具的MD5(128bit)校验值是？(大写字母与数字组合，格式如：E26EE9E) (13分)  
  
**24A31C476EF63A669E3699473510BF24**  
   
  
RCDATA 资源 TOOL  
（res/10_TOOL_1033.bin，1,564,160 B）整体用字节 ^0x3C  
 还原 → 有效 MZ/PE（32 位 Delphi VCL、ASPack，字符串 SqliteTool  
/TPASSWORDDIALOG  
），即“可直接解密本地用户信息文件的工具”，对该明文 PE 计算 MD5。  
#### 4. [填空题]对可执行程序文件取证分析，本地用户信息文件的解密工具的加密密钥是？(十六进制，0x后面大写字母与数字格式，格式如：0xA2) (12分)  
  
0x3C   
  
同上，TOOL 资源逐字节异或密钥 0x3C  
 即得可执行文件。  
#### 5. [填空题]对可执行程序文件取证分析，用户登录时密码校验文件解密后文件前8字节是 (十六进制，0x后面大写字母与数字格式，格式：0x010AD030405060708) (13分)  
  
**0x53514C6974652066**  
   
  
本地密码校验库即 RCDATA DB  
（110592 B = 27×4096，FireDAC 页级加密，页保留 32 字节，TFDCipher9.ReserveLength=0  
）。用工具以 abcd1234  
 打开后从进程内存取出解密页 1：SQLite format 3\0  
 + page size 4096 + reserved 32 + pagecount 27 ⇒ 前 8 字节 53 51 4C 69 74 65 20 66  
（"SQLite f"）。  
#### 6. [填空题]对可执行程序文件取证分析，本地用户信息的文件中用户姓名的密钥是(十六进制，0x后面大写字母与数字格式，格式如：0xA2) (12分)  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3Bkxl9ud7XOb7Cz9X4CH6zRCpbnFQibibzibd5AGfElibT0x16Oj56Kvz0LppSoNJAvYaYWHHvej23SAJeR8gqA6gmzvBPK1bJK2mGicEXEztq5k/640?wx_fmt=png "")  
#### 7. [填空题]对可执行程序文件取证分析，报案人“周林”的邀请人身份证ID分别是？(格式：351381xxxx04025863) (13分)  
  
530321xxxx06111519   
  
（xxxx=1992）  
  
这个需要打开程序操作分析，所以我用的让ai调用vmware cli去执行有关操作 **① 在 user_extents 中用 周林 的密文（F4B39BA2CBF1）定位记录，查到 uid=10020、inviter_id=136**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3Bkxl9ud7XOMUygnInSU9XYPjjiaoBkMfgpFGo8v8HocT8J2Hu0h2UE4whHYicEXKPQgvKGYxBUF6GUgSeYShmwJuHNpdo4CKL7S8sMlpmicoY/640?wx_fmt=png "")  
  
SQL：select uid, inviter_id, hex(real_name) as rn from user_extents where hex(real_name) like '%F4B39BA2CBF1%'  
（周林  
 的 UTF-8 E591A8E69E97  
 与 Q6 密钥 112233445566778899  
 循环异或得到 F4B39BA2CBF1  
）  
  
**② 查询邀请人 uid=136，姓名解出为“宋涛”（F48CB8A2E3FD 异或密钥 = E5AE8BB6… → 宋涛），但 id_number 列位于网格右侧、被滚动条挡住**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/3Bkxl9ud7XNjPoIKXc1qchPeXW3fw08bNfJknwPuF8rkL7aEdTb3lF7llJdBG8TfK2Oryqz3k230mSGILKKUvM3WPI3vC17MfUXobyhl04w/640?wx_fmt=png "")  
  
**③ 直接查看 id_number 的存储原值（变换后的 TEXT，肉眼看起来是乱码，说明必须按算法还原）**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3Bkxl9ud7XPqkbuRmnUp3CzdbPK6udyic3m1eN56TQnsqq4pMsABVw9rRj3okm2Ex2AW8dFsOWCVhVe9hNQF692vTk4hExhfMJmlmlwS27Ik/640?wx_fmt=png "")  
  
**④ 把 id_number 拆成两段，输出 18 字节 hex（避免列宽截断）：64625F626160606868 | 615F65606060646068**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3Bkxl9ud7XPWFUsCN9AXVWgicxSo8VeClMsPOOFMPLwmFvtKcAlOzwic04ZxHibE2DT9C0S6cMpXNMGGXnc1CwdKEdE8kf2xyJeCay8EBVYGDQ/640?wx_fmt=png "")  
  
**⑤ 交叉复核：length(id_number)=18、第 18 字符码值 =104、hex(substr(id_number,15,4))=60646068（与④的第 15–18 字节一致）**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/3Bkxl9ud7XMB3EqdZHgmGv4PDVCP0aXl6ElOz1ZjrQcgzMd2WdYib5FXuLU9wQNB13GTOLzdHicnTaTXY1rZBYZtUQpEYVNDgJ2f0AzH7LJ7M/640?wx_fmt=png "")  
  
**还原计算**  
（sub_116EDD0  
 的 p = (c+14) % 0x5E + 33  
，脚本在本机运行）：  
  
<table><caption><section><span leaf=""><br/></span></section></caption><thead><tr><th style="padding: 9px 12px;border-width: 1px;border-color: rgb(216, 216, 216);background-color: rgb(240, 240, 240);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">序号</span></section></th><th style="padding: 9px 12px;border-width: 1px;border-color: rgb(216, 216, 216);background-color: rgb(240, 240, 240);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">1</span></section></th><th style="padding: 9px 12px;border-width: 1px;border-color: rgb(216, 216, 216);background-color: rgb(240, 240, 240);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">2</span></section></th><th style="padding: 9px 12px;border-width: 1px;border-color: rgb(216, 216, 216);background-color: rgb(240, 240, 240);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">3</span></section></th><th style="padding: 9px 12px;border-width: 1px;border-color: rgb(216, 216, 216);background-color: rgb(240, 240, 240);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">4</span></section></th><th style="padding: 9px 12px;border-width: 1px;border-color: rgb(216, 216, 216);background-color: rgb(240, 240, 240);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">5</span></section></th><th style="padding: 9px 12px;border-width: 1px;border-color: rgb(216, 216, 216);background-color: rgb(240, 240, 240);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">6</span></section></th><th style="padding: 9px 12px;border-width: 1px;border-color: rgb(216, 216, 216);background-color: rgb(240, 240, 240);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">7</span></section></th><th style="padding: 9px 12px;border-width: 1px;border-color: rgb(216, 216, 216);background-color: rgb(240, 240, 240);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">8</span></section></th><th style="padding: 9px 12px;border-width: 1px;border-color: rgb(216, 216, 216);background-color: rgb(240, 240, 240);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">9</span></section></th><th style="padding: 9px 12px;border-width: 1px;border-color: rgb(216, 216, 216);background-color: rgb(240, 240, 240);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">10</span></section></th><th style="padding: 9px 12px;border-width: 1px;border-color: rgb(216, 216, 216);background-color: rgb(240, 240, 240);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">11</span></section></th><th style="padding: 9px 12px;border-width: 1px;border-color: rgb(216, 216, 216);background-color: rgb(240, 240, 240);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">12</span></section></th><th style="padding: 9px 12px;border-width: 1px;border-color: rgb(216, 216, 216);background-color: rgb(240, 240, 240);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">13</span></section></th><th style="padding: 9px 12px;border-width: 1px;border-color: rgb(216, 216, 216);background-color: rgb(240, 240, 240);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">14</span></section></th><th style="padding: 9px 12px;border-width: 1px;border-color: rgb(216, 216, 216);background-color: rgb(240, 240, 240);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">15</span></section></th><th style="padding: 9px 12px;border-width: 1px;border-color: rgb(216, 216, 216);background-color: rgb(240, 240, 240);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">16</span></section></th><th style="padding: 9px 12px;border-width: 1px;border-color: rgb(216, 216, 216);background-color: rgb(240, 240, 240);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">17</span></section></th><th style="padding: 9px 12px;border-width: 1px;border-color: rgb(216, 216, 216);background-color: rgb(240, 240, 240);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">18</span></section></th></tr></thead><tbody><tr><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">存储 hex</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">64</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">62</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">5F</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">62</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">61</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">60</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">60</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">68</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">68</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">61</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">5F</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">65</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">60</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">60</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">60</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">64</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">60</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">68</span></section></td></tr><tr><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">还原</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">5</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">3</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">0</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">3</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">2</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">1</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">x</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">x</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">x</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">x</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">0</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">6</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">1</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">1</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">1</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">5</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">1</span></section></td><td style="padding: 9px 12px;border-color: rgb(216, 216, 216);font-size: 0.75em;line-height: 22px;color: rgb(34, 34, 34);vertical-align: top;"><section><span leaf="">9</span></section></td></tr></tbody></table>  
  
→ 530321xxxx06111519  
；GB 11643 加权和 267、267 % 11 = 3  
 → 校验码 9  
，与第 18 位相同，**自洽**  
。  
#### 8. [填空题]对可执行程序文件取证分析，包含数据同步的网络地址的文件密钥是？(按实际值填写) (13分)  
  
**pwd!@#qaz**  
  
IP  
 资源解包得 cygwin1.dll / mysecret.jpg / og.exe  
，代码 sub_116FEA0  
 执行 og.exe -k "pwd!@#qaz" -r mysecret.jpg addr.txt  
（.text 文件偏移 0xD6FB30），读回 addr.txt  
 显示于界面 —— 含数据同步网络地址的文件是 mysecret.jpg  
，口令 pwd!@#qaz  
。  
### 数据分析 (8个小题, 共90分)  
#### 9. [填空题]某市最近发生多起盗窃案件，经过现场勘察走访，公安机关侦察思路为先摸排出案发时间出现在案发地点附近的人员，以缩小侦察范围。现筛选出了【01案发地手机基站轨迹数据表】，请分析出在2026年5月1号、2026年5月3号这两天都有出现在案发地周边的可疑人员总数？（本案例不同的手机号码代表不同的人员，数据表没有重复记录、脏数据，“接入时间”字段格式都为yyyy-MM-dd HH:mm:ss，直接按要求做题即可） (11分)  
  
数据分析我也偷懒了，直接交给Qiko ![](https://mmbiz.qpic.cn/mmbiz_png/3Bkxl9ud7XPTB4wQ5kpN9J6iaXzjRrKXcJEvfvM3XavTh4gmBRqHRADUlW8rCPQjiaMJAIKKsOeiaQsrOxicAwqIWl0HtKWnuOf5lnQ9sWuwiagA/640?wx_fmt=png "")  
  
#### 10. [填空题]当前新型网络犯罪持续高发、猖獗蔓延，部分涉诈人员出境至境外因护照时限问题无法正常入境，故选择偷渡回国。现根据【02出入境信息表】和【02住宿记录信息表】找出出境后疑似偷渡回国人员总数？（“偷渡回国”指最后一次出入境记录是出境且之后没有入境记录，但在国内有住宿轨迹信息（例如张三最后一次的出入境记录是2025-01-01出境，在2025-01-01以后至现在建模时都还没有入境记录，但是却在2026-02-10有入住酒店的住宿记录，则张三即为疑似偷渡回国人员）；该表无重复数据，入住时间字段格式均为yyyy-MM-dd HH:mm:ss；答案填写数字） (11分)  
  
**5**  
   
  
最后一次出入境为“出”且其后仍有住宿的证件号。查 02出入境人员信息表  
 取每人 max(出入境时间)  
：24 人中 20 人末次为“出”（另 4 人末次为“入”，排除），再与 02住宿信息表  
 按证件号（upper()  
 归一，1 个号码为小写 x）取 入住日期 > 最后出境日期  
：命中 5 人——350825xxxx03215009(05-08出→05-25/06-09住)、350825xxxx1102501X(05-07→05-26)、350825xxxx06055011(04-30→05-05)、350825xxxx07085025(04-15→05-08)、350825xxxx01100012(05-25→05-31)。另 3 人有住宿但日期在出境之前，不计。  
#### 11. [填空题]根据情报部门线索推送，部分涉诈前科人员被处理后，继续从事涉诈犯罪，该类人员通常到达云南后，偷渡出境。某地反诈中心已经梳理出本地涉诈前科人员两张表【03-2024年涉诈前科人员信息表】【03-2025年涉诈前科人员信息表】（两张前科表需要去重），请结合【03铁路出行信息表】分析出在2026年有乘车到昆明站的涉诈前科人员总数？(本题需注意数据处理：【03-2024年涉诈前科人员信息表】中由于人工录入的原因，导致部分的“证件号码”字段的值存汉字的情况，如：张350825xxxx03025015、35吴 082519某960623500X，需先用将汉字清除以便后续关联；直接按要求做题，答案提交数字） (11分)  
  
**3**  
  
两前科表 UNION  
 去重得 208 人（2024 表 6 个脏证件号已用 re.sub(r'[^0-9Xx]','')  
 清汉字：张三丰350429xxxx10011223、420704xxxx06261234李、320830xxxx05051289赵、陈真341024xxxx03209230、362203xxxx10151234王某、3604251李96607103236）；与 03铁路出行信息表  
 中 发车日期  
 属 2026 年且 到站  
 含“昆明”的 72 行比对，命中 3 人：360425xxxx07103236(K178,2026-01-09)、460006xxxx02111234(K176,2026-04-07)、510213xxxx12261234(K177,2026-05-08)。2026 年铁路数据中的前科人员共 3 人且全部到昆明，“昆明南”1 行属非前科人员。  
#### 12. [填空题]“同行同住”是公安机关在轨迹类建模中常见的分析方式。现需要根据铁路出行轨迹分析同行人员，请根据《04铁路乘客信息表》《04涉诈前科人员信息表》，请找出存在和涉诈前科人员同一天同车次同出发站同到达站同车厢同排出行且座位左右相邻的非涉诈人员总数？（本案例座位001A代表坐在第一排的A位，座位号由数字和字母组成，数字代表排数，字母代表在该排的座位，例如涉诈前科人员坐在C的座位，则与他同排座位为B和D的与C座位相邻；本案例没有重复数据直接按要求做题即可，答案填写数字） (11分)  
  
13   
  
按  
“同车次+同日+同发站+同到站+同车厢+同排+座序相邻”关联涉诈前科 11 人的乘车记录与非涉诈乘客。关键口径：该表座位字母只有 A/B/C/D/F（无 E 座），同排座序为 A-B-C-D-F，相邻取该序列内前后一位（题面“C 的相邻是 B 和 D”即此规则；数据中 015D 两侧确为 015C、015F，014D 旁为 014F，佐证 D 与 F 视为相邻）。得相邻座位对 13 组，对应 13 名不同非涉诈人员。若按“字母 ASCII 步长=1”的严格口径（D↔E、E↔F，E 座不存在则 F 无邻），则为 9 人——两种口径的差额全部来自 D-F 邻座，建议以 13 报，9 作备注。  
#### 13. [填空题]某地在打击境外入境卖淫女性人员时，发现这类人群通常以旅游名义入境，入境后从事卖淫活动。现有【05旅游签注类出入境信息表】和【05境内外国人住宿信息表】，通过分析已打击人员，发现他们具备以下特征（注意：两表存在完全重复数据，请先将重复数据清洗掉；时间格式全部为yyyy-MM-dd；请严格按题目规则定义做题）： 特征1：女性年龄大于等于20小于等于45（直接使用性别和年龄字段判断即可）； 特征2：最后一次入境逗留时长大于等于3天（最后一次有入境但是没有出境的这部分人员不在考虑范围；所有人员除了最后一次可能未出境，其它次全部是有入就有出；最后一次入境逗留时长的计算方式为：最后一次入境和出境的年月日差值加1，例如最后一次入境是2025-08-01，最后一次出境是2025-08-09，则逗留时长为年月日差值8+1=9天）； 特征3：最后一次入境后，在境内住宿时长占境内逗留时长小于等于40%或者最后一次入境后无住宿记录（举例说明：例如某人最后一次入境是2025-08-01，离境时间是2025-08-09，则考虑其在2025-08-01(含)~2025-08-09(含)期间住宿天数占其逗留时长的占比；住宿天数计算：入住日期与离店日期的年月日差值，例如某人最后一次入境为2025-08-01，离境为2025-08-09，在逗留期间其一共有三次住宿记录分别为（2025-08-01入住、2025-08-03离店）、（2025-08-03入住、2025-08-03离店）、（2025-08-05入住、2025-08-09离店），则其住宿时长计算为：2+0+4=6天，则境内住宿时长与境内逗留时长占比为6/9约等于66.66%；若最后一次出入境期间无住宿记录也符合条件；为了简化大家计算请严格按上述定义）。 请分析满足所有特征的疑似入境卖淫女总数（答案只填写数字）  
  
**201**  
  
两表先去整行重复（16252→16223、19137→19108）。05旅游签注类出入境信息表  
 无出入标志列，按题设“除末次外有入必有出”结构，对每人按 出入境时间  
 排序后按 入/出 交替配对（首条为入境）；并用间隔分布反证：偶位间隔（入境→出境，即境内逗留）中位数 22 天，奇位（出境→入境）64 天，方向成立。漏斗：3906 人 → 女且 20≤年龄≤45 且末次入境有出境 923 人 → 末次逗留(出境−入境+1)≥3 天 884 人 → 特征3（窗内 入住日期∈[入境,出境]  
 的 Σ(离店−入住)  
 天数 ÷ 逗留天数 ≤40%，或无住宿记录）**201 人**  
（有住宿 96、无住宿 105）。年龄取最后一次入境行的年龄字段。  
#### 14. [填空题]近年来，中国公民出境从事电信网络诈骗的违法犯罪行为日趋严重。该类人员频繁出境至涉诈高危国家，以“旅游”等名义入境至窝点后从事电信网络诈骗工作。现已筛选出全是前往涉诈高危国家的【06出入境人员信息表】，分析出至少往返涉诈高危国家次数大于等于5次，且所有往返的逗留时长都大于等于30天的疑似涉诈金主总数？（特别说明，本案例的往返定义为：某次的出境和该次的入境构成一个往返；逗留时长的定义为：某次的出境和该次的入境的年月日的天差值，例如某人2026-01-01 16:10:10出境，2026年01-13 10:10:10入境，则该次逗留时长为12天。每个人除了最后一次出境可能没有入境以外，其它的出入境中都是有出就会有入，最后一次是出境未归的出境不参与计算。本题没有重复数据。） (11分)  
  
**182**  
  
06出入境人员信息表  
 按人排序，出(2)→入(1) 成对构成一次往返，末次出境未归不计；逗留=入境日−出境日（天差，不加1）。数据中无相邻同向记录（结构严格交替）。800 人中往返≥5 次且每次均≥30 天者 **182 人**  
。  
#### 15. [填空题]根据已筛选出的矛盾纠纷类的报警数据【07家庭矛盾纠纷接处警信息表】，统计分析出至少报警3次且前三次的报警记录中相邻的两次报警时间间隔都小于等于24小时（24小时等于86400秒）的报警人员总数？（注：例如李某的前三次报警时间分别为：第一次2023-05-01 10:10:10、第二次2023-05-01 15:10:10、第三次2023-05-02 07:10:10，可知其第一次报警时间跟第二次报警时间间隔在24小时内，第二次报警时间跟第三次报警时间间隔也在24小时内，所以李某符合上述要求，时间格式都为yyyy-MM-dd HH:mm:ss，直接按要求做题即可，答案填写数字） (12分)  
  
**7**  
  
07家庭矛盾纠纷接处警信息表  
 按报警人升序取前三条，要求 (t2−t1)≤86400 秒  
 且 (t3−t2)≤86400 秒  
，且报警次数≥3。174 名报警人命中 7 人（如 371726xxxx09025552 6 次，前三次间隔 25816s/76107s）。  
#### 16. [填空题]出于网络安全和数据合规监管的要求，现需要对频繁访问境外网站人员进行预警分析。请根据【08境外网站访问日志表】分析出存在至少连续5天访问境外网站的不同用户总数？（本题年月日相同则为同一天，该数据表没有重复记录；不要考虑网站格式，假设所有境外网站格式都是正确的；时间格式都为yyyy-MM-dd HH:mm:ss，直接按逻辑做题，答案提交数字，如88） (12分)  
  
**64**  
  
08境外网站访问日志表  
 先按用户取“访问日期”去重集合，再求最长连续自然日长度，≥5 天者计。500 用户中命中 **64 人**  
（最长连续 5～7 天）。  
### APK (8个小题, 共82分)  
  
apk所给的是雷电模拟器的备份文件，这个其实是zip形式的压缩包，里面压缩了两个vmdk和一个配置说明 ![](https://mmbiz.qpic.cn/mmbiz_png/3Bkxl9ud7XOvIc5tszVHjvGBTvCPN1iauAicwNMNdxTXEZmOg9MQHRgt1jRAqVDM8OZqRyqvbcRzlz18zo1wf4F7bsDUCrQ9icOZ7qxXVkpfuw/640?wx_fmt=png "")  
 解压之后挂载即可分析  
#### 17. [填空题]该涉案APP调用的im厂商的调证值是什么？（答案格式,区分大小写,符号为英文半角：123.abc!） (13分)  
  
**a80c76c9228aa4ad8c0bb2e73ccb2cca**  
   
  
思路：先确认 IM 厂商 → 网易云信（Netease NIM）→ 反编译壳内真实 dex，找到 appKey 来源。  
- 【事实】壳 libccbjqshq.so  
（lib/arm64-v8a/libccbjqshq.so  
）AES-128-ECB 解密三个 .ccb  
 得真 dex，其中 ujzyqnth.ccb  
 含业务包 com/ddtx/dingdatacontact  
（641 个类），另两个分别是 NIM SDK（com/netease/nimlib  
、com/netease/nrtc  
）与 apache lucene 库。  
  
- 【事实】app_jadx/sources/d/f/a/j.java:215  
：return new LoginInfo(k, g2, d.f.a.q.i.a.j("appkey", k.c().getS1_key()));  
，:265  
 同源；com/ddtx/dingdatacontact/Entity/AppData.java:15  
 字段 s1_key  
。  
  
- 【事实】assets/data.txt  
 base64→DES 解密后 JSON："s1_key":"a80c76c9228aa4ad8c0bb2e73ccb2cca"  
（32 hex，符合云信 appKey 形态），即 NIM LoginInfo  
 的 appKey，也就是调证值。  
  
#### 18. [填空题]加密该涉案APP调用的im厂商的调证值的加密算法的密钥是什么？（答案格式,区分大小写,符号为英文半角：123.abc!） (12分)  
  
**9ae05717**  
  
【事实】d/f/a/j.java:257-259  
：GetChildAc.setDesKey("9ae05717")  
 + DESUtil.decryptDES(d.f.a.t.b.e(getAssets().open("data.txt")), "9ae05717")  
；main_jadx/sources/com/netease/nim/uikit/utils/DESUtil.java  
 中 decryptDES  
 = Base64.decode(str)  
 → Cipher.getInstance("DES")  
 → new DESKeySpec(str2.getBytes())  
。用 9ae05717  
 实测解出上述 JSON，自校验通过（合法 JSON + 包名吻合）。  
#### 19. [填空题]加密该涉案APP调用的im厂商的调证值的加密算法是什么？(如有字母，使用大写字母) (13分)  
  
**DES**  
 同上：TRANSFORMATION = "DES"  
、algorithm = "DES"  
，未指定模式/填充，取 JCE 默认 DES/ECB/PKCS5Padding  
；pycryptodome DES.MODE_ECB  
 复现成功。  
#### 20. [填空题]该涉案APP的真实核心代码文件解密密钥是什么？（答案格式,区分大小写,符号为英文半角：123.abc!） (12分)  
  
**)#j#3n&q(cn:z$xh**  
  
【事实】System.loadLibrary("ccbjqshq")  
 → libccbjqshq.so  
 导入 EVP_aes_128_ecb  
；decrypt  
 处 EVP_DecryptInit_ex(ctx, EVP_aes_128_ecb(), 0, key@0x10ead0, 0)  
。密钥在 libccbjqshq.so  
 文件偏移 **0x10ead0**  
，16 字节 ASCII = 29 23 6a 23 33 6e 26 71 28 63 6e 3a 7a 24 78 68  
。用它 AES-128-ECB 解三个 .ccb  
 全部得到 64 65 78 0a 30 33 35 00  
（dex 035）魔数，PKCS5 去除后结构完好。  
#### 21. [填空题]该涉案APP的真实核心代码文件解密前的md5值是多少？（小写字母与数字组合，格式如：e10adc3949ba59abbe56e057f20f883e） (11分)  
  
**a64f6c24c82f169732f00464baf3803c**  
   
  
【事实】承载业务代码（com/ddtx/dingdatacontact  
、WelcomeActivity  
、AppData  
）的是 ujzyqnth.ccb  
（APK 根目录，6998352 字节），md5 = a64f6c24c82f169732f00464baf3803c  
。另两个：snzktgvbpjnsk.ccb  
 = 9e6f5c16dde417ad20d1157ae03a78f0  
（NIM SDK）、qeaabfupyd.ccb  
 = a8da4409707964c49ef4d5603d710066  
（apache 库）。  
#### 22. [填空题]涉案APP“极客传媒”的安装包的md5值是多少？（小写字母与数字组合，格式如：e10adc3949ba59abbe56e057f20f883e） (6分)  
  
**3d167332f566586c7914333d486837f5**  
   
  
【事实】/data/app/com.aepzvpgobsa.joksthtkug.eglyjdlct-1/base.apk  
（52799382 字节）与工作区只读副本 md5 一致；resources.arsc  
 内含 UTF-8 “极客传媒”，确认为该应用。  
#### 23. [填空题]该模拟器检材设置的手机号是多少？（答案格式：13300001111） (8分)  
  
18012345678   
  
配置文件里有写 ![](https://mmbiz.qpic.cn/mmbiz_png/3Bkxl9ud7XPARALD86Nk7o5VBBfNsYDM5jgjeuXabKUZN3tDg8t75hBvYicVQ8PgdqPnC70CUw3wHiaiaWvR5A2GibJlagVAb0dEbseictXXR34U/640?wx_fmt=png "")  
  
#### 24. [填空题]该模拟器检材的AndroidID是多少？（答案格式，区分大小写：123abc） (7分)  
  
**cf32cf64f8802bcd**  
   
  
【事实】/system/users/0/settings_secure.xml  
：<setting id="33" name="android_id" value="cf32cf64f8802bcd" package="android" />  
，文件 mtime 2026-06-02 00:09:12（末次启动，晚于 local/cfg-ngg/kcfg_bin_231127  
 的 00:08:38 配置下发），说明该值即末次开机由模拟器写入的当前 AndroidID。另一值 154c78c17b7f9c18  
 仅存在于 com.android.flysilkworm  
 的 ldsdk_device_pref_file.xml  
（模拟器旧值缓存，mtime 00:08:57，早于系统写入），非当前生效值。  
#### 网络流量 (10个小题, 共109分)  
#### 25. [填空题]分析流量包检材，该流量包文件的MD5哈希值前8位为？【答题格式：小写字母数字】 (4分)  
  
**71851516**  
   
  
也是直接拿软件算就行  
#### 26. [填空题]端口扫描过程中发现被攻击的机器开放了几个tcp端口？【答题格式：按实际值填写】 (10分)  
  
2   
  
用lovelyspark绘图分析可以看到被攻击方是192.168.116.130 ![](https://mmbiz.qpic.cn/mmbiz_png/3Bkxl9ud7XPafmA3W5f2NVc50lssmdBUWCwfvFg4kzDia8UpsAY3FzOCwBrPVeelLugMVhAMk3kD1YEVn1VtqY2mU6q1Iric375BibyOick9LzI/640?wx_fmt=png "")  
 端口扫描检测可以看出来就开了两个 ![](https://mmbiz.qpic.cn/sz_mmbiz_png/3Bkxl9ud7XORa7D735Fp0iaXgVN7FHnGKQ2Y3qicicJM2AaFSwpiaicEuKeUd9bP7bgHmI0TIWR8LYiaeBj0gibVntZMO0Y30y2CFJwF6JHsn1Chicw/640?wx_fmt=png "")  
  
#### 27. [填空题]网站管理员的密码被修改为？【答题格式：按实际值填写】 (10分)  
  
anyone123   
  
一般的登录api请求成功后都会跳回页面，所以应该是302返回码，同时post账号密码 ![](https://mmbiz.qpic.cn/mmbiz_png/3Bkxl9ud7XOiaG9hvVmcyuRxIFLv1qBp8WhNmlgYkkMNaQDgAicKUMbrrkic9hxGpzKZnwmkXMUsjJJFA6wI5Bp4CvS3IkoIVKpXBTtONhomts/640?wx_fmt=png "")  
 所以密码被修改为了anyone123  
  
web爆破识别也能看到 ![](https://mmbiz.qpic.cn/sz_mmbiz_png/3Bkxl9ud7XNcKBnVduVbFZN20fH8iaqTziaOSbh6VuCE5iaXnujL2BT92YRia8K3SFicRUFMJSupibmbQPCR8D6WJe4mMYiaZNCDw5GplwaL4CoB7o/640?wx_fmt=png "")  
  
#### 28. [填空题]攻击者获取到的反弹shell的端口为？【答题格式：1024】 (11分)  
  
110 ![](https://mmbiz.qpic.cn/sz_mmbiz_png/3Bkxl9ud7XNA2eTJuyEzDNnAloHRwO3OYD3Z3lrA2iaAtypdGa3iaWmOlCVOBcBKkCGzTEIAtOoHbmH7qP6ibH9oyqKbibojMljA2SeSlLNoQIE/640?wx_fmt=png "")  
  
#### 29. [填空题]被攻击网站使用的数据库连接密码为？【答题格式：按实际值填写】 (12分)  
  
john   
  
这里显示的应该有点问题，实际上是指向下面的，故密码为john（数据库名字叫darkhole） ![](https://mmbiz.qpic.cn/mmbiz_png/3Bkxl9ud7XMPgKDtEibCsmN9pfs9yqnowmGYSyTibD8yvlaw8tMFqZUc47l8xyrhc7ttsCorymfX13PKytKD7Yp7dSVdU1xiaAHrl408oy9Hcw/640?wx_fmt=png "")  
  
#### 30. [填空题]网站管理员原始的登录密码可能为？【答题格式：按实际值填写】 (12分)  
  
MZuFf^xdh1)12_i   
  
搜sql发现有一次请求成功，下载了backup.sql ![](https://mmbiz.qpic.cn/mmbiz_png/3Bkxl9ud7XNIkibOKjlSum2DOBiaicbG67QFYXXqgfjicJuobnuJUsp60oeTog6TibWGwtpsO1B6rGY9kjGYsrYnqJcVdyVLrfibUaeyQbkKor9xY/640?wx_fmt=png "")  
 查看文件就可以发现密码 ![](https://mmbiz.qpic.cn/mmbiz_png/3Bkxl9ud7XNr1tegJGv5tdacjq4l4puRcAyqYfefRxvA4tQsAVPjvNCNrDfxiafvC4nQicOqXsjlZLrliar9icFyBFSGHYia9ZUf2ibw5oAib8h69o/640?wx_fmt=png "")  
  
#### 31. [填空题]攻击者通过执行某个文件完成了从网站用户www-data到普通用户的提权，该文件的sha256值前10位为？【答题格式：哈希值中字母采用小写】 (12分)  
  
a59945b56d   
  
反弹shell的命令执行部分，通过使用cms成功成为john用户 ![](https://mmbiz.qpic.cn/sz_mmbiz_png/3Bkxl9ud7XPdVV3rNiajdnX1Lj2uNiaTcCxfagrWHlIw1wdAheO4BDTorh5VrkibvU4H1gNfd5rnsI6ZictDSeUWJ5WkRpJCszogo8ydARW2qf8/640?wx_fmt=png "")  
 看webshell，发现返回的是elf ![](https://mmbiz.qpic.cn/mmbiz_png/3Bkxl9ud7XN4aUjbmcZESFJhhIHvJZockjIQDJ8Shia79icPJviaYhRbM1sg6socmNqKtJn5C7YftPrUia7nVEia9yrOic7e0N0Jajm35V6iajW4uo/640?wx_fmt=png "")  
 所以问一下Qiko这是执行了什么操作 ![](https://mmbiz.qpic.cn/mmbiz_png/3Bkxl9ud7XPz2ea8MOLLzDSulBBUx8vsnCgkmD6FccdlCEdD7R0sOQPh9fy73jKJ4RyeF06VHydGiaSvY0Bia3zrgqPzgzzx5ckWiaMMunSpIk/640?wx_fmt=png "")  
 此处下载了cms，所以导出返回的数据计算sha256即可 a59945b56d89ac5e87bdcd355bd82b48573d6c1d53dca2dc258ca4bc1baa4ccc  
#### 32. [填空题]网站用户id为3的用户使用的邮箱为？【答题格式：按实际值填写】 (13分)  
  
webshell分析得到 ![](https://mmbiz.qpic.cn/sz_mmbiz_png/3Bkxl9ud7XMxWHj6dxiaOOBMghBwGjIQmIPed98pyHUzlW3orLKB0k3QorNS1jGMv17iaKvgvlZK2tl3KX9EFCsIg9FMPuFSapA6aWicT0yricc/640?wx_fmt=png "")  
  
#### 33. [填空题]攻击者通过什么文件拿到了服务器的最高权限？【答题格式：按实际值填写】 (13分)  
  
web.py 执行了之后就变成root了 ![](https://mmbiz.qpic.cn/sz_mmbiz_png/3Bkxl9ud7XMuYiacWLhE0tPRSVpTZ9X5fygNznupd2OmkzbIzzDVuj7dT7aib5ibxmAxwjLia6g66IbYKd7Eaw5HOqkFsqOX0Cmp5DNRvW2Fia5k/640?wx_fmt=png "")  
  
#### 34. [填空题]攻击者通过某个文件完成了持久化，该文件中记录的密钥算法为？【答题格式：如遇字母小写，例：md5】 (12分)   
  
持久化都和authorized_keys有关 ![](https://mmbiz.qpic.cn/mmbiz_png/3Bkxl9ud7XMr710telLON5GxY2iaksGftHrJ637W8kH23KF281rmSeicnTRUXBgDOwPN010BuYudTxuShsEgHQuPdVVsG5hTKc1jiaHbsNYeXM/640?wx_fmt=png "")  
 这个是通过webshell上传的，所以就是ssh-ed25519 ![](https://mmbiz.qpic.cn/sz_mmbiz_png/3Bkxl9ud7XNMc0ApNCicpTC5LQ850lXFmPW21880R1mand4ibhn18legiceZoicbObej7JQdJqODibCQTA7JIqQBmqwLhek4rc6MG6aYlFPGbnxI/640?wx_fmt=png "")  
  
#### 手机 (14个小题, 共146分)  
#### 35. [填空题]通过对“检材-手机.rar”进行分析，分析检材中微信ID:wxid_f4s0jmpvrc522对应的手机号。【参考格式：13800000000】】 (8分)  
  
17859628390 ![](https://mmbiz.qpic.cn/sz_mmbiz_png/3Bkxl9ud7XPON5ciaOayGOxqwTmmiaF8HC25iaxu0ypg6pCuicWibII19Lb3he7xy5VVn0jRC2uwPLhL0TJTFjK2O7MzXFK04fxAd5DuN75uncTU/640?wx_fmt=png "")  
  
#### 36. [填空题]对手机检材进行取证分析，分析检材中“华为应用市场”第一次安装日期。【参考格式：2025-08-24】 (9分)  
  
2025-09-24 19:37:42 ![](https://mmbiz.qpic.cn/mmbiz_png/3Bkxl9ud7XNVS0zNzM4tibd7Bz8IJfianw01IHOZEkhQDAKRB4acdOlZuxOz05ZsSFspicOy1BOPdJuD7dwJ2Ftia2ChN8nYyib5hPxVHlFSG9ibo/640?wx_fmt=png "")  
  
#### 37. [填空题]对手机检材进行取证分析，找出检材中钱包APP，请列出该APP中ETH地址后八位。【字母全大写，参考格式：A12B34D5】 (10分)  
  
503FE61F ![](https://mmbiz.qpic.cn/sz_mmbiz_png/3Bkxl9ud7XPKhWrwbrQnp8JsB1Xvw174P1gG53y6LtzmpoE5Xddac3ztEFleIaoxDuNT2ulDD3SuHQAEtFHaO0ktXicoG6DRFAemNuZW84WM/640?wx_fmt=png "")  
  
#### 38. [填空题]对手机检材进行取证分析，手机相册中有张“imtoken助记词1.PNG”图片被破坏，请修复该图片，列出该图片中第三个单词。【参考格式：abcd】 (10分)  
  
boost 搜索得到照片位置 ![](https://mmbiz.qpic.cn/sz_mmbiz_png/3Bkxl9ud7XOuYHL9icKNQdbVFnyf3ggxk2dhic0G49MaKT1l5wwkJpfzLj6ZbibCYiaUyFbu5EckwFowjhxkOTcMjCqnYZUFbLOHolLL7Jia4GxU/640?wx_fmt=png "")  
 加载图片发现crc校验报错 ![](https://mmbiz.qpic.cn/mmbiz_png/3Bkxl9ud7XMAE9kZmAots7bgzVicFsphZF1pcuFzkCDiaicGzItMXdQgneiaotV9hkRnncHgOOD9eOnobcmHE1637BCr9RHfDAgSVzWpBvO3Mjo/640?wx_fmt=png "")  
 使用爆破脚本获得实际宽高 ![](https://mmbiz.qpic.cn/sz_mmbiz_png/3Bkxl9ud7XOl2SYib6n2ooq3GewNTcvhLZ49R4lel8KdcD6UCQQ6MGRz1TRd8qRP85Y4UiahPXH6p7vbTxnzeNhqOzdiceQFMr1rl4xAQpQyTU/640?wx_fmt=png "")  
  
```
import zlibimport structimport argparseimport itertoolsparser = argparse.ArgumentParser()parser.add_argument("-f", type=str, default=None, required=True,                    help="输入同级目录下图片的名称")args  = parser.parse_args()bin_data = open(args.f, 'rb').read()crc32key = zlib.crc32(bin_data[12:29]) # 计算crcoriginal_crc32 = int(bin_data[29:33].hex(), 16) # 原始crcif crc32key == original_crc32: # 计算crc对比原始crc    print('宽高没有问题!')else:    input_ = input("宽高被改了, 是否CRC爆破宽高? (Y/n):")    if input_ not in ["Y", "y", ""]:        exit()    else:        for i, j in itertools.product(range(4095), range(4095)): # 理论上0x FF FF FF FF，但考虑到屏幕实际/cpu，0x 0F FF就差不多了，也就是4095宽度和高度            data = bin_data[12:16] + struct.pack('>i', i) + struct.pack('>i', j) + bin_data[24:29]            crc32 = zlib.crc32(data)            if(crc32 == original_crc32): # 计算当图片大小为i:j时的CRC校验值，与图片中的CRC比较，当相同，则图片大小已经确定                print(f"\nCRC32: {hex(original_crc32)}")                print(f"宽度: {i}, hex: {hex(i)}")                print(f"高度: {j}, hex: {hex(j)}")                exit(0)
```  
  
改动一下ihdr宽度 ![](https://mmbiz.qpic.cn/sz_mmbiz_png/3Bkxl9ud7XMWZbNXa74BQjGln2MN1COKRgK2kgficGYGRI2OtVEG1ibBhnBFLo49lnoMXmBRI1UJa13f0GyrB5VH74WY6p3IETZ2zZNEXU0rE/640?wx_fmt=png "")  
 再打开即可看到助记词 ![](https://mmbiz.qpic.cn/mmbiz_png/3Bkxl9ud7XOgrEJY0rcA4gjzzPQG8kj1cowZ0s1YJHaqRe8AqMjJW1TsLrnURTX7gSUWVUtG8p641QwAAAYP4ckVGG2oSARcODY1iabdAiaek/640?wx_fmt=png "")  
  
#### 39. [填空题]对手机检材进行取证分析，找出一张PNG图片，该图片上显示“助记词2”，请列出该图片上显示的第二个单词。【参考格式：abcd】 (10分)  
  
delay   
  
猜测名字差不多 ![](https://mmbiz.qpic.cn/mmbiz_png/3Bkxl9ud7XOgt55ic0gs3n6Ww2ZYz9z4177H8w2Va5Ww1X8wiboIbwleJKicL3WR8iaEfroEA5LJXaic1Oh8eKgcrFrofAbTMReTNuQF0L7VFp64/640?wx_fmt=png "")  
 果然搜到了 ![](https://mmbiz.qpic.cn/sz_mmbiz_png/3Bkxl9ud7XPmGAicaZtyMqfJQPN4Ky3F27LytJwjfzNNTkl8PMlgCXnPVz2kszEKRZxha2NKId8YgcS8ibLCuAxh50RlnWicMSFsO3ZLCY5cvs/640?wx_fmt=png "")  
  
#### 40. [填空题]对手机检材进行取证分析，找出检材中显示“助记词3”的文档，列出该文档中记录的第三个助记词单词。【参考格式：abcd】 (10分)  
  
quarter   
  
该文件夹下有两个文件，一个xlsx，一个没后缀名的 ![](https://mmbiz.qpic.cn/mmbiz_png/3Bkxl9ud7XOsBJsBwIp8fA4322KPceAxvvicBIQxvo0ryjic78WXunloPXNPpy8Fibria9Ik42O2nGxPVrWZufbGic1GrvvAdWxHv3jZG6njY1Wo/640?wx_fmt=png "")  
 其中备忘录的文件末尾有助记词 ![](https://mmbiz.qpic.cn/mmbiz_png/3Bkxl9ud7XOlicaxvCMyIFMg6akZaDibMo7RpLtWhmEmurYaDTQPu7WLT55gMgEgqrU1utfyUc8x8eaAdicvZsSrytlHnwjqlprbjic8jo7KTQk/640?wx_fmt=png "")  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/3Bkxl9ud7XMCywyoKyvQLuurN50DkI0UicjeW7icpYULB1jgLZ8ibqwLPd7KiarhN83U9ceS5ReEEYvMiaXJwEOicAWtibVnPX8Yqsgd5kjEvY30rY/640?wx_fmt=png "")  
  
#### 41. [填空题]对手机检材进行取证分析，分析出该组助记词正常顺序中最后一个单词（已知助记词1、助记词2、助记词3中的单词顺序有被调整）。【参考格式：abcd】 (10分)  
  
这个题不会做，ai也跑不出来，看看各位师傅有什么好的想法可以一起交流，我尝试了爆破但是没爆破出来  
#### 42. [填空题]对手机检材进行取证分析，分析出邮箱中收件人QQ号为“850563586”的姓名。【参考格式：张三】 (10分)  
  
刘佳雨 ![](https://mmbiz.qpic.cn/sz_mmbiz_png/3Bkxl9ud7XOXBMmAGVW4upHva5uiasC39DUicGicyE3bSDUfiaTibO8BvTxnvKeQJVIx1hZydPeicFC6kYXB6fpiaolHb9LSModg7EEAza0vN4pbwU/640?wx_fmt=png "")  
  
#### 43. [填空题]对手机检材进行取证分析，得知机主通过某个应用给HHshAL发送了一个文档，该应用的数据包名是什么？【参考格式：com.test.abc】 (10分)  
  
dingtong.saichuang   
  
从44题往前看，那肯定就是小众通讯软件了 在检材里的小众取证软件也只有叮通比较可疑 ![](https://mmbiz.qpic.cn/mmbiz_png/3Bkxl9ud7XP2JyNmictvThEPdWrAhLdP6MIMCZds2nAYgMoaAO8Xj2uIBJMk3sB5vjycWG605dlEeA9SV2iab1jntzoO7mfOu07aYxQ3EgUb0/640?wx_fmt=png "")  
  
#### 44. [填空题]接上题，对手机检材进行取证分析，该应用聊天记录数据库的打开密码是什么？【参考格式：ABCabc123】 (10分)  
  
@1@#!aajsk1*JKJ  
   
  
接下来就到逆向部分了，这部分交给Qiko解决了 使用专家去解题，效果很好 ![](https://mmbiz.qpic.cn/mmbiz_png/3Bkxl9ud7XPJGwrKW7gRpRrSU7HnU5Ofl7KE2s25Q3bCzEcGzRgLZqYyYwslBjhicbicicibvvCnMzZbmFeFLUcur9Letu43dd0POEIiakFLbHxg/640?wx_fmt=png "")  
 反编译 base.apk  
（jadx）：i2/a.java  
 → b = "@1@#!aajsk1*JKJ"  
，Myapplication.java:2312 getWritableDatabase(i2.a.b)  
；用该口令 + page=1024/reserve=48/PBKDF2-SHA1/iter=64000 实际解密成功（sdec2.py  
 → dt_dec2.db  
，20 张表全部可读），属**实证可解密**  
而非仅字符串引用  
#### 45. [填空题]接上题，机主发送的这个加密文档，打开密码是什么？【参考格式：ABCabc123】 (12分)  
  
QWERT666 同上库 sys_message  
 第二条（1758713421620）：密码：--.-/.--/./.-./-/-..../-..../-..../  
 → 摩斯 Q W E R T 6 6 6 = QWERT666；用 msoffcrypto  
 对该文件（derived/sent.xlsx  
，md5 b2be3432…）解密成功得 dec_QWERT666.xlsx  
（6 人清单）→ 口令正确  
#### 46. [填空题]对手机检材进行取证分析，厉明的身份证地址登记的门牌号是多少？？【参考格式：123】 (12分)  
  
搜到一个sfz ![](https://mmbiz.qpic.cn/sz_mmbiz_png/3Bkxl9ud7XNibZDQCz63WiaOOeIQNk8m5SCibZemOV8MP3Bt9qsQSW0ePtCrhLCaSO6ZgcqrjqWbyXwu7HpB7JtDaKfgHoyrm78SiciaSCZjAz0c/640?wx_fmt=png "")  
 发现其实是压缩包 ![](https://mmbiz.qpic.cn/mmbiz_png/3Bkxl9ud7XN4R4DLrMtd1p7DIZKBULS74UcPPiaial1hwgic4wCRicVaibibibKQzUgrJSGwINypOibAicnVAZ5PNR2Qc5EPhbO7kbwqmeIibIdD5jHOE/640?wx_fmt=png "")  
  
  
反向去找com.android.mxt的数据库，得到密码 ![](https://mmbiz.qpic.cn/mmbiz_png/3Bkxl9ud7XOhCib2Okk8R8Dy41ibTqic51G0HFCsJQecia9ChtUw6yg2JImR8gMkaAxXFuadgzZBwMs8PJd4oqUmRib0o5VZhWGEpe4mfbic0BW5Q/640?wx_fmt=png "")  
 打开压缩包即可获得sfz  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3Bkxl9ud7XNGja1Cicl9VDiaFjQCVdtHKsBQW8nUjyE7IwRE7avoJ0pbU9j2238XO0AXy2vW9uHvibIozFegelhKxBmibrPbeANR7AibxcYMEFzY/640?wx_fmt=png&from=appmsg "")  
#### 47. [填空题]对手机检材进行取证分析，分析出“important1.xlsx”文件中体现的“金达欣”银行卡后六位？【参考格式：123456】 (12分)  
  
935629 还是刚刚的数据库里有密码 ![](https://mmbiz.qpic.cn/sz_mmbiz_png/3Bkxl9ud7XOfYiaMvSTniaA2k97JTVI2bkn33MoujYKGiaWkubibDjnDXXCNSStF90klcJ8B0eNiaBPNMdL84Vh6XkG3IuyOQKicOib4NjGb3Iiaq3Y/640?wx_fmt=png "")  
 打开excel就可以获得信息   
#### 48. [填空题]接上题，“important1.xlsx”打开密码数据的应用，该应用的启动密码是什么？【参考格式：1234】 (13分)  
  
1596 这道题同样需要逆向，交给Qiko jadx 反编译 base.apk  
（$SEEU_WORKSPACE/derived/mxtapk/jout/sources  
）：  
- com/android/mxt/constant/HawkEnum.java:7  
 → LOCK(101,"LOCK")  
，LOCK_CHECK(102,…)  
 是配套的提示字段；  
  
- com/android/mxt/act/SplashActivity.java:100  
 new Handler().postDelayed(new b(),2000)  
 → :94 SplashActivity.this.a(LockActivity.class)  
：**一进应用就跳 LockActivity**  
；  
  
- com/android/mxt/act/LockActivity.java:215  
 savePsw = h0.a(HawkEnum.LOCK...)  
，:218  
 空则把输入当新密码保存，非空则 psw.equals(savePsw)  
 才进 MainActivity  
；  
  
- TransitionActivity.java:26-30  
 onResume 时再加一次锁（OPEN_TYPE=1  
）。 所以：LOCK 是“启动/锁屏 4 位密码”，解锁逻辑就是拿它做等值比较 → 答案 **1596**  
。 E:/data/data/com.android.mxt/shared_prefs/Hawk2.xml  
 只有 8 个键，值全是这种形态（Hawk 2.0 库）：  
  
```
LOCK        = java.lang.String##0V@AQLk2HE2VYmQb05p1d37u0byCdx+FeyJr8/DuZhlrnd4+P7wLOCK_CHECK  = java.lang.String##0V@AQKMffGLJm4NxL0zbADQkETXNDJJYU1JzZNIfln/8jqx9s+TMlgt2xw=MXT_PWD / R_PWD / LOCK_STATE / LOCK_ACTIVITY / LOCK_CHECK_POSITION / POLICY / RISK_CONTENT
```  
  
类型名##  
 是 Java 类型提示，0V@  
 之后才是密文载荷。  
- 密钥：crypto.KEY_256.xml  
 → cipher_key = VFgb2FX+xdFQBTxFBFBfdGFwTDmfOVLLQa+7OFx62+4=  
（base64 后正好 32 字节，即 CryptoConfig.KEY_256  
）。  
  
- 载荷布局（去掉 类型名##  
 和 3 字符前缀 0V@  
 后做 base64 解码）： [1B version=0x01][1B cipherId=0x02 → AES/GCM][12B iv][ciphertext][16B tag]  
  
- **AAD = bytes([version, cipherId]) + 键名**  
（如 b"\x01\x02LOCK"  
）——这是关键。AAD 一旦不对，GCM tag 就校验不过；这也是我一开始直接按“iv+ct”解不出来的原因。  
  
```
raw = base64.b64decode(val.split("##",1)[1][3:])      # 去 0V@ver, cid, iv, tag = raw[0], raw[1], raw[2:14], raw[-16:]c = AES.new(KEY, AES.MODE_GCM, nonce=iv)c.update(bytes([ver,cid]) + name.encode())            # AAD=版本‖算法‖键名pt = c.decrypt_and_verify(raw[14:-16], tag)
```  
  
结果为1596  
  
