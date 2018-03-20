title: gemfire使用
date: 2017/6/13 20:46:25
---


### 5.5.1 使用外部配置文件配置region

一般引入cache.xml.

`<gfe:cache cache-xml-location="cache.xml"/>`

`<gfe:lookup-region id="region-bean" name="Orders"/>`

即会自动扫描到cache.xml中的Order region，加载进context中。

### 5.5.2 自动扫描外部文件region

当然，也可以使用全自动扫描

`<gfe:auto-region-lookup/>`

这个标签会把cache.xml中的所有region都注册到spring中

当要使用时，可以如下:

```
package example;

import ...

@Repository("appDao")
@DependsOn("gemfireCache")
public class ApplicationDao extends DaoSupport {

  @Resource(name = "Parent")
  private Region<?, ?> parent;

  @Resource(name = "/Parent/Child")
  private Region<?, ?> child;

  ...
}
```

### 5.5.3 regions 配置



### 5.5.3.1 Cache Listeners

它会监听配置对应的region事件

```
public class MyListener implements CacheListener<Object,Object> {
    public void afterCreate(EntryEvent<Object, Object> entryEvent) {
        System.out.println(entryEvent.getKey() + ":" + entryEvent.getNewValue());
    }

    public void afterUpdate(EntryEvent<Object, Object> entryEvent) {

    }

    public void afterInvalidate(EntryEvent<Object, Object> entryEvent) {

    }

    public void afterDestroy(EntryEvent<Object, Object> entryEvent) {

    }

    public void afterRegionInvalidate(RegionEvent<Object, Object> regionEvent) {

    }

    public void afterRegionDestroy(RegionEvent<Object, Object> regionEvent) {

    }

    public void afterRegionClear(RegionEvent<Object, Object> regionEvent) {

    }

    public void afterRegionCreate(RegionEvent<Object, Object> regionEvent) {

    }

    public void afterRegionLive(RegionEvent<Object, Object> regionEvent) {

    }

    public void close() {

    }
}
```

```xml
 <gfs:client-region id="test1" shortcut="PROXY">
            <gfs:cache-listener>
                <bean class="com.ttaocloud.listener.MyListener"/>
            </gfs:cache-listener>
  </gfs:client-region>
 
```

### 5.5.3.2 Cache Loaders and Cache Writers



### 5.5.3.3 Subregions(子region)

```xml
<gfe:replicated-region name="Customer">
        <gfe:replicated-region name="Address"/>
    </gfe:replicated-region>

    <gfe:replicated-region name="Employee">
        <gfe:replicated-region name="Address"/>
    </gfe:replicated-region>
```



### 5.5.4 Region Templates (Region 配置模版)

| Tag Name                            | Description                              |
| ----------------------------------- | ---------------------------------------- |
| `<gfe:region-template>`             | Defines common, generic Region attributes; extends `regionType` in the SDG 1.5 XSD |
| `<gfe:local-region-template>`       | Defines common, 'Local' Region attributes; extends `localRegionType` in the SDG 1.5 XSD |
| `<gfe:partitioned-region-template>` | Defines common, 'PARTITION' Region attributes; extends `partitionedRegionType` in the SDG 1.5 XSD |
| `<gfe:replicated-region-template>`  | Defines common, 'REPLICATE' Region attributes; extends `replicatedRegionType` in the SDG 1.5 XSD |
| `<gfe:client-region-template>`      | Defines common, 'Client' Region attributes; extends `clientRegionType` in the SDG 1.5 XSD |

配置事例：

```Xml
<gfe:async-event-queue id="AEQ" persistent="false" parallel="false" dispatcher-threads="4">
  <gfe:async-event-listener>
    <bean class="example.AeqListener"/>
  </gfe:async-event-listener>
</gfe:async-event-queue>

<gfe:region-template id="BaseRegionTemplate" cloning-enabled="true"
    concurrency-checks-enabled="false" disk-synchronous="false"
    ignore-jta="true" initial-capacity="51" key-constraint="java.lang.Long"
    load-factor="0.85" persistent="false" statistics="true"
    value-constraint="java.lang.String">
  <gfe:cache-listener>
    <bean class="example.CacheListenerOne"/>
    <bean class="example.CacheListenerTwo"/>
  </gfe:cache-listener>
  <gfe:entry-ttl timeout="300" action="INVALIDATE"/>
  <gfe:entry-tti timeout="600" action="DESTROY"/>
</gfe:region-template>

<gfe:region-template id="ExtendedRegionTemplate" template="BaseRegionTemplate"
    index-update-type="asynchronous" cloning-enabled="false"
    concurrency-checks-enabled="true" key-constraint="java.lang.Integer"
    load-factor="0.55">
  <gfe:cache-loader>
    <bean class="example.CacheLoader"/>
  </gfe:cache-loader>
  <gfe:cache-writer>
    <bean class="example.CacheWriter"/>
  </gfe:cache-writer>
  <gfe:membership-attributes required-roles="readWriteNode" loss-action="limited-access" resumption-action="none"/>
  <gfe:async-event-queue-ref bean="AEQ"/>
</gfe:region-template>

<gfe:partitioned-region-template id="PartitionRegionTemplate" template="ExtendedRegionTemplate"
    copies="1" local-max-memory="1024" total-max-memory="16384" recovery-delay="60000"
    startup-recovery-delay="15000" enable-async-conflation="false"
    enable-subscription-conflation="true" load-factor="0.70"
    value-constraint="java.lang.Object">
  <gfe:partition-resolver>
    <bean class="example.PartitionResolver"/>
  </gfe:partition-resolver>
  <gfe:eviction type="ENTRY_COUNT" threshold="8192000" action="OVERFLOW_TO_DISK"/>
</gfe:partitioned-region-template>

<gfe:partitioned-region id="TemplateBasedPartitionRegion" template="PartitionRegionTemplate"
    copies="2" local-max-memory="8192" total-buckets="91" disk-synchronous="true"
    enable-async-conflation="true" ignore-jta="false" key-constraint="java.util.Date"
    persistent="true">
  <gfe:cache-writer>
    <bean class="example.CacheWriter"/>
  </gfe:cache-writer>
  <gfe:membership-attributes required-roles="admin,root" loss-action="no-access" resumption-action="reinitialize"/>
  <gfe:partition-listener>
    <bean class="example.PartitionListener"/>
  </gfe:partition-listener>
  <gfe:subscription type="ALL"/>
</gfe:partitioned-region>	
```

