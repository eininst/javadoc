# 分享
## IO请求批量查询
**错误案例**

```python
cities = City.query.all()
for city in cities:
    city_info = CityInfo.query.filter(CityInfo.city_id == city.id).first()
```
**正确案例**

```python
cities = City.query.all()
city_ids = [ct.id for ct in cities]
city_info_list = CityInfo.query.filter(CityInfo.city_id.in_(city_ids)).all()
city_info_map = {ct_info.city_id: ct_info for ct_info in city_info_list}

for city in cities:
    if city.id not in city_info_map:
        continue
    city_info = city_info_map[city.id]
```
> IO请求能一次批量获取的，就不要循环单个获取, 比如数据库查询, http请求等等

## 尽量避免JOIN
**业务代码要尽量JOIN查询, 走单表查询, 需要关联的数据在代码层组装**

```sql
select * from city ct 
inner join city_info ci on ci.id = ct.id

-- 应该分成2条SQL查询
select * from city;
select * from city_info where city_id in (...);
```
**在代码层组装**

```python
city_info_map = {ct_info.city_id: ct_info for ct_info in city_info_list}
for city in cities:
    if city.id not in city_info_map:
        continue
    city_info = city_info_map[city.id]
```
> 这样做的好处有很多, 后续 统一打缓存, 分库分表非常方便, 从设计上避免写出慢SQL, 优化索引也简单


## BeanUtils
**对象转换帮助类(python2.7版本)**

> convert
> 
> json2obj
> 
> obj2json
> 
> dict2obj
> 
> obj2dict

#### ORM对象转DTO, Dict, Json

```python
class OrderEtaConfigDTO:
    start = None  # type:int
    end = None  # type:int
    time = None  # type: int
 
order_eta_configs = OrderEtaConfig.query.all()
order_eta = OrderEtaConfig.query.first()

#转DTO
config_dtos = BeanUtils.convert(order_eta_configs, OrderEtaConfigDTO)
conf_dto = BeanUtils.convert(order_eta, OrderEtaConfigDTO)

#转dict
diict_list = BeanUtils. obj2dict(order_eta_configs)
diict = BeanUtils. obj2dict(order_eta)

#转json str
BeanUtils. obj2json(order_eta_configs)
BeanUtils. obj2json(order_eta)
```

#### DTO对象转ORDER

```python
model = Model(**BeanUtils.obj2dict(dto))
```


## Requests
### 设置timeout

requests.get(url, timeout=10)
> 默认timeout是None, 表示没有限制, **永久等待!!!**
>
> **一定要设置timeout**


### Http连接池

HTTP协议是无状态的协议，即每一次请求都是互相独立的。因此它的最初实现是，每一个http请求都会打开一个tcp socket连接，当交互完毕后会关闭这个连接。

所以建立连接与断开连接是要经过三次握手与四次挥手的。显然在这种设计中，每次发送Http请求都会消耗很多的额外资源，即连接的建立与销毁。


 **在HTTP/1.1协议 支持了长链接协议， 我们可以配合连接池提前建立好连接，减少这本部分消耗**
>  性能大概会提升3倍左右

### 使用Request Session
requests 已经封装好了Session的长连接类, 可以直接使用

```python

session = requests.Session()
session.mount('http://', HTTPAdapter(pool_connections=10, pool_maxsize=10, max_retries=2))
session.mount('https://', HTTPAdapter(pool_connections=10, pool_maxsize=10, max_retries=2))
session.get(url, timeout=10)
        
```
> pool_connections 连接池大小,  要维持多少连接
> 
> pool_maxsize 最大支持多少连接
> 
> max_retries 请求失败的重试次数，默认0




## 外卖管理台权限
**权限目前是控制到页面级别,  既不同的账号 看到的页面不一样**

* 账号表 -> 角色表, 一对多
* 角色 -> 菜单, 一对多

#### 定义一个菜单
```python
mod = Blueprint('lab', __name__, url_prefix='/lab')

@mod.route('config_index', methods=['GET'], is_menu=True, menu_name=u'实验功能配置')
@requires_login
def config_index():
    return render_template('lab/config.html')
```
> 定义路由时， 传入 **is_menu**参数， 声明是一个菜单路由,  menu_name设置菜单名称

