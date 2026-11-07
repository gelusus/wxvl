#  海外SRC漏洞挖掘：\绕过403 + 接口不校验token字段签发JWT  
原创 LL.
                    LL.  狂做有氧   2026-11-07 09:36  
  
很久没有写东西了 根据平台规则 不能披露细节 见谅 说下思路    
  
利用过程如下：  
  
1. 访问 /api/login-xx 403 利用\绕过 访问 /api\login-xx 400   
  
2. 发送  '{"login":"任意邮箱","token":"任意值",...其他字段}' 返回带jwt字段的json  
  
3. 利用返回的jwt从/api\login-aaa换取登录cookie  
  
4. 利用其他的正常业务接口可以做的操作：全量PII 水平越权敏感信息读写  
  
技巧:  
  
curl的 --path-as-is 选项可以绕过路径规范化 用来测试接口比较好使  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ZibkibejKcK5wYhE9Ox3R4u89lBLTDUxK5qvtMl6penoeMGvp1Fia08KshDj9KVhmurfw4HPV9vUeNYCE0Gg83HsiaI9xdWfice3AQRD5KTrHsbc/640?wx_fmt=png&from=appmsg "")  
  
  
