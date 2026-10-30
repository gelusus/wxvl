#  突发！1day漏洞致使大量NAS用户文件被加密  
欢迎关注→
                    欢迎关注→  安全客   2026-10-30 02:24  
  
今年10月以来，360反勒索服务连续接到多起针对群晖NAS设备的勒索攻击求助。反勒索团队利用360AI安全专家，对被攻击设备进行了采样分析，确认本轮勒索攻击的主要源头是老牌勒索家族Cerber的一个变种“  
C3RB3R  
”，该家族长期针对NAS设备实施加密勒索，此次攻击使用了两个今年9月披露的漏洞。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/g5KiabmYVDH1xUYSfytwGHb8TiaBSBmcT3jOqFDSBHtzfianCnViavauK9M07NSFUWEBZvOTqMCdicApEInSnpbMB2yeSYI0SavWsuBE9GRgbBkE/640?wx_fmt=jpeg "")  
![]( "")  
![]( "")  
![]( "")  
  
▲ 图1：Cerber（C3RB3R）攻击群晖NAS流程示意  
  
经详细排查发现，本轮攻击的切入点是两个高危远程漏洞，CVE编号分别为  
CVE-2026-13639  
和  
CVE-2026-13684  
。这两个漏洞于今年9月披露，距今仅一个月左右，这意味着网络中仍有大量设备尚未更新系统修补漏洞，很多用户也根本不清楚这两个漏洞的存在和危害性，本次攻击是一个典型的  
1day漏洞攻击场景  
。  
  
一、拆解分析：Cerber勒索"走新路，穿旧鞋"  
  
Cerber虽然在攻击入口阶段采用了这两个1day高危漏洞，但其在入侵成功进入受害设备后，执行的操作依然是  
加密、留信、等赎金  
的"老三样"。  
  
1️⃣ 扫描系统  
  
Cerber进入受害设备后，会扫描当前系统中的文件，跳过特定系统目录以避免加密文件导致系统崩溃，进而影响自身的持续加密。跳过的目录共  
27个  
，具体列表如下：  
  
/boot /run /proc /sys /dev  
  
/etc /config /sbin /bin /lib  
  
/lib32 /lib64 /usr/sbin /usr/bin /usr/lib  
  
/usr/lib32 /usr/lib64 /usr/include /usr/local /usr/share  
  
/usr/src /usr/syno /var/packages /var/lock /var/run  
  
/var/tmp /tmp  
  
而对于其他所有目录，Cerber均会进入目录进行递归扫描并加密其中的文件。对于文件名及扩展名，Cerber会跳过常见的可执行文件或脚本，以及被加密后的文件。被跳过的文件名及扩展名如下：  
  
.exe .dll .lnk .sys .msi .bat .mod  
  
read-me3.txt C3RB3R_LOG.txt init-rsysd.lo init-ld.lo .L0CK3D  
  
2️⃣ 加密文件  
  
进入核心的文件加密流程，Cerber勒索软件依然采用当前较为主流的  
ChaCha8（对称加密算法）+ RSA2048（非对称加密算法）  
的加密算法策略。实施加密时，Cerber会对每个待加密文件生成一份单独的  
256位密钥文件及64位nonce  
，再使用ChaCha8算法对文件内容实施加密。加密完成后，利用样本内置的RSA公钥，对加密密钥及加密模式、加密位置、文件原始信息等数据进行加密，并将全部密文附在被加密文件的数据尾部。最终，将被加密文件扩展名修改为"  
.L0CK3D  
"，以标记该文件已加密完成。  
  
此外，勒索样本中还附带了一份文件扩展名列表。扩展名在该列表中的文件，Cerber采用了完整加密的策略，而其他文件则采用区分大小的选择性加密策略。其加密策略如下：  
  