#### 控制菜单的接口权限
例如上面的例子, lab/config.html页面 可以访问哪些接口?

* 菜单接口访问的隔离是按照 **Blueprint** 级别的
* 既 lab/config.html页面 只能访问 Blueprint('lab', __name__, url_prefix='/lab') 这个蓝图 定义的接口

> 管理台一个页面对应一个蓝图 ,  这样开发才能保证接口的权限



## SQS
### Producer (生产者)
**参数说明：** 

|参数名|必传|类型|说明|
|:----    |:---|:----- |-----   |
|address |是  |string |队列名称   |
|msg  |是  |object | 消息内容, 可以被json序列化的对象    |
|key      |否  |string | 该队列下的唯一标示,  出现问题，可以快速精确定位到消息 |
|group_id  |否  |string | 分组名， 有序队列必须设置该参数， 用于标示，消息在该分组下有序   |
|delay_seconds  |否  |int | 延迟xx秒后再发送, 最大支持 7200 秒|

#### 发送无序消息
```python
producer.publish("test", "hello world")
```

#### 发送有序消息
```python
producer.publish("test.fifo", "hello world", group_id="test")
```

#### 发送延迟消息
```python
producer.publish("test.fifo", "hello world", delay_seconds=30)
```

#### 批量发送
```python
producer.session.add("test", "hello world")
producer.session.add("test", "hello world")
producer.session.add("test.fifo", "hello world", delay_seconds=30)
producer.session.add("test.fifo", "hello world", delay_seconds=30)
producer.session.commit()
```

> 如果一个请求 会涉及到 发送多个消息, 建议都使用Session模式, 进行批量发送

1. `add会把要发送的消息保存 request context (g)`
2. `commit的时候会调用批量发送接口统一发送消息`
3. `和 db.session.add 用法类似`


#### 有序和无序队列区别:
`无序队列:`

1. 无序队列只能保证至少收到一次消息
2. 支持接近无限的每秒吞吐量, 加硬件就可以提示吞吐量
3. 支持延迟消息
3. 尽可能的有序

`有序队列:`

1. 保证先发的消息一定会先收到
2. 不支持延迟消息
3. 每秒最大事务数3000


### Consumer (消费者)
**参数说明：** 

|参数名|必传|类型|说明|
|:------------------|:------------------|:----- |-----   |
| address |是  |string |队列名称   |
| call  |是  | function | 处理消息的函数    |
| consumer_ thread_ number  |否  |int | 消费线程数(默认4个) |
| msg_lifetime  |否  |int | 消息过期时间, 多少秒后没收到消息丢弃消息(默认 86400 * 3)   |
| visibility_ timeout_ seconds  |否  |int | 可见性超时(默认300秒) |
| is_retry  |否  | bool | 业务抛出异常是否进行重试(默认True) |

#### 可见性超时
* 在收到消息后，消息将立即保留在队列中。为防止其他使用者再次处理消息，Amazon SQS 设置了可见性超时，这是 Amazon SQS 防止其他使用者接收和处理消息的时间段。消息的默认可见性超时为 30 秒。最小值为 0 秒。最大值为 12 小时

* 对于标准队列,可见性超时不能保证不会两次收到消息。将订阅方程序设计为幂等 应用程序 (多次处理同一消息时，它们不应受到不利影响)

* 程序处理时间 超过 可见性超时, 其他服务器 将可以获取到该消息

> 重试机制就是利用可见性超时实现的， 订阅方如果抛出异常， 消息将不会被删除， 等待到达可见性超时时间之后再次获取到该消息

## DB
* 数据库表 禁止使用 外键约束
* 给识别读比较高的字段建索引, 唯一性太差的字段不适合单独创建索引
* 尽量避免使用子查询，用join代替(只要差别在于子查询会有创建临时表的消耗)
*  明确知道值是唯一的情况下， 使用**唯一索引**，默认情况下性能比普通索引要好， 普通索引在匹配到值的之后，还会继续往下找，除非手动指定**limit 1;**
*  多使用explain 检查 索引使用情况和扫描行数
*  合理使用联合索引 (最左匹配原则)

**执行原生SQL的语句, 使用text函数包装, 防止SQL注入**

