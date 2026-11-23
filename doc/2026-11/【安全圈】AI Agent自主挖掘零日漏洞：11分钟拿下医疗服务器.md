#  【安全圈】AI Agent自主挖掘零日漏洞：11分钟拿下医疗服务器  
安全圈
                    安全圈  安全圈   2026-11-23 11:00  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/aBHpjnrGylgOvEXHviaXu1fO2nLov9bZ055v7s8F6w1DD1I0bx2h3zaOx0Mibd5CngBwwj2nTeEbupw7xpBsx27Q/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "")  
  
**关键词**  
  
  
  
AI，漏洞  
  
  
**核心事实**  
：亚马逊威胁情报团队（Amazon Threat Intelligence）与 AWS 安全专家披露全球首起自主 AI 智能体在野全自动挖掘 0-day 的真实攻防战。攻击者仅给出一句“获取目标 Shell”的自然语言提示词，AI 智能体便在 **11 分钟**  
 内自主探测开源医疗影像平台 MONAI Label、逆向挖掘出未公开的路径穿越漏洞 **CVE-2026-133968**  
（CVSS 9.8 严重），并成功拿到某大学科研服务器的底层 Root 最高控制权。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/sbq02iadgfyE0e0afXnhbr8DicOpOkBS2fDGu3t8mw88SBokHpiarH9gRQKmzEy4HKg1KribibA4OdAJc8sR5xJlTOJ1pZ2WPfs1YXiaQhLSU7JRI/640?wx_fmt=other&from=appmsg "")  
  
本起事件标志着自动化网络攻击技术发生质变：攻击者不再依靠人工寻找薄弱点，而是由 AI Agent 自主规划攻击链条。调查人员在公网捕获到了失陷的渗透测试框架面板，完整揭示了 AI 从端口摸排、漏洞挖掘到权限维持的全套自动化战法。  
## 🔍 漏洞底层机理：医疗影像服务未清洗路径直通任意写  
  
MONAI Label 是广泛应用于医疗 AI 科研领域、基于 PyTorch 的智能影像标注服务器：  
- **无过滤路径拼接**  
：漏洞存在于服务端存储客户端上传图像与标注标签的 API 接口中。后端代码将客户端提交的文件标识符直接与存储目录进行字符串拼接写入磁盘，完全未做规范化校验与穿越字符过滤。  
  
- **默认免认证敞开大门**  
：该平台默认配置下无需任何身份认证即可调用核心接口。任何能够通过网络触达该服务的攻击者，均可触发该接口向服务器任意路径写入任意文件。  
  
- **受影响范围**  
：MONAI Label 0.8.5 及所有历史旧版本均受波及。官方在源码层合并了防御补丁，但截至目前 PyPI 官方包管理器仍未发布可直接安装的更新包。  
  
## 💥 AI Agent 攻击全过程复盘：11分钟打穿防御  
  
亚马逊调查团队发现攻击者部署的开源多智能体渗透框架 **Cairn**  
 控制台暴露在互联网上，完整还原了 AI Agent 的攻击步骤：  
- **0-11 分钟（自主挖洞）**  
：Agent 接入目标服务器端口后，主动调用 OpenAPI 文档，逐一构造测试用例探测接口传参，仅用 11 分钟便自主发现了可穿越目录的文件写入 0-day 漏洞。  
  
- **后门植入与驻留**  
：为绕过普通的 Web 限制，Agent 极其精准地选定了 Python 解释器在启动时自动静默加载的配置文件，利用任意写漏洞将恶意代码注入宿主环境。  
  
- **合法触发反弹 Root**  
：Agent 接着调用 MONAI Label 的模型训练 API，服务端在启动 Python 训练子进程时立刻加载了恶意脚本，以系统 Root 权限向外部攻击者服务器建立反向连接。  
  
- **完全自主决策**  
：幕后操纵者全程仅下达了高层意图指令，漏洞挖掘策略、路径穿越利用及提权姿势全部由 AI 独立决策闭环。  
  
## 📦 溯源研判与受害特征  
  