<table><tbody><tr><td data-colwidth="243" width="341" valign="top" style="background: rgb(242, 242, 242);border-width: 1pt;border-style: solid;border-color: windowtext;padding: 0pt 5.4pt;"><p style="margin-bottom: 0px;line-height: 1.75em;text-align: center;margin-left: 8px;margin-right: 8px;"><span style="font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;font-variant: normal;text-transform: none;color: rgb(51, 51, 51);font-weight: bold;" mpa-font-style="mvt6hx7kxcd"><span leaf=""><span textstyle="" style="letter-spacing: 1px;">文件类型</span></span></span></p></td><td data-colwidth="316" width="341" valign="top" style="background: rgb(242, 242, 242);border-width: 1pt;border-style: solid;border-color: windowtext;padding: 0pt 5.4pt;"><p style="margin-bottom: 0px;line-height: 1.75em;text-align: center;margin-left: 8px;margin-right: 8px;"><span style="font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;font-variant: normal;text-transform: none;color: rgb(51, 51, 51);font-weight: bold;" mpa-font-style="mvt6hx7k1nue"><span leaf=""><span textstyle="" style="letter-spacing: 1px;">加密策略</span></span></span></p></td></tr><tr><td data-colwidth="243" width="341" valign="top" style="border-width: 1pt;border-style: solid;border-color: windowtext;padding: 0pt 5.4pt;"><p style="margin-bottom: 0px;line-height: 1.75em;text-align: justify;margin-left: 8px;margin-right: 8px;"><span style="font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;font-variant: normal;text-transform: none;color: rgb(70, 70, 70);" mpa-font-style="mvt6hx7kck8"><span leaf=""><span textstyle="" style="letter-spacing: 1px;">文件后缀在优先列表中</span></span></span></p></td><td data-colwidth="316" width="341" valign="top" style="border-width: 1pt;border-style: solid;border-color: windowtext;padding: 0pt 5.4pt;"><p style="margin-bottom: 0px;line-height: 1.75em;text-align: justify;margin-left: 8px;margin-right: 8px;"><span style="font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;font-variant: normal;text-transform: none;color: rgb(70, 70, 70);" mpa-font-style="mvt6hx7kct"><span leaf=""><span textstyle="" style="letter-spacing: 1px;">不区分文件大小，全文件加密</span></span></span></p></td></tr><tr><td data-colwidth="243" width="341" valign="top" style="border-width: 1pt;border-style: solid;border-color: windowtext;padding: 0pt 5.4pt;"><p style="margin-bottom: 0px;line-height: 1.75em;text-align: justify;margin-left: 8px;margin-right: 8px;"><span style="font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;font-variant: normal;text-transform: none;color: rgb(70, 70, 70);" mpa-font-style="mvt6hx7k1q39"><span leaf=""><span textstyle="" style="letter-spacing: 1px;">小于约1MB的普通文件</span></span></span></p></td><td data-colwidth="316" width="341" valign="top" style="border-width: 1pt;border-style: solid;border-color: windowtext;padding: 0pt 5.4pt;"><p style="margin-bottom: 0px;line-height: 1.75em;text-align: justify;margin-left: 8px;margin-right: 8px;"><span style="font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;font-variant: normal;text-transform: none;color: rgb(70, 70, 70);" mpa-font-style="mvt6hx7k1z9g"><span leaf=""><span textstyle="" style="letter-spacing: 1px;">全文件加密</span></span></span></p></td></tr><tr><td data-colwidth="243" width="341" valign="top" style="border-width: 1pt;border-style: solid;border-color: windowtext;padding: 0pt 5.4pt;"><p style="margin-bottom: 0px;line-height: 1.75em;text-align: justify;margin-left: 8px;margin-right: 8px;"><span style="font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;font-variant: normal;text-transform: none;color: rgb(70, 70, 70);" mpa-font-style="mvt6hx7k1sm3"><span leaf=""><span textstyle="" style="letter-spacing: 1px;">约1MB到约5MB的普通文件</span></span></span></p></td><td data-colwidth="316" width="341" valign="top" style="border-width: 1pt;border-style: solid;border-color: windowtext;padding: 0pt 5.4pt;"><p style="margin-bottom: 0px;line-height: 1.75em;text-align: justify;margin-left: 8px;margin-right: 8px;"><span style="font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;font-variant: normal;text-transform: none;color: rgb(70, 70, 70);" mpa-font-style="mvt6hx7k1why"><span leaf=""><span textstyle="" style="letter-spacing: 1px;">只加密头部约1MB的数据</span></span></span></p></td></tr><tr><td data-colwidth="243" width="341" valign="top" style="border-width: 1pt;border-style: solid;border-color: windowtext;padding: 0pt 5.4pt;"><p style="margin-bottom: 0px;line-height: 1.75em;text-align: justify;margin-left: 8px;margin-right: 8px;"><span style="font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;font-variant: normal;text-transform: none;color: rgb(70, 70, 70);" mpa-font-style="mvt6hx7k1f05"><span leaf=""><span textstyle="" style="letter-spacing: 1px;">大于约5MB的普通文件</span></span></span></p></td><td data-colwidth="316" width="341" valign="top" style="border-width: 1pt;border-style: solid;border-color: windowtext;padding: 0pt 5.4pt;"><p style="margin-bottom: 0px;line-height: 1.75em;text-align: justify;margin-left: 8px;margin-right: 8px;"><span style="font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;font-variant: normal;text-transform: none;color: rgb(70, 70, 70);" mpa-font-style="mvt6hx7kouw"><span leaf=""><span textstyle="" style="letter-spacing: 1px;">以约5MB大小分段加密，仅加密每段头部约1MB数据</span></span></span></p></td></tr></tbody></table>  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/g5KiabmYVDH36ibp86yxtfo8GmnGSy87Ex1XvUcddyK0Ba71Z3FjMARM9IPsUkYGxF1d76rWda5EScmwaVKFCyYob1yibuKR8m00t8SQk52kjY/640?wx_fmt=jpeg "")  
![]( "")  
![]( "")  
![]( "")  
  
