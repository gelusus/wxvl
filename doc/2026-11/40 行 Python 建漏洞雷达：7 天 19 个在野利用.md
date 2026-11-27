#  40 行 Python 建漏洞雷达：7 天 19 个在野利用  
红队安全圈
                    红队安全圈  红队安全圈   2026-11-27 00:00  
  
等消息告诉你某个产品爆发高危漏洞时，那个洞往往已经在野外打了几轮。自己拉 API 对资产表，一分钟出结果。  
  
## 01  本周的账面数字  
  
近 7 天新增严重级  
46 条（CVSS 9 分档）  
  
近 7 天新进在野清单  
19 条（CISA KEV）  
  
在野清单累计  
1805 条  
  
示例资产表撞出  
Panabit 网关 / WooCommerce 插件  
  
这些数字都是脚本现拉的，不是我记的。下面几块代码跑完就是这张表。  
  
## 02  为什么不能靠推送  
  
一天四五十条 9 分洞，肉眼刷不过来。更难的是名称对不上：公告里写 CVE 号和版本号，资产表里写的是泛微 e-cology、WooCommerce、Traefik 这种产品名。两边没有共同的键，只能靠人脑联想，漏是必然的。  
  
对表这件事适合交给机器：把这一周所有严重级洞的英文描述拉下来，逐条跟资产表里的关键词做包含匹配，命中就打印。粗糙，但一分钟能跑完。  
  
## 03  先把本周的严重洞拉下来  
  
只用标准库 urllib，不用装 requests。时间窗是往前推 7 天，cvssV3Severity=CRITICAL 只留 9 分档。  
  
import json,urllib.request as u  
  
from datetime import(timedelta,  
  
    timezone,datetime)  
  
now=datetime.now(timezone.utc)  
  
f=lambda x:x.strftime(  
  
    '%Y-%m-%dT%H:%M:%SZ')  
  
q='pubStartDate=%s&pubEndDate=%s'  
  
q+='&cvssV3Severity=CRITICAL'  
  
q=q%(f(now-timedelta(7)),f(now))  
  
url='https://services.nvd.nist.gov'  
  
url+='/rest/json/cves/2.0?'+q  
  
r=u.Request(url,headers=  
  
    {'User-Agent':'asset-check'})  
  
resp=u.urlopen(r,timeout=60)  
  
data=json.load(resp)  
  
n=data['totalResults']  
  
print('7天严重洞:',n)  
  
## 04  拿资产表逐条撞  
  
assets 换成你自己的：网关型号、CMS、中间件、VPN 品牌，全写小写关键词。英文描述里出现这个词就算命中。  
  
assets=['panabit',  
  
        'woocommerce','jira']  
  
for v in data['vulnerabilities']:  
  
  c=v['cve']  
  
  desc=[x['value'] for x in  
  
      c['descriptions']  
  
      if x['lang']=='en']  
  
  ks=['cvssMetricV31',  
  
      'cvssMetricV40']  
  
  m=c['metrics']  
  
  s=next((m[k][0]['cvssData']  
  
      ['baseScore'] for k in ks  
  
      if k in m),'')  
  
  hit=[a for a in assets  
  
      if a in desc[0].lower()]  
  
  if hit:  
  
    print(c['id'],s,hit)  
  
    print(' ',desc[0][:80])  
  
## 05  我跑出来的结果  
  
7天严重洞: 46  
  
CVE-2026-135790 9.9 ['panabit']  
  
CVE-2026-135791 9.9 ['panabit']  
  
CVE-2026-135792 10.0 ['panabit']  
  
CVE-2026-135794 9.9 ['panabit']  
  
CVE-2026-104998 9.8 ['woocommerce']  
  
CVE-2026-136240 9.1 ['woocommerce']  
  
46 条里撞出 6 条。Panabit 那几条全挂在同一固件 TANGr7p10 上，被指的接口有 /cgi-bin/dashboard/ajax_chart_dump、/cgi-bin/flowcontrol/ajax_data_pie、user.cgi 和 admin/cgi-bin/sac/ajax_sac_crontab，其中一条走邮件验证码组件。WooCommerce 是两个不同插件：一个是 Product Input Fields 的任意文件上传，一个是 PDF Invoice 的 PHP 对象注入。  
  
一张资产表，六条待办，跑一次的成本是几十秒。  
  
## 06  三个会坑你的地方  
  
关键词匹配只能做粗筛。不少漏洞描述只写库函数名，不提产品名，这类撞不上。要查全就换 CPE 匹配，请求参数用 cpeName，代码要长一倍，命中精度也高得多。  
  
同一个洞会有好几个分数。Panabit 那条 CVE-2026-135790，CVSS 3.1 给 9.9，4.0 给 8.6，老的 2.0 口径给 9.0。代码里我固定先读 3.1、没有再退 4.0，你按 4.0 排序的话，整张优先级表会被压低。  
  
接口有限流。NVD 不带 key 的时候卡得很紧，别把这个请求塞进每分钟跑的循环。拉一次落成 JSON，后面全读本地文件；要频繁跑就去开发者页面申请 key。  
  
## 07  再补一刀：在野清单单独拉  
  
cisa='https://www.cisa.gov/'  
  
cisa+='sites/default/files/feeds/'  
  
cisa+='known_exploited_'  
  
cisa+='vulnerabilities.json'  
  
k=json.load(u.urlopen(cisa))  
  
print('KEV 库:',k['count'])  
  
KEV 只收已被利用的洞，条目少、判断简单：进了这个清单不用再讨论评分，直接排进今晚的修复队列。这周新增的 19 条里，Mitel MiCollab 一家占了 5 条，Danfoss 工控 3 条（含两个 2023 年的老洞才补登），另外 Drupal Core、VMware vCenter、Cisco Expressway、SAP NetWeaver、Apache Roller 各 1 条。  
  
老洞突然进清单这件事本身就值得注意：不是新洞太狠，是有人在捡旧的打。  
  
## 08  一句判断  
  
这套东西两边都能用。蓝队拿它出本周补丁清单，红队拿它给客户做打点前的资产画像，撞到没修的组件就是入口。但 API 拉回来的只是候选，版本号还得自己对一遍，别把匹配结果直接当结论发出去。  
  
  
工具实战技巧更新在下一篇，**关注**  
后新工具上线先收到。  
  
— END —  
  
  
