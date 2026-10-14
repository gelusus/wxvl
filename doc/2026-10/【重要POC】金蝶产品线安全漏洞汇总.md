#  【重要POC】金蝶产品线安全漏洞汇总  
原创 mourenzheng
                    mourenzheng  网络安全民工   2026-10-14 14:46  
  
# 金蝶产品线安全漏洞汇总  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/T0ibbhsCmribRHXPZlnZibPmZnibIk6K5OlJOQE4J5DynvCaDpos5wBPudvG8JWzsXtphSCs3icjjU6yqjZoH8ibuXUUxpZMV5AU16icJVICY4SqDE/640?wx_fmt=png&from=appmsg "")  
> 整理范围：金蝶云星空（K3Cloud）、金蝶 EAS / EAS Cloud、金蝶 Apusic 应用服务器、金蝶 KIS 云、金蝶 AI 苍穹 / 星瀚，以及部分历史产品线。  
  
 用途：安全自查、漏洞管理、补丁跟踪、WAF/IPS 规则梳理。  
  
 注意：请仅用于授权安全测试与防御加固，禁止未授权攻击。  
  
## 目录  
- 一、金蝶云星空（K3Cloud / Cloud-Starry-Sky）  
  
- 二、金蝶 EAS / EAS Cloud  
  
- 三、金蝶 Apusic 应用服务器（天燕中间件）  
  
- 四、金蝶 KIS 云  
  
- 五、金蝶 AI 苍穹 / 星瀚  
  
- 六、金蝶医疗 / 云之家（历史）  
  
- 七、新增漏洞清单速览  
  
- 八、通用修复与缓解建议  
  
- 九、参考来源  
  
# 一、金蝶云星空（K3Cloud / Cloud-Starry-Sky）  
## 1.1 反序列化远程代码执行（BinaryFormatter RCE）  
### 1.1.1 .kdsvc 接口 BinaryFormatter 反序列化 RCE  
- **漏洞标题**  
：金蝶云星空管理中心/应用层 BinaryFormatter 反序列化远程代码执行漏洞  
  
- **漏洞概述**  
：管理中心通信层默认采用 BinaryFormatter，序列化与反序列化过程未签名或校验。攻击者可篡改序列化数据，在服务端反序列化时造成远程命令执行，完全控制服务器。  
  
- **影响版本**  
：  
  
- 6.x：≤ 6.2.1012.4  
  
- 7.x：7.0.352.16 ~ 7.7.0.202111  
  
- 8.x：8.0.0.202205 ~ 8.1.0.20221110  
  
- **漏洞 URL**  
：已公开 9 个 .kdsvc  
 接口，路径前缀 /K3Cloud/  
：  
  
- Kingdee.BOS.ServiceFacade.ServicesStub.User.UserService.SaveUserPassport.common.kdsvc?dcInfo=11  
  
- Kingdee.BOS.ServiceFacade.ServicesStub.InOutDataService.GetImportOutData.common.kdsvc  
  
- Kingdee.BOS.ServiceFacade.ServicesStub.DevReportService.GetBusinessobjectData.common.kdsvc  
  
- Kingdee.BOS.ServiceFacade.ServicesStub.ServiceGateway.GetServiceUri.common.kdsvc  
  
- Kingdee.BOS.ServiceFacade.ServicesStub.DynamicForm.DynamicFormService.CloseForm.common.kdsvc  
  
- Kingdee.BOS.ServiceFacade.ServicesStub.AppDesigner.AppDesignerService.RecordCurDevCodeInfo.common.kdsvc  
  
- Kingdee.BOS.ServiceFacade.ServicesStub.BusinessData.BusinessDataService.BatchLoad.common.kdsvc  
  
- Kingdee.BOS.ServiceFacade.ServicesStub.Account.AccountService.GetDataCenterList.common.kdsvc  
  
- Kingdee.BOS.ServiceFacade.ServicesStub.BusinessData.BusinessDataService.Audit.common.kdsvc  
  
- **POC/利用要点**  
：  
  
- 请求方法：POST  
  
- Content-Type: text/json  
  
- 参数格式示例：ap0/ap1/ap2  
 按方法参数位置传入序列化数据，format=3  
 或 format=Binary  
  
- 也支持 parameters  
 参数格式  
  
- 请求头 cmd  
 可指定命令；仅部分链支持回显  
  
- **漏洞编号**  
：QVD-2023-14179 / CNVD-2024-13011  
  
- **修复建议**  
：升级官方补丁；限制 .kdsvc  
 接口暴露；禁用不安全 BinaryFormatter；增加签名校验与白名单。  
  
### 1.1.2 ServiceGateway.GetServiceUri.common 反序列化漏洞  
- **漏洞标题**  
：金蝶云星空 ServiceGateway.GetServiceUri.common 接口反序列化漏洞  
  
- **漏洞概述**  
：servicegateway-getserviceuri-common  
 存在反序列化漏洞，外部攻击者可通过恶意手段获取权限，进而接管服务器控制权。  
  
- **影响版本**  
：金蝶云星空 6.x / 7.x / 8.x 私有云和混合云部署  
  
- **漏洞 URL**  
：/Kingdee.BOS.ServiceFacade.ServicesStub.ServiceGateway.GetServiceUri.common.kdsvc  
  
- **漏洞编号**  
：攻击 ID 1070210427  
  
- **备注**  
：2024 年攻防演练中列为高危漏洞。  
  
### 1.1.3 UserService 反序列化漏洞  
- **漏洞标题**  
：金蝶云星空 UserService 接口反序列化漏洞  
  
