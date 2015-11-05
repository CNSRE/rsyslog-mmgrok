# rsyslog-mmgrok

Rsyslog 可以通过 rainerscript 和 mmnoramlize 模块对日志数据做一定的切割解析。但是 mmnormalize 的 rulebase 表达式和传统的正则表达式差异很大，功能有局限，而且可读性不高，大多数人不愿意使用。

Logstash 自带的 Grok 正则，在标准正则表达式基础上，添加了预定义表达式的功能，随着 ELK Stack 的流行 Grok 接受程度也在逐渐提高。logstash-patterns-core 附带了数百种预定义表达式，很大程度上帮助了对正则表达式不算精通的运维人员。

rsyslog-mmgrok 插件，用来在 rsyslog 中，使用 grok 正则处理数据。

## 编译

0. 安装 libgrok 库：
```
# MacOS 上可以直接安装
port install grok
# CentOS 上没有现成的，需要自己编译
git clone git@github.com:jordansissel/grok.git
rpm -bb grok.spec.template
# 使用 yum 而不是 rpm，因为 grok 有 libevent, pcre, tokyocabinet 依赖
yum install grok*.rpm
```
1. 下载 rsyslog 源码包：
```
git clone https://github.com/rsyslog/rsyslog.git
```
2. 复制本仓库源码文件到 rsyslog 源码目录内：
```
cp -r src/contrib/mmgrok ../rsyslog/contrib/
cp src/configure.ac ../rsyslog/
cp src/Makefile.am ../rsyslog/
```
3. 编译 rsyslog：
```
yum install -y libjson-c-devel glib-devel
export PKG_CONFIG_PATH=/lib64/pkgconfig/
autoconf
./configure  --enable-mmgrok
make
make install
```

## 配置示例

```
module(load="mmgrok")
action(type="mmgrok" patterndir="path/to/yourpatternsDir" match="%{WORD:test}" soure="msg" target="!msg")
template(name="tmlp" type="string" string="%$!msg!test%\n")
action(type="omfile"  file="path/to/file" template="tmlp")
```

## 参数说明
* patterndir: grok模板路径(默认/usr/share/grok/patterns/base）
* match：grok使用的匹配模板，在/usr/share/grok/patterns/base文件下可以查看预定义的模板
* source: 所要解析的源消息
* target : grok解析后的消息为json tree,target参数指定了json tree的root
