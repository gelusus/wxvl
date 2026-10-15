#  CISA 警告管理员立即修复被利用为零日漏洞的 Ivanti 漏洞  
爱拍照的老李
                    爱拍照的老李  爱拍照的老李   2026-10-15 01:03  
  
**导****读**  
  
  
  
CISA敦促立即修补一个 Ivanti Connect Secure (ICS) 漏洞，该漏洞自10月1日起就被用于零日攻击。该漏洞编号为CVE-2026-104935，是高严重级别的身份验证绕过漏洞，远程攻击者通过无需用户交互的低复杂度攻击，在目标工业控制系统（ICS）上执行任意命令。  
  
  
Ivanti 于周二发布了 ICS 22.8R2.5、25.1.3.0、25.1.3.1 和 25.1.1.2 版本，以修复这一安全漏洞，并警告客户因“该漏洞已被有限利用”而“立即采取行动”。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/PaFY6wibdwyI2Mh1XviaWfvGq46EEGLVGaEDA2wPVPBzQA0ELgCHKNPIHHoHsc4cP8twAhEgiac2kG7nI6Q2MwzbHJ8N1UlkkG4WWok31xRsfE/640?wx_fmt=png&from=appmsg "")  
  
  
尽管 Ivanti 未披露针对 CVE-2026-104935 的持续攻击相关更多信息，但网络安全公司 Volexity 报告称，其追踪的名为 UTA0576 的未知威胁组织自至少10月3日起，已在攻击中将该安全漏洞与另外两个漏洞组合利用。  
  
  
“该攻击链结合了身份验证绕过漏洞、远程代码执行（RCE）漏洞和权限提升漏洞，实现了无需身份验证的  
root  
权限代码执行。远程代码执行和身份验证绕过漏洞已被收录为 CVE-2026-104935。”Volexity表示。  
  
  
Volexity 识别出针对 Active Directory 凭据和密钥的威胁行为者实施的一系列后渗透操作，包括本地管理员密码解决方案（LAPS）密码和 BitLocker 恢复密钥。  
  
  
威胁情报公司 Defused 也证实了Volexity 的报告，称其于10月1日首次在野外发现攻击者利用 CVE-2026-104935 漏洞的证据。  
  
  
互联网威胁监督机构 Shadowserver 目前追踪到超过 5600 个带有工业控制系统（ICS）指纹的互联网暴露主机。目前没有信息说明其中有多少主机能抵御持续的攻击，也没有多少是用于捕获恶意活动的蜜罐。  
  
  
CISA 建议防御人员在发现 Ivanti ICS 设备被入侵时采取以下措施：  
- 创建设备配置的备份。  
- 执行恢复出厂设置并将设备升级至最新版本（对于云环境和虚拟系统，需重建虚拟实例并升级至最新版本）。  
- 从步骤1创建的备份中恢复设备配置。  
- 重置所有账户的密码，包括域账户、管理员账户和本地账户。撤销并重新颁发存储的证书；重置存储的API密钥以及许可服务器凭证。  
自2021年11月以来，CISA已将36个Ivanti漏洞加入其被主动利用的安全问题清单，其中12个漏洞还曾被用于勒索软件攻击。  
  
  
技术报告：  
  
https://www.volexity.com/blog/2026/10/13/a-series-of-unauthenticated-events-chained-exploitation-of-ivanti-connect-secure-zero-days/  
  
****  
扫码关注  
  
爱拍照的老李  
  
**讲述普通人能听懂的安全故事**  
  
  
