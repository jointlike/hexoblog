
---
title: ceph 概念命令
date: 2018-05-24 14:50:25
updated: 2018-05-24 14:50:25
categories: 分布式存储系统
tags: ['分布式存储','ceph']
---

 结合网络、官网、手动查询等多方渠道，整理ceph维护管理常用命令，并且梳理常规命令在使用过程中的逻辑顺序。另外整理期间发现ceph 集群的命令体系有点乱，详细情况各自体验。



一：ceph集群启动、重启、停止

1：ceph 命令的选项如下：

选项	简写	描述

--verbose	-v	详细的日志。

--valgrind	N/A	（只适合开发者和质检人员）用 Valgrind 调试。

--allhosts	-a	在 ceph.conf 里配置的所有主机上执行，否 则它只在本机执行。

--restart	N/A	核心转储后自动重启。

--norestart	N/A	核心转储后不自动重启。

--conf	-c	使用另外一个配置文件。



Ceph 子命令包括：

命令	描述

start	启动守护进程。

stop	停止守护进程。

forcestop	暴力停止守护进程，等价于 kill -9

killall	杀死某一类守护进程。

cleanlogs	清理掉日志目录。

cleanalllogs	清理掉日志目录内的所有文件。



2：启动所有守护进程

	要启动、关闭、重启 Ceph 集群，执行 ceph 时加上 相关命令，语法如下：

/etc/init.d/ceph [options] [start|restart|stop] [daemonType|daemonID]

下面是个典型启动实例：

sudo /etc/init.d/ceph -a start

加 -a （即在所有节点上执行）执行完成后Ceph本节点所有进程启动。



把 CEPH 当服务运行，按此语法：

service ceph [options] [start|restart] [daemonType|daemonID] 

典型实例：  service ceph -a start



3:启动单一实例

	要启动、关闭、重启一类守护进程  本例以要启动本节点上某一类的所有 Ceph 守护进程，

/etc/init.d/ceph  [start|restart|stop] [daemonType|daemonID]

/etc/init.d/ceph start osd.0



 把ceph当做服务运行，启动一节点上某个 Ceph 守护进程，

按此语法： service ceph  start {daemon-type}.{instance} 

service ceph  start osd.0



二：集群维护常用命令概览

1：检查集群健康状况

	启动集群后、读写数据前，先检查下集群的健康状态。你可以用下面的命令检查：

ceph health   或者 ceph health detail （输出信息更详细）

	要观察集群内正发生的事件，打开一个新终端，然后输入：

ceph -w

输出信息里包含：

集群唯一标识符

集群健康状况

监视器图元版本和监视器法定人数状态

OSD 版本和 OSD 状态摘要

其内存储的数据和对象数量的粗略统计，以及数据总量等。



新版本新增选项如下：

  -s, --status          show cluster status

  -w, --watch           watch live cluster changes

  --watch-debug         watch debug events

  --watch-info          watch info events

  --watch-sec           watch security events

  --watch-warn          watch warn events

  --watch-error         watch error events

  --version, -v         display version

  --verbose             make verbose

  --concise             make less verbose

使用方法演示：

ceph -w --watch-info



2：检查集群的使用情况

	检查集群的数据用量及其在存储池内的分布情况，可以用 df 选项，它和 Linux 上的 df 相似。如下：

ceph df

输出的 GLOBAL 段展示了数据所占用集群存储空间的概要。

SIZE: 集群的总容量；

AVAIL: 集群的空闲空间总量；

RAW USED: 已用存储空间总量；

% RAW USED: 已用存储空间比率。用此值参照 full ratio 和 near full \ ratio 来确保不会用尽集群空间。

详情见存储容量。

输出的 POOLS 段展示了存储池列表及各存储池的大致使用率。没有副本、克隆品和快照占用情况。例如，如果你把 1MB 的数据存储为对象，理论使用率将是 1MB ，但考虑到副本数、克隆数、和快照数，实际使用率可能是 2MB 或更多。

NAME: 存储池名字；

ID: 存储池唯一标识符；

USED: 大概数据量，单位为 KB 、 MB 或 GB ；