▲ 图2：受害群晖NAS中被加密的文件（扩展名被改为.L0CK3D）  
  
而不区分大小全文件加密的优先扩展名共  
187个  
，全部列举如下：  
  
.4dd .4dl .accdb .accdc .accde .accdr .accdt  
  
.accft .adb .ade .adf .adp .arc .ora  
  
.alf .ask .btr .bdf .cat .cdb .ckp  
  
.cma .cpd .dacpac .dad .dadiagrams .daschema .db-shm  
  
.db-wal .db3 .dbc .dbf .dbs .dbt .dbv  
  
.dbx .dcb .dct .dcx .ddl .dlis .dp1  
  
.dqy .dsk .dsn .dtsx .dxl .eco .ecx  
  
.edb .epim .exb .fcd .fdb .fic .fmp  
  
.fmp12 .fmpsl .fol .fp3 .fp4 .fp5 .fp7  
  
.fpt .frm .gdb .grdb .gwi .hdb .his  
  
.ibd .idb .ihx .itdb .itw .jet .jtx  
  
.kdb .kexi .kexic .kexis .lgc .lwx .maf  
  
.maq .mar .mas .mav .mdb .mdf .mpd  
  
.mrg .mud .mwb .myd .ndf .nnt .nrmlib  
  
.ns2 .ns3 .ns4 .nsf .nv2 .nwdb .nyf  
  
.odb .oqy .orx .owc .p96 .p97 .pan  
  
.pdb .pdm .pnz .qry .qvd .rbf .rctd  
  
.rod .rodx .rpd .rsd .sas7bdat .sbf .scx  
  
.sdb .sdc .sdf .sis .spq .sql .sqlite  
  
.sqlite3 .sqlitedb .temx .tmd .tps .trc .trm  
  
.udb .udl .usr .v12 .vis .vpd .vvv  
  
.wdb .wmdb .wrk .xdb .xld .xmlff .abcddb  
  
.abs .abx .accdw .adn .db2 .fm5 .hjt  
  
.icg .icr .lut .maw .mdn .mdt .vdi  
  
.vhd .vmdk .pvm .vmem .vmsn .vmsd .nvram  
  
.vmx .raw .qcow2 .subvol .bin .vsv .avhd  
  
.vmrs .vhdx .avdx .vmcx .iso  
  
3️⃣ 勒索信息  
  
