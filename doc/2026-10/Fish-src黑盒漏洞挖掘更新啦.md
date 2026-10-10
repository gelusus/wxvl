#  Fish-src黑盒漏洞挖掘更新啦  
原创 One
                    One  One security   2026-10-10 06:16  
  
由于前面几个版本的agent是通过vps本地搭建且本地访问，若需要互联网访问总是需要设置ssh代理转发才能访问，此举目的是防止互联网暴露被恶意攻击，今天要介绍的是经过二次更新后的版本。  
  
更新点：  
  
1. 认证方面：使用了nginx反向代理将服务进行公网访问，同时对所有接口进行强鉴权模式，必须登录认证后才能正常使用，避免了未授权恶意使用。  
  
2. agent方面：增强了信息收集及分析能力，在原来的基础上对其进行了更完善的信息收集分析划分，让其更加详细完整；同时增加了专项漏洞挖掘(sql注入、ssrf)，只针对指定漏洞进行详细挖掘，适合深入测试。在各个漏洞测试方面增加强红线规则，强制性禁止高并发、删除、大批量写入等危险操作，对写入和修改测试严格记录原始数据、修改数据、写入数据内容。  
  
3. UI方面：增加了漏洞管理标签，在每个任务结束后会对发现的漏洞进行整理成固定的模板格式，在标签栏中直接查询全部漏洞详情，方便及时预览。  
  
界面展示：  
  
认证界面  
  
![](https://mmbiz.qpic.cn/mmbiz_png/pqOYI94QTU5iaEdp9ic8pRo4Rltv7rbOO9jbSroIibjYy7wC6S9AZkXdoxMRbFZCm0l8ZcA3xfP6T4E3wCch4ibEAib5O0q4FLTDaGIZrhuDZrtQ/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/pqOYI94QTU4QP1283ziaBbvs7b74nvGibicDkm0UxR3Ficao8dEftEyCmJGLG5sl8zM5icAiaxmHwONuv5wuxlXNJpofj93OvD7JneOVcxsm7U2tU/640?wx_fmt=png&from=appmsg "")  
  
agent专项界面  
  
![](https://mmbiz.qpic.cn/mmbiz_png/pqOYI94QTU7UPIxsJb4P8LAIBBlRTRYb3iadLTIXuwicClqe2Z3sYmTxZicjHKgR40ykUo8LcWd9S0tnqzQvecGRfIqQR28V3x5fHibkP4E2Uww/640?wx_fmt=png&from=appmsg "")  
  
漏洞UI界面  
  
![](https://mmbiz.qpic.cn/mmbiz_png/pqOYI94QTU40lX4icI6icp0LYZRxAotUVZtJhKkScnp08Z8P0icIkrEghRSt5M5ya8BbzKIgB5UoEFGiatjhxOgK7xryg4s0GRk2Z6K4TfVDy7g/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/pqOYI94QTU6eH028WGo0EjfRia3hL7JoyhKiaiaFP3FlOibAbDTBSQSqJIFA3yYSB7nDN72pnn4bFFhTzaeBIvgiaNkzJaibmrJgoCS6aJ3ZFyFcQ/640?wx_fmt=png&from=appmsg "")  
  
  
  
  
  
欢迎师傅进行交流  
  
![](https://mmbiz.qpic.cn/mmbiz_png/pqOYI94QTU4ibpnx9pGHGx2aBA1YwHeHGdNSOJ0DExKujeKRr9qrRS3nX2tfuc42h4Mouwo3AAna6ibceBZSe9DbIrFzfukpbzwa8r9pta6qM/640?wx_fmt=png&from=appmsg "")  
  
  
  