- **漏洞概述**  
：UserService  
 接口存在反序列化漏洞，未授权攻击者可直接利用此漏洞执行系统命令，获取服务器权限。  
  
- **影响版本**  
：金蝶云星空  
  
- **漏洞 URL**  
：/Kingdee.BOS.ServiceFacade.ServicesStub.User.UserService.*  
  
- **检测**  
：已有公开 Nuclei 模板 kingdee-cloud-user-deserialization-vuln  
，严重级别 critical。  
  
- **漏洞编号**  
：原文未提供  
  
### 1.1.4 BusinessDataService.BatchLoad.common.kdsvc 代码执行漏洞  
- **漏洞标题**  
：金蝶云星空 BatchLoad 接口代码执行漏洞  
  
- **漏洞概述**  
：BatchLoad.common.kdsvc  
 接口存在代码执行漏洞。  
  
- **影响版本**  
：金蝶云星空  
  
- **漏洞 URL**  
：/kingdee.BOS.ServiceFacade.ServicesStub.BusinessData.BusinessDataService.BatchLoad.common.kdsvc  
  
- **漏洞编号**  
：CVD-2025-3153  
  
- **备注**  
：关联资产数约 541，全系产品可检测、可验证。  
  
### 1.1.5 kdsvc 通用反序列化漏洞  
- **漏洞标题**  
：金蝶云 K3 API common.kdsvc  
 反序列化漏洞  
  
- **漏洞概述**  
：客户端发出的数据可被攻击者恶意篡改，写入包含恶意代码的序列化数据，达到在服务端远程命令执行的效果。  
  
- **影响版本**  
：金蝶云星空、金蝶 OA 等使用 .kdsvc  
 接口的产品  
  
- **漏洞 URL**  
：/Kingdee.BOS.ServiceFacade.ServicesStub.DevReportService.GetBusinessObjectData.common.kdsvc  
  
- **漏洞编号**  
：攻击 ID 1070210290 / 1070210447  
  
### 1.1.6 管理中心反序列化 RCE（默认 8000 端口）  
- **漏洞标题**  
：金蝶云星空管理中心反序列化命令执行漏洞  
  
- **漏洞概述**  
：管理中心通信层默认采用二进制数据格式，序列化/反序列化过程未签名或校验，攻击者可篡改数据实现 RCE。管理中心默认开放于 8000 端口。  
  
- **影响版本**  
：金蝶云星空 6.x 等  
  
- **漏洞 URL**  
：管理中心默认端口 8000  
  
- **利用方式**  
：公开资料提及“反序列化 RCE + 内存马”利用方式。  
  
- **漏洞编号**  
：原文未提供  
  
## 1.2 任意文件上传  
### 1.2.1 ScpSupRegHandler 任意文件上传  
- **漏洞标题**  
：金蝶云星空私有云任意文件上传漏洞（ScpSupRegHandler）  
  
- **漏洞概述**  
：ScpSupRegHandler  
 接口未严格验证上传路径和后缀，可通过路径穿越上传恶意脚本，执行任意代码。POC/EXP 已公开流传，存在在野利用。  
  
- **影响版本**  
：V6.2（含 17 年 12 月补丁）至 V8.1（含 23 年 9 月补丁）；企业版私有云、企业版私有云（订阅）、标准版私有云（订阅）。  
  
- **漏洞 URL**  
：/k3cloud/SRM/ScpSupRegHandler  
  
- **POC/检测要点**  
：  
  
- 上传表单字段：dbId_v  
、FID  
、FAtt  
  
- 文件名示例：../../../../uploadfiles/test.ashx.  
  
- 访问路径示例：/K3Cloud/uploadfiles/Test.ashx  
  
- Snort 检测规则：  
  
text  
```
```  
  
- **漏洞编号**  
：QVD-2023-44581 / KDPSIRT-2023-00023  
  
### 1.2.2 MOBFileUpload 任意文件上传  
- **漏洞标题**  
：金蝶云星空 MOBFileUpload 任意文件上传漏洞  
  
- **漏洞概述**  
：MOBFileUpload  
 接口存在任意文件上传漏洞，已纳入 Xray 等主流扫描器检测规则。  
  
- **影响版本**  
：金蝶 K3 Cloud 财务软件系统  
  
- **漏洞 URL**  
：/K3Cloud/MOBFileUpload  
  
- **POC/检测要点**  
：  
  
http  
```
```  
  
- **漏洞编号**  
：原文未提供  
  
## 1.3 任意文件读取/下载  
### 1.3.1 CommonFileServer 任意文件读取  
- **漏洞标题**  
：金蝶云星空 CommonFileServer 接口任意文件读取漏洞  
  
- **漏洞概述**  
：CommonFileServer  
 未做权限校验和路径过滤，可读取服务器任意文件。该接口同时被 CNVD-2021-86835 收录（管理中心任意文件读取）。  
  
- **影响版本**  
：V7.X、V8.X 所有私有云和混合云版本  
  
- **漏洞 URL**  
：/K3Cloud/CommonFileServer/  
 或 /CommonFileServer/  
（管理中心）  
  
- **POC/检测要点**  
：  
  
http  
```
```  
  
http  
```
```  
  
- **漏洞编号**  
：CNVD-2021-86835  
  
### 1.3.2 FileProxyHandler.kdfile 任意文件下载  
- **漏洞标题**  
：金蝶云星空 FileProxyHandler.kdfile 接口任意文件下载漏洞  
  
- **漏洞概述**  
：FileProxyHandler.kdfile  
 存在任意文件下载漏洞，可读取配置文件、数据库凭据等。  
  
- **影响版本**  
：金蝶云星空  
  