最终，Cerber会在每个被加密的目录中留下一份名为"  
read-me3.txt  
"的勒索信文件，提醒用户文件已被加密，要求尽快交付赎金。这份勒索信息的文件内容如下（文中问号部分，在实际执行时会替换为本地生成的6位设备识别ID）：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/g5KiabmYVDH1T7pSpE0fFS1ksIqW85dHYUgNpljwNDnY7Xbh3g2ialNcMdwdcLW4Mbu1r5ENR6fu2O8hic8kDVUZPL6BZlyRUJSshgaoGOCJEc/640?wx_fmt=png "")  
![]( "")  
![]( "")  
![]( "")  
  
▲ 图3：Cerber（C3RB3R）勒索信 read-me3.txt  
  
二、修漏补缺，及时更新NAS系统  
  
由于本轮攻击的主要目标是群晖的NAS设备，其运行的是硬件设备上高度定制化的私有系统。对于这种环境中的安全防护，往往非常依赖供应商提供的系统更新或漏洞补丁。  
  
所幸的是，群晖在漏洞被披露的第一时间（  
2026年9月18日  
），就发布了针对这批漏洞补丁的系统更新。具体的系统更新版本号如下：  
  
<table><tbody><tr><td data-colwidth="131" width="341" valign="top" style="background: rgb(242, 242, 242);border-width: 1pt;border-style: solid;border-color: windowtext;padding: 0pt 5.4pt;"><p style="margin-bottom: 0px;line-height: 1.75em;text-align: justify;margin-left: 8px;margin-right: 8px;"><span style="font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;font-variant: normal;text-transform: none;color: rgb(51, 51, 51);font-weight: bold;" mpa-font-style="mvt6hx7k241q"><span leaf=""><span textstyle="" style="letter-spacing: 1px;">设备系统版本</span></span></span></p></td><td data-colwidth="304" width="341" valign="top" style="background: rgb(242, 242, 242);border-width: 1pt;border-style: solid;border-color: windowtext;padding: 0pt 5.4pt;"><p style="margin-bottom: 0px;line-height: 1.75em;text-align: justify;margin-left: 8px;margin-right: 8px;"><span style="font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;font-variant: normal;text-transform: none;color: rgb(51, 51, 51);font-weight: bold;" mpa-font-style="mvt6hx7kjmp"><span leaf=""><span textstyle="" style="letter-spacing: 1px;">漏洞已被修补的更新版本号</span></span></span></p></td></tr><tr><td data-colwidth="131" width="341" valign="top" style="border-width: 1pt;border-style: solid;border-color: windowtext;padding: 0pt 5.4pt;"><p style="margin-bottom: 0px;line-height: 1.75em;text-align: justify;margin-left: 8px;margin-right: 8px;"><span style="font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;font-variant: normal;text-transform: none;color: rgb(70, 70, 70);" mpa-font-style="mvt6hx7k1p5k"><span leaf=""><span textstyle="" style="letter-spacing: 1px;">DSM 7.4</span></span></span></p></td><td data-colwidth="304" width="341" valign="top" style="border-width: 1pt;border-style: solid;border-color: windowtext;padding: 0pt 5.4pt;"><p style="margin-bottom: 0px;line-height: 1.75em;text-align: justify;margin-left: 8px;margin-right: 8px;"><span style="font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;font-variant: normal;text-transform: none;color: rgb(70, 70, 70);" mpa-font-style="mvt6hx7klhb"><span leaf=""><span textstyle="" style="letter-spacing: 1px;">更新至 7.4-90075 或更高版本</span></span></span></p></td></tr><tr><td data-colwidth="131" width="341" valign="top" style="border-width: 1pt;border-style: solid;border-color: windowtext;padding: 0pt 5.4pt;"><p style="margin-bottom: 0px;line-height: 1.75em;text-align: justify;margin-left: 8px;margin-right: 8px;"><span style="font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;font-variant: normal;text-transform: none;color: rgb(70, 70, 70);" mpa-font-style="mvt6hx7k6lx"><span leaf=""><span textstyle="" style="letter-spacing: 1px;">DSM 7.3</span></span></span></p></td><td data-colwidth="304" width="341" valign="top" style="border-width: 1pt;border-style: solid;border-color: windowtext;padding: 0pt 5.4pt;"><p style="margin-bottom: 0px;line-height: 1.75em;text-align: justify;margin-left: 8px;margin-right: 8px;"><span style="font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;font-variant: normal;text-transform: none;color: rgb(70, 70, 70);" mpa-font-style="mvt6hx7kpar"><span leaf=""><span textstyle="" style="letter-spacing: 1px;">更新至 7.3.2-86009-4 或更高版本</span></span></span></p></td></tr><tr><td data-colwidth="131" width="341" valign="top" style="border-width: 1pt;border-style: solid;border-color: windowtext;padding: 0pt 5.4pt;"><p style="margin-bottom: 0px;line-height: 1.75em;text-align: justify;margin-left: 8px;margin-right: 8px;"><span style="font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;font-variant: normal;text-transform: none;color: rgb(70, 70, 70);" mpa-font-style="mvt6hx7k12vg"><span leaf=""><span textstyle="" style="letter-spacing: 1px;">DSM 7.2.2</span></span></span></p></td><td data-colwidth="304" width="341" valign="top" style="border-width: 1pt;border-style: solid;border-color: windowtext;padding: 0pt 5.4pt;"><p style="margin-bottom: 0px;line-height: 1.75em;text-align: justify;margin-left: 8px;margin-right: 8px;"><span style="font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;font-variant: normal;text-transform: none;color: rgb(70, 70, 70);" mpa-font-style="mvt6hx7k1hi8"><span leaf=""><span textstyle="" style="letter-spacing: 1px;">更新至 7.2.2-72806-9 或更高版本</span></span></span></p></td></tr><tr><td data-colwidth="131" width="341" valign="top" style="border-width: 1pt;border-style: solid;border-color: windowtext;padding: 0pt 5.4pt;"><p style="margin-bottom: 0px;line-height: 1.75em;text-align: justify;margin-left: 8px;margin-right: 8px;"><span style="font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;font-variant: normal;text-transform: none;color: rgb(70, 70, 70);" mpa-font-style="mvt6hx7k1262"><span leaf=""><span textstyle="" style="letter-spacing: 1px;">DSM 7.2.1</span></span></span></p></td><td data-colwidth="304" width="341" valign="top" style="border-width: 1pt;border-style: solid;border-color: windowtext;padding: 0pt 5.4pt;"><p style="margin-bottom: 0px;line-height: 1.75em;text-align: justify;margin-left: 8px;margin-right: 8px;"><span style="font-size: 15px;font-family: mp-quote, -apple-system-font, BlinkMacSystemFont, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;font-variant: normal;text-transform: none;color: rgb(70, 70, 70);" mpa-font-style="mvt6hx7k11io"><span leaf=""><span textstyle="" style="letter-spacing: 1px;">更新至 7.2.1-69057-12 或更高版本</span></span></span></p></td></tr></tbody></table>  
  
