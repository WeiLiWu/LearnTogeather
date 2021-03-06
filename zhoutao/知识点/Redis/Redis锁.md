#### 普通的Redis锁

set key value nx/xx ex/px  
1. nx：只有key不存在的时候才设置
2. xx：只有key存在的时候set
3. ex：秒
4. px：毫秒

**问题**：当客户端获取了锁，执行的时间比较长，长到锁已经消失了，业务逻辑还未执行，这个时候去释放锁，假如该锁已经被其他的客户端占有，直接释放，导致其他客户端代码还没有执行完毕就无锁了。  

**解决方案**：set的时候设置一个随机值，当锁删除的时候判断一下获取的随机值和设置的随机值是否一样，如果不一样就不释放锁
 
#### 分布式锁
##### RedisLock算法
***背景：redis cluster 有5个master实例 锁的过期时间为ttl，获取锁的时间为t***
1. 获取当前时间戳，单位毫秒
2. 客户端轮流尝试在每个redis节点上获取锁，key，value都相同，一般获取锁的时间t设置很短，如果失败，去下一个节点
3. 如果锁的节点数目大于n/2+1，并且当前时间-第一步获取的时间戳大于ttl，分布式锁获取成功，真正的分布式锁时间只有当前时间-第一步的时间戳
4. 如果时间超过了ttl，客户端释放所有节点的锁，防止其他客户端获取锁失败
 
##### Redission
