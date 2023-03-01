步骤一：启用 ELRepo从 CentOS 8 开始，ELRepo 已经加入到官方软件仓库中，名称为 “elrepo-release”。AlmaLinux 和 Rocky Linux 同样适用。启用 ELRepo 只需要执行命令：
	* dnf -y install elrepo-release
	* #or
	* yum -y install elrepo-release
	* 

rpm 包在线安装方法：（两种方法任选一种即可）
	* # Import the public key:
	* rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
	* 
	* #To install ELRepo for RHEL-9:
	* yum install https://www.elrepo.org/elrepo-release-9.el9.elrepo.noarch.rpm
	* 
	* #To install ELRepo for RHEL-8:
	* yum install https://www.elrepo.org/elrepo-release-8.el8.elrepo.noarch.rpm
	* 
	* #To install ELRepo for RHEL-7, SL-7 or CentOS-7:
	* yum install https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm

步骤二：升级内核
	* # 列出可用的稳定版本内核相关包：
	* yum --disablerepo="*" --enablerepo="elrepo-kernel" list available
	* # 列出可用的所有版本内核相关包：
	* yum --disablerepo="*" --enablerepo="elrepo-kernel" list available --showduplicates

术语解释

	* kernel-mlkernel-ml 中的 ml 是英文 “mainline stable” 的缩写，elrepo-kernel 中列出来的是最新的稳定主线版本。
	* kernel-ltkernel-lt 中的 lt 是英文 “long term support” 的缩写，elrepo-kernel 中列出来的长期支持版本。


安装最新的主线稳定内核：
	* yum --enablerepo=elrepo-kernel install kernel-ml -y

步骤三：设置 GRUB 默认的内核版本为了让新安装的内核成为默认启动选项，你需要如下修改 GRUB 配置：打开并编辑 /etc/default/grub 并设置 GRUB_DEFAULT=0。意思是 GRUB 初始化页面的第一个内核将作为默认内核。
	* GRUB_TIMEOUT=5
	* GRUB_DEFAULT=0
	* GRUB_DISABLE_SUBMENU=true
	* GRUB_TERMINAL_OUTPUT="console"
	* GRUB_CMDLINE_LINUX="rd.lvm.lv=centos/root rd.lvm.lv=centos/swap crashkernel=auto rhgb quiet"
	* GRUB_DISABLE_RECOVERY="true"

接下来运行下面的命令来重新创建内核配置。
	* grub2-mkconfig -o /boot/grub2/grub.cfg

查看默认内核和当前是否一样。
	* grubby --default-kernel

接下来reboot重启服务器就完成了！ 步骤四：安装BBRLinux内核4.9版本以后自带BBR，之前我们已经更新了内核，所以，现在只需要开去BBR功能即可。
	* echo "net.ipv4.tcp_congestion_control= bbr" >> /etc/sysctl.conf
	* echo "net.core.default_qdisc = fq" >> /etc/sysctl.conf
	* sysctl -p
	* reboot

重启后执行检查命令，看是否有输出BBR。
	* sysctl net.ipv4.tcp_available_congestion_control
	* net.ipv4.tcp_available_congestion_control = reno cubic bbr

现在内核中已经BBR功能。下面检测是否正在运行。
	* lsmod | grep bbr
	* tcp_bbr 20480 8

看到 tcp_bbr 说明 BBR 已经成功在运行了。步骤五：删除旧内核（可选）
	* # 再查看系统已安装的内核，确认旧内核版本已经删除：
	* rpm -qa | grep kernel
	* # 删除旧内核
	* yum remove kernel-4.18.0 kernel-core-4.18.0 kernel-modules-4.18.0 kernel-devel-4.18.0 kernel-tools-4.18.0 kernel-tools-libs-4.18.0 kernel-headers-4.18.0
	* # 注意，会同时删除一些依赖包（如 gcc、systemtap 等），需要重新补安装回来。

 


文章作者：IDC评估网
文章名称：AlmaLinux8、CentOS7/8、Rocky Linux 8升级更新内核并开启bbr加速以及删除多余旧内核
文章链接：https://www.idceval.com/76.html
版权声明：文章版权归作者所有，未经允许请勿转载。