%USED: 各存储池的大概使用率；

Objects: 各存储池内的大概对象数。



新版本新增ceph osd df  命令，可以详细列出集群每块磁盘的使用情况，包括大小、权重、使用多少空间、使用率等等



3：检查集群状态

	要检查集群的状态，执行下面的命令：:

ceph status



4：检查MONITOR状态

	查看监视器图，执行下面的命令：:

ceph mon stat

或者：

ceph mon dump

	要检查监视器的法定人数状态，执行下面的命令：

ceph quorum_status

5：检查 MDS 状态:

	元数据服务器为 Ceph 文件系统提供元数据服务，元数据服务器有两种状态： up | \ down 和 active | inactive ，执行下面的命令查看元数据服务器状态为 up 且 active ：

ceph mds stat

	要展示元数据集群的详细状态，执行下面的命令：

ceph mds dump



三：集群命令详解



######################mon 相关####################

1：查看mon的状态信息

[mon@ptmind~]# ceph mon stat



2：查看mon的选举状态

[mon@ptmind~]# ceph quorum_status



3：查看mon的映射信息

[mon@ptmind~]# ceph mon dump



4：删除一个mon节点

[mon@ptmind~]# ceph mon remove cs1



5：获得一个正在运行的mon map，并保存在1.txt文件中

[mon@ptmind~]# ceph mon getmap -o 1.txt



6：读取上面获得的map

[mon@ptmind~]# monmaptool --print 1.txt 



7：把上面的mon map注入新加入的节点

[mon@ptmind~]# ceph-mon -i nc3 --inject-monmap 1.txt



8：查看mon的amin socket

[mon@ptmind~]# ceph-conf --name mon.nc3 --show-config-value admin_socket





9：查看ceph mon log日志所在的目录

[mon@ptmind~]# ceph-conf --name mon.nc1 --show-config-value log_file    

/var/log/ceph/ceph-mon.nc1.log



10：查看一个集群ceph-mon.nc3参数的配置、输出信息特别详细，集群所有配置生效可以在此参数下确认

[mon@ptmind~]# ceph --admin-daemon /var/run/ceph/ceph-mon.nc3.asok config show | less



######################### msd 相关 ###################

1：查看msd状态

[mon@ptmind~]# ceph mds stat



2：删除一个mds节点

[mon@ptmind~]# ceph  mds rm 0 mds.nc1



3：设置mds状态为失败

[mon@ptmind~]# ceph mds rmfailed <int[0-]>   

 

4：新建pool 

[mon@ptmind~]# ceph mds add_data_pool <poolname>



5：关闭mds集群

[mon@ptmind~]# mds cluster_down



6：启动mds集群

[mon@ptmind~]# mds cluster_up

 

7：设置cephfs文件系统存储方式最大单个文件尺寸

[mon@ptmind~]# ceph mds set max_file_size 1024000000000



### 清除cephfs 文件系统步骤 ####



强制mds状态为featrue

[mon@ptmind~]# ceph mds fail 0



删除mds文件系统

[mon@ptmind~]# ceph fs rm leadorfs --yes-i-really-mean-it

#删除fs数据文件夹

[mon@ptmind~]# ceph osd pool delete cephfs_data cephfs_data --yes-i-really-really-mean-it 



#删除元数据文件夹

[mon@ptmind~]# ceph osd pool delete cephfs_metadata cephfs_metadata --yes-i-really-really-mean-it 



然后再删除 mds key  ，残留文件等



拆除文件系统前推荐先删除节点，待验证

[mon@ptmind~]# ceph  mds rm 0 mds.node242

 

################################ ceph auth 相关 ################################

1：查看ceph集群中的认证用户及相关的key

[mon@ptmind~]# ceph auth list



2：为ceph创建一个admin用户并为admin用户创建一个密钥，把密钥保存到/etc/ceph目录下：

[mon@ptmind~]# ceph auth get-or-create client.admin mds ‘allow‘ osd ‘allow *‘ mon ‘allow *‘ > /etc/ceph/ceph.client.admin.keyring

或

