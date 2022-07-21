# RCU

## updater

在对copy的数据更新完成后，需要通过 `rcu_assign_pointer()`，用这个copy替换原节点在链表中的位置，并移除对原节点的引用，而后调用`synchronize_rcu()`或`call_rcu()`进入grace period。

因为`synchronize_rcu()`会阻塞等待，所以只能在进程上下文中使用，而`call_rcu()`可在中断上下文中使用。

~~~ c
rcu_assign_pointer 实质是 wmb
~~~

## reader

在调用 `rcu_read_lock()` 进入临界区后，因为所使用的节点可能被updater解除引用，因而需要通过 `rcu_dereference()` 保留一份对这个节点的指针指向。进入grace period意味着数据已经更新，而这些reader在退出临界区之前，只能使用旧的数据，也就是说，它们需要暂时忍受“过时”的数据，不过这在很多情况下是没有多大影响的。

~~~ c
rcu_read_lock
rcu_read_unlock
rcu_dereference 引用内容
~~~

## reclaimer

对于所有进入**grace period**之前就进入临界区的reader，需要等待它们都调用了 `rcu_read_unlock()` 退出临界区，之后**grace period**结束，原节点所在的内存区域被释放。

`rcu_read_lock` & `rcu_read_unlock` 设立**grace period**临界区。

> 参考

https://lwn.net/Articles/609973/
