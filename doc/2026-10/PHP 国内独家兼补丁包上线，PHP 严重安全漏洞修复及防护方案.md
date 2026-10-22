#  PHP 国内独家兼补丁包上线，PHP 严重安全漏洞修复及防护方案  
 利刃信安   2026-10-22 11:30  
  
尊敬的宝塔面板用户：  
  
针对 PHP **CVE-2026-109618、CVE-2026-109619**  
 两项严重安全漏洞，宝塔已同步相关 PHP 安全修复版本，并提供首页风险一键加固功能。用户可结合 Nginx 防火墙、系统加固、防入侵和 PHP 网站安全告警，完成漏洞处置及后续防护。  
  
付费安全服务  
  
**宝塔提供付费安全服务，如需专业人员协助完成 PHP 漏洞处置、安全排查及加固，请扫描二维码联系客服**  
  
**宝塔是国内首家针对本次漏洞手动适配旧版 PHP 安全补丁的运维面板服务商，并提供独家修复包与面板内更新方案。**  
 目前已为 Linux 的 PHP 7.0、7.1、7.2、7.3、7.4、8.0、8.1 提供安全修复包，帮助仍依赖旧版 PHP 的用户完成本次漏洞修复。其中，PHP 7.0 至 7.3 的修复版本及安装命令已公布，请先在测试环境验证后再升级。  
  
本次漏洞可能导致未经身份验证的远程代码执行。请受影响用户优先安装 PHP 安全补丁；暂时无法升级的用户，请先执行一键加固，并开启相关防护。具体修复方案及操作步骤如下。  
  
以下安全中心与插件操作以宝塔 Linux 面板为例，Windows 用户请按照下方说明选择 PHP 修复版本。**安装 PHP 安全补丁无需先升级面板。**  
  
补丁升级仍是必要步骤  
  
首页风险中的一键加固，以及 Nginx 防火墙、系统加固、防入侵、PHP 安全告警等防护措施，都不能完整替代 PHP 安全补丁。开启防护后，仍需完成 PHP 升级。  
## 一、安装 PHP 安全补丁  
  
安装 PHP 安全补丁无需先升级宝塔面板。Linux 的 PHP 7.4 及以上版本可通过软件商店升级，PHP 7.0 至 7.3 请使用下方对应的安装命令。请先检查网站实际使用的 PHP 版本，再完成对应的安全更新。  
### PHP 安全修复版本  
  
本次漏洞已提供的修复版本如下：  
  
