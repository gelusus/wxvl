#  漏洞预警 | VMware Directory Service认证绕过漏洞  
浅安
                    浅安  浅安安全   2026-11-26 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-59309  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
VMware vCenter Server是VMware推出的集中式虚拟化管理平台，用于管理vSphere环境中的ESXi主机、虚拟机、存储与网络资源。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/7stTqD182SUsIwGlCfN2dZicibqWiaNHKgXtZoficGruNicBicXQfnPOYAIFEvOX4wRXia41InkIOZ4ZlaWwqzhEMoK6w/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-59309**  
  
**漏洞类型：**  
认证绕过  
  
**影响：**  
服务接管  
  
**简述：**  
VMware Directory Service存在认证绕过漏洞，由于其SRP认证流程对认证算法实现存在缺陷，攻击者可通过该漏洞绕过身份验证机制，直接获得对vCenter系统的未授权访问权限。  
  
**0x04 影响版本**  
- VMware vCenter Server 9.1.x.x < 9.1.0.0300  
  
- VMware vCenter Server 9.0.x.x < 9.0.2.0100  
  
- VMware vCenter Server 8.0 < 8.0 U3k  
  
- VMware Cloud Foundation 9.1.x.x  
  
- VMware Cloud Foundation 9.0.x.x  
  
- VMware Cloud Foundation 5.x  
  
- VMware vSphere Foundation 9.1.x.x  
  
- VMware vSphere Foundation 9.0.x.x  
  
- VMware Telco Cloud Infrastructure 3.0  
  
- VMware Telco Cloud Platform 5.1.x  
  
- VMware Telco Cloud Platform 5.0.x  
  
- VMware Telco Cloud Platform 4.x  
  
**0x05 POC状态**  
- 已公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://www.vmware.com/cn/  
  
  
  