- **漏洞 URL**  
：/K3Cloud/FileProxyHandler.kdfile  
  
- **POC/检测要点**  
：  
  
http  
```
```  
  
- **漏洞编号**  
：原文未提供  
  
## 1.4 模板引擎注入（SSTI）  
### 1.4.1 Freemarker 模板引擎注入  
- **漏洞标题**  
：Kingdee Cloud-Starry-Sky Enterprise Edition Freemarker 模板引擎注入漏洞  
  
- **漏洞概述**  
：Freemarker 引擎组件中 plugin.buildMobilePopHtml  
 函数未对模板特殊元素过滤，可利用 SSTI 执行恶意代码。官方修复版将 Freemarker 设置为 ALLOWS_NOTHING_RESOLVER  
。  
  
- **影响版本**  
：6.x / 7.x / 8.x / 9.0  
  
- **漏洞 URL**  
：/k3/o2o/bos/webapp/action/DynamicForm4Action.class  
（plugin.buildMobilePopHtml  
 函数）  
  
- **CVE 编号**  
：CVE-2025-6761  
  
## 1.5 远程命令执行（RCE）  
### 1.5.1 星空私有云 BBC 系统远程命令执行  
- **漏洞标题**  
：星空私有云 BBC 系统存在远程命令执行漏洞  
  
- **漏洞概述**  
：BBC 系统存在命令注入，攻击者可远程执行未授权命令，获取服务器敏感数据。漏洞等级为严重。  
  
- **影响版本**  
：星空私有云 BBC 系统  
  
- **漏洞 URL**  
：KDPSIRT-2025-00090（官方安全公告，具体接口未公开）  
  
- **POC**  
：官方未公开  
  
- **关联编号**  
：KDPSIRT-2025-00090 / 00091 / 00093 / 00097 / 00100；修复方案已合并，解决 KDPSIRT-2025-00100 即可。  
  
## 1.6 XML 实体注入（XXE）  
### 1.6.1 金蝶云 K3 Cloud XML 实体注入  
- **漏洞标题**  
：金蝶云 K3 Cloud 存在 XML 实体注入漏洞  
  
- **漏洞概述**  
：未对 XML 外部实体有效过滤，可利用 XXE 读取服务器敏感文件。  
  
- **影响版本**  
：金蝶云 K3Cloud v6.0  
  
- **漏洞 URL**  
：CNVD-2019-37191，具体接口未公开  
  
- **POC**  
：CNVD 附件暂不公开  
  
## 1.7 业务逻辑漏洞  
### 1.7.1 金蝶云星空管理中心业务逻辑漏洞  
- **漏洞标题**  
：金蝶云星空管理中心业务逻辑漏洞  
  
- **漏洞概述**  
：管理中心业务接口存在业务逻辑漏洞，会导致业务处理异常，无法正常访问。漏洞等级为高危。  
  
- **影响版本**  
：企业版私有云、企业版私有云（订阅）、标准版私有云（订阅）  
  
- **漏洞位置**  
：\k3\o2o\bos\webapp\action\DynamicForm4Action.class  
  
- **组件**  
：Freemarker Engine  
  
- **利用方式**  
：利用 Freemarker 模板注入语法执行任意命令  
  
- **漏洞 URL**  
：KDPSIRT-2026-00036（具体接口未公开）  
  
- **POC**  
：官方未公开  
  
# 二、金蝶 EAS / EAS Cloud  
## 2.1 反序列化/远程代码执行  
### 2.1.1 appUtil.jsp Fastjson 反序列化命令执行  
- **漏洞标题**  
：金蝶 EAS appUtil.jsp  
 存在 Fastjson 反序列化命令执行漏洞  
  
- **漏洞概述**  
：appUtil.jsp  
 使用 Fastjson 解析 list  
 参数，可构造恶意 JSON 触发反序列化，实现 RCE。  
  
- **影响版本**  
：金蝶 EAS 8.0/8.1/8.2/8.5、EAS Cloud 8.6/8.6.1/8.8  
  
- **漏洞 URL**  
：/easportal/tools/appUtil.jsp  
  
- **POC/检测要点**  
：  
  
http  
```
```  
  
解码后 JSON：  
  
json  
```
```  
  
Nuclei 检测：nuclei -t kingdee-eas-apputil-rce.yaml -l targets.txt  
  
- **漏洞编号**  
：原文未提供  
  
### 2.1.2 autoLogin.jsp Fastjson 远程代码执行  
- **漏洞标题**  
：金蝶 EAS Cloud autoLogin.jsp  
 Fastjson 远程代码执行漏洞  
  
- **漏洞概述**  
：autoLogin.jsp  
 同样使用 Fastjson 解析参数，与 appUtil.jsp  
 属于同源漏洞，可注入恶意代码控制服务器。  
  
- **影响版本**  
：金蝶 EAS 8.0/8.1/8.2/8.5、EAS Cloud 8.6 私有云/公有云/8.6.1/8.8  
  
- **漏洞 URL**  
：/easportal/tools/autoLogin.jsp  
  
- **POC/检测要点**  
：  
  
http  
```
```  
  
- **漏洞编号**  
：原文未提供  
  
### 2.1.3 金蝶 EAS 及 EAS Cloud JNDI 注入  
- **漏洞标题**  
：金蝶 EAS 及 EAS Cloud JNDI 注入漏洞  
  
- **漏洞概述**  
：EAS 通常与 Apusic 应用服务器捆绑部署，Apusic 管控台 loadTree  
 接口存在 JNDI 注入，可触发 JNDI 查询向攻击者控制的 LDAP/RMI 服务器发起请求，实现 RCE。与 Apusic loadTree  
 JNDI 注入同源。  
  
