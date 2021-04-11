# 大表删除

## 策略

1. 批量删除，***limit\***

2. 增加*key_buffer_size*，是global变量，详情参见Mysql官方文档： https://dev.mysql.com/doc/refman/5.7/en/server-configuration.html

3. **DELETE QUICK + OPTIMIZE TABLE**， MyISAM Tables

    *Why:* MyISAM删除的数据维护在一个链表中，这些空间和行的位置接下来会被Insert的数据复用。 直接的delete后，mysql会合并索引块，涉及大量内存的拷贝移动；而OPTIMIZE TABLE直接重建索引，即直接把数据块情况，再重新搞一份（联想JVM垃圾回收算法）。

4. **表分区，直接删除过期日期所在的分区（最终方案—秒杀）**， 分区方式RANGE、KEY、LIST、HASH

如果删除的数据超过表数据的百分之50，建议拷贝所需数据到临时表，然后删除原表，再重命名临时表为原表