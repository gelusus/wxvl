#  第109篇 AI全栈 · golang实现漏洞监控05——0day漏洞信息监控  
原创 陈看山
                    陈看山  安全诸子   2026-10-22 00:41  
  
0day漏洞的公开渠道有很多，但真正能直接用于监控的却不多。最近在梳理漏洞情报源时发现，Sploitus这类聚合平台虽然方便，但它本质上是在爬取别人的数据——GitHub、Exploit-DB、PacketStorm、Metasploit都在它的抓取范围内。问题在于，如果你依赖聚合平台，你拿到的时间、原始链接、上下文都可能被二次加工过，延迟和失真都难以避免。与其追着聚合器跑，不如直接盯住源头。这次要处理的是0day.today，一个比较特殊的漏洞交易与公开披露平台。  
### 为什么0day.today值得单独监控  
  
0day.today的定位不同于普通CVE数据库。它更偏向于实际可利用的exploit代码和漏洞交易信息，很多条目在NVD或CNNVD里还没建档，就已经出现在这里了。对于安全运营团队来说，这种提前量意味着你可以比攻击者更快地知道某个漏洞已经被公开利用，或者某段利用代码已经在流传。  
  
但访问0day.today有一个很现实的问题——它有反爬机制，而且不是简单的User-Agent检查。打开首页会先弹出一个“I agree”的确认框，点掉之后还会再弹出一个推广弹窗，必须关掉才能看到正常内容。这种交互式障碍，用普通HTTP客户端很难绕过，必须依赖无头浏览器来模拟真实用户操作。  
### 页面结构与数据定位  
  
进入正常的漏洞列表页后，每条漏洞记录的第一列是日期，点击后会跳转到类似https://0day.today/date/17-03-2022  
这样的URL。也就是说，平台本身就提供了按日期归档的页面结构。这意味着你不需要去解析首页的动态列表，只需要直接访问指定日期的URL，就能拿到当天全部公开条目。  
  
这种设计对监控来说非常友好——你不需要维护复杂的爬虫状态机，也不需要处理翻页逻辑，只需要每天定时去抓取前一天的日期页面即可。  
### 用chromedp处理交互式反爬  
  
既然有两次点击操作，直接选用chromedp来驱动Chrome完成整个流程。关键配置有三个：关闭headless模式、关闭自动化控制标志、禁用Blink自动化检测特性。这三个配置缺一不可，否则页面会识别出你是一个自动化程序，行为表现会和真人用户完全不同。  
```
ctx, _ := chromedp.NewExecAllocator(context.Background(), append(     chromedp.DefaultExecAllocatorOptions[:],     chromedp.Flag("headless", false),     chromedp.Flag("enable-automation", false),     chromedp.Flag("disable-blink-features", "AutomationControlled"), )...) ctx, _ = chromedp.NewContext(ctx) ctx, cancel := context.WithTimeout(ctx, 600*time.Second) defer cancel() 
```  
  
页面加载完成后，先等待“I agree”按钮出现，点击后再等待弹窗的关闭按钮出现，再点击关闭，最后把整个页面的HTML取出来。  
```
agree_xpath := "/html/body/div/div[1]/div[14]/div[3]/form/input" close_xpath := "/html/body/div[5]/div/div/a"  err := chromedp.Run(ctx,     chromedp.Navigate(url),     chromedp.WaitVisible(agree_xpath, chromedp.BySearch),     chromedp.Click(agree_xpath),     chromedp.WaitVisible(close_xpath, chromedp.BySearch),     chromedp.Click(close_xpath),     chromedp.InnerHTML("html", &result), ) 
```  
  
这里有个细节需要注意：XPath路径是硬编码的，如果目标站点的前端结构发生变化，这段逻辑就会失效。在生产环境中，建议至少加一个元素文本匹配作为兜底，而不是完全依赖XPath路径。  
### 用goquery解析漏洞条目  
  
拿到完整HTML之后，解析工作交给goquery处理。每条漏洞记录都包在.ExploitTableContent  
这个class里，遍历这个选择器就能提取出所有关键字段。  
```
doc, err := goquery.NewDocumentFromReader(strings.NewReader(result)) if err != nil {     log.Fatal(err) }  doc.Find(".ExploitTableContent").Each(func(i int, s *goquery.Selection) {     date := s.Find(".td a").First().Text()     title := s.Find("h3 a").Text()     category := s.Find("a[href^='/platforms']").Text()     risk := s.Find("[class*=tips_risk_color_]").Text()     cve := s.Find("a[href^='/cve']").Next().Text()     price := s.Find(".tips_price_0").Text() }) 
```  
  
