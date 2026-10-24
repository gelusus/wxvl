#  php2shell漏洞全解析--带来的思考  
原创 mkbksec
                    mkbksec  水刃安全   2026-10-24 09:16  
  
php2shell：PHP 底层 RCE 漏洞全解析  
  
前言  
  
为了避免面试再被拷打，深度分析了一下最近很火的  
php2shell，总结为实战不容易黑盒打出漏洞，也不存在通用poc对全网网站构成严重威胁，这个漏洞也给我们带来了思考。  
  
现在ai进化确实迅速，但是也带来了问题，很多ai小子他并不具备分辨漏洞的能力，不重视基础，过度信任ai，闹出了很多笑话，相信各大src厂商深受其害，ai可以帮我们更好的分析漏洞学习技术，但是不是我们能够完全依赖ai的理由，老实打好基础依旧重要，面试也不是一句都是ai打的就能把甲方糊弄过去。  
  
送给正在学习刚入行的师傅们一句话：希望大家在这个浮躁的时代能沉下心来认真学习挖掘漏洞的方法和原理，这样才能学的更好走的更远。  
  
ps：我不搞培训也不割韭菜，但是如果遇到实在解决不了的问题我看到了会回答，不要来问我有没有培训班了，语雀库是给朋友看的暂不接受付费进入，谢谢阅读。  
  
一、漏洞原理（源码级）  
  
1.1 漏洞在哪一行代码里  
  
漏洞根源在 ext/session/session.c  
 的 php_session_track_init()  
。它在进程级全局变量里保存了一个指向请求堆的指针，但 OOM 时 longjmp 跳过清理：  
```
static void php_session_track_init(void){    zend_string *var_name = ZSTR_INIT_LITERAL("_SESSION", 0);    zval *session_var;    /* 在全局符号表注册 $_SESSION */    session_var = zend_hash_add_new(&EG(symbol_table),                                    var_name, &EG(uninitialized_zval));    ZVAL_NEW_REF(session_var, &EG(uninitialized_zval));    Z_ADDREF_P(session_var);// ★ 漏洞在这行 ★    // 进程级全局变量 → 指向请求堆上的临时对象    PS(http_session_vars) = Z_COUNTED_P(session_var);    zend_string_release(var_name);}// 正常清理（RSHUTDOWN）://   PS(http_session_vars) = NULL;// 但 OOM 的 longjmp() 跳过 RSHUTDOWN！//   → 指针仍指向已释放内存 → 下一个请求 UAF
```  
  
1.2 OOM 怎么触发 longjmp  
  
攻击者发送超大上传耗尽 memory_limit 时，走进 Zend/zend_alloc.c  
：  
```
static ZEND_NORETURN void zend_mm_safe_error(...){    // 打印 "Allowed memory size exhausted" 后：    zend_mm_fatal_error(...);longjmp(EG(bailout), FAILURE);    // ★ 直接跳转回 php_execute_script    // ★ 当前调用栈上的所有清理代码被跳过    // ★ 包括 session RSHUTDOWN 中的指针清零}
```  
  
1.3 攻击者的完整触发序列  
```
# 请求 A（生产者）— 建立悬空指针POST / HTTP/1.1Content-Type: multipart/form-data; boundary=BOUNDARY--BOUNDARYContent-Disposition: form-data; name="PHPSESSID"\x00                    # 空ID → 新session + send_cookie=1--BOUNDARYContent-Disposition: form-data; name="PHP_SESSION_UPLOAD_PROGRESS"pwn                    # 触发 upload_progress 回调--BOUNDARYContent-Disposition: form-data; name="f"; filename="a.txt"AAAA                  # FILE_START → session_init() → 保存悬空指针--BOUNDARYContent-Disposition: form-data; name="big"BBBBBBBB...           # 12MB → OOM → longjmp → 跳过清理 ★--BOUNDARY--# 请求 B（消费者）— 覆盖悬空指针 → RCEPOST / HTTP/1.1Content-Type: multipart/form-data; boundary=BOUNDARY--BOUNDARYContent-Disposition: form-data; name="PHPSESSID"attacksessionid0001--BOUNDARYContent-Disposition: form-data; name="PHP_SESSION_UPLOAD_PROGRESS"pwn                    # 无文件字段 → END cleanup 消费悬空指针--BOUNDARYContent-Disposition: form-data; name="a0..a14"[伪造的 HashTable]     # 167字节×15个字段，pDestructor=zif_system--BOUNDARYContent-Disposition: form-data; name="f0"[伪造的 zend_reference] # 55字节，精确覆盖悬空指针--BOUNDARY--
```  
  
