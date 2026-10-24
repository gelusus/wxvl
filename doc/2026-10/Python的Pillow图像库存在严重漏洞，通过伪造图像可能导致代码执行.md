#  Python的Pillow图像库存在严重漏洞，通过伪造图像可能导致代码执行  
原创 骨哥说事
                    骨哥说事  骨哥说事   2026-10-24 03:01  
  
<table><tbody><tr><td data-colwidth="557" width="557" valign="top" style="word-break: break-all;"><h1 data-selectable-paragraph="" style="white-space: normal;outline: 0px;max-width: 100%;font-family: -apple-system, system-ui, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.544px;background-color: rgb(255, 255, 255);box-sizing: border-box !important;overflow-wrap: break-word !important;"><strong style="outline: 0px;max-width: 100%;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="outline: 0px;max-width: 100%;font-size: 18px;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="color: rgb(255, 0, 0);"><strong><span style="font-size: 15px;"><span leaf="">声明：</span></span></strong></span></span></strong><span style="outline: 0px;max-width: 100%;font-size: 18px;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="font-size: 15px;"><span leaf="">文章中涉及的程序(方法)可能带有攻击性，仅供安全研究与教学之用，读者将其信息做其他用途，由用户承担全部法律及连带责任，文章作者不承担任何法律及连带责任。</span></span></span></h1></td></tr></tbody></table>#   
  
#   
  
****# 防走失：https://gugesay.com/  
  
**不想错过任何消息？设置星标****↓ ↓ ↓**  
#   
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlbXyV4tJfwXpicwdZ2gTB6XtwoqRvbaCy3UgU1Upgn094oibelRBGyMs5GgicFKNkW1f62QPCwGwKxA/640?wx_fmt=png&from=appmsg "")  
  
## Python图像处理的基石面临严重安全威胁  
  
2026年10月16日，Python社区最广泛使用的图像库Pillow发布了**版本13.0.0**  
，修复了超过40个安全漏洞，其中包括多个**CVSS评分9.2**  
的关键漏洞。最引人关注的是**CVE-2026-112900**  
，这个参数注入漏洞允许攻击者通过操纵EPS图像文件名，利用Ghostscript实现在服务器上任意命令执行。根据OX Security估计，Pillow每月下载量高达**4.36亿次**  
，这意味着全球Python图像处理生态系统面临重大安全风险。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/TKdPSwEibsZiaAURL7YrZWo8L6maosm2F5wWKEtCdYAqJzVYP25SXAQMoVZyuXxA8LcO9aQJgqClYPVrJEKKb3ybaVlZEicx2B1nHeLPlyUicEo/640?wx_fmt=png&from=appmsg "")  
## 三个CVE，多个攻击向量  
### CVE-2026-112900：Ghostscript参数注入到RCE  
  
这是最危险的漏洞，存在于Pillow处理EPS图像的方式中。Pillow打开EPS图像时需要调用外部程序Ghostscript进行光栅化，但存在严重的代码实现缺陷：  
  
**根因**  
：PIL.EpsImagePlugin.Ghostscript()  
构建Ghostscript命令行时，使用输入文件路径(fp.name  
)而**未进行消毒**  
，将其放置在-f <infile>  
位置。文件名开头的-  
会被Ghostscript解释为选项而不是文件，从而允许命令行参数注入。这可以将**攻击升级为任意命令执行**  
（需要开头的@  
参数文件文件名，并控制引用的包含额外Ghostscript参数的文本文件）。  
  
**影响版本**  
：从Pillow 2.6.0直到13.0.0的所有版本。该漏洞存在于Pillow的默认配置中（启用-dSAFER  
），已在Pillow 12.3.0和Ghostscript 10.02.1上验证，最新GS版本(10.07.1)同样受影响。  
### CVE-2026-112393：TIFF内存缓冲区溢出  
  
该漏洞自2013年以来就存在于Pillow中，当程序将压缩的TIFF图像保存到内存（如io.BytesIO()  
）而不是磁盘文件时，会发生内存缓冲区溢出。这对于重新编码上传文件并返回的服务来说是非常常见的模式。安全公告指出，报告人员使用它使进程崩溃，**仅在第单独的内存地址泄漏缺陷的帮助下才能将其转化为代码执行**  
。保存到磁盘上的实际文件不受影响。  
### CVE-2026-112403：JPEG缩略图缓冲区溢出  
  