PHP 官方修复版 · Linux / Windows  
<table><thead><tr style="box-sizing: border-box;"><th style="box-sizing: border-box;padding: 9px 12px;background-color: rgb(234, 245, 237);color: rgb(23, 103, 47);font-weight: 700;text-align: left;border: 1px solid rgb(220, 232, 223);font-size: 14px;"><section><span leaf="">PHP 分支</span></section></th><th style="box-sizing: border-box;padding: 9px 12px;background-color: rgb(234, 245, 237);color: rgb(23, 103, 47);font-weight: 700;text-align: left;border: 1px solid rgb(220, 232, 223);font-size: 14px;"><section><span leaf="">安全修复版本</span></section></th></tr></thead><tbody><tr style="box-sizing: border-box;"><td style="box-sizing: border-box;padding: 9px 12px;border: 1px solid rgb(220, 232, 223);text-align: left;font-size: 14px;color: rgb(52, 68, 58);"><section><span leaf="">PHP 8.5</span></section></td><td style="box-sizing: border-box;padding: 9px 12px;border: 1px solid rgb(220, 232, 223);text-align: left;font-size: 14px;color: rgb(52, 68, 58);"><strong style="box-sizing: border-box;font-weight: 700;color: rgb(24, 60, 37);"><span leaf="">8.5.12</span></strong></td></tr><tr style="box-sizing: border-box;"><td style="box-sizing: border-box;padding: 9px 12px;border: 1px solid rgb(220, 232, 223);text-align: left;font-size: 14px;color: rgb(52, 68, 58);"><section><span leaf="">PHP 8.4</span></section></td><td style="box-sizing: border-box;padding: 9px 12px;border: 1px solid rgb(220, 232, 223);text-align: left;font-size: 14px;color: rgb(52, 68, 58);"><strong style="box-sizing: border-box;font-weight: 700;color: rgb(24, 60, 37);"><span leaf="">8.4.27</span></strong></td></tr><tr style="box-sizing: border-box;"><td style="box-sizing: border-box;padding: 9px 12px;border: 1px solid rgb(220, 232, 223);text-align: left;font-size: 14px;color: rgb(52, 68, 58);"><section><span leaf="">PHP 8.3</span></section></td><td style="box-sizing: border-box;padding: 9px 12px;border: 1px solid rgb(220, 232, 223);text-align: left;font-size: 14px;color: rgb(52, 68, 58);"><strong style="box-sizing: border-box;font-weight: 700;color: rgb(24, 60, 37);"><span leaf="">8.3.36</span></strong></td></tr><tr style="box-sizing: border-box;"><td style="box-sizing: border-box;padding: 9px 12px;border: 1px solid rgb(220, 232, 223);text-align: left;font-size: 14px;color: rgb(52, 68, 58);"><section><span leaf="">PHP 8.2</span></section></td><td style="box-sizing: border-box;padding: 9px 12px;border: 1px solid rgb(220, 232, 223);text-align: left;font-size: 14px;color: rgb(52, 68, 58);"><strong style="box-sizing: border-box;font-weight: 700;color: rgb(24, 60, 37);"><span leaf="">8.2.35</span></strong></td></tr></tbody></table>  
宝塔 Linux 修复版 · 软件商店升级  
<table><thead><tr style="box-sizing: border-box;"><th style="box-sizing: border-box;padding: 9px 12px;background-color: rgb(234, 245, 237);color: rgb(23, 103, 47);font-weight: 700;text-align: left;border: 1px solid rgb(220, 232, 223);font-size: 14px;"><section><span leaf="">PHP 分支</span></section></th><th style="box-sizing: border-box;padding: 9px 12px;background-color: rgb(234, 245, 237);color: rgb(23, 103, 47);font-weight: 700;text-align: left;border: 1px solid rgb(220, 232, 223);font-size: 14px;"><section><span leaf="">安全修复版本</span></section></th></tr></thead><tbody><tr style="box-sizing: border-box;"><td style="box-sizing: border-box;padding: 9px 12px;border: 1px solid rgb(220, 232, 223);text-align: left;font-size: 14px;color: rgb(52, 68, 58);"><section><span leaf="">PHP 8.1</span></section></td><td style="box-sizing: border-box;padding: 9px 12px;border: 1px solid rgb(220, 232, 223);text-align: left;font-size: 14px;color: rgb(52, 68, 58);"><strong style="box-sizing: border-box;font-weight: 700;color: rgb(24, 60, 37);"><span leaf="">8.1.35</span></strong></td></tr><tr style="box-sizing: border-box;"><td style="box-sizing: border-box;padding: 9px 12px;border: 1px solid rgb(220, 232, 223);text-align: left;font-size: 14px;color: rgb(52, 68, 58);"><section><span leaf="">PHP 8.0</span></section></td><td style="box-sizing: border-box;padding: 9px 12px;border: 1px solid rgb(220, 232, 223);text-align: left;font-size: 14px;color: rgb(52, 68, 58);"><strong style="box-sizing: border-box;font-weight: 700;color: rgb(24, 60, 37);"><span leaf="">8.0.31</span></strong></td></tr><tr style="box-sizing: border-box;"><td style="box-sizing: border-box;padding: 9px 12px;border: 1px solid rgb(220, 232, 223);text-align: left;font-size: 14px;color: rgb(52, 68, 58);"><section><span leaf="">PHP 7.4</span></section></td><td style="box-sizing: border-box;padding: 9px 12px;border: 1px solid rgb(220, 232, 223);text-align: left;font-size: 14px;color: rgb(52, 68, 58);"><strong style="box-sizing: border-box;font-weight: 700;color: rgb(24, 60, 37);"><span leaf="">7.4.34</span></strong></td></tr></tbody></table>  
宝塔 Linux 修复版 · PHP 7.0 至 7.3  
<table><thead><tr style="box-sizing: border-box;"><th style="box-sizing: border-box;padding: 9px 12px;background-color: rgb(234, 245, 237);color: rgb(23, 103, 47);font-weight: 700;text-align: left;border: 1px solid rgb(220, 232, 223);font-size: 14px;"><section><span leaf="">PHP 分支</span></section></th><th style="box-sizing: border-box;padding: 9px 12px;background-color: rgb(234, 245, 237);color: rgb(23, 103, 47);font-weight: 700;text-align: left;border: 1px solid rgb(220, 232, 223);font-size: 14px;"><section><span leaf="">安全修复版本</span></section></th></tr></thead><tbody><tr style="box-sizing: border-box;"><td style="box-sizing: border-box;padding: 9px 12px;border: 1px solid rgb(220, 232, 223);text-align: left;font-size: 14px;color: rgb(52, 68, 58);"><section><span leaf="">PHP 7.3</span></section></td><td style="box-sizing: border-box;padding: 9px 12px;border: 1px solid rgb(220, 232, 223);text-align: left;font-size: 14px;color: rgb(52, 68, 58);"><strong style="box-sizing: border-box;font-weight: 700;color: rgb(24, 60, 37);"><span leaf="">7.3.34</span></strong></td></tr><tr style="box-sizing: border-box;"><td style="box-sizing: border-box;padding: 9px 12px;border: 1px solid rgb(220, 232, 223);text-align: left;font-size: 14px;color: rgb(52, 68, 58);"><section><span leaf="">PHP 7.2</span></section></td><td style="box-sizing: border-box;padding: 9px 12px;border: 1px solid rgb(220, 232, 223);text-align: left;font-size: 14px;color: rgb(52, 68, 58);"><strong style="box-sizing: border-box;font-weight: 700;color: rgb(24, 60, 37);"><span leaf="">7.2.35</span></strong></td></tr><tr style="box-sizing: border-box;"><td style="box-sizing: border-box;padding: 9px 12px;border: 1px solid rgb(220, 232, 223);text-align: left;font-size: 14px;color: rgb(52, 68, 58);"><section><span leaf="">PHP 7.1</span></section></td><td style="box-sizing: border-box;padding: 9px 12px;border: 1px solid rgb(220, 232, 223);text-align: left;font-size: 14px;color: rgb(52, 68, 58);"><strong style="box-sizing: border-box;font-weight: 700;color: rgb(24, 60, 37);"><span leaf="">7.1.34</span></strong></td></tr><tr style="box-sizing: border-box;"><td style="box-sizing: border-box;padding: 9px 12px;border: 1px solid rgb(220, 232, 223);text-align: left;font-size: 14px;color: rgb(52, 68, 58);"><section><span leaf="">PHP 7.0</span></section></td><td style="box-sizing: border-box;padding: 9px 12px;border: 1px solid rgb(220, 232, 223);text-align: left;font-size: 14px;color: rgb(52, 68, 58);"><strong style="box-sizing: border-box;font-weight: 700;color: rgb(24, 60, 37);"><span leaf="">7.0.34</span></strong></td></tr></tbody></table>  
PHP 7.0 至 7.3 为宝塔自建补丁，建议先测试。对应安装命令见文末附录。  
  
