# redis cluster集群安装

## 一、集群安装

当前使用一台虚拟机启动6个实例，多虚拟机安装应该是一样的步骤

### 1、下载redis安装包

中文官方网站[http://www.redis.cn/download.html](http://www.redis.cn/download.html) 下载redis安装包，文章使用的是redis-5.0.8.tar.gz

### 2、解压

```
tar -zxvf redis-5.0.8.tar.gz
mv redis-5.0.8 redis
```

### 3、移动到/usr/local/src

```
mv redis /usr/local/src/
```

### 4、安装实例

```
cd /usr/local/src/redis
make PREFIX=/usr/local/redis install
```

### 5、建配置文件目录

在/usr/local/redis/ 新建 etc目录 在etc目录下新建r1、r2、r3、r4、r5、r6目录，拷贝源码包中的redis.conf到rx目录下

```
mkdir -p /usr/local/redis/etc/r1
mkdir -p /usr/local/redis/etc/r2
mkdir -p /usr/local/redis/etc/r3
mkdir -p /usr/local/redis/etc/r4
mkdir -p /usr/local/redis/etc/r5
mkdir -p /usr/local/redis/etc/r6
cd /usr/local/src/redis/
cp redis.conf /usr/local/redis/etc/r1/
cp redis.conf /usr/local/redis/etc/r2/
cp redis.conf /usr/local/redis/etc/r3/
cp redis.conf /usr/local/redis/etc/r4/
cp redis.conf /usr/local/redis/etc/r5/
cp redis.conf /usr/local/redis/etc/r6/
```

### 6、修改各个配置文件redis.conf

（1）r1配置文件 配置完成后拷贝这个文件到其他目录，剩下的只需要修改不同的部分即可。

```
bind 0.0.0.0

#关闭保护模式，通过网络连接连接
protected-mode no

# 修改端口
port 6379

# 打开内存注释 设置200M 根据实际情况定义
maxmemory 241172480 

# 设置后台运行
daemonize yes

# 设置pid file
pidfile /var/run/redis_6379.pid

# 设置log file
logfile /var/log/redis_6379.log

# 配置主从策略 默认yes，
# 当从节点和主节点的连接断开或者复制正在进行中，如果设置为yes，那么继续提供服务，
# 如果设置为no，那么返回sync with master in progress
replica-serve-stale-data yes

# 配置从节点是否是只读，默认是只读
replica-read-only yes

# 设置aof刷新
appendonly yes
appendfilename "appendonly.aof"

# 将cluster-enabled 注释打开
cluster-enabled yes

# 设置集群配置文件，这个是由redis自己生成的
cluster-config-file /usr/local/redis/etc/r1/nodes-6379.conf

# 设置集群超时时间
cluster-node-timeout 15000

#设置密码 打开requirepass 注释
requirepass 123456
```

（2）r2配置文件，只记录不同的地方

```
port 6380
pidfile /var/run/redis_6380.pid
logfile /var/log/redis_6380.log
cluster-config-file /usr/local/redis/etc/r2/nodes-6380.conf
```

（3）r3配置文件，只记录不同的地方

```
port 6381
pidfile /var/run/redis_6381.pid
logfile /var/log/redis_6381.log
cluster-config-file /usr/local/redis/etc/r3/nodes-6381.conf
```

（4）r4配置文件，只记录不同的地方

```
port 6382
pidfile /var/run/redis_6382.pid
logfile /var/log/redis_6382.log
cluster-config-file /usr/local/redis/etc/r4/nodes-6382.conf
```

（5）r5配置文件，只记录不同的地方

```
port 6383
pidfile /var/run/redis_6383.pid
logfile /var/log/redis_6383.log
cluster-config-file /usr/local/redis/etc/r5/nodes-6383.conf
```

（6）r6配置文件，只记录不同的地方

```
port 6384
pidfile /var/run/redis_6384.pid
logfile /var/log/redis_6384.log
cluster-config-file /usr/local/redis/etc/r6/nodes-6384.conf
```

### 7、启动实例

```
/usr/local/redis/bin/redis-server /usr/local/redis/etc/r1/redis.conf
/usr/local/redis/bin/redis-server /usr/local/redis/etc/r2/redis.conf
/usr/local/redis/bin/redis-server /usr/local/redis/etc/r3/redis.conf
/usr/local/redis/bin/redis-server /usr/local/redis/etc/r4/redis.conf
/usr/local/redis/bin/redis-server /usr/local/redis/etc/r5/redis.conf
/usr/local/redis/bin/redis-server /usr/local/redis/etc/r6/redis.conf
```

### 8、创建集群

关联集群节点&#x20;

**注意：** 这个地方一定要注意、一定要按照实际的IP地址来配置，下面的127.0.0.1 会导致java程序连接时报错

```
/usr/local/redis/bin/redis-cli --cluster create 127.0.0.1:6379 127.0.0.1:6380 127.0.0.1:6381 127.0.0.1:6382 127.0.0.1:6383 127.0.0.1:6384 --cluster-replicas 1 -a 123456
```

**最终执行命令如下：**

```
/usr/local/redis/bin/redis-cli --cluster create 192.168.198.138:6379 192.168.198.138:6380 192.168.198.138:6381 192.168.198.138:6382 192.168.198.138:6383 192.168.198.138:6384 --cluster-replicas 1 -a 123456
```

如果有密码的话，需要添加 -a 123456（123456是我设置的密码）&#x20;

\--cluster-replicas 1  配置，1代表的是一个master有一个slave；前三个ip是master,后三个ip是对应的slave。

**执行完成的结果**

```
[root@localhost etc]# /usr/local/redis/bin/redis-cli --cluster create 192.168.198.138:6379 192.168.198.138:6380 192.168.198.138:6381 192.168.198.138:6382 192.168.198.138:6383 192.168.198.138:6384 --cluster-replicas 1 -a 123456
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 192.168.198.138:6383 to 192.168.198.138:6379
Adding replica 192.168.198.138:6384 to 192.168.198.138:6380
Adding replica 192.168.198.138:6382 to 192.168.198.138:6381
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: b439ee4f7e957449dccd31b237dd56335be226da 192.168.198.138:6379
   slots:[0-5460] (5461 slots) master
M: bc7fef53c9dc5a763d30f7e258237e5622e0dfec 192.168.198.138:6380
   slots:[5461-10922] (5462 slots) master
M: abf4ce7052785293a3d9acd36912e51e0aa26430 192.168.198.138:6381
   slots:[10923-16383] (5461 slots) master
S: d7329e9d622bd39814c7f675e75ed070745fe440 192.168.198.138:6382
   replicates abf4ce7052785293a3d9acd36912e51e0aa26430
S: b969cd9203e757ef9fd2a8ea104e42e395b0d39b 192.168.198.138:6383
   replicates b439ee4f7e957449dccd31b237dd56335be226da
S: 7642371db0a0fdb1cba44d165165bf2b9ecbd14a 192.168.198.138:6384
   replicates bc7fef53c9dc5a763d30f7e258237e5622e0dfec
Can I set the above configuration? (type 'yes' to accept): yesa
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
....
>>> Performing Cluster Check (using node 192.168.198.138:6379)
M: b439ee4f7e957449dccd31b237dd56335be226da 192.168.198.138:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: abf4ce7052785293a3d9acd36912e51e0aa26430 192.168.198.138:6381
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: b969cd9203e757ef9fd2a8ea104e42e395b0d39b 192.168.198.138:6383
   slots: (0 slots) slave
   replicates b439ee4f7e957449dccd31b237dd56335be226da
S: 7642371db0a0fdb1cba44d165165bf2b9ecbd14a 192.168.198.138:6384
   slots: (0 slots) slave
   replicates bc7fef53c9dc5a763d30f7e258237e5622e0dfec
M: bc7fef53c9dc5a763d30f7e258237e5622e0dfec 192.168.198.138:6380
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: d7329e9d622bd39814c7f675e75ed070745fe440 192.168.198.138:6382
   slots: (0 slots) slave
   replicates abf4ce7052785293a3d9acd36912e51e0aa26430
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

### 9、查看集群状态

使用命令行工具连接进去，并输入相关命令

&#x20;/usr/local/redis/bin/redis-cli -c -p 6379&#x20;

cluster info

&#x20;cluster nodes

```
192.168.198.138:6381> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:3
cluster_stats_messages_ping_sent:3106
cluster_stats_messages_pong_sent:3250
cluster_stats_messages_meet_sent:3
cluster_stats_messages_sent:6359
cluster_stats_messages_ping_received:3247
cluster_stats_messages_pong_received:3109
cluster_stats_messages_meet_received:3
cluster_stats_messages_received:6359
192.168.198.138:6381> cluster nodes
b439ee4f7e957449dccd31b237dd56335be226da 192.168.198.138:6379@16379 master - 0 1621911894248 1 connected 0-5460
d7329e9d622bd39814c7f675e75ed070745fe440 192.168.198.138:6382@16382 slave abf4ce7052785293a3d9acd36912e51e0aa26430 0 1621911896000 4 connected
b969cd9203e757ef9fd2a8ea104e42e395b0d39b 192.168.198.138:6383@16383 slave b439ee4f7e957449dccd31b237dd56335be226da 0 1621911898279 5 connected
bc7fef53c9dc5a763d30f7e258237e5622e0dfec 192.168.198.138:6380@16380 master - 0 1621911897269 2 connected 5461-10922
abf4ce7052785293a3d9acd36912e51e0aa26430 192.168.198.138:6381@16381 myself,master - 0 1621911894000 3 connected 10923-16383
7642371db0a0fdb1cba44d165165bf2b9ecbd14a 192.168.198.138:6384@16384 slave bc7fef53c9dc5a763d30f7e258237e5622e0dfec 0 1621911896262 6 connected
192.168.198.138:6381>
```

### 10、spring连接集群并测试

**Redis配置类**

```
package com.test.config;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import io.lettuce.core.TimeoutOptions;
import io.lettuce.core.cluster.ClusterClientOptions;
import io.lettuce.core.cluster.ClusterTopologyRefreshOptions;
import org.apache.commons.pool2.impl.GenericObjectPoolConfig;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.data.redis.RedisProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.env.Environment;
import org.springframework.core.env.MapPropertySource;
import org.springframework.data.redis.connection.RedisClusterConfiguration;
import org.springframework.data.redis.connection.lettuce.LettuceClientConfiguration;
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
import org.springframework.data.redis.connection.lettuce.LettucePoolingClientConfiguration;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.time.Duration;
import java.util.HashMap;
import java.util.Map;

@Configuration
public class RedisConfig {

    @Autowired
    private Environment environment;

    @Autowired
    RedisProperties redisProperties;

    @Bean
    public RedisTemplate<String, Object> redisTemplate(LettuceConnectionFactory lettuceConnectionFactory) {
        //值序列化
        ObjectMapper om = new ObjectMapper();
        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<Object>(Object.class);

        //解决查询缓存转换异常的问题
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);

        RedisTemplate<String, Object> template = new RedisTemplate<String, Object>();
        RedisSerializer<String> stringSerializer = new StringRedisSerializer();
        template.setConnectionFactory(lettuceConnectionFactory);

        //设置字符串
        template.setKeySerializer(stringSerializer);
        template.setValueSerializer(jackson2JsonRedisSerializer);

        //设置hash
        template.setHashKeySerializer(jackson2JsonRedisSerializer);
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        return template;
    }

    @Bean
    public GenericObjectPoolConfig genericObjectPoolConfig() {
        GenericObjectPoolConfig genericObjectPoolConfig = new GenericObjectPoolConfig();
        genericObjectPoolConfig.setMaxIdle(redisProperties.getLettuce().getPool().getMaxIdle());
        genericObjectPoolConfig.setMinIdle(redisProperties.getLettuce().getPool().getMinIdle());
        genericObjectPoolConfig.setMaxTotal(redisProperties.getLettuce().getPool().getMaxActive());
        genericObjectPoolConfig.setMaxWaitMillis(redisProperties.getLettuce().getPool().getMaxWait().toMillis());
        return genericObjectPoolConfig;
    }

    @Bean
    public LettuceConnectionFactory redisConnectionFactory(GenericObjectPoolConfig genericObjectPoolConfig) {
        Map<String, Object> source = new HashMap<String, Object>();
        source.put("spring.redis.cluster.nodes", environment.getProperty("spring.redis.cluster.nodes"));
        source.put("spring.redis.cluster.timeout", environment.getProperty("spring.redis.cluster.timeout"));
        source.put("spring.redis.cluster.max-redirects", environment.getProperty("spring.redis.cluster.max-redirects"));
        RedisClusterConfiguration redisClusterConfiguration = new RedisClusterConfiguration(new MapPropertySource("RedisClusterConfiguration", source));
        redisClusterConfiguration.setPassword(redisProperties.getPassword());

        //支持自适应集群拓扑刷新和静态刷新源
        ClusterTopologyRefreshOptions clusterTopologyRefreshOptions = ClusterTopologyRefreshOptions.builder()
                //.enablePeriodicRefresh(Duration.ofSeconds(5))
                .enableAllAdaptiveRefreshTriggers()
                .adaptiveRefreshTriggersTimeout(Duration.ofSeconds(10))
                //.dynamicRefreshSources(false)
                //.enablePeriodicRefresh(Duration.ofSeconds(10))
                .build();

        ClusterClientOptions clusterClientOptions = ClusterClientOptions.builder().timeoutOptions(TimeoutOptions.enabled(Duration.ofSeconds(30)))
                //.autoReconnect(false)  是否自动重连
                //.pingBeforeActivateConnection(Boolean.TRUE)
                //.cancelCommandsOnReconnectFailure(Boolean.TRUE)
                //.disconnectedBehavior(ClientOptions.DisconnectedBehavior.REJECT_COMMANDS)
                .topologyRefreshOptions(clusterTopologyRefreshOptions).build();

        LettuceClientConfiguration lettuceClientConfiguration = LettucePoolingClientConfiguration.builder()
                .poolConfig(genericObjectPoolConfig)
                //.readFrom(ReadFrom.NEAREST)
                .clientOptions(clusterClientOptions).build();

        return new LettuceConnectionFactory(redisClusterConfiguration, lettuceClientConfiguration);
    }
}
```

**application.yml配置** 节点的ip需要根据实际IP进行配置

```
spring:
  redis:
    cluster:
      nodes: 192.168.198.138:6379,192.168.198.138:6380,192.168.198.138:6381,192.168.198.138:6382,192.168.198.138:6383,192.168.198.138:6384
      max-redirects: 3
    password: 123456
    timeout: 3000ms
    lettuce:
      pool:
        max-active: -1
        max-wait: -1ms
        max-idle: 8
        min-idle: 0