该漏洞在程序对精心制作的JPEG进行缩略图制作时溢出内存缓冲区。安全公告称其导致的是"崩溃，而不是代码执行"。特别值得注意的是，**这个漏洞是Anthropic使用Claude研究开源软件安全性时发现的**  
，并由Ada Logics手动验证。这是公司发现**数百个开源库缺陷**  
的更广泛努力的一部分。  
## 技术深度分析：CVE-2026-112900的实现细节  
### 漏洞核心：EPS文件路径处理缺陷  
  
在src/PIL/EpsImagePlugin.py  
中的关键漏洞代码：  
```
infile_temp = Noneif hasattr(fp, "name") and os.path.exists(fp.name):    infile = fp.name                      # 用户控制的路径直接使用，未经验证else:    in_fd, infile_temp = tempfile.mkstemp()    os.close(in_fd)    infile = infile_temp    # ... (复制文件指针到临时文件)# 构建Ghostscript命令command = [    gs_binary,    "-q",    f"-g{width:d}x{height:d}",    f"-r{res_x:f}x{res_y:f}",    "-dBATCH",    "-dNOPAUSE",    "-dSAFER",                            # 默认沙箱 — 不阻止此注入    f"-sDEVICE={device}",    f"-sOutputFile={outfile}",    "-c",    f"{-bbox[0]} {-bbox[1]} translate",    "-f",    infile,                               # 在此处注入：插入到参数，未检查选项前缀(-, --)     "-c",                                 # 或Ghostscript的@-文件标记    "showpage",                           # => 参数注入(CWE-88)]# ...subprocess.check_call(command, startupinfo=startupinfo)   # 生成Ghostscript
```  
### 攻击载荷构造  
  
攻击者可以通过以下方式利用此漏洞：  
  
**参数注入验证**  
：因为文件名未经验证，前导的-  
/--  
被Ghostscript解析为选项而不是文件。控制传递给Image.open()  
的文件名的攻击者可以注入Ghostscript命令行参数。单参数注入始终成功。  
  
**潜在RCE（已验证）**  
：文件名也未经验证以防止Ghostscript的@  
-文件语法：形式为@<path>  
的文件名会导致Ghostscript从<path>  
读取额外的命令行参数，允许多个参数同时注入。如果攻击者还能控制引用的文件，则可以升级为任意命令执行。  
### 攻击场景示例  
  
攻击者可以创建如下结构的攻击载荷：  
```
EPS = b"%!PS-Adobe-3.0 EPSF-3.0\%%BoundingBox: 0 0 100 100\%%EndComments\nshowpage\n"name = "@arg"# 文件名控制with open(name, "wb") as f:    f.write(EPS)# 在文本文件'arg'中包含：# -sOutputFile=%pipe%/bin/id# -c# 1# dict# dup# /OutputFile# (%pipe%/bin/id)# put# setpagedevice
```  
  
当使用Image.open(name).load()  
时，Pillow会执行id  
命令。  
## FlashPix(FPX) JPEG漏洞深度分析  
### CVE信息不足但同样致命  
  
在Pillow 13.0.0的发布说明中，还有一个关于FlashPix(FPX)格式的高危漏洞：  
  
GHSA-5x8w-8jj5-q52h  
：FlashPix(FPX) JPEG解码中的堆缓冲区溢出导致代码执行  
  
**漏洞机制**  
：在FpxImagePlugin  
中，对于JPEG压缩的子图像(compression == 2  
)，插件构建"jpeg"  
解码图块，并通过MODES  
表从攻击者控制的FPX颜色ID派生(rawmode, jpegmode)  
对。输出缓冲区按rawmode大小分配，但由于模式映射错误，libjpeg写入的数据超出了分配的缓冲区大小。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/TKdPSwEibsZhQINZPqseTgbVxc5xrGHZ8qZykbCY1OoRjVf8QIHvPa5VKC0mIjlUmWmdTnZHnJZGnC3Gqyno2wHbXaywC1Maf8icibSpM51Cpk/640?wx_fmt=png&from=appmsg "")  
### 关键攻击特征  
1. **内容检测而非扩展名检测**  
：Image.open()  
通过OLE2魔法字节D0CF11E0...  
自动检测FPX文件，只要有可选的olefile  
包  
  
