#  微软亲手挖的坑？CLFS认证机制曝出提权漏洞，可轻松绕过任意杀软防御  
原创 3had0w
                    3had0w  潇湘信安   2026-11-17 03:30  
  
<table><thead><tr style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><th data-colwidth="390" style="padding: 6px 13px;-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;word-break: break-all;hyphens: auto;"><section style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;"><span leaf=""><span textstyle="" style="font-size: 14px;">Ima知识库名称</span></span></section></th><th data-colwidth="171" style="padding: 6px 13px;-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;word-break: break-all;hyphens: auto;"><section style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;text-align: center;"><span leaf=""><span textstyle="" style="font-size: 14px;">加入条件</span></span></section></th></tr></thead><tbody><tr><td data-colwidth="390"><p><span leaf=""><a class="weapp_text_link js_weapp_entry" style="" data-unique-id="mmgq2ycy-fenn11" data-miniprogram-type="text" data-miniprogram-appid="wx4c6401744b734596" data-miniprogram-path="pages/index/index?shareId=5a450acc0ba7011d52e5087819ece0eaaf0706e9dee5bfce575c9c243bd86cc5" data-miniprogram-nickname="ima知识库" data-miniprogram-servicetype="0" data-miniprogram-applink=""><span textstyle="" style="font-size: 14px;">潇湘信安协同知识库（更新~ing!）</span></a></span></p></td><td data-colwidth="171"><p style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 14px;">限时免费</span></span></p></td></tr><tr><td data-colwidth="390"><p><span leaf=""><a class="weapp_text_link js_weapp_entry" style="" data-unique-id="mmgq3uc4-4eb0f6" data-miniprogram-type="text" data-miniprogram-appid="wx4c6401744b734596" data-miniprogram-path="pages/index/index?shareId=e3cb38cefa77595a6e512eae049953037e275bbab66071ed2f33d98fdf64e6f3" data-miniprogram-nickname="ima知识库" data-miniprogram-servicetype="0" data-miniprogram-applink=""><span textstyle="" style="font-size: 14px;">潇湘信安学习资料库（更新~ing!）</span></a></span></p></td><td data-colwidth="171"><p style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 14px;">≥3年粉丝</span></span></p></td></tr><tr><td data-colwidth="390"><p><span leaf=""><a class="weapp_text_link js_weapp_entry" style="" data-unique-id="mmgq4fe2-yfukhq" data-miniprogram-type="text" data-miniprogram-appid="wx4c6401744b734596" data-miniprogram-path="pages/index/index?shareId=da55c9143f5ee86a843ae20db3b689d73aa80ae9334b701add96a22ec001ac1c" data-miniprogram-nickname="ima知识库" data-miniprogram-servicetype="0" data-miniprogram-applink=""><span textstyle="" style="font-size: 14px;">潇湘信安内部知识库（更新~ing!）</span></a></span></p></td><td data-colwidth="171"><p style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 14px;">星球成员</span></span></p></td></tr></tbody></table>  
现在只对常读和星标公众号才展示大图推送，建议大家把“  
潇湘信安  
”  
设为星标  
，  
否则可能看不到了  
！  
  
## 一、漏洞概述：什么是SignDrop？  
  
今天给大家分享一个近期曝光的重量级Windows本地提权0day漏洞——**SignDrop**  
。这个漏洞的特殊之处在于：它并非源于某个被忽视的代码缺陷，而是**藏在微软为了修复漏洞而推出的安全缓解措施之中**  
。换句话说，微软为了让系统更安全而加的"补丁"，反而创造了一个新的提权漏洞。  
  
SignDrop漏洞存在于Windows通用日志文件系统（CLFS）的认证机制中，攻击者可以利用普通用户权限，通过精巧的构造和对象管理器符号链接操纵，诱导高权限进程（如Windows Defender）为其创建系统级认证文件，最终实现从普通用户到**SYSTEM权限**  
的本地提权。  
  
更令人担忧的是，据漏洞作者估计，全球约**80%的Windows设备**  
都可能受到此漏洞影响。如果你安装系统已经超过90天，那么几乎可以肯定你的系统正处于风险之中。  
## 二、背景知识：CLFS认证缓解机制的来龙去脉  
  
