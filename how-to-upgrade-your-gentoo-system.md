title: 如何正确的升级你的 Gentoo 系统
slug: how-to-upgrade-your-gentoo-system
date: 2015-05-01 00:15:57
tags: gentoo, linux

1. 同步 Portage 树至最新

       # emerge --sync        // 或者是 eix-sync, 如果你安装了的话

2. 升级整个系统

       # emerge -avuDN --with-bdeps=y @world       // 也就是 emerge --ask --verbose --update --deep --newuse --with-bdeps=y @world

3. 更新配置文件

   升级后, 新版本的包可能带有新的配置文件, 但是如果你之前修改过配置文件, 新安装的配置文件不会覆盖你之前的, 而是生成一个临时的以 _cfgXXXX 结尾的, 你可以手动合并它们:

       # etc-update       // 也可以用  dispatch-conf

4. 修复静态库

       # lafilefixer --justfixit | grep -v skipping

5. 清理没用的包

       # emerge -av --depclean

6. 修复 (重新构建) 那些依赖旧库的包

   升级系统后, 一些 so 库可能会改变名字 (子版本号), 有些包可能会依赖这些旧的 so 库, 而在前一步, 这些旧的 so 库十有八九已经被清理了, 所以这里需要重新构建一下那些 link 旧 so 库的包

       # emerge -pv @preserved-rebuild
       # revdep-rebuild

   第一句能够解决大部分的旧包的依赖, 但是第二个命令是一个好习惯. 参见: https://wiki.gentoo.org/wiki/Preserve-libs

7. 清理下载的旧的源码包

       # eclean -d distfiles

## 参考

https://forums.gentoo.org/viewtopic-t-807345.html
http://forums.gentoo.org/viewtopic-t-837192-start-0-postdays-0-postorder-asc-highlight-.html?sid=71d14ef7badfa8ed3ed1295b14aae618
