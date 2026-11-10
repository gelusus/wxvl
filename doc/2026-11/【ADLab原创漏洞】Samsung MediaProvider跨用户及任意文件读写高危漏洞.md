#  【ADLab原创漏洞】Samsung MediaProvider跨用户及任意文件读写高危漏洞  
启明星辰
                    启明星辰  ADLab   2026-11-10 10:05  
  
更多安全资讯和分析文章请关注启明星辰ADLab微信公众号及官方网站（adlab.venustech.com.cn）  
  
  
  
**一、研究背景**  
  
  
  
在Android生态中，各大厂商为了拓展外设接入、USB OTG传输以及跨介质数据管理等功能，通常会在核心系统组件（如  
：  
MediaProvider）基础上定制特有 Provider或服务。由于多媒体管理模块天然拥有广泛的文件系统访问特权与跨用户资源交互能力，一旦其暴露的对外接口缺乏完备的输入过滤、权限校验与沙箱约束，极易引发高危的输入路径污染与越权访问问题。  
  
启明星辰ADLab  
通过对Samsung定制版MediaProvider的多用户与文件交互机制开展安全审计，重点排查公开暴露且持有系统级特权的ContentProvider，发现其OTG处理逻辑存在严重的任意文件读写及跨用户资源窃取漏洞。  
  
  
**二、Samsung MediaProvider机制与安全上下文**  
  
  
  
2.1 运行上下文与权限特权  
  
受影响组件属于Samsung系统级多媒体管理模块 com.samsung.android.providers.media，该应用在系统底层具备极高的特权配置：  
- 进程UID：与android.media共享UID（android:sharedUserId="android.media"），运行于系统级媒体进程上下文。  
  
- 持有权限：该进程声明并持有READ_EXTERNAL_STORAGE、WRITE_EXTERNAL_STORAGE、MANAGE_EXTERNAL_STORAGE以及跨用户交互权限（如：INTERACT_ACROSS_USERS/INTERACT_ACROSS_USERS_FULL）。  
  
- 资源能力：作为系统多媒体的核心托管者，该进程具有跨用户空间遍历多媒体库、读取公有及受保护外置存储路径的底层能力。  
  
2.2 漏洞触发模型  
  
通常情况下，Android规范要求ContentProvider实现严格的沙箱限定与URI权限校验。当满足以下攻击模型条件时，将直接穿透隔离机制：  
- 系统特权组件在AndroidManifest.xml中将关键数据接口配置为exported=true，且未声明读写权限保护（缺失readPermission/writePermission）；最新版本补丁已经增加了权限限制。  
  
- 写入通道（如  
：  
insert）允许直接由外部未受信数据填充物理路径字段（如  
：  
_data），且未实施路径白名单校验或规范化清洗  
。  
  
- 消费通道（如  
：  
openFile/openFileHelper）无条件信任持久化存储中取回的路径，并以特权身份执行底层open操作。  
  
#   
  
**三、漏洞原理（Samsung CaseID：I-121929）**  
#   
  
3.1 核心组件与配置缺陷  
  
漏洞存在于com.samsung.android.providers.media模块内的OtgProvider组件。  
  
（1）  
组件配置  
  
该Provider设置了android:exported="true"，且完全未配置任何访问权限门槛。设备上任意零权限的三方应用均可通过ContentResolver发起跨进程调用。  
  
（  
2）  
核心缺陷接口  
- OtgProvider.insertInternal()：对外暴露数据插入入口。  
  
- OtgDao_Impl.insert()：将传入的数据对象持久化至SQLite数据库。  
  
- OtgProvider.openFileCommon()：根据URI ID取出记录并直接调用文件句柄。  
  
3.2 漏洞成因分析  
  
（  
1）  
边界校验缺失  
  
OtgFileItem.Companion.fromValues()在解析ContentValues中的_data键时，仅仅验证了字符串非空，未对路径格式进行正则匹配、目录规范化（Canonicalize）处理。  
  
（2）  
持久化盲目信任  
  
