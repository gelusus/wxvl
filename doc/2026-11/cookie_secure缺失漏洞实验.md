#  cookie_secure缺失漏洞实验  
原创 彭瑞
                    彭瑞  新蜂网络安全实验室   2026-11-24 12:25  
  
复现一些在漏洞扫描报告中多次出现的漏洞，以帮助开发人员理解漏洞的原理和风险，从而加快漏洞整改的速度。  
  
# 一、实验简介  
  
在kali系统中演示会话cookie不设置secure属性的危害。  
  
在  
Kali靶机的LAMP环境中，使用PHP创建一个登录页面，演示会话Cookie不设置Secure属性的危害。通过Wireshark捕获流量可以观察到：当浏览器通过HTTP协议访问该站点时，Cookie以明文形式出现在HTTP请求头中；而将同一站点配置为HTTPS并给Cookie加上Secure属性后，浏览器不会再通过HTTP发送该Cookie，Wireshark中也无法直接读取到明文会话标识。  
  
核心原理：  
Secure是浏览器层面的强制约束，禁止Cookie在非加密信道中传输。缺失该属性时，浏览器会在任何向该域名发起的HTTP请求中自动附带该Cookie，意味着：  
  
只要用户在不安全的网络环境下（如开放式  
Wi-Fi、被控路由器、同一局域网内）正常访问一次HTTP页面，攻击者无需任何诱导即可被动截获其Cookie；  
  
进一步地，如果站点同时监听  
HTTP和HTTPS，攻击者还可以主动诱导受害者打开一个指向该站点HTTP地址的链接或图片资源（即便受害者平时都用HTTPS访问），强制触发一次HTTP请求，专门用来窃取Cookie。本实验主要演示前一种（被动嗅探）场景。  
  
# 二、实验环境  
  
角色，系统  
类型，  
软件版本，  
操作系统版本，  
IP地址  
  
靶机  
，  
虚拟机，LAMP、wireshark、浏览器，kali 25  
，  
192.168.242.252  
  
# 三、实验步骤  
## （一）安装软件  
  
#   
安装并启动  
Apache + PHP  
  
sudo apt update  
  
sudo apt install -y apache2 php libapache2-mod-php  
  
sudo systemctl start apache2  
  
  
（二）缺失Secure属性的Cookie  
### 1、创建登录页面  
  
cd  
 /var/www/html/  
  
sudo vim   
secure_cookie_demo.php  
  # 加入下面的内容  
  
<?php  
  
session_start();  
  
   
  
if ($_SERVER['REQUEST_METHOD'] === 'POST') {  
  
    $user = $_POST['username'] ?? '';  
  
    // 模拟登录成功，设置会话变量  
  
    $_SESSION['user'] = $user;  
  
    // 故意不设置Secure属性，模拟漏洞场景  
  
    setcookie('session_id', session_id(), [  
  
        'path' => '/',  
  
        'httponly' => true,  
  
        // 'secure' => true,  // 缺失关键属性  
  
        'samesite' => 'Lax'  
  
    ]);  
  
    echo "<h2>登录成功</h2>";  
  
    echo "<p>用户：" . htmlspecialchars($user) . "</p>";  
  
    echo "<p>当前Cookie未设置Secure属性。</p>";  
  
    echo '<p><a href="dashboard.php">进入控制面板</a></p>';  
  
    exit;  
  
}  
  
?>  
  
<!DOCTYPE html>  
  
<html>  
  
<head><title>Secure Cookie缺失演示</title></head>  
  
<body>  
  
<h2>登录（Cookie缺失Secure属性）</h2>  
  
<form method="POST" action="">  
  
    用户名：<input type="text" name="username"><br><br>  
  
    <button type="submit">登录</button>  
  
</form>  
  
</body>  
  
</html>  
  
保存退出。  
  
这里要注意，session_start()本身就会让PHP自动下发一个名为PHPSESSID的会话Cookie，（是否带Secure取决于php.ini里的session.cookie_secure，默认是关闭的）。代码中手工使用setcookie('session_id',  
   
...)创建了第二个自定义Cookie，本文中讨论的  
secure属性是针对这个cookie的。  
  
   
  
#   
同时创建dashboard.php，用于查看Cookie是否被浏览器携带：  
  
cd /var/www/html/  
  
sudo vim dashboard.php  # 加入下面的内容  
  
<?php  
  
session_start();  
  
if (!isset($_SESSION['user'])) {  
  
    die('未登录');  
  
}  
  
echo "<h2>控制面板</h2>";  
  
echo "<p>用户：" . htmlspecialchars($_SESSION['user']) . "</p>";  
  
echo "<p>浏览器发送的Cookie：</p>";  
  
echo "<pre>" . htmlspecialchars($_COOKIE['session_id'] ?? '无') . "</pre>";  
  
保存退出。  
  
   
### 2、启动Wireshark抓包  
  
Kali靶机打开Wireshark，选择eth0，开始捕获。显示过滤器输入http.cookie，查看包含Cookie的HTTP请求。  
  
   
### 3、通过HTTP登录并访问控制面板  
  
浏览器访问：  
  
