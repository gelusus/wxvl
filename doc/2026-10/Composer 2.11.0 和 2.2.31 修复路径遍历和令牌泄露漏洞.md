#  Composer 2.11.0 和 2.2.31 修复路径遍历和令牌泄露漏洞  
sec随谈
                    sec随谈  sec随谈   2026-10-30 01:08  
  
**内容摘要**  
  
PHP 依赖管理器 Composer 的维护者于 2026 年 10 月 29 日披露了两个漏洞。这两个安全缺陷可在安装软件包时删除或覆盖任意本地文件，并会把私有仓库的 API 令牌暴露在锁文件（lock file）中。开发者和构建工程师必须立即升级到 Composer 2.11.0 或 2.2.31（LTS 版本）。  
  
**为何值得关注**  
  
Composer 是 PHP 生态系统最基础的依赖管理器，每月通过 Packagist 处理数十亿次软件包安装，在 GitHub 上有超过 2.9 万颗星。公告没有给出受影响的 CI/CD 流水线和私有仓库的具体数量，但自动化构建流程高度依赖缓存的依赖和锁文件。  
  
不修复这两个漏洞，持续集成（CI）运行器就面临文件系统被篡改的风险，而且敏感的认证令牌会泄露到版本控制历史中。目前还没有公开的 PoC 利用代码，也没有确认的在野利用。  
  
**攻击原理**  
  
这两个漏洞分别针对依赖解析过程中的本地软件包元数据校验和锁文件生成环节。  
  
**通过已安装软件包元数据实现路径遍历**  
  
第一个是高危路径遍历漏洞（CVSS 7.1，CWE-22），由研究员 euriconicacio 报告，详见  
安全公告  
。Composer 此前已经会拒绝 composer.lock 和远程仓库元数据中的恶意软件包名，但它却信任本地记录在 vendor/composer/installed.json 中的软件包名。  
  
如果攻击者修改 vendor/composer/installed.json，加入一个带 ..  
 路径段的软件包名，那么运行 composer install 时就会删除 vendor/ 目录之外的某个目录。更进一步，运行 composer reinstall 会以运行 Composer 的用户权限，把该软件包的文件写入那个外部路径。这种情况主要影响这样的 CI 流水线：它们从一个可被不受信任的任务（例如来自 fork 的 pull request）写入的共享缓存中恢复 vendor/ 目录。  
  
**锁文件中的 API 令牌泄露**  
  
第二个是中危信息泄露漏洞（CVSS 5.3，CWE-200），由 Saku0512 报告，详见  
安全公告  
。当在全局 config.json 文件中通过 options.http.header 为 composer 类型的仓库配置了内联 HTTP 头时，Composer 会把这些 HTTP 头复制到 composer.lock 中每个已下载软件包的 transport-options 字段下。当开发者把 composer.lock 提交到版本控制，或将其作为构建产物发布时，其中嵌入的任何 API 令牌或凭据都会暴露给所有能读取该锁文件的人。存储在 auth.json 或 COMPOSER_AUTH 环境变量中的凭据不受影响。  
  
**受影响版本**  
  
这两个漏洞影响 1.0 至 2.2.31 之前，以及 2.3.0 至 2.11.0 之前的所有版本。已修复的 2.11.0 和 2.2.31（LTS）版本不受影响。  
  
**补丁与缓解措施**  
  
维护者在 Composer 2.11.0 和 2.2.31（LTS）中修复了这两个问题：对本地已安装的元数据强制执行严格的软件包名校验，并且不再把仓库的 HTTP 头写入 composer.lock。由于现有锁文件中仍保留着之前写入的 HTTP 头，开发者在升级后必须运行 composer update --lock  
 来清除它们。此外，共享过受影响锁文件的团队，还需在 Git 历史中搜索 transport-options 下的 header 条目，吊销所有已暴露的令牌，并把内联凭据迁移到 auth.json 中。  
  
参考链接：  
  
https://github.com/composer/composer/security/advisories/GHSA-fm2w-xgf2-f9fh  
  
https://github.com/composer/composer/security/advisories/GHSA-c968-33g3-wh7p  
  