[mon@ptmind~]# ceph auth get-or-create client.admin mds ‘allow‘ osd ‘allow *‘ mon ‘allow *‘ -o /etc/ceph/ceph.client.admin.keyring



3：为osd.0创建一个用户并创建一个key

[mon@ptmind~]# ceph auth get-or-create osd.0 mon ‘allow rwx‘ osd ‘allow *‘ -o /var/lib/ceph/osd/ceph-0/keyring



4：为mds.nc3创建一个用户并创建一个key

[mon@ptmind~]# ceph auth get-or-create mds.nc3 mon ‘allow rwx‘ osd ‘allow *‘ mds ‘allow *‘ -o /var/lib/ceph/mds/ceph-cs1/keyring



5：导入key信息

[mon@ptmind~]# ceph auth import   /var/lib/ceph/mds/ceph-cs1/keyring



6：删除集群中的一个认证用户

[mon@ptmind~]# ceph auth del osd.0



################################ osd  相关 ################################

1：查看osd列表

[mon@ptmind~]# ceph osd tree



2：查看数据延迟

[mon@ptmind~]# ceph osd perf 

osd fs_commit_latency(ms) fs_apply_latency(ms) 

  0                     3                    4 

  1                   333                  871 

  2                    33                   49 

  3                     1                    2 

。。。。。。。。。。。。





3：详细列出集群每块磁盘的使用情况，包括大小、权重、使用多少空间、使用率等等

[mon@ptmind~]# ceph osd df 



4：down掉一个osd硬盘

[mon@ptmind~]# ceph osd down 0   #down掉osd.0节点

 

5：在集群中删除一个osd硬盘

[mon@ptmind~]# ceph osd rm 0



6：在集群中删除一个osd 硬盘 crush map

[mon@ptmind~]# ceph osd crush rm osd.0



7：在集群中删除一个osd的host节点

[mon@ptmind~]# ceph osd crush rm cs1



8：查看最大osd的个数 

 [mon@ptmind~]# ceph osd getmaxosd

max_osd = 90 in epoch 1202     #默认最大是90个osd节点



9：设置最大的osd的个数（当扩大osd节点的时候必须扩大这个值）

 [mon@ptmind~]# ceph osd setmaxosd 2048

 

10：设置osd crush的权重为1.0

ceph osd crush set {id} {weight} [{loc1} [{loc2} ...]]

例如：

[mon@ptmind~]# ceph osd crush set osd.1 0.5 host=node241



11：设置osd的权重

[mon@ptmind~]# ceph osd reweight 3 0.5

reweighted osd.3 to 0.5 (8327682)



或者用下面的方式

[mon@ptmind~]# ceph osd crush reweight osd.1 1.0





12：把一个osd节点逐出集群

[mon@ptmind~]# ceph osd out osd.3:



3       1    osd.3   up      0      # osd.3的reweight变为0了就不再分配数据，但是设备还是存活的



13：把逐出的osd加入集群

[mon@ptmind~]# ceph osd in osd.3

marked in osd.3. 



14：暂停osd （暂停后整个集群不再接收数据）

[mon@ptmind~]# ceph osd pause

      

15：再次开启osd （开启后再次接收数据） 

[mon@ptmind~]# ceph osd unpause



16：查看一个集群osd.0参数的配置、输出信息特别详细，集群所有配置生效可以在此参数下确认

[mon@ptmind~]# ceph --admin-daemon /var/run/ceph/ceph-osd.0.asok config show | less





17：设置标志 flags ，不允许关闭osd、解决网络不稳定，osd 状态不断切换的问题

[mon@ptmind~]# ceph osd set nodown



取消设置

[mon@ptmind~]# ceph osd unset nodown



################################pool  相关################################



1：查看ceph集群中的pool数量

[mon@ptmind~]# ceph osd lspools   或者 ceph osd pool ls



2：在ceph集群中创建一个pool

[mon@ptmind~]# ceph osd pool create rbdtest 100            #这里的100指的是PG组:



3：查看集群中所有pool的副本尺寸

[mon@ptmind~]# ceph osd dump | grep ‘replicated size‘



4：查看pool 最大副本数量

[mon@ptmind~]# ceph osd pool get rbdpool size

