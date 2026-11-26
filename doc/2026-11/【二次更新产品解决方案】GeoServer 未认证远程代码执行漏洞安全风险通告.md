#  【二次更新产品解决方案】GeoServer 未认证远程代码执行漏洞安全风险通告  
零日雷达
                    零日雷达  安恒信息CERT   2026-11-25 09:33  
  
![零日雷达安全通告](https://mmbiz.qpic.cn/sz_mmbiz_png/P5dz00jGoySEQjNXewtZe8dZ0oWBBr2icNicCrvy51icc42SHwGVfMicVrNIshTzib1yJcqkriaAaJH8GBGvQIJm2601Spxqd4SVCXrTFy9wcqPLk/640?from=appmsg "")  
  
<table><thead><tr><th colspan="4" style="background:#08287F;color:#FFFFFF;font-weight:800;padding:11px 12px;text-align:center;font-size:15px;letter-spacing:2px;border:1px solid #08287F;"><section><span leaf="">漏洞概述</span></section></th></tr></thead><tbody><tr><td style="padding:9px 8px;border:1px solid #D2DCED;color:#08287F;font-weight:700;text-align:left;vertical-align:top;background:#F3F6FC;line-height:1.7;white-space:nowrap;font-size:13px;"><section><span leaf="">漏洞名称</span></section></td><td colspan="3" style="padding:9px 10px;border:1px solid #D2DCED;color:#334155;text-align:left;vertical-align:top;line-height:1.7;word-break:break-word;font-size:14px;"><p style="margin:0;line-height:1.9;"><span leaf="">GeoServer 未认证远程代码执行漏洞</span></p></td></tr><tr><td style="padding:9px 8px;border:1px solid #D2DCED;color:#08287F;font-weight:700;text-align:left;vertical-align:top;background:#F3F6FC;line-height:1.7;white-space:nowrap;font-size:13px;"><section><span leaf="">CVE编号</span></section></td><td colspan="3" style="padding:9px 10px;border:1px solid #D2DCED;color:#334155;text-align:left;vertical-align:top;line-height:1.7;word-break:break-word;font-size:14px;"><p style="margin:0;line-height:1.9;"><span leaf="">CVE-2026-129242、CVE-2026-129244、CVE-2026-129243、CVE-2026-120705、CVE-2026-129245</span></p></td></tr><tr><td style="padding:9px 8px;border:1px solid #D2DCED;color:#08287F;font-weight:700;text-align:left;vertical-align:top;background:#F3F6FC;line-height:1.7;white-space:nowrap;font-size:13px;"><section><span leaf="">CNVD编号</span></section></td><td colspan="3" style="padding:9px 10px;border:1px solid #D2DCED;color:#334155;text-align:left;vertical-align:top;line-height:1.7;word-break:break-word;font-size:14px;"><p style="margin:0;line-height:1.9;"><span leaf="">未分配</span></p></td></tr><tr><td style="padding:9px 8px;border:1px solid #D2DCED;color:#08287F;font-weight:700;text-align:left;vertical-align:top;background:#F3F6FC;line-height:1.7;white-space:nowrap;font-size:13px;"><section><span leaf="">CNNVD编号</span></section></td><td colspan="3" style="padding:9px 10px;border:1px solid #D2DCED;color:#334155;text-align:left;vertical-align:top;line-height:1.7;word-break:break-word;font-size:14px;"><p style="margin:0;line-height:1.9;"><span leaf="">未分配</span></p></td></tr><tr><td style="padding:9px 8px;border:1px solid #D2DCED;color:#08287F;font-weight:700;text-align:left;vertical-align:top;background:#F3F6FC;line-height:1.7;white-space:nowrap;font-size:13px;"><section><span leaf="">零日雷达编号</span></section></td><td colspan="3" style="padding:9px 10px;border:1px solid #D2DCED;color:#334155;text-align:left;vertical-align:top;line-height:1.7;word-break:break-word;font-size:14px;"><p style="margin:0;line-height:1.9;"><span leaf="">WM-202611-000004</span></p></td></tr><tr><td style="padding:9px 8px;border:1px solid #D2DCED;color:#08287F;font-weight:700;text-align:left;vertical-align:top;background:#F3F6FC;line-height:1.7;white-space:nowrap;font-size:13px;"><section><span leaf="">零日雷达评级</span></section></td><td style="padding:9px 10px;border:1px solid #D2DCED;color:#334155;text-align:left;vertical-align:top;line-height:1.7;word-break:break-word;font-size:14px;"><section><span leaf="">1级</span></section></td><td style="padding:9px 8px;border:1px solid #D2DCED;color:#08287F;font-weight:700;text-align:left;vertical-align:top;background:#F3F6FC;line-height:1.7;white-space:nowrap;font-size:13px;"><section><span leaf="">CVSS3.1评分</span></section></td><td style="padding:9px 10px;border:1px solid #D2DCED;color:#334155;text-align:left;vertical-align:top;line-height:1.7;word-break:break-word;font-size:14px;"><span style="color:#C7001A;font-weight:800;"><span leaf="">9.8</span></span></td></tr><tr><td style="padding:9px 8px;border:1px solid #D2DCED;color:#08287F;font-weight:700;text-align:left;vertical-align:top;background:#F3F6FC;line-height:1.7;white-space:nowrap;font-size:13px;"><section><span leaf="">POC情况</span></section></td><td style="padding:9px 10px;border:1px solid #D2DCED;color:#334155;text-align:left;vertical-align:top;line-height:1.7;word-break:break-word;font-size:14px;"><section><span leaf="">已发现</span></section></td><td style="padding:9px 8px;border:1px solid #D2DCED;color:#08287F;font-weight:700;text-align:left;vertical-align:top;background:#F3F6FC;line-height:1.7;white-space:nowrap;font-size:13px;"><section><span leaf="">EXP情况</span></section></td><td style="padding:9px 10px;border:1px solid #D2DCED;color:#334155;text-align:left;vertical-align:top;line-height:1.7;word-break:break-word;font-size:14px;"><section><span leaf="">已发现</span></section></td></tr><tr><td style="padding:9px 8px;border:1px solid #D2DCED;color:#08287F;font-weight:700;text-align:left;vertical-align:top;background:#F3F6FC;line-height:1.7;white-space:nowrap;font-size:13px;"><section><span leaf="">在野利用</span></section></td><td style="padding:9px 10px;border:1px solid #D2DCED;color:#334155;text-align:left;vertical-align:top;line-height:1.7;word-break:break-word;font-size:14px;"><section><span leaf="">未发现</span></section></td><td style="padding:9px 8px;border:1px solid #D2DCED;color:#08287F;font-weight:700;text-align:left;vertical-align:top;background:#F3F6FC;line-height:1.7;white-space:nowrap;font-size:13px;"><section><span leaf="">研究情况</span></section></td><td style="padding:9px 10px;border:1px solid #D2DCED;color:#334155;text-align:left;vertical-align:top;line-height:1.7;word-break:break-word;font-size:14px;"><section><span leaf="">已复现</span></section></td></tr><tr><td style="padding:9px 8px;border:1px solid #D2DCED;color:#08287F;font-weight:700;text-align:left;vertical-align:top;background:#F3F6FC;line-height:1.7;white-space:nowrap;font-size:13px;"><section><span leaf="">危害描述</span></section></td><td colspan="3" style="padding:9px 10px;border:1px solid #D2DCED;color:#334155;text-align:left;vertical-align:top;line-height:1.7;word-break:break-word;font-size:14px;"><p style="margin:0;line-height:1.9;"><span leaf="">利用该漏洞不需要任何登录凭证，攻击者即可在目标 GeoServer 服务器上执行任意系统命令。若按官方容器镜像的默认配置部署，GeoServer 进程以 root 身份运行，攻击者一旦利用成功即完全掌控容器，并可能进一步威胁宿主机；若以普通用户运行，危害范围虽受进程权限限制，仍可能造成空间数据与安全配置外泄、服务中断，或被用作进一步控制主机的跳板。</span></p></td></tr></tbody></table>  
  
该产品主要使用客户行业分布广泛，漏洞危害性极高，建议客户尽快做好自查及防护。  
  
**安恒研究院卫兵实验室已通过恒脑·零日雷达复现此漏洞，成功复现远程代码执行。**  
  
![repro-01](https://mmbiz.qpic.cn/mmbiz_png/P5dz00jGoyR34Q1qCZW4p0yQSia7hQLgbIiaicBkwwzCf2WSrxLbpN39HJRD2NHibiaD9SocYriaKOX3h1nYMglURLKEUpGeUWUW4O7icWlcADA854/640?from=appmsg "")  
  
01### 漏洞信息  
  
VULNERABILITY  
  
GeoServer 是 OSGeo 维护的 Java 地理空间服务端，按照 OGC 规范对外提供 WMS、WFS、WCS 等空间数据服务，常见于政务、测绘、自然资源及行业 GIS 平台的数据发布环节。  
  
漏洞描述  
  
漏洞危害等级：严重  
  
漏洞类型：远程代码执行  
  
影响范围  
  
影响版本：  
  
GeoServer<= 2.27.6  
  
2.28.0<=GeoServer<=2.28.5  
  
3.0.0<=GeoServer<=3.0.1  
  
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
  
建议将 GeoServer 升级至 3.0.2、2.28.6 或 2.27.7，或对应分支的更高版本。升级前应核对下载页与发布说明中的版本号，并确认现有扩展组件、数据目录与新版本兼容。这条利用路径由五个相互独立的缺陷串联而成，只修补其中一处并不能消除组合利用的风险。  
  
信息来源：https://geoserver.org/download/  
  
临时缓解方案  
  
暂时无法升级时，可先收缩对外暴露面：WFS 服务仅向可信来源开放；取消匿名访问，统一实施身份认证和权限控制；无实际业务需求时停用存储查询；Web 管理界面、REST 接口及用户/角色服务限制在受控网段访问。上述手段只能破坏攻击链的部分前置条件，无法规避配置读取、会话伪造和反序列化执行等核心风险，不能代替升级；相关限制可能影响依赖匿名 WFS 的业务，升级完成后应逐项复核并恢复。  
  
信息来源：https://cryptocat.me/blog/research/analysis/geoserver_preauth_rce/  
  
  
03### 产品能力覆盖  
  
COVERAGE  
  
<table><thead><tr><th data-colwidth="244" style="background:#08287F;color:#FFFFFF;font-weight:700;padding:8px 12px;text-align:left;font-size:14px;"><section><span leaf="">产品名称</span></section></th><th style="background:#08287F;color:#FFFFFF;font-weight:700;padding:8px 12px;text-align:left;font-size:14px;"><section><span leaf="">覆盖补丁包</span></section></th></tr></thead><tbody><tr><td data-colwidth="244" style="padding:8px 12px;border-bottom:1px solid #D2DCED;color:#334155;line-height:1.7;word-break:break-word;font-size:14px;"><section><span leaf=""><span textstyle="" style="font-size: 14px;">明鉴漏洞扫描系统</span></span></section></td><td style="padding:8px 12px;border-bottom:1px solid #D2DCED;color:#334155;line-height:1.7;word-break:break-word;font-size:14px;"><section><span leaf=""><span textstyle="" style="font-size: 14px;">WEB包版本号V1.3.2539.2166</span></span></section></td></tr><tr><td data-colwidth="244" style="padding:8px 12px;border-bottom:1px solid #D2DCED;color:#334155;line-height:1.7;word-break:break-word;font-size:14px;background:#F3F6FC;"><section><span leaf=""><span textstyle="" style="font-size: 14px;">明御APT攻击预警平台</span></span></section></td><td style="padding:8px 12px;border-bottom:1px solid #D2DCED;color:#334155;line-height:1.7;word-break:break-word;font-size:14px;background:#F3F6FC;"><section><span leaf=""><span textstyle="" style="font-size: 14px;">主机包版本号20261127</span></span></section></td></tr><tr><td data-colwidth="244" style="padding:8px 12px;border-bottom:1px solid #D2DCED;color:#334155;line-height:1.7;word-break:break-word;font-size:14px;"><section><span leaf=""><span textstyle="" style="font-size: 14px;">玄武盾云防护</span></span></section></td><td style="padding:8px 12px;border-bottom:1px solid #D2DCED;color:#334155;line-height:1.7;word-break:break-word;font-size:14px;"><section><span leaf=""><span textstyle="" style="font-size: 14px;">主机包版本号1.4.10；WEB 包版本号1.4.10</span></span></section></td></tr></tbody></table>  
  
  
04### 参考资料  
  
REFERENCES  
  
[01]  
https://github.com/geoserver/geoserver/security/advisories/GHSA-4v26-c537-jvxc  
  
[02]  
https://github.com/geoserver/geoserver/security/advisories/GHSA-rv5g-g2r9-58rj  
  
[03]  
https://github.com/geoserver/geoserver/security/advisories/GHSA-39mw-5chv-jh47  
  
[04]  
https://github.com/geoserver/geoserver/security/advisories/GHSA-hgm3-wq4h-8vvq  
  
[05]  
https://github.com/geoserver/geoserver/security/advisories/GHSA-9wf2-fcgq-vxhq  
  
[06]  
https://cryptocat.me/blog/research/analysis/geoserver_preauth_rce/  
  
[07]  
https://geoserver.org/download/  
  
  
05### 技术支持  
  
SUPPORT  
  
如有漏洞相关需求支持请联系  
  
400-6059-110  
  
获取相关能力支撑。  
  