- **影响版本**  
：金蝶 EAS / EAS Cloud 多版本（需低版本 JDK）  
  
- **漏洞 URL**  
：/appmonitor/protect/jndi/loadTree  
，可通过 /admin/;//protect/jndi/loadTree  
 绕过权限  
  
- **POC/检测要点**  
：  
  
http  
```
```  
  
或：  
  
http  
```
```  
  
- **漏洞编号**  
：QVD-2023-48297  
  
### 2.1.4 金蝶 EAS AdminService 接口远程命令执行  
- **漏洞标题**  
：金蝶 EAS AdminService 接口存在远程命令执行漏洞  
  
- **漏洞概述**  
：/ormrpc/services/AdminService  
 接口存在远程命令漏洞，可通过 POST 恶意 XML 数据调用 RPC 服务执行系统命令。  
  
- **影响版本**  
：金蝶 EAS / EAS Cloud 多版本  
  
- **漏洞 URL**  
：/ormrpc/services/AdminService  
  
- **POC/利用要点（四步）**  
：  
  
- 探测：  
  
http  
```
```  
  
- 部署恶意 Axis 服务：  
  
http  
```
```  
  
关键服务名示例：joglfhlb  
，provider=java:RPC  
，className=freemarker.template.utility.Execute  
，allowedMethods=*  
。  
  
- 调用服务执行命令：  
  
http  
```
```  
  
SOAP Body 中调用 util:exec  
，传入命令。  
  
- 删除服务：  
  
http  
```
```  
  
- **漏洞编号**  
：原文未提供  
  
- **备注**  
：金蝶 Apusic 官方声明称产品本身未集成 Axis 组件，该攻击路径在产品中不成立。详见 3.7.1。  
  
## 2.2 任意文件上传  
### 2.2.1 myUploadFile 任意文件上传  
- **漏洞标题**  
：金蝶 EAS myUploadFile  
 接口任意文件上传漏洞  
  
- **漏洞概述**  
：myUploadFile.do  
 未严格校验上传文件，可上传恶意 JSP 获取 WebShell。  
  
- **影响版本**  
：金蝶 EAS / EAS Cloud  
  
- **漏洞 URL**  
：/easportal/buffalo/../cm/myUploadFile.do  
  
- **POC/检测要点**  
：  
  
http  
```
```  
  
WebShell 路径：/easportal/buffalo/../test.jsp  
  
- **漏洞编号**  
：CNVD-2023-57598  
  
### 2.2.2 uploadLogo.action 任意文件上传  
- **漏洞标题**  
：金蝶 EAS uploadLogo.action  
 任意文件上传漏洞  
  
- **漏洞概述**  
：uploadLogo.action  
 存在文件上传漏洞，可执行恶意代码、写入后门，上传 JSP 获取 WebShell。  
  
- **影响版本**  
：金蝶 EAS / EAS Cloud  
  
- **漏洞 URL**  
：/plt_portal/setting/uploadLogo.action  
 或 /easportal/tools/uploadLogo.action  
  
- **POC/检测要点**  
：  
  
http  
```
```  
  
表单字段包括：chooseLanguage_top  
、dataCenter  
、insId  
、type  
、upload  
；上传 test.jsp  
。  
  
 	WebShell 路径：/portal/res/file/upload/test.jsp  
  
- **漏洞编号**  
：DVB-2023-4442  
  
## 2.3 任意文件读取  
### 2.3.1 pdfViewLocal.jsp 任意文件读取  
- **漏洞标题**  
：金蝶 EAS Cloud pdfViewLocal.jsp  
 任意文件读取漏洞  
  
- **漏洞概述**  
：pdfViewLocal.jsp  
 未对读取路径有效校验，可读取服务器任意敏感文件。  
  
- **影响版本**  
：金蝶 EAS Cloud  
  
- **漏洞 URL**  
：/plt_document/fragments/content/pdfViewLocal.jsp  
  
- **POC/检测要点**  
：  
  
http  
```
```  
  
- **漏洞编号**  
：原文未提供  
  
## 2.4 信息泄露  
### 2.4.1 金蝶 EAS Cloud 信息泄露  
- **漏洞标题**  
：金蝶 EAS Cloud 存在信息泄露漏洞  
  
- **漏洞概述**  
：多个重要文件未设置权限，导致未授权访问，包含系统账号密码等信息，可解密后获取系统权限。  
  
- **影响版本**  
：金蝶 EAS Cloud  
  
- **漏洞 URL**  
：/easportal/tools/getenvs.jsp  
 等未授权接口  
  
- **POC/检测要点**  
：  
  
http  
```
```  
  
返回系统环境变量、账号密码等敏感信息。  
  
- **漏洞编号**  
：CNVD-2021-09698 / DVB-2021-2796  
  
## 2.5 目录遍历  
### 2.5.1 金蝶 EAS Cloud 目录遍历  
- **漏洞标题**  
：金蝶 EAS Cloud 存在目录遍历漏洞  
  
- **漏洞概述**  
：未对目录访问请求严格过滤，可遍历服务器目录获取敏感文件。  
  
- **影响版本**  
：金蝶 EAS Cloud  
  
- **漏洞 URL**  
：CNVD-2021-09699，具体接口未公开  
  
- **POC**  
：CNVD 附件暂不公开  
  
### 2.5.2 金蝶 EAS 任意文件读取（AVD）  
- **漏洞标题**  
：金蝶 EAS 存在任意文件读取漏洞  
  
- **漏洞概述**  
：金蝶 EAS 存在任意文件读取漏洞，可获取敏感信息。  
  
