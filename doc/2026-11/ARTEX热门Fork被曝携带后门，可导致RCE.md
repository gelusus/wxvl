#  ARTEX热门Fork被曝携带后门，可导致RCE  
微步情报局
                    微步情报局  微步在线   2026-11-04 06:39  
  
## 一、事件概要：热门Fork有后门  
  
11月4日，AI 自主渗透测试工具ARTEX的热门Fork项目被曝出携带后门。事件起因为今年10月ARTEX下架后，社区出现大量Fork。其中高星仓库trhhubsd/ARTEX ，  
维护方将docker-compose 中的官方镜像替换为恶意镜像 tomicdd/artex。  
  
该镜像在登录接口植入后门：当 captcha 字段以“!”开头时，直接通过 sh -c 执行命令并在密码验证前返回结果，实现未认证远程命令执行。  
  
目前，该 Fork 项目已被大量使用，建议用户立即排查。  
## 二、攻击事件与后门分析  
  
攻击者使用的ARTEX克隆仓库为：  
  
https://github.com/trhhubsd/ARTEX  
，截至目前该克隆仓库已有1.3k的Star（项目已被删除）：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/KJ5kFuz2K95oQgEicibATsHibrE6B1e1h9qjTVXfdeA5kAZasAhLLbic9801lHfe257HKicOpSzQgBuEuTn8leaMt7p2VsibAnqia1kEVoMNuY5icRs/640?wx_fmt=png&from=appmsg "")  
  
在该克隆仓库中的Issues，已经被指出该克隆项目存在后门：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/KJ5kFuz2K95qdnKHBBYvTGDThNKkyhicBNN2iaNXMq7pZVp1h2cOqBPUYM7Pwn4Wwk0hb3Et34kUML50ibia5e58mUS7Zhje8JNCyHYLcgicR1SA/640?wx_fmt=png&from=appmsg "")  
  
克隆仓库存在一处改动：trhhubsd/ARTEX 修改了 docker-compose.yml，把官方镜像 autumn27/artex 替换为 tomicdd/artex：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/KJ5kFuz2K95iaibbKefuiakgWKMfxrz1b7oLTd9ZrrogJD7MUicm5eJ1MaxtIG9TbyaTrx3DYF1ADVHtxsUNKRzp8EB1HEw0AcP3avzSSNdxGlw/640?wx_fmt=png&from=appmsg "")  
  
而该镜像累计下载量为8.2k，镜像hash为：  
  
sha256:ce85d8369ccc1a85423f1f75d2f3afaa2fab759e580da37a6891af2f7392745e  
  
在该镜像的 app/artex 文件存在后门函数 verifyCaptcha：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/KJ5kFuz2K961NwCfAUNwsFkSPa5rELU9qfOVFJXKN6WSJy34ZaNnloeIWNweGy1DmY4QDjFQUGL8JH0FmFQYlkkxl71BacFOzYOI5zulLOA/640?wx_fmt=png&from=appmsg "")  
  
verifyCaptcha 将传入内容作为 sh -c 的命令执行，并返回合并后的标准输出/错误。而调用这个后门函数的是在入口登录函数 authLogin：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/KJ5kFuz2K97hKgdCfQGYrlaDEzFoYllq5Mw5aMxqCYiaXlibbnInrzko5T42BWxiamv1cibia37npZubDfSWicQRzoDcHltM2VpLkcrribyfO7pQvM/640?wx_fmt=png&from=appmsg "")  
  
整体后门机制为  
：  
1. 登录接口 /api/auth/login 额外接收 captcha 字段。  
  
1. 若 captcha 以 ! 开头，则去掉 ! 后执行 sh -c <剩余内容>。  
  
1. 通过 os/exec.Command 运行命令，返回 stdout/stderr。  
  
1. 在用户名/密码验证之前  
 直接返回 HTTP 200 和命令输出。  
  
## 三、排查措施与缓解方案  
- 立刻停止相关容器与编排，断开对外映射（常见管理端口如 8787）。  
  
- 停用并删除本地缓存的 tomicdd/artex 镜像，避免再次 compose up 拉起。  
  
- 核查该时段登录面、异常进程与出站连接；按需做主机侧排查与重建。  
  
- 若仍需使用ARTEX，不要信任来历不明的一键镜像，可联系微步申请试用ARTEX或者在RedAgents中使用ARTEX相关能力。  
  
## 四、IOC  
  
d5070f52262effe216b840711343bff8f2787f39208d887d8dfd9c673f8d1573  
  
ce85d8369ccc1a85423f1f75d2f3afaa2fab759e580da37a6891af2f7392745e  
  
  
·END·  
  
  
