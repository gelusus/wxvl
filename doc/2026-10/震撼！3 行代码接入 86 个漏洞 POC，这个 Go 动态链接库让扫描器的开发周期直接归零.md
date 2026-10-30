#  震撼！3 行代码接入 86 个漏洞 POC，这个 Go 动态链接库让扫描器的开发周期直接归零  
棉花糖糖糖
                    棉花糖糖糖  棉花糖网络安全工具箱   2026-10-30 02:51  
  
免责声明：本文仅用于安全研究与防御性检测方向的技术讨论，文中所涉 POC 均指向验证性探测与理论判定，不包含漏洞利用流程，请勿对未授权目标使用。
Kunpeng 是一个用 Go 编写的漏洞 POC 检测框架，以动态链接库的形式对外提供调用。调用方传入一段描述目标资产与检测目标的 JSON，库内部完成插件调度、请求发送、响应匹配与结果序列化，返回结构化的漏洞结论。项目当前收录 86 个检测插件，覆盖数据库、中间件、Web 组件、CMS 与网络服务等资产类型，运行于 Windows、Linux、Darwin。
重点导读定位

漏洞库与调用方解耦：业务系统只保留调度逻辑，POC 集合独立维护
跨语言调用：C ABI 统一入口，Python、Java、Node.js、PHP、Go 均可加载
单文件分发：编译产物为一个动态链接库，升级即覆盖
非利用化设计：只做验证与理论判定，不进入漏洞利用流程

重点导读架构

插件注册表：GoPlugins 与 JSONPlugins 双映射，按 target 聚合
任务调度：plugin.Scan，输入任务结构，输出结果切片
目标匹配：CVE 精确匹配、KPID 精确匹配、target 模糊匹配、全量匹配
请求层：util.RequestDo 封装 HTTP 发送，util.TCPSend 封装裸协议探测
配置层：config.Config，全局单例，JSON 读写
日志层：util.Logger，info、warning、error 三级，支持跨语言缓冲取回
辅助验证：util.AiderCheck，标识串回调判定
接口层：web/api.go，基于 gin 暴露 HTTP 路由

重点导读插件
PART 01形态
Go 插件在编译期通过 plugin.Regist 注册，实现 Init、Check、GetResult 三个方法，适合弱口令、协议握手、RCE 指纹这类需要自定义连接逻辑的检测。JSON 插件由 target、meta、request、verify 四段结构描述，verify 支持 string、regex、md5 三种响应匹配方式，编译时由 esc 打包进二进制，运行时支持 extra_plugin_path 目录轮询加载，轮询周期 20 秒，目录内文件删除后对应插件自动下线。
PART 02覆盖
共 51 个 Go 插件与 35 个 JSON 插件，编号从 KP-0001 累加至 KP-0086，关联 26 个 CVE 编号。Go 插件的类型分布为 RCE 18 个、弱口令 16 个、未授权访问 3 个、SQL 注入 3 个、信息泄漏 3 个、SSRF 2 个、任意文件读取 2 个，其余为 XXE、URL 跳转、任意文件上传与配置类问题。JSON 插件的目标覆盖 discuz、wordpress、joomla、drupal、django、thinkphp、weblogic、docker、elasticsearch、hadoop、phpmyadmin、coremail、glassfish 等组件。等级分布为严重 30 个、高危 13 个、中危 4 个、低危 3 个、提示 1 个。
PART 03调度

target 命中 cve- 前缀：按插件声明的 CVE 编号精确匹配
target 命中 kp- 前缀：按插件声明的 KPID 精确匹配
其余情况：按插件注册名做模糊包含匹配
target 置为 all：触发全量插件执行
任务类型区分：service 类型只执行 Go 协议类插件，web 类型继续执行 JSON 插件
任务格式校验：web 任务要求目标以 http 开头，service 任务要求不带 http 协议头，并内置站点过滤规则
异常隔离：单插件 panic 由 recover 捕获并记录，不影响其余插件的调度结果

重点导读接口
PART 04C ABI
main.go 通过导出注释对外暴露全部入口：Check 执行检测任务，GetPlugins 返回插件清单，SetConfig 写入配置，ShowLog 开启日志输出，GetVersion 返回版本号，StartWebServer 启动 HTTP 服务，StartBuffer 与 GetLog 用于跨语言取回执行日志。出入参统一为 JSON 字符串。
PART 05HTTP

GET /api/pluginList：返回按 KPID 排序后的插件元数据
POST /api/check：提交检测任务，返回漏洞结果数组
POST /api/config：运行时热更新配置

PART 06任务结构
任务字段包括 type、netloc、target 三项，以及 meta 扩展项。meta 承载 system 操作系统标识、pathlist 目录路径列表、filelist 文件路径列表、passlist 自定义弱口令字典，为无状态接口提供上下文。
重点导读配置

timeout：连接与请求超时，缺省为 15 秒
http_proxy：HTTP 代理，作用于统一请求函数发出的流量
pass_list：弱口令字典，未配置时使用内置字典
aider：辅助验证服务地址，用于无法通过回显判断的场景
extra_plugin_path：外部 JSON 插件目录，周期性热加载

弱口令字典内置 {user}、{user}123、admin、123456 与空口令五项，字典中的 {user} 占位符会在各插件内替换为目标服务的常见用户名。日志方面，ShowLog 控制标准输出，StartBuffer 开启内存缓冲，GetLog 按分隔符取回日志并自动重置缓冲，缓冲总量上限 4096 字节。
重点导读构建

依赖管理使用 Go Modules
JSON 插件打包：esc -include='.json$' -o plugin/json/JSONPlugin.go -pkg jsonplugin plugin/json/
全语言版本：go build -buildmode=c-shared 产出 kunpeng_c.so
Go 专用版本：go build -buildmode=plugin 产出 kunpeng_go.so，仅在 Linux 可用
版本号通过链接参数 -X main.VERSION 注入，调用方以 GetVersion 获取

重点导读场景

扫描器漏洞库：端口探测与指纹识别之后，由 Kunpeng 承接漏洞层
资产安全监控：存活探测、指纹匹配、漏洞验证三段式流水线中的验证环节
渗透测试辅助：作为交互式控制台的漏洞验证模块
Web 化控制台：仓库内 poc-scanner 示例以 Tornado 面板加 so 代理实现
跨语言集成：仓库提供 C、Python、Java、JavaScript、Lua、Go 六种调用样例，以及基于 nmap 的联动脚本

重点导读生态
Kunpeng 是星链计划 2.0 体系中的一个项目，与该计划下的其他工具共同组成资产测绘与漏洞检测的协作链路。
星链计划 2.0
重点导读效果
Kunpeng 运行效果
本公众号非项目作者，仅做技术分享。
本文介绍的项目开源地址如下：
https://github.com/opensec-cn/kunpeng

## 广告时间  


  
    低价考证包括但不限于CISP系列、PMP等等国内网安证书、网络安全交流群请关注公众号后点菜单栏的找棉花糖。
  
  
    糖心会员站，网络安全必备网站，包括在线内网靶场、web靶场、src靶场、应急响应靶场，以及各种网安资料、教程、方案模版、以及超级多在线工具，99元包年！详细介绍：棉花糖会员站介绍(26年4月26日版本) ：在线内网靶场、网安资料方案、在线工具全能资源站，看完介绍百分百心动！
  
  
    
  
  
    
  
