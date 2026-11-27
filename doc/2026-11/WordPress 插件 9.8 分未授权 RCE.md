#  WordPress 插件 9.8 分未授权 RCE  
红队安全圈
                    红队安全圈  红队安全圈   2026-11-27 12:00  
  
30 秒看懂  
  
影响范围 ·   
WordPress 站点装了 Sneeit Framework 插件 8.7 及更早版本。插件随 Sneeit 系列主题打包，不在官方插件库，未登录即可触发。  
  
利用状态 ·   
暂无公开 PoC，也未见在野利用；但无需账号、无 nonce，一个请求就能调用站内任意 PHP 函数。  
  
补丁状态 ·   
已修  
，8.8 修正。插件不在官方插件库，后台不会给更新提示，只能跟主题更新或手动替换。  
  
不用登录，一个 curl 就能让 WordPress 多出一个管理员账号。8.7 及更早的 Sneeit Framework 把 PHP 的函数调度器直接挂在了公开 AJAX 接口上。  
  
## 01  事情是什么  
  
这周公开的 CVE-2026-106125 落在 WordPress 插件 Sneeit Framework 上，CVSS 9.8，类型是 CWE-94 代码生成控制不当。触发点是一个前端 AJAX 接口，处理函数挂在   
wp_ajax_nopriv_  
 钩子上。前缀 nopriv 的意思就是未登录用户也能打。  
  
代码里只做了一件事就交给了 PHP：从请求里取一个函数名，确认这个函数存在，然后把请求里的 JSON 当成参数数组喂进去执行。没有权限判断，没有 nonce，没有函数白名单。  
  
Sneeit Framework 没有走 WordPress 官方插件库，后台插件页搜不到它，一般随 Sneeit 系列主题一起打包安装。所以受害面不好统计，很多站点的管理员自己都不知道站上有这个插件。  
  
## 02  为什么危险  
  
8.7 里的核心逻辑一共十行，问题全在头两行：  
  
$callback =  
  
  sneeit_get_server_request(  
  
    'callback');  
  
if (function_exists($callback)) {  
  
  $args =  
  
    sneeit_get_server_request(  
  
      'args');  
  
  $args = json_decode(trim(  
  
    wp_unslash($args)), true);  
  
  $args['block_id'] =  
  
    'sneeit-menu-mega-content-'  
  
    . $args['item_id'];  
  
  echo call_user_func($callback,  
  
    $args);  
  
}  
  
sneeit_get_server_request() 就是先读 GET 再读 POST，等于把 PHP 的函数调度器直接暴露给互联网。唯一的门槛 function_exists() 只回答这个函数存不存在，不回答这个函数该不该被外部调用。参数同样来自请求，json_decode 之后原样作为第一个参数传进去。  
  
WordPress 内核里有大量接收数组、副作用极大的函数。最省事的一条是   
wp_insert_user()  
：它接收关联数组，字段包括 user_login、user_pass、role。传一个 role 为 administrator 的数组进去，站点上就多了一个自己知道密码的管理员，后面就是常规的登录后台、装插件、改主题。也可以换成改站点配置、写 option 这类操作，看攻击者想要什么。  
  
这个洞比一般插件漏洞更麻烦的地方在于流量长相：请求体里只有几个短字符串，没有上传文件，没有 PHP 代码特征，也没有常见注入关键字，规则拦截的麻烦程度和收益完全不成比例。  
  
8.8 的修法反过来证明了这一点。新版不再从请求里取函数名，改成读主题注册菜单时写进配置数组的   
mega_block_display_callback  
，加了 is_string() 校验，参数也收敛成 absint() 过的固定字段。攻击者能控的东西从函数名加参数全套，缩到只剩菜单项 ID。  
  
要确认自己的站点是否中招，跑这段（换成自己的域名）：  
  
U=https://SITE/wp-admin/admin-ajax.php  
  
A=sneeit_compact_menu_mega_content  
  
curl -s -X POST "$U" \  
  
  -d action="$A" \  
  
  -d callback=wp_json_encode \  
  
  -d 'args={"item_id":1,"p":"1"}'  
  
wp_json_encode 是个无害函数。命中时响应体里会出现一段把 p 原样返回的 JSON；打了 8.8 之后，请求会在菜单定位那一步被拦下，返回空。这一步只验证漏洞存不存在，不碰数据。  
  
## 03  谁受影响  
  
WordPress 站点，装了 Sneeit Framework 插件，版本 0 到 8.7 全部受影响。  
  
插件通常随 Sneeit 系列主题静默打包，后台插件列表里可能只显示为主题依赖。  
  
需要主题启用了 compact menu 的 mega 菜单，Sneeit 主题的默认配置一般都会启用。  
  
admin-ajax.php 必须可访问，这一条几乎所有站点都满足，套了 CDN 也一样。版本号看插件主文件头部的 Version 字段，或后台插件列表的版本列。  
  
## 04  怎么修  
  
1. 先查有没有：服务器上 ls wp-content/plugins 看有没有 sneeit 目录，或后台插件列表搜 Sneeit。  
  
2. 有就升到 8.8，从主题更新包里拿，或直接替换插件目录。  
  
3. 别等后台更新提示，这个插件不在官方插件库，WordPress 不会给你推更新。  
  
4. 暂时升不了：停用插件（主题菜单会降级成普通菜单，站还能开），或在 WAF 上拦 action 为 sneeit_compact_menu_mega_content 的 POST。  
  
5. 顺手排查有没有已经被打过：wp user list --role=administrator 看有没有陌生账号，访问日志里搜 sneeit_compact_menu_mega_content。  
  
第 5 步别省。这类洞从公开到被扫，通常不到一周；被打了不会在页面上留痕迹，只会多一个账号。  
  
## 05  一句判断  
  
这个洞本身不复杂，一处 call_user_func 少了一次白名单，代码不到十行。但它的利用成本几乎为零，一个请求就够，而且不需要任何前置信息。如果你站上确实有这个插件，今晚处理掉；查完发现没有，那就睡吧，它不是能跨站传播的那种问题。  
  
## 06  参考地址  
  
https://github.com/tiennguyenvan/sneeit-framework/commit/a9f822a392  
  
https://www.wordfence.com/threat-intel/vulnerabilities/id/9654f07e-e595-46d4-b093-5d2ab4389806  
  
第一个是修复提交，两版差异一眼可见；第二个是漏洞条目，含 CVSS 向量与影响版本。暂无公开 PoC。  
  
  
下期继续蹲厂商补丁和 PoC 动静，**关注**  
后漏洞先知道，修复不踩坑。  
  
— END —  
  
  
