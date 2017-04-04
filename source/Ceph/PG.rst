Some notes about PG/PGLog
######

BASIC TYPES
----
OLH
    代表 Object Logical Head

eversion_t
    compound rados version type,有三个成员变量，分别为
 
#. version_t version;
#. epoch_t epoch;
#. __u32 __pad;

pg_log_entry_t
    代表 pg log 中的一个单独的条目或者事件，事件类型有如下：

.. code::
  
    enum {                                                                                             
    MODIFY = 1,   // some unspecified modification (but not *all* modifications)                         
    CLONE = 2,    // cloned object from head                                                                                                                             
    DELETE = 3,   // deleted object                                                                
    BACKLOG = 4,  // event invented by generate_backlog [deprecated]                                                                                   
    LOST_REVERT = 5, // lost new version, revert to an older version.                                                                                              
    LOST_DELETE = 6, // lost new version, revert to no object (deleted).                                                                                       
    LOST_MARK = 7,   // lost new version, now EIO                                                                                                     
    PROMOTE = 8,     // promoted object from another tier                                                                                                                     
    CLEAN = 9,       // mark an object clean                                                                                                                                          
    ERROR = 10,      // write that returned an error                                                                                                                     
  };     
  
  
pg_log_t
    最近PG修改的增量日志，最近修改的恢复队列。真实的日志数据存储在mempool::osd::list<pg_log_entry_t> log中，维护了head/tail/can_rollback_to/rollback_info_trimmed_to等指针信息。

pg_missing_item
    summary of missing objects。只包含两个成员 eversion_t need, have; 在 Ceph 的实现中，是对每一个 hobject_t 维护一个 pg_missing_item，来判断该对象是否missing。
    
pg_info_t
    summarry of a pg
    
pg_interval_t  
    information about a past interval. 这个类的作用是什么？
    
last_commit/last_update
    Ceph 多副本之间保证一致性的一种方法，详细介绍可以参考 http://www.infoq.com/cn/articles/consistency-storage-problem-of-ceph-based-on-pglog

object_copy_cursor_t
    一个用来表示数据复制进度的类，成员有 data_offset, omap_offset （？为什么没有 attr_offset)
    
