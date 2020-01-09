开机自动挂载硬盘
####################################

手动处理 mount 不是很人性化，我们不能每次进入 Linux 系统就挂载一次多硬盘（那简直是浪费生命）！可以设置开机自动挂载，这样就方便多了。

自动挂载的配置文件：``/etc/fstab`` 及 ``/etc/mtab``

在开始修改配置文件前，先来说一说系统挂载的一些限制：

* 根目录 ``/`` 是必须挂载的﹐而且一定要先于其它 mount point 被挂载进来。
* 其它 mount point 必须是已建立的目录，可任意指定，但要遵守系统目录架构原则 (FHS)
* 所有 mount point 在同一时间之内﹐只能挂载一次。
* 所有 partition 在同一时间之内﹐只能挂载一次。
* 如果要卸载﹐您必须先将工作目录移到 mount point（及其子目录）之外。


文件格式
************************************

先让我们看一个 ``/etc/fstab`` 的例子！

.. highlight:: none

::

    [root@study ~]# cat /etc/fstab
    # Device                              Mount point  filesystem parameters    dump fsck
    /dev/mapper/centos-root                   /       xfs     defaults            0 0
    UUID=94ac5f77-cb8a-495e-a65b-2ef7442b837c /boot   xfs     defaults            0 0
    /dev/mapper/centos-home                   /home   xfs     defaults            0 0
    /dev/mapper/centos-swap                   swap    swap    defaults            0 0


其实 ``/etc/fstab`` (filesystem table) 就是将我们利用 mount 指令进行挂载时， 将所有的选项与参数预先写入到这个档案中。除此之外， ``/etc/fstab`` 还加入了 dump 这个备份用指令的支持！ 与开机时是否进行文件系统检验 fsck 等指令有关。 这个档案的内容共有六个字段， 各个字段的总结数据与详细数据如下：

::

    [硬件名/UUID等]  [挂载点]  [文件系统]  [文件系统参数]  [dump]  [fsck]


第一栏：磁盘装置文件名
====================================

这个字段可以填写的数据主要有三个：

* 文件系统或磁盘的装置文件名，如 ``/dev/vda2`` 等
* 文件系统的 UUID 名称，如 UUID=xxx
* 文件系统的 LABEL 名称，例如 LABEL=xxx

因为每个文件系统都可以有上面三个项目，所以你喜欢哪个项目就填哪个项目！无所谓的！

::

    [root@study ~]# blkid  # 列出系统中所有已挂载文件的类型和 UUID

    [root@study ~]# blkid -s UUID /dev/sda5  # 查看指定分区


第二栏：挂载点
====================================

挂载文件的目录（挂载好硬盘后可以通过挂载点访问硬盘的内容）！


第三栏：磁盘分区的文件系统
====================================

在手动挂载时可以让系统自动测试挂载，但在这个档案当中我们必须要手动写入文件系统才行！包括 ``xfs, ext4, vfat, reiserfs, nfs`` 等等。


第四栏：文件系统参数
====================================

文件系统参数就是写入在这个字段，这里用表格的方式汇整如下：

================    ======================
 参数                 内容意义
================    ======================
async/sync
异步/同步             设定磁盘是否以异步方式运作！预设为 async(效能较佳)
auto/noauto
自动/非自动           当下达 mount -a 时，此文件系统是否会被主动测试挂载。预设为 auto。
rw/ro
读写/只读             让该分区以可读写或者是只读的型态挂载上来，如果设定为只读。则不论对分区是否设定 w 权限，都无法写入！
exec/noexec
可执行/不可执行        限制在此文件系统内是否可以进行『执行』的操作。默认不设置此参数。举例来说，如果你将 noexec 设定在 /var ，当某些软件将一些执行文件放置于 /var 下时，那就会产生很大的问题喔！ 因此，建议这个 noexec 最多仅设定于你自定义或分享的一般数据目录。
user/nouser
允许/不允许挂载        是否允许普通用户使用 mount 指令来挂载。一般应该设定为 nouser ！
suid/nosuid
具有/不具有 suid       该文件系统是否允许 SUID 的存在。如果不是执行文件放置目录，也可以设定为 nosuid 来取消这个功能！
defaults              同时具有 rw, suid, dev, exec, auto, nouser, async 等参数。 基本上，预设情况使用 defaults 设定即可！
================    ======================


第五栏：能否被 dump 备份指令作用
====================================

dump 是一个用来做为备份的指令，不过现在有太多的备份方案了，所以可以不用理会这个项目，直接输入 0 ！


第六栏：是否以 fsck 检验扇区
====================================

早期开机的流程中，会检验本机的文件系统，看看文件系统是否完整 (clean)。 不过这个方式主要是透过 fsck 去实现，现在用的 xfs 文件系统不适用此方法（xfs 会自动进行检验，不需要额外进行这个动作）所以直接输入 0 就好。

小技巧
*****************************

``/etc/fstab`` 是开机时的配置文件，不过，实际 filesystem 的挂载是记录到 ``/etc/mtab`` 与 ``/proc/mounts`` 这两个档案当中的。每次我们在编辑 filesystem 的挂载时，会自动更新这两个档案喔！但是，万一你在 ``/etc/fstab`` 输入的数据错误，导致无法开机，而进入单人维护模式当中，那时候的 ``/`` 可是 read only 的状态，当然你就无法修改 ``/etc/fstab`` ，也无法更新 ``/etc/mtab`` 啰～那怎么办？没关系，可以利用底下这一招：

::

    [root@study ~]# mount -n -o remount,rw /