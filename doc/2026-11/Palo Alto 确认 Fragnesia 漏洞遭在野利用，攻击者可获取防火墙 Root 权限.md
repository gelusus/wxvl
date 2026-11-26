#  Palo Alto 确认 Fragnesia 漏洞遭在野利用，攻击者可获取防火墙 Root 权限  
 看雪学苑   2026-11-25 10:06  
  
Palo Alto Networks 于 2026 年 11 月 25 日发布安全公告，确认**攻击者正在利用 Linux 内核提权漏洞 Fragnesia（CVE-2026-46300），获取运行 PAN-OS 的防火墙设备 Root 权限。**  
  
  
根据厂商披露的信息，目前已观察到的攻击均**针对管理接口暴露于互联网的设备。**  
Palo Alto Networks 建议管理接口可从公网访问的客户立即升级至修复版本，并将管理访问限制在可信任的内部 IP 地址范围内。  
  
  
**No.1**  
  
**攻击链涉及两个漏洞**  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Cpo2XCpI7K3eZnCu4OPSxVd3hZQV2nWBHyKgD7byqALrmYAl1eeTxFx9jD6KficJzynIyegCz2dAEnKuqCPAhYRf2qIfwpmzjlrLJTBUAbE0/640?wx_fmt=png&from=appmsg "")  
  
  
Fragnesia 是一个本地权限提升漏洞，攻击者利用该漏洞可以将已有的有限权限提升至系统最高权限 Root，从而获得对底层操作系统的控制能力，包括修改系统文件及干扰防火墙正常运行。  
  
  
Palo Alto Networks 指出，Fragnesia 并非攻击者最初进入设备的入口。目前已知的攻击路径始于 PHP2Shell 漏洞：攻击者首先通过暴露于互联网的管理接口利用 PHP2Shell 执行代码，随后利用 Fragnesia 将权限提升至 Root。  
  
  
此前，厂商旗下 Unit 42 研究团队已于 10 月披露针对 PAN-OS 的 PHP2Shell 攻击活动，并确认攻击者将这两个漏洞结合使用。  
  
  
**No.2**  
  
**PAN-OS 多个版本已发布修复**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K0DNmybDUBK2wibAib25WAJr8biaqrxHK0wBVbV6DJxDgWicZ5SmpjwtkJg9rqDoeh9KkLbI9USlyyVSFz2Fq4prT48EjhY9Qe3RYk/640?wx_fmt=png&from=appmsg "")  
  
  
11 月 25 日发布的安全公告列出了五个 PAN-OS 软件分支的修复版本：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K2WibZGib5qRgs9IOicmsronk2EVN4jAMkPZp9XoUQDSVCv87MuCp9yXm6RiaeojBSRCK54aEwMWpdeL5BV1kHkYpE87aPOON9rLyc/640?wx_fmt=png&from=appmsg "")  
  
  
对于仍在使用较早维护版本的用户，应对照厂商完整修复版本列表进行核查，特别注意版本号中的 Hotfix 后缀。  
  
  
厂商同时说明，此前 PHP2Shell 安全公告中列出的部分修复版本，也能够阻断目前已观察到的攻击路径，但这并不意味着所有 PHP2Shell 修复版本都同时修复了 Fragnesia 漏洞。例如，PAN-OS 12.2.4-h1 修复了 PHP2Shell，而修复 Fragnesia 的版本为 12.2.4-h3。  
  
  
**No.3**  
  
**厂商建议检查管理接口暴露情况**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Cpo2XCpI7K3fAc6owroyfdhkqrt40cMb0SfKiaBPR2wORe47iafr8vYmKE6DzZBmSicYBxjPanYHABgI8EI7cvibZxdqYQzsjmpoJyks6gtVSsg/640?wx_fmt=png&from=appmsg "")  
  
  
Palo Alto Networks 建议管理员同时检查专用管理端口，以及启用了管理访问功能的其他网络端口。如果这些接口能够从互联网直接访问，应立即升级并取消公网直接访问权限。  
  
  
厂商推荐使用独立管理网络和跳板机进行设备管理，避免管理服务直接暴露在公网。  
  
  
对于此前曾暴露管理接口的设备，管理员还应检查管理员操作记录和配置日志，排查是否存在异常变更。完成漏洞修复并不能确认设备在升级前未遭到入侵，因此仍需进行必要的安全审查。  
  
  
本次安全公告被厂商标记为信息性公告，尚未披露实际遭到入侵的防火墙数量。对于已经将管理访问限制在可信任内部 IP 地址范围内的设备，厂商建议在下一次计划维护窗口完成 Fragnesia 修复。  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Cpo2XCpI7K2bcKOzJaF3OhVOqDPiaDZmBtKj1PwE0ia8mdRwIVD29bP4QVkX3qWic5gtw3gDicCP8yH3lPCvFic23bGk3loXLpGREJmubhZtCcR0/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Cpo2XCpI7K3eOLdu2rKoibgSuuTR15s6Ky9kphx4CAdmWT3gTscC1szgsPNNmtsJhbuEtsXEjZHzhnnicxag0ib7qPkAgicQnmbS44JK4eaDCVQ/640?wx_fmt=gif&from=appmsg "")  
  
**球分享**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Cpo2XCpI7K10rxqCOBWKGBebHAsibOJfvvyrib3bTTL6FdMfgZ2P8XxCZ3wu11CANybzY8KOb35dkic7OFbhb6X35wWkFkE23sSQMA06AjI1ibc/640?wx_fmt=gif&from=appmsg "")  
  
**球点赞**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/Cpo2XCpI7K3We6CRMDxwGVkmgZp4lOCcJOQGic6MCYMiby0oibUq18bzFTcw2en8NpTSLlPVyOwrqiaqEsxTDJ7n6G4BmTpDaJArRGJiaoQqhg88/640?wx_fmt=gif&from=appmsg "")  
  
**球在看**  
  
