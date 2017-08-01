ceph-mgr 的实现解析
####################


注：代码基于 master@5d71220838679dadc61393184c7746b6ace501f8

ceph-mgr 的高可用模式
====================

ceph-mgr 使用 standby 的模式，也就是系统可以同时启动多个ceph-mgr实例，其中只有一个ceph-mgr实例是active的状态，并对外进行服务。在发生故障的时候，ceph-mgr 自动选举其他的 ceph-mgr 实例来作为 active，保证系统的可用性。

关于 standby 模式的逻辑实现在 src/mgr/MgrStandby.{h,cc} 文件中。主要是通过 ceph-mon 提供的 paxos 服务来存储 mgr map，由 mgr map 来决定哪个实例是 active，哪些是 standby。



ceph-mgr 的集群状态维护
=====================

src/mgr/ClusterState.{h,cc} 定义了一个类 ClusterState，来维护整个集群的状态。包括：osdmap(好像是通过 objecter* 来获取osdmap)，monmap(通过MonClient*来获取)，fsmap,mgrmap,pgmap, existing_pools。

#. MMgrDigest 来得到 health_json, mon_status_json
#. MPGStats 来得到

ceph-mgr 的启动与运行流程
=======================

ceph-mgr 二进制文件由 ceph-mgr.cc 的 main 函数提供入口点，然后调用 MgrStandby::init() 和 MgrStandby::main 函数，将控制权交由 MgrStandby 类。

MgrStandby 对于接收到的消息：

#. MMgrMap 消息，就判断 wether it need to be updated its state. It will be respawned if there are different pymodules or it's not active.
#. 透传给 Mgr 类来处理

ceph-mgr 的 respawn
=====================

当 pymodules 发生变化时， ceph-mgr 就要 respawn 了，或者之前是 active ，变更为 standby 状态后，也要 respawn。