群晖在这波更新中共修复了   
8个漏洞  
，这些漏洞的CVE编号及作用分别如下：  
  
CVE-2026-13684 / CVE-2026-13639 / CVE-2026-13635  
  
这一组三个漏洞允许远程攻击者读/写任意文件、进行拒绝服务（DoS）攻击或获取非敏感信息。  
  
CVE-2026-13673 / CVE-2026-6205 / CVE-2026-13666  
  
这一组三个漏洞允许远程认证用户读/写任意文件；当受害用户点击共享网址时，允许写入权限受限的文件并实施拒绝服务（DoS）攻击。  
  
CVE-2026-13623 / CVE-2026-13683  
  
这一组两个漏洞允许具有管理员权限的远程认证用户读/写权限受限的文件或获取非敏感信息。  
  
其他更多信息可以参考群晖官方发布的更新公告：  
  
https://www.synology.com/en-br/security/advisory/Synology_SA_26_13  
  
三、安全建议  
  
针对本轮Cerber对群晖NAS发起的勒索攻击，我们给出如下安全建议：  
- 关注网络安全资讯，第一时间更新  
NAS设备系统或安装安全补丁，封堵高危漏洞；  
  
- 尽量  
不要直接把  
NAS管理端口映射  
至  
公网，  
如需  
外网访问  
，优先使用  
NAS厂商提供的跳转业务  
或者  
VPN  
；  
  
