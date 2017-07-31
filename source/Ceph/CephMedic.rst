Ceph集群诊断工具
##############

项目地址：https://github.com/ceph/ceph-medic

ceph-medic 是一个用来诊断Ceph集群是否有一些未知的问题的小工具，可以用来帮助我们及早的发现问题。目前，项目处于初始阶段，主体的框架实现完毕，但是检查项还较少。


如何安装
--------

考虑到我们主要的环境为无外网环境，因为需要使用离线安装，提前下载好 ceph-medic 及其依赖的包，然后在服务器进行安装。

下载所有的依赖包：

.. code::

git clone https://github.com/ceph/ceph-medic.git
cd ceph-medic
python setup.py sdist
mkdir deps
pip install -d deps execnet
pip install -d deps tambo
pip install -d deps remoto
cp disk/*.tar.gz deps
ls deps
tar cvzf ceph-medic-offline.tar.gz deps


现在 ceph-medic-offline.tar.gz 包就包含了所有安装所依赖的python包，我们可以把该包上传到服务器上，进行安装。


如何配置
--------

我们首先，需要在当前目录下新建一个 hosts 文件，告诉 ceph-medic 每个服务器的角色，该配置文件的格式如下：

.. code::

[mons]
node1
node2
node3

[osds]
node4
node5
node6



node[1-6] 为机器的 hostname。

如何使用
--------

在配置完成后，我们可以运行如下命令来执行对ceph集群的巡检。

.. code::

ceph-medic check



ceph-medic 会打印一些信息，告诉您检查的结果。如下：

.. code::

=======================  Starting remote check session  ========================
Version: 1.0.0    Cluster Name: "ceph"
Total hosts: [39]
OSDs:   36    MONs:    3     Clients:    0
MDSs:    0    RGWs:    0     MGRs:       0

================================================================================

------------ osds ------------
 node11
 node10
   .
   .
   .
 node7
 node6

------------ mons ------------
 node39
 node38
 node40

201 passed, on 39 hosts



代码实现
---------

ceph-medic 的代码实现比较简单，runner.py 负责加载测试，测试按照模块划分为 common/client/mon/osd/mgr/mds 等，在 checks 目录下面包括对应模块的测试文件。

如果需要添加新的检查项，可以直接在 checks/{module}.py 中添加测试函数。