- **影响版本**  
：金蝶 EAS  
  
- **漏洞 URL**  
：AVD-2021-894844，具体接口未公开  
  
- **POC**  
：AVD 平台收录，具体 POC 未公开  
  
## 2.6 官方安全公告补充  
- 2024.03.05：EAS 及 EAS Cloud 安全公告更新，涉及多个安全问题修复。  
  
- 2023.11.07：接到 NVDB 反馈《金蝶 EAS Cloud 存在远程代码执行漏洞》和《金蝶 EAS Cloud 存在信息泄漏》，已分析修复。  
  
- 2023.02.22 / 2023.03.17 / 2023.05.24：EAS Cloud 系列安全公告，涉及多个安全更新。  
  
# 三、金蝶 Apusic 应用服务器（天燕中间件）  
## 3.1 JNDI 注入  
### 3.1.1 Apusic loadTree JNDI 注入  
- **漏洞标题**  
：金蝶 Apusic 应用服务器 loadTree  
 JNDI 注入漏洞  
  
- **漏洞概述**  
：权限验证不当，可向 loadTree  
 接口执行 JNDI 注入，造成 RCE。需低版本 JDK。  
  
- **影响版本**  
：金蝶 Apusic 应用服务器  
  
- **漏洞 URL**  
：/appmonitor/protect/jndi/loadTree  
，可通过 /admin/;//protect/jndi/loadTree  
 绕过权限  
  
- **POC/检测要点**  
：  
  
http  
```
```  
  
或：  
  
http  
```
```  
  
- **漏洞编号**  
：QVD-2023-48297  
  
### 3.1.2 Apusic createDataSource JNDI 注入  
- **漏洞标题**  
：金蝶 Apusic 应用服务器 createDataSource  
 JNDI 注入漏洞  
  
- **漏洞概述**  
：权限验证不当，可向 createDataSource  
 接口执行 JNDI 注入实现 RCE。需低版本 JDK 且服务器上有数据库驱动。  
  
- **影响版本**  
：金蝶 Apusic 应用服务器  
  
- **漏洞 URL**  
：/admin/;//protect/datasource/createDataSource  
  
- **POC/检测要点**  
：  
  
http  
```
```  
  
- **漏洞编号**  
：原文未提供  
  
## 3.2 任意文件上传  
### 3.2.1 Apusic deployApp 任意文件上传  
- **漏洞标题**  
：金蝶 Apusic 应用服务器 deployApp  
 任意文件上传漏洞  
  
- **漏洞概述**  
：deployApp  
 接口存在任意文件上传漏洞，可通过目录穿越写入恶意文件到 Web 目录。  
  
- **影响版本**  
：Apusic 应用服务器 V9.0 以下版本  
  
- **漏洞 URL**  
：/admin//protect/application/deployApp  
  
- **POC/检测要点**  
：  
  
http  
```
```  
  
上传包含恶意 JSP 的 ZIP 包，利用路径穿越写入：  
../../../../applications/default/public_html/shell.jsp  
  
- **漏洞编号**  
：原文未提供  
  
### 3.2.2 Apusic 应用服务器文件上传致 RCE  
- **漏洞标题**  
：金蝶 Apusic 应用服务器未授权文件上传致 RCE 漏洞  
  
- **漏洞概述**  
：深信服千里目披露，攻击者无需身份认证即可上传恶意文件，最终实现 RCE。与 deployApp  
 为同一组件的不同攻击面。  
  
- **影响版本**  
：Apusic 应用服务器（无需用户认证）  
  
- **漏洞 URL**  
：/admin/  
 管控台文件上传接口，路径绕过：/admin/;//  
  
- **POC/检测要点**  
：利用 /admin/;//  
 路径绕过权限控制，上传恶意文件至应用目录，访问后执行远程代码。  
  
- **漏洞编号**  
：原文未提供  
  
## 3.3 权限绕过  
### 3.3.1 Apusic 管控台权限绕过远程代码执行  
- **漏洞标题**  
：金蝶 Apusic 应用服务器管控台权限绕过 RCE 漏洞  
  
- **漏洞概述**  
：管控台存在路径权限控制失效，可绕过权限控制访问管控台，配合其他缺陷实现 RCE。  
  
- **影响版本**  
：V9.0 SP7 及以下版本  
  
- **漏洞 URL**  
：/admin/  
 管控台接口，路径绕过：/admin/;//  
  
- **POC/原理**  
：  
  
