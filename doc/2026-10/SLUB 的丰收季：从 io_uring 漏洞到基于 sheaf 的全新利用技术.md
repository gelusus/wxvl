#  SLUB 的丰收季：从 io_uring 漏洞到基于 sheaf 的全新利用技术  
naup96321
                    naup96321  赛博生存指南   2026-10-28 04:54  
  
> 原文：https://arxiv.org/abs/2609.37608  
作者：Hao-Yu Yang（naup96321）、Yu-Ting Lin  
发布时间：2026-09-29  
翻译模型：GLM-5.3  
  
  
**编者按：**  
 Linux 6.18 给 SLUB 分配器加了两层新缓存：per-CPU 的 sheaf 和 NUMA node 级的 barn。命名用的是收割的比喻——sheaf 是谷捆，barn 是谷仓，对象像庄稼一样打捆入仓。这篇论文围绕这套新机制做了三件事：在 io_uring 挖出两个 0-day，其中 provided buffer 的一个漏洞被串成完整提权链，测试环境开着 KASLR、KPTI、SMAP、SMEP、SLAB_FREELIST_HARDENED 等防护；分析 sheaf 为什么让传统 cross-cache 失效，并给出两个解法；最后反过来把 sheaf/barn 本身当攻击面——指出它的设计让 SLAB_FREELIST_HARDENED 的双重保护形同虚设，并给出三种新技术，其中基于 RCU slab_sheaf 的 cross-cache 摆脱了对 buddy 系统的依赖。两个漏洞均已上报并修复，文中附披露时间线和 PoC 链接。  
## 摘要  
  
Linux 内核对更高 I/O 性能和更高效内存管理的追求，引入了一批新机制；这些机制在提升性能的同时也带来了新的攻击面。本文把其中两个放在一起研究：io_uring 子系统，以及 Linux 6.18 中加进 SLUB 分配器的 sheaf/barn 缓存机制。我们提出了 io_uring 中两个此前未知的漏洞，并把其中一个发展成加固内核配置下完整的本地提权链。研究表明，sheaf/barn 机制改变了 cross-cache 攻击等既有利用技术背后的长期假设，其设计也削弱了现有的 SLUB freelist 防护。我们提出三种基于 sheaf 的新型利用技术，其中 RCU-sheaf cross-cache 技术摆脱了传统上跨 cache 移动对象时对 buddy 系统的依赖。  
## 1 背景  
### 1.1 io_uring 概述  
  
io_uring 在 Linux 5.1 [1] 中引入，是面向高性能异步 I/O 的内核核心组件，主要解决两个问题：同步阻塞 I/O 的瓶颈，以及每次 I/O 操作附带的 syscall 开销。为此，io_uring 提供了覆盖面很宽的异步 I/O 接口，文件、socket、轮询类操作都在支持范围内。  
  
如图 1 所示，io_uring 在内核与用户态之间共享两个环：提交队列（Submission Queue，SQ）和完成队列（Completion Queue，CQ）。应用程序把要执行的 I/O 操作打包成 SQE（Submission Queue Entry）放进 SQ；内核从 SQ 读取 SQE、执行对应操作，生成 CQE（Completion Queue Entry）写入 CQ；应用程序再从 CQ 读取结果，整个 I/O 过程不需要任何 syscall。  
  
