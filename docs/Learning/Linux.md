Linux 学习文档

基础指令可以转到 [Tool](./ToolDoc.md)

为什么安装软件前要先进行`apt-get update` 官网地址[apt-get(8) - Linux man page](https://linux.die.net/man/8/apt-get)

在执行`upgrade`或者`install`时, 系统会从包列表`/etc/apt/sources.list`中找到包索引,然后根据索引进行下载, 这个包列表可能会随着时间变化,以致你在install时找不到包索引或者无法下载成功

`apt-get` 用于从源文件重新同步包索引文件。可用包的索引从/etc/apt/sources.list中指定的位置获取。应始终在升级或版本升级之前执行更新。

并且`apt`可以作为`apt-get`的上位替代. [APT 与 APT-GET 之间有什么区别？](https://aws.amazon.com/cn/compare/the-difference-between-apt-and-apt-get/)


## sed 命令

[Linux Sed命令详解](https://qianngchn.github.io/wiki/4.html)

s：替换字符串，通常s命令的用法是这样的：1,$s/Regexp/Replacement/Flags，分隔字符/可以用其他任意单字符代替，用Replacement替换掉匹配字符串

多由管道符`|`与tee一起使用

`tee命令用于读取标准输入的数据，并将其内容输出成文件`

