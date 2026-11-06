#  强推一个自动化 SRC 漏洞挖掘系统  
StanleyNull
                    StanleyNull  菜鸟学信安   2026-11-06 00:32  
  
## 介绍  
##   
  
**AutoHunter**  
 是一个多 Agent 协同的自动化漏洞挖掘系统。你把一台机器交给它当作 7×24 小时不停歇的挖洞平台，自己只做「人工复审员」——AI 初审去掉垃圾洞后，你几分钟内完成调级 / 通过 / 打回 / 编辑 / 标记提交。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/5yYXmGfnscTeBZfG7yauQUTJXlJMs2LrBgOcwQOn3ggg7icrspw2crpPSEzUJVILMK7Xg1fSXxc66hocEhnczTeGPKP8iacSrQuG0FllyYqk4/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=4 "")  
- **Collector：从 FOFA 持续产出目标，探活、预筛、评分、归属标注后入队。**  
- **Worker：每个目标一个 Worker，LLM 自主侦察 + 调用真实工具链挖洞，出洞即提交。**  
- **Reviewer：极理性 AI 初审，过滤半成品 / 误报，只把够格的洞送到人工面前。**  
- **控制台：实时看板一眼看清每个 Worker 在干什么、目标优先级、事件流；结果区高效复审、编辑、标记提交。**  
## 功能亮点  
  
<table><tbody><tr style="box-sizing: border-box;background-color: rgb(255, 255, 255);border-top: 1.11111px solid rgba(209, 217, 224, 0.7);"><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><section style="text-align: left;"><span leaf="">🤝 </span><strong style="box-sizing: border-box;font-weight: 600;margin-bottom: 0px;"><span leaf="">多 Agent 协同</span></strong></section></td><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><section style="text-align: left;"><span leaf="">Collector / Worker / Reviewer / 通杀 / 扩大危害 全流水线自动跑</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(246, 248, 250);border-top: 1.11111px solid rgba(209, 217, 224, 0.7);"><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><section style="text-align: left;"><span leaf="">🌙 </span><strong style="box-sizing: border-box;font-weight: 600;margin-bottom: 0px;"><span leaf="">24×7 无人值守</span></strong></section></td><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><section style="text-align: left;"><span leaf="">挂机过夜，重启自动续跑；你醒来只做复审决策</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(255, 255, 255);border-top: 1.11111px solid rgba(209, 217, 224, 0.7);"><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><section style="text-align: left;"><span leaf="">🔧 </span><strong style="box-sizing: border-box;font-weight: 600;margin-bottom: 0px;"><span leaf="">真实工具链挖洞</span></strong></section></td><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><section style="text-align: left;"><span leaf="">容器内置 nmap · nuclei · sqlmap · httpx · whatweb，LLM 真实发包/执行，不是纸上谈兵</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(246, 248, 250);border-top: 1.11111px solid rgba(209, 217, 224, 0.7);"><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><section style="text-align: left;"><span leaf="">🧪 </span><strong style="box-sizing: border-box;font-weight: 600;margin-bottom: 0px;"><span leaf="">极理性 AI 初审</span></strong></section></td><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><section style="text-align: left;"><span leaf="">只认「实际可利用 + 实锤危害」，过滤半成品，减少无效人工复审</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(255, 255, 255);border-top: 1.11111px solid rgba(209, 217, 224, 0.7);"><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><section style="text-align: left;"><span leaf="">🏫 </span><strong style="box-sizing: border-box;font-weight: 600;margin-bottom: 0px;"><span leaf="">归属证明</span></strong></section></td><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><section style="text-align: left;"><span leaf="">按 IP/域名查 ip138（标记/运营商/ASN 归属地），自动填报告归属单位与证明 + EduSRC 提交 JSON</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(246, 248, 250);border-top: 1.11111px solid rgba(209, 217, 224, 0.7);"><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><section style="text-align: left;"><span leaf="">💥 </span><strong style="box-sizing: border-box;font-weight: 600;margin-bottom: 0px;"><span leaf="">通杀 Hunter</span></strong></section></td><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><section style="text-align: left;"><span leaf="">出洞后自动分析能否「一打一片」，实打多个同款站点验证</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(255, 255, 255);border-top: 1.11111px solid rgba(209, 217, 224, 0.7);"><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><section style="text-align: left;"><span leaf="">🧠 </span><strong style="box-sizing: border-box;font-weight: 600;margin-bottom: 0px;"><span leaf="">情报沉淀复用</span></strong></section></td><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><section style="text-align: left;"><span leaf="">验证过的凭证/端点/指纹入全局情报库，后续 Worker 直接复用</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(246, 248, 250);border-top: 1.11111px solid rgba(209, 217, 224, 0.7);"><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><section style="text-align: left;"><span leaf="">🛡️ </span><strong style="box-sizing: border-box;font-weight: 600;margin-bottom: 0px;"><span leaf="">内置 WAF + 鉴权</span></strong></section></td><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><section style="text-align: left;"><span leaf="">应用层 WAF 默认开启，多角色访问令牌（全权/只读/观摩）</span></section></td></tr><tr style="box-sizing: border-box;background-color: rgb(255, 255, 255);border-top: 1.11111px solid rgba(209, 217, 224, 0.7);"><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><section style="text-align: left;"><span leaf="">💾 </span><strong style="box-sizing: border-box;font-weight: 600;margin-bottom: 0px;"><span leaf="">数据库备份</span></strong></section></td><td style="box-sizing: border-box;padding: 6px 13px;border-color: rgb(209, 217, 224);border-style: solid;border-width: 1.11111px;border-image: none 100% / 1 / 0 stretch;"><section style="text-align: left;"><span leaf="">设置页一键下载/恢复 SQLite（在线一致快照），可选打包工作目录，服务器定时留快照</span></section></td></tr></tbody></table>  
## 快速开始  
  