亚马逊安全报告指出，根据暴露控制台中的操作习惯与元数据判断：  
- **背景线索**  
：操作者提示词包含简体中文，运行环境配置为中国标准时间（UTC+8），攻击目标呈现高度针对性，中等置信度推测攻击者具中文背景，但尚未归类至已知国家级黑客组织。  
  
- **受害单位**  
：本次沦陷的受害者为某大学医学院的科研服务器，内部存有大量患者未脱敏病理切片及训练模型。  
  
## ⚡ 应急处置：科研与医疗 AI 平台防御指南  
  
鉴于官方 PyPI 尚未释出编译版，所有部署了 MONAI Label 及同类 AI 标注平台的机构必须立即自查加固：  
  
**措施一：阻断外网访问与下线公开端口**  
  
立即将 MONAI Label 服务（通常占用 8000 等端口）收敛至内网专用安全隔离区，严禁映射到公网，必须经由安全 VPN 或堡垒机访问。  
  
**措施二：强制开启 API 鉴权机制**  
  
在服务器启动参数中显式配置认证中间件，禁止匿名未授权访问接口。  
  
**措施三：严格降权与容器只读挂载**  
  
禁止使用 Root 身份直接在宿主机裸跑 AI 平台。必须使用独立低权限系统账号，并采用只读根文件系统（--read-only  
）容器化部署，仅为标注目录开辟临时挂载卷。  
  
**措施四：检查系统 Python 目录完整性**  
  
排查 Python 环境库目录下的 sitecustomize.py  
、usercustomize.py  
 及 .pth  
 文件，核对最近修改时间，清除未经授权的外部写入内容。  
  
  
   END    
  
  
阅读推荐  
  
  
[【安全圈】单月1130起勒索，3.11亿条数据被挂牌](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652080017&idx=1&sn=70d92ae03f620a7ace3b8888d687816a&scene=21#wechat_redirect)  
  
  
   
  
[【安全圈】UAC被曝蠢到离谱的绕过，微软不分配CVE](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652080017&idx=2&sn=c4ef9c02feb49f867d9b5e707a06405a&scene=21#wechat_redirect)  
  
  
   
  
[【安全圈】AI工具打进欧盟委员会，HR系统停摆10天](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652080017&idx=3&sn=e011a548515da4f906b2a03b32765abd&scene=21#wechat_redirect)  
  
  
   
  
[【安全圈】Linux KVM曝高危逃逸漏洞：虚机跨租户直读宿主机127GB内存](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079967&idx=1&sn=38dfa6a339d62a690035a68d6f9dad4c&scene=21#wechat_redirect)  
  
  
   
  
[【安全圈】黑客部署自主AI智能体：11分钟自挖0day攻陷服务器夺Root](https://mp.weixin.qq.com/s?__biz=MzIzMzE4NDU1OQ==&mid=2652079967&idx=2&sn=85167f3d14ddd041423b52b5fed14503&scene=21#wechat_redirect)  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCEft6M27yliapIdNjlcdMaZ4UR4XxnQprGlCg8NH2Hz5Oib5aPIOiaqUicDQ/640?wx_fmt=gif "")  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCEDQIyPYpjfp0XDaaKjeaU6YdFae1iagIvFmFb4djeiahnUy2jBnxkMbaw/640?wx_fmt=png "")  
  
**安全圈**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCEft6M27yliapIdNjlcdMaZ4UR4XxnQprGlCg8NH2Hz5Oib5aPIOiaqUicDQ/640?wx_fmt=gif "")  
  
  
←扫码关注我们  
  
**网罗圈内热点 专注网络安全**  
  
**实时资讯一手掌握！**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCE3vpzhuku5s1qibibQjHnY68iciaIGB4zYw1Zbl05GQ3H4hadeLdBpQ9wEA/640?wx_fmt=gif "")  
  
**好看你就分享 有用就点个赞**  
  
**支持「****安全圈」就点个三连吧！**  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/aBHpjnrGylgeVsVlL5y1RPJfUdozNyCE3vpzhuku5s1qibibQjHnY68iciaIGB4zYw1Zbl05GQ3H4hadeLdBpQ9wEA/640?wx_fmt=gif "")  
  
  
