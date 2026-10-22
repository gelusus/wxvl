#  PHP官方喊你立刻升级：两个9.8无认证RCE  
原创 赤霄攻防实验室
                    赤霄攻防实验室  赤霄攻防实验室   2026-10-22 01:05  
  
![封面｜漏洞情报（制图：赤霄攻防实验室）](https://mmbiz.qpic.cn/mmbiz_png/9yNnjAJL8acaFicibUjYiblDUnxBstZkeJbjfIbgoKsrC2Aouiatr16I1PKTBKgb2Jl36v39T07iaKvvo63rPdOeZYkMPHmFqgaYcydGOUjbzEyk/640?wx_fmt=png "")  
  
▲ 封面｜漏洞情报（制图：赤霄攻防实验室）  
  
TL;DR  
  
啥事  
：PHP 官网首页挂出紧急横幅，两个 9.8 分的无认证 RCE（CVE-2026-109618 / 109619），8.5.12 / 8.4.27 / 8.3.36 / 8.2.35 四个版本同时修复  
  
多严重  
：所有跑 Web 请求的 PHP 都在射程内。mod_php 最惨，一个未认证请求就能触发，发生在你应用代码跑起来之前  
  
怎么办  
：今晚升级。就这么简单，也就这么急  
  
![漏洞档案卡（制图：赤霄攻防实验室）](https://mmbiz.qpic.cn/sz_mmbiz_png/9yNnjAJL8ac7ziafDojLnJMxBARm3frc376YKol0LQlYTB4nYxTPMsH6GFNslNEQ7eutniaXLDSUjOSkhAkl3kPcQZuRJoz4xZyY8Atvmicak0/640?wx_fmt=png "")  
  
▲ 漏洞档案卡（制图：赤霄攻防实验室）  
## PHP 官网挂横幅，这很少见  
  
php.net 的首页平时安静得像文档站。10 月 21 号，它顶部挂出了一条紧急横幅：发现严重远程代码执行漏洞，请立即更新。PHP 官方用这种方式喊话的次数，一只手数得过来。上一次有这么大动静，还是多年前那批 libc 相关的连锁反应。  
  
这次是两个漏洞一起放出来：CVE-2026-109618 和 CVE-2026-109619，都是 9.8 分，都不需要认证，都能直接打到 RCE。修复版覆盖四个仍在维护的分支：8.5.12、8.4.27、8.3.36、8.2.35。官方原话是「强烈建议所有 PHP 用户立即更新」。  
  
![php.net 官方紧急公告（来源：php.net）](https://mmbiz.qpic.cn/mmbiz_png/9yNnjAJL8ad3VZdtNaiaO2Ut8M9X3nxFmTJZP0gHaQiaHS7jqlaibfDrqGnFWGX7iaec6iaQ0XJE91Vt0ibloGe3XdjSfzHLueMJNRcqUWrbRPQMA/640?wx_fmt=png "")  
  
▲ php.net 官方紧急公告（来源：php.net）  
## 第一个洞：内存耗尽时的连环坑  
  
CVE-2026-109618 走的是内存耗尽这条路。PHP 在内存不够用的时候，会用 longjmp 直接跳出当前执行上下文。问题在于，有些内部代码路径没料到这种「从天而降」的跳转：请求数据才初始化到一半，longjmp 一跳，控制流就走了，留下半成品数据在内存里。  
  
接下来请求清理流程启动，它不知道这块数据是半成品，照单全收地往下处理。内存破坏就这么发生了。研究者发现多种触发内存耗尽的策略都能稳定命中这个窗口，也就是说这不是概率性 bug，是确定性攻击面。  
## 第二个洞：worker 的记忆不该留下来  
  
CVE-2026-109619 更有意思。PHP-FPM 和 mod_php 这类常驻进程模式，一个 worker 进程会处理很多个请求。PHP 内部有一些全局状态指向请求内存，正常流程是请求结束时重置这些指针。  
  
但某些场景下，请求的内存被释放了，全局状态里的指针还留在那。同一个 worker 处理的下一个请求，在正常初始化完成之前就可能读到这块陈旧数据。上一个请求的残留，变成下一个请求的陷阱。这种跨请求的内存问题在共享进程模型里格外危险：攻击者可以利用它影响完全不相干的别人会话。  
  
![两条攻击路径（制图：赤霄攻防实验室）](https://mmbiz.qpic.cn/sz_mmbiz_png/9yNnjAJL8acjiaASphticZSchdPcMvR4Icr1YibOzLNBJQe4C2fiaIsjaEc99NgngujvX8XWnjibHwialeicAbliaAHf1Emc2E7DEhYum1dUSm62snk/640?wx_fmt=png "")  
  
▲ 两条攻击路径（制图：赤霄攻防实验室）  
## mod_php 为什么最危险  
  
官方公告里专门点名了 mod_php：已验证的利用中，一个未认证的 HTTP 请求就能触发攻击，而且发生在任何应用代码执行之前。这句话的分量要掂量清楚。  
  
你的 WAF 规则再严，它拦的是应用层输入；你的代码写得再稳，它压根还没来得及跑。攻击发生在 PHP 引擎处理请求的最早期，应用层的一切防御都不在攻击路径上。用 mod_php 的站点，升级是唯一解，迁到 PHP-FPM 是长期解。  
  
![GHSA 安全公告（来源：GitHub Advisory）](https://mmbiz.qpic.cn/mmbiz_png/9yNnjAJL8ae36bTxucGWuA53gDou4QMPUgFib7GqUjbwkocxicdxXutKMsOv8iavy5U7e7GPhTvaDhDkXmgULjXGicsrtocQyCSKKZcBDegCnVs/640?wx_fmt=png "")  
  
▲ GHSA 安全公告（来源：GitHub Advisory）  
## 今晚的清单  
  
升级到修复版本：8.5.12、8.4.27、8.3.36、8.2.35，对号入座。还在用 mod_php 的，评估迁到 PHP-FPM，进程模型的隔离性好一个量级。FPM 用户加一条 pm.max_requests 配置，让 worker 定期回收，算是给「陈旧状态」类问题上的保险。有条件的话给 OOM 和进程异常重启加告警，内存耗尽本身就是触发器。  
  
PHP 跑着全球七成多的网站，这两个洞的影响面不用我多说。升级这事，今晚就办。  
## 参考  
  
1. php.net 官方公告：https://www.php.net/index.php  
  
2. GHSA-w8hw-w9fx-j79w（OOM 损坏）/ GHSA-6wmf-4rv5-p3rp（陈旧全局状态）  
  
3. 修复版本：8.5.12 / 8.4.27 / 8.3.36 / 8.2.35  
  
免责声明：本文仅用于安全研究与防御交流。  
  
