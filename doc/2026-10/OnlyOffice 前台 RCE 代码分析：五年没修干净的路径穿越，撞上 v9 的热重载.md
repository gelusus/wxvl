#  OnlyOffice 前台 RCE 代码分析：五年没修干净的路径穿越，撞上 v9 的热重载  
原创 Kratos
                    Kratos  Kratos Sec   2026-10-10 14:01  
  
# OnlyOffice 前台 RCE 代码分析：五年没修干净的路径穿越，撞上 v9 的热重载  
  
10 月 8 日，CISA 把一个"高龄"漏洞 CVE-2021-3199 加进 KEV 目录：OnlyOffice Document Server 5.6.3 之前版本 /upload  
 接口的路径穿越，2021 年 1 月就修了，五年来一直在野外被打，联邦机构的整改截止日期只给了三天（10 月 11 日）。两天前，一篇新的公开研究又把同一个产品送回聚光灯下：现行 9.x 版本上，一条未授权路径穿越 + 配置热重载的组合链，从任意文件写直达 RCE，全程不需要任何凭据。把这两件事放在一起看，这是一个"修了五年没修干净"的典型样本。这篇从代码层面把新链路拆开讲，顺带回答自查和处置该怎么做。  
## 0x0 事件概述  
  
先把两件叠加的事分清楚，预警稿经常把它们混成一团：  
  
<table><thead><tr><th style="color: rgb(102, 102, 102);font-size: 16px;line-height: 1.5em;letter-spacing: 0em;text-align: left;font-weight: bold;background: none left top / auto no-repeat scroll padding-box border-box rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">项目</span></section></th><th style="color: rgb(102, 102, 102);font-size: 16px;line-height: 1.5em;letter-spacing: 0em;text-align: left;font-weight: bold;background: none left top / auto no-repeat scroll padding-box border-box rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">事件 A：KEV 收录</span></section></th><th style="color: rgb(102, 102, 102);font-size: 16px;line-height: 1.5em;letter-spacing: 0em;text-align: left;font-weight: bold;background: none left top / auto no-repeat scroll padding-box border-box rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">事件 B：新公开研究</span></section></th></tr></thead><tbody><tr style="color: rgb(102, 102, 102);background-attachment: scroll;background-clip: border-box;background-color: rgb(255, 255, 255);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">编号</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">CVE-2021-3199（CWE-22）</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">暂无 CVE（截稿时官方未公告）</span></section></td></tr><tr style="color: rgb(102, 102, 102);background-attachment: scroll;background-clip: border-box;background-color: rgb(248, 248, 248);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">内容</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><code><span leaf="">/upload</span></code><section><span leaf=""> 图片上传参数路径穿越，可 RCE</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><code><span leaf="">/downloadas</span></code><section><span leaf=""> 任意文件写 + v9 配置热重载 → 未授权 RCE</span></section></td></tr><tr style="color: rgb(102, 102, 102);background-attachment: scroll;background-clip: border-box;background-color: rgb(255, 255, 255);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">影响版本</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">&lt; 5.6.3（2021 年 1 月修复）</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">写原语 5.0+ 就有；v9.x 可即时触发（验证于 9.4/master）</span></section></td></tr><tr style="color: rgb(102, 102, 102);background-attachment: scroll;background-clip: border-box;background-color: rgb(248, 248, 248);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">时间线</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">10 月 8 日入 KEV，10 月 11 日整改截止</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">10 月 3 日 PoC 仓库公开，HKCERT 10 月 9 日发公告</span></section></td></tr></tbody></table>  
  
事件 A 是老洞的在野利用确认，CVE-2021-3199 的 NVD 评分 9.8，CISA 要求按 BOD 26-04 做取证分诊。事件 B 是新的攻击面，本文主角。两者的共同点才是重点：**穿越点都在 DocumentServer 的存储层，而存储层的路径检查至今没有统一的边界校验。**  
## 0x1 攻击面：三行路由和默认关闭的 JWT  
  
