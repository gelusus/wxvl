#  关于TSRC漏洞奖励方案调整及测试行为违规通报的公告  
 腾讯安全应急响应中心   2026-10-20 09:25  
  
![](https://mmbiz.qpic.cn/mmbiz_png/LXHLqNpSh2wPMJQOvEnq0VtZwbLrqUHMkDxW5XjqIdDB59cWVfQ7ZsutGyib22Ra1629MbEnJvG3HRqeLJIrU3w/640?wx_fmt=png "")  
  
  
各位师傅  
  
感谢大家过去的每一次提交。每一个真实、有效的漏洞，都在让我们的产品离安全更近一步。  
  
为了让奖励更精准地回馈高质量的发现，TSRC全面更新了基础奖励方案，同时对漏洞测试过程中白帽的违规行为进行通报，详细公告内容如下。  
  
  
  
  
**一、基础奖励调整**  
  
本方案自  
2026年10月21日 0点起施行。  
  
方案生效前提交的漏洞，按提交时间对应的奖励标准执行，最新贡献系数如下。  
<table><tbody><tr><th colspan="5" data-colwidth="101,104,98,98,101"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 12px;">TSRC贡献体系</span></span></section></th></tr><tr><td rowspan="2" data-colwidth="101"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 12px;">危害分级</span></span></section></td><td rowspan="2" data-colwidth="104"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 12px;">积分</span></span></section></td><td colspan="3" data-colwidth="98,98,101"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 12px;">贡献系数</span></span></section></td></tr><tr><td data-colwidth="98"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 12px;">其他</span></span></section></td><td data-colwidth="98"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 12px;">重点</span></span></section></td><td data-colwidth="101"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 12px;">核心</span></span></section></td></tr><tr><td data-colwidth="101"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 12px;">低</span></span></section></td><td data-colwidth="104"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 12px;">1-2</span></span></section></td><td data-colwidth="98"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 12px;font-weight: bold;">1</span></span></section></td><td data-colwidth="98"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 12px;font-weight: bold;">3</span></span></section></td><td data-colwidth="101"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 12px;font-weight: bold;">6</span></span></section></td></tr><tr><td data-colwidth="101"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 12px;">中</span></span></section></td><td data-colwidth="104"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 12px;">3-5</span></span></section></td><td data-colwidth="98"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 12px;font-weight: bold;">2</span></span></section></td><td data-colwidth="98"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 12px;font-weight: bold;">9</span></span></section></td><td data-colwidth="101"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 12px;font-weight: bold;">17</span></span></section></td></tr><tr><td data-colwidth="101"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 12px;">高</span></span></section></td><td data-colwidth="104"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 12px;">6-8</span></span></section></td><td data-colwidth="98"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 12px;">11</span></span></section></td><td data-colwidth="98"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 12px;">90</span></span></section></td><td data-colwidth="101"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 12px;">134</span></span></section></td></tr><tr><td data-colwidth="101"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 12px;">严重</span></span></section></td><td data-colwidth="104"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 12px;">9-10</span></span></section></td><td data-colwidth="98"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 12px;">18</span></span></section></td><td data-colwidth="98"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 12px;font-weight: bold;">211</span></span></section></td><td data-colwidth="101"><section style="text-align: center;"><span leaf=""><span textstyle="" style="font-size: 12px;font-weight: bold;">314</span></span></section></td></tr></tbody></table>  
  
1. 严重漏洞基础奖励上调  
<table><tbody><tr><th data-colwidth="80"><section><span leaf=""><span textstyle="" style="font-size: 12px;">业务类型</span></span></section></th><th data-colwidth="77"><section><span leaf=""><span textstyle="" style="font-size: 12px;">危害分级</span></span></section></th><th data-colwidth="118"><section><span leaf=""><span textstyle="" style="font-size: 12px;">调整前（元）</span></span></section></th><th data-colwidth="113"><section><span leaf=""><span textstyle="" style="font-size: 12px;">调整后（元）</span></span></section></th></tr><tr><td data-colwidth="80"><section><span leaf=""><span textstyle="" style="font-size: 12px;">核心</span></span></section></td><td data-colwidth="77"><section><span leaf=""><span textstyle="" style="font-size: 12px;">严重</span></span></section></td><td data-colwidth="118"><section><span leaf=""><span textstyle="" style="font-size: 12px;">12870～14300</span></span></section></td><td data-colwidth="113"><section><span leaf=""><span textstyle="" style="font-size: 12px;">14130～15700</span></span></section></td></tr><tr><td data-colwidth="80"><section><span leaf=""><span textstyle="" style="font-size: 12px;">重点</span></span></section></td><td data-colwidth="77"><section><span leaf=""><span textstyle="" style="font-size: 12px;">严重</span></span></section></td><td data-colwidth="118"><section><span leaf=""><span textstyle="" style="font-size: 12px;">8640～9600</span></span></section></td><td data-colwidth="113"><section><span leaf=""><span textstyle="" style="font-size: 12px;">9495～10550</span></span></section></td></tr></tbody></table>  
本次调整仅涉及基础奖励，严重漏洞的「季度精锐计划」额外奖励维持不变，仍按[更优质的发现，更丰厚的回报｜严重漏洞额外奖励全面升级](https://mp.weixin.qq.com/s?__biz=MjM5NzE1NjA0MQ==&mid=2651208677&idx=1&sn=c8b8447236999622d5ebec232e1a7425&scene=21#wechat_redirect)  
规则进行评选。  
  
2. 中危漏洞奖励调整  
<table><tbody><tr><th data-colwidth="84"><section><span leaf=""><span textstyle="" style="font-size: 12px;">业务类型</span></span></section></th><th data-colwidth="77"><section><span leaf=""><span textstyle="" style="font-size: 12px;">危害分级</span></span></section></th><th data-colwidth="105"><section><span leaf=""><span textstyle="" style="font-size: 12px;">调整前（元）</span></span></section></th><th data-colwidth="103"><section><span leaf=""><span textstyle="" style="font-size: 12px;">调整后（元）</span></span></section></th></tr><tr><td data-colwidth="84"><section><span leaf=""><span textstyle="" style="font-size: 12px;">核心</span></span></section></td><td data-colwidth="77"><section><span leaf=""><span textstyle="" style="font-size: 12px;">中</span></span></section></td><td data-colwidth="105"><section><span leaf=""><span textstyle="" style="font-size: 12px;">525～875</span></span></section></td><td data-colwidth="103"><section><span leaf=""><span textstyle="" style="font-size: 12px;">255～425</span></span></section></td></tr><tr><td data-colwidth="84"><section><span leaf=""><span textstyle="" style="font-size: 12px;">重点</span></span></section></td><td data-colwidth="77"><section><span leaf=""><span textstyle="" style="font-size: 12px;">中</span></span></section></td><td data-colwidth="105"><section><span leaf=""><span textstyle="" style="font-size: 12px;">270～450</span></span></section></td><td data-colwidth="103"><section><span leaf=""><span textstyle="" style="font-size: 12px;">135～225</span></span></section></td></tr><tr><td data-colwidth="84"><section><span leaf=""><span textstyle="" style="font-size: 12px;">其他</span></span></section></td><td data-colwidth="77"><section><span leaf=""><span textstyle="" style="font-size: 12px;">中</span></span></section></td><td data-colwidth="105"><section><span leaf=""><span textstyle="" style="font-size: 12px;">75～125</span></span></section></td><td data-colwidth="103"><section><span leaf=""><span textstyle="" style="font-size: 12px;">30～50</span></span></section></td></tr></tbody></table>  
3. 低危漏洞奖励调整  
<table><tbody><tr><th data-colwidth="84"><section><span leaf=""><span textstyle="" style="font-size: 12px;">业务类型</span></span></section></th><th data-colwidth="77"><section><span leaf=""><span textstyle="" style="font-size: 12px;">危害分级</span></span></section></th><th data-colwidth="105"><section><span leaf=""><span textstyle="" style="font-size: 12px;">调整前（元）</span></span></section></th><th data-colwidth="106"><section><span leaf=""><span textstyle="" style="font-size: 12px;">调整后（元）</span></span></section></th></tr><tr><td data-colwidth="84"><section><span leaf=""><span textstyle="" style="font-size: 12px;">核心</span></span></section></td><td data-colwidth="77"><section><span leaf=""><span textstyle="" style="font-size: 12px;">低</span></span></section></td><td data-colwidth="105"><section><span leaf=""><span textstyle="" style="font-size: 12px;">110～220</span></span></section></td><td data-colwidth="106"><section><span leaf=""><span textstyle="" style="font-size: 12px;">30～60</span></span></section></td></tr><tr><td data-colwidth="84"><section><span leaf=""><span textstyle="" style="font-size: 12px;">重点</span></span></section></td><td data-colwidth="77"><section><span leaf=""><span textstyle="" style="font-size: 12px;">低</span></span></section></td><td data-colwidth="105"><section><span leaf=""><span textstyle="" style="font-size: 12px;">60～120</span></span></section></td><td data-colwidth="106"><section><span leaf=""><span textstyle="" style="font-size: 12px;">15～30</span></span></section></td></tr><tr><td data-colwidth="84"><section><span leaf=""><span textstyle="" style="font-size: 12px;">其他</span></span></section></td><td data-colwidth="77"><section><span leaf=""><span textstyle="" style="font-size: 12px;">低</span></span></section></td><td data-colwidth="105"><section><span leaf=""><span textstyle="" style="font-size: 12px;">20～40</span></span></section></td><td data-colwidth="106"><section><span leaf=""><span textstyle="" style="font-size: 12px;">5～10</span></span></section></td></tr></tbody></table>  
  
  
**二、白帽子违规处理通报**  
  
腾讯安全应急响应中心遵循 “有功必奖，有错必纠” 的原则。不定期发布白帽子违规处置通告，以明确处理原则与标准。我们坚持处理违规行为，以共同守护安全生态的底线。  
  
1. 【历史通告内容】  
  
[TSRC关于白帽子行为规范及违规处理的通告](https://mp.weixin.qq.com/s?__biz=MjM5NzE1NjA0MQ==&mid=2651208097&idx=1&sn=a058a49c8f8d328c7bfa41f5f2629eae&scene=21#wechat_redirect)  
  
  
[TSRC关于AI辅助漏洞挖掘报告的违规处理通告](https://mp.weixin.qq.com/s?__biz=MjM5NzE1NjA0MQ==&mid=2651208514&idx=1&sn=a2490b5914b28f9534786c4126f39b2d&scene=21#wechat_redirect)  
  
  
2. 【本次通报】  
  
（1）在漏洞测试过程中，部分白帽因违规行为  
严重影响腾讯业务正常运行，违反TSRC安全测试边界，且  
严重干扰TSRC运营秩序。  
下述6个案例  
所关联  
白帽账号自确认违规起永久封禁，禁止参与TSRC平台任何漏洞提交及奖励计划。  
- 白帽（Y*******泽）  
通过其他漏洞获取的内部账号进行测试并提交漏洞，经业务审计属于恶意获取业务数据，且存在主观瞒报行为。  
  
- 白帽子（P*****n、榆）  
未严格遵守SSRF测试范围，违规进行扫描内网资产等内网渗透行为  
。  
  
- 白帽（豆****球）  
在AI辅助漏洞挖掘过程中，未合理规范好AI测试边界，导致业务数据被篡改且导致业务不可用。  
  
- 白帽（小*、A***范、1*****e、K***g、P****n）  
在AI辅助漏洞挖掘的报告中，  
未按[《TSRC关于AI辅助漏洞挖掘报告的提交规范》](https://mp.weixin.qq.com/s?__biz=MjM5NzE1NjA0MQ==&mid=2651208362&idx=1&sn=d023ab60dba98fa2e6d7f35d1f371664&scene=21#wechat_redirect)  
  
提供必要的人工验证结果  
。  
  
- 白帽（二**租、不**肉、随**安、梦*、邹********2、七***呛、A***范）  
使用违规违法素材进行测试  
。  
  
- 白帽子（A*******ㅍ）  
在漏洞报告确  
认无效后，申诉沟通过程中使用AI与TSRC团队成员对话，几经提醒仍未调整且言语威胁工作人员  
。  
  
（2）在漏洞测试过程中，部分白帽因违规行  
为  
严重干扰正常用户行为和安全，  
违  
反TSRC安全测试边界，  
对腾讯业务及TSRC运营秩序造成负面影响  
。  
下述5个案例  
所关联  
白帽账号自确认违规起降低相关漏洞奖励并严厉警告，且不得参与年度任何额外激励。  
- 白帽（  
粥  
）  
在已经证明可获取用户敏感信息后，仍违规使用公司员工账号进行测试。  
  
- 白帽（沦*）  
使用AI测试验证越权场景时，未合理规范AI测试边界导致业务数据被修改。  
  
- 白帽（雨*、L**c）  
在已经证明漏洞危害后，仍违规操作写入多条测试数据。  
  
- 白帽（一***子、墨*）  
在未经腾讯授权的情况下，对外分享、传播漏洞详情。  
  
- 白帽（C*******u、哇*哒、C***n、s*******a、A********a、B*G、恋***て）  
在已证明漏洞危害情况下，仍对业务数据过度查阅且下载业务数据。  
  
（3）在漏洞测试过程中，部分白帽因未规范合理提交漏洞报告无法参与漏洞奖励计划。  
下述1个案例所关联白帽账号自确认违规起取消相关漏洞奖励。  
- 白帽（李**、二*、椰*狼、  
珞**戍、  
H********l、  
阿*、  
轩、  
N**h、  
流****凯、  
0**4  
）。  
  
上述行为分别违反了《TSRC安全测试规范》以及《关于AI辅助漏洞挖掘报告的提交规范》  
的相关规定  
构成违规。具体条款和  
【处罚措施及法律责任】  
如下：  
  
《TSRC安全测试规范》  
  
第2条   
"  
您不得利用计算机病毒、网络攻击、网络侵入、干扰腾讯网络正常功能、窃取腾讯网络数据等危害网络安全行为的技术措施（包括但不限于程序、工具）开展安全测试，如上传恶意文件及木马、增删改其他用户个人数据、添加后门账号/权限、扫描攻击内网等渗透行为，不得对国家安全、国计民生、公共利益的关键信息基础设施产生任何危害。"  
  
第4条   
"  
未经腾讯授权，您不得向任何第三方公开漏洞或提供任何与腾讯产品有关的安全情报。"  
  
第5条   
"禁  
止使用任何违反平台规定或法律法规的文字、图片、视频作为测试素材，或上传至TSRC平台的行为。"  
  
第6条（1）  
"在测试过程中如包含数据获取功能时，包括但不限于 SQL 注入、用户资料的越权获取等，应尽可能的采取手动尝试，且获取的数据量不能超过 10 组。"  
  
第6条（4）  
"  
禁止使用可能造成业务影响的漏洞尝试工具和手法，请谨慎开展安全测试，严禁影响业务正常运行。"  
  
第6条（5）   
"测试越权漏洞或其他可能影响用户数据的操作时，请将尝试操作控制在自己创建的多个账号生成的内容中，不得影响线上业务其他用户的正常数据。"  
  
第6条（7）   
"请将所有测试操作和行为及时、如实、完整报告给TSRC，因故意瞒报、漏报等造成业务损害或潜在风险，TSRC保留追责权利。"  
  
《关于AI辅助漏洞挖掘报告的提交规范》  
第4条   
  
“多次提交无效报告的处置，累计提交超过5个此类报告的白帽子，平台有权拉黑您的腾讯安全应急响应中心（TSRC）账号。”  
  
【处罚措施及法律责任】  
  
1、虚构事实、恶意抹黑TSRC平台，或对平台审核/运营人员实施言语攻击、辱骂威胁等恶劣行为，对该账号实施永久封禁处罚。  
  
2、 未经腾讯授权，私自以任何形式传播或公开漏洞（包括但不限于漏洞情报、漏洞细节、漏洞报告、漏洞处理及沟通过程）的行为，一经查实，视情况予以追回奖励、账号永久封禁等处罚。  
  
3、 以安全测试为借口，利用情报信息进行损害用户利益、影响业务正常运作、修复前公开、盗取用户/业务数据等行为，对该账号实施永久封禁、全额追回已发放奖励等处罚，并追究其法律责任。  
  
4、  
若您违反上述安全测试规范1次，TSRC将取消您当次漏洞奖励并视情况予以账号永久封禁等处罚。  
  
  
我们珍视每一位遵守规则、负责任地参与安全建设的白帽子。惩罚并非目的，而是为了守护这片我们共同耕耘和信赖的网络安全环境。  
  
  
  
维护好自己的AI，就是在维护我们共同的安全生态。  
  
希望师傅们在拥抱新工具的同时守住真实与专业。  
  
TSRC珍视师傅们的每一份高质量发现。  
  
-TSRC团队  
  
  