- security-constraint  
 仅保护 /protect  
 和 /protect/*  
。  
  
- request.getRequestURI()  
 不做 URL 解码，而 Spring 框架做路径归一化。  
  
- 因此 /admin/;//protect/xxx  
 可绕过权限校验并正常访问受保护资源。  
  
- 配合 JNDI 注入、文件上传等可实现 RCE。  
  
- **漏洞编号**  
：QVD-2023-48297  
  
## 3.4 目录遍历  
### 3.4.1 Apusic server_file 目录遍历  
- **漏洞标题**  
：金蝶 Apusic 应用服务器 server_file  
 目录遍历漏洞  
  
- **漏洞概述**  
：server_file  
 接口存在目录遍历漏洞，可获取服务器敏感信息。  
  
- **影响版本**  
：金蝶 Apusic 全版本  
  
- **漏洞 URL**  
：/appmonitor/protected/selector/server_file/files  
  
- **POC/检测要点**  
：  
  
http  
```
```  
  
- **漏洞编号**  
：原文未提供  
  
### 3.4.2 Apusic 应用服务器未授权目录遍历  
- **漏洞标题**  
：金蝶 Apusic 应用服务器存在未授权目录遍历漏洞  
  
- **漏洞概述**  
：未授权攻击者可读取内部配置文件路径及信息。  
  
- **影响版本**  
：金蝶 Apusic 应用服务器  
  
- **漏洞 URL**  
：DVB-2024-6631，具体接口未公开  
  
- **POC**  
：DVB-2024-6631 平台收录，具体接口未公开  
  
## 3.5 反序列化（IIOP 协议）  
### 3.5.1 Apusic V10.0 IIOP 反序列化 RCE  
- **漏洞标题**  
：金蝶应用服务器 V10.0 IIOP 反序列化远程代码执行漏洞  
  
- **漏洞概述**  
：IIOP 协议默认监听在 0.0.0.0  
，攻击者可通过 IIOP 协议发送恶意反序列化数据，无需授权即可执行任意代码。由 NVDB 反馈。  
  
- **影响版本**  
：Apusic 应用服务器 V10.0 企业版 SP1 至 SP8，严重级别：高  
  
- **漏洞 URL**  
：IIOP 协议端口，默认 6837  
、6838  
、6839  
  
- **POC/检测要点**  
：  
  
- 检查监听：  
  
bash  
```
```  
  
若显示 0.0.0.0:6837  
，则存在风险。  
  
- 利用：通过 IIOP 协议向目标端口发送恶意 Java 反序列化数据，可用 ysoserial 等生成，通过 CORBA/IIOP 发送。  
  
- **修复方案**  
：  
  
- 修改 ApusicAS/aas/domains/mydomain/config/domain.xml  
，将所有 iiop-listener  
 的 address="0.0.0.0"  
 改为 "127.0.0.1"  
。  
  
- 删除所有 iiop-listener  
 配置项，停用 IIOP 协议。  
  
- 升级至 V10.0 SP9（默认仅本地访问）。  
  
- **漏洞编号**  
：AVD-2025-1795705 / CNVD-2025-08294 / NVDB-CITIVD-2025865374  
  
## 3.6 源码泄露  
### 3.6.1 Apusic V10.0 企业版 JSP 源码泄露  
- **漏洞标题**  
：金蝶 Apusic 应用服务器 V10.0 企业版 JSP 源码泄露漏洞  
  
- **漏洞概述**  
：构造特殊请求访问 SpringMVC 框架应用，可下载 JSP 页面源码，泄露业务逻辑和敏感信息。  
  
- **影响版本**  
：Apusic 应用服务器 V10.0 企业版（SP1 至 SP10），漏洞等级为中  
  
- **漏洞 URL**  
：SpringMVC 框架应用路径（特殊构造请求触发 JSP 源码下载）  
  
- **触发条件**  
：部署了 SpringMVC 框架应用的 Apusic V10.0 企业版  
  
- **POC/检测要点**  
：构造特殊请求访问 SpringMVC 应用，框架会在根目录查找资源，导致 JSP 源码泄露。  
  
- **备注**  
：WEB-INF/  
 和 META-INF/  
 下的 JSP 不受影响，非 SpringMVC 应用不受影响。  
  
- **补丁地址**  
：https://file.apusic.com/fsdownload/pbLEeUl31/V10  
  
## 3.7 补充漏洞与官方声明  
### 3.7.1 关于 Axis 动态 WebService 注册漏洞的官方申明  
- **声明日期**  
：2025.12.17  
  
- **声明内容**  
：金蝶 Apusic 应用服务器产品本身未集成 Axis 组件，该漏洞描述的“通过 AdminService 方法注册危险类并在 /ormrpc/services  
 路径调用执行代码”的攻击路径在金蝶产品中不成立，产品本身不存在上述漏洞。  
  
### 3.7.2 pdfViewLocal.jsp 任意文件读取  
- **漏洞标题**  
：金蝶天燕 ApusicServer pdfViewLocal.jsp  
 任意文件读取漏洞  
  
- **漏洞概述**  
：pdfViewLocal.jsp  
 文件存在任意文件读取漏洞，攻击者可利用此漏洞读取服务器任意文件，导致敏感信息泄露。  
  
- **影响版本**  
：金蝶天燕 ApusicServer  
  
- **漏洞 URL**  
：/pdfViewLocal.jsp  
  
- **备注**  
：该漏洞在 2024 年 1 月被披露为 0day/1day 未公开漏洞。  
  
### 3.7.3 负载均衡器漏洞  
- **漏洞标题**  
：金蝶 Apusic 负载均衡器漏洞  
  
- **漏洞概述**  
：金蝶 Apusic 负载均衡器存在安全漏洞。  
  
- **影响版本**  
：ALB V2.0.1、2.0.2、2.0.5  
  
- **CVE 编号**  
：CVE-2026-42945  
  
- **修复公告日期**  
：2026.05.19  
  
### 3.7.4 ServiceFactoryService 命令执行漏洞  
- **漏洞标题**  
：金蝶天燕 /ormrpc/services/ServiceFactoryService  
 命令执行漏洞  
  
- **漏洞概述**  
：该接口存在命令执行漏洞，攻击者可利用该接口执行任意系统命令。  
  
- **影响版本**  
：金蝶天燕 ApusicServer  
  
- **漏洞 URL**  
：/ormrpc/services/ServiceFactoryService  
  
- **POC**  
：已有公开 POC  
  
- **披露时间**  
：2025-08-22  
  
### 3.7.5 金蝶天燕漏洞汇总列表  
  
根据 2025 年 8 月整理的漏洞列表，金蝶天燕相关漏洞共 23 个，主要包括：  
<table><thead><tr><th style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;background: #f5f5f5;font-weight: 700;"><section><span leaf="">漏洞名称</span></section></th><th style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;background: #f5f5f5;font-weight: 700;"><section><span leaf="">披露时间</span></section></th></tr></thead><tbody><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><code style="font-family: monospace;background: #f5f5f5;padding: .1em .3em;border-radius: 3px;font-size: .9em;"><span leaf="">/ormrpc/services/ServiceFactoryService</span></code><section><span leaf=""> 命令执行漏洞</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">2025-08-22</span></section></td></tr><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><code style="font-family: monospace;background: #f5f5f5;padding: .1em .3em;border-radius: 3px;font-size: .9em;"><span leaf="">kingdee-oa-apusic-server-file-traversal</span></code><section><span leaf=""> 目录遍历</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">2025-08-01</span></section></td></tr><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">Apusic 应用服务器 IIOP 反序列化 RCE</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">2024-11-15</span></section></td></tr><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">Apusic 监控管理平台配置文件读取漏洞</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">2024-11-15</span></section></td></tr><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">ApusicServer </span><code style="font-family: monospace;background: #f5f5f5;padding: .1em .3em;border-radius: 3px;font-size: .9em;"><span leaf="">pdfViewLocal.jsp</span></code><span leaf=""> 任意文件读取</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">2024-01-08</span></section></td></tr><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">Apusic 应用服务器 </span><code style="font-family: monospace;background: #f5f5f5;padding: .1em .3em;border-radius: 3px;font-size: .9em;"><span leaf="">deployApp</span></code><span leaf=""> 任意文件上传</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">2024-02-08</span></section></td></tr><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">Apusic 应用服务器 JNDI 注入漏洞</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">2024-01-10</span></section></td></tr></tbody></table># 四、金蝶 KIS 云  
## 4.1 KIS 云安全公告（2024.05.22）  
- **漏洞概述**  
：收到 CNVD 反馈，金蝶 KIS 云系列产品存在安全风险，KIS 云产品部已发布两个安全加固补丁。  
  
- **影响版本**  
：金蝶 KIS 云系列产品  
  
- **漏洞 URL**  
：原文未提供  
  
- **POC**  
：原文未提供  
  
- **漏洞编号**  
：原文未提供  
  
- **修复建议**  
：安装 2024 年 5 月发布的安全加固补丁。  
  
## 4.2 KIS 专业版权限提升漏洞  
- **漏洞标题**  
：Kingdee KIS Professional Edition 权限提升漏洞  
  
- **漏洞概述**  
：攻击者可通过未指定的漏洞获取计算机管理员权限。  
  
- **影响版本**  
：Kingdee KIS Professional Edition  
  
- **CVE 编号**  
：CVE-2021-37274  
  
- **危害等级**  
：High（8.8）  
  
# 五、金蝶 AI 苍穹 / 星瀚  
## 5.1 KDPSIRT-2024-00073  
- **漏洞概述**  
：金蝶 AI 苍穹 / 星瀚 / 星空 产品线存在安全漏洞，涉及 MC 管理中心。  
  
- **影响版本**  
：金蝶 AI 苍穹 MC 管理中心低于 7.0.4 版本  
  
- **漏洞 URL**  
：原文未提供  
  
- **POC**  
：原文未提供  
  
- **漏洞编号**  
：KDPSIRT-2024-00073 / KDPSIRT-2024-00070  
  
- **修复方案**  
：参考官方修复操作说明，升级苍穹端到指定版本。  
  
# 六、金蝶医疗 / 云之家（历史）  
## 6.1 金蝶医疗安全漏洞（历史）  
- **弱口令到 SQL 注入**  
：金蝶医疗后台存在弱口令（test/test），可进一步进行 SQL 注入。  
  
- **预约挂号平台支付漏洞**  
：可通过修改挂号费进行支付逻辑绕过。  
  
- **SQL 注入漏洞**  
：金蝶某系统存在 SQL 注入漏洞，已发布验证脚本。  
  
> 注：以上医疗相关漏洞多为 2013—2015 年期间披露，当前系统可能已修复。  
  
## 6.2 云之家（历史）  
- **任意用户密码重置**  
：云之家博客存在任意用户密码重置漏洞（SSV-93767，2014 年）。  
  
- **存储型 XSS**  
：云之家存在存储型 XSS 漏洞（wooyun-2014-073005）。  
  
- **隐私合规问题**  
：云之家 APP（版本 11.0.11）因隐私政策未逐一列出收集使用个人信息的目的、方式、范围等问题被通报。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/T0ibbhsCmribQJGLO3iaB1kgEciby57RxQMZnwS6Uay3iaD5JByj7tI9ZtkV4B36c2axRlich7N5v7ZwX1Yvic8yqdFxpyxZKP84hUeLYk1t08lcCA/640?wx_fmt=png&from=appmsg "")  
# 七、新增漏洞清单速览  
<table><thead><tr><th style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;background: #f5f5f5;font-weight: 700;"><section><span leaf="">产品线</span></section></th><th style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;background: #f5f5f5;font-weight: 700;"><section><span leaf="">漏洞名称</span></section></th><th style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;background: #f5f5f5;font-weight: 700;"><section><span leaf="">漏洞编号</span></section></th><th style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;background: #f5f5f5;font-weight: 700;"><section><span leaf="">危害等级</span></section></th></tr></thead><tbody><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">云星空</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">ServiceGateway.GetServiceUri 反序列化</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">攻击 ID 1070210427</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">严重</span></section></td></tr><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">云星空</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">UserService 反序列化</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">原文未提供</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">Critical</span></section></td></tr><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">云星空</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">BatchLoad.common.kdsvc 代码执行</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">CVD-2025-3153</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">高危</span></section></td></tr><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">云星空</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">kdsvc 通用反序列化 RCE</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">攻击 ID 1070210290</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">严重</span></section></td></tr><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">云星空</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">管理中心反序列化 RCE（8000 端口）</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">原文未提供</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">严重</span></section></td></tr><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">KIS 云</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">KIS 云安全风险</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">CNVD 反馈</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">待确认</span></section></td></tr><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">KIS 专业版</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">权限提升漏洞</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">CVE-2021-37274</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">High (8.8)</span></section></td></tr><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">AI 苍穹/星瀚</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">KDPSIRT-2024-00073</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">KDPSIRT-2024-00073</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">高危</span></section></td></tr><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">Apusic</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">IIOP 反序列化 RCE</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">CNVD-2025-08294</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">严重</span></section></td></tr><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">Apusic</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">pdfViewLocal.jsp 任意文件读取</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">原文未提供</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">中危</span></section></td></tr><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">Apusic</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">负载均衡器漏洞</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">CVE-2026-42945</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">待确认</span></section></td></tr><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">Apusic</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">ServiceFactoryService 命令执行</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">原文未提供</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">严重</span></section></td></tr><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">EAS</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">appUtil.jsp 文件上传/下载</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">原文未提供</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">高危</span></section></td></tr><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">EAS</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">myUploadFile JSP 上传</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">CNVD-2023-57598</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">高危</span></section></td></tr><tr><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">EAS</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">uploadLogo.action JSP 上传</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">原文未提供</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">高危</span></section></td></tr></tbody></table># 八、通用修复与缓解建议  
1. **优先升级官方补丁**  
：按各产品线公告升级至修复版本，尤其是云星空 6/7/8.x、EAS/EAS Cloud、Apusic V9/V10、KIS 云、AI 苍穹/星瀚。  
  
1. **限制暴露面**  
：禁止将 /K3Cloud/  
、/easportal/  
、/ormrpc/  
、/appmonitor/  
、/admin/  
 等管理/服务接口直接暴露至互联网。  
  
1. **反序列化防护**  
：禁用或限制 BinaryFormatter、Fastjson autoType、IIOP 反序列化；使用白名单与签名校验。  
  
1. **上传防护**  
：上传目录禁止执行脚本；校验后缀、MIME、路径穿越；文件重命名。  
  
1. **权限与路径归一化**  
：修复 /admin/;//protect/  
 类绕过；统一 URL 解码与路径归一化顺序。  
  
1. **WAF/IPS 检测**  
：针对 .kdsvc  
、ScpSupRegHandler  
、MOBFileUpload  
、appUtil.jsp  
、autoLogin.jsp  
、loadTree  
、deployApp  
、server_file  
、ServiceFactoryService  
、pdfViewLocal.jsp  
 等路径增加检测规则。  
  
1. **最小权限运行**  
：应用服务器、数据库、中间件均使用低权限账户运行，降低 RCE 后影响。  
  
1. **定期自查**  
：使用官方补丁、CNVD/CVE/NVDB 通告、Nuclei 模板等定期自查。  
  
# 九、参考来源  
<table><thead><tr><th data-colwidth="75" style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;background: #f5f5f5;font-weight: 700;"><section><span leaf="">序号</span></section></th><th style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;background: #f5f5f5;font-weight: 700;"><section><span leaf="">来源</span></section></th><th style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;background: #f5f5f5;font-weight: 700;"><section><span leaf="">说明/链接</span></section></th></tr></thead><tbody><tr><td data-colwidth="75" style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">1</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">CNVD（国家信息安全漏洞共享平台）</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">https://www.cnvd.org.cn/</span></section></td></tr><tr><td data-colwidth="75" style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">2</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">CVE / NVD（美国国家漏洞数据库）</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">NVD - Home</span></section></td></tr><tr><td data-colwidth="75" style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">3</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">阿里云 AVD 漏洞库</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">阿里云漏洞库</span></section></td></tr><tr><td data-colwidth="75" style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">4</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">长亭百川云 / CT Stack 安全社区</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">https://rivers.chaitin.cn/CT Stack 安全社区</span></section></td></tr><tr><td data-colwidth="75" style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">5</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">奇安信 CERT</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">安全内参 | 决策者的网络安全知识库</span></section></td></tr><tr><td data-colwidth="75" style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">6</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">微步在线 X 情报社区</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">https://x.threatbook.com/</span></section></td></tr><tr><td data-colwidth="75" style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">7</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">金蝶天燕官方安全公告</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">产品要闻-金蝶天燕 基础软件平台提供商</span></section></td></tr><tr><td data-colwidth="75" style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">8</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">金蝶官方 PSIRT</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">金蝶信任中心-金蝶官网</span></section></td></tr><tr><td data-colwidth="75" style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">9</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">GitHub 开源 POC 仓库</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">GitHub · Change is constant. GitHub keeps you ahead. · GitHub · GitHub</span></section></td></tr><tr><td data-colwidth="75" style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">10</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">DayDayPoc</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">DayDayPoc</span></section></td></tr><tr><td data-colwidth="75" style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">11</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">Seebug 漏洞平台（知道创宇）</span></section></td><td style="border: 1px solid #ddd;padding: .5em .75em;text-align: left;"><section><span leaf="">知道创宇 Seebug 漏洞平台 - 洞悉漏洞，让你掌握第一手漏洞情报！</span></section></td></tr></tbody></table>> 免责声明：本文档仅供授权安全测试、漏洞管理与防御加固使用。请勿用于未授权攻击。  
  
  