要理解SignDrop漏洞，我们得先从CLFS说起。  
  
**CLFS（Common Log File System，通用日志文件系统）**  
 是Windows内核提供的一套高性能日志服务，被广泛用于事务处理、数据库操作等场景。由于CLFS在内核模式下解析复杂的日志文件结构，长期以来都是漏洞挖掘的重灾区——一旦解析出错，就可能导致内核内存损坏，进而被利用实现权限提升。  
  
为了遏制CLFS漏洞的泛滥，微软在**2025年10月28日**  
正式发布了CLFS认证缓解机制。该机制的核心思路是：为每个CLFS日志文件添加数字签名认证码，确保日志文件的完整性和合法性，防止恶意构造的日志文件被内核加载。  
  
根据微软官方文档的说明：  
> 目前，采用期为90天，从系统首次启动并加载此版本CLFS时开始计算。90天采用期结束后，驱动程序将在下次启动时自动切换到强制模式，此后CLFS将要求所有日志文件都包含有效的认证码。  
  
  
这一措施推出后，公开的CLFS漏洞利用数量确实急剧下降，攻击面大幅缩小。然而，谁也没有想到，这个旨在提升安全性的缓解措施，竟然引入了一个全新的、更巧妙的漏洞——SignDrop。  
  
**参考链接：**  
- CLFS Authentication Mitigation in Windows 90-day Learn Mode  
  
- Microsoft官方说明 - Common Log File System (CLFS) Authentication Mitigation  
  
## 三、漏洞原理：安全机制如何反被利用？  
### 3.1 CLFS日志文件的创建与认证  
  
SignDrop漏洞的根源在于 clfs.sys  
 驱动处理认证文件（.cnpf  
 后缀）创建时的逻辑缺陷。  
  
当用户通过 \\CLFS  
 设备创建新的日志文件时，CLFS驱动会生成一个 .blf  
 后缀的基础日志文件。你还可以通过调用 AddLogContainer  
 API 关联额外的日志容器，每个关联的日志文件都会有一个对应的认证文件（以 .cnpf  
 结尾）。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/tNyBeBKReNL6u82nGzC5UEgNcDcAc17F3Hpa4Vp1MoiaIRuOZHYtLZibBpA92FI3l7XicRwX2TkktgWU9k5bnwAuy4pE57ntrxnurR4KAYpmys/640?wx_fmt=jpeg&from=appmsg "")  
  
clfs_device  
在正常情况下，CLFS会通过指定 OBJ_FORCE_ACCESS_CHECK  
 标志来强制执行访问检查，确保新创建的对象都在调用进程的安全上下文中完成。到这一步为止，CLFS的实现都是正确的。  
### 3.2 关联容器的签名行为  
  
在 CclfsContainer::Create  
 例程中，可以看到容器文件和认证文件的创建/打开逻辑。这里有一个有趣的行为：当你将一个新的日志容器关联到原始容器后，只要打开原始的 .blf  
 容器，关联的容器也会被自动打开。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/tNyBeBKReNJpmjfFp0CemK83iaCI3pPWA4giag1GjJk4kHNrjWtH9puQmInd1tyLTO1sRicwjCQRfeEqMuf0ib5aTI7fwJVCiahG1QIyan9kAmZs/640?wx_fmt=jpeg&from=appmsg "")  
  
clfs_device2  
更关键的发现是：**如果新关联的容器文件是空的，CLFS会尝试为其签名，无论签名文件是否存在**  
。虽然此时仍然是在调用进程的上下文中执行，看似没有问题，但隐患已经埋下。  
### 3.3 调用进程切换的陷阱  
  
问题的关键在于：**打开日志文件不需要在 NtCreateFile 中指定任何特殊标志**  
，只需要提供正确的路径即可。例如，路径 \\CLFS\\??\\C:\\test\\t  
 就会打开 t.blf  
 日志文件。如果该文件指定了关联日志文件，关联文件也会被一并打开。  
  
