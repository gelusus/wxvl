#  【已复现】VMware vCenter 认证绕过漏洞安全风险通告  
零日雷达
                    零日雷达  安恒信息CERT   2026-11-25 10:31  
  
![零日雷达安全通告](https://mmbiz.qpic.cn/mmbiz_png/P5dz00jGoyQN4sdnwAEW4XykwRcC3wnCBPBIqODOcmueoBXMukIXMpicibS3bj2SiauqvA6UhX4AzTZjyoA8rK7QFeyjZliauUTsf6TxAYMTHjw/640?from=appmsg "")  
  
<table><thead><tr><th colspan="4" style="background:#08287F;color:#FFFFFF;font-weight:800;padding:11px 12px;text-align:center;font-size:15px;letter-spacing:2px;border:1px solid #08287F;"><section><span leaf="">漏洞概述</span></section></th></tr></thead><tbody><tr><td style="padding:9px 8px;border:1px solid #D2DCED;color:#08287F;font-weight:700;text-align:left;vertical-align:top;background:#F3F6FC;line-height:1.7;white-space:nowrap;font-size:13px;"><section><span leaf="">漏洞名称</span></section></td><td colspan="3" style="padding:9px 10px;border:1px solid #D2DCED;color:#334155;text-align:left;vertical-align:top;line-height:1.7;word-break:break-word;font-size:14px;"><p style="margin:0;line-height:1.9;"><span leaf="">VMware vCenter 认证绕过漏洞（CVE-2026-123903）</span></p></td></tr><tr><td style="padding:9px 8px;border:1px solid #D2DCED;color:#08287F;font-weight:700;text-align:left;vertical-align:top;background:#F3F6FC;line-height:1.7;white-space:nowrap;font-size:13px;"><section><span leaf="">CVE编号</span></section></td><td colspan="3" style="padding:9px 10px;border:1px solid #D2DCED;color:#334155;text-align:left;vertical-align:top;line-height:1.7;word-break:break-word;font-size:14px;"><p style="margin:0;line-height:1.9;"><span leaf="">CVE-2026-123903</span></p></td></tr><tr><td style="padding:9px 8px;border:1px solid #D2DCED;color:#08287F;font-weight:700;text-align:left;vertical-align:top;background:#F3F6FC;line-height:1.7;white-space:nowrap;font-size:13px;"><section><span leaf="">CNVD编号</span></section></td><td colspan="3" style="padding:9px 10px;border:1px solid #D2DCED;color:#334155;text-align:left;vertical-align:top;line-height:1.7;word-break:break-word;font-size:14px;"><p style="margin:0;line-height:1.9;"><span leaf="">未分配</span></p></td></tr><tr><td style="padding:9px 8px;border:1px solid #D2DCED;color:#08287F;font-weight:700;text-align:left;vertical-align:top;background:#F3F6FC;line-height:1.7;white-space:nowrap;font-size:13px;"><section><span leaf="">CNNVD编号</span></section></td><td colspan="3" style="padding:9px 10px;border:1px solid #D2DCED;color:#334155;text-align:left;vertical-align:top;line-height:1.7;word-break:break-word;font-size:14px;"><p style="margin:0;line-height:1.9;"><span leaf="">未分配</span></p></td></tr><tr><td style="padding:9px 8px;border:1px solid #D2DCED;color:#08287F;font-weight:700;text-align:left;vertical-align:top;background:#F3F6FC;line-height:1.7;white-space:nowrap;font-size:13px;"><section><span leaf="">零日雷达编号</span></section></td><td colspan="3" style="padding:9px 10px;border:1px solid #D2DCED;color:#334155;text-align:left;vertical-align:top;line-height:1.7;word-break:break-word;font-size:14px;"><p style="margin:0;line-height:1.9;"><span leaf="">WM-202611-000005</span></p></td></tr><tr><td style="padding:9px 8px;border:1px solid #D2DCED;color:#08287F;font-weight:700;text-align:left;vertical-align:top;background:#F3F6FC;line-height:1.7;white-space:nowrap;font-size:13px;"><section><span leaf="">零日雷达评级</span></section></td><td style="padding:9px 10px;border:1px solid #D2DCED;color:#334155;text-align:left;vertical-align:top;line-height:1.7;word-break:break-word;font-size:14px;"><section><span leaf="">3级</span></section></td><td style="padding:9px 8px;border:1px solid #D2DCED;color:#08287F;font-weight:700;text-align:left;vertical-align:top;background:#F3F6FC;line-height:1.7;white-space:nowrap;font-size:13px;"><section><span leaf="">CVSS3.1评分</span></section></td><td style="padding:9px 10px;border:1px solid #D2DCED;color:#334155;text-align:left;vertical-align:top;line-height:1.7;word-break:break-word;font-size:14px;"><span style="color:#C7001A;font-weight:800;"><span leaf="">9.8</span></span></td></tr><tr><td style="padding:9px 8px;border:1px solid #D2DCED;color:#08287F;font-weight:700;text-align:left;vertical-align:top;background:#F3F6FC;line-height:1.7;white-space:nowrap;font-size:13px;"><section><span leaf="">POC情况</span></section></td><td style="padding:9px 10px;border:1px solid #D2DCED;color:#334155;text-align:left;vertical-align:top;line-height:1.7;word-break:break-word;font-size:14px;"><section><span leaf="">已发现</span></section></td><td style="padding:9px 8px;border:1px solid #D2DCED;color:#08287F;font-weight:700;text-align:left;vertical-align:top;background:#F3F6FC;line-height:1.7;white-space:nowrap;font-size:13px;"><section><span leaf="">EXP情况</span></section></td><td style="padding:9px 10px;border:1px solid #D2DCED;color:#334155;text-align:left;vertical-align:top;line-height:1.7;word-break:break-word;font-size:14px;"><section><span leaf="">已发现</span></section></td></tr><tr><td style="padding:9px 8px;border:1px solid #D2DCED;color:#08287F;font-weight:700;text-align:left;vertical-align:top;background:#F3F6FC;line-height:1.7;white-space:nowrap;font-size:13px;"><section><span leaf="">在野利用</span></section></td><td style="padding:9px 10px;border:1px solid #D2DCED;color:#334155;text-align:left;vertical-align:top;line-height:1.7;word-break:break-word;font-size:14px;"><section><span leaf="">未发现</span></section></td><td style="padding:9px 8px;border:1px solid #D2DCED;color:#08287F;font-weight:700;text-align:left;vertical-align:top;background:#F3F6FC;line-height:1.7;white-space:nowrap;font-size:13px;"><section><span leaf="">研究情况</span></section></td><td style="padding:9px 10px;border:1px solid #D2DCED;color:#334155;text-align:left;vertical-align:top;line-height:1.7;word-break:break-word;font-size:14px;"><section><span leaf="">已复现</span></section></td></tr><tr><td style="padding:9px 8px;border:1px solid #D2DCED;color:#08287F;font-weight:700;text-align:left;vertical-align:top;background:#F3F6FC;line-height:1.7;white-space:nowrap;font-size:13px;"><section><span leaf="">危害描述</span></section></td><td colspan="3" style="padding:9px 10px;border:1px solid #D2DCED;color:#334155;text-align:left;vertical-align:top;line-height:1.7;word-break:break-word;font-size:14px;"><p style="margin:0;line-height:1.9;"><span leaf="">攻击者只需能够访问 vCenter 目录服务端口，提交一次以 Administrator@vsphere.local 为用户名、口令错误的 SASL SRP 认证，即可在该失败连接上获得管理员权限，进而创建管理员账号并接管 vCenter。</span></p></td></tr></tbody></table>  
  
该产品主要使用客户行业分布广泛，漏洞危害性高，建议客户尽快做好自查及防护。  
  
**安恒研究院卫兵实验室已通过恒脑·零日雷达复现此漏洞，成功获取管理员权限。**  
  
![复现截图](https://mmbiz.qpic.cn/sz_mmbiz_png/P5dz00jGoySAaPIMop2MGNGKdrgpdvlP5FJbibRPtGmN13aqibAQwbu9q5mUAXcju00ibFFQB13uhZSjGlNF9c1ENNI2h3oiccqEM6Odxo63LaE/640?from=appmsg "")  
  
01### 漏洞信息  
  
VULNERABILITY  
  
VMware vCenter 是 VMware 虚拟化环境的核心管理平台，用于统一管理 ESXi 主机与虚拟机。  
  
漏洞描述  
  
漏洞危害等级：严重  
  
漏洞类型：安全特性绕过  
  
影响范围  
  
影响版本：  
  
VMware vCenter 7.0、8.0（低于 8.0 U3m）、9.0.x（低于 9.0.2.0300）、9.1.x（低于 9.1.2）；  
  
VMware Cloud Foundation 5.x（低于 5.2.5）；  
  
VMware Telco Cloud Platform 3.0、4.x、5.0.x、5.1.x、5.2；  
  
VMware Telco Cloud Infrastructure 3.0  
  
CVSS向量  
  
访问途径（AV）：网络  
  
攻击复杂度（AC）：低  
  
所需权限（PR）：无需任何权限  
  
用户交互（UI）：不需要用户交互  
  
影响范围（S）：不变  
  
机密性影响（C）：高  
  
完整性影响（I）：高  
  
可用性影响（A）：高  
  
  
02### 修复方案  
  
FIX  
  
官方修复方案  
  
按 VMSA-2026-0011 响应矩阵升级至修复版本：VMware vCenter 9.1.x 升级至 9.1.2；VMware vCenter 9.0.x 升级至 9.0.2.0300；VMware vCenter 8.0 升级至 8.0 U3m；VMware Cloud Foundation 5.x 升级至 5.2.5；VMware vCenter 7.0 请联系 Broadcom 支持获取修复；VMware Telco Cloud Platform 与 Telco Cloud Infrastructure 的修复方式参见 KB463501。补丁为累积更新，可一并修复同类 vCenter 漏洞。  
  
信息来源：https://support.broadcom.com/web/ecx/support-content-notification/-/external/content/SecurityAdvisories/0/39626  
  
临时缓解方案  
  
以下措施为基于漏洞暴露面的保守缓解，可能影响正常管理运维，升级完成后应及时恢复：通过防火墙或安全组限制 TCP 389、636、2012、443 等管理端口仅对运维网段开放，禁止办公网、业务网及互联网直连；收敛 vCenter 管理面（含 /ui、/sdk）的对外暴露，远程管理场景通过跳板机或 VPN 接入；加强 vCenter 与 SSO 账号审计，排查是否存在非预期创建的管理员账号与异常认证记录。  
  
信息来源：https://brcm.tech/vmsa-2026-0011-qna  
  
  
03### 参考资料  
  
REFERENCES  
  
[01]  
https://support.broadcom.com/web/ecx/support-content-notification/-/external/content/SecurityAdvisories/0/39626  
  
[02]  
https://brcm.tech/vmsa-2026-0011-qna  
  
  
04### 技术支持  
  
SUPPORT  
  
如有漏洞相关需求支持请联系  
  
400-6059-110  
  
获取相关能力支撑。  
  
