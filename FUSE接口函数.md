# FUSE接口函数
- llfuse.init(ops, mountpoint, options=default_options)
  - 初始化并挂在文件系统。
  - ops：必须是llfuse.Operations的（子类）实例（或鸭子对象）。
  - args：必须是一组字符串。default_options提供一些合理的默认值，建议以此为基础根据需要增减配置，例如：

	my_opts = set(llfuse.default_options)

	my_opts.add("allow_other")

	my_opts.discard("default_permissions")

	llfuse.init(ops, mountpoint, my_opts)

  - 有效的options在“[struct fuse_opt fuse_mount_opts[]][0]”和“[struct fuse_opt fuse_ll_opts[]][1]”。

- llfuse.main(workers=None)
  - 运行FUSE主循环。
  - workers指定并发处理请求的线程数量。如果workers是None，LLFUSE会选一个合理的大于一的数；如果workers是一，所有请求将被调用llfuse.main的线程处理。
  - 该函数也会为内部用途启动额外的线程（即使workers设置为一）。这些（全部worker线程）在llfuse.main返回时保证结束。
  - 当该函数运行时，专门信号处理方法将会为SIGTERM/SIGINT(CTRL+C)/SIGHUP/SIGUSR1/SIGPIPE信号安装。SIGPIPE将被忽略，SIGINT将不会导致KeyboardInterrupt异常，其余三个会引发处理停止和该函数返回的请求。

- llfuse.close(unmount=True)
  - 清理并保证文件系统卸载。
  - 如果unmount是False，仅执行清理，文件系统并未明确卸载。
  - 通常，文件系统通过用户调用unmount(8)或fusermount(1)来卸载，这会终止FUSE的主循环。然而，也会被异常或信号终止。在后者这种情况下，文件系统维持挂载，但是任何访问尝试都被阻塞（在文件系统进程仍然运行）或返回错误（在文件系统进程终止后）。
  - 如果unmount是True，该函数会确保文件系统正确的卸载。
  - 注意：如果通过“/sys/fs/fuse/connections/”接口终止到内核的连接，该函数将不会卸载文件系统，即使unmount是True。

- llfuse.invalidate_inode(fuse_info_t inode, attr_only=False)
  - 验证INode缓存。
  - 通知（instruct）FUSE内核模块忘记（forgot）inode已缓存的属性和数据（除非attr_only是True）。
  - 该操作异步执行，就是**该函数可能先于内核完成请求处理而返回**。

- llfuse.invalidate_entry(fuse_ino_t inode_p, name)
  - 验证目录入口。
  - 通知FUSE内核模块忘记关联inode_p的目录下的目录入口name。
  - 该操作异步。

- llfuse.notify_store(inode, offset, data)
  - 保存内核页缓存的数据。
  - 发送数据给内核保存到inode的页缓存的offset的位置。
  - 如果提供的数据超出当前文件大小，该文件自动扩容。
  - 如果该函数抛出异常，部分数据仍有可能保存完成。

- llfuse.get_ino_t_bits()
  - 返回INode编号可用的**比特数**。
  - 尝试使用需要更多字节的INode值会导致OverflowError。

- llfuse.get_off_t_bits()
  - 返回文件偏移可用的**字节数**。
  - 尝试使用需要更多字节表达的值会导致OverflowError。

---
[0]: https://github.com/libfuse/libfuse/blob/master/lib/mount.c#L82
[1]: https://github.com/libfuse/libfuse/blob/master/lib/fuse_lowlevel.c#L2626