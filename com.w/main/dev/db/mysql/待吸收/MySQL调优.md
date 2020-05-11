# 调优

## ip地址

```sql
 select INET_ATON('192.168.1.124');
```

## 语句调整

```sql
set profiling = 1;
show profiles;
show profile;
show profile for query 4;

use performance_schema;
//监控较低等级的操作的性能情况
```