http://192.168.242.252/secure_cookie_demo.php  
  
输入用户名：testuser，点击登录。  
  
点击“进入控制面板”，查看用户和  
cookie信息：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/rRrniaqspTs129YibzU5Q6pp5456ywR0lmCwxk5OaWjEqRia4lGNILdZEHQv8M3zK7TUbw3fUeoHJiaxcq910KY3gWcFjQDW89lx9adBJtzokUU/640?wx_fmt=png&from=appmsg "")  
  
可以看到，页面输出了用户会话的  
cookie信息。  
  
   
### 4、在Wireshark中观察明文Cookie  
  
停止捕获，在Wireshark中找到发往dashboard.php的GET请求。查看  
HTTP数据流，可以看到cookie的详细信息：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/rRrniaqspTs3VibIp4qbIMd5l0Mkt0AIJ0syrsI5xeDj4tib2hLrEykoACLPl2K7ETH8nm6Hxibxp5MHCVPCp1B5FrgMib6A9QnnwaWTiciatrL4e8/640?wx_fmt=png&from=appmsg "")  
  
会话标识以完全明文的形式出现在  
HTTP请求头中。任何能够抓包的节点（同一局域网、开放式Wi-Fi、被控路由器）都可以直接读取并利用该Cookie冒充用户会话（会话固定/劫持）。  
  
   
## （三）启用HTTPS并设置Secure属性（对照）  
### 1、生成自签名证书并启用SSL  
  
# 生成自签名证书  
  
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \  
  
  -keyout /etc/ssl/private/demo.key \  
  
  -out /etc/ssl/certs/demo.crt \  
  
  -subj "/CN=192.168.242.252"  
  
# 启动SSL  
  
sudo a2enmod ssl  
  
sudo a2ensite default-ssl  
  
# 编辑apache配置文件  
  
sudo vim  
 /etc/apache2/sites-available/default-ssl.conf  
  
确保证书路径指向上述文件  
  
#   
重启Apache  
  
sudo systemctl restart apache2  
  
   
### 2、为cookie添加secure属性  
  
#   
修改PHP代码，为Cookie添加Secure属性  
  
sudo vim /var/www/html/secure_cookie_demo.php  #   
将setcookie部分修改为：  
  
setcookie('session_id', session_id(), [  
  
    'path' => '/',  
  
    'httponly' => true,  
  
    'secure' => true,        // 取消行首的注释  
  
    'samesite' => 'Lax'  
  
]);  
  
保存退出。  
  
   
### 3、浏览器操作  
  
浏览器访问  
(可能需要点击"高级"→"接受风险并继续")  
：  
  
https://192.168.242.252/secure_cookie_demo.php  
  
输入用户名：testuser，点击登录。  
  
点击“进入控制面板”，显示cookie信息：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/rRrniaqspTs0zA4VPWFDhweYQiceDu4V6n07eruTicC4umegric19KoOUibXbmjFWhZg5hgkQJJMf4W5qC144maRqq7jic89F864ibsLHS2yIZwDIw/640?wx_fmt=png&from=appmsg "")  
  
可以看到，页面正常输出了用户名和  
cookie。  
  
   
  
手动将地址栏中的  
 https:// 改为 http://，尝试访问 http://192.168.242.252/dashboard.php  
  
![](https://mmbiz.qpic.cn/mmbiz_png/rRrniaqspTs1tZNL9SW05jP8wTAFicHTk4wtYyWE0lN8Ms25mticibObTrkVM5AnUzs9wsPe8ibQcXsMa4iaq0iaMozjqHNK8YG1ktlThgDbwsWcnM/640?wx_fmt=png&from=appmsg "")  
  
可以看到，只输出了用户名，未输出用户的  
cookie信息。  
  
   
### 4、观察对照结果  
  
浏览器行为：  
  
由于Cookie带有Secure属性，浏览器不会在HTTP请求中发送它。因此在使用  
HTTP协议时，dashboard.php页面中的cookie部分显示“无”，但是在使用HTTPS协议时正常显示cookie。  
  
   
  
Wireshark抓包：  
  
过滤http.cookie，在发往dashboard.php的HTTP请求中找不到session_id的Cookie头  
:  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/rRrniaqspTs3KjkpecKEWqJgBZ76ia83uyWD0XF7KSdzjoibekK7QQsv1f5KCtoe92O3uXxEFt7rYJydRGxWPF0icYNKnAfDQVfnPIhrgNMzJPo/640?wx_fmt=png&from=appmsg "")  
  
在  
HTTPS流量中，Cookie被发送时封装在TLS加密的流量中，无法直接读取。(无截图)  
  
# 四、实验结论  
  
Secure属性阻止了敏感Cookie在明文HTTP信道中暴露。缺失该属性时，浏览器会在该域名下的任何HTTP请求中无条件附带Cookie—哪怕用户只是在不安全网络环境下进行一次最普通的访问，攻击者也无需任何诱导即可被动截获会话标识并冒充用户登录；而启用Secure属性后，即便攻击者能够进行网络嗅探，也无法从流量中直接获得可用的会话Cookie。  
  
