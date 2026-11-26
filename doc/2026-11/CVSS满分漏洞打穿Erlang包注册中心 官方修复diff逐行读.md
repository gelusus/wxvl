#  CVSS满分漏洞打穿Erlang包注册中心 官方修复diff逐行读  
原创 紫禁
                    紫禁  紫禁玄科   2026-11-26 06:18  
  
SECURITY · INTELLIGENCE  
# CVSS满分漏洞让包注册中心被打穿  
  
2026年11月26日 · 10 min · #供应链安全 #Elixir #反序列化  
  
周一深夜，六台服务器同时被人在操作系统层面执行了命令。它们不是某家公司的业务机，而是 **hex.pm**  
——整个 Elixir/Erlang 生态取依赖的包注册中心。用掉的漏洞评分满分 10.0，代码里躺了 **十年零九个月**  
。  
  
昨天（11-25）维护团队发布了完整事故复盘，GitHub 上两个修复 commit 全程公开。我花了一晚上把复盘、diff、NVD 记录全部核对了一遍，这篇文章讲三件事：**这个漏洞的利用链为什么设计得如此"优雅又恶毒"；一支十人左右的志愿团队如何在 15 分钟内全网止血、又如何在取证上做足功夫；以及同一周 GreyNoise 披露的 GT13 行动里，攻击者用 AI 流水线打防火墙管理中心的另一组数字。**  
  
