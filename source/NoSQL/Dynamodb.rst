DYNAMODB
##############


INTERFACE
----------

1. why does Dynamodb provide Put/Get only? why not Scan?

我们知道，Cassandra 是 dynamodb 的开源实现，Cassandra 实现了 order by 操作，为什么 dynamodb 只提供了 Put 和 Get 接口，不提供 Scan 接口呢？详见论文的 4.1 章:

.. code:: c

  Dynamo stores objects associated with a key through a simple
  interface; it exposes two operations: get() and put(). The get(key)
  operation locates the object replicas associated with the key in the
  storage system and returns a single object or a list of objects with
  conflicting versions along with a context. The put(key, context,
  object) operation determines where the replicas of the object
  should be placed based on the associated key, and writes the
  replicas to disk. The context encodes system metadata about the
  object that is opaque to the caller and includes information such as
  the version of the object. The context information is stored along
  with the object so that the system can verify the validity of the
  context object supplied in the put request.


因为 dynamodb 是基于一致性哈希来做副本机制的，对于一个key，要先用hash取得一个token，才可以确定要写到哪个node上。因为dynamodb并不是全局有序的NoSQL，而是每个token是有序的。因此，如果要实现Scan接口，要不就是要扫描整个存储空间，要么就要限定到一个token级别，代价特别大。

Cassandra 对用户暴露了两个概念：

#. partition key: 用来定位到 vnode
#. order key: 用来排序

2. Cassandra 的 Put 接口要传入一个 context 参数，这个参数的目的是什么，如何没有v-clock时怎么办？


Failure Tolerance
-------------------

1. 如果一台机器故障了，要如何恢复，大概要恢复多久？



vnode 的设计可以不要吗？
-----------------------------

使用 v-node 的设计有如下好处：

	1. 如果一个结点变的不可用（因为故障或者路由变更），这个结点的负载最终会分布到其它所有结点。
	2. 当有新结点加入时，它的负载与其它可用结点的负载相当。
	3. 允许一个集群中异构机型



相邻的几个v-node全落在一个物理结点上怎么办？
---------------------------------------------

论文中是这么说的 ::

          To address this, the
          preference list for a key is constructed by skipping positions in the
          ring to ensure that the list contains only distinct physical nodes.

通过忽略同一个物理结点上的v-node的方法，那在该物理结点故障后，v-node被打散到多个机器后，对于请求如何路由到正确的v-node？

I don't have any ideas about this problem



写冲突的问题，如果冲突，如何解决呢？
------------------------------------
dynamodb 使用 vector clock来做这件事，一个 vector clock是 list<pair<node, counter>>。dynamodb 中的每个 object 的每个 version 都有一个 vector clock标识。我们可以根据 vector clock 来判断两个对象是否冲突。判断方法如下 ::

     If the counters on the first object’s clock are less-than-or-equal to all of the nodes
     in the second clock, then the first is an ancestor of the second and
     can be forgotten.

