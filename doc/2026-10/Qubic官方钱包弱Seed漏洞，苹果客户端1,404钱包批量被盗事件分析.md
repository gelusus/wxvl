#  Qubic官方钱包弱Seed漏洞，苹果客户端1,404钱包批量被盗事件分析  
tuto
                    tuto  杂杂咱谈   2026-10-21 00:28  
  
10月17日至18日，攻击者利用@Qubic官方Web钱包在Apple平台用Math.random()生成钱包Seed的随机性缺陷，离线枚举全部2^32个可能Seed，碰撞出1,404个有余额的钱包私钥，并盗走2,939,606,531,944 QU，大约$1.3M。  
  
攻击发生后，Qubic进行了保护性停机，87.1%被盗资金冻结在链上，12.9%进入MEXC并被冻结账户。官方目前已发布Web钱包v4.26.4修复(改用crypto.getRandomValues)，请相关用户及时更新。  
  
一、受害者范围  
  
1,404个钱包地址，这些地址的共同点: Seed由Qubic官方客户端在下述环境中生成，随机性不足:  
  
iPhone/iPad上任意浏览器(Safari、Chrome、Firefox等均被强制使用WebKit内核)，2026-10-18之前创建；  
  
MacBook/iMac的Safari，2026-10-18之前创建；  
  
Qubic iOS App，2025年7月之前创建；  
  
二、漏洞根因:Math.random()生成的密钥种子的随机性不足  
  
Qubic的Seed是55位小写字母串，理论空间26^55 (≈2^258)，但实际安全性取决于生成它的PRNG的内部状态熵。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/xoBWaEOhvRGRb9sAT4H2wdicLLUhk7mzR6Hwvdx1iaKSGHjEvRibbdKic095GcEXV79ojEcwlVnYF9yFZx7xFiaGdGcVqOrVrRYwkSNEKwEQR9y4/640?wx_fmt=jpeg "")  
  
Math.random()在不同JS引擎中的种子强度截然不同，这也是本次攻击仅限于Apple平台的原因:  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/xoBWaEOhvRGOibJ2FNGCBqB7PAjuvkJ06nxkUGe6icucu0JRia4XpeqVLicnDJdW7ORgqxVZLmTh1FppB6e4fbC2Fay5sic39znibmRPosQB6CJtI/640?wx_fmt=jpeg "")  
  
2026年9月，针对JavaScriptCore该弱点的公开研究与工具相继出现，演示了秒级恢复JSC Math.random()的32位种子并预测全部输出。攻击者在一个月后将其武器化并运用到本次Qubic攻击事件中  
。  
  
三、攻击手法(离线碰撞+链上清扫)  
  
①枚举: 攻击者离线遍历2^32个WeakRandom种子，按钱包的算法各生成一条55位Qubic Seed，再经KangarooTwelve派生私钥、公钥和60字符地址(Qubic矿工生态已有高度优化的K12实现，枚举成本极低)；  
  
②碰撞: 将枚举出的地址与链上有余额的地址比对，命中即意味着完整掌握了该钱包的私钥和资产；  
  
③盗币: 2026-10-17 23:42 UTC起，攻击者用受害者私钥进行转账，将余额转入7个归集地址，攻击中途即开始将部分资金经跳板地址转入MEXC充值地址套现，之后被冻结。  
  
四、通过crypto.getRandomValues修复  
  
事件发生后，Qubic发布新版本v4.26.4进行了修复。  
  
新实现通过crypto.getRandomValues(bytes)填充Uint8Array，取得取值为0–255的随机字节，使用了浏览器提供的密码学安全伪随机数生成器(CSPRNG)，解决了Math.random()随机性不足的问题。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/xoBWaEOhvREVdqWM8Kcb1db96exNjtpPSAe7pO2zDicMA6N0PsicOhE0icAjjiccl7IxA2o2GYpoBg1fIS4gQOPVnHoicwPh90cKOuxO59sJvn2U/640?wx_fmt=jpeg "")  
  
五、弱随机数问题在2026年频繁爆发  
  
①Ill Bloom/旧版CryptoJS弱随机数实现漏洞: 2026年5—8月发生多轮被盗被盗， Coinspect发现部分钱包在生成恢复短语时沿用了CryptoJS的弱随机数实现，确认受影响应用包括RRWallet、Bexo、NanChat、Bitcoin Libre和Milo，造成损失约570万美元。  
  
②Coldcard硬件钱包被盗: 2026年7月30日，Coldcard旧固件生成的弱随机性种子被利用，被盗资产涉及1,816 BTC、价值约1.16亿美元，覆盖5,200多个地址。  
  
这些案例说明: AI时代，加密算法破解的门槛被快速降低(尤其是加密算法的代码实现漏洞)，安全审计需要覆盖种子最初生成时的真实随机源、平台实现和历史依赖；仅检查密钥长度、升级当前客户端或加强生成后的保管，无法消除已形成的弱种子风险。  
  
  
https://blog.drstra.in/posts/jsc-randomness-predictor/(二)  
  
https://coinspect.com/blog/ill-bloom-investigation/(五  
①  
)  
  
https://coldcard.com/security/status(五  
②  
)  
  
引用来源: GoPlus  
  
  
