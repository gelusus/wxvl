#  Strix AI 自动化渗透测试工具实战：安装部署、配置与漏洞挖掘教程  
原创 小枣信安
                        小枣信安  小枣信安   2026-10-28 10:40  
  
# Strix介绍  
  
网络安全怎么搞，小枣信安来唠唠。  
  
今天介绍的是关于Strix智能体如何进行黑白盒漏洞挖掘，Strix是一款开源的AI渗透测试工具，地址是github上的usestrix/strix。该工具可以接收URL通过智能体进行黑盒扫描，也可以接收一个源码目录或github源码地址进行白盒扫描，同时也可以接收源码+URL的混合模式，即源码发现问题后，通过黑盒进行验证。  
## 环境准备  
  
Strix推荐部署在Linux，我这里是Windows系统，所以就以自带的WSL2中的Ubuntu为例，另外Strix运行漏洞验证时需要隔离沙箱，需要用到Docker，所以Windows上也要装好Docker（直接安装Docker Desktop即可）。  
  
因为Docker运行需要Linux内核，所以Windows上的Docker需要借助Linux虚拟机，WSL2是Windows内置的一个轻量的Linux虚拟机，相比较传统的Hyper-V来说，更稳定，性能也更快，所以这里使用WSL2作为Docker的引擎。  
  
打开Docker Desktop，在设置-General下查看运行引擎是否选择的WSL2。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/b4u6JicHMjECerdxr4EL4SXemsMDvyc1L38AuWphZiaicjZIvibo0jQ9PBQDdiaHjUIRoU6IOsFFKbzaukyUWS75kK3vb7SAvyib9l4YBf0Ye0wxg/640?wx_fmt=png&from=appmsg "")  
  
之后在设置-Resources下找到WSL integration，这个是WSL集成，下面有个启用集成的选项，需要选中。  
  
WSL是个虚拟机，Ubuntu是跑在这个虚拟机上的，当开启Docker集成后，Ubuntu中就能直接调用宿主机上的Docker了，不用在Ubuntu中再单独安装了  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/b4u6JicHMjEDnSic43meaZPgETDiaYtbc1kd7IGibIibIMnAIun9wDdprtH56AX6Fm1YQJaBThicib4NL3fnicuLZbFGgRGmr3WOaseEM4eU85Wvnb4/640?wx_fmt=png&from=appmsg "")  
  
通过wsl -l -v可以查看wsl虚拟机上的系统，通过-d参数指定系统名，可以进入系统：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/b4u6JicHMjEBVVYRSiaenbnF7tdrEQiaGtUereZ4xar6kuA1r3sN3WUldbiaL5Us38esPpMudN9ia3sflRUKBUjvZwzibciav6pPA8JgK6BlEjbmSo/640?wx_fmt=png&from=appmsg "")  
  
进入系统后就能执行docker命令了，这里为了后续strix调用docker碰到权限等问题，我把当前用户加入到了docker用户组。  
```
# 将当前用户加入docker用户组
sudo usermod -aG docker $USER
# 立即刷新用户组凭据
newgrp docker
# 验证权限，可列出宿主机docker上跑的容器
docker ps

```  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/b4u6JicHMjEDaDrH6coyF0v1zt4nPGqZEnY8vxALODMLlTjAIVpcGibaQCdztMuAtueSbO9ib2Fz5zZVicCicFdBw1SNgfsHx1yF6QklxGLVFvr0/640?wx_fmt=png&from=appmsg "")  
  
  
此时环境就准备完毕了。  
## Strix部署  
  
Strix官方提供了一键安装脚本，直接在Ubuntu中运行如下命令：  
```
curl -sSL https://strix.ai/install | bash

```  
  
这里可能会碰到一些问题，记录一下：  
  
问题1：安装过程中可能会碰到网络问题，可以给终端添加http和https代理，因为我这里在WSL中，代理在宿主机，所以代理地址要填WSL的网关IP，注意宿主机的代理应用要开启允许局域网连接功能，配置命令如下：  
```
# 获取网关IP
export HOST_IP=$(ip route | grep default | awk '{print $3}')
# 设置代理
export http_proxy="http://${HOST_IP}:7890"
export https_proxy="http://${HOST_IP}:7890"
# 查看外部网络是否通畅
curl -I https://google.com

```  
  
问题2：安装过程中也可能会碰到Docker的沙盒镜像拉取失败问题，它会给出手动pull的命令，手动pull一下即可。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/b4u6JicHMjECy3LWqLthMGgztWmmma0yzibJukJpI45WJAQKDvG3euC6vSMyj9iaCvnYf41Tyqia6ib7TeqgsQAtmrkOcicND3R8hMH34EO3X2DZs/640?wx_fmt=png&from=appmsg "")  
  
  
问题3：代理配好后，如果代理不通，可能是Windows防火墙拦截，可以添加一个入站规则，把代理端口允许一下。  
  
除此外，官方还提供了另一种安装方式，可通过pip、pipx、uv进行安装，任选一种即可，参考命令：  
```
pip install strix-agent
pipx install strix-agent
uv tool install strix-agent

```  
  