这意味着什么？意味着攻击者可以：  
1. 以普通用户身份创建并设置好CLFS日志容器（包含一个指向受控路径的关联容器）  
  
1. 诱导高权限进程打开这个日志文件  
  
1. 当高权限进程打开时，如果关联容器的签名文件不存在，CLFS会以高权限进程的身份创建认证文件  
  
在漏洞的PoC中，攻击者通过操纵**对象管理器符号链接**  
，诱导Windows Defender的反恶意软件引擎（调用 ServerMpUpdateEngineSignature  
）打开CLFS容器。由于调用进程是特权进程，认证文件将以 **SYSTEM权限**  
 创建。  
  
更雪上加霜的是，CLFS在创建新的认证文件时，会使用原始文件的DACL（自由访问控制列表），这意味着普通用户可能有权限访问这个以SYSTEM身份创建的认证文件。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/tNyBeBKReNJFIMF28bdWvmR8M2mMPxXaArQ5ts5DJgkD6S3PkdeN6KwKgtrTgzdy75ZicQkKYde3uayWLXUPjLNibv9q6Bgic4uWXI2VJicvC4E/640?wx_fmt=jpeg&from=appmsg "")  
  
clfs_device3## 四、利用过程详解：从普通用户到SYSTEM权限  
  
整个漏洞的利用链条可以说是相当精妙——微软的每一步实现都"正确无误"，但组合在一起就形成了一个完美的权限提升路径。更有趣的是，这个合法的安全缓解机制竟然可以被用来对付它自己。  
### 4.1 基本利用流程  
1. **构造CLFS日志容器**  
：以普通用户身份创建一个基础日志文件，并关联一个指向目标路径的空容器文件  
  
1. **设置符号链接**  
：通过对象管理器符号链接，将打开操作重定向到精心构造的路径  
  
1. **诱导高权限进程**  
：利用Windows Defender等特权进程的文件访问行为，使其打开我们构造的CLFS日志文件  
  
1. **触发认证文件创建**  
：当高权限进程打开关联容器时，发现认证文件不存在，于是以SYSTEM权限创建 .cnpf  
 认证文件  
  
1. **利用认证文件**  
：由于认证文件继承了原始文件的DACL，普通用户可以访问并利用这个SYSTEM级别的文件实现进一步提权  
  
### 4.2 更强大的用途：绕过杀软自防护  
  
SignDrop漏洞的独特之处不仅在于本地提权，更在于它可以被用来**绕过任意杀毒软件或EDR的自防护（篡改防护）机制**  
。  
  
原理很简单：你可以触发任意文件在**反恶意软件进程自身的上下文**  
中创建。从杀毒软件内核驱动的角度来看，这个文件创建请求来自于它们自己的合法进程，因此会顺利通过检查。这意味着攻击者可以利用这个漏洞，在杀软的"眼皮底下"创建恶意文件，甚至直接关闭杀毒软件的防御功能。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/tNyBeBKReNJxkblShCJB8GNcmZibUiauJKdr1vm8uReYboKiajueuKAkEzoa9qhjnu4gxWPsCWicic1D2W4icNYEVqSdgOaxuCQ9LrCSJXXcEOqII/640?wx_fmt=jpeg&from=appmsg "")  
  
Untitled## 五、影响范围：哪些Windows系统中招了？  
### 5.1 已验证的受影响系统  
  
漏洞作者已在以下系统中验证了漏洞的存在：  
- **Windows 11 26H2**  
  
- **Windows 11 26H2 实验版**  
  
- **Windows Server 2025 实验版**  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/tNyBeBKReNKB462YZqtntk26xEalFnFc5V6bicpjYicicgKHLV1DR32PJBfOHC2FBSAC6DnibA5OBxQbqJ01WuXWTeWrSibocTVYVtgc80IU60VY/640?wx_fmt=jpeg&from=appmsg "")  
  
poc### 5.2 潜在影响范围  
  
根据漏洞作者的估计，全球约 **80%的Windows安装**  
 可能受到此漏洞影响。判断依据很简单：  
- 如果你的Windows系统安装时间超过90天，且CLFS认证缓解机制已经启用（处于学习模式或强制模式），那么你很可能受到影响  
  