Tip  
  
全程基于 Docker + Docker Compose v2，任意装得上 Docker 的系统都能跑。生产环境推荐 Linux（2C4G 起步，磁盘 ≥ 20G）。  
  
```
# 1. 装 Docker（官方一键脚本，已装可跳过）
curl -fsSL https://get.docker.com | sh && sudo systemctl enable --now docker
sudo usermod -aG docker $USER && newgrp docker      # 免 sudo，重登生效

# 2. 拉代码 + 一键部署（交互式引导：填 LLM/FOFA Key → 自动生成令牌 → 构建启动）
git clone https://github.com/StanleyNull/AutoHunter.git autohunter && cd autohunter
bash scripts/install.sh
```  
  
引导脚本会：检查 Docker → 引导填 **LLM API Key**  
（必填）、**FOFA Key**  
（推荐）→ 自动生成高强度访问令牌 → 构建镜像并启动 → 打印访问地址和令牌。  
  
首次构建会编译前端 + 安装挖洞工具，约 **5–15 分钟**  
，请耐心等待。构建完浏览器访问 http://<服务器IP>:18800/  
，用打印出的令牌登录。  
  
**开放端口**  
（默认 18800，云服务器还需在厂商安全组放行）：  
```
sudo ufw allow 18800/tcp                                              # Ubuntu/Debian
sudo firewall-cmd --permanent --add-port=18800/tcp && sudo firewall-cmd --reload   # CentOS/RHEL
```  
  
Windows（Docker Desktop + WSL2）  
  
1. 管理员 PowerShell 装 WSL2：wsl --install  
，装完重启。  
  
2. 装   
Docker Desktop  
，安装勾选 “Use WSL 2 based engine”，Settings → Resources → WSL Integration 打开集成。  
  
3. 在 **WSL / Git Bash**  
 里：  
```
git clone https://github.com/StanleyNull/AutoHunter.git autohunter && cd autohunter
bash scripts/install.sh
```  
  
4. 访问 http://localhost:18800/  
。  
  
💡 代码放在 **WSL 文件系统内**  
（如 ~/autohunter  
）比放 C:\  
 挂载盘性能好很多。只用 PowerShell 的话走下方手动部署。  
  
项目地址  
  
https://github.com/StanleyNull/AutoHunter  
  
  
