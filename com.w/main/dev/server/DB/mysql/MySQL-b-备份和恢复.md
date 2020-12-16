# 恢复

## 方式

### 备份方法划分

Hot Backup

数据库运行中直接备份

Cold Backup

数据库停止状态下的备份

Warm Backup

运行状态下备份，但对当前的数据库有影响（例如：使用全局锁保证数据一致性）

### 备份后文件划分

逻辑备份

- dump
- select * into outfile ‘d:\a.txt’ from [table_a]

裸文件备份

- ibbackup
- xtrabackup

### 备份内容划分

完全备份

增量备份

官方未提供增量备份

xtrabackup：原理记录每页最后LSN，大于之前全备时的LSN，则备份，否则不必备份

日志备份

## 获取一致性备份

mysqldump --single-transaction获取一致性备份

## 冷备

备份frm文件，共享表空间文件，独立表空间文件（*.idb），重做日志文件

建议：定期备份my.cnf，便于恢复

## 恢复

方式1：mysql -u xxx -p < data.txt

方式2：load data infile 'd:/a.txt'

方式3：mysqlimport [options] db_name txt1...

实际上是2中的命令接口，支持多张表的导入

## 二进制日志备份恢复

配置中启用