字段提取的逻辑比较直观：标题在h3 a  
里，分类通过/platforms  
链接识别，风险等级在tips_risk_color_  
开头的class里。CVE字段需要特别处理——先找到/cve  
链接，再取它的下一个兄弟节点文本，因为0day.today的CVE编号是跟在CVE链接后面的纯文本。  
### 风险等级与可获取性判定  
  
在数据落地之前，需要做两个关键判断。第一是判断该漏洞是否已有CVE编号：如果cve  
字段为空字符串，说明这是一个尚未被CVE组织收录的0day或半公开漏洞，这类信息通常价值更高。第二是判断exploit是否公开可用：price  
字段如果是“free”，说明任何人都可以直接下载利用代码；如果显示具体价格，说明这是一个付费才能访问的交易条目。  
  
这两种判断直接决定了监控告警的优先级——有CVE且免费公开的漏洞应该立即通知应急响应团队，有CVE但付费的可以降级为关注，无CVE且免费的则需要人工研判是否有内部资产受影响。  
### 风险点、信号、验证与防御  
<table><thead><tr><th style="border: 1px solid #d9dfe4; padding: 8px 10px; background: #f7f7f7; color: #4870ac; text-align: center; font-weight: 700; line-height: 1.45;"><section><span leaf="">风险点</span></section></th><th style="border: 1px solid #d9dfe4; padding: 8px 10px; background: #f7f7f7; color: #4870ac; text-align: center; font-weight: 700; line-height: 1.45;"><section><span leaf="">典型信号</span></section></th><th style="border: 1px solid #d9dfe4; padding: 8px 10px; background: #f7f7f7; color: #4870ac; text-align: center; font-weight: 700; line-height: 1.45;"><section><span leaf="">合法验证思路</span></section></th><th style="border: 1px solid #d9dfe4; padding: 8px 10px; background: #f7f7f7; color: #4870ac; text-align: center; font-weight: 700; line-height: 1.45;"><section><span leaf="">防御建议</span></section></th></tr></thead><tbody><tr><td style="border: 1px solid #d9dfe4; padding: 8px 10px; color: #334155; line-height: 1.55; vertical-align: top;"><section><span leaf="">平台披露的0day被攻击者快速武器化</span></section></td><td style="border: 1px solid #d9dfe4; padding: 8px 10px; color: #334155; line-height: 1.55; vertical-align: top;"><section><span leaf="">某漏洞在0day.today出现后24小时内出现大规模扫描</span></section></td><td style="border: 1px solid #d9dfe4; padding: 8px 10px; color: #334155; line-height: 1.55; vertical-align: top;"><section><span leaf="">在授权范围内查询自身资产是否命中受影响版本</span></section></td><td style="border: 1px solid #d9dfe4; padding: 8px 10px; color: #334155; line-height: 1.55; vertical-align: top;"><section><span leaf="">建立0day.today监控，匹配内部资产指纹，提前部署虚拟补丁</span></section></td></tr><tr><td style="border: 1px solid #d9dfe4; padding: 8px 10px; color: #334155; line-height: 1.55; vertical-align: top;"><section><span leaf="">付费漏洞被二次转卖或泄露</span></section></td><td style="border: 1px solid #d9dfe4; padding: 8px 10px; color: #334155; line-height: 1.55; vertical-align: top;"><section><span leaf="">某付费条目出现在免费渠道</span></section></td><td style="border: 1px solid #d9dfe4; padding: 8px 10px; color: #334155; line-height: 1.55; vertical-align: top;"><section><span leaf="">不主动获取付费漏洞内容，只关注是否已有免费利用代码</span></section></td><td style="border: 1px solid #d9dfe4; padding: 8px 10px; color: #334155; line-height: 1.55; vertical-align: top;"><section><span leaf="">关注免费公开条目即可，付费条目仅记录元数据</span></section></td></tr><tr><td style="border: 1px solid #d9dfe4; padding: 8px 10px; color: #334155; line-height: 1.55; vertical-align: top;"><section><span leaf="">自动化监控被平台识别并封禁</span></section></td><td style="border: 1px solid #d9dfe4; padding: 8px 10px; color: #334155; line-height: 1.55; vertical-align: top;"><section><span leaf="">请求频率过高导致IP被封</span></section></td><td style="border: 1px solid #d9dfe4; padding: 8px 10px; color: #334155; line-height: 1.55; vertical-align: top;"><section><span leaf="">控制抓取频率为每日一次，使用合规的User-Agent标识</span></section></td><td style="border: 1px solid #d9dfe4; padding: 8px 10px; color: #334155; line-height: 1.55; vertical-align: top;"><section><span leaf="">部署在独立IP上，使用代理池做轮换，监控账号被封禁提前量</span></section></td></tr><tr><td style="border: 1px solid #d9dfe4; padding: 8px 10px; color: #334155; line-height: 1.55; vertical-align: top;"><section><span leaf="">解析逻辑随页面改版失效</span></section></td><td style="border: 1px solid #d9dfe4; padding: 8px 10px; color: #334155; line-height: 1.55; vertical-align: top;"><section><span leaf="">关键字段提取为空或解析报错</span></section></td><td style="border: 1px solid #d9dfe4; padding: 8px 10px; color: #334155; line-height: 1.55; vertical-align: top;"><section><span leaf="">每次抓取后校验条目数量是否在合理区间</span></section></td><td style="border: 1px solid #d9dfe4; padding: 8px 10px; color: #334155; line-height: 1.55; vertical-align: top;"><section><span leaf="">增加数据完整性校验，异常时触发告警并人工介入</span></section></td></tr><tr><td style="border: 1px solid #d9dfe4; padding: 8px 10px; color: #334155; line-height: 1.55; vertical-align: top;"><section><span leaf="">误将非漏洞信息当作漏洞告警</span></section></td><td style="border: 1px solid #d9dfe4; padding: 8px 10px; color: #334155; line-height: 1.55; vertical-align: top;"><section><span leaf="">抓取到广告条目或无效链接</span></section></td><td style="border: 1px solid #d9dfe4; padding: 8px 10px; color: #334155; line-height: 1.55; vertical-align: top;"><section><span leaf="">对标题和描述做关键词过滤，排除明显非漏洞内容</span></section></td><td style="border: 1px solid #d9dfe4; padding: 8px 10px; color: #334155; line-height: 1.55; vertical-align: top;"><section><span leaf="">在入库前增加二次筛选，只保留包含exploit、vulnerability、bypass等关键词的条目</span></section></td></tr></tbody></table>### 误判、误用与边界  
  