针对已停止 PHP 官方安全维护的 **PHP 7.4、8.0、8.1**  
，宝塔已应用 PHP 官方提供的回补补丁，完成 Linux 修复包打包并提供面板内升级，便于仍依赖旧版 PHP 的用户及时处置本次漏洞。**Windows 的 PHP 8.1 及更早版本当前需修改配置、禁用相关功能进行临时缓解**  
，并尽快规划迁移至受支持的 PHP 分支及对应安全修复版本。  
  
上述旧版 PHP 均已停止 PHP 官方安全维护，仍建议尽快规划迁移至受支持的版本。  
### 通过软件商店升级 PHP 7.4 及以上版本  
  
升级前，备份网站、数据库及相关 PHP 配置，并选择合适的业务维护时间。如果已开启系统加固，且页面提示加固规则阻止升级，请按提示临时关闭相关防护，完成后恢复。  
  
以下步骤与截图以 Linux 面板为例：  
1. 进入「软件商店」，选择 已安装  
。  
  
1. 先单击右上角的 更新软件列表 / 支付状态  
，获取最新软件信息。  
  
1. 找到网站正在使用的 PHP 分支，单击 更新  
，按提示安装对应修复版本。  
  
![在软件商店更新软件列表，再单击 PHP 对应的更新按钮](https://mmecoa.qpic.cn/mmecoa_png/gf6D0zv7SKRTU0hfC4XA55gGyHdqdCHH2YvqFvU5fuEOIkr9A7TmvTibuYA2QbP9ZEDvoNf4FRbAS56rSGT7vzSwaBkGqe8r5jcgtiaVXTgib0/640?wx_fmt=png&from=appmsg "")  
  
在软件商店更新软件列表，再单击 PHP 对应的更新按钮  
  
1. 更新完成后，确认对应 PHP 服务已重启；如更新过程没有重启服务，请手动重启，使所有工作进程使用修复后的版本。  
  
1. 检查网站实际使用的 PHP 版本，并验证页面访问、登录、文件上传、数据库连接及其他核心业务功能。  
  
服务器安装了多个 PHP 分支时，请逐一核对仍在使用的版本。只更新其中一个 PHP 分支，不能覆盖其他分支运行的网站。Windows 用户请通过软件商店升级 PHP 8.2 及以上版本，并在升级后核对服务和网站的实际版本。  
### PHP 7.0 至 7.3 宝塔修复版已提供  
  
**PHP 7.0 至 7.3 已停止官方安全维护，官方不再为这些分支提供安全补丁。宝塔已以 PHP 7.4 的官方补丁为基础，逐行比对并回移适配，提供 7.0.34、7.1.34、7.2.35、7.3.34 四个 Linux 修复版本。**  
  
安装命令及升级注意事项见文末附录，请先备份并在测试环境验证。  
## 二、通过安全中心执行一键加固  
  
对于暂时无法完成 PHP 升级的用户，宝塔 Linux 企业版面板 **13.1 及以上版本**  
 支持通过「安全中心 → 首页风险」执行一键加固，操作方式如下：  
1. 确认已开通企业版授权，且 Linux 面板版本为 **13.1 或更新的稳定版本**  
。版本较低时，在首页右上角单击 更新  
，按提示完成面板更新，刷新后确认版本。  
  
1. 进入「安全中心 → 安全概览」，找到「首页风险」并单击 查看  
，打开「安全风险」窗口。  
  
1. 单击 重新扫描  
，找到涉及 **CVE-2026-109618、CVE-2026-109619**  
 的 PHP 风险项，查看受影响的 PHP 版本和处理建议。  
  
1. 单击该风险项右侧的 修复  
；也可只勾选该 PHP 风险项，使用 一键修复  
，避免同时修改尚未核对的其他风险项。  
  
1. 如果界面提示先关闭系统加固，请按提示临时关闭相关防护；修复完成后恢复原有防护状态。  
  
1. 在「结果确认」中核对处理的 PHP 版本、配置内容及修复结果。  
  
![在安全中心打开首页风险，重新扫描 PHP 漏洞并查看加固结果](https://mmecoa.qpic.cn/sz_mmecoa_png/gf6D0zv7SKQazKpCRFjF5MnMkSUfEVwO5pwUS1HXjJgILMsjP1QzlzdqaJ3kwTJVH3YSIm6n5D5SXyLhiaITNkBwhBGCVSV7xW9jtbIuVvj8/640?wx_fmt=png&from=appmsg "")  
  
在安全中心打开首页风险，重新扫描 PHP 漏洞并查看加固结果  
  
**修复结果显示“修复成功”后，请务必重启结果中列出的每个 PHP 服务，确保配置生效。**  
 随后重新扫描，并检查网站的上传、登录等功能是否正常。修复失败时，请根据结果提示检查对应 php.ini  
，按风险详情中的方案手动处理。  
  
一键加固只能降低部分攻击路径的风险，不能完整修复本次两项漏洞。扫描状态和修复结果也不能代替对 PHP 实际补丁版本的核对。  
  
PS：安全中心一键加固为企业版专享功能，需使用 13.1 及以上版本的宝塔 Linux 企业版面板。如需购买，请**扫描文末二维码联系客服**  
## 三、开启 Nginx 防火墙，拦截 Web 攻击  
  
**宝塔 Nginx 防火墙（WAF）9.9.0 版本已新增 CVE-2026-109618 和 CVE-2026-109619 漏洞攻击拦截功能。**  
 对于恶意超长参数等命中防护规则的请求，WAF 可在请求进入 PHP 处理前进行拦截，降低漏洞利用风险。  
  
使用 Nginx 的服务器，建议将 Nginx 防火墙更新至 **9.9.0 或更新版本**  
，启用 WAF 并检查站点防护状态，配合 PHP 安全补丁、系统加固及防入侵等功能共同防护。  
1. 进入「软件商店」，搜索并打开 Nginx防火墙  
。  
  
1. 在首页开启 防火墙开关  
，并在「站点配置」中核对需要保护的网站及其规则状态。  
  
1. 根据业务检查「全局配置」和站点规则，确认正常的登录、上传和接口请求可以访问。  
  
1. 查看「封锁历史」，核对被拦截的站点、来源 IP、时间和原因；如需调整规则，请先确认业务请求及命中的具体规则。  
  
    ![](https://mmecoa.qpic.cn/sz_mmecoa_png/gf6D0zv7SKQZQqoibjwQeJWicSVvmziafNyO2ZYtTXpnCMw42aneJfOqKXTUGW7Wz7iasXXkShCRoeDvVLoOjIDEsBsxD7UcLMICxXGoMRia9Qjs/640?wx_fmt=png&from=appmsg "")  
  
Nginx 防火墙的总开关、站点配置及封锁历史入口  
  
下图为相关超长参数请求的 WAF 拦截示例：该 POST 请求命中 参数过长  
 过滤规则，系统已执行拦截。用户可在「攻击详情」中核对过滤规则、请求地址和 HTTP 信息，确认具体拦截原因。  
  
![](https://mmecoa.qpic.cn/mmecoa_png/gf6D0zv7SKRnLWTEo0DfCNY9oOVzwwH66cSA378pREsb69NUDJA5aEZePgu5eg0JsYHyQsbF7e6c1nII5Uicn69q0s7EyxdQnttQaCHnerEg/640?wx_fmt=png&from=appmsg "")  
  
Nginx WAF 攻击详情显示 POST 请求因参数过长被拦截  
  
WAF 用于拦截匹配规则的攻击请求，仍需配合 PHP 安全补丁。  
  
PS：Nginx 防火墙为专业版、企业版功能。如需购买，请**扫描文末二维码联系客服**  
## 四、开启系统加固，保护关键配置与文件  
  
建议启用宝塔系统加固，保护服务器关键目录、账号、服务和计划任务，降低攻击者修改系统配置、植入持久化任务或破坏服务的风险。  
1. 进入「软件商店」，搜索 系统加固  
，安装或打开「宝塔系统加固」。  
  
1. 在「防护配置」中开启顶部的 宝塔系统加固  
 总开关。  
  
1. 根据业务情况配置并开启需要的加固项，优先检查关键目录、计划任务、用户和服务的保护范围。  
  
1. 查看「日志审计」「操作日志」，确认是否有被拦截的操作，并核对正常业务是否受到影响。  
  
![宝塔系统加固的总开关、加固项目及配置入口](https://mmecoa.qpic.cn/mmecoa_png/gf6D0zv7SKS8q020Y2xMaeYPpugibCBTsxXMY9xjjIWUxW8psdDFIju9abUpWcExzM7PE6kuSqbN2mZibrXPCALaUiacKkiaIiasD3AOibdNfCG3Q/640?wx_fmt=png&from=appmsg "")  
  
宝塔系统加固的总开关、加固项目及配置入口  
  
建议重点检查：  
- **关键目录加固：保护系统关键文件，减少被修改或替换的风险。**  
- **计划任务加固：限制计划任务被添加、删除或篡改，降低恶意任务驻留的风险。**  
- **用户、环境变量与服务加固：限制账号、运行环境和服务配置被异常修改。**  
- **管理进程白名单：结合实际业务进程配置，避免正常服务被误结束。**  
配置完成后，确认总开关及所需加固项均已开启。加固规则可能影响软件安装、PHP 更新、账号变更和计划任务调整；维护时按需使用 临时开关  
，完成后及时恢复防护。  
  
PS：系统加固为专业版、企业版功能。如需购买，请**扫描文末二维码联系客服**  
## 五、开启防入侵，拦截恶意提权操作  
  
PHP 漏洞被利用后，攻击者可能获得网站运行账号的权限，并进一步尝试提权。建议启用堡塔防入侵，拦截恶意提权操作，降低攻击者获取 root  
 权限的风险。  
1. 进入「软件商店」，搜索并打开 堡塔防入侵  
。  
  
1. 在「概览」中开启顶部的 防入侵开关  
。  
  
1. 核对 PHP 网站实际运行的账号，重点检查 www  
 用户对应的「防入侵」状态是否为 已开启  
。  
  
1. 开启所需的命令日志记录，通过 命令日志  
、操作日志等入口查看异常命令与拦截记录。  
  
![堡塔防入侵的总开关、www 用户防护状态及命令日志入口](https://mmecoa.qpic.cn/mmecoa_png/gf6D0zv7SKR3UUibbrF4PXiaj1akhNQPQqn8jBYic6NmxCz1DFWeufWPEu3ZkqFtrtaBYX0RlY24nc20V1Micxdd1kceiaCndIDCZxN9XzPzgDBI/640?wx_fmt=png&from=appmsg "")  
  
堡塔防入侵的总开关、www 用户防护状态及命令日志入口  
  
**总开关与用户防护状态都需要检查。**  
 仅开启命令日志用于记录操作，不能代替开启防入侵。使用自定义网站运行账号时，应核对插件对该账号的支持和实际防护状态。  
  
防入侵用于限制和拦截攻击中的提权行为，仍需配合 PHP 补丁与系统安全更新。发现异常提权记录时，应继续检查相关网站、文件和进程，处理攻击入口。  
  
PS：堡塔防入侵为企业版功能。如需购买，请**扫描文末二维码联系客服**  
## 六、开启 PHP 网站安全告警  
  
建议启用 PHP 网站安全告警，及时发现可疑的 PHP 执行行为，并对匹配规则的行为进行告警或拦截，协助管理员处理一句话木马、文件包含等风险。  
1. 进入「安全中心 → PHP网站安全」，打开「PHP网站安全告警」。  
  
1. 在首页开启 监控开关  
。  
  
1. 进入「网站管理」，核对需要保护的网站；在「PHP模块」中检查对应 PHP 版本的模块状态，按界面提示完成配置。  
  
1. 进入「告警设置」，配置界面支持的通知方式和接收信息，确保管理员能及时收到告警。  
  
1. 返回首页，查看风险列表中的网站、来源 IP、触发时间和检测类型，逐项核查异常。  
  
![PHP 网站安全告警的监控开关、风险记录、PHP 模块及告警设置入口](https://mmecoa.qpic.cn/sz_mmecoa_png/gf6D0zv7SKTd28gSoibxqfRFM01qvHjWNleqcCeHv3ShUT8n3HAzJ8e2hs2PBSp4xibn2WXEbcDtUoib20iaVdRMPev473z1ic7AIB5TibQricKgvA/640?wx_fmt=png&from=appmsg "")  
  
PHP 网站安全告警的监控开关、风险记录、PHP 模块及告警设置入口  
  
处理告警时，先核对涉及的站点、文件路径与请求是否属于正常业务，再决定隔离文件、调整规则或添加白名单。白名单应限制在确认需要放行的范围内。  
  
**没有产生告警，不代表 PHP 漏洞已经修复。**  
 补丁版本、模块状态和告警记录需要分别核对。  
  
PS：PHP 网站安全告警为企业版功能。如需购买，请**扫描文末二维码联系客服**  
## 七、安全服务与客服协助  
  
如需开通专业版、企业版或相关功能授权，可扫描下方二维码联系客服。  
  
付费安全服务  
  
**宝塔同时提供付费安全服务**  
。如您不便自行操作，或需要专业人员协助进行 PHP 漏洞处置、安全排查及加固，请扫描下方二维码联系客服，咨询具体服务方案。  
  
![扫码联系宝塔客服](https://mmecoa.qpic.cn/mmecoa_png/gf6D0zv7SKTlS4rW2qfo5XfJ56zh9Vp7oNCnwZZK9pJeEQlquJgwHqRicvicJRyxSHytPQ4uNSpjD9f4ppav9gIDzDCvdjiaSvicjdmYoJ1a71o/640?wx_fmt=png&from=appmsg "")  
  
扫描二维码，联系宝塔客服  
## 八、升级及防护配置后的检查  
  
请在完成上述操作后，逐项核对以下内容：  
- 所有已有修复包且仍在使用的 PHP 分支均已升级至对应安全修复版本。  
  
- 如使用首页风险一键加固，已开通企业版授权，Linux 面板版本为 **13.1 或更新的稳定版本**  
，且已重启对应 PHP 服务，网站核心功能正常。  
  
- Nginx 环境中的防火墙总开关及需要保护的站点规则已开启，正常业务请求可以访问。  
  
- 系统加固总开关及所需规则、防入侵总开关及网站运行账号的防护均已开启。  
  
- PHP 网站安全告警的监控和对应模块正常，告警接收信息已配置。  
  
- 查看 Web 访问日志、PHP 错误日志、异常进程及提权记录，确认没有需要继续处理的异常。  
  
对于暂时没有可用补丁的旧版本，应保留临时缓解与防护措施，并尽快完成版本迁移。  
  
宝塔将持续跟进 PHP 安全更新及相关攻击风险，及时同步修复版本与防护建议。请各位用户尽快完成版本检查、补丁升级及防护配置，保障网站与业务正常运行。  
## 附录：PHP 7.0 至 7.3 修复版安装命令  
  
执行前请先备份并测试  
- 执行前，请先备份站点、数据库及相关 PHP 配置。  
  
- 升级会重启对应 PHP 服务，站点会短暂不可用，建议在业务低峰期操作。  
  
- 以上版本使用宝塔自建补丁，请先在测试环境验证业务正常，再用于生产环境。  
  
  
  
请在宝塔 Linux 面板的「终端」中，根据网站当前使用的 PHP 分支选择一条对应命令执行。安装这些补丁无需先升级面板。  
  
**PHP 7.0：升级至 7.0.34 宝塔修复版**  
```
curl -fL -o php7_cve.sh https://download.bt.cn/install/php7_cve.sh && bash php7_cve.sh install 70
```  
  
**PHP 7.1：升级至 7.1.34 宝塔修复版**  
```
curl -fL -o php7_cve.sh https://download.bt.cn/install/php7_cve.sh && bash php7_cve.sh install 71
```  
  
**PHP 7.2：升级至 7.2.35 宝塔修复版**  
```
curl -fL -o php7_cve.sh https://download.bt.cn/install/php7_cve.sh && bash php7_cve.sh install 72
```  
  
**PHP 7.3：升级至 7.3.34 宝塔修复版**  
```
curl -fL -o php7_cve.sh https://download.bt.cn/install/php7_cve.sh && bash php7_cve.sh install 73
```  
  
完成后，请核对对应 PHP 服务及网站实际使用的版本，并验证页面访问、登录、文件上传、数据库连接等核心功能。若同时运行多个旧版 PHP 分支，请分别处理。  
  
宝塔面板  
  
2026 年 10 月 22 日  
  
