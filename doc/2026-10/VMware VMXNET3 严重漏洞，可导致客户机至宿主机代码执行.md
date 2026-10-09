#  VMware VMXNET3 严重漏洞，可导致客户机至宿主机代码执行  
 网安百色   2026-10-09 10:12  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WibvcdjxgJnsB1KkDSwADTicRhFCQmqYp58m1xsibq4cVKiaSHgXQQ4sKpNTtRtriboJ3ObHxEf8Dv9iahB9DRpdicPdr09IhSnQM9rZsF1mpArCibk/640?wx_fmt=png&from=appmsg "")  
  
安全研究人员 0xCyberstan 现已公开了 **CVE-2026-59346**  
 的概念验证（PoC）代码。该漏洞是 VMware VMXNET3 虚拟网络适配器中存在的一个严重的整数溢出漏洞。  
  
该漏洞允许拥有客户机虚拟机（Guest VM）管理员权限的攻击者在宿主机（Host）上执行代码。然而，目前公布的 PoC 仅能导致宿主机进程崩溃，尚不包含可成功执行代码的利用程序（Exploit）。  
  
博通（Broadcom）在 CVSS v3 评分系统中将该漏洞的严重等级评为 **9.3 分**  
，并已在 2026 年 9 月 3 日发布的 VMware Workstation 和 Fusion **26H1u1**  
 版本中提供了修复补丁。  
  
其安全公告指出，受影响的版本包括 Workstation 和 Fusion 的 25H2 及 26H1 版本。厂商同时强调，目前尚无可用的临时缓解措施（workaround）。  
  
该漏洞存在于宿主机 vmware-vmx  
 进程中的 **TCP 分段卸载（TSO, TCP Segmentation Offload）**  
 处理路径中。TSO 的作用是将大型网络数据包拆分为较小的分段。存在漏洞的代码例程在计算这些分段所需的内存大小时，会将分段数量与每个分段所需的空间进行相乘。  
  
该计算过程使用的是 32 位乘法运算。当乘积结果超出 32 位整数所能表示的范围时，数值会发生回绕（wrap around），变成一个较小的值。随后，宿主机将使用这个被缩小的值来分配内存缓冲区。  
  
与此同时，数据复制循环仍会按照原始的分段数量继续执行。这导致由客户机控制的数据包数据被写入到已分配缓冲区的边界之外，从而造成**越界写入（Out-of-Bounds Write）**  
。  
  
研究人员指出，该弱点与之前 **CVE-2025-41236**  
 漏洞所修复的代码路径相同。早期的修复检查虽然将单个数据包字段及其总和限制在 9,216 以内，但并未对最终的乘法运算结果进行有效性验证。  
### VMware VMXNET3 漏洞 PoC 发布  
  
该 PoC 代码已在 GitHub 仓库公开，它以 Linux 内核模块的形式在配置了 VMXNET3 适配器的客户机内部运行。该模块直接写入网络发送描述符，绕过客户机驱动程序的常规 TSO 处理逻辑，随后请求宿主机进行处理。执行此操作需要客户机内部具备足够的权限，因此它**不属于**  
未经身份验证的远程网络攻击。  
  
由此产生的越界写入会触及未映射的内存区域，导致 vmware-vmx  
 进程发生段错误（Segmentation fault）并崩溃，进而致使受影响的虚拟机意外关机。  
  
0xCyberstan 的代码仓库警告称，测试该 PoC 可能会破坏客户机未保存的状态（数据），并在文档中说明了其测试环境配置：在 Ubuntu 宿主机上运行 VMware Workstation Pro 25.0.1，客户机系统为 Alpine Linux。  
  
本公众号所载文章为本公众号原创或根据网络搜索下载编辑整理，文章版权归原作者所有，仅供读者学习、参考，禁止用于商业用途。因转载众多，无法找到真正来源，如标错来源，或对于文中所使用的图片、文字、链接中所包含的软件/资料等，如有侵权，请跟我们联系删除，谢谢！  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/1QIbxKfhZo5lNbibXUkeIxDGJmD2Md5vKicbNtIkdNvibicL87FjAOqGicuxcgBuRjjolLcGDOnfhMdykXibWuH6DV1g/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&randomid=p6hk1x4r&tp=webp#imgIndex=1 "")  
  
