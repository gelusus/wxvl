#  速修 | Chrome 156 修复 117 个漏洞，13 个 Critical  
红队安全圈
                    红队安全圈  红队安全圈   2026-10-21 12:00  
  
ChromeOS 推送 156.0.8078.42，一次性带入 117 个浏览器安全修复，13 个 Critical，内核侧还有能从 Guest 打穿宿主机的高危组件，Chromebook 用户今天就该更。  
  
## 01  更新概况  
  
这次是 ChromeOS 稳定通道的里程碑跨级更新：系统版本 16836.19.0，自带浏览器版本 156.0.8078.42。因为跨了一个里程碑，浏览器安全修复是整批打包进来的，数量远超常规周更——Critical 13 个、High 87 个、Medium 15 个、Low 2 个，合计 117 个。  
  
## 02  漏洞速览  
  
更新主体  
ChromeOS 16836.19.0  
  
浏览器版本  
156.0.8078.42  
  
修复规模  
117 个 CVE  
  
等级分布  
13 Critical / 87 High  
  
主要类型  
Use after free 为主  
  
重灾区  
V8 / Media / PDF / WebRTC  
  
在野利用  
公告未提及  
  
## 03  关键 CVE 与成因  
  
浏览器侧最值得留意的几个 Critical：  
  
CVE-2026-106347  UAF in Track  
  
CVE-2026-106358  UAF in Navigation  
  
CVE-2026-110149  UAF in Browser  
  
CVE-2026-110922  UAF in Prerender  
  
CVE-2026-113303  MemCorrupt in WebRTC  
  
成因上这一批高度同质：UAF 占了绝对大头，集中在 Media、PDF、V8、WebRTC 这些历史重灾区。Media 一个组件就贡献了近十个 High，说明该子系统的对象生命周期管理仍是最容易出问题的地方。另有少量逻辑类问题值得单独点名：CVE-2026-106329 是 FileSystem 的 Incorrect authorization，CVE-2026-106346 是 DevTools 的 Improper state validation，CVE-2026-110159 是 Updater 的 Incorrect authorization——这类不靠内存破坏的洞在利用链里往往更好拼。  
  
ChromeOS 专属部分反而更刺激：  
  
mali UAF: kbase_vmap_phy_pages  
  
非特权可触发, mt8196 平台  
  
mali_kbase: dma_buf 双重释放  
  
错误路径, 内核内存破坏  
  
arc-keymaster: Guest-to-host  
  
fall-off-end UB, wild call  
  
前两个是非特权权限即可触发的内核内存破坏，第三个出在 ARC 容器组件上，是虚拟化逃逸性质的 Guest-to-host 问题——对装了 Android 容器（ARC）的 Chromebook 来说，这组修复比浏览器批次更关键。另外还有一个 Medium 级修复别漏掉：第三方 CA 证书失陷会导致所有客户端的 TLS 受影响，虽然成因在证书侧不是代码侧，但影响是全局性的。  
  
## 04  利用分析  
  
公告没有给出任何利用详情，也没有标注在野利用，悬赏金额大多还是 TBD，属于典型的批量防御性更新。但按成因可以做推演：  
  
浏览器 UAF/类型混淆类：通常需要先诱导用户访问恶意页面触发渲染进程漏洞，再配合沙箱逃逸才能落地，单个洞危害有限，但 13 个 Critical 里 Browser 进程和 Navigation 级别的（如 CVE-2026-110149、CVE-2026-106358）本身就是沙箱内高价值目标。  
  
mali 内核 UAF/双重释放：非特权即可触发，配合浏览器 RCE 可以拼完整链，是这批修复里攻防价值最高的。arc-keymaster 的 Guest-to-host 问题在企业 managed Chromebook 场景里尤其值得注意——Guest 会话是默认可用的入口。  
  
## 05  检测与修复  
  
地址栏打开下面这个页面查当前版本，浏览器版本低于 156.0.8078.42 即受影响；ChromeOS 系统版本低于 16836.19.0 同样需要更新：  
  
chrome://version  
  
更新路径：Settings - About ChromeOS - Check for updates，更新后重启生效。桌面版 Chrome 用户走 Help - About Chrome 拿同批次浏览器修复即可。  
  
## 06  红队视角  
  
这批洞单独拿出来每个都不算新鲜，但两点值得记：一是 M 系大版本跨级更新是抓 0day 窗口的好时机——修复量上百时，内部回归测试覆盖必然稀释，历史上这类批次出过补丁的补丁；二是 ChromeOS 的攻击面正在从浏览器转向内核与虚拟化组件（mali 驱动、arc-keymaster），做企业 Chromebook 攻防时优先盯 Guest 会话可达的内核接口。等 TBD 悬赏金额陆续公布，金额高的那几个大概率就是质量最高的漏洞，可以拿来反推研究优先级。  
  
  
下期继续蹲厂商补丁和 PoC 动静，**关注**  
后漏洞先知道，修复不踩坑。  
  
— END —  
  
  

  
  