- 换句话说，只要你的系统不是全新安装（90天以内），大概率都是脆弱的  
  
这个估计可能听起来有些夸张，但考虑到CLFS是Windows核心组件，且认证缓解机制已经推送了一段时间，受影响范围确实可能非常广泛。  
## 六、检测方法：一键判断你的电脑是否脆弱  
  
想要知道自己的系统是否受到SignDrop漏洞影响？方法很简单，检查一下注册表中CLFS认证模式的配置即可。  
### 6.1 通过注册表检查  
  
打开注册表编辑器，定位到以下路径：  
```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\CLFS\Authentication\Mode
```  
  
根据 Mode  
 的值来判断：  
  
<table><thead><tr style="margin-right: 0px;margin-bottom: 0px;margin-left: 0px;padding: 0px;box-sizing: border-box;border-width: medium;border-style: none;border-color: currentcolor;border-image: none;background-color: transparent;"><th style="margin-right: 0px;margin-bottom: 0px;margin-left: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 2px;border-style: none none solid;border-color: currentcolor currentcolor rgb(4, 120, 87);border-image: none;font-size: 15px;line-height: 21px;text-align: center;background: transparent;color: rgb(19, 64, 52);font-weight: 700;letter-spacing: 0.3px;min-width: 100px;white-space: nowrap;"><section><span leaf="">Mode值</span></section></th><th style="margin-right: 0px;margin-bottom: 0px;margin-left: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 2px;border-style: none none solid;border-color: currentcolor currentcolor rgb(4, 120, 87);border-image: none;font-size: 15px;line-height: 21px;text-align: center;background: transparent;color: rgb(19, 64, 52);font-weight: 700;letter-spacing: 0.3px;min-width: 100px;white-space: nowrap;"><section><span leaf="">状态</span></section></th><th style="margin-right: 0px;margin-bottom: 0px;margin-left: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 2px;border-style: none none solid;border-color: currentcolor currentcolor rgb(4, 120, 87);border-image: none;font-size: 15px;line-height: 21px;text-align: center;background: transparent;color: rgb(19, 64, 52);font-weight: 700;letter-spacing: 0.3px;min-width: 100px;white-space: nowrap;"><section><span leaf="">是否受影响</span></section></th></tr></thead><tbody><tr style="margin-right: 0px;margin-bottom: 0px;margin-left: 0px;padding: 0px;box-sizing: border-box;border-width: medium;border-style: none;border-color: currentcolor;border-image: none;background-color: transparent;"><td style="margin-right: 0px;margin-bottom: 0px;margin-left: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(228, 233, 230);border-image: none;font-size: 15px;color: rgb(36, 42, 38);line-height: 21px;text-align: center;min-width: 100px;white-space: nowrap;"><section><span leaf="">0</span></section></td><td style="margin-right: 0px;margin-bottom: 0px;margin-left: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(228, 233, 230);border-image: none;font-size: 15px;color: rgb(36, 42, 38);line-height: 21px;text-align: center;min-width: 100px;white-space: nowrap;"><section><span leaf="">未启用 / 关闭</span></section></td><td style="margin-right: 0px;margin-bottom: 0px;margin-left: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(228, 233, 230);border-image: none;font-size: 15px;color: rgb(36, 42, 38);line-height: 21px;text-align: center;min-width: 100px;white-space: nowrap;"><section><span leaf="">⚠️ 受影响</span></section></td></tr><tr style="margin-right: 0px;margin-bottom: 0px;margin-left: 0px;padding: 0px;box-sizing: border-box;border-width: medium;border-style: none;border-color: currentcolor;border-image: none;background-color: transparent;"><td style="margin-right: 0px;margin-bottom: 0px;margin-left: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(228, 233, 230);border-image: none;font-size: 15px;color: rgb(36, 42, 38);line-height: 21px;text-align: center;min-width: 100px;white-space: nowrap;"><section><span leaf="">1</span></section></td><td style="margin-right: 0px;margin-bottom: 0px;margin-left: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(228, 233, 230);border-image: none;font-size: 15px;color: rgb(36, 42, 38);line-height: 21px;text-align: center;min-width: 100px;white-space: nowrap;"><section><span leaf="">学习模式（Learn Mode）</span></section></td><td style="margin-right: 0px;margin-bottom: 0px;margin-left: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(228, 233, 230);border-image: none;font-size: 15px;color: rgb(36, 42, 38);line-height: 21px;text-align: center;min-width: 100px;white-space: nowrap;"><section><span leaf="">⚠️ 受影响</span></section></td></tr><tr style="margin-right: 0px;margin-bottom: 0px;margin-left: 0px;padding: 0px;box-sizing: border-box;border-width: medium;border-style: none;border-color: currentcolor;border-image: none;background-color: transparent;"><td style="margin-right: 0px;margin-bottom: 0px;margin-left: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(228, 233, 230);border-image: none;font-size: 15px;color: rgb(36, 42, 38);line-height: 21px;text-align: center;min-width: 100px;white-space: nowrap;"><section><span leaf="">2</span></section></td><td style="margin-right: 0px;margin-bottom: 0px;margin-left: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(228, 233, 230);border-image: none;font-size: 15px;color: rgb(36, 42, 38);line-height: 21px;text-align: center;min-width: 100px;white-space: nowrap;"><section><span leaf="">强制模式（Enforcement Mode）</span></section></td><td style="margin-right: 0px;margin-bottom: 0px;margin-left: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(228, 233, 230);border-image: none;font-size: 15px;color: rgb(36, 42, 38);line-height: 21px;text-align: center;min-width: 100px;white-space: nowrap;"><section><span leaf="">✅ 不受影响</span></section></td></tr><tr style="margin-right: 0px;margin-bottom: 0px;margin-left: 0px;padding: 0px;box-sizing: border-box;border-width: medium;border-style: none;border-color: currentcolor;border-image: none;background-color: transparent;"><td style="margin-right: 0px;margin-bottom: 0px;margin-left: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(228, 233, 230);border-image: none;font-size: 15px;color: rgb(36, 42, 38);line-height: 21px;text-align: center;min-width: 100px;white-space: nowrap;"><section><span leaf="">3</span></section></td><td style="margin-right: 0px;margin-bottom: 0px;margin-left: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(228, 233, 230);border-image: none;font-size: 15px;color: rgb(36, 42, 38);line-height: 21px;text-align: center;min-width: 100px;white-space: nowrap;"><section><span leaf="">审计模式（Audit Mode）</span></section></td><td style="margin-right: 0px;margin-bottom: 0px;margin-left: 0px;padding: 6px 8px;box-sizing: border-box;border-width: medium medium 1px;border-style: none none solid;border-color: currentcolor currentcolor rgb(228, 233, 230);border-image: none;font-size: 15px;color: rgb(36, 42, 38);line-height: 21px;text-align: center;min-width: 100px;white-space: nowrap;"><section><span leaf="">✅ 不受影响</span></section></td></tr></tbody></table>  
  