1. **无需库扩展名**  
：服务器可以处理任何文件类型，无论扩展名如何  
  
1. **单文件执行**  
：打开一个精心制作的文件即可实现代码执行  
  
### 真实影响示例  
  
根据漏洞报告者的PoC演示，攻击者可以制作一个单独的.fpx  
文件，当目标应用程序使用Image.open(sys.argv[1]).load()  
打开它时，可以执行任意命令：  
```
$ python3 vulnerable.py shell.fpxuid=0(root) gid=0(root) groups=0(root)corrupted size vs. prev_size
```  
  
Image.open()  
的精美文件通过system("id")  
运行了baked命令：以运行图像处理的用户身份执行命令。  
## 严重性评估与统计信息  
### CVSS评分分歧  
  
**官方vs实际严重性**  
：  
- **Pillow维护者评级**  
：中等（因为攻击者需要控制文件名）  
  
- **GitHub分配的CVE记录评级**  
：9.2/10（关键）  
  
- **实际情况**  
：在适当的攻击场景下，这确实是一个关键漏洞  
  
### 影响范围的统计事实  
1. **月下载量**  
：约4.36亿次  
  
1. **无处不在**  
：Web上传表单、头像和缩略图工具、文档和PDF管道、大多数机器学习项目都使用它  
  
1. **行业影响**  
：WordPress、Drupal、Joomla、Magento、Shopware等框架和平台都依赖Pillow  
  
### 修复版本对比旧版本影响  
```
受影响版本：2.6.0 ≤ Pillow < 13.0.0修复版本：Pillow 13.0.0修复方案：os.path.abspath(fp.name) 确保路径以路径分隔符或驱动器字母开头
```  
## 实际应用风险分析  
### 文件上传处理的典型风险场景  
  
**常见易受攻击模式**  
：  
```
# 漏洞代码示例from PIL import Imageimport osdef process_uploaded_image(uploaded_file):    # 使用原始文件名 - 这很危险    filename = uploaded_file.filename    # 保存文件    uploaded_file.save(filename)    # 打开处理 - 如果文件是EPS且文件名被恶意利用    image = Image.open(filename)    image.load()  # 触发Ghostscript调用    # ... 其他处理# 安全代码示例def safe_process_uploaded_image(uploaded_file):    # 生成安全的随机文件名    import uuid    safe_name = f"{uuid.uuid4()}.tmp"    # 保存文件    uploaded_file.save(safe_name)    try:        image = Image.open(safe_name)        # 检查文件类型        if image.format == 'EPS':            # 对于EPS文件需要特别小心            raise ValueError("EPS文件处理需要特别安全措施")        image.load()    finally:        # 清理临时文件        os.remove(safe_name)
```  
### 典型易受攻击的应用类型  
1. **社交媒体平台**  
：用户头像上传和缩略图生成  
  
1. **电子商务网站**  
：产品图片上传和处理  
  
1. **文档处理服务**  
：PDF到图像转换服务  
  
1. **机器学习平台**  
：图像数据预处理管道  
  
1. **内容管理系统**  
：媒体库管理  
  
## 紧急缓解措施与长期解决方案  
### 立即缓解措施（升级前）  
  
**禁用Ghostscript EPS支持**  
：  
```
# 在所有处理不可信上传的主机上移除gs程序sudo apt remove ghostscript  # Debian/Ubuntusudo yum remove ghostscript  # RHEL/CentOS# 或在应用程序代码中禁用EPS处理import PILPIL.EpsImagePlugin._FORCE_EPS = False
```  
  
**临时配置修改**  
：  
```
# 在php.ini或FPM池中设置session.upload_progress.enabled = 0# 注意：不是通过ini_set()或.user.ini设置
```  
  
