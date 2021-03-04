# cnpm在vscode报错：无法加载文件
新办公电脑 vscode内置终端使用`cnpm`时候报错 提示
>  cnpm: 无法加载文件

## 解决办法 
管理员模式运行cmd 
如果是win10 直接右键左下角的win-logo — 命令提示符(管理员)

输入命令: `set-ExecutjionPolicy RemoteSigned`
会显示目前执行的策略 是N 直接改成`A` 开放所有策略

就可以了
(完)