- 停用  
设备初始的  
默认  
管理员  
账号，  
为所有  
账号设置强  
口令，尽可能  
开启两步验证  
及  
自动  
IP封锁  
等安全验证机制；  
  
- 开启共享文件夹快照并打开快照防篡改功能  
；  
定期检查快照是否正常生成  
；  
  
- 至少为重要数据  
准备  
一  
份离线隔离备份  
；该备份  
平时  
需断开与  
NAS  
的任何连接  
，避免被勒索  
软件  
一并加密  
；  
  
- 开启  
设备  
防火墙，  
仅  
放行必要端口  
，尽量为管理后台的访问权限  
配置可信  
IP白名单  
；  
  
- 定期查看日志中心，留意异常登录、批量文件修改，发现  
可疑  
后缀  
或疑似  
勒索文件  
等情况，应  
立刻断网  
进行安全检查；  
  
- 清理闲置共享链接、不用的套件和服务，缩小攻击面  
；  
  
- 不要支付赎金  
！  
不要支付赎金  
！  
不要支付赎金  
！  
  
🚨 请立即检查你的群晖NAS系统版本，尽快升级固件、修补漏洞，谨防数据被恶意加密造成不可挽回的损失！  
  
附录：入侵威胁指标（IOCs）  
  
MD5：  
  
a1375e0f82ae86df528876c049b5a23c  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/o0buL62hK7M8RnVz7mqRVDRkqm2sJeT2icM4WyR7kMkHpLVaicR3tJ4gr5kIb4zje9lXgd5PuOw42Z5KtathltcQ/640?from=appmsg "")  
  
**END**  
  
推荐阅读  
  
[凌晨收到GitLab安全公告，CVSS满分，我看了看自家版本号](https://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649790630&idx=1&sn=0acf40c9d3d8e770595f319ed91c4318&scene=21#wechat_redirect)  
  
  
2026-10-28  
[](https://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649790630&idx=1&sn=0acf40c9d3d8e770595f319ed91c4318&scene=21#wechat_redirect)  
  
  
[Node.js、Python、Bun全中招：一个压缩库的内存漏洞，让半个互联网"裸奔"](https://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649790621&idx=1&sn=e1c8b6b58db8ded6cf523adf6e2ea73d&scene=21#wechat_redirect)  
  
  
2026-10-27  
[](https://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649790621&idx=1&sn=e1c8b6b58db8ded6cf523adf6e2ea73d&scene=21#wechat_redirect)  
  
  
[AI自己写勒索脚本、自己挑目标、自己搞钱：俄罗斯黑客的Phantom框架，把勒索软件玩成了自动化流水线](https://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649790617&idx=1&sn=5a7f21ee8b2e0b8ad29b71179e2ff751&scene=21#wechat_redirect)  
  
  
2026-10-26  
[](https://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649790617&idx=1&sn=5a7f21ee8b2e0b8ad29b71179e2ff751&scene=21#wechat_redirect)  
  
  
[你的对手不是人了：Claude+Grok组队，400家金融机构被AI全自动攻破](https://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649790612&idx=1&sn=bd189312c5882e6f90847903ad4866d7&scene=21#wechat_redirect)  
  
  
2026-10-23  
[](https://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649790612&idx=1&sn=bd189312c5882e6f90847903ad4866d7&scene=21#wechat_redirect)  
  
  
[75%的网站都该警惕了！PHP「php2shell」高危漏洞深度解析与修复指南](https://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649790606&idx=1&sn=143aa778ebfd3d88d84926430d4a9727&scene=21#wechat_redirect)  
  
  
2026-10-22  
[](https://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649790606&idx=1&sn=143aa778ebfd3d88d84926430d4a9727&scene=21#wechat_redirect)  
  
  
**更多AI知识，请关注Aiker World社区**  
  
  