DocumentServer 的 DocService 是个 Express 应用（现行版本源码在 ONLYOFFICE/server 仓库），路由注册就几行：  
```
// DocService/sources/server.jsapp.post('/converter', utils.checkClientIp, rawFileParser, converterService.convertJson);app.param('docid', (req, res, next, val) => {          // 只校验 URL 里的 :docid  if (constants.DOC_ID_REGEX.test(val)) next();  else res.sendStatus(403);});app.post('/upload/:docid*', rawFileParser, fileUploaderService.uploadImageFile);app.post('/downloadas/:docid', rawFileParser, canvasService.downloadAs);app.post('/savefile/:docid', rawFileParser, canvasService.saveFile);
```  
  
两个事实决定了风险面。第一，DOC_ID_REGEX  
 只管 URL 路径段，真正参与存储路径拼接的 id  
、savekey  
、format  
 全部来自 query 里的 cmd  
 JSON，不经过这个正则。第二，JWT 校验默认是关的：  
```
// Common/config/default.json"token": { "enable": { "browser": false, "request": { "inbox": false, "outbox": false } } }
```  
  
官方 Docker 镜像 7.2 之后默认开启 JWT 并生成随机密钥，但源码默认值是全关，裸机 deb/rpm 安装、老版本升级上来的部署、以及自行改过配置的实例，很可能处于"前台完全裸奔"的状态。源码里甚至专门打了警告（server.js:141），JWT 没开全时启动日志会提示你。/downloadas  
 处理器里的校验逻辑长这样：  
```
// DocService/sources/canvasservice.js downloadAsconst strCmd = req.query['cmd'];const cmd = new commonDefines.InputCommand(JSON.parse(strCmd));  // cmd 字段全部攻击者可控...if (tenTokenEnableBrowser || cmd.getTokenDownload() || cmd.getTokenSession()) {  // 校验 JWT，失败 403}// browser 开关为 false 且请求不带 token：上面整段跳过，直接往下走cmd.setData(req.body);                                          // 请求体原样进数据
```  
  
JWT 关闭时，任何人一个 POST 就能走到存储写入。  
## 0x2 任意文件写：savetype=3 绕过归一化  
  
写入链路三步，每一步都能在代码里看到"防了一半"的痕迹。  
  
**第一步，进 saveParts。**downloadAs  
 里 c:"save"  
 分支进入 commandSave  
，把用户可控的 format  
 拼进文件名：  
```
// DocService/sources/canvasservice.jsfunction* commandSave(ctx, cmd, outputData) {  const format = cmd.getFormat() || 'bin';  const completeParts = yield* saveParts(ctx, cmd, 'Editor.' + format);
```  
  
**第二步，归一化只做了一半。**saveParts  
 的代码和注释都值得细读：  
```
function* saveParts(ctx, cmd, filename) {  const saveType = cmd.getSaveType();  if (SAVE_TYPE_COMPLETE_ALL !== saveType) {           // savetype != 3 才处理    const ext = pathModule.extname(filename);    const saveIndex = parseInt(cmd.getSaveIndex()) || 1; //prevent path traversal    filename = pathModule.basename(filename, ext) + saveIndex + ext;  }  ...  yield storage.putObject(ctx, cmd.getDocId() + cmd.getSaveKey() + '/' + filename,                          buffer, buffer.length);
```  
  
注释写着 prevent path traversal  
，手段是 path.basename  
 剥掉所有目录成分——但这个处理被包在 savetype !== 3  
 的条件里。攻击者把 savetype  
 设成 3（COMPLETE_ALL），归一化整段跳过，filename  
 原样进入拼接。同文件 361 行还有一条历史注释："set saveKey as postfix to fix vulnerability with path traversal"——同一个函数附近，两次针对穿越的补丁痕迹，都没有覆盖全部路径。  
  
拼接串 docId + saveKey + '/' + filename  
 三个成分全部来自 cmd JSON，savekey  
 自己带上就跳过随机化（417 行：只有没带 saveKey 才生成随机值）。三者组合出的相对路径完全可控。  
  
**第三步，存储层裸 join。**  
```
// Common/sources/storage/storage-fs.jsfunction getFilePath(storageCfg, strPath) {  return path.join(storageCfg.fs.folderPath, strPath);   // 无包含性检查}async function putObject(storageCfg, strPath, buffer, _contentLength) {  const fsPath = getFilePath(storageCfg, strPath);  await mkdir(path.dirname(fsPath), {recursive: true});  // 递归建目录  ...  await writeFile(fsPath, buffer);                       // 原样落盘}
```  
  