![图 1：用户态与内核共享的 io_uring 提交队列与完成队列](https://mmbiz.qpic.cn/sz_mmbiz_png/JpU6JH8dicqWzmZsEF0wU3CQHPDDUGsNudklhBxDJ1paLI1HB2TrL8PYMwxxCq3zbWXQUnWTPuhHyOso1d0Y1weZROKXdT4gialib9IQw7owNU/640?wx_fmt=png&from=appmsg "")  
  
io_uring 还支持批量处理，一次提交多个 SQE。触发方式有两种：应用程序主动调用 io_uring_enter syscall；或者使用 SQPOLL——由内核启动的 iou-sqp 内核线程持续轮询 SQ，应用程序完全不需要发 syscall。  
  
与传统的 epoll 方案相比，epoll 只在 I/O 就绪时通知应用程序，实际 I/O 还要应用自己执行；io_uring 则允许应用把完整的 I/O 请求交给内核，由内核执行并回报结果，降低了应用侧管理 I/O 流程的复杂度。此外，io_uring 也解决了传统异步 I/O（AIO）接口的若干固有问题：支持的 I/O 类型有限、接口碎片化、每个请求都需要专用控制结构和多次 syscall [2, 3]。  
### 1.2 SLUB 与 sheaf 机制  
  
如图 2 所示，SLUB 分配器是内核的主要内存管理器，负责小块内存的分配与释放。它的核心抽象是 kmem_cache，代表一个特定大小对象的内存池。每个内存池由多个 slab 组成，一个 slab 是一组物理连续的页面，存放一批特定大小的对象。传统上，SLUB 分配器分为两层：per-CPU 层（kmem_cache_cpu）和 per-NUMA node 层（kmem_cache_node）。随着 CPU 核数越来越多、NUMA 机器规模越来越大，这套 per-CPU 缓存的同步与管理开销变得不可忽视。于是在 Linux 6.18 [4] 中，内核引入了新的 SLUB 对象缓存机制：逐步替代 kmem_cache_cpu 的本地对象管理，基于 sheaf 和 barn 构建新的缓存层 [5]。SLUB 分配器因此形成三个管理层：sheaf 层、barn 层和原有的 slab 管理层（kmem_cache_node）。  
  
![图 2：引入 sheaf 之后 SLUB 对象管理的三层架构](https://mmbiz.qpic.cn/mmbiz_png/JpU6JH8dicqWXYbzJwniaVbWvK3FDngXkSsjg1Fic5qEjLYMBQYmEq6JJoROQibGW5Alje6MfxzkDMgHeGichIcgaw8tyusQSrZZW7ECxt8q6ibKs/640?wx_fmt=png&from=appmsg "")  
  
第一层里，每个 CPU 的本地 sheaf 集合由三个 slab_sheaf 槽位组成：main、spare、rcu_free。main sheaf 是当前正在使用的 sheaf，所有能走 sheaf 机制的对象分配和释放都经由它处理；spare sheaf 是备位，当 main sheaf 没有对象可分、或没有空间再存放对象时换上；rcu_free sheaf 则用来在 kfree_rcu 之后批量处理 RCU 对象。per-CPU sheaves 的结构定义如下：  
```
// mm/slub.c
// https://elixir.bootlin.com/linux/v7.1.4/source/mm/slub.c#L420
struct slub_percpu_sheaves {
	local_trylock_t lock;
	struct slab_sheaf *main;      /* never NULL when unlocked */
	struct slab_sheaf *spare;     /* empty or full, may be NULL */
	struct slab_sheaf *rcu_free;  /* for batching kfree_rcu() */
};

```  
#### 1.2.1 第二层：barn 层（node_barn）  
  
barn（node_barn）是 SLUB 对象管理架构的第二层，保存完全空闲或完全占满的 sheaf 作为储备，在 NUMA node 内共享。当 per-CPU 的 sheaf 用尽时，内核会先拿 spinlock，再把 barn 里的 sheaf 与 per-CPU 层的交换。barn 能保存的 sheaf 数量有上限——MAX_FULL_SHEAVES 和 MAX_EMPTY_SHEAVES，这两个常数在后文的利用分析里会反复出现。node_barn 的结构定义如下：  
```
// mm/slub.c
// https://elixir.bootlin.com/linux/v7.1.4/source/mm/slub.c#L396
struct node_barn {
	spinlock_t lock;
	struct list_head sheaves_full;
	struct list_head sheaves_empty;
	unsigned int nr_full;
	unsigned int nr_empty;
};

```  
#### 1.2.2 第三层：slab 管理层（kmem_cache_node）  
  
slab 管理层（kmem_cache_node）是 SLUB 对象管理架构的第三层，在 NUMA node 级别管理 slab 的状态，处理跨 CPU、跨 node 的内存回收与均衡，并维护该 node 上全部 slab 的完整链表，跟踪每个 slab 处于 partial、full 还是 empty 状态。  
## 2 漏洞一：CQ/SQ resize 与延迟 task work 之间的 race condition  
### 2.1 根因  
  
漏洞通过 io_uring_register syscall 触发，opcode 为 IORING_REGISTER_RESIZE_RINGS，处理函数是 io_register_resize_rings()。这个特性允许在不重建 io_uring 实例的前提下，扩展或收缩已有实例的 SQ/CQ ring。  
```
// fs/io_uring/register.c
// https://elixir.bootlin.com/linux/v6.19.3/source/fs/io_uring/register.c#L328
case IORING_REGISTER_RESIZE_RINGS:
	ret = -EINVAL;
	if (!arg || nr_args != 1)
		break;
	ret = io_register_resize_rings(ctx, arg);
	break;

```  
  
io_register_resize_rings() 会先持有 ctx->mmap_lock 和 ctx->completion_lock，把 rings 暂时置为 NULL；分配好新的 ring 内存后，再把新地址写回：  
```
// fs/io_uring/register.c
// https://elixir.bootlin.com/linux/v6.19.3/source/fs/io_uring/register.c#L348
mutex_lock(&ctx->mmap_lock);
spin_lock(&ctx->completion_lock);
o.rings = ctx->rings;
ctx->rings = NULL;
o.sq_sqes = ctx->sq_sqes;
ctx->sq_sqes = NULL;
...
ctx->sq_entries = p->sq_entries;
ctx->cq_entries = p->cq_entries;
ctx->rings = n.rings;

```  
  
与之交互的是 IORING_SETUP_DEFER_TASKRUN。按照官方文档 [6] 的说法：io_uring 默认会在任何 syscall 结束或线程中断时处理全部未完成的工作，这可能拖慢应用程序的进度。设置这个标志是给 io_uring 一个提示：把工作推迟到应用程序调用带 IORING_ENTER_GETEVENTS 的 io_uring_enter 时再做。  
  
被推迟的任务会进入 task work 队列。新 task work 入队时，内核会设置 IORING_SQ_TASKRUN 标志，通知用户态有待处理的任务。问题在于：更新 io_ring_ctx 中的标志时，这个操作并没有持有与 resize 相同的锁——race condition 就出在这里。  
```
// fs/io_uring/io_uring.c
// https://elixir.bootlin.com/linux/v6.19.3/source/fs/io_uring/io_uring.c#L1258
static void io_req_local_work_add(struct io_kiocb *req, unsigned flags)
{
	struct io_ring_ctx *ctx = req->ctx;
	...
	if (!head) {
		if (ctx->flags & IORING_SETUP_TASKRUN_FLAG)
			atomic_or(IORING_SQ_TASKRUN, &ctx->rings->sq_flags);
		if (ctx->has_evfd)
			io_eventfd_signal(ctx, false);
	}
}

```  
### 2.2 PoC  
  
PoC 的思路是让这两个组件持续相互触发：先创建一个带 IORING_SETUP_DEFER_TASKRUN 的 io_uring 实例并 mmap ring；再创建一个每 100µs 到期一次的 timerfd，为它提交一个 multishot 的 IORING_OP_POLL_ADD（POLLIN）。timerfd 每次到期都会产生一个 poll 完成事件，该事件经 task work 延迟处理，于是内核反复执行 io_req_local_work_add() 并设置 IORING_SQ_TASKRUN。  
  
主循环每一轮先用带 GETEVENTS 的 io_uring_enter 让内核执行积压的 task work，然后调用 IORING_REGISTER_RESIZE_RINGS 重新配置 ring。当 task work 正要通过 ctx->rings 修改 sq_flags，而另一个 CPU 同时在做 resize，或执行流在关键窗口内被 resize 抢占时，race condition 就可能让 ctx->rings 引用的 ring 结构失效——最终造成 null pointer dereference。  
  
**披露时间线：**  
- 2026-03-07：发现漏洞  
  
- 2026-03-09：上报上游并附修复建议  
  
- 2026-03-17：补丁合入并发布  
  
**相关产物：**  
- 补丁：https://git.kernel.org/pub/scm/linux/kernel/git/stable/stable-queue.git/commit/?id=99ea6d049a509ca24b02347a6a4e0614c284c3a7  
  
- PoC：https://hackmd.io/@naup96321/rkXiDMqoVg  
  
- 演示：https://www.youtube.com/watch?v=d5j2BfGGtsc  
  
## 3 漏洞二：provided buffer bundle 扩容流程错误处理路径中的逻辑错误  
### 3.1 根因  
  
漏洞出在 IORING_OP_SEND [7] 的处理路径上——异步 socket 发送，由 io_send 函数处理。内核在处理时构建 buf_sel_arg，通过 io_buffers_select() 从注册的缓冲池中为这次发送选取数据源。iovec 指针先从 kmsg 拷贝进 arg.iovs，再传给 io_buffers_select()：  
```
// fs/io_uring/net.c
// https://elixir.bootlin.com/linux/v6.19.3/source/fs/io_uring/net.c#L620
struct buf_sel_arg arg = {
	.iovs = &kmsg->fast_iov,
	.max_len = min_not_zero(sr->len, INT_MAX),
	.nr_iovs = 1,
	.buf_group = sr->buf_group,
};
int ret;

if (kmsg->vec.iovec) {
	arg.nr_iovs = kmsg->vec.nr;
	arg.iovs = kmsg->vec.iovec;
	arg.mode = KBUF_MODE_FREE;
}
if (!(sr->flags & IORING_RECVSEND_BUNDLE))
	arg.nr_iovs = 1;
else
	arg.mode |= KBUF_MODE_EXPAND;

ret = io_buffers_select(req, &arg, sel, issue_flags);
if (unlikely(ret < 0))
	return ret;
if (arg.iovs != &kmsg->fast_iov && arg.iovs != kmsg->vec.iovec) {
	kmsg->vec.nr = ret;
	kmsg->vec.iovec = arg.iovs;
	req->flags |= REQ_F_NEED_CLEANUP;
}

```  
  
io_buffers_select() 依据 buf_group 定位 buffer list。本文选择 provided buffer 作为分析对象：它允许应用程序在收发时不为每个请求指定具体内存地址，由内核从预先注册的缓冲池中自动选取空闲内存，缓冲池通过 IORING_REGISTER_PBUF_RING [8] 注册。  
```
// fs/io_uring/kbuf.c
// https://elixir.bootlin.com/linux/v6.19.3/source/fs/io_uring/kbuf.c#L352
int io_buffers_select(struct io_kiocb *req, struct buf_sel_arg *arg,
		      struct io_br_sel *sel, unsigned int issue_flags)
{
	struct io_ring_ctx *ctx = req->ctx;
	int ret = -ENOENT;

	io_ring_submit_lock(ctx, issue_flags);
	sel->buf_list = io_buffer_get_list(ctx, arg->buf_group);
	if (unlikely(!sel->buf_list))
		goto out_unlock;
	if (sel->buf_list->flags & IOBL_BUF_RING) {
		ret = io_ring_buffers_peek(req, arg, sel->buf_list);

```  
  
io_ring_buffers_peek() 从 buffer ring 的头部开始，按顺序取一个或多个 buffer，转换成 iovec 供 send/recv 使用；启用 bundle 时一次会选多个 buffer/iovec。kmsg 自带一个 iovec 数组，不够大时内核会 kmalloc 一块新内存扩容：  
```
// fs/io_uring/kbuf.c
// https://elixir.bootlin.com/linux/v6.19.3/source/fs/io_uring/kbuf.c#L287
iov = kmalloc_objs(struct iovec, nr_avail);
if (unlikely(!iov))
	return -ENOMEM;
if (arg->mode & KBUF_MODE_FREE)
	kfree(arg->iovs);
arg->iovs = iov;
nr_iovs = nr_avail;

```  
  
处理向上返回后，arg 中更新过的 iovec 会写回 kmsg。但扩容之后，内核会逐个校验 provided buffer 的地址，一旦发现非法地址就报错返回。这个错误返回发生在 kmsg 的指针被更新之前——此时 kmsg 持有的仍是指向旧 iovec 数组的指针，而旧数组已经被释放——UAF 由此产生：  
```
// fs/io_uring/kbuf.c
// https://elixir.bootlin.com/linux/v6.19.3/source/fs/io_uring/kbuf.c#L318
if (unlikely(!access_ok(iov->iov_base, len))) {
	if (arg->iovs != org_iovs)
		kfree(arg->iovs);
	return -EFAULT;
}

```  
### 3.2 漏洞利用  
  
测试环境开启了以下防护：KASLR、KPTI、SMAP、SMEP；CONFIG_MEMCG_KMEM；CONFIG_STATIC_USERMODEHELPER；CONFIG_SLAB_FREELIST_HARDENED；CONFIG_SLAB_FREELIST_RANDOM。  
#### 3.2.1 泄露 KASLR  
  
这套 UAF 原语相当强：kmalloc 的尺寸可控，分配走 GFP_KERNEL 内存池，还能把数据写到 UAF 结构上。但由于开启了 CONFIG_MEMCG_KMEM，heap spray 的目标结构要避开分配时带 GFP_KERNEL_ACCOUNT 标志的 kmalloc 结构。替代方案是做 cross-cache——它的一个创新版本会在第 4、6 节讨论 [9]。获取 UAF 的流程如下，后续几次利用都沿用这套流程，只是初始 iovec 的大小要按想回收的目标结构的尺寸调整。  
  
第一步：用 add_key 预分配 user_key_payload，提前创建好 socketpair，创建 io_uring 实例并映射 CQ 和 SQ。  
  
第一次发送：创建并注册 provided buffer。设置 4 个 provided buffer，对应的 4 个 iovec 来自 kmalloc-64。SQE 配置成发送 provided buffer 里的数据，指定 IOSQE_BUFFER_SELECT [10]——I/O 缓冲从 provided buffer 中选取；同时必须指定 IORING_RECVSEND_BUNDLE [11]，让 send 尝试一次填满多个 buffer 而不是单个，这样才能走到 iovec 扩容路径。发出第一次 send，此时 kmsg 持有一个 4 条目的 iovec 数组。  
  
第二次发送：在 index 4 到 11 位置再设置 8 个 provided buffer，并让其中一个 buffer 的地址指向非法地址——触发 access_ok 失败，kmsg 持有的旧 iovec 对象被释放。access_ok 失败后整个 I/O 请求失败，后续清理流程会再次释放这个 iovec——double free。为什么这不会触发 CONFIG_SLAB_FREELIST_HARDENED 的 double free 检测，见 5.1 节。  
  
第三次发送：用 update_key 重新分配 key 结构，再发出第三次 send，让 iovec 数组与 user_key_payload 结构重叠：  
```
// security/keys/keyctl.c
// https://elixir.bootlin.com/linux/v6.19.3/source/security/keys/keyctl.c#L325
long keyctl_update_key(key_serial_t id,
		       const void __user *_payload,
		       size_t plen)
{
	...
	/* pull the payload in if one was supplied */
	payload = NULL;
	if (plen) {
		ret = -ENOMEM;
		payload = kvmalloc(plen, GFP_KERNEL);
		if (!payload)
			goto error;

```  
  
user_key_payload 的结构如下：  
```
// include/keys/user-type.h
// https://elixir.bootlin.com/linux/v6.19.3/source/include/keys/user-type.h#L27
struct user_key_payload {
	struct rcu_head rcu;        /* RCU destructor */
	unsigned short datalen;     /* length of this data */
	char data[] __aligned(__alignof__(u64)); /* actual data */
};

```  
  
偏移 0x10 处的 datalen 是控制 key 数据长度的元数据，它与 iovec 数组第二项地址的低 2 字节重叠。因此，在提供 provided buffer 时，事先映射一个低 16 位为 0x1000 的地址来覆写 key 结构。最后读回 key，产生越界读，泄露 KASLR。  
#### 3.2.2 泄露 task_struct 地址  
  
提权思路：泄露 task_struct，把 cred 指针覆写成 init_cred。泄露当前进程 task_struct 的地址要借助 io_ring_ctx——一个 kmalloc-2048 尺寸的对象。创建实例时设置 IORING_SETUP_SINGLE_ISSUER [12]，实例会绑定到当前 task、只允许单个 task 提交请求，task_struct 就记录在 io_ring_ctx 中：  
```
// fs/io_uring/io_uring.c
// https://elixir.bootlin.com/linux/v6.19.3/source/fs/io_uring/io_uring.c#L3065
if (ctx->flags & IORING_SETUP_SINGLE_ISSUER
    && !(ctx->flags & IORING_SETUP_R_DISABLED))
	ctx->submitter_task = get_task_struct(current);

```  
  
对这一结构做 heap spray，再走一遍同样的流程，user_key_payload 即可获得越界读。这一轮用 setxattr 与来自 kmalloc-2048 的 key 结构重叠：  
```
// fs/xattr.c
// https://elixir.bootlin.com/linux/v6.19.3/source/fs/xattr.c#L645
ctx->kvalue = vmemdup_user(ctx->cvalue, ctx->size);
if (IS_ERR(ctx->kvalue)) {
	error = PTR_ERR(ctx->kvalue);
	ctx->kvalue = NULL;
}

```  
  
当前进程的 task_struct 地址就此泄露。  
#### 3.2.3 一次性覆写 cred，实现本地提权  
  
拿到 task_struct 地址后，下一步是写 task_struct 的 cred 指针。准备流程基本相同：把 kmalloc-256 的 double free 塞进 sheaf，用 setxattr 与 msgsnd 分配出的 msg_msg 重叠。  
  
要覆写 cred 指针，就要滥用重叠的 msg_msg 做任意写。具体做法：m_list.next 写成 task_struct->real_cred，m_list.prev 写成 init_cred，security 写成一个可读地址。security 必须可读，因为 msgrcv 时 SELinux 钩子 security_msg_queue_msgrcv 会访问这个字段。  
```
// include/linux/msg.h
// https://elixir.bootlin.com/linux/v6.19.3/source/include/linux/msg.h#L9
struct msg_msg {
	struct list_head m_list;
	long m_type;
	size_t m_ts;		/* message text size */
	struct msg_msgseg *next;
	void *security;
	/* the actual message follows immediately */
};

```  
```
// ipc/msgutil.c
// https://elixir.bootlin.com/linux/v6.19.3/source/ipc/msgutil.c#L54
static struct msg_msg *alloc_msg(size_t len)
{
	struct msg_msg *msg;
	struct msg_msgseg **pseg;
	size_t alen;

	alen = min(len, DATALEN_MSG);
	msg = kmem_buckets_alloc(msg_buckets, sizeof(*msg) + alen, GFP_KERNEL);
	if (msg == NULL)
		return NULL;

```  
  
最终触发 list_del 时，cred 指针被写成 init_cred——task_struct->real_cred + 8 恰好就是 task_struct->cred——提权到 root：  
```
// include/linux/list.h
// https://elixir.bootlin.com/linux/v6.19.3/source/include/linux/list.h#L224
static inline void __list_del(struct list_head *prev, struct list_head *next)
{
	next->prev = prev;
	WRITE_ONCE(prev->next, next);
}

```  
  
**披露时间线：**  
- 2026-07-03：发现漏洞  
  
- 2026-07-06：上报  
  
- 2026-07-12：修复  
  
**相关产物：**  
- 补丁：https://git.kernel.org/pub/scm/linux/kernel/git/axboe/linux.git/commit/?h=io_uring-7.2&id=5f6a8b0648db8fbc26eeba1bfde401d1a2a8b3bf  
  
- Exploit：https://hackmd.io/@naup96321/S18WofCsVg  
  
- 演示：https://www.youtube.com/watch?v=1yv7jC3AaaI  
  
## 4 sheaf 对传统利用技术的影响与应对方案  
  
正如 3.2.1 所述，6.18 起 SLUB 机制因 sheaf/barn（见 1.2 节）发生了重大变化，传统 cross-cache 因此失效。本节分析原因，提出两个解法，并给出一条完全绕开 sheaf 缓存、直接利用旧 slab 层的思路。  
### 4.1 sheaf 机制的影响  
  
传统 cross-cache [13, 14, 15] 的做法是：大量释放对象，制造出一批已释放的 slab，其中就包括持有 UAF 对象的那些，以此耗尽对应 kmem_cache 的 partial slab 空间；buddy 系统回收这些页后，UAF 对象所在的页被另一个 cache 分走，UAF 结构便与那个 kmem_cache 的结构重叠。  
  
但 sheaf 直接改变了 slab 页的回收流程。slab 中的对象被 sheaf 缓存时，虽然逻辑上已经释放，从 slab 的视角看却仍处于使用中。这阻止了持有 UAF 对象的 slab 进入预期的回收状态——页无法被 buddy 系统回收，也就无法重新分配给别的 kmem_cache。  
  
这正是分析 provided buffer 漏洞利用流程时，传统 cross-cache 失效的主要原因：虽然能触发 UAF 对象的释放，但它所在的 slab 页始终不会被 buddy 回收，自然无法通过另一个 kmem_cache 分配对象去覆盖那块内存，预期中的 cross-cache 无从发生。  
### 4.2 方案一：把 sheaf 和 barn 中的对象刷回 slab 管理层  
  
第一个解法是在 cross-cache 流程里加一步。先把含 UAF 对象的 sheaf 从 per-CPU 层换入 barn；当 barn 中满/空 sheaf 的数量达到 MAX_FULL_SHEAVES 或 MAX_EMPTY_SHEAVES 后，它们会被刷回 slab：  
```
// mm/slub.c
// https://elixir.bootlin.com/linux/v7.1.4/source/mm/slub.c#L393
#define MAX_FULL_SHEAVES 10
#define MAX_EMPTY_SHEAVES 10

```  
```
// mm/slub.c
// https://elixir.bootlin.com/linux/v7.1.4/source/mm/slub.c#L5722
empty = barn_replace_full_sheaf(barn, pcs->main, allow_spin);
...
/* sheaf_flush_unused() doesn't support !allow_spin */
if (PTR_ERR(empty) == -E2BIG && allow_spin) {
	...
	sheaf_flush_unused(s, to_flush);
}

```  
  
__kmem_cache_free_bulk() 最终会走到 __slab_free()，把对象归还给 slab。只有到这一步，slab->inuse 才会下降，buddy 系统才可能回收这个 slab 的页：  
```
// mm/slub.c
// https://elixir.bootlin.com/linux/v7.1.4/source/mm/slub.c#L2914
static void sheaf_flush_unused(struct kmem_cache *s, struct slab_sheaf *sheaf)
{
	...
	__kmem_cache_free_bulk(s, sheaf->size, &sheaf->objects[0]);
}

```  
```
// mm/slub.c
// https://elixir.bootlin.com/linux/v7.1.4/source/mm/slub.c#L7063
static void __kmem_cache_free_bulk(struct kmem_cache *s, size_t size, void **p)
{
	...
	do {
		struct detached_freelist df;

		size = build_detached_freelist(s, size, p, &df);
		...
		__slab_free(df.s, df.slab, df.freelist, df.tail, df.cnt,
			    _RET_IP_);
	} while (likely(size));
}

```  
  
实际操作上：先分配远超 sheaf_capacity * (MAX_FULL_SHEAVES + 1) 个同 cache 的对象，再一次性全部释放，之后就能按正常流程做 cross-cache。  
### 4.3 方案二：通过跨 NUMA node 分配绕过 sheaf 缓存层  
  
另一个思路是完全绕开 sheaf 缓存，直接利用原有的 slab 机制。对象释放时，free 快路径上的 can_free_to_pcs 判断这个对象能不能进 sheaf；不能的话，就走 __slab_free 回到传统的分配/回收机制：  
```
// mm/slub.c
// https://elixir.bootlin.com/linux/v7.1.4/source/mm/slub.c#L6246
static __fastpath_inline
void slab_free(struct kmem_cache *s, struct slab *slab, void *object,
	       unsigned long addr)
{
	memcg_slab_free_hook(s, slab, &object, 1);
	alloc_tagging_slab_free_hook(s, slab, &object, 1);
	if (unlikely(!slab_free_hook(s, object, slab_want_init_on_free(s), false)))
		return;
	if (likely(can_free_to_pcs(slab)) && likely(free_to_pcs(s, object, true)))
		return;
	__slab_free(s, slab, object, object, 1, addr);
	stat(s, FREE_SLOWPATH);
}

```  
  
原始设计里，sheaf 机制本会缓存来自远端 NUMA node 的对象，但缓存远端对象会降低后续分配命中本地 NUMA node 的概率，带来额外开销 [16]。因此，来自远端 NUMA node 的对象被直接放回原来的 slab：  
```
// mm/slub.c
// https://elixir.bootlin.com/linux/v7.1.4/source/mm/slub.c#L6002
static __always_inline bool can_free_to_pcs(struct slab *slab)
{
	int slab_node;
	int numa_node;

	if (!IS_ENABLED(CONFIG_NUMA))
		goto check_pfmemalloc;
	slab_node = slab_nid(slab);
	...
	numa_node = numa_mem_id();
	if (likely(slab_node == numa_node))
		goto check_pfmemalloc;
	...
	numa_node = numa_node_id();
	/* freed object is from this cpu's node, proceed */
	if (likely(slab_node == numa_node))
		goto check_pfmemalloc;
	...
	return false;
check_pfmemalloc:
	return likely(!slab_test_pfmemalloc(slab));
}

```  
  
这个特性在有两个 NUMA node 的机器上可用：对象在第一个 NUMA node 上分配、在第二个 node 上释放时，sheaf 缓存层的 NUMA node 与对象实际所在的 node 不一致，对象不进 sheaf，而是走 __slab_free 在第三层释放。这样就轻松绕开了 sheaf 机制，继续传统的 cross-cache 流程。  
## 5 sheaf 的设计缺陷：与传统 slab 实现的不一致  
  
如 3.2.1 所述，provided buffer 漏洞的利用正是钻了 sheaf 的一个设计缺陷，避开了 CONFIG_SLAB_FREELIST_HARDENED [17] 的 double free 检测。本节从这个缺陷出发，系统梳理 sheaf 开发过程中为何没有考虑到原 slab freelist 的 CONFIG_SLAB_FREELIST_HARDENED 缓解，以及它的两道保护——double free 检测和指针保护——如何双双失效。  
### 5.1 缺陷一：绕过 CONFIG_SLAB_FREELIST_HARDENED 的 double free 检测  
  
传统 SLUB 中，对象释放后通过 set_freepointer() 更新 freelist 指针。SLUB 在写入下一个空闲对象之前，会检查当前对象是否指向自己，以此检测 double free：  
```
// mm/slub.c
// https://elixir.bootlin.com/linux/v7.1.4/source/mm/slub.c#L541
static inline void set_freepointer(struct kmem_cache *s, void *object, void *fp)
{
	unsigned long freeptr_addr = (unsigned long)object + s->offset;
#ifdef CONFIG_SLAB_FREELIST_HARDENED
	BUG_ON(object == fp); /* naive detection of double free or corruption */
#endif
	freeptr_addr = (unsigned long)kasan_reset_tag((void *)freeptr_addr);
	*(freeptr_t *)freeptr_addr = freelist_ptr_encode(s, fp, freeptr_addr);
}

```  
  
但 sheaf 存放新释放的对象时并不经过 set_freepointer()。结果是：同一个对象被反复释放时，sheaf 会直接存下重复的对象引用，后续分配时两个结构就可能共享同一块内存。此外，sheaf 缓存位于 SLUB freelist 之前，对象先由 sheaf 接管——攻击者由此拿到了可用的 double free 原语，全程不触发 SLUB freelist 的 double free 保护。  
### 5.2 缺陷二：绕过 CONFIG_SLAB_FREELIST_HARDENED 的指针保护  
  
CONFIG_SLAB_FREELIST_HARDENED 除 double free 检测外，还提供 freelist 指针保护：防止攻击者直接篡改 freelist 中存放的下一个对象指针。  
  
传统 SLUB 中，空闲对象的 next 指针不以明文存放，而是经 freelist_ptr_encode() 编码——把 next 指针与 s->random（每个 kmem_cache 初始化时生成的随机值）做异或：  
```
// mm/slub.c
// https://elixir.bootlin.com/linux/v7.1.4/source/mm/slub.c#L504
static inline freeptr_t freelist_ptr_encode(const struct kmem_cache *s,
					    void *ptr, unsigned long ptr_addr)
{
	unsigned long encoded;

#ifdef CONFIG_SLAB_FREELIST_HARDENED
	encoded = (unsigned long)ptr ^ s->random ^ swab(ptr_addr);
#else
	encoded = (unsigned long)ptr;
#endif

	return (freeptr_t){.v = encoded};
}

```  
  
sheaf 缓存层引入后，释放的对象先进入 slab_sheaf 管理的 object 数组，而 sheaf 里的对象指针没有做任何加密。现有保护就此失效：攻击者可以轻松劫持指针，进而控制 kmem_cache 分配出的内存位置。具体怎么利用，见第 6 节。  
## 6 全新技术：基于 sheaf 的利用  
  
前面几节在漏洞二的基础上，借助 sheaf 相关的部分提出了额外的提权路径，并降低了 sheaf/barn 对既有 SLUB 技术的影响。为了更深入地讨论 sheaf 机制对 Linux 内核利用的影响，本节提出三种全新的利用技术。  
  
截至 2026 年 7 月，网上关于 sheaf 相关利用的讨论和文章还相当有限。最有代表性的是 SheafJack [18]，它同样关注 sheaf 机制引入的攻击面，是少数公开讨论这个话题的先行工作。不过它提出的方向在不少细节上含糊，也存在错误描述——例如其对攻击向量 2 的描述提到通过 refill_sheaf 触发 cross-cache 流程，但核对实际源码，并不存在这一行为。本节会对基于 sheaf 的利用给出更详细、更具体的解释，并介绍新的利用技术。  
### 6.1 技术 1：对象数组劫持  
  
slab_sheaf 本质上是一个对象指针数组（详见 1.2 节）。对象释放时，内核从当前 slub_percpu_sheaves 取出 main sheaf，把对象存进去并更新 size：  
```
// mm/slub.c
// https://elixir.bootlin.com/linux/v7.1.4/source/mm/slub.c#L5806
static __fastpath_inline
bool free_to_pcs(struct kmem_cache *s, void *object, bool allow_spin)
{
	struct slub_percpu_sheaves *pcs;
	...
	if (unlikely(pcs->main->size == s->sheaf_capacity)) {
		...
	}
	pcs->main->objects[pcs->main->size++] = object;

```  
  
反过来，内核分配内存时会尝试从 main sheaf 取对象：  
```
// mm/slub.c
// https://elixir.bootlin.com/linux/v7.1.4/source/mm/slub.c#L4707
static __fastpath_inline
void *alloc_from_pcs(struct kmem_cache *s, gfp_t gfp, int node)
{
	struct slub_percpu_sheaves *pcs;
	bool node_requested;
	void *object;
	...
	object = pcs->main->objects[pcs->main->size - 1];
	...
	pcs->main->size--;
	return object;
}

```  
  
如果 slab_sheaf 里的对象数组指针被改写，SLUB 分配出的内存就能被劫持到任意想要的位置。  
```
// mm/slub.c
// https://elixir.bootlin.com/linux/v7.1.4/source/mm/slub.c#L404
struct slab_sheaf {
	union {
		struct rcu_head rcu_head;
		struct list_head barn_list;
		/* only used for prefilled sheafs */
		struct {
			unsigned int capacity;
			bool pfmemalloc;
		};
	};

	struct kmem_cache *cache;
	unsigned int size;
	int node; /* only used for rcu_sheaf */
	void *objects[];
};

```  
  
对象数组是变长的指针数组。每个 kmem_cache 都有自己的 sheaf 层；一个 slab_sheaf 从哪个 kmem_cache 分配，取决于它管理的对象大小（表 1）。实际容量取表 1 中的值与 kmem_cache 初始化时指定容量的较大者。  
  
表 1：默认 sheaf 容量与承载 struct slab_sheaf 的 kmalloc cache  
<table><thead><tr><th style="background: rgb(14, 116, 144);padding: 9px 12px;border: 1px solid rgb(56, 189, 248);font-weight: 700;color: rgb(255, 255, 255);text-align: left;"><section><span leaf="">s-&gt;size</span></section></th><th style="background: rgb(14, 116, 144);padding: 9px 12px;border: 1px solid rgb(56, 189, 248);font-weight: 700;color: rgb(255, 255, 255);text-align: left;"><section><span leaf="">Capacity</span></section></th><th style="background: rgb(14, 116, 144);padding: 9px 12px;border: 1px solid rgb(56, 189, 248);font-weight: 700;color: rgb(255, 255, 255);text-align: left;"><section><span leaf="">承载的 kmalloc cache</span></section></th></tr></thead><tbody><tr><td style="padding: 9px 12px;border: 1px solid rgb(186, 230, 253);color: rgb(30, 41, 59);vertical-align: top;"><section><span leaf="">&lt; 256</span></section></td><td style="padding: 9px 12px;border: 1px solid rgb(186, 230, 253);color: rgb(30, 41, 59);vertical-align: top;"><section><span leaf="">60</span></section></td><td style="padding: 9px 12px;border: 1px solid rgb(186, 230, 253);color: rgb(30, 41, 59);vertical-align: top;"><section><span leaf="">kmalloc-512</span></section></td></tr><tr><td style="padding: 9px 12px;border: 1px solid rgb(186, 230, 253);color: rgb(30, 41, 59);vertical-align: top;"><section><span leaf="">&gt;= 256 且 &lt; 1024</span></section></td><td style="padding: 9px 12px;border: 1px solid rgb(186, 230, 253);color: rgb(30, 41, 59);vertical-align: top;"><section><span leaf="">26</span></section></td><td style="padding: 9px 12px;border: 1px solid rgb(186, 230, 253);color: rgb(30, 41, 59);vertical-align: top;"><section><span leaf="">kmalloc-256</span></section></td></tr><tr><td style="padding: 9px 12px;border: 1px solid rgb(186, 230, 253);color: rgb(30, 41, 59);vertical-align: top;"><section><span leaf="">&gt;= 1024 且 &lt; PAGE_SIZE</span></section></td><td style="padding: 9px 12px;border: 1px solid rgb(186, 230, 253);color: rgb(30, 41, 59);vertical-align: top;"><section><span leaf="">12</span></section></td><td style="padding: 9px 12px;border: 1px solid rgb(186, 230, 253);color: rgb(30, 41, 59);vertical-align: top;"><section><span leaf="">kmalloc-128</span></section></td></tr><tr><td style="padding: 9px 12px;border: 1px solid rgb(186, 230, 253);color: rgb(30, 41, 59);vertical-align: top;"><section><span leaf="">&gt;= PAGE_SIZE</span></section></td><td style="padding: 9px 12px;border: 1px solid rgb(186, 230, 253);color: rgb(30, 41, 59);vertical-align: top;"><section><span leaf="">4</span></section></td><td style="padding: 9px 12px;border: 1px solid rgb(186, 230, 253);color: rgb(30, 41, 59);vertical-align: top;"><section><span leaf="">kmalloc-64</span></section></td></tr></tbody></table>```
// mm/slub.c
// https://elixir.bootlin.com/linux/v7.1.4/source/mm/slub.c#L7741
	if (s->size >= PAGE_SIZE)
		capacity = 4;
	else if (s->size >= 1024)
		capacity = 12;
	else if (s->size >= 256)
		capacity = 26;
	else
		capacity = 60;

	/* Increment capacity to make sheaf exactly a kmalloc size bucket */
	size = struct_size_t(struct slab_sheaf, objects, capacity);
	size = kmalloc_size_roundup(size);
	capacity = (size - struct_size_t(struct slab_sheaf, objects, 0)) / sizeof(void *);
	return max(capacity, args->sheaf_capacity);

```  
  
因此，通过大量分配/释放由 slab_sheaf 管理的对象，kmalloc-64、kmalloc-128、kmalloc-256、kmalloc-512 这四个 bucket 里的对象都能被 heap spray，最后用 UAF 劫持对象数组，得到任意读写。  
  
![图 3：技术 1，劫持 slab_sheaf 的对象数组](https://mmbiz.qpic.cn/mmbiz_png/JpU6JH8dicqWppFxMLSwkgNibwa80rAgDQzlwia5UVHVichowCU0q3tRzzGjibRhWVUwzEcd7aw2RgibSukYWXwwUqXQJibe3PKe4wHoTPc6EOjToM/640?wx_fmt=png&from=appmsg "")  
  
实际流程如下。目标是 heap spray slab_sheaf，让 UAF 对象被 slab_sheaf 回收。假设手上有一个 kmalloc-512 的 UAF，heap spray 选用 maple tree 节点：每 mmap 一个新区域，内核就会分配一个新 VMA，进而从 maple_node_cache 分配新的 maple node：  
```
// lib/maple_tree/maple_tree.c
// https://elixir.bootlin.com/linux/v7.1.4/source/lib/maple_tree/maple_tree.c#L1089
static inline void mas_alloc_nodes(struct ma_state *mas, gfp_t gfp)
{
	...
	mas->alloc = mt_alloc_one(gfp);
	if (!mas->alloc)
		goto error;

```  
  
这个 kmem_cache 初始化时指定了容量 32，因此它的 slab_sheaf 从 kmalloc-512 分配：  
```
// lib/maple_tree/maple_tree.c
// https://elixir.bootlin.com/linux/v7.1.4/source/lib/maple_tree/maple_tree.c#L5630
void __init maple_tree_init(void)
{
	struct kmem_cache_args args = {
		.align = sizeof(struct maple_node),
		.sheaf_capacity = 32,
	};
}

```  
  
heap spray slab_sheaf 时，先通过大量分配再释放，把 barn 和 slub_percpu_sheaves 中的 slab_sheaf 实例全部耗尽。之后再分配新对象时，由于现有 slab_sheaf 都没有对象可给，内核只能分配新的 slab_sheaf：  
```
// mm/slub.c
// https://elixir.bootlin.com/linux/v7.1.4/source/mm/slub.c#L4649
static struct slub_percpu_sheaves *
__pcs_replace_empty_main(struct kmem_cache *s, struct slub_percpu_sheaves *pcs, gfp_t gfp)
{
	struct slab_sheaf *empty = NULL;
	...
	if (!empty) {
		empty = alloc_empty_sheaf(s, gfp);
		if (!empty)
			return NULL;
	}

```  
  
接着用 UAF 把对象数组改写到 modprobe_path 或其他任意目标位置，让分配器把目标地址当作对象返回，在那个位置做任意读写，提权即成。  
  
为了简化验证，技术 1 和技术 2 都用一个带 UAF 的自编内核驱动来验证，代码见下方链接。  
  
**相关产物：**  
- Exploit：https://hackmd.io/@naup96321/BkNvsGqsVe  
  
- 演示：https://www.youtube.com/watch?v=Zp7y5bSBfLA  
  
### 6.2 技术 2：对象数组越界写  
  
并不总有足够长度的任意写可用；有时 UAF 对象上的写原语不完整，直接覆写对象数组很容易把内核写崩。这里再给两个不需要控制对象数组的提权方法：第一个如下，第二个见 6.3 节。  
  
再看一遍对象释放时的行为：  
```
// mm/slub.c
// https://elixir.bootlin.com/linux/v7.1.4/source/mm/slub.c#L5806
	...
	if (unlikely(pcs->main->size == s->sheaf_capacity)) {
		...
	}
	pcs->main->objects[pcs->main->size++] = object;

```  
  
当当前 main sheaf 的 size 字段等于 capacity 时，sheaf 被视为已满，不能再收对象，main sheaf 会被换掉。但如果把 slab_sheaf 的 size 字段控制成超过 capacity 的值，内核不会做任何检查——下一次释放对象时，对象指针会被写到越界位置，得到一次任意指针写。  
  
利用流程如下。先在内存里放两个带 root 权限的 cred 结构，并把 usage 字段设成很大的值，防止它们在后续利用中被意外回收。  
  
接着，把 slab_sheaf 和 task_struct 交错 heap spray：task_struct 用 fork 喷，每个子进程反复检查自己的 uid。  
  
通过破坏 size 字段，把下一次写入的偏移改到 task_struct->real_cred 字段，然后把事先排布好的对象依次释放。第一次释放：对象经 free_to_pcs 存到 objects[size]，覆写 real_cred 指针；第二次释放：存到 objects[size+1]，覆写 cred 指针。越界操作完成后，把 size 恢复原位，避免后续释放把内核写崩。  
  
循环中的子进程发现自己的 uid 变成 0（root）后，打开 root shell，提权完成。  
  
![图 4：技术 2，越过 slab_sheaf 对象数组末尾的越界写](https://mmbiz.qpic.cn/sz_mmbiz_png/JpU6JH8dicqWY3q2TayCb6IU2thrZj0h4UOHf3G79ty9V77oibfYcEicWPnKIxD1ClnZ52Tewn0XYFU5yWzMJs1OeTTP39ealadIaHt5tXRViaQ/640?wx_fmt=png&from=appmsg "")  
  
**相关产物：**  
- Exploit：https://hackmd.io/@naup96321/rkPPoG9sNx  
  
- 演示：https://www.youtube.com/watch?v=5ApPz0p_0g0  
  
### 6.3 技术 3：借助 RCU slab_sheaf 的 cross-cache  
  
这项新技术解决了传统 cross-cache 的三个问题：  
- 传统做法需要大量释放对象去填满 SLUB 的 partial freelist，让它被 buddy 系统回收；没有精确的办法知道持有 UAF 对象的页是否已经被回收。  
  
- 操作系统里噪声很多，回收了 UAF 对象的页很可能被别的 kmem_cache 拿走，利用随之失败。  
  
- 当 buddy 系统发现相邻页都已释放时，会把它们合并进更大的 free_area，目标页就更难拿回来了。  
  
新技术能把 UAF 对象精确地送进任意大小、任意类型的 kmem_cache，在更受限的环境或更弱的原语下完成提权。  
  
除了 main 和 spare，slab_sheaf 还有 rcu 槽位。对象经 kfree_rcu 释放后，rcu sheaf 会一直保存这些对象，直到宽限期结束，整捆再刷进 barn；如果 barn 已到上限，对象会被直接刷回对应 kmem_cache 的 slab：  
```
// mm/slab_common.c
// https://elixir.bootlin.com/linux/v7.1.4/source/mm/slab_common.c#L1957
void kvfree_call_rcu(struct rcu_head *head, void *ptr)
{
	unsigned long flags;
	struct kfree_rcu_cpu *krcp;
	...
	if (!IS_ENABLED(CONFIG_PREEMPT_RT) && kfree_rcu_sheaf(ptr))
		return;

	success = add_ptr_to_bulk_krc_lock(&krcp, &flags, ptr, !head);
	if (!success) {
		...
		head->func = ptr;
		head->next = krcp->head;
		WRITE_ONCE(krcp->head, head);
		atomic_inc(&krcp->head_count);
	}
	...
}
EXPORT_SYMBOL_GPL(kvfree_call_rcu);

```  
  
借助 rcu sheaf，内核不必为每个对象都创建 RCU callback，可以一次处理一大批 RCU 对象。call_rcu 注册的 callback rcu_free_sheaf 在宽限期结束后被调用，开始清理 rcu sheaf 中的对象：  
```
// mm/slub.c
// https://elixir.bootlin.com/linux/v7.1.4/source/mm/slub.c#L5974
bool __kfree_rcu_sheaf(struct kmem_cache *s, void *obj)
{
	...
	rcu_sheaf->objects[rcu_sheaf->size++] = obj;
	...
	if (rcu_sheaf)
		call_rcu(&rcu_sheaf->rcu_head, rcu_free_sheaf);
}

```  
  
对象放进哪个 barn、刷回哪个 kmem_cache，都依据 rcu slab_sheaf 自己的 cache 字段来查；用哪个 kmem_cache 的 barn，也由这个字段决定：  
```
// mm/slub.c
// https://elixir.bootlin.com/linux/v7.1.4/source/mm/slub.c#L5956
static void rcu_free_sheaf(struct rcu_head *head)
{
	struct slab_sheaf *sheaf;
	struct node_barn *barn = NULL;
	struct kmem_cache *s;

	s = sheaf->cache;
	...
	barn = get_barn_node(s, sheaf->node);
	...
	if (data_race(barn->nr_full) < MAX_FULL_SHEAVES) {
		barn_put_full_sheaf(barn, sheaf);
		return;
	}
	...
flush:
	stat(s, BARN_PUT_FAIL);
	sheaf_flush_unused(s, sheaf);
}

```  
  
有了 UAF 原语，劫持 cache 字段，再大量 heap spray RCU 对象，就能在 RCU callback 里把 UAF 对象刷进任意的 kmem_cache。  
  
这是一条全新的 cross-cache 路径：不依赖传统上经由 buddy 系统把 UAF 对象搬进另一个内存池的做法，减少了 buddy 系统内部状态变化带来的不确定性，整条攻击流程更稳定；也不需要 4.2 节那种复杂的释放流程，更不依赖 4.3 节的 NUMA node 操作，整体执行流程更简洁，可移植性更好。  
  
![图 5：技术 3，把 RCU sheaf 重定向进任意 kmem_cache](https://mmbiz.qpic.cn/mmbiz_png/JpU6JH8dicqU30daRekkQUiakeHTRk4lux4N4BYLDGPa2WBZ071YukZteZQdKKHsQNgYmQNnzY3s8PX6CU2ibicg6GdzMAuVMVcsRSJOEnj9UjQ/640?wx_fmt=png&from=appmsg "")  
### 6.4 把技术 3 应用到漏洞二  
  
这里把技术 3 应用到 provided buffer 漏洞上，给出一条新的提权路径。  
#### 6.4.1 三重 free：信息泄露与 RCU slab_sheaf 重叠  
  
目标是破坏一个 kmalloc-192 管理的对象，而它的 slab_sheaf 来自 kmalloc-512，所以接下来的操作横跨这两个 kmem_cache。  
  
第一步：除了预分配 user_key_payload、之后用 update key 分配对象之外，还要提前创建大量 IPv6 多播 socket，供后续 heap spray RCU slab_sheaf 使用。  
  
然后照常触发漏洞，但这次把 double free 做两遍，在 sheaf 上留下三个指向同一地址的指针备用。  
  
之后，从 kmalloc-512 分配的 user_key_payload 与 iovec 结构重叠，覆写 datalen 得到越界读。这次越界读主要用于观察 sheaf 的状态、泄露 cred_jar 的地址；留下的第三个指针，留给之后新分配的 sheaf 使用。  
  
接着大量分配并释放 kmalloc-192 的对象，把对应 kmem_cache 的 main 和 barn 中的空闲 sheaf 全部耗尽。这会迫使内核在 RCU 释放 kmalloc-192 对象、把它放进 rcu slab_sheaf 时，通过 alloc_empty_sheaf 申请新内存；新 sheaf 会拿走上面留下的第三个指针，让 iovec 与 user_key_payload 重叠。占用是否成功，看读 user_key_payload 时返回的数据长度——那个值会被分配过去的 slab_sheaf 压小。  
  
rcu sheaf 的 heap spray 用 IPv6 多播 socket 完成。MCAST_MSFILTER 是过滤多播源的 API。为了支持多播源过滤器在旧版与新版之间安全切换，内核用到了 RCU，并在结尾调用 kfree_rcu：  
```
// net/ipv6/mcast.c
// https://elixir.bootlin.com/linux/v7.1.4/source/net/ipv6/mcast.c#L574
int ip6_mc_msfilter(struct sock *sk, struct group_filter *gsf,
		    struct sockaddr_storage *list)
{
	struct ip6_sf_socklist *psl;
	...
	kfree_rcu(psl, rcu);
	return err;
}

```  
#### 6.4.2 泄露 cred kmem_cache 的地址  
  
再次更新 iovec，恢复 datalen，重新获得信息泄露能力。通过 fork 子进程 heap spray 并释放 cred，迫使内核从 cred 的 kmem_cache 分配 slab_sheaf，让 slab_sheaf->cache 出现在内存中。该结构的地址和字段出现在 user_key_payload 附近，借助信息泄露拿到 cred_jar 的位置。cred 虽然也是 kmalloc-192 的尺寸，但它是一个独立于通用 cache 池的 kmem_cache，不会影响前面制造的状态。  
```
// kernel/cred.c
// https://elixir.bootlin.com/linux/v7.1.4/source/kernel/cred.c#L36
static struct kmem_cache *cred_jar;

```  
```
// kernel/cred.c
// https://elixir.bootlin.com/linux/v7.1.4/source/kernel/cred.c#L535
void __init cred_init(void)
{
	/* allocate a slab in which we can store credentials */
	cred_jar = KMEM_CACHE(cred, SLAB_HWCACHE_ALIGN | SLAB_PANIC | SLAB_ACCOUNT);
}

```  
#### 6.4.3 污染 RCU cache、cross-cache 攻击 cred_jar，完成提权  
  
再用前面的方法制造 UAF 对象。重叠的两份里，一份是 iovec，用来写对象；另一份让 IPv6 多播 socket 的 ip6_sf_socklist 拿走。RCU 宽限期结束后，UAF 对象被放进先前与其他对象重叠过的 RCU slab_sheaf。  
  
接下来把 RCU slab_sheaf->cache 改写成 cred_jar。  
  
然后做大量 heap spray，让 ip6_sf_socklist 把 RCU slab_sheaf 填满。sheaf 一满就会被换进 barn；由于所属 cache 已被污染，它会被放进 cred 的 barn——UAF 对象就此 cross-cache 进入了另一个 kmem_cache。  
  
最后，通过大量 fork 子进程去争夺进入 cred_jar 的 UAF 对象，并用 iovec 主动写它。每个子进程在循环里持续检查自己的 uid 或 gid 是否被覆写，最终完成提权。  
  
**相关产物：**  
- Exploit：https://hackmd.io/@naup96321/HJxFOv9s4e  
  
- 演示：https://www.youtube.com/watch?v=W8BF8KlTKQw  
  
## 7 结论  
  
本文发现了 io_uring 中的两个 0-day，均已上报上游、修复并发布补丁。  
  
第一个漏洞出在 CQ/SQ ring resize 特性与延迟 task work 机制之间：ring resize 期间对 ctx->rings 的更新，与修改 ring 标志的 task work 操作没有持有恰当的锁、缺乏足够同步，race condition 可能导致 null pointer dereference。  
  
第二个漏洞出在 provided buffer bundle 特性的错误处理流程：内核尝试扩容 iovec 数组后，若 provided buffer 中存在非法地址，回滚路径上 kmsg 自身的 iovec 指针没有得到正确更新，导致 UAF。我们基于它构建了完整的利用链，在开启多项现代防护的环境下实现了本地提权。  
  
在利用开发过程中，我们发现 6.18 之后引入的 SLUB sheaf/barn 在原有 slab 机制之上新增了两层缓存（per-CPU 层与 NUMA 层），显著改变了 SLUB 传统的对象管理流程，许多过去内核利用中广泛使用的技巧在 sheaf 引入后失效。以 cross-cache 攻击为例：释放的对象可能长时间停留在新的缓存层中，slab 自身会把对象标记为仍在使用，导致其无法被 buddy 系统回收。我们给出两个解法：  
- 通过 heap spray 更多对象，把大量 per-CPU sheaf 换入 barn。当 barn 中满/空 sheaf 达到上限后，它们会被刷回 slab，之后就能按原有流程完成 cross-cache。  
  
- 在有多个 NUMA node 的环境下，把当前进程绑定到一个 NUMA node 上分配、在另一个 node 上释放。sheaf 不缓存来自远端 NUMA node 的对象（那会降低后续分配命中本地 NUMA node 的概率，是性能问题），由此可以直接利用原有的 slab 机制。  
  
深入研究后我们还发现，sheaf/barn 的设计让原有 SLUB 的 SLAB_FREELIST_HARDENED 缓解几乎完全失效。该缓解原本提供两道保护：  
- double free 检查：对象挂入 freelist 时会检查该对象是否已在链上，以此发现重复释放。这条检查在 sheaf 快路径上从未使用；正常情况下对象总是先被 sheaf/barn 接管，因此对同一对象做简单的 double free 放进 sheaf，就足以支撑后续利用。  
  
- 指针保护：freelist 是链表，每个被释放的对象通过指针指向下一个；该保护在 kmem_cache 创建时生成随机值，对指针做异或编码。sheaf 机制里，slab_sheaf 使用对象指针数组，分配/释放按当前 sheaf 持有的对象数（size）存取。对象指针数组不受指针加密保护，因此只需简单 heap spray slab_sheaf、篡改对象数组指针，就能轻松劫持对象的分配与释放。  
  
更进一步，sheaf/barn 带来的不只是麻烦，还有更大的攻击面。我们提出三种新技术：  
- 对象数组劫持：利用 slab_sheaf->objects[] 不受指针保护覆盖的事实，直接控制 SLUB 分配器在任意位置返回对象。  
  
- 对象数组越界写：利用可控制的 sheaf 元数据（size 字段）对对象指针数组做越界写。通过堆布局把 task_struct 与 slab_sheaf 排成特定结构，篡改 size 即可实现一次任意指针写；指针指向的内容事先填好带 root 权限的 cred，用这次任意指针写覆写 task_struct->cred 完成提权。  
  
- RCU sheaf cross-cache：传统 cross-cache 依赖 buddy 系统在 partial freelist 达到上限后回收无用页（slab），把 UAF 对象搬进另一个 kmem_cache。本技术利用 RCU 在宽限期内把 rcu 对象攒进 slab_sheaf、一次性为多个对象创建 callback 的机制：宽限期结束后，slab_sheaf 中的对象被刷回 barn 或 slab，而 slab_sheaf->cache 记录着这些对象来自哪个 cache——劫持这份元数据，就能更灵活、更稳定地把对象放进任意 cache 池。相比传统做法，它不依赖 buddy 系统回收页面，避开了 buddy 内部状态和不同 kmem_cache 争抢页面带来的不确定性。  
  
最后，我们基于新的 cross-cache 技术，为第二个漏洞构建了一条新的利用链。  
  
总结：本文发现了 io_uring 中两个新的内存安全漏洞，并为其中一个构建了完整的提权路径；以此为起点，深入研究了新的 SLUB sheaf/barn 分配器如何影响既有的内核利用技术，并给出了解法；更进一步，我们把 sheaf/barn 本身当作攻击面，分析了其设计缺陷如何令原有 SLUB 缓解失效，并提出了三种新的利用技术。其中基于 RCU sheaf 的 cross-cache 打破了传统 cross-cache 对 buddy 系统页面回收的依赖，让攻击者能更灵活、更稳定地在 cache 池之间搬运对象。希望这些发现能为未来的 Linux 内核利用研究提供新的方向和攻击面。  
## 译者解读  
  
**sheaf/barn 到底改了什么。**  
 传统 SLUB 的对象管理只有两层：per-CPU 的 kmem_cache_cpu 和 NUMA 级的 kmem_cache_node。sheaf/barn 在中间插入了一套「对象指针数组」式的缓存：释放的对象先攒进 per-CPU 的 main sheaf，main 满了换进 barn 保管，只有 barn 攒到上限才刷回 slab。关键就在「最后才回 slab」——对象在逻辑上已经 free，slab 却认为它还在用，slab->inuse 不降，buddy 系统永远收不走这页，传统 cross-cache 的地基没了。4.2/4.3 的两个解法，本质都是想办法把对象「推」回 slab 管理层：前者靠堆量把 barn 填到上限触发回刷，后者干脆利用 NUMA 策略让对象绕开 sheaf。  
  
**为什么 SLAB_FREELIST_HARDENED 会失效。**  
 这套加固是为 freelist 链表设计的：double free 靠 set_freepointer() 里的自检发现，指针篡改靠 XOR 编码挡住。而 sheaf 用的是明文对象数组，对象先被 sheaf 接管，根本走不到 freelist，两道保护都盖不到新路径。教训很直接：缓解措施必须跟着数据结构走，新结构不接入旧缓解，等于旧门上了锁、新门敞开着。  
  
**技术 3 的巧思。**  
 kfree_rcu 的对象会攒进 rcu sheaf，宽限期结束后按 sheaf->cache 字段决定这捆对象归还到哪个 cache 的 barn。劫持这个字段，就让 RCU 回调替攻击者把 UAF 对象「搬运」到任意 cache：不需要 buddy 系统拆页重分，不受页面合并、噪声分配的干扰，比传统 cross-cache 稳定得多。6.4 把它落到漏洞二上：先靠越界读泄露 cred_jar 地址，再污染 cache 字段，把 UAF 对象直接送进 cred 的专用 cache，最后 fork 子进程抢到它覆写凭据。传统 cross-cache 的三大不确定性——页是否回收、页被谁抢走、页被合并——在这条路径上全部消失。  
  
**两个漏洞的共同模式。**  
 漏洞一出在 resize rings 这个新特性与 task work 的锁配合上，漏洞二出在 bundle 扩容这个新特性的错误回滚路径上，共同点是新接口的同步设计和错误处理没有跟上功能本身。io_uring 这几年持续被挖，这次的两个漏洞也不在核心排队逻辑里，而在后加的特性上——一个错在锁的配合，一个错在回滚路径。  
  
**防御视角。**  
 这篇论文反过来也是一份加固检查表。对应到具体修复：sheaf 快路径应补上与 set_freepointer() 等价的 double free 校验；对象数组指针应纳入编码或完整性保护；rcu_free_sheaf 在按 cache 字段归还前应校验字段合法性。每补上一项，论文里的利用链就断一环。  
## 参考  
- [1] Faster IO through io_uring (Kernel Recipes 2019)：https://www.slideshare.net/slideshow/kernel-recipes-2019-faster-io-through-iouring/176705140  
  
- [2] Ensure usable io_uring interface：https://lists.openwall.net/linux-kernel/2019/03/10/405  
  
- [3] [GIT PULL] io_uring fixes for 5.2-rc：https://lists.openwall.net/linux-kernel/2019/05/27/275  
  
- [4] [PATCH v8 00/23] SLUB percpu sheaves：https://lists.openwall.net/linux-kernel/2025/09/10/485  
  
- [5] [PATCH RFC v2 00/20] slab: replace cpu (partial) slabs with sheaves：https://patchew.org/linux/20260112-sheaves-for-all-v2-0-98225cfb50cf@suse.cz/  
  
- [6] io_uring_setup man page：https://man7.org/linux/man-pages/man2/io_uring_setup.2.html  
  
- [7] io_uring_enter man page：https://man7.org/linux/man-pages/man2/io_uring_enter.2.html  
  
- [8] IORING_REGISTER_PBUF_RING man page：https://man7.org/linux/man-pages/man2/io_uring_register.2.html  
  
- [9] mm: memcg: add per-memcg kmalloc pools：https://lists.openwall.net/linux-kernel/2023/11/02/322  
  
- [10] IORING_OP_SEND man page：https://man7.org/linux/man-pages/man2/io_uring_enter.2.html  
  
- [11] io_uring: add IORING_RECVSEND_BUNDLE support：https://git.kernel.org/pub/scm/linux/kernel/git/axboe/linux-block.git/commit/?id=47b9d0f50638  
  
- [12] IORING_SETUP_SINGLE_ISSUER man page：https://man7.org/linux/man-pages/man2/io_uring_setup.2.html  
  
- [13] Linux Kernel Exploitation and Linux Kernel heap feng shui：https://duasynt.com/blog/linux-kernel-heap-feng-shui  
  
- [14] Linux kernel heap quarantine：https://a13xp0p0v.github.io/2020/11/30/linux-kernel-heap-quarantine.html  
  
- [15] GTSG CTF 2021 - Fullmetal Alchemist, Kernel UAF + ROP：https://www.kileak.com/ctf/2021/tsgctf/full-metal/  
  
- [16] percpu: free remote percpu page：https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=1a5d0a0c1d4e  
  
- [17] slub: add support for per-cache freelist hardening：https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=2482ddec670f  
  
- [18] SheafJack: Advanced Kernel Exploitation in Linux 6.18+ via slab_sheaf Hijack (Blue Dragon Security Research Lab, 2026)：https://www.slideshare.net/slideshow/sheafjack-advanced-kernel-exploitation-in-linux-6-18-via-slab-sheaf-hijack/287245786  
  