简单来说：**如果Mode值为0或1，你的系统就存在被SignDrop漏洞利用的风险**  
；如果为2或3，则相对安全。  
### 6.2 使用PowerShell快速检查  
  
你也可以以管理员身份运行PowerShell，执行以下命令快速查看：  
```
Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\CLFS\Authentication" -Name "Mode"
```  
## 七、修复与防护建议  
### 7.1 目前的困境  
  
遗憾的是，截至漏洞公开时，**微软尚未发布针对SignDrop漏洞的安全补丁**  
。  
  
更棘手的是，简单地禁用CLFS认证缓解机制并不可取——因为这样会让系统重新暴露在CLFS日志文件解析导致的内存损坏漏洞面前。毕竟，认证缓解机制之所以存在，正是为了应对这些更严重的内存破坏问题。  
  
这就形成了一个两难的局面：  
- **启用认证缓解**  
 → 面临SignDrop提权漏洞  
  
- **禁用认证缓解**  
 → 面临CLFS内存损坏漏洞  
  
### 7.2 临时防护建议  
  
在微软发布正式补丁之前，可以考虑以下防护措施：  
1. **关注微软安全更新**  
：密切关注微软每月的补丁星期二（Patch Tuesday）更新，及时安装相关安全补丁  
  
1. **限制非信任用户访问**  
：在企业环境中，严格限制本地普通用户的权限，减少攻击面  
  