**避免危险的API使用模式**  
：  
```
# 危险操作image.thumbnail((100, 100))  # 对不可信JPEG使用thumbnail()image.draft()                # 对不可信JPEG使用draft()# 更安全的替代方法image.load()                 # 首先加载image.thumbnail((100, 100), reducing_gap=None)  # 传递reducing_gap=None
```  
### 永久解决方案  
  
**升级到Pillow 13.0.0**  
：  
```
pip install -U "Pillow>=13.0.0"
```  
  
**重要注意事项**  
：  
1. Pillow 13放弃了Python 3.10支持（2026年10月结束生命周期）  
  
1. 必须重新启动所有Python进程/容器，因为旧工作进程仍在运行易受攻击的代码  
  
1. 验证运行版本：import PIL; print(PIL.__version__)  
  
### 依赖管理策略  
  
**容器和打包注意事项**  
：  
```
# 在Dockerfile中FROM python:3.12-slimRUN pip install --upgrade pipRUN pip install "Pillow>=13.0.0"  # 明确指定版本# 验证RUN python -c "import PIL; assert PIL.__version__ >= '13.0.0'"
```  
  
**Python版本兼容性矩阵**  
：  
- Python 3.11+：支持Pillow 13.0.0  
  
- Python 3.10：需要升级到Python 3.11或更高版本  
  
- 旧版Python：考虑使用供应商提供的后向移植修复  
  
  
### 安全代码审查清单  
  
**图像处理代码审查重点**  
：  
1. **文件名控制**  
：是否传递用户控制的文件名给Image.open()  
  
1. **文件类型验证**  
：是否基于信任内容而非扩展名  
  
1. **临时文件管理**  
：是否使用安全临时文件  
  
1. **错误处理**  
：是否妥善处理异常情况  
  
1. **资源清理**  
：是否及时清理临时资源  
  
### 生产环境监控  
  
**关键监控指标**  
：  
1. **Ghostscript调用频率**  
：监控意外的Ghostscript进程启动  
  
1. **内存使用模式**  
：检测异常的缓冲区溢出模式  
  
1. **文件类型分布**  
：跟踪上传文件类型的异常变化  
  
1. **错误率监控**  
：监控图像处理失败率的异常上升  
  
## 结论与行动号召  
### 关键结论  
1. **严重性不容忽视**  
：Pillow中的漏洞可能允许**未经身份验证的远程代码执行**  
  
1. **影响范围极大**  
：每月4.36亿次下载意味着全球影响  
  
1. **修复紧迫性**  
：已有多个关键漏洞修复可用  
  
1. **安全意识教育**  
：开发人员需要了解图像处理的安全风险  
  
### 立即行动清单  
  
**对于开发人员**  
：  
1. 立即升级到Pillow 13.0.0  
  
1. 检查所有使用Image.open()  
的代码路径  
  
1. 实施文件类型验证和内容检测  
  
1. 审查所有上传处理逻辑  
  
**对于运维团队**  
：  
1. 更新所有部署中的Pillow版本  
  
1. 重新构建容器镜像和部署包  
  
1. 实施运行时监控和异常检测  
  
1. 制定应急响应计划  
  
**对于安全团队**  
：  
1. 扫描代码库中的Pillow使用  
  
1. 评估第三方依赖的漏洞影响  
  
1. 更新WAF规则以检测异常图像上传  
  
1. 开展安全意识培训  
  
Pillow作为Python图像处理的基石，其安全性关系到整个Python生态系统。这次安全更新不仅是漏洞修复，更是对整个开源安全维护的一次重要提醒：**即使是经过多年审查的成熟项目也可能存在严重安全缺陷**  
，而持续的安全投资和及时的更新是维护可信软件生态的必要条件。  
  
**参考资料**  
：  
- The Hacker News：Python流行Pillow图像库中的关键漏洞允许通过精心制作的图像执行代码   
  
- GitHub安全公告：Ghostscript调用中的参数注入，可能导致RCE  
  
- CVE记录：CVE-2026-112900  
  
- Pillow 13.0.0发布说明  
  
- GitHub安全公告：FlashPix(FPX) JPEG解码中的堆缓冲区溢出导致代码执行  
  
- END -  
  
**感谢阅读，如果觉得还不错的话，动动手指给个三连吧～**  
  
