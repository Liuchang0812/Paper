一个准则就是：对任意PG的写操作在被该PG的 acting set 的所有成员持久化之前都不会向客户端回应成功。这表示：如果我们可以和每个acting set的一个成员交流，就将会有从上一次peering成功后的每个操作的纪录。这也表明：对于当前的primay来说是可以构建传播一个新的权威的历史纪录。

同时也要感谢在peering过程中 OSD map 角色（所有书籍的OSDs以及他们的状态，以及一些关于 PG 的信息）：

  当 OSD up或者down（或者被添加、移除）时，可能会影响一些PG的active sets。
  在 primay 成功的完成 peering 过程前，OSD map必须要表现出 OSD 是活着的，and well as of the first epoch in the current interval.
  修改只能在成功的完成peering后执行。


因此，一个新的primary可以使用最新的OSD map来生成一个 past intervals 的集合来决定哪一个 OSD 必须被consulted(商议)在我们可以成功的peer。一个OSD发现一个

对当前PG primary来说一个高层次的过程如下：

#. 取得一个最近的 OSDmap 来确认所有感兴趣的 acting sets的成员，确认我们还是primary
#. 生成一个从 last epoch 开始的 past interval 列表。成员的peering需要我们可以从每个past interval的acting set中联系至少一个OSD。
#. 询问list中每个结点的PGinfo（包括最近的PG写入，last epoch）。如果我们看到一个 last epoch 我们自己的新，我们可以减少更老的past interval，降低我们需要联系的OSD个数。
#. 如果有结点的PGlog中存在我没有的操作，指示他们给我发送差的日志，这样primary的PG log是最新的。
#. 对于当前 acting set的每个成员：
    #. 要求它复制从last epoch开始的所有PG log，这样我可以验证他们同意我的. 如果集群在一个操作被acting set的所有成员持久化之前故障，同时紧接着的peering没有记得那个操作，一个记得那个操作的结点重新加入，它的日志就会纪录一个不同的（divergent)的历史（相对于故障之后peering构建的authoritative history）。因为 divergent 事件没有被其它日志纪录，他们之前也没被确认给客户端，所以丢弃他们没有坏处（这样所有的OSD同意 the authoritative history)。但是，我们将不得不让存储的divergent日志的OSD删除被影响的对象（现在是标记为可疑的）。
    #. 问他们的missing set.在我们可以接收写操作之前必须被完全复制。

#. 在这个时间点，primary的PG log包含了 authoritative history，OSD 现在有足够的信息让acting set的其它OSD更新到最新的数据。
#. 如果在当前OSD map中的 primary 的 up_thru 小于 current interval 的第一个epoch，发送一个请求到 monitor 来更新它。
#. 对当前 acting set 的每个成员：
    #. 给他们发送日志更新来让他们的PG log与我的 authoritative history相同
    #. 等待他们已经持久化PG log的确认信息
#. 在这个时间点，所有acting set的OSD对所有的元数据达成一致，在之后任意的peering都会返回相同的关于更新的内容。




