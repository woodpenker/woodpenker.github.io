---
title: XPS13-intel-CPU超频导致发热严重
tags: [fedora,linux,折腾]
categories: 折腾笔记
---

问题
===
前段时间入手了一台xps13,安装了fedora28后,经常在启动和关机时出项cpu过热的报警信息:
<!--more-->
```
[26670.911219] CPU1: Core temperature above threshold, cpu clock throttled (total events = 1)
[26670.911221] CPU0: Core temperature above threshold, cpu clock throttled (total events = 1)
[26670.911222] CPU2: Package temperature above threshold, cpu clock throttled (total events = 1)
[26670.911223] CPU3: Package temperature above threshold, cpu clock throttled (total events = 1)
[26670.911224] CPU0: Package temperature above threshold, cpu clock throttled (total events = 1)
[26670.911232] CPU1: Package temperature above threshold, cpu clock throttled (total events = 1)
```

主要是由于主板打开了超频选项,内核也支持超频,使得cpu由1.2GHZ超频到4GHZ以上,明显感觉主机发热严重.使用`s-tui`记录CPU温度都达到100度.真的是*烤鸡*了...

解决办法
===
查阅相关资料显示,intel的cpu可以通过修改`/sys/devices/system/cpu/intel_pstate`下的设置修改cpu超频.

```bash
╰─$ ls /sys/devices/system/cpu/intel_pstate 
max_perf_pct  min_perf_pct  no_turbo  num_pstates  status  turbo_pct
```

可以通过修改`no_turbo`为`1`来关闭超频.

```bash
echo 1 |sudo tee /sys/devices/system/cpu/intel_pstate/no_turbo
```

cpu温度明显的降到了40度左右,发热现象消失了.
开了n个虚拟机和chrome视频站点以及vscode,检测cpu的使用率并未负载严重,说明这样已经基本满足日常使用需求了,起码可以省电,减少发热.

这样做在重启后会失效,除非自己修改内核配置再编译内核.关闭内核超频开关,或者开机F12进入主板设置,把turbo关闭.
但是这样做不够灵活,万一我突然有大量的计算要跑呢?

工具和方法有很多:

可以参考这里的回答:[stop cpu from overheating](https://askubuntu.com/questions/391474/stop-cpu-from-overheating/875872#875872)

- TLP工具是个选择,不仅能够减少发热,还能极端的降低电池消耗,延长电池使用时间.

- 简单点的即使关闭`turbo`,可以通过systemd自动加载服务的方式来保证开机自动关闭参考[
	Manage Intel Turbo Boost with systemd](https://blog.christophersmart.com/2017/02/08/manage-intel-turbo-boost-with-systemd/):
	新建一个文件`/usr/lib/systemd/system/disable-turbo-boost.service` 打开并编辑该文件写入:

```
[Unit]
Description=Disable Turbo Boost on Intel CPU

[Service]
ExecStart=/bin/sh -c "/usr/bin/echo 1 > \
/sys/devices/system/cpu/intel_pstate/no_turbo"
ExecStop=/bin/sh -c "/usr/bin/echo 0 > \
/sys/devices/system/cpu/intel_pstate/no_turbo"
RemainAfterExit=yes
User=root

[Install]
WantedBy=sysinit.target
```

使用`sudo systemctl daemon-reload`加载服务配置
使用:`sudo systemctl enable disable-turbo-boost.service`来开启该服务,
使用`sudo systemctl start disable-turbo-boost.service`来启动该服务.
这样就可以开机自动启动关闭turbo的功能.如果像临时关闭turbo,那么就使用`sudo systemctl stop disable-turbo-boost.service`
	
- 可是我不想每次需要关闭的时候都去执行systemd命令,我想在gnome dash中执行程序,那么在gnome下我也可以自己新建一个app:	`~/.local/share/applications/open-turbo.desktop`
编辑其内容为:

```
[Desktop Entry]
Version=1.0
Type=Application
Name=openturbo
Comment=open cpu turbo boost
Exec=gksudo -k -u root /bin/sh -c "/usr/bin/echo 0 > /sys/devices/system/cpu/intel_pstate/no_turbo"
Terminal=false
Categories=Utilities;
```
	
- 如果你习惯使用gnome的插件,希望在桌面就可以控制cpu频率上限和超频选项,那么可以使用[extension](https://extensions.gnome.org/extension/945/cpu-power-manager/),其代码地址[github](https://github.com/martin31821/cpupower)

有关于tubo加速的内核参数设置可以参考这里[服务器server的频率知识整理](http://lynnapan.github.io/2016/12/15/Turbo%E5%92%8CIntel_Pstate/):