## licence启动服务

lmgrd 启动licence服务

lmdown 关闭licence服务

lmreread 重新读取修改的licence

## 更改网络MAC地址为licence中的地址

licence中地址为000C29597134

## 保持licence中hostname与虚拟机中的hostname一致

命令：hostname

查看虚拟机的hostname

## LINUX系统设置文件夹快捷方式

```scheme{.line-numbers}
Pwd；获取文件路径
Ln -s 原路径 新路径（桌面路径） 
```

## AMD和sentaurus不兼容的办法

使用AMD处理器进行sdevice仿真时，可能出现如下错误导致仿真中断：

```
Error: Child process with pid '***' got the signal 'SIGSEGV' (segmentation violation)
gjob exits with status 1
```

解决办法：添加环境变量

```scheme{.line-numbers}
export MKL_DEBUG_CPU_TYPE=5
```

具体步骤：

```
su    	进入管理员模式
	输入密码
gedit /etc/bashrc  进入编辑页面，在最后添加，保存并关闭
```
