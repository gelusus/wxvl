#  RCE 拿到了，却只会用 netcat？这才是真正的反弹 Shell 与 TTY 升级手册  
原创 暗夜铭少
                    暗夜铭少  黑帽渗透技术   2026-10-31 23:41  
  
反弹Shell  
多语言  
TTY升级  
权限维持  
痕迹清理  
付费合集 · 第 12 篇  
  
在前面 11 篇的利用链里，我们无数次用到了反弹 Shell。很多师傅觉得，反弹 Shell 不就是 nc -e /bin/bash  
 吗？背个命令就完事了。  
  
但在真实的内网环境中，你永远不知道对面的机器到底支持什么：**没有 nc、没有 bash、只有 sh、Python 版本太低、防火墙只允许特定端口出网、甚至连个正经的 TTY 都没有，按一下退格键满屏乱码。**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/VqJrJwib7Fo2urUpb9KN4RpYhwbWsZMcLvDdAzrnauHgelBicowoe64cNBTKeDqLoxiaNBw7xa8ichRpmM7bGgJTibd9dt0BYHoCFxh7AxWPkdBc/640?wx_fmt=png&from=appmsg "")  
  
