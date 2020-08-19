# 开发Http

## 开发Http

## 配置、error日志、请求上下文

## 访问第三方服务

## 开发简单Http过滤模块

## Nginx提供高级数据结构

ngx_queue_t双向链表

不负责内存分配，只负责节点管理

轻量级链表容器

ngx_array_t动态数组

ngx_list_t单向链表

负责内存分配

ngx_radix_tree_t基数树和ngx_rbtree_t红黑树

ngx_radix_tree_t

必须以整数作为关键字

插入删除无需旋转

ngx_rbtree_t

红黑树

### 为什么？



ngx_radix_tree_t基数

支持通配符的散列表