size: 3



5：查看pool 最小副本数量

[root@node241 ~]# ceph osd pool get rbdpool min_size

min_size: 2



6：设置一个pool的pg数量

[mon@ptmind~]# ceph osd pool set rbdtest pg_num 100



7：设置一个pool的pgp数量

[mon@ptmind~]# ceph osd pool set rbdtest pgp_num 100



8: 修改ceph，数据最小副本数、和副本数

ceph osd pool set $pool_name min_size 1

ceph osd pool set $pool_name size 2



示例：

[mon@ptmind~]# ceph osd pool set rbdpool min_size 1

[mon@ptmind~]# ceph osd pool set rbdpool size 2



验证：

[mon@ptmind~]# ceph osd dump

pool 3 ‘rbdpool‘ replicated size 2 min_size 1 





9:设置rbdtest池的最大存储空间为100T（默认是1T)

[mon@ptmind~]# ceph osd pool set rbdtest target_max_bytes 100000000000000





10: 为一个ceph pool配置配额、达到配额前集群会告警，达到上限后无法再写入数据

[mon@ptmind~]# ceph osd pool set-quota rbdtest max_objects 10000



11: 在集群中删除一个pool,注意删除poolpool 映射的image 会直接被删除，线上操作要谨慎。

[mon@ptmind~]# ceph osd pool delete rbdtest  rbdtest  --yes-i-really-really-mean-it  #集群名字需要重复两次



12: 给一个pool创建一个快照

[mon@ptmind~]# ceph osd pool mksnap rbdtest   rbdtest-snap20150924



13: 查看快照信息

[mon@ptmind~]# rados lssnap -p rbdtest

1       rbdtest-snap20150924    2015.09.24 19:58:55

2       rbdtest-snap2015092401  2015.09.24 20:31:21

2 snaps



14:删除pool的快照

[mon@ptmind~]# ceph osd pool rmsnap rbdtest  rbdtest-snap20150924



验证，剩余一个snap

[mon@ptmind~]# rados lssnap -p rbdtest

2       rbdtest-snap2015092401  2015.09.24 20:31:21

1 snaps





################################rados命令相关##################

rados 是和Ceph的对象存储集群（RADOS），Ceph的分布式文件系统的一部分进行交互是一种实用工具。



1：查看ceph集群中有多少个pool （只是查看pool)

[mon@ptmind~]# rados lspools    同  ceph osd pool ls 输出结果一致



2：显示整个系统和被池毁掉的使用率统计，包括磁盘使用（字节）和对象计数

[mon@ptmind~]# rados df 



3：创建一个pool

[mon@ptmind~]# rados mkpool test



4：创建一个对象object 

[mon@ptmind~]# rados create test-object -p test



5：查看对象文件

[mon@ptmind~]# rados -p test ls

test-object



6：删除一个对象

[mon@ptmind~]# rados rm test-object-1 -p test



7：删除foo池 (和它所有的数据)

[mon@ptmind~]# rados rmpool test test –yes-i-really-really-mean-it 

	

8：查看ceph pool中的ceph object （这里的object是以块形式存储的）

[mon@ptmind~]# rados ls -p test | more



9：为test pool创建快照

[mon@ptmind~]# rados -p test mksnap testsnap

created pool test snap testsnap



10：列出给定池的快照

[mon@ptmind~]# rados -p test lssnap         

1       testsnap        2015.09.24 21:14:34



11：删除快照

[mon@ptmind~]# rados -p test rmsnap testsnap

removed pool test snap testsnap



12：上传一个对象到test pool

[mon@ptmind~]# rados -p test put myobject blah.txt





################################ PG 相关 ################################

PG =“放置组”。当集群中的数据，对象映射到编程器，被映射到这些PGS的OSD。



1：查看pg组的映射信息

[mon@ptmind~]# ceph pg dump    或者 ceph pg ls



2：查看一个PG的map

[mon@ptmind~]# ceph pg map 0.3f

osdmap e88 pg 0.3f (0.3f) -> up [0,2] acting [0,2]   #其中的[0,2]代表存储在osd.0、osd.2节点，osd.0代表主副本的存储位置



