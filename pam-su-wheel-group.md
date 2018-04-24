title: PAM, su 以及 wheel 用户组
slug: pam-su-wheel-group
date: 2015-05-02 11:27:31
tags: pam, su

在我的 gentoo 系统下, su 使用 pam 模组, 要求只有处于 wheel 用户组的普通用户才能够使用 su 切换到 root 用户的权限. 如果你查看 /etc/pam.d/su, 可以看到下面这一行:

     auth required pam_wheel.so use_uid

但是 GNU su 不支持 wheel 用户组, 也就是说, 如果你使用的是 GNU su, 那么当你 (普通用户) 执行 GNU su 来切换到 root 权限时, GNU su 不会检查你是否是 wheel 用户组的一员, 只要你给出了 root 用户的密码就能够切换成功.

这是怎么实现的呢, 可以看出, 上面那一行 pam 规则中, flag 值用的是 required, required 这个值在这里表示不管 pam_wheel.so 模块的检查结果是成功还是失败, 后续的检查都会继续运行下去, 那么 GNU su 只要忽略 pam_wheel.so 的检查结果就可以了. 而其他的 su 实现, 则不会忽略 pam_wheel.so 的检查结果, 当 pam_wheel.so 结果为失败的时候, su 就会阻止用户切换权限.

至于为什么 GNU su 要不支持 wheel 用户组. Rchard Stallman (向大神致敬!) 给出了解释:

> (This section is by Richard Stallman.)
> Sometimes a few of the users try to hold total power over all the rest. For example, in 1984, a few users at the MIT AI lab decided to seize power by changing the operator password on the Twenex system and keeping it secret from everyone else. (I was able to thwart this coup and give power back to the users by patching the kernel, but I wouldn't know how to do that in Unix.)
> However, occasionally the rulers do tell someone. Under the usual su mechanism, once someone learns the root password who sympathizes with the ordinary users, he or she can tell the rest. The “wheel group” feature would make this impossible, and thus cement the power of the rulers.
> I'm on the side of the masses, not that of the rulers. If you are used to supporting the bosses and sysadmins in whatever they do, you might find this idea strange at first.

总的来说, 就是说支持 wheel 用户组, 会对普通用户更加不利, 而对掌管着 root 用户的系统管理员更加有利. Richard Stallman 把系统管理员比作 boss, 老板. 他反问用户, 你希望你的老板独自拥有最大的权利吗?

## 参考

1. http://linux.vbird.org/linux_basic/0410accountmanager.php#pam_setting
2. http://www.gnu.org/software/coreutils/manual/html_node/su-invocation.html#index-fascism-2365
3. http://forums.gentoo.org/viewtopic-t-23378.html
4. https://wiki.gentoo.org/wiki/FAQ#Why_can.27t_a_user_su_to_root.3F