```python
sql = text("""
 select id,(st_distance(center, point(:lng, :lat)) * 111195) AS distance 
 from hive where MBRWithin(st_geomfromtext('POINT(:lng :lat)'),polys) and tag = :tag and city_id = :city_id
 order by distance limit 1;
""")
result = db.engine.execute(sql, {
    'city_id': city_id,
    'lng': float(lng),
    'lat': float(lat),
    'tag': tag
})
```

**批量插入数据使用,insert values模式**
> modes = [...] 
> 
> db.session.bulk_save _objects(models)

#### 模糊查询
按照字符串最左匹配原则 可以使用到索引, 长字符串尽量使用前缀 INDEX(name(6))

```sql
select * from `item` where name like '一%';
```
#### fulltext
1.  模糊查询怎么办，这时我们需要使用到fulltext索引，但是需要注意只有MyISAM这个引擎的MySQL才能支持
2. 对中文支持不好，主要是中文分词

如果使用fulltext查询不到结果那么可能有以下几个问题：

1. 你的数据库引擎不对
2. 中文分词
3. MySQL有一个设置：ft_min_word_len这个是最短的索引字符串，默认为4，如果查询的字符串小于这个值，也查下不到
4. 如果你的关键词频率超过了50%，使用默认的查询方式（自然语言）是查询不到的，需要好似用IN BOOLEAN MODE模式才可以查询到

### 什么时候使用master query ?
* 依赖数据最新状态进行操作的业务
* 基于原数据，要去修改的业务



## 乐观锁
### 什么时候使用乐观锁?
1. 在并发更新数据的时候, 保证数据一致性, 例如扣库存， 修改订单状态等
2. 适用于竞争不激烈的场景， 对性能要求不高的业务

### 乐观锁原理
* 表设计时，需要往表里加一个version字段
* 更新数据时，判断数据库的version与当前对象的version 值是否相同，如果相同则更新数据, 并把version 进行+1
* 如果version不一致, 则说明，该数据并发了， 已被人修改过了
* 进行自旋轮训，重新获取数据进行更新， 直到更新成功


### 乐观锁助手
**首先在model 里面增加 version_id 字段**

```python
# 默认version_id 会自增
class User(db.Model):
    __tablename__ = 'user'

    id = Column(Integer, primary_key=True)
    version_id = Column(Integer, nullable=False)
    name = Column(String(50), nullable=False)

    __mapper_args__ = {
        "version_id_col": version_id
    }
    
# Custom Version Counters / Types
import uuid

class User2(db.Model):
    __tablename__ = 'user'

    id = Column(Integer, primary_key=True)
    version_uuid = Column(String(32), nullable=False)
    name = Column(String(50), nullable=False)

    __mapper_args__ = {
        'version_id_col':version_uuid,
        'version_id_generator':lambda version: uuid.uuid4().hex
    }
```

 **然后在执行修改的方法上加 @version_lock修饰器，@version_lock修饰器示例说明如下**

 ```python
@version_lock 表示：
# 最大重试5次，每次重试前等待0.01秒
# 超过重试此时后抛出ServiceException异常, code:400, msg: The network is busy, please try again later

@version_lock(max_retry_num=6,retry_sleep_time=5,try_max_end_exception=ServiceException(code=401,msg='我错了'))
# 最大重试6次，每次重试前等待5秒
# 超过重试此时后抛出ServiceException异常, code:401, msg: 我错了

@version_lock(try_max_end_return=(True,12))
# 最大重试5次，每次重试前等待0.01秒
# 超过重试此时后 返回(True,12)

@version_lock(try_max_end_return=return_function)
# 最大重试5次，每次重试前等待0.01秒
# 超过重试此时后 执行 return_function 方法 并返回返回值

@version_lock(try_exception_run_fun=try_exception_run_fun)
# 最大重试5次，每次重试前 执行stale_data_error_run_fun 方法并等待0.01秒
# 超过重试此时后抛出ServiceException异常, code:400, msg: The network is busy, please try again later

# :param max_retry_num: 最大重试次数
# :param retry_sleep_time: 重试时睡眠时间(秒)
# :param try_max_end_return: 达到最大次数后 （返回的 tuple 或者 运行的fun） 如果没有将抛出异常
# :param try_max_end_exception: 自定义抛出异常  （默认抛出 ServiceException(code=400, msg='xx') ）
# :param try_exception_run_fun:  抛出 StaleDataError 时 可以执行的function 参数中有 retry_exception 为 拦截的异常
 ```
 
 