3：查看PG状态

[mon@ptmind~]# ceph pg stat



4：查询一个pg的详细信息

[mon@ptmind~]# ceph pg  0.26 query



5：要洗刷一个pg组，执行命令：

[mon@ptmind~]# ceph pg scrub {pg-id}



6：查看pg中stuck的状态

要获取所有卡在某状态的归置组统计信息，执行命令：

ceph pg dump_stuck inactive|unclean|stale [--format <format>] [-t|--threshold <seconds>]



[mon@ptmind~]# ceph pg dump_stuck unclean

[mon@ptmind~]# ceph pg dump_stuck inactive

[mon@ptmind~]# ceph pg dump_stuck stale

Inactive （不活跃）归置组不能处理读写，因为它们在等待一个有最新数据的 OSD 复活且进入集群。

Unclean （不干净）归置组含有复制数未达到期望数量的对象，它们应该在恢复中。

Stale （不新鲜）归置组处于未知状态：存储它们的 OSD 有段时间没向监视器报告了（由 mon_osd_report_timeout 配置）。

可用格式有 plain （默认）和 json 。阀值定义的是，归置组被认为卡住前等待的最小时间（默认 300 秒）





7：显示一个集群中的所有的pg统计

[mon@ptmind~]# ceph pg dump --format plain



8：恢复一个丢失的pg

如果集群丢了一个或多个对象，而且必须放弃搜索这些数据，你就要把未找到的对象标记为丢失（ lost ）。

如果所有可能的位置都查询过了，而仍找不到这些对象，你也许得放弃它们了。这可能是罕见的失败组合导致的，集群在写入完成前，未能得知写入是否已执行。

当前只支持 revert 选项，它使得回滚到对象的前一个版本（如果它是新对象）或完全忽略它。要把 unfound 对象标记为 lost ，执行命令：

ceph pg {pg-id} mark_unfound_lost revert|delete





9：查看某个PG内分布的数据状态，具体状态可以使用选项过滤输出

ceph pg ls {<int>} {active|clean|down|replay|splitting|scrubbing|scrubq|degraded|inconsistent|peering|repair|recovering|backfill_wait|incomplete|stale|remapped|deep_scrub|backfill|

backfill_toofull|recovery_wait|undersized [active|clean|down|replay|splitting|scrubbing|scrubq|degraded|inconsistent|peering|repair|recovering|backfill_wait|incomplete|stale|remapped|

deep_scrub|backfill|backfill_toofull|recovery_wait|undersized...]} :  list pg with specific pool, osd, state

实例如下：

                    pg号   过滤输出的状态

[mon@ptmind~]# ceph   pg   ls       1     clean





10：查询osd 包含pg 的信息，过滤输出pg的状态信息

pg ls-by-osd <osdname (id|osd.id)>       list pg on osd [osd]

 {<int>} {active|clean|down|replay|splitting|scrubbing|scrubq|degraded|    

 inconsistent|peering|repair|recovering| backfill_wait|incomplete|stale|remapped|deep_scrub|backfill|backfill_toofull|recovery_wait|undersized[active|clean|down|replay|splitting|    

 scrubbing|scrubq|degraded|inconsistent| peering|repair|recovering|backfill_ wait|incomplete|stale|remapped|deep_scrub|backfill|backfill_toofull|recovery_wait|undersized...]} 

实例如下：

[mon@ptmind~]# ceph pg ls-by-osd osd.5 



11：查询pool包含pg 的信息，过滤输出pg的状态信息

ceph pg  ls-by-pool   poolname   选项