```

**存入到redis的基础类**

```
package com.test.controller;

import lombok.Data;

import java.io.Serializable;

@Data
public class TTest implements Serializable {
    private String a;
}
```

**测试类MyTest**

```
package com.test;

import com.test.controller.TTest;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.test.context.junit4.SpringRunner;

import javax.annotation.Resource;

@RunWith(SpringRunner.class)
@SpringBootTest
public class MyTest {

    @Resource
    RedisTemplate<String, Object> redisTemplate;

    @Test
    public void addKeyValue() throws Exception {
        TTest TTest = new TTest();
        TTest.setA("33");
        redisTemplate.opsForValue().set("a", TTest);
    }

    @Test
    public void getKeyValue() throws Exception {
        TTest t = (TTest)redisTemplate.opsForValue().get("a");
        System.out.println(t);
    }
}
```

**查看结果：**

&#x20;运行测试类，登陆redis集群查看执行结果

```
192.168.198.138:6381> get "a"
"[\"com.test.controller.TTest\",{\"a\":\"33\"}]"
```

同时通过java也能获取到结果&#x20;

![](<../.gitbook/assets/image (25).png>)

**最后说明**

集群模式只支持database为0的库

输入命令测试

```
192.168.198.138:6381> select 1
(error) ERR SELECT is not allowed in cluster mode
```

所以在使用集群的时候，java客户端设置不了其他database
