一个准则就是：对任意PG的写操作在被该PG的 acting set 的所有成员持久化之前都不会向客户端回应成功。这表示：如果我们可以和每个acting set的一个成员交流，就将会有从上一次peering成功后的每个操作的纪录。这也表明：对于当前的primay来说是可以构建传播一个新的权威的历史纪录。

同时也要感谢在peering过程中 OSD map 角色（所有书籍的OSDs以及他们的状态，以及一些关于 PG 的信息）：

  当 OSD up或者down（或者被添加、移除）时，可能会影响一些PG的active sets。
  在 primay 成功的完成 peering 过程前，OSD map必须要表现出 OSD 是活着的，and well as of the first epoch in the current interval.
  修改只能在成功的完成peering后执行。


因此，一个新的primary可以使用最新的OSD map来生成一个 past intervals 的集合来决定哪一个 OSD 必须被consulted(商议)在我们可以成功的peer。一个OSD发现一个