![图1 · Hex.pm 沦陷与自救时间线](https://mmbiz.qpic.cn/sz_mmbiz_png/Ah5NAr7MJFiaHdYV6yvnMQlEsZ0oraXkhTxvTH5O4wE205EKhvYhrz08aE6jc6Yx5VQffqErlYnsXbMOdsibO3atE0worTzlUia2M6CLu9fBgI/640?from=appmsg "")  
  
▲ 图1 · Hex.pm 沦陷与自救时间线  
## 一、先给结论：这次事件是防御范本，不是防御事故  
  
很多读者看到"包注册中心沦陷 CVSS 10.0"会条件反射想到 SolarWinds。但读完整个时间线我的判断相反——**Hex.pm 团队的响应是我近两年见过的小团队应急处置教科书**  
。先把关键节点摆出来（北京时间，括号内为原文 UTC）：  
<table><tbody><tr><th style="border:1px solid #d7dde6;padding:8px 10px;line-height:1.6;color:#252a34;background:#eef2ff;font-weight:700;"><section><span leaf="">时间</span></section></th><th style="border:1px solid #d7dde6;padding:8px 10px;line-height:1.6;color:#252a34;background:#eef2ff;font-weight:700;"><section><span leaf="">事件</span></section></th></tr><tr><td style="border:1px solid #d7dde6;padding:8px 10px;line-height:1.6;color:#252a34;"><section><span leaf="">11-24 02:16–02:29</span></section></td><td style="border:1px solid #d7dde6;padding:8px 10px;line-height:1.6;color:#252a34;"><section><span leaf="">负载均衡日志共 69 次利用请求，全部来自两个 IP</span></section></td></tr><tr><td style="border:1px solid #d7dde6;padding:8px 10px;line-height:1.6;color:#252a34;"><section><span leaf="">攻击期间</span></section></td><td style="border:1px solid #d7dde6;padding:8px 10px;line-height:1.6;color:#252a34;"><section><span leaf="">恶意文件最快在请求后 </span><strong><span leaf="">3 毫秒</span></strong><span leaf=""> 落地；3 台服务器因滚动重启回到带病版本（补丁没盖住发布流程）</span></section></td></tr><tr><td style="border:1px solid #d7dde6;padding:8px 10px;line-height:1.6;color:#252a34;"><section><span leaf="">11-24 02:31</span></section></td><td style="border:1px solid #d7dde6;padding:8px 10px;line-height:1.6;color:#252a34;"><strong><span leaf="">全部生产服务器跑上修复构建</span></strong><section><span leaf="">——距首次被利用约 15 分钟</span></section></td></tr><tr><td style="border:1px solid #d7dde6;padding:8px 10px;line-height:1.6;color:#252a34;"><section><span leaf="">11-24 03:07</span></section></td><td style="border:1px solid #d7dde6;padding:8px 10px;line-height:1.6;color:#252a34;"><section><span leaf="">一份独立描述同型漏洞的安全报告才通过邮件送达，此时攻击已结束</span></section></td></tr><tr><td style="border:1px solid #d7dde6;padding:8px 10px;line-height:1.6;color:#252a34;"><section><span leaf="">事后一周内</span></section></td><td style="border:1px solid #d7dde6;padding:8px 10px;line-height:1.6;color:#252a34;"><section><span leaf="">替换所有可能被读取的凭据（除 registry 签名密钥）、向瑞典数据保护局/CISA/ENISA 通报、逐个比对数据库快照证明包未被篡改</span></section></td></tr></tbody></table>  
两个细节值得放大看。第一，"3 台机器重启后回归旧版本"说明他们当晚就发现了自己止血动作里的漏洞并如实写进报告——这种自我揭短在厂商通告里罕见。第二，侦察流量回溯显示攻击者很可能先在本地复刻了一份 hex.pm 做靶场试打。**防御方连对手的工程习惯都替你标好了注脚。**  
  
![图2 · CVE-2026-129514 五步利用链](https://mmbiz.qpic.cn/mmbiz_png/Ah5NAr7MJFia371icLVBKVxQ1DYVk77WTUVic4CuwG7ib0Bybg6cq37gVkhoHD3ica4iceia46JE9VWVia5ktc0DO0zjViakdyjKQ7ibjkQafCKSeBCMY/640?from=appmsg "")  
  
▲ 图2 · CVE-2026-129514 五步利用链  
## 二、利用链拆解：一个键名如何走到命令执行  
  
漏洞编号 CVE-2026-129514，NVD 实查确认收录于 11-24，CVSS v4.0 满分 10.0。它的成因链条，我用官方修复 commit 的代码注释来还原（下面这段就是 lib/hexpm_web/erlang_format.ex  
 修复时的原文注释，不是我转述的）：  
```
# A decoded map with a :__struct__ key is dispatched to that struct's protocol # implementations, so inspecting or enumerating the params runs code chosen # by the client.
```  
  
**第一步：免认证入口。**  
 hex.pm 支持一种 Erlang external term format 编码的请求体，解析发生在路由和鉴权*之前*——未登录的 POST 打到任意路径都会触碰解码器。  
  
**第二步：:safe 不够 safe。**  
 解码用了 Plug.Crypto.non_executable_binary_to_term(binary, [:safe])  
。这个选项确实拦住了函数项、进程端口这类真正危险的项——但它对 map 一视同仁地放行。而 Elixir 的结构体在运行时**就是一个含 :__struct__ 键的普通 map**  
。于是攻击者的请求体可以伪装成应用已加载的任何结构体——比如 URI  
，比如某个协议实现特别活跃的模块。  
  
**第三步：伪结构体本身不执行任何东西。**  
 这是链条最精巧的一环：构造出来的 %{__struct__: SomeModule, ...}  
 静静躺在参数里什么也不做。触发点是**错误处理**  
——带着非法字段值的伪结构体引发报错，而 Phoenix/Ecto 的错误报告代码出于好意要去 inspect  
 这个结构体内容。  
  
**第四步：报错即武器。**inspect  
 走协议派发（protocol dispatch），Ecto/DBConnection 相关模块的实现会对结构体做枚举、转换——**每一步派发的目标代码由 __struct__ 指向的模块决定，而这个值是客户端给的**  
。攻击者借此调用选定函数，最终以 registry 服务账号权限执行操作系统命令。  
  
一句话总结这条链的设计哲学：**它不注入代码，它劫持类型系统**  
。Erlang/Elixir 社区争论了很多年的"term 反序列化到底安不安全"，答案原来取决于你信不信保留键名。  
## 三、逐行读修复 diff：十八行代码关掉十年的门  
  
修复 commit 379b0a29b  
（"Refuse structs in Erlang term request bodies"）的核心是一个递归检查，我把它原文贴出来：  
```
def decode(binary) do   term = Plug.Crypto.non_executable_binary_to_term(binary, [:safe])    if struct?(term) do     {:error, "bad binary_to_term"}   else     {:ok, term}   end rescue   ArgumentError ->     {:error, "bad binary_to_term"} end  # 新增：递归拒绝任何嵌套层含 __struct__ 的 term defp struct?(map) when is_map(map) do   Map.has_key?(map, :__struct__) or     Enum.any?(map, fn {key, value} -> struct?(key) or struct?(value) end) end  defp struct?([head | tail]), do: struct?(head) or struct?(tail) defp struct?(tuple) when is_tuple(tuple), do: struct?(Tuple.to_list(tuple)) defp struct?(_other), do: false
```  
  
注意三个工程细节：① 检查覆盖 map 的**键和值两侧**  
（测试用例里有 %{struct => "value"}  
 这种把结构体放键位的变体）；② 列表用 cons 遍历、元组先转 list，四种容器形态全兜住；③ 默认分支明确返回 false，不留隐式行为。  
  
第二个 commit 2d0be98a1  
 更有意思：团队修完请求体入口后**主动排查了同型风险面**  
——发布 tarball 的 metadata 也走 term 解码，于是把 struct?  
 提升为公共函数 Hexpm.Utils.contains_struct?/1  
，在 release controller 加了 check_metadata_structs  
，并把前一个 commit 里的私有实现删掉统一复用。测试用例直接构造 %{"requirements", struct}  
、%{"links", %{"GitHub" => struct}}  
 等四种注入位置逐一断言 422 拒绝。  
  
**从"补一个洞"到"扫一遍同类洞"再到"重构防漂移"，这三步在一个 commit 序列里全可见。**  
 如果你团队在做安全代码评审，这两个 diff（加起来不过几十行）值得当教材打印。  
  
还有个历史彩蛋：报告的溯源说那条拦住 __struct__  
 的安全检查早在 **2016 年 2 月 23 日的一次代码清理中被删除**  
——CVE 的年龄就是这么来的。十年间无数次重构没人记得那行代码为什么存在。这比任何"依赖扫描"都扎心：**有些防线删掉时不会报警。**  
## 四、用户侧清单：现在该做什么  
  
按官方建议整理（未建号纯拉包开发者：什么都不用做，只需近期只从 repo.hex.pm 取包、别用第三方镜像——registry 签名密钥尚未轮换）：  
  
1. 改 hex.pm 密码（bcrypt 哈希存储，但弱口令仍可被慢速爆破）  
  
1. **重新设置 2FA**  
：新建 authenticator 条目、保存新恢复码（Totp secret 可能被读过）  
  
1. 客户端重新登录：mix hex.user auth  
（Erlang 项目用 rebar3 hex user auth  
）  
  
1. 等下一个 Hex client 版本上线（带来新 registry 签名体系）  
  
1. 警惕钓鱼：邮箱可能暴露，凡"关于 Hex 的邮件"先核对链接是否真指向 hex.pm  
  
1. API key 不用换——128 位随机值仅以 HMAC-SHA256 哈希存储，官方判断无法现实爆破  
  
  
  
自建 registry 的组织注意：漏洞在服务端代码不在客户端，**不打补丁你的实例同样裸奔**  
。  
  
同周还有三个 Elixir 库各自披露了"不可信数据构造结构体"家族的新洞（ash / db_connection / flop，各有独立 CVE 与利用前提，均非针对 hex.pm 的攻击），官方表格齐全，升级即可——这里能看出 __struct__  
 问题正在成为生态级议题。  
  
![图3 · GT13 行动画像](https://mmbiz.qpic.cn/mmbiz_png/Ah5NAr7MJFggIFiaHl55kEIbBDITRNsPiadcOayATFLulanEo47DVM8rAQZhZ0fEQA9bluAotjvzQLNtdQMS1iabaotoy14iafGVAOcuqI8ppibg/640?from=appmsg "")  
  
▲ 图3 · GT13 行动画像  
## 五、同一周的另一个故事：GT13，AI 驱动的防火墙管理中心猎杀  
  
GreyNoise 昨天披露的行动代号 GT13，画像和 Hex.pm 事件形成诡异互补——那边是"传统手法+一流响应"，这边是"**AI 流水线+至今查不到初始入口**  
"：  
  
- **窗口**  
：10-17 至 11-02，99 个 IP 全是 ExpressVPN 一次性出口或被盗用的 Check Point 设备自身  
  
- **战果**  
：24 台 Check Point Security Management 服务器沦陷，波及 22 家机构、7 个行业、横跨 18 国（金融/能源/政府/交通/矿业/工程/制造）  
  
- **规模证据**  
：1400+ 网络会话、800+ 命令行痕迹、1200+ 文件操作；工具链 Nmap/Chisel/Impacket/Paramiko 加 Python 脚本——GreyNoise 判断其扫描、利用、对被控资产交互均由 agentic harness 自动完成  
  
- **跳板艺术**  
：先用 Rejetto HFS 的 CVE-2026-61500（NVD 实查 9.8 分）自动化拿下一个小服务器，往 root 的 SSH authorized_keys 文件  
 塞公钥持久化，然后把它当中转站回捞 24 台 CP 设备的数据；还尝试部署 Linux implant ZULT（未成功）  
  
- **深挖**  
：对其中一家网络，用已修补漏洞 CVE-2026-16232 的脚本发起 100+ 管理任务，偷防火墙数据库和内部 CA 文件、铸造新证书  
  
  
  
最冷的一句出自报告原文：**"GT13 在拿下 HFS 之前就已控制那些 Check Point 管理服务器——我们查不到最初是怎么进去的。"**  
 我们把 ARTEX/MadPot、Cairn/MONAI、再到今天 GT13 三篇连起来看，趋势已经很清楚：攻击方的 agent 化不再是实验室演示，而是跨供应商、跨国界、以周为单位稳定产出的作战方式；而且它们专挑"管理面"下手——包注册中心、防火墙管理平台，谁控制更新通道谁就控制下游一切。  
## 六、两条主线合起来的启示  
  
Hex.pm 证明了小团队靠纪律可以打赢遭遇战：15 分钟止血、快照比对自证清白、公开 diff 接受全世界审计。GT13 则提醒我们另一半场：当入侵检测引擎变成 AI agent，"查不到初始入口"会成为常态——**假设初访问必然发生，把防御重心移到"控制面权限最小化 + 出站异常 + 更新通道完整性"上**  
。  
  
至于那个 __struct__  
 键名，我想在所有做过 RPC 框架、消息中间件、工作流引擎的人耳边重复一遍官方注释那句话：**inspecting or enumerating the params runs code chosen by the client.**  
 你的序列化协议里有没有这样一个保留键？今晚就去 grep。  
  
*信源（检索日期 2026-11-25 ~ 26）：hexpm 官方事故复盘《The November 23 security incident》及 GitHub commit 379b0a29b / 2d0be98a1 原始 diff（本文代码段为直接摘录）；NVD CVE-2026-129514（CVSS v4.0 10.0，2026-11-24 收录，本机 API 实查）、CVE-2026-61500、CVE-2026-16232；GreyNoise GT13 行动报告；Ash/DBConnection/Flop 各自安全公告；Elixir Forum 维护者澄清帖。文中时间均已换算北京时间。*  
  
