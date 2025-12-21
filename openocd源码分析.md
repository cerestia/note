# openocd 源码分析
commit id:6f84e90d5cc0e98f895e
本文分析openocd源码。
## 源码分析

## 主流程分析
openocd_main函数为主函数,主要流程为
    1.tcl命令注册
    2.运行openocd_thread函数，解析函数，连接被调试对象
    3.资源的回收与释放
openocd_main函数的内容不多，主要功能在openocd_thread。下面分析这个函数.
1.解析命令行参数
2.解析配置文件参数
3.执行服务初始
4.如果有init命令，运行init命令
5.开始主体循环，接受命令
```c
static int openocd_thread(int argc, char *argv[], struct command_context *cmd_ctx)
{
	int ret;

	if (parse_cmdline_args(cmd_ctx, argc, argv) != ERROR_OK)
		return ERROR_FAIL;

	if (server_preinit() != ERROR_OK)
		return ERROR_FAIL;

	ret = parse_config_file(cmd_ctx);
	if (ret == ERROR_COMMAND_CLOSE_CONNECTION) {
		server_quit(); /* gdb server may be initialized by -c init */
		return ERROR_OK;
	} else if (ret != ERROR_OK) {
		server_quit(); /* gdb server may be initialized by -c init */
		return ERROR_FAIL;
	}

	ret = server_init(cmd_ctx);
	if (ret != ERROR_OK)
		return ERROR_FAIL;

	if (init_at_startup) {
		ret = command_run_line(cmd_ctx, "init");
		if (ret != ERROR_OK) {
			server_quit();
			return ERROR_FAIL;
		}
	}

	ret = server_loop(cmd_ctx);

	int last_signal = server_quit();
	if (last_signal != ERROR_OK)
		return last_signal;

	if (ret != ERROR_OK)
		return ERROR_FAIL;
	return ERROR_OK;
}

```
## 配置文件与命令解析
openocd使用jim-tcl解析命令。命令参数解析流程：
    1.通过setup_command_handler函数注册命令
    2.在openocd_thread函数中解析命令行参数与配置文件参数
    3.注销注册的命令,释放内存
## 读取寄存器
## 读取内存
## 与gdb交互
## flash命令