path.join  
 会归一化 ..  
，穿越序列直接吃掉前面的固定段。仓库里其实有 checkPathTraversal  
（utils.js:1156，检查解析后路径是否逃出根目录、顺带防空字节），但全局搜调用点，只有 FileConverter/converter.js  
 用了，/downloadas  
 这条路上一处都没有。  
  
生产配置的存储根是 /var/lib/onlyoffice/documentserver/App_Data/cache/files  
（production-linux.json），请求形态大致是：  
```
POST /downloadas/aaa?cmd={"c":"save","id":"PWNA","savekey":"SK","savetype":3,     "format":"/../../../../var/www/onlyoffice/Data/runtime.json"}Content-Type: text/plain<任意字节，原样落盘>
```  
  
任意文件写原语成立：路径、文件名、内容三者全部攻击者可控。  
## 0x3 从文件写到 RCE：v9 的新灯  
  
任意文件写不等于 RCE，通常要等重启、等加载。v9 把这个"等"消掉了。  
  
**v9 引入了 runtime.json 热重载。**  
 生产配置里 runtimeConfig.filePath  
 指向 /var/www/onlyoffice/Data/runtime.json  
，管理器对这个文件做 fs.watch  
，变更后 200ms 防抖重载：  
```
// Common/sources/runtimeConfigManager.jsconst RELOAD_DEBOUNCE_MS = 200;reloadTimer = setTimeout(() => {  nodeCache.del(configFileName);  operationContext.global.cleanRuntimeConfigCache();   // 清空配置缓存  getConfig(...);                                       // 立即重读文件}, RELOAD_DEBOUNCE_MS);
```  
  
**热重载的配置优先级高于一切。**  
 每个请求初始化上下文时的合并顺序：  
```
// Common/sources/operationContext.jsthis.config = utils.deepMergeObjects({}, moduleReloader.getBaseConfig(),                                     runtimeConfig, tenantConfig);
```  
  
runtimeConfig 排在 baseConfig 后面，同名键直接覆盖。也就是说，写进 runtime.json 的任何配置项，200ms 内全局生效，不用重启，没有键位限制。  
  
**执行点在 FileConverter。**  
 converter 的 docbuilder 路径同样从这份热配置里读：  
```
// FileConverter/sources/converter.jsconst tenDocbuilderPath = resolveConverterPath(  ctx.getCfg('FileConverter.converter.docbuilderPath', cfgDocbuilderPath));...processPath = tenDocbuilderPath;const spawnAsyncPromise = spawnAsync(processPath, childArgs, spawnOptions);  // 直接 spawn
```  
  
而 resolveConverterPath  
 对绝对路径原样放行（converterPaths.js：注释明说 "Absolute paths are returned unchanged"）。  
  
完整链条收拢成四步：  
1. POST /downloadas  
 穿越写入 /var/www/onlyoffice/Data/runtime.json  
，内容为攻击者构造的 JSON：关闭 token.enable.*  
，把 FileConverter.converter.docbuilderPath  
 指向自己的可执行文件；  
  
1. 200ms 后热重载生效，JWT 校验被关掉（研究里观察到 /converter  
 从报错 -8 实时翻转为放行）；  
  
1. POST /docbuilder  
 投递构建任务；  
  
1. converter spawn 劫持后的路径，代码执行。社区版 converter 在进程内跑内存队列，下一个任务就会触发。  
  
顺带一提，spawnOptions.env  
、x2tPath  
 等键同样能通过这份配置注入，LD_PRELOAD 一类的玩法也在射程内。v9 之前的版本没有热重载，但写原语一样成立：覆写 local.json  
 等重启、覆写服务端 JS 文件、覆写 web-apps  
/sdkjs  
 静态资源打存储型 XSS 偷 JWT，只是从"即时"退化成"待机而动"。  
## 0x4 历史回响：同一个类别的三次补丁  
  