## Redis
**不要依赖Redis来做重要数据的持久化, 仅用于缓存数据**

* 查询数据尽量使用 redis_ro _store
*  操作数据使用redis_store
*  不要单独使用 redis_store.expire(60),  应当在设置key的时候设置过期时间 eg: redis _store.set(key, val, expire=7200)
* 避免使用delete操作(性能不好), 尽量使用过期时间删除数据
*  避免使用keys xxx*, scan等范围查询命令

#### 批量操作, 使用pipeline模式
```python
pipe = redis_store.pipeline()
for _ in range(10):
	pipe.hset(key, id, val)
pipe.execute()
```
### 数据缓存案例
```python
data = redis_ro_store.client.get(somekey)
if data:
    return json.loads(data)
else:
    # 设置缓存
    model = SomeModel.query.get(id)
    data = BeanUtils.obj2dict(model)
    redis_store.set(key, json.dumps(data), expire=7200)
    return data

```



### 使用RedisLock
#### 什么情况下需要加锁?

当多个请求、多个线程等, 用户同时竞争同一个资源时，需要加锁。比如，下订单减库存，抢票，抢红包等。如果在此处没有锁的控制，会导致很严重的问题，下订单减库存的时候不加锁，会导致商品超卖; 抢票的时候不加锁，会导致两个人抢到同一个位置;


#### 常见的分布式锁
1. 基于数据库实现分布式锁 (利用 select … where … for update 排他锁 或是 乐观锁)
2. RedisLock
3. 基于Zookeeper实现分布式锁


#### 对比
> 数据库

* 简单, 容易理解和实现
* db操作性能较差，并且有锁表的风险
* 非阻塞操作失败后，需要轮询，占用cpu资源;
* 长时间不commit或者长时间轮询，可能会占用较多连接资源

> Redis

* 性能好
* 过期时间不好控制
* 非阻塞，需要有锁的情况，需要自旋轮训 排队等待, 占用cpu资源;

> ZK

* 更可靠
* 性能不如redis的实现

```
zk锁性能比redis低的原因：zk中的角色分为leader，flower，
每次写请求只能请求leader，leader会把写请求广播到所有flower，
如果flower都成功才会提交给leader，其实这里相当于一个2PC的过程。
在加锁的时候是一个写请求，当写请求很多时，zk会有很大的压力，最后导致服务器响应很慢
```
	
#### RedisLock不靠谱的情况:
    例如一主二从。我们的set命令对应的数据写到主库，然后同步到从库。
    当我们申请一个锁的时候，对应就是一条命令 setnx mykey myvalue ，在redis sentinel集群中，这条命令先是落到了主库。
    假设这时主库down了，而这条数据还没来得及同步到从库，sentinel将从库中的一台选举为主库了。
    这时，我们的新主库中并没有mykey这条数据，若此时另外一个client执行 setnx mykey hisvalue , 也会成功，即也能得到锁。
    这就意味着，此时有两个client获得了锁。
    
    不过这种是概率极低的现象， 只会在主从集群redis, 当主实例挂掉的情况 可能会出现
    
   
### RedisLock使用方式
```python
# lock_name可以理解为锁的范围， 比如lock_name=update_odb_order_123 代表对123这个订单加锁

with RedisLock(lock_name='update_odb_order_123', timeout=30, expire=30) as is_success:
    if is_success:
        # 加锁成功
        order = Order.query.get(123)
        update_odb(order)
    else:
        return

# 在这里面使用update_odb更新order为123的相关信息是原子操作
```

**1. timeout 等待锁的超时时间(秒)**

**2. expire 锁的存活时间(秒), (在redis突然挂掉的时候,用于防止死锁现象)**

> 锁的时间控制， 要预估业务代码的耗时， 如果业务代码执行时间超过timeout(等待锁的时间), 那么其他用户就会进来, **is_success** 就会返回False
> 
> 不控制is_success(是否加锁成功的话),  可以理解为只能保证尽可能的 **"原子性"**
