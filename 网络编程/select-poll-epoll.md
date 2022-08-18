# select/poll/epoll/kqueue

[12](ftp://192.168.31.180:2121/)

![1658637650827](C:\Users\xlp\AppData\Roaming\Typora\typora-user-images\1658637650827.png)



Linux kernel源码官网：https://www.kernel.org/

由于官网下载速度太慢，需要科学上网，因此可以通过连接http://ftp.sjtu.edu.cn/sites/ftp.kernel.org/pub/linux/kernel/下载。









## Epoll

LOCKING:

epoll需要三种等级的所定：

1. epmutex (mutex)
2. ep->mtx (mutex)
3. ep->lock (rwlock)

The acquire order is the one listed above, from 1 to 3. We need a rwlock (ep->lock) because we manipulate objects from inside the poll callback, that might be triggered from a wake_up() that in turn might be called from IRQ context. So we can't sleep inside the poll callback and hence we need a spinlock. During the event transfer loop (from kernel to user space) we could end up sleeping due a copy_to_user(), so we need a lock that will allow us to sleep. This lock is a mutex (ep->mtx). It is acquired during the event transfer loop, during epoll_ctl(EPOLL_CTL_DEL) and during eventpoll_release_file(). Then we also need a global mutex to serialize eventpoll_release_file() and ep_free().

获取顺序为上面列出的顺序，从1到3。我们需要rwlock (ep-&gt;锁)，因为我们从轮询回调内部操作对象，它可能由一个wake_up()触发，而这个wake_up()又可能从IRQ上下文中调用。所以我们不能在poll回调中睡觉，因此我们需要一个自旋锁。在事件传输循环期间(从内核到用户空间)，由于copy_to_user()，我们可能会进入睡眠状态，因此我们需要一个允许我们进入睡眠状态的锁。这个锁是一个互斥锁(ep-&gt;mtx)。它是在事件传输循环期间，在epoll_ctl(EPOLL_CTL_DEL)和在eventpoll_release_file()期间获得的。然后我们还需要一个全局互斥锁来序列化eventpoll_release_file()和ep_free()。

This mutex is acquired by ep_free() during the epoll file cleanup path and it is also acquired by eventpoll_release_file() if a file has been pushed inside an epoll set and it is then close()d without a previous call to epoll_ctl(EPOLL_CTL_DEL). It is also acquired when inserting an epoll fd onto another epoll fd. We do this so that we walk the epoll tree and ensure that this insertion does not create a cycle of epoll file descriptors, which could lead to deadlock. We need a global mutex to prevent two simultaneous inserts (A into B and B into A) from racing and constructing a cycle without either insert observing that it is going to.

该互斥量由ep_free()在epoll文件清理路径中获取，如果文件被推送到epoll集中，它也由eventpoll_release_file()获取，然后关闭()d，不需要之前调用epoll_ctl(EPOLL_CTL_DEL)。在将epoll fd插入到另一个epoll fd时也会获得它。这样做是为了遍历epoll树，并确保这次插入不会创建epoll文件描述符的循环，这可能会导致死锁。我们需要一个全局互斥锁来防止两个同步插入(a插入B和B插入a)在没有任何一个插入观察到它将要进行的情况下相互竞争和构建一个循环。

It is necessary to acquire multiple "ep->mtx"es at once in the case when one epoll fd is added to another. In this case, we always acquire the locks in the order of nesting (i.e. after epoll_ctl(e1, EPOLL_CTL_ADD, e2), e1->mtx will always be acquired before e2->mtx). Since we disallow cycles of epoll file descriptors, this ensures that the mutexes are well-ordered. In order to communicate this nesting to lockdep, when walking a tree of epoll file descriptors, we use the current recursion depth as the lockdep subkey.

当一个epoll fd被添加到另一个fd时，需要同时获得多个“ep-&gt;mtx”es。在本例中，我们总是按照嵌套的顺序获取锁(即在epoll_ctl(e1, EPOLL_CTL_ADD, e2)之后，e1-&gt;mtx总是在e2-&gt;mtx之前)。由于我们不允许epoll文件描述符循环，这确保互斥锁是有序的。为了将这个嵌套传递给lockdep，在遍历epoll文件描述符树时，我们使用当前的递归深度作为lockdep的子键。

It is possible to drop the "ep->mtx" and to use the global mutex "epmutex" (together with "ep->lock") to have it working, but having "ep->mtx" will make the interface more scalable. Events that require holding "epmutex" are very rare, while for normal operations the epoll private "ep->mtx" will guarantee a better scalability.

可以删除“ep->mtx”并使用全局互斥锁“epmutex”(与“ep-&gt;lock”一起使用)来使其工作，但使用“ep-&gt;mtx”将使接口更具可伸缩性。需要持有“epmutex”的事件非常罕见，而对于普通操作，epoll私有的“ep-&gt;mtx”将保证更好的可伸缩性。











```c
/* Wait structure used by the poll hooks */轮询钩子使用的等待结构
struct eppoll_entry {
	/* List header used to link this structure to the "struct epitem" */用于将该结构链接到“struct epitem”的列表头
	struct eppoll_entry *next;

	/* The "base" pointer is set to the container "struct epitem" */
	struct epitem *base;

	/*
	 * Wait queue item that will be linked to the target file wait queue head.
	 * 
	 */
	wait_queue_entry_t wait;

	/* The wait queue head that linked the "wait" wait queue item */
	wait_queue_head_t *whead;
};
```



每当调用epoll_ctl增加一个fd时，内核就会为我们创建一个epitem实例



```c
/*
 * Each file descriptor added to the eventpoll interface will have an entry of this type linked to the "rbr" RB tree.
 * 添加到eventpoll接口的每个文件描述符都有一个链接到“rbr”RB树的这种类型的条目。
 * Avoid increasing the size of this struct, there can be many thousands of these on a server and we do not want this to take another cache line.
 * 避免增加这个结构的大小，服务器上可能有成千上万个这样的结构，我们不希望它占用另一条缓存线。
 */
struct epitem {
	union {
		/* RB tree node links this structure to the eventpoll RB tree */
		struct rb_node rbn;
		/* Used to free the struct epitem */
		struct rcu_head rcu;
	};

	/* List header used to link this structure to the eventpoll ready list */
	struct list_head rdllink;

	/*
	 * Works together "struct eventpoll"->ovflist in keeping the single linked chain of items.
	 * 
	 */
	struct epitem *next;

	/* The file descriptor information this item refers to */
	struct epoll_filefd ffd;

	/* List containing poll wait queues */
	struct eppoll_entry *pwqlist;

	/* The "container" of this item */
	struct eventpoll *ep;

	/* List header used to link this item to the "struct file" items list */
	struct hlist_node fllink;

	/* wakeup_source used when EPOLLWAKEUP is set */
	struct wakeup_source __rcu *ws;

	/* The structure that describe the interested events and the source fd */
	struct epoll_event event;
};
```





```c
/*
 * This structure is stored inside the "private_data" member of the file structure and represents the main data structure for the eventpoll interface.
 * 该结构存储在文件结构的“private_data”成员中，并表示eventpoll接口的主要数据结构。
 * 
 */
struct eventpoll {
	/*
	 * This mutex is used to ensure that files are not removed while epoll is using them. This is held during the event collection loop, the file cleanup path, the epoll file exit code and the ctl operations.
	 * 这个互斥锁用于确保文件在epoll使用时不会被删除。这在事件收集循环、文件清理路径、epoll文件退出代码和ctl操作期间保持。
	 * 
	 * 
	 */
	struct mutex mtx;

	/* Wait queue used by sys_epoll_wait() */sys_epoll_wait()使用的等待队列
	wait_queue_head_t wq;

	/* Wait queue used by file->poll() */file->poll()使用的等待队列
	wait_queue_head_t poll_wait;

	/* List of ready file descriptors */准备好的文件描述符列表
	struct list_head rdllist;

	/* Lock which protects rdllist and ovflist */保护rdllist和ovfllist的锁
	rwlock_t lock;

	/* RB tree root used to store monitored fd structs */RB树根用于存储被监控的fd结构
	struct rb_root_cached rbr;

	/*
	 * This is a single linked list that chains all the "struct epitem" that happened while transferring ready events to userspace w/out holding ->lock.
	 * 这是一个单一的链接列表，将所有发生的"struct epitem"转移到用户空间w/out持有->lock。
	 */
	struct epitem *ovflist;

	/* wakeup_source used when ep_scan_ready_list is running */当ep_scan_ready_list运行时使用的wakeup_source
	struct wakeup_source *ws;

	/* The user that created the eventpoll descriptor */创建事件轮询描述符的用户
	struct user_struct *user;

	struct file *file;  //这是eventpoll对应的匿名文件

	/* used to optimize loop detection check */用于优化环路检测检查
	u64 gen;
	struct hlist_head refs;

#ifdef CONFIG_NET_RX_BUSY_POLL
	/* used to track busy poll napi_id */用于跟踪繁忙的投票napi_id
	unsigned int napi_id;
#endif

#ifdef CONFIG_DEBUG_LOCK_ALLOC
	/* tracks wakeup nests for lockdep validation */跟踪用于locklock验证的唤醒巢
	u8 nests;
#endif
};
```