CHANGELOG 里的记录可以直接当编年史读：5.6.2 "Fix Path Traversal vulnerability via savefile  
 param"，5.6.3 "Fix Path Traversal vulnerability via image upload params"（即 CVE-2021-3199）。2022 年某红队实录正是拿老版 OnlyOffice 的 savefile 任意文件写突进内网；2024 年长亭的低版本复现分析、10 月 3 日的 v9 新研究，打的都是同一类问题：**存储层路径拼接缺少统一的根目录边界校验，修复长期是"哪个参数被报了修哪个"。**  
  
这次 KEV 收录说明老洞在野外持续有流量。而对已升级到新版本的实例，事件 A 不构成直接影响，事件 B 的写原语却依然存在——这也是为什么自查不能只看版本号。  
## 0x5 自查  
  
**第一层，暴露面。**  
 DocumentServer 按设计只该被集成应用（ownCloud/Nextcloud/Confluence 等）的内网回调访问。查你的实例是否映射到公网：/healthcheck  
 返回 true  
 即为 DocService。FOFA/Shodan 上搜 ONLYOFFICE 相关特征，把自家的实例数量先摸清。  
  
**第二层，版本与 JWT 状态。**  
 版本用包管理器查（dpkg -l | grep onlyoffice  
 / rpm -qa | grep onlyoffice  
）。低于 5.6.3 的直接按失陷处理——KEV 在野加三年半的暴露窗口。JWT 状态看 /etc/onlyoffice/documentserver/local.json  
 或容器环境变量 JWT_ENABLED  
，确认 token.enable.browser  
 和 token.enable.request.inbox  
 都是 true  
 且密钥为强随机值。注意 downloadas  
 的校验挂在 browser  
 开关上，两个开关要一起看。  
  
**第三层，失陷痕迹。**  
 按研究给出的 IOC 排查：  
- /var/www/onlyoffice/Data/runtime.json  
 的 mtime 与内容，重点看有没有 token.enable  
、docbuilderPath  
、spawnOptions  
、x2tPath  
 相关键；  
  
- 存储根 /var/lib/onlyoffice/documentserver/App_Data/cache/files  
 下出现 ../  
 无法解释的目录结构、.cmd/.sh/.so  
 等异常扩展名文件；  
  
- DocService 日志里 Start downloadAs  
 记录的 cmd JSON 含 savetype":3  
 且 format 带 ../  
；  
  
- converter/docbuilder/x2t 子进程的可执行路径异常、DocService 派生 shell。  
  
**临时拦截。**  
 WAF 对 /downloadas  
、/upload  
、/converter  
、/docbuilder  
 四个路径的请求做限制：来源收敛到集成应用网段，query 的 cmd  
 参数中出现 ../  
 直接拦截。  
## 0x6 修复与缓解  
  
**升级是基础动作但不是全部。**  
 5.6.3 之前版本立即升级，老洞的五年在野利用是实打实的。但要向管理层讲清楚：新研究的链路截稿时还没有官方补丁公告，9.x 一样在影响范围内，升级解决不了事件 B。  
  
**现阶段真正有效的三道防线：**  
1. JWT 全开 + 强随机密钥 + 密钥不落代码仓库。写原语的入口在没有 token 的接口上，把门关上，链条第一步就断了；  
  
1. 网络收敛。DocServer 只允许集成应用所在网段访问，公网直连一律掐掉，这是对未知穿越点最普适的防御；  
  
1. 监控 runtime.json 的变更与 converter 的进程派生行为，作为兜底告警。  
  
**对厂商的期待（研究者的修复建议，值得跟进验证）：**  
 存储层统一做 path.resolve  
 后的根目录前缀校验，拒绝含 ..  
 的键；DOC_ID_REGEX  
 覆盖到 cmd 内的 id  
；savetype=3  
 同样过 basename；runtime/tenant 配置对 token.*  
、secret.*  
、*Path  
、spawnOptions  
 这类敏感键做白名单隔离，热重载不该能改安全开关。  
  
**参考**  
- https://github.com/FlowerWitch/OnlyOffice_Document_Server_Unauthenticated_RCE  
  
- https://github.com/ONLYOFFICE/server  
  
**免责声明**  
：本文仅用于安全研究与学习，未经授权不得用于任何非法渗透测试活动。  
  
  