1. **监控异常行为**  
：监控CLFS相关的异常文件创建和符号链接操作  
  
1. **联系微软安全团队**  
：如有紧急问题，可发送邮件至 secure@microsoft.com 联系微软安全响应中心  
  
### 7.3 长期建议  
  
对于企业和安全研究人员来说：  
- 及时升级到最新版本的Windows系统  
  
- 关注CLFS认证模式的变更，了解90天学习期结束后系统状态的变化  
  
- 建立完善的终端检测与响应（EDR）机制，即使面对提权攻击也能及时发现和响应  
  
## 八、总结与思考  
  
SignDrop漏洞给我们带来了很多值得深思的启示：  
  
**第一，安全防御本身也可能成为攻击面。**  
 微软为了修复CLFS漏洞而推出的认证机制，反而创造了一个新的提权路径。这提醒我们，每添加一层安全防护，都需要仔细审视其本身是否会引入新的安全问题。  
  
**第二，逻辑漏洞往往比内存漏洞更难防范。**  
 SignDrop不是一个传统的内存破坏漏洞，而是一个逻辑设计缺陷。微软的每一步实现都"符合规范"，但组合在一起就形成了漏洞。这类逻辑漏洞的发现和修复都更具挑战性。  
  
**第三，安全软件自身也可能被利用。**  
 漏洞利用Windows Defender等安全软件的高权限特性来实现提权，甚至可以绕过杀软的自防护机制。这再次证明，没有绝对安全的系统。  
  
**第四，0day漏洞的威胁始终存在。**  
 SignDrop是一个0day漏洞，在公开时微软尚未修复。对于普通用户和企业来说，保持系统更新、建立多层次的安全防护体系仍然是最有效的应对策略。  
  
SignDrop漏洞的发现者用一个巧妙的命名向我们展示了安全世界的复杂性——有时候，最坚固的堡垒，恰恰是从内部被攻破的。希望微软能够尽快推出修复补丁，也希望每一位读者都能重视这个漏洞，及时检查自己的系统安全状态。  
> 漏洞POC项目：https://github.com/MSNightmare/SignDrop  
> 漏洞影响评估：建议所有Windows用户尽快检查系统状态，等待微软官方补丁发布  
  
  
  
**关注我们**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOeSsicAgIUNHtMib9a69NOWXw1A7mgRqqiat1SycQ0b6e5mBqC0pVJ3oicrQnCTh4gqMGiaKUPicTsUc4Tw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
 还在等什么？赶紧点击下方名片开始学习吧   
