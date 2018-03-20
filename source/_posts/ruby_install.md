title: Ruby环境安装
date: 2017/8/13 20:46:25
---

# 一.准备

1. 良好的网络

# 二.安装(linux与mac环境)

## 步骤一：安装lvm

```
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB

curl -sSL https://get.rvm.io | bash -s stable
```

通过以上两个步骤，就可以安装好lvm
<!--more-->
## 步骤二：安装ruby

1. 查看可用ruby列表

   `rvm list know`

   ```
   # MRI Rubies
   [ruby-]1.8.6[-p420]
   [ruby-]1.8.7[-head] # security released on head
   [ruby-]1.9.1[-p431]
   [ruby-]1.9.2[-p330]
   [ruby-]1.9.3[-p551]
   [ruby-]2.0.0[-p648]
   [ruby-]2.1[.10]
   [ruby-]2.2[.7]
   [ruby-]2.3[.4]
   [ruby-]2.4[.1]
   ruby-head

   # for forks use: rvm install ruby-head-<name> --url https://github.com/github/ruby.git --branch 2.2

   # JRuby
   jruby-1.6[.8]
   jruby-1.7[.27]
   jruby[-9.1.13.0]
   jruby-head
   .....
   ```

2. 安装ruby

   可用列表中有ruby2.4,择我们可以直接安装

   ```Shell
   rvm install 2.4	
   ```

3. 异常处理

   1.  rvm dyld: lazy symbol binding failed: Symbol not found: _utimensat   Referenced from: /Users/geekyzk/.rvm/src/ruby-2.3.4/./miniruby   Expected in: /usr/lib/libSystem.B.dylib

   在mac下，安装如果出现这个问题，是需要安装xcode的命令行工具，解决方式如下

   在命令行中运行`xcode-select —install`，根据提示安装好xcode的命令行工具后。再次执行`rvm install version`

4. gem管理工具

   在ruby的新版本，默认已经安装有gem这个管理工具