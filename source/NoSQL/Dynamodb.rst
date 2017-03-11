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
