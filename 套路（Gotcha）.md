# 套路（Gotcha）
这里列出一些常用套路。

- 在unlink请求处理方法中删除INodes

	假如文件系统挂在在“/mnt”，如下代码堪称完美：

	with open("/mnt/file_one", "w+") as fh1:

		fh1.write("what")
		fh1.flush()
		with open("/mnt/file_one", "a") as fh2:
			os.unlink("/mnt/file_one")
			assert "file_one" not in os.listdir("/mnt")
			fh2.write("fuck")
		os.close(os.dup(fh1.fileno()))
		fh1.seek(0)
		assert fh1.read() == "whatfuck"

	如果出错，很可能实现的unlink请求处理方法有误、应当延迟（defer）删除在forget请求处理中清除文件内容。