![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOeSsicAgIUNHtMib9a69NOWXw1A7mgRqqiat1SycQ0b6e5mBqC0pVJ3oicrQnCTh4gqMGiaKUPicTsUc4Tw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
  
  
  
**知 识 星 球**  
  
  
  
星球已过800人，暂不再发放优惠券，如还有需要的师傅可加我  
VX  
：**S_3had0w，等你一起来学习...！**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/XOPdGZ2MYOe8h8vDq3IN1GhNuNnkbWYgcibYRicgo0SRy1febylE4kBDdibgIJV1ia1QNSBYiaZAZWCzAjESoYfxNjw/640?wx_fmt=jpeg&from=appmsg "")  
<table><thead><tr style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;background-color: rgb(255, 255, 255);border-top: 1px solid rgba(209, 217, 224, 0.7);"><th data-colwidth="390" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);padding: 6px 13px;outline: 0px;word-break: break-all;hyphens: auto;"><section style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;"><span leaf=""><span textstyle="" style="font-size: 14px;">Ima知识库名称</span></span></section></th><th data-colwidth="171" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);padding: 6px 13px;outline: 0px;word-break: break-all;hyphens: auto;"><section style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);outline: 0px;text-align: center;"><span leaf=""><span textstyle="" style="font-size: 14px;">加入条件</span></span></section></th></tr></thead><tbody><tr><td data-colwidth="390"><p><span leaf=""><a class="weapp_text_link js_weapp_entry" style="" data-unique-id="mmgq2ycy-fenn11" data-miniprogram-type="text" data-miniprogram-appid="wx4c6401744b734596" data-miniprogram-path="pages/index/index?shareId=5a450acc0ba7011d52e5087819ece0eaaf0706e9dee5bfce575c9c243bd86cc5" data-miniprogram-nickname="ima知识库" data-miniprogram-servicetype="0" data-miniprogram-applink=""><span textstyle="" style="font-size: 14px;">潇湘信安协同知识库（更新~ing!）</span></a></span></p></td><td data-colwidth="171"><p style="text-align: center;" data-pm-slice="2 2 [&#34;table&#34;,{&#34;interlaced&#34;:&#34;disabled&#34;,&#34;align&#34;:null,&#34;class&#34;:null,&#34;style&#34;:&#34;letter-spacing: 0.034em;&#34;},&#34;table_body&#34;,null,&#34;table_row&#34;,{&#34;class&#34;:null,&#34;style&#34;:null},&#34;table_cell&#34;,{&#34;colspan&#34;:1,&#34;rowspan&#34;:1,&#34;colwidth&#34;:[171],&#34;width&#34;:null,&#34;valign&#34;:null,&#34;align&#34;:null,&#34;style&#34;:null}]"><span leaf=""><span textstyle="" style="font-size: 14px;">限时</span></span><span leaf=""><span textstyle="" style="font-size: 14px;">免费</span></span></p></td></tr><tr><td data-colwidth="390"><p><span leaf=""><a class="weapp_text_link js_weapp_entry" style="" data-unique-id="mmgq3uc4-4eb0f6" data-miniprogram-type="text" data-miniprogram-appid="wx4c6401744b734596" data-miniprogram-path="pages/index/index?shareId=e3cb38cefa77595a6e512eae049953037e275bbab66071ed2f33d98fdf64e6f3" data-miniprogram-nickname="ima知识库" data-miniprogram-servicetype="0" data-miniprogram-applink=""><span textstyle="" style="font-size: 14px;">潇湘信安学习资料库（更新~ing!）</span></a></span></p></td><td data-colwidth="171"><p style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 14px;">≥3年粉丝</span></span></p></td></tr><tr><td data-colwidth="390"><p><span leaf=""><a class="weapp_text_link js_weapp_entry" style="" data-unique-id="mmgq4fe2-yfukhq" data-miniprogram-type="text" data-miniprogram-appid="wx4c6401744b734596" data-miniprogram-path="pages/index/index?shareId=da55c9143f5ee86a843ae20db3b689d73aa80ae9334b701add96a22ec001ac1c" data-miniprogram-nickname="ima知识库" data-miniprogram-servicetype="0" data-miniprogram-applink=""><span textstyle="" style="font-size: 14px;">潇湘信安内部知识库（更新~ing!）</span></a></span></p></td><td data-colwidth="171"><p style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 14px;">星球成员</span></span></p></td></tr></tbody></table>  
<table><tbody><tr><td data-colwidth="287"><section nodeleaf=""><mp-common-miniprogram class="js_uneditable custom_select_card mp_miniprogram_iframe" data-pluginname="insertminiprogram" data-miniprogram-path="pages/index/index?shareId=5a450acc0ba7011d52e5087819ece0eaaf0706e9dee5bfce575c9c243bd86cc5" data-miniprogram-nickname="ima知识库" data-miniprogram-avatar="http://mmbiz.qpic.cn/sz_mmbiz_png/JkibkGcLiaJbVFXj8nYrmL5iab8KYwalibCtCjnicUJ6KFK8G4uC0VgesgxIL1iaTckOZYlKXNEtaBNeazcD8PLcic0zA/640?wx_fmt=png&amp;wxfrom=200" data-miniprogram-title="潇湘信安协同知识库（更新~ing!）" data-miniprogram-imageurl="http://mmbiz.qpic.cn/mmbiz_jpg/zqnzfCSS1kB81aIibHjJDKmPS8BOiacfOmDUsm8ibwibG3bvo3EsfbkqO70BJNIo7rE4LqOia3xKRITWTZSOHKLwOO2ribkueicxLrLeEsV1FfDicus/0?wx_fmt=jpeg" data-miniprogram-type="card" data-miniprogram-servicetype="0" data-miniprogram-appid="wx4c6401744b734596" data-miniprogram-imageurlback="https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_jpg%2FzqnzfCSS1kDw29J3ia1mGnlrZzHxa4HsTezib66icQ99G6vicWPvurHOicRL3wMv4PUyOibicDeN00EnibyP8feJsmyKUYnRHNRP8CA8Ja2HnPB6Ed8%2F0%3Fwx_fmt%3Djpeg%26from%3Dappmsg" data-miniprogram-cropperinfo="%7B%22c%22%3A%7B%22x%22%3A0%2C%22y%22%3A59%2C%22x2%22%3A182%2C%22y2%22%3A204.6%2C%22w%22%3A182%2C%22h%22%3A145.6%7D%7D"></mp-common-miniprogram></section></td><td data-colwidth="287"><section nodeleaf=""><mp-common-miniprogram class="js_uneditable custom_select_card mp_miniprogram_iframe" data-pluginname="insertminiprogram" data-miniprogram-path="pages/index/index?shareId=e3cb38cefa77595a6e512eae049953037e275bbab66071ed2f33d98fdf64e6f3" data-miniprogram-nickname="ima知识库" data-miniprogram-avatar="http://mmbiz.qpic.cn/sz_mmbiz_png/JkibkGcLiaJbVFXj8nYrmL5iab8KYwalibCtCjnicUJ6KFK8G4uC0VgesgxIL1iaTckOZYlKXNEtaBNeazcD8PLcic0zA/640?wx_fmt=png&amp;wxfrom=200" data-miniprogram-title="潇湘信安学习资料库（更新~ing!）" data-miniprogram-imageurl="http://mmbiz.qpic.cn/sz_mmbiz_jpg/zqnzfCSS1kC6ohbbn1EEAjmjkMbLqKC0TevicIR9DOX2IiafOBL5J7ygLr5khHY1XvTibQjqsA412O6HicaL1eyHvANjXdfI5DkCJWficVQibIzia0/0?wx_fmt=jpeg" data-miniprogram-type="card" data-miniprogram-servicetype="0" data-miniprogram-appid="wx4c6401744b734596" data-miniprogram-imageurlback="https%3A%2F%2Fmmbiz.qpic.cn%2Fsz_mmbiz_jpg%2FzqnzfCSS1kABnOfCKmD6QCfNgmCk1eSkyoGEqZHw5Z54qG2khyAQ32PcGWx9Fz4ESFE2wjVOyRuNLtBhnl3jRbnY7clNeC2uppGr6jXArJ0%2F0%3Fwx_fmt%3Djpeg%26from%3Dappmsg" data-miniprogram-cropperinfo="%7B%22c%22%3A%7B%22x%22%3A0%2C%22y%22%3A59%2C%22x2%22%3A182%2C%22y2%22%3A204.6%2C%22w%22%3A182%2C%22h%22%3A145.6%7D%7D"></mp-common-miniprogram></section></td></tr></tbody></table>  
  
**推 荐 阅 读**  
  
  
  
  
[](https://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247499188&idx=1&sn=9ce15a0e66b2595285e544aaa0c49c24&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247496043&idx=1&sn=4daa27ade9915de6021fea1c2a21d7bc&scene=21#wechat_redirect)  
  
[](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247486327&idx=1&sn=71fc57dc96c7e3b1806993ad0a12794a&chksm=cfa6af64f8d1267259efd56edab4ad3cd43331ec53d3e029311bae1da987b2319a3cb9c0970e&scene=21#wechat_redirect)  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdAPjIVeN2ZahG9ibP0Y3wlfg6BO1WO7MZfo1JeW7zDWcLSTQ5Ek8zXAia5w1nMnogpbpXP6OxXXOicA/640?wx_fmt=png "")  
  