安装后需要配置下大模型的KEY，Strix支持多个服务商，这里以Deepseek为例，编辑~/.bashrc文件，将配置以环境变量形式添加进去：  
```
export STRIX_LLM="deepseek/deepseek-chat" 
export LLM_API_KEY="你的_API_KEY"

```  
  
这里不需要填模型地址，Strix内部会根据模型名进行匹配，配置后需要刷新下环境变量，执行如下命令进行刷新：  
```
source ~/.bashrc

```  
  
此时Strix就部署好了。常用的参数不多，可以自行查看：  
```
strix -h

```  
## Strix黑盒  
  
先来看下它的黑盒模式，只需要提供一个URL，通过target参数指定，这里为了方便测试，我让模型用python写了一个简单的商城系统，可以直接在GPT或豆包聊天页面让它编写，完事打包成zip，我们直接下载运行就行，同时让它故意留几个漏洞进行。  
```
strix --target http://127.0.0.1:5000

```  
  
可以发现它是通过多个智能体去完成测试任务，会先从信息搜集开始，调用自身集成的相关工具去执行。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/b4u6JicHMjEABK1V0qWnlIdQDEYgpxKIbTQ6q60myaQtsCdaTyO30AS3boWtGScteGlvyblzWgfiaL7lWva1Vl1SBEiceIVVcyicZNtQsjFGyyE/640?wx_fmt=png&from=appmsg "")  
  
  
任务完成后会给出一个整体结果信息，同时会在运行strix命令的目录下创建一个strix_runs文件夹用于存放漏洞报告。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/b4u6JicHMjECt6eF8aeHoU4kULLm3eiaxpSOh7ibbXZq2MxK9XQLXHxLcuHAdEK4szokP70Svj54Bqbn4sblZ6TJ5fJ5ibAeicAXTgA3n5BmqnMA/640?wx_fmt=png&from=appmsg "")  
  
同时也可以通过instruction参数添加提示词，这个提示词会给到智能体，该参数也有很多应用场景，比如目标平台需要登录，我们可以把账号密码通过提示词传入，如下示例：  
```
strix --target http://127.0.0.1:5000 --instruction "平台登录账号是admin，密码是AdminDemo123!"

```  
  
Strix也提供了页面查看功能，通过view参数会启动一个web服务，可浏览器直接访问，在该页面上可查看相关的智能体及漏洞情况：  
```
strix view

```  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/b4u6JicHMjECgrvE0Qh0icrIrZ8ia7CQY3ts2JZ6PVxa7iaSf7qqDt1YiafdEwCcj7icv91Bwr2a0M5fZw27UAkPqHhgU8jbjFBqicHYMcCdUAZbqk/640?wx_fmt=png&from=appmsg "")  
  
## Strix白盒  
  
对源码扫描也是通过target参数，指定本地源码所在目录，或者是github地址，示例如下：  
```
strix --target /mnt/c/Users/Administrator/Desktop/store

```  
  
这个源码扫描效果感觉挺好，不仅找出了故意留下的所有漏洞，同时也找出了额外的一些漏洞，当然，这些漏洞仅仅是通过分析源码得到的，并不代表就一定能复现成功。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/b4u6JicHMjEAibzRqkOhVvopT5iciaM5Z6SIKQpH7GQzThsLpZcOIicRXyuznVLgFnsA677xJh8zTRxGrYkZY4LHiavLCA4perHwF3iby4Gar9eovc/640?wx_fmt=png&from=appmsg "")  
  
  
针对每个漏洞Strix都会单独写一个报告，报告会非常详细，包括了从描述、危害、利用、POC、修复等等内容。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/b4u6JicHMjEDhPpByBLQHxwcRoIOdhK30LOOrRgmxmm7gF0NWib6hZRyiaibqCmictvgF8MDT26n3Jibd4Lkh2MbxYwzwZCTHBSlA7hM7ia5UX3tFQ/640?wx_fmt=png&from=appmsg "")  
## Strix混合模式  
  
在白盒模式中，Strix报告了很多漏洞，但是并不代表百分百都存在，我们可以用混合模式，让其对源码报出来的漏洞进行验证，这里需要通过两个t参数，即还是target参数，一个指定源码，一个指定URL，如下示例：  
```
strix -t /mnt/c/Users/Administrator/Desktop/store -t http://127.0.0.1:5000

```  
## 总结  
  
Strix不论是黑盒还是白盒还是混合模式，都是通过target这一个参数指定的。Strix在任务完成后通常会进行一些询问，比如是否有进一步信息提供等，如果想让它全自动，可以加上-n参数，全称是non-interactive，代表非交互模式运行。需要注意的是，对于很多登录功能，带有图形验证码、非人类识别或者手机验证码的，则需要我们参与一下，登录成功后，可以把cookie等认证信息放到指令参数中供Agent使用。  
  
经过上面测试，发现有源码是效果最好的，之后就是黑盒带凭证的情况，最后就是无凭证的情况。  
  
小枣信安：  
专注于网络安全，涵盖智能体安全、大模型安全、漏洞挖掘、红蓝攻防、应用安全、AI赋能网络安全等方向。  
  
