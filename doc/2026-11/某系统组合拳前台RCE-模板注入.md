#  某系统组合拳前台RCE-模板注入  
进击安全
                    进击安全  猎洞时刻   2026-11-23 05:40  
  
**免责声明**  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/bL2iaicTYdZn6mG6TyJornrhz9JticBo3Nx4zhzUFXcggEDw1lkfzMI0KuLp7dW4dDCvbfgAKlLSX3yGmYg0gtXcw/640?wx_fmt=gif&wxfrom=5&wx_lazy=1 "")  
  
  
```
本公众号“猎洞时刻”旨在分享网络安全领域的相关知识，仅限于学习和研究之用。本公众号并不鼓励或支持任何非法活动。
本公众号中提供的所有内容都是基于作者的经验和知识，并仅代表作者个人的观点和意见。这些观点和意见仅供参考，不构成任何形式的承诺或保证。
本公众号不对任何人因使用或依赖本公众号提供的信息、工具或技术所造成的任何损失或伤害负责。
本公众号提供的技术和工具仅限于学习和研究之用，不得用于非法活动。任何非法活动均与本公众号的立场和政策相违背，并将依法承担法律责任。
本公众号不对使用本公众号提供的工具和技术所造成的任何直接或间接损失负责。使用者必须自行承担使用风险，同时对自己的行为负全部责任。
本公众号保留随时修改或补充免责声明的权利，而不需事先通知。
```  
  
  
内容来源：进击安全，[学院投稿-某系统组合拳前台RCE-模板注入](https://mp.weixin.qq.com/s?__biz=MzkyMjM5NDM3NQ==&mid=2247487128&idx=1&sn=4837043d646b0f83a22d62a23fc2138f&scene=21#wechat_redirect)  
  
  
****  
  
一、前言  
  
    这里为组合拳，使用Shiro鉴权，并且进行Shiro的权限绕过+后台SSTI模板注入组合拳，进行注入内存马。  
  
  
二、漏洞分析  
  
首先漏洞成因是因为下面这段代码：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZRKuxIKRyhWJgnMZIeBoyakGcolDVkjQ8piaqoVa98SPzalZhkTyCtUQj5SC4nD7kwFmibV8Q2OtI4K9Gs1AUdDw/640?wx_fmt=png&from=appmsg "")  
  
evaluate方法source字符串进行模版渲染，且source可控，造成模版注入；  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZRKuxIKRyhWJgnMZIeBoyakGcolDVkjQP8JYWPpMLYibficDejgzbFER1SXoAGgGXe7t3uKib8axiaeS6MNhSN4KmQ/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZRKuxIKRyhWJgnMZIeBoyakGcolDVkjQPoeUb4NDw8CfQ80MMibibFh3zZ0M5Mf0af1b35Bfiap9IIg6sEUdXTPtw/640?wx_fmt=png&from=appmsg "")  
  
    首先需要重点关注message是否可控，实际上这个方法只有两个参数，taskId和vars，也就是说这个方法并不接受message参数，但是不着急，看看message是怎么生成的。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZRKuxIKRyhWJgnMZIeBoyakGcolDVkjQ6TuvTElfa89pS2NAwZneCHCdhqMMIH0ICdS6utR9H9kWq1AOednGCw/640?wx_fmt=png&from=appmsg "")  
  
实际上是因为这个循环取出了一个activiti，然后获取activiti的message，那么实际上也是可控的，无非就是触发的时候需要多调用一个方法。  
  
这段代码的大致意思是:  
- 获取当前任务；  
  
- 记录当前流程定义下任务完成前的任务列表；  
  
- 完成该任务；  
  
- 进行任务完成后的自定义逻辑；  
  
- 更新任务指派；  
  
- 获取完成任务后的任务列表；  
  
- 找出新出现的任务；  
  
- 对每个新任务发送待办提醒,通过 VelocityUtils.RenderData  
 渲染消息模板。  
  
看起来有点复杂，但是和业务流程挂钩，首先要理解什么是流程和任务，以及几个循环的条件，理解后总结一下触发条件:  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZRKuxIKRyhWJgnMZIeBoyakGcolDVkjQrU2lyG2krCFO9y3DzBhxvCejM3vaLIuf9mlZVlSPRc2DxbKgUblicIA/640?wx_fmt=png&from=appmsg "")  
  
也没那么复杂，一句话就可以总结了:该流程定义下任务完成后出现了新的任务;  
  
那么我们知道了触发条件，先不着急控制message，先一步一步跟进到controller了层，因为这一分支只需要触发complete方法，所以不必看的很仔细，直接给出方法调用链。  
  
可以看到有两条，为了方便分析，我选取一条跟进LeaveServiceImpl[#update]()  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZRKuxIKRyhWJgnMZIeBoyakGcolDVkjQYC5jhcSfCVRqSPhCdRDMcOMgYNfG4KYFQMicUHFpBjRm2EEGfEbYBxg/640?wx_fmt=png&from=appmsg "")  
  
继续往上直接跟进到了controller层:  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZRKuxIKRyhWJgnMZIeBoyakGcolDVkjQATHiahKpAX4Tiasia1ADIVT2VlgOksXgiapOhYAJ0tmKaiboicmokI8viaHsA/640?wx_fmt=png&from=appmsg "")  
  
那么整个目前为止，可以触发危险方法的整个调用链是:  
```
LeaveController#update
  >>LeaveServiceImpl#update
    >>ActTaskService#complete
      >>VelocityUtils#RenderData
        >>VelocityEngine#evaluate
```  
  
调用链是走完了，但是还有一个message的控制问题，哪里可以控制它呢？看下面的代码:  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZRKuxIKRyhWJgnMZIeBoyakGcolDVkjQ9HLZa3OJPPxqM5necPuflu0kAnic4vnichib8qBTMt4TxHjokGxHiaNCpg/640?wx_fmt=png&from=appmsg "")  
  