子模版若有于父模版相同的属性，则会覆盖父模版



### 5.5.6 Data Persistence(数据持久化)

Region 是可以使其数据持久化的，方便后面系统重启或者出现意外时能恢复数据

`<gfe:partitioned-region id="persitent-partition" persistent="true"/>`

只要设置`persistent=true`就就可以实现

当然，也是可以使用data-policy，这个必须跟region的type对应。

`<gfe:partitioned-region id="persitent-partition" data-policy="PERSISTENT_PARTITION"/>`

如果当设置了`data-policy="PERSISTENT_PARTITION"` 还设置`persistent="false"`,就会抛出`initialization exception`

在使用了数据持久化，最好设置`disk-store-ref`属性，这样能时其更有效率。

`disk-synchronous`属性是设置synchronously or asynchronously写入磁盘

`<gfe:partitioned-region id="persitent-partition" persistent="true" disk-store-ref="myDiskStore" disk-synchronous="true"/>`



### 5.5.7 Subscription Interest Policy



### 5.5.8 Data Eviction and Overflowing(数据溢出)

Spring Data Gemfire 支持多种数据溢出处理策略，包括一下几种：

- Entry count(实体总数)
- memory (内存大小)
- heap usage

如下，当内存超过512mb时，就会触发写入硬盘的操作，建议同时配上`disk-store`属性。

```xml
<gfe:partitioned-region id="overflow-partition">
     <gfe:eviction type="MEMORY_SIZE" threshold="512" action="OVERFLOW_TO_DISK"/>
</gfe:partitioned-region>	
```



### 5.5.9 Data Expiration 数据有效时间

GemFire 允许设置数据在缓存中的有效时间，支持一下两种

- **Time-to-Live (TTL)** 

  The amount of time, in seconds, the object may remain in the cache after the last creation or update. For entries, the counter is set to zero for create and put operations. Region counters are reset when the Region is created and when an entry has its counter reset.

- **Idle Timeout (TTI)** 

  The amount of time, in seconds, the object may remain in the cache after the last access. The Idle Timeout counter for an object is reset any time its TTL counter is reset. In addition, an entry’s Idle Timeout counter is reset any time the entry is accessed through a get operation or a netSearch . The Idle Timeout counter for a Region is reset whenever the Idle Timeout is reset for one of its entries.

Each of these may be applied to the Region itself or entries in the Region. Spring Data GemFire provides `<region-ttl>`, `<region-tti>`, `<entry-ttl>` and `<entry-tti>`Region child elements to specify timeout values and expiration actions.



### 5.5.10 Annotation-based Data Expiration 基于注解设置有效时间（未完成）

在Spring Data Gemfire1.7后支持

```java
@Expiration(timeout = "1800", action = "INVALIDATE")
public static class SessionBasedApplicationDomainObject {
}
```

```java
@TimeToLiveExpiration(timeout = "3600", action = "LOCAL_DESTROY")
@IdleTimeoutExpiration(timeout = "1800", action = "LOCAL_INVALIDATE")
@Expiration(timeout = "1800", action = "INVALIDATE")
public static class AnotherSessionBasedApplicationDomainObject {
}
```

`@IdleTimeoutExpiration` and `@TimeToLiveExpiration`会优先于`@Expiration`

 Spring Data GemFire do allow you to set Region Expiration using the SDG XML namespace, like so…

```
<gfe:*-region id="Example" persistent="false">
  <gfe:region-ttl timeout="600" action="DESTROY"/>
  <gfe:region-tti timeout="300" action="INVALIDATE"/>
</gfe:*-region>
```



### 5.5.11 Local Region（未完成）



### 5.5.12 Replicated Region(未完成)



### 5.5.13 Partitioned Region(未完成)



### 5.5.14 Client Region(未完成)



### 5.5.14.1 Client Interests(未完成)



## 5.6  Creating an Index 创建索引





## 5.7 Configuring a Disk Store 配置磁盘存储



Example:

```Xml
<gfe:disk-store id="diskStore1" queue-size="50" auto-compact="true"
        max-oplog-size="10" time-interval="9999">
        <gfe:disk-dir location="/gemfire/store1/" max-size="20"/>
        <gfe:disk-dir location="/gemfire/store2/" max-size="20"/>
</gfe:disk-store>
```



## 5.8 Using the GemFire Snapshot Service



## 5.9 Configuring GemFire’s Function Service