0day.today上的信息质量参差不齐，不是所有条目都经过严格验证。有些条目可能只是POC片段，有些可能是过期漏洞的重新包装，还有些可能根本就是恶意链接——诱导安全人员去下载带后门的“exploit”。所以在使用这个数据源时，务必保持怀疑态度。  
  
**误判边界**  
：无CVE不等于0day，也可能是漏洞提交者懒得申请CVE编号。免费公开也不等于绝对安全，付费漏洞的交易信息本身就是敏感数据，不应该被公开讨论或转发。  
  
**合规约束**  
：0day.today本身涉及灰色地带，部分内容可能违反当地法律法规。作为安全研究人员，你的合法动作仅限于监控公开可见的元数据和标题，不应该去购买、下载或执行任何付费漏洞内容。如果确实需要验证某个漏洞的影响，应该在自建靶场中复现，或者使用厂商官方提供的检测规则，绝不能拿生产环境做实验。  
### 对团队安全建设的启发  
  
这个案例给到的最大启示是：漏洞监控不是越多数据源越好，而是越接近原始来源越好。聚合平台虽然方便，但会引入额外的延迟和噪音。0day.today这类垂直平台虽然访问门槛高，但数据质量更高，提前量也更明显。建议在实现中把这类源和传统的NVD、厂商公告放在不同的处理管道里——前者走快速告警通道，后者走常规同步流程。  
  
另外，chromedp这套方案虽然可行，但维护成本不低。如果团队有更稳定的代理池或验证码识别能力，可以考虑用纯HTTP方案替代浏览器自动化。不过在当前阶段，先用无头浏览器跑通流程、验证数据价值，比追求极致的爬虫性能更重要。等确认这个数据源确实能持续产出高质量告警，再考虑架构优化也不迟。  
### 给读者的实践建议  
  
如果你打算在自己的漏洞监控体系里加入0day.today这个源，建议按以下顺序推进：  
  
先写一个最小验证脚本，手动跑一次某天的数据抓取，确认能拿到完整字段。然后把这套逻辑封装成定时任务，每天上午执行一次，抓取前一天的归档页面。入库时把CVE编号、风险等级、是否免费这三个字段作为核心索引，其余字段作为补充信息存储。最后接入现有的告警渠道，设定不同的通知策略——免费且高危的条目直接推送，中危的进每日汇总，低危的只做记录。  
  
整个过程中，最重要的不是爬虫代码本身，而是对数据质量的判断逻辑。一个错误的告警会消耗应急响应团队大量精力，一个漏报的0day则可能让你错过最佳防御窗口。在正式上线之前，至少人工比对两周的抓取结果和平台实际页面，确保解析逻辑稳定可靠，再做自动化切换。  
  
