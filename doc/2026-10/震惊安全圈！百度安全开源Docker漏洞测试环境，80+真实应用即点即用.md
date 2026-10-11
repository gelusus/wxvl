#  震惊安全圈！百度安全开源Docker漏洞测试环境，80+真实应用即点即用  
棉花糖糖糖
                    棉花糖糖糖  棉花糖网络安全工具箱   2026-10-11 02:39  
  
安全声明：本文仅用于技术研究，任何涉及漏洞利用的行为需遵守适用法律与安全规范。
重点导读概述
app-env-docker 是百度安全开源的基于 Docker 的真实应用测试环境。该项目主要服务于安全研究人员，支持漏洞复现、扫描器开发、OpenRASP 防护效果验证等场景。项目仓库包含 80 余个真实应用环境，覆盖 Web 漏洞、中间件、数据库等多种类型。
重点导读特性
PART 01极速部署
所有漏洞资源托管于百度云存储节点，国内访问下载速度显著优于海外镜像源。环境启动仅需执行 make -C src/<应用名>/<版本号> 即可完成构建与运行。
PART 02纯 Dockerfile 架构
每个漏洞环境均采用原生 Dockerfile 定义，完全透明可定制。相比预编译镜像方案，Dockerfile 方式支持灵活修改配置参数，满足个性化测试需求。
PART 03漏洞覆盖全面
项目专注 Web 安全领域，涵盖以下类型漏洞环境：



类别
代表性应用




CMS 系统
Discuz、WordPress、Drupal、Joomla


中间件
Tomcat、Weblogic、Jetty、Apache


框架类
Struts、Spring、Shiro、ThinkPHP


数据库
MySQL、Redis、MongoDB、Elasticsearch



PART 04Socks5 智能代理
项目内置改造版 socks5 代理服务，支持基于域名自动识别目标容器并转发请求。相比修改本机 DNS 方式，该方案安全性更高，且支持全端口转发。
重点导读架构
PART 05核心组件

src 目录：存放所有漏洞环境的 Dockerfile 与构建脚本
socks5 目录：自动化代理服务实现

PART 06代理工作流程

浏览器或工具通过 *.xip.io 域名访问
socks5 代理解析子域名获取容器名称
调用 Docker HTTP API 查询容器 IP
自动建立转发连接

重点导读使用方法
PART 07构建环境
bashmake -C src/zzcms/8.2

PART 08启动代理服务
bashcd socks5
GOPATH=$PWD go build -o /tmp/docker-socks5
/tmp/docker-socks5

PART 09配置 PAC 脚本
javascriptfunction FindProxyForURL(url, host) {
    if (host.indexOf('.192.168.154.200.xip.io') != -1) {
        return 'SOCKS5 192.168.154.200:3080';
    }
    // ...
}

重点导读截图
socks5代理服务主界面
重点导读应用场景

CVE 漏洞覆盖验证
Web 安全扫描器开发测试
RASP 防护产品效果评估
安全研究员日常漏洞研究

本公众号非项目作者，仅做技术分享。
本文介绍的项目开源地址如下：
https://github.com/baidu-security/app-env-docker

## 广告时间  


  
    低价考证包括但不限于CISP系列、PMP等等国内网安证书、网络安全交流群请关注公众号后点菜单栏的找棉花糖。
  
  
    糖心会员站，网络安全必备网站，包括在线内网靶场、web靶场、src靶场、应急响应靶场，以及各种网安资料、教程、方案模版、以及超级多在线工具，99元包年！详细介绍：棉花糖会员站介绍(26年4月26日版本) ：在线内网靶场、网安资料方案、在线工具全能资源站，看完介绍百分百心动！
  
  
    
  
  
    
  
