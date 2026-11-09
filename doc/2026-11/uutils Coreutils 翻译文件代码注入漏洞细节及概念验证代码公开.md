#  uutils Coreutils 翻译文件代码注入漏洞细节及概念验证代码公开  
sec随谈
                    sec随谈  sec随谈   2026-11-09 01:18  
  
**摘要**  
  
研究人员 JoeCyberTech 公开了 uutils coreutils 中一个严重漏洞的完整细节和概念验证代码，该漏洞的 CVSS 评分为 9.9。利用该漏洞，特制的翻译文件可以在构建时向这些核心工具注入 Rust 代码。维护者已在 uucore 0.12.0 中修复该漏洞。  
  
uucore 的构建脚本会把 Fluent 翻译文本粘贴进一个 Rust 原始字符串（raw string），但没有正确转义。翻译文本中只要包含 "#  
 这两个字符，就能提前闭合该字符串，其后的内容便会被当作代码编译。请升级到 uucore 0.12.0 或更高版本。  
  
**为何重要**  
  
uutils 是用 Rust 重写的 GNU 核心工具集，包括 ls、cp、cat 等命令。研究人员称，它"如今已作为默认的 coreutils 随主流 Linux 发行版发布"。每个工具都链接共享的 uucore crate，crates.io 显示该 crate 的下载量已超过 250 万次。  
  
因此，只需合并一个拉取请求，就可能在受信任的系统二进制文件中植入后门。研究人员警告："一个所有审查者都视为无害文本的文件，实际上却是通向供应链后门的路径。"目前尚未在野外发现恶意翻译文件。  
  
**攻击原理**  
  
以 r#"  
 开头的 Rust 原始字符串，"会在第一次出现 "#  
 这对字节时直接结束"。构建脚本只检查翻译文本中是否含有双引号，从未检查结束符本身。因此，提前出现的 "#  
 之后的所有内容都会"被 rustc 当作代码解析"。  
  
公开的概念验证在一条看似无害的消息中加入了一条 shell 命令。此外，214 个翻译文件中已有 34 个包含引号，而项目的代码检查工具关闭了引号检查。  
  
**受影响版本**  
  
GitHub 安全公告  
列出的受影响版本为 uucore 0.2.2 至 0.11.0。  
  
**修补与缓解措施**  
- 升级到 2026 年 9 月 17 日发布的 uutils coreutils 和 uucore 0.12.0。  
  
- 用已修复的源码树重新构建软件包。  
  
- 审查所有引入项目的 .ftl 文件中是否含有 "#  
 字符组合。  
  
修复方案改用 Rust 的 Debug 格式化器转义所有引号，使利用字符串"变成无害的数据"。如需了解这个 uutils coreutils 漏洞的更多信息，请阅读 JoeCyberTech 关于 uucore FTL 注入的  
完整技术分析  
。  
  
参考链接：  
  
https://github.com/uutils/coreutils/security/advisories/GHSA-c522-hqhp-v2qq  
  
https://joecybertech.com/blog-uucore-ftl-injection-ghsa-c522-hqhp-v2qq.html  
  