OtgProvider.openFileCommon()获取到对应ID的记录后，误认为数据库中的数据为系统内部安全可信数据，直接将pathById传入new File()并通过ParcelFileDescriptor.open()返回给发起调用的三方应用。  
  
（  
3）  
特权放大与跨用户危害  
  
由于MediaProvider拥有android.media高权限与存储管理特权，攻击者不仅可以通过指定目标应用的私有目录破坏应用沙箱，还可以指定包含其他用户标识（如  
：  
/data/user/10/...或主用户共享存储路径）的路径，实现跨用户空间任意敏感图片、媒体数据、文档配置的越权窃取与任意覆盖。  
  
  
**四、漏洞处置与修复建议**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/d1YpVG2tPVqE8mA6rQTd0OI0GgibJJf1VbrlJOhUust8VNmp567laD5piafqTFbLrO8et6vEYEsEkBEZy6lgBdSFk6qEBTz7mYCjhScQoDahg/640?wx_fmt=png "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/d1YpVG2tPVqHH4toaFjn5eDcK4znibqudDvbphibNFt60qyQ5rO3N3J2Rsgua0ickx334MicLbNFMSTdmXNpiaycBpQYCDe4kXS3B9Jpk9LxdjSk/640?wx_fmt=png "")  
  
Samsung官方已发布S9210CHC6DZH3最新版本修复该漏洞，建议相关用户立即升级至S9210CHC6DZH3或更高版本。  
  
针对该类由于Provider暴露和路径校验缺失引发的高危漏洞，建议从以下几个维度进行加固：  
  
（  
1）  
权限最小化与导出控制  
  
若OtgProvider仅供系统内部或特定系统组件协同工作，应将android:exported改为false。  
  
若必须对外导出，应配置签名级保护权限（android:protectionLevel="signature"），禁止普通三方应用访问该Provider。  
  
（  
2）  
跨用户安全边界对齐  
  
对涉及文件操作的请求，严格校验调用方UID与当前操作的用户空间 userId，防范利用系统特权中转跨越多用户隔离边界。  
  
  
**五、小 结**  
  
  
此类由于公开Provider充当“特权代理”而引发的任意文件读写与越权漏洞，是Android框架定制中常见的系统级安全隐患。由于缺陷存在于系统级核心进程内部，一旦触发往往具备高危害性与高稳定性。厂商在定制拓展外设传输及媒体管理能力时，需严格遵循权限最小化原则，对所有来源于外部的数据输入施加严格沙箱限制，避免系统特权被外部普通应用借道利用。  
  
  
**六、漏洞披露时间线**  
  
- 2026  
年  
6  
月  
27  
日，ADLab向Samsung Mobile提交漏洞报告。  
  
- 2026  
年  
6  
月  
29  
日，  
ADLab补充漏洞细节。  
  
- 2026  
年  
7  
月  
2  
日，  
Samsung官方判定漏洞存在。  
  
- 2026  
年  
11  
月  
9  
日，ADLab复测Samsung官方发布的S9210CHC6DZH3新版本,确认漏洞修复。  
  
启明星辰积极防御实验室（ADLab）  
  
  
  
  
  
ADLab成立于1999年，是中国安全行业最早成立的攻防技术研究实验室之一，微软MAPP计划核心成员，  
“黑雀攻击”概  
念首推者。截至目前，ADLab已通过 CNVD/CNNVD/NVDB/CVE累计发布安全漏洞7000余个，持续保持国际网络安全领域一流水准。实验室研究方向涵盖基础安全研究、电信  
运营商基础设施安全研究  
、  
移动终端安全研究、云安全研究、  
信创安全研究、  
物联网安全研究、  
车联网安全研究、工控安全研究、  
数据安全研究、5G安全研究、  
AI安全研究、卫星安全研究、  
低空安全研究  
、高级威胁研究、攻防体系建设。研究成果应用于产品核心技术研究、国家重点科技项目攻关、专业安全服务等。  
  
  
  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/VuRGkncX57ONOtW3DSPMEXiaLPqrs8a20KxsFg78IaJzyEf51AIjLGNkDG5tsCH76Qo7PoVz74JGQqKJbCh5PdQ/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=2 "")  
  