这个方法实际上会接受一个activities,那么自然也就可控了，message只是这个对象的一个属性罢了，表示待办消息。  
  
至此所有条件满足，可以复现漏洞了。  
  
  
三、漏洞复现  
### SSTI注入  
  
直接在界面中操作，看数据包和刚才分析的代码就行，不赘述。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZRKuxIKRyhWJgnMZIeBoyakGcolDVkjQhJwTWuP7LOYUDIVlBfFZjjO9QCiagKuzj6dls44zyXJsQnkMxG6NSfg/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZRKuxIKRyhWJgnMZIeBoyakGcolDVkjQn7xjb5A5noqJRIMfynqS6r6ibDA6hhujvZ85AvzhNdFyTtAKLp8wAtw/640?wx_fmt=png&from=appmsg "")  
  
payload如下：  
```
#set($a='')$a.getClass().forName('java.lang.Runtime').getRuntime().exec('calc')
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZRKuxIKRyhWJgnMZIeBoyakGcolDVkjQQ0puiaZcxPfDUxP4WH3Nthmia2c9pxTJ5CO6T0ZzuaicYxRAPIDRKwMSg/640?wx_fmt=png&from=appmsg "")  
  
和预期一致，edit方法指定id和message就行。  
  
请个假，三天以上，至于为什么需要看业务代码，不重要，看看数据包  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZRKuxIKRyhWJgnMZIeBoyakGcolDVkjQibyU9jvUswSTwuXDhS7TCxLeibKu8CtvY3ollTjdXsiaRFrfkVp4vBicgA/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZRKuxIKRyhWJgnMZIeBoyakGcolDVkjQbNtkQId1FpNwp5xVniaH5JzR4FzQVHNfTouve3icWTLt8jaCm3YsCq6g/640?wx_fmt=png&from=appmsg "")  
  
这个方法没有分析，但是就很简单嘛，在请假流程下创建一个审批任务而已，现在为止就有了update方法的触发条件了，看看效果:  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZRKuxIKRyhWJgnMZIeBoyakGcolDVkjQ4iaw5PkcInZI2NLWLZmF0RZX79MfyAxNDBtok79oiblzMib1p0IstKQ6Q/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZRKuxIKRyhWJgnMZIeBoyakGcolDVkjQvX20Feo6XJriaHTCT9vvKvC4NXt0icgvFQMbw9IvQ7fOMBY4rvVeB2hg/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZRKuxIKRyhWJgnMZIeBoyakGcolDVkjQVqWMnWtebfNibiciabwdEfl7aIoEMD5JWeYvkPoDqM4RG6T5CicmGg6l7w/640?wx_fmt=png&from=appmsg "")  
  
数据包是update方法，和预期完全一致:  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZRKuxIKRyhWJgnMZIeBoyakGcolDVkjQOVjlmZBXeDzyuP6CQTjFpEtdnjrDmUicBTictX5qoblic11Ydnnay9trw/640?wx_fmt=png&from=appmsg "")  
  
四、鉴权绕过  
  
这里是存在shiro的低版本，使用/../可以直接绕过。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZRKuxIKRyhWJgnMZIeBoyakGcolDVkjQn5icmbg613dlfCIb7Aej4sIujFPgxiaRMAXt9pvInZ9yDtJicyktXJJvA/640?wx_fmt=png&from=appmsg "")  
  
鉴权信息如下：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZRKuxIKRyhWJgnMZIeBoyakGcolDVkjQ4kONAKZIddgq49aO5bzyoBxOFbofyAicLHt1ALqdhyrUYLtwubUte6A/640?wx_fmt=png&from=appmsg "")  
  
这里随便找一个后台路由，不带Cookie进行访问。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZRKuxIKRyhWJgnMZIeBoyakGcolDVkjQDA8dDLBQnzNDxJlqNcYQiajhrtEmicVsdfIUln3p3pvGYich6tN8NUnnw/640?wx_fmt=png&from=appmsg "")  
  
可以看到会进行跳转到登录页面，低版本Shiro可以使用../进行绕过。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZRKuxIKRyhWJgnMZIeBoyakGcolDVkjQUNW9CuHiaDvusSIw7PCA6CxRYM5BxpIibxKgOWauWXIt2eQjticndMgCw/640?wx_fmt=png&from=appmsg "")  
  
这里也成功能够访问到，成功拿下，剩下的使用jmg直接生成内存马注入即可。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZRKuxIKRyhWJgnMZIeBoyakGcolDVkjQY6W3JHe77iaANbxwqtI0BoAibcr8mBOK3nW1rFfxs23ibQ1iaRFibdPJcKA/640?wx_fmt=png&from=appmsg "")  
  
****  
网安挖洞交流、网安考证，学习欢迎加群！  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ucA3HFqtup77AHicZ4DWumh9DjV8cm3yOCCcGTD8A7VvVzHMzMISOn3TjbcVKsRBSGBK4glgRbDIX0EjpDotAfGuKkZyAwPXhsibFgIaHlTUs/640?wx_fmt=png&from=appmsg "")  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/doerbbgECwpeFnACfhfnjV0KRib0wx8dSYibBibXfvnlSJKZjVwqDkT4p8w3YR3kE9V23USEQoTLWyuxElcVuDohMJ7XibHRYDiaKjomSfhbOx7A/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/PRjdmUELtVDZsOJib3BlegVyicZrfRqrkUKQ2sIP9mwxpzUymkzJrv7ZbXh9rJdWGeAJdlkFtQGCBgd5XmaqJdMEzusgnCkyPeGYrn75kcKc0/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/G0iaFtEibWU95PS1s38UE9icpFGHNBvGE8hBeZwkn4LbnvYJfIezdAS9Xqibfaot7vGRRkl21iaBHcKQCxzWCSm3nLpMIGlqCib7xqnZwVrGERtek/640?from=appmsg "")  
  
挖洞、赏金和入职实战班  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/IpAzibH6mLiaXTmWkTA7SecLb1jwcPkGLUTxdh3RrsOTpfbeDAhJDpYx84YbCPRO1jVVbTez7icvxTkJeCPTYtkQ6dLVxded0jzsl2bGy417cA/640?from=appmsg "")  
  
NEW JOURNEY IN TECHNOLOGY  
  
AI赋能挖洞培训  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/n4yj6uk7icAClBAquKgcqEdUm6z5bsKzGicqXtoX1Om6N3yP4QmZal5V9M19efUWMunG6pdHiaUfVzmTmGoFfn3PgDvic3d8HyDTSuqLMpjgAlk/640?from=appmsg "")  
  
  
网安新人应该做什么？  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/ne7st7osWQv1cP0Rq7A6Kc0LuuWKCz4UWxGAAlcibvibYSPOEbrMmjCSzYAgtHlBoEuEpBwOczupW4VKQOgphRoQCmPdjKpIfallIqhgQ4SwE/640?from=appmsg "")  
  
  
  
 经常有新人问我，我明明自学了那么多网安技术，B站也学了很多，一两年了为什么迟迟在实战无法上手？只能去打靶场？  
 那是因为你没经过系统化实战训练，你学的技术全是碎片化的，今天学Web漏洞，明天学python，后天被抖音的人忽悠学Kali，今天靶场明天又实战，实战挖洞还不会，硬要学内网渗透，  
最后技术存在断崖鸿沟，  
这样学一辈子难以入门，更有垃圾网安培训，全程只有靶场，根本就没有带学员去实战挖洞培训。  
  
  
其次还有些新人，  
大三大四了还不会实战挖洞，只会点DVWA靶场，完全走的是学校老师的路线，太慢了！很多大学老师到大三才开始教学漏洞原理和靶场，而你的同龄人已经早已去大厂实习了，所以说别幻想学校老师能教会你什么。  
是不是温水青蛙，自己学的水不水，你自己清楚，别再欺骗自己了！  
  
  
只有真正的实战挖洞能力  
，永远都是你在网安这个行业的  
核心竞争力  
！挖洞足够强，无论是就业大厂，还是实现SRC赏金，都是水到渠成！这些远比其他的更重要！  
  
  
如果有挖洞入职培训咨询、网安考证、扩列等，欢迎加我。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ucA3HFqtup5CibqjOAgcKHNZ1YnlPjWTibMUQunqC46C4icblvvlKhPyWwhPiba6ibW5ZUG4HWSQiccN3Ka9HhtwrMzt5WrAoAcCwfYb42MkiaojCk/640?wx_fmt=png&from=appmsg "")  
  
猎洞时刻团队介绍  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/ne7st7osWQv1cP0Rq7A6Kc0LuuWKCz4UWxGAAlcibvibYSPOEbrMmjCSzYAgtHlBoEuEpBwOczupW4VKQOgphRoQCmPdjKpIfallIqhgQ4SwE/640?from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/UKNVa82CwibfF1ovv4q3fnQ8sib70fohUsibxRVGetvW2trw171icE2PACxlWpIbLOATzduVTL40KNS6GEalG5tqDxdoibe57mYApibHRb4j8iaSyc/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/XNw8Gs7pfmhLibl5aWHeBYCtCO74HXfmFDj5PcJUHfBG4To3m8gGZG1dUFIM5hTU24zSeuQvpZjicc29nJfkuIrxUukkyLBfLpribX42eTqAzY/640?from=appmsg "")  
  
我们团队自从2023年成立以来，历经三年发展，已经在安全圈内获取超多学员的加入，并且经过培训和一对一指点解答后，成员已经遍布网络安全大厂，如  
长亭科技、奇安信、绿盟科技、安恒信息、360、深信服科技、启明星辰、亚信、微步等多个安全厂商，现在经常在安全厂商，同一个办公室、区域，可能就有我们多个学员，也非常感谢各位师傅们的支持，往后的课程，内容和质量只增不减！  
  
  
团队学员对于一些SRC平台，如EDUSRC平台，  
2026年斩获团队榜单第二、2026个人榜第一，  
在2025斩获团队榜单第四、成员个人榜单第一，诞生多个千分Rank和证书大满贯的师傅。  
  
  
在企业SRC方面，网易SRC2026年榜第一、蚂蚁集团SRC2026年榜第五、看云SRC2026年榜第二、爱奇艺SRC2026年榜第四、麦当劳SRC2026年榜第三、国通星驿2026SRC年榜第一，等等其他多个SRC年榜前十，  
还有CNVD单人获取证书量摆不下展台，并且在AI时代的来临，我们也是同样推出AI自动化挖洞等AI系列课程  
。  
  
  
我们的上面介绍的挖洞排名下面文章有真实图文介绍：  
  
[猎洞时刻SRC挖洞&入职培训｜近期成绩 （文末抽奖）](https://mp.weixin.qq.com/s?__biz=MzkyNTUyNTE5OA==&mid=2247490991&idx=1&sn=478c4ec5cd1d512c79492d10a93addab&scene=21#wechat_redirect)  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/EL1yQYzv6RpXc24k1JYGxLA89trBry5yYP0biaEd6ljbwPgnAashzKG8JA2gRNyuE3DklaXvSPmres8ocAM3wSSpHzfRL6X5HwUJX4pjbehc/640?from=appmsg "")  
  
PART.01  
  
  
网安大厂真的很难进入吗？  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ZDHiayEzyt870FWQg0ZdaNuq3nMgnZrASHYqznaMKcVD7cE4F4iccJM35sWZytlKVe6DV0gOmS2caibg3mUMmc8cxErpFvXmLf9W99udVBd9Uk/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/kgEbbowanP9gZiavdq67qv1UhejqWMNHRyhmdoZsiat0Ow1peiaJ5hhlK8efgFnWnlKtJ0GgE80kyexnEQiawzZ4Gia6dsm9t7GBGbLurmtloZiaU/640?from=appmsg "")  
  
    我们团队学员，进入网安大厂的非常之多，并且我们在这方面也很有培训入职经验，本质上就业安全大厂并不难，原因如下：  
  
  
比如：网安大厂卡学历吗？  
  
    网安大厂，只是在网安行业是大厂，放眼全国，实际上也只是规模上千人或者勉强上万人的企业，  
是没法和互联网大厂企业的规模比拟的，  
所以说，他并没有那么高的难度，并且也不怎么卡学历  
，很多都是双非本科可以随便去，剩下主要看技术能力！当然，专科的也可以，比如长亭，不卡专科，只看技术。  
  
     那么剩下的就是技术能力的要求，那么现代网安大厂，需要什么技术能力？  
  
  
简单就是下面几个要求：  
  
1、对于常规实战挖洞必须非常熟练，至少Web、小程序和APP你都能渗透测试，还有云安全相关渗透经验。  
  
2、其次具有丰富的项目经验，那么项目经验从哪来的？也就是你平时参与一些护网项目、渗透项目，还有实习时候的项目都可以。  
  
3、掌握学习前沿技术的能力，比如目前的AI非常火爆，现在很多安全企业，在面试时候，会把AI能力纳入面试要求。  
  
4、这个就是拔尖要求了，你如果没有，也不影响你就业，你如果会，那么就是加分项，比如过硬的代码审计、Java安全、内网渗透、CTF之类的加分项。  
  
  
前面三个算是必须项，  
第四个算是加分项  
，所以从此可以看出来，都是围绕着实战挖洞进行的，你只有掌握实战挖洞，才能做到去做项目、去实习和就业。  
  
  
如果你  
只会打靶场  
的水平，那么在我看来，你和零基础没有多少区别，远远达不到就业和挖赏金的要求，  
这种完全零基础和靶场水平的差距，花一个月系统化培训就能抹平。  
  
  
       我们猎洞网安主要做实战方面的培训，近两年，在我们猎洞团队内出来的，  
入职这几个安全大厂，比如长亭科技、奇安信、360、绿盟科技、安恒信息、亚信安全、启明星辰等安全公司，起码也有上百个。  
几乎每个安全大厂，每个地区分公司，每个安全技术部门，都有好几个学员是来自我们这里。  
  
其次还有入职字节跳动、腾讯、智谱、快手、希音、bilibili、联想等互联网大厂的安全部门！  
  
  
口说无凭，证据呢？如下，可以  
左右  
翻转看一看，这也只是一部分成绩  
  
（这个动图显示不全，只截取了三分之一，实际上更多offer）：  
  
  
  
《--可以左右滑动看猎洞学员offer--》  
  
  
←左右滑动查看更多→  
实际offer比这个多的多。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Ivib6bxzklFLGzk9icoQ1lmhSwMbj4KY9dwia7B0vkqicjnGEpJExgbFE9wneficBEGSHS6tA1gDJfhDiaoTr8cRfuFczl06pcaBicoXSwzBMGqEsU/640?from=appmsg "")  
  
  
  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/sZNhSEe8By4vxLsNWQQzRFqCouibouukVI8ibChbXNwlxM08j2icZy0jUrewYicD68tBibgW1N9U8Nl3ZV1jslabgxQpIx3micLhRWLAhOaQg76EY/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/CGSu2ia7A7PiaFQHzywGD1oibcIHefVOOqHng7I038nfNNUHSVv3Eyibe6xdo0PX1a75vMWpdzz1q4NJCbDXgVwibdibU8fArDfLmnZQfTl7ruHaE/640?from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/sZNhSEe8By4XzViaeg75icD56Zo6C2OO77OKibjbpWDgGImcU2zQZodoXt18qHBQqDnLjZ08HO4UrVUnN66rIs5ajvLmJQYU6ricK2cBgHHODfs/640?from=appmsg "")  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/EL1yQYzv6RpXc24k1JYGxLA89trBry5yYP0biaEd6ljbwPgnAashzKG8JA2gRNyuE3DklaXvSPmres8ocAM3wSSpHzfRL6X5HwUJX4pjbehc/640?from=appmsg "")  
  
PART.02  
  
  
猎洞培训有没有学员成绩介绍？  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ZDHiayEzyt870FWQg0ZdaNuq3nMgnZrASHYqznaMKcVD7cE4F4iccJM35sWZytlKVe6DV0gOmS2caibg3mUMmc8cxErpFvXmLf9W99udVBd9Uk/640?from=appmsg "")  
  
01  
  
EDUSRC&CNVD&企业SRC  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/9swdDnKgkCYWGrQV3g2v7D8QiaiaS4z5UF8h1x9licWXYD6WIaJ3IzUZkiaQPaPia86ibI2dxAAiajD7AvOEYNgYA7Tc8qNQnwZh0aC9ibajhjccJiaw/640?from=appmsg "")  
  
  
下面是我们团队成员的一些荣誉总结，欢迎随时查询，我们只做真材实料的培训，助力你完成自己的就业网安大厂目标 or 副业赏金目标！  
  
（2026年未截止，部分排名会有上下微小浮动）  
  
  
EDUSRC 2026年榜个人榜单第一  
  
EDUSRC 2026年榜团队榜第二  
  
EDUSRC 2026年榜个人榜单第五  
  
EDUSRC 2026年榜个人榜单第九  
  
EDUSRC 2026年榜个人榜单第十  
  
EDUSRC 2025年榜个人榜单第一  
  
EDUSRC 2025年榜团队榜第四  
  
EDUSRC 全平台团队榜第三  
  
EDUSRC 全平台个人榜第四  
  
EDUSRC 常态化演习个人榜单第四  
  
蚂蚁集团SRC 2026年榜第五  
  
国通星驿SRC 2026年榜第一  
  
国通星驿SRC 2026总榜第三  
  
网易SRC 2026年榜第一  
  
理想汽车SRC2026年榜第五  
  
理想汽车SRC2026年榜第九  
  
看云SRC 2026年榜第二  
  
法大大SRC 2026年榜第二  
  
补天-北森云SRC 2026年榜第二  
  
补天-人教社SRC 2026年榜第四  
  
微博SRC2026年榜第八  
  
微博SRC2026年榜第十  
  
UCloud-SRC 2026年榜第三  
  
UCloud-SRC 2026年榜第四  
  
新浪SRC 2026年榜第七  
  
新浪SRC 总榜第九  
  
麦当劳SRC 2026年榜第三  
  
麦当劳SRC 2026年榜第八  
  
爱奇艺SRC 2026年榜第四  
  
爱奇艺SRC 2026年版第十  
  
喜马拉雅SRC 2026年榜第五  
  
喜马拉雅SRC 2026年榜第十  
  
Soul-SRC 2026年榜第七  
  
途虎安全SRC 2026年榜第五  
  
途虎安全SRC 2026年榜第八  
  
途虎安全SRC 2026年榜第十  
  
知识星球SRC 2026年榜第四  
  
敦煌网SRC 2026年榜第三  
  
合合安全SRC 2026年榜第九  
  
新东方SRC 2026年榜第九  
  
哈罗出行SRC 2025年榜第八  
  
唯品会SRC 2025年榜第六  
  
NCC国家网络空间安全云社区-产能榜第六  
  
  
更多成绩具体详情图可以看这个：  
  
[猎洞时刻SRC挖洞&入职培训｜近期成绩 （文末抽奖）](https://mp.weixin.qq.com/s?__biz=MzkyNTUyNTE5OA==&mid=2247490991&idx=1&sn=478c4ec5cd1d512c79492d10a93addab&scene=21#wechat_redirect)  
  
  
EDUSRC 2026年榜个人榜单第一  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ucA3HFqtup5Zic3WGtdet82utDqZSrrXHegX468foYlggJGCKLichDnicNUrpm3q4IC5pEgVYNs1ibUDFKjKGe9NoEzGVUOKuJNqDMfpmH1ibWKQ/640?wx_fmt=png&from=appmsg "")  
  
EDUSRC 2026年榜团队榜第二  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ucA3HFqtup6eibIPXe3JQZiaTurXpfiaCuzt2sicibI2B57WnSEmQJrTprsOXiaPf2Sz5qjdNGoEydW5X0mRrrfgyW0TGY6pVP2HibbicATO7337Vuk/640?wx_fmt=png&from=appmsg "")  
  
![bd060519f29a48600791b3595e15feb1.png](https://mmbiz.qpic.cn/sz_mmbiz_png/ucA3HFqtup48D2EaTWe2ebroALchoGXZpzlfDbibCMPv0L3JklhgEDlYA2qN30QkfNhQ4l58QvdMkh0YVYIcjLYKHWgbsx7lF6jdibflz0xvA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=41 "")  
  
2026EDUSRC个人榜，前十名，有四个是来自我们猎洞团队。  
  
EDUSRC 2026年榜个人榜单第一  
  
EDUSRC 2026年榜个人榜单第五  
  
EDUSRC 2026年榜个人榜单第九  
  
EDUSRC 2026年榜个人榜单第十  
  
![image.png](https://mmbiz.qpic.cn/mmbiz_png/ucA3HFqtup7qBY6CbicXsZInFTLdb5WDb7I354kt0O9js4VhlH6vD5yF2akZK3sYA7ZLGR2OVSqEUJ1bqJuQO1cY6KxUwhMECdSkBH6OHIqU/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=44 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/ucA3HFqtup4d1JocvJ0CzzRUWib7AJ2R0LgxZ9U4Oia0VrJJwaIiaibhemcBMSic2EFNcPuG2XrkX28klT62MM8MvWwhCCuVonXkm5Z6z6mU0ibVk/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/ucA3HFqtup5XbqKx8NX4ZibM6gKl5EzQvBzdnmdOtxByfvdiciagTianlEJYvbicR1exgG1uG3icxwP6wbUN039lullLTDTCXlG0v2GpX7aafq9Ao/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ucA3HFqtup6icBib3ffKm9LtLQlfvlXMMBgLFDkTy2VT4qH8prPibe1XSXrsLR7RRkibortZToBMdyiawPnrZv2LzDZqibcXMaDMa1DiaXAUjq9VAs/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ucA3HFqtup7Bd0T4T32s6bSOjw7qgQJYC6oUgFFWRZWibfJV9yHKdJQNS3UE5Et0WlBEyLDwQFPQaqwhX6Cc5dQYEt9l3Wg3D7eaciaiaDCFN4/640?wx_fmt=png&from=appmsg "")  
  
蚂蚁集团SRC2026年榜第五  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ucA3HFqtup7Bft6cmeKIOEN4HQoZZEDCJqEuGRLiawwGfQYAjmm12tf84L5whPhD9e906ibKic2zbwV6cwLj4KcqhtKhhttbLwAG9WAnibOz2o0/640?wx_fmt=png&from=appmsg "")  
  
网易SRC2026年榜第一  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ucA3HFqtup4adCGZIfSSc4J3ojA8CC5oV786lGIzgjeoJiaHWmtJhNluwmv4bA3WP56t2C5eo4SdT4ZUSMhl9dsFhAtaVPUntSYtfpD2Zarw/640?wx_fmt=png&from=appmsg "")  
  
看云SRC2026年榜第二，单个漏洞破万元赏金。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ucA3HFqtup5Xo8ib6HYSGyTdoWI5QWribvCiaibicfW5XpYamb1JcUB20b2H6f8uaGNGDr9uVdGIKJCcOzn2OqESVJ0OUzmf8ysu8VOLGvEP7mXU/640?wx_fmt=png&from=appmsg "")  
  
法大大SRC2026年榜第二  
  
![image.png](https://mmbiz.qpic.cn/sz_mmbiz_png/ucA3HFqtup7W2Tzo3KucPILic8zuDYPSPYgEhnc108ibaaesAhEo5FyfCGJOmP9LUw8ibdRYAU7Yjt0SUEQ0aUYWCqSS9Bl8sKNrg8sicFSplEA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=33 "")  
  
麦当劳SRC2026年榜第三  
  
麦当劳SRC2026年榜第八  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ucA3HFqtup7ibjmnCVbaj5SGOt0Sgh3Gkwicb2K5aFMX8EO6eHApWkwic5WCvQM0uaaW8d9nciaZWqMWjnmKUJX1E1zEGoQbqs3ZYc4I38E1ibo0/640?wx_fmt=png&from=appmsg "")  
  
新浪SRC 2026年榜第七  
  
新浪SRC 总榜第九  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ucA3HFqtup4tDFpIW6ic3hL9hruucYAgzePdPyF4GtH2AZbKibB1dWlKQJuId83td2ymK8hsTSmvPlFia1GhSsZ5P2hAGTicM1P03mXyTljNkib8/640?wx_fmt=png&from=appmsg "")  
  
爱奇艺SRC 2026年榜第四  
  
![image.png](https://mmbiz.qpic.cn/sz_mmbiz_png/ucA3HFqtup78JgXcwIUicMWdibib9OGobh3m0maWibLgNiasV5g5EOqgaM0ibvZCWpibVZZZN45jt5LGCTvuTSCO9vib3rpj2licE2WialibDcxyTlJ2ak/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=17 "")  
  
微博SRC2026年榜第八  
  
微博SRC2026年榜第十  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/ucA3HFqtup4r7tbudv63WOga9f8d92uqTrG3emNFFNHxuRGcn5GIN2OtV4GicXANweR2JjniaTMdRc1d0iazEOlaGqJWIiaauiclZu5DX3EStqZI/640?wx_fmt=jpeg&from=appmsg "")  
  
Soul SRC 2026年榜第七  
  
![image.png](https://mmbiz.qpic.cn/sz_mmbiz_png/ucA3HFqtup4vnPoJsuU36xiazl4sdZwSgbYLibGe0spzv0TFl8eYySkg1SDTAdZib0nK7klGPC8JrkxQ5h4AGfsGZm8mzTibibHYBMwO1QBHgPv0/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=24 "")  
  
Ucloud SRC 2026年榜第四和第五  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ucA3HFqtup6LZbzCwhAZRkicBcvH4bYl5VHfMRV3MGibmvhYyDhLRuIGqo6CaiaPJ7DSs43rRIROvOzK8ZyAAANqT3qPQ9OkhRPIn4LKhTujjo/640?wx_fmt=png&from=appmsg "")  
  
唯品会SRC2025年榜第六  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ucA3HFqtup4QCZOc0RicfKdBT0PkmB9VCPmv2wibSo24Dj9W6NcBWRhic1Cs15qAOxbEjCTENfMM5G8VGAazeQ2qQlb2MTNadlMBRCJozhpPqg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=23 "")  
  
理想汽车SRC 2026年榜第五  
  
理想汽车SRC 2026年榜第九  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ucA3HFqtup6CGXklysiaOMZ1LTICck8bmCibk7ER7aNFX1dZlNX5ib4tgIR4Hfj6YWlCegu4dYLmviaLfj2ib41evicQAp85M9Umcia03addqmlP80/640?wx_fmt=png&from=appmsg "")  
  
还有其他多个企业SRC年榜前十，就不一一列举了。  
  
以下众多学员漏洞赏金，单个漏洞赏金破万等。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ucA3HFqtup6QNm3BZZM7ZVqzsiaNlK7jvW6lBib5pqeMgTAGZRAeHZtrHn8Dexu2V4iabKI5uV6WHppAPhVDRzCqRNNticiaPIXrIVbp3AqCTCibw/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ucA3HFqtup7aRZWFjzoMNPCv3jM4C56bzAPIbAKAOdDHKHTp7LLN4fjaj83BK98qIL5dtpqeJjfrDibntAxJ1sVs5WprMmT5RByPVwqA42jg/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ucA3HFqtup5gx0vRDEtNugy3I6pv7hMrvfLHficNWPCdjkEz4OyCibdLDjDjn8KKYqYtgS2GfpbheurvVrfjkt2mpTBPyqrAyiclic8icrTeLxqA/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ucA3HFqtup6Xfw1tqotTNhVtezd2Dhd6WwQLqUGCtMFKkdtR9nEA78IeMu6qZdfzpHTI4ziao3ap3szks0S8O9l3cfsDWt3pDje6OicnJvk0I/640?wx_fmt=png&from=appmsg "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/d6JIQYCSTHico4s7YfpmcHmWDlhInfXQ3onZicYXP4Uj0ouTBT5XjibfTpA5kiaZzewcDnlKhicLxy12Oa2lm7jhU0g/640?wx_fmt=other&from=appmsg&randomid=vkc2323j&tp=webp&wxfrom=5&wx_lazy=1 "")  
  
更多成绩内容欢迎加我了解！  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/sZNhSEe8By4XzViaeg75icD56Zo6C2OO77OKibjbpWDgGImcU2zQZodoXt18qHBQqDnLjZ08HO4UrVUnN66rIs5ajvLmJQYU6ricK2cBgHHODfs/640?from=appmsg "")  
  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/EL1yQYzv6RpXc24k1JYGxLA89trBry5yYP0biaEd6ljbwPgnAashzKG8JA2gRNyuE3DklaXvSPmres8ocAM3wSSpHzfRL6X5HwUJX4pjbehc/640?from=appmsg "")  
  
PART.03  
  
  
AI自动化渗透挖洞  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ZDHiayEzyt870FWQg0ZdaNuq3nMgnZrASHYqznaMKcVD7cE4F4iccJM35sWZytlKVe6DV0gOmS2caibg3mUMmc8cxErpFvXmLf9W99udVBd9Uk/640?from=appmsg "")  
  
01  
  
培训内容是否包含AI自动化渗透和AI安全相关内容？  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/9swdDnKgkCYWGrQV3g2v7D8QiaiaS4z5UF8h1x9licWXYD6WIaJ3IzUZkiaQPaPia86ibI2dxAAiajD7AvOEYNgYA7Tc8qNQnwZh0aC9ibajhjccJiaw/640?from=appmsg "")  
  
  
关于AI自动化挖洞，这方面我们课程也是当然包含的！我们目前猎洞培训课程第四期，一直都是持续更新与时俱进的！目前AI如此火热，我们当然也会开展相关的能力提升，祝学员们也能掌握AI能力，赋能挖洞和就业！  
  
  
有想要了解的也可以看看下面的文章哦，关于AI挖洞方面的。点击下方链接即可跳转！  
  
  
[AI挖洞目前已经是大势所趋](https://mp.weixin.qq.com/s?__biz=MzkyNTUyNTE5OA==&mid=2247490564&idx=1&sn=fe84fc6c251f6037fa6f67003f149055&scene=21#wechat_redirect)  
  
  
  
也有学员通过AI挖洞，拿到单洞6000元赏金。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ucA3HFqtup5iceh6K7FNaQJnSUv0hN2xVxQUIiaJF5VYgKbdn2uNicOdyzJUMAynJlVgyYU7icgoeYLhXZOI95BuKiaA3KTpZfczYlyByRgMCcwM/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/sZNhSEe8By4XzViaeg75icD56Zo6C2OO77OKibjbpWDgGImcU2zQZodoXt18qHBQqDnLjZ08HO4UrVUnN66rIs5ajvLmJQYU6ricK2cBgHHODfs/640?from=appmsg "")  
  
  
  
  
猎洞第四期挖洞入职培训介绍  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ZDHiayEzyt870FWQg0ZdaNuq3nMgnZrASHYqznaMKcVD7cE4F4iccJM35sWZytlKVe6DV0gOmS2caibg3mUMmc8cxErpFvXmLf9W99udVBd9Uk/640?from=appmsg "")  
  
01  
  
如何联系？价格怎么样？福利有哪些？  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/9swdDnKgkCYWGrQV3g2v7D8QiaiaS4z5UF8h1x9licWXYD6WIaJ3IzUZkiaQPaPia86ibI2dxAAiajD7AvOEYNgYA7Tc8qNQnwZh0aC9ibajhjccJiaw/640?from=appmsg "")  
  
  
1、我们目前第四期培训的价格为 1888¥ ，第五期价格预估涨价到两千多。  
  
2、一次报名可以永久每期学习，报名后，之前的1～4期，包含以后的第五期、第六期等都能永久学习，不会二次收费。  
  
3、报名包含一对一解答，包含网安的挖洞实战问题、技术问题、大厂入职规划等问题。  
  
4、报名培训，包含直播课程、录播课程、对应详情课件、工具和赠送知识星球，内部交流技术群。  
  
5、如果是完全零基础的学员，报名第四期，  
会额外赠送一套打基础课程，1～2内足够完成零基础阶段，剩下直接学第四期实战，可以的短期内挖到自己第一个实战漏洞。  
  
6、对于网安方面的考证，如CISP、NISP、PTE均有超低内部价格。  
  
  
  
报名如何联系？扫码加我微信，备注“培训”即可，我来给你详情解答！可以找我了解更多我们的学员成绩！  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ucA3HFqtup6GtnGvOIYXYbOH2UNVQKD2ZfBSvmOMs4ibaRDIesGc7v0aibclQTt6OZpPgdm69JjmHXrJEyBdz1T7EmiatV15lz1XluBkXMFmn8/640?wx_fmt=png&from=appmsg "")  
  
  
下面就是贴一下第四期实战课程课表和赠送的零基础课程课表。  
  
  
  
  
Part 01  
  
![](https://mmecoa.qpic.cn/sz_mmecoa_gif/wCXxeZIKEBLevtPDS3ffqMuEJXkwLCHiaEdicyYhTGVGHRPPc5Mxs8MdHW5hpPg9d4VlChq6unVNxZgDlkOUxmWPn2eBXj1AyQxFRnJwMS990/640?from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/I7v1x5D6icYW4cNXqF7dQDQwDVQ8MdAJojCWCGuvvTFazzrGpQCia5Gwp3IqNNgydwW9Rusrd43Z259a3mYIOciciaPp0XCnOAib59qrA2Q80JibU/640?from=appmsg "")  
  
  
第四期实战培训课表  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/cr9YyS063QrVstianyX9gPA4EZicfkbyKdBQyPtQHPwSLJePicuZmXcBiaLaRSTWrY6UibPpAaeNxLjOSiaeHaSvdHMg/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/CGEwnc7DGPkXwCkjLX8HzCgjKO1KuxPTWw8L9BNNTM3b8WVfHQxV3vIibDycqksck67KWnnVu75ctUZFfpde2kw/640?from=appmsg "")  
  
覆盖企业赏金SRC，众测赏金，Edusrc，cnvd和工作项目渗透挖掘。  
  
内容方面主要是AI大模型赋能网络安全、AI自动化挖洞和逆向、Web挖洞、小程序挖洞、APP挖洞、JS逆向、云安全、护网培训、项目漏洞实战、前端vue路由渗透等内容。  
  
  
你直接按照我这个课程路线走（送零基础），半年可以达到人家学生无指导情况下自学2–3年的效果，很多学生还在学C语言、学PHP、Java的时候，你已经步入实战搞赏金了。  
  
  
下面都是一些课件内容。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ucA3HFqtup7m5rT4icCKzn020oh6vlQ2BYgAwu7U16p8vDgFjgMwlVIzNUVC08U3KN3WZ2Urnb0Dxkr7PLlTTtVO6qwjscC7uibwVlia8V6nRY/640?wx_fmt=png&from=appmsg "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ucA3HFqtup6ByTKVvlkO3Sl6L5ibFNpml7RKl03qYhGyGGetgsWxI6YzLKNtQ79IJnOFHfGHagN2xoKTjQkicJ5QGbl9ynPmqbJb40icQqTuvQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=91 "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ucA3HFqtup7Hea1SfF0TQZgFL3fRicEvuS4Sxam1XcRkOJ8Y4Lu2ynRuia8L2ISQqluTUN0UOIVYXZsEYqfPHMMxiaj0UHRHic1xvODu8KXspIM/640?wx_fmt=png&from=appmsg "")  
  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/ucA3HFqtup76NxtgBcKPCBNPWERnxwG4T7l0K12JX03JFRicibPsQOhRnMpj67SmWhibtOXRjvdn6w9nKSlawCMYTjss7ATVS8jos3yVmtnmng/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=93 "")  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/ucA3HFqtup74HCfgmuDiacG3lwW1YM5lgcMh4fuic745QHjNet6dGY2ib4sbLzUNHVg2RxJBk64msgAValkI0LWStJcRQic8Llv0ziaCuZc2Vjaw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=94 "")  
  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ucA3HFqtup6ByTKVvlkO3Sl6L5ibFNpml7RKl03qYhGyGGetgsWxI6YzLKNtQ79IJnOFHfGHagN2xoKTjQkicJ5QGbl9ynPmqbJb40icQqTuvQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=91 "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/66Px0lScLmhsSQ81eCsVKJ3rdZkSvWiajhKcPGVl5T8wjtDrfJwp0JOA6IpdLIBVawKV3q1zsXaOP95lR6Gm0zg/640?from=appmsg "")  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/UKNVa82CwibfF1ovv4q3fnQ8sib70fohUsibxRVGetvW2trw171icE2PACxlWpIbLOATzduVTL40KNS6GEalG5tqDxdoibe57mYApibHRb4j8iaSyc/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/XNw8Gs7pfmhLibl5aWHeBYCtCO74HXfmFDj5PcJUHfBG4To3m8gGZG1dUFIM5hTU24zSeuQvpZjicc29nJfkuIrxUukkyLBfLpribX42eTqAzY/640?from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ucA3HFqtup45Wug1n4EGjKGvwd0BmUlN6rv4D4UtI6UicFTeicukqpmHLquzgxJPgVGpYmNO0umQHdfMJ8zpOxczibHQKnLEghdhhp5Cw4errk/640?wx_fmt=png&from=appmsg "")  
  
  
Part 02  
  
![](https://mmecoa.qpic.cn/sz_mmecoa_gif/wCXxeZIKEBLevtPDS3ffqMuEJXkwLCHiaEdicyYhTGVGHRPPc5Mxs8MdHW5hpPg9d4VlChq6unVNxZgDlkOUxmWPn2eBXj1AyQxFRnJwMS990/640?from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/I7v1x5D6icYW4cNXqF7dQDQwDVQ8MdAJojCWCGuvvTFazzrGpQCia5Gwp3IqNNgydwW9Rusrd43Z259a3mYIOciciaPp0XCnOAib59qrA2Q80JibU/640?from=appmsg "")  
  
  
零基础培训课表  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ucA3HFqtup6oGf4hxvbeichYWyMkqiaeibGtXzATgia5IWtgI6hBnqkrCO5hFBiapIQlfHbnmbwU3khMCQQc04LHGbeREQSk9pnaHhumUq2lib4cQ/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/sZNhSEe8By4XzViaeg75icD56Zo6C2OO77OKibjbpWDgGImcU2zQZodoXt18qHBQqDnLjZ08HO4UrVUnN66rIs5ajvLmJQYU6ricK2cBgHHODfs/640?from=appmsg "")  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/PRjdmUELtVDZsOJib3BlegVyicZrfRqrkUKQ2sIP9mwxpzUymkzJrv7ZbXh9rJdWGeAJdlkFtQGCBgd5XmaqJdMEzusgnCkyPeGYrn75kcKc0/640?from=appmsg "")  
  
END  
  
  
有想法的朋友，欢迎来找我咨询！  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ucA3HFqtup6bvzcgcFuDJ0HX1ibRBS1ZvWYqKTK7Sp7uboeOephL9e0xaCh3icmYWM58kwopTJdhCa8FRfLVR63jbmwGCwS7iaPFAskT2B0gqQ/640?wx_fmt=png&from=appmsg "")  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zOpksvrDPHytc93z86c7OicUUa77PH9rdAtqjQP6OkoBQNK4LQoDJliahUYePoS13at5QDTNzxq45n4gMFXySVdKoxbCWftLBliblQpvpstfcs/640?from=appmsg "")  
  
  
  
  
  