1.4 最终 RCE 触发链  
```
// B 请求的 END cleanup 调用链：php_session_track_init()  → zend_delete_global_variable("_SESSION")    → zend_hash_del_ind()      → 跟随 IS_INDIRECT 到悬空指针        → zval_ptr_dtor(ref)              // refcount 1→0          → zend_reference_destroy(ref)            → zval_ptr_dtor(&ref->val)   // 伪造的 IS_ARRAY_EX              → zend_array_destroy(fake_ht)                → zend_hash_destroy(fake_ht)                  → ht->pDestructor(&bucket->val)  // = zif_system！                    → zif_system(fake_execute_data) // "id > /tmp/p1"                      → system("id > /tmp/p1")        // ★ RCE
```  
  
二、利用条件（缺一不可）  
  
✅ Apache + mod_php  
  
必须用 mod_php，不是 nginx+FPM  
  
✅ session.upload_progress.enabled = On  
  
PHP 出厂默认开启，但部分面板会关闭  
  
✅ multipart 上传能发进去  
  
WAF 不能拦截  
  
✅ PHP 版本未修复  
  
低于 8.2.35 / 8.3.36 / 8.4.27 / 8.5.12  
  
✅ 有目标环境的堆布局参数  
  
最难一步——需白盒标定  
  
三、为什么实战目前打不了  
  
1. php-fpm 结构性免疫  
```
// fpm_main.c — FPM worker 的请求循环while (fcgi_accept_request(&request) >= 0) {    zend_try {        // ... 请求处理 ...        // 如果 OOM → longjmp 跳到这里    } zend_catch {// FPM 捕获了 bailout    } zend_end_try;// ★ 无论是否 OOM，shutdown 总是执行    php_request_shutdown();    // → session RSHUTDOWN 清空指针 → 无残留 → 打不了// 对比 mod_php：Apache handler 吞掉 bailout// → shutdown 未被调用 → 指针残留 → 可利用
```  
  
双版本实测（8.2.32/8.4.26）：FPM 下 OOM 要么进程退出（无残留），要么完整清理（无残留）。  
  
2. 生产环境大量关闭了 upload_progress  
```
# 宝塔面板默认 php.ini：session.upload_progress.enabled = Off# 我们实测的三个真实站点：1  → upload_progress OFF（WAF 还拦 multipart）2  → upload_progress OFF3 → upload_progress OFF + nginx/FPM
```  
  
3. WAF 默认拦截攻击流量  
```
# mod_security 默认规则即可拦截：SecRule REQUEST_BODY_LENGTH "@gt 10000000" "deny"   # 12MB bodySecRule ARGS_COUNT "@gt 500" "deny"                    # 900+ 字段# 实测第一站：所有 multipart POST → 406
```  
  
4. 堆布局参数不可通用  
```
// stage3 需要 55B fake ref 落在 R0-0x10 处// 我们 Debian 12 靶场实测（与参考作者同构建）：R0      = chunk + 0x24a0    // 悬空引用位置（恒定）55B落点  = chunk + 0x62068   // fake ref 实际落点（恒定）差值     = 0x5FBD8 (392KB)  // ★ 永远对不上 ★// 参考作者 Docker 镜像中：差值 = 0（恰好对齐）// 二进制级复刻仍差 392KB → 内核 mmap 布局不同// → 黑盒拿不到，每个环境要 gdb 白盒标定
```  
  
📊 一句话总结  
  
官方 Docker 镜像 + 未升级 = 能打  
  
其他环境 = 需白盒标定堆参数  
  
nginx+FPM / upload_progress 关 / WAF 拦 multipart = 直接放弃  
  
  