ceph pg ls-by-pool   <poolstr>  {active|clean| down|replay|splitting|scrubbing|scrubq|  degraded|inconsistent|peering|repair| recovering|backfill_wait|incomplete|  stale|remapped|deep_scrub|backfill|     

 backfill_toofull|recovery_wait| undersized [active|clean|down|replay|  splitting|scrubbing|scrubq|degraded| inconsistent|peering|repair|recovering| backfill_wait|incomplete|stale| remapped|deep_scrub|backfill|backfill_ 

实例如下：

[mon@ptmind~]# ceph pg ls-by-pool test  





12：查询某个osd状态为 primary pg ，可以根据需要过滤状态

pg ls-by-primary <osdname (id|osd.id)> {<int>} {active|clean|down|replay|splitting|scrubbing|scrubq|degraded|inconsistent|peering|repair|recovering|backfill_wait|incomplete|stale|remapped|deep_scrub|backfill|

backfill_toofull|recovery_wait|undersized [active|clean|down|replay|splitting|scrubbing|scrubq|degraded|inconsistent|peering|repair|recovering|backfill_wait|incomplete|stale|remapped|deep_scrub|backfill|

backfill_toofull|recovery_wait|undersized...]} :  list pg with primary = [osd]

 

 实例如下：

                         osd号   过滤输出的状态

[mon@ptmind~]# ceph pg ls-by-primary    osd.3    clean 

 

############################### rbd命令相关 #########################3 



1：在test池中创建一个命名为kjh的10000M的镜像

[mon@ptmind~]# rbd create -p test --size 10000 kjh



2：查看ceph中一个pool里的所有镜像

[mon@ptmind~]# rbd ls test

kjh



3：查看新建的镜像的信息

[mon@ptmind~]# rbd -p test info kjh    



4：查看ceph pool中一个镜像的信息

[mon@ptmind~]# rbd info -p test  --image kjh

rbd image ‘kjh‘:

        size 1000 MB in 250 objects

        order 22 (4096 kB objects)

        block_name_prefix: rb.0.92bd.74b0dc51

        format: 1

		

5：删除一个镜像

[mon@ptmind~]# rbd rm  -p test  kjh



6：调整一个镜像的尺寸

[mon@ptmind~]# rbd resize -p test --size 20000 kjh



[mon@ptmind~]# rbd -p test info kjh   #调整后的镜像大小

rbd image ‘kjh‘:

        size 2000 MB in 500 objects

        order 22 (4096 kB objects)

        block_name_prefix: rb.0.92c1.74b0dc51

        format: 1



#########rbd pool 快照功能测试###########



1：新建个pool叫’ptmindpool’同时在下面创建一个’kjhimage’



[mon@ptmind~]# ceph osd pool create ptmindpool 256 256

pool ‘ptmindpool‘ created





2:创建镜像

[mon@ptmind~]# rbd create kjhimage --size 1024 --pool ptmindpool



3:查看镜像

[mon@ptmind~]# rbd --pool ptmindpool ls

kjhimage



4:创建snap,快照名字叫’snapkjhimage’

[mon@ptmind~]# rbd snap create ptmindpool/kjhimage@snapkjhimage





5:查看kjhimage的snap

[mon@ptmind~]# rbd snap ls ptmindpool/kjhimage

SNAPID NAME         SIZE

     2 snapkjhimage 1024 MB

	 

6:回滚快照，

[mon@ptmind~]# rbd snap rollback ptmindpool/kjhimage@snapkjhimage





7:删除snap 删除snap报（rbd: snapshot ‘snapshot-xxxx‘ is protected from removal.）写保护 ，使用 rbd snap unprotect volumes/snapshot-xxx‘ 解锁，然后再删除

[mon@ptmind~]# rbd snap rm ptmindpool/kjhimage@snapkjhimage





8:删除kjhimage的全部snapshot



[mon@ptmind~]# rbd snap purge ptmindpool/kjhimage





9: 把ceph pool中的一个镜像导出

导出镜像

[mon@ptmind~]# rbd export -p ptmindpool --image kjhimage /tmp/kjhimage.img            

Exporting image: 100% complete...done.



验证查看导出文件

l /tmp/kjhimage.img        

-rw-r--r-- 1 root root 1073741824 Sep 24 23:15 /tmp/kjhimage.img





10:把一个镜像导入ceph中

[mon@ptmind~]# rbd import /tmp/kjhimage.img  -p ptmindpool --image importmyimage1

Importing image: 100% complete...done.



验证查看导入镜像文件

rbd -pptmindpool ls

importmyimage1







本文出自 “康建华” 博客，转载请与作者联系！
