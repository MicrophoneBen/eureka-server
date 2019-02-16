# eureka-server

### 1.master版本
有如下4个服务：
- eureka-server
- bank-server
- company-server
- user-server

此版本演示了2种场景：
#### 1.consumer -> provider
bank-server -> user-server
#### 2.consumer -> provider1，provider2
bank-server -> user-server,company-server

### 2.0.1版本
有如下4个服务：
- eureka-server
- bank-server
- company-server
- user-server

此版本演示了1种场景：
#### 1.consumer -> provider1->provider2
bank-server -> company-server -> user-server

此场景下，company-server在链路中，不仅是provider，也是consumer。


### 3.角色转换
一个provider转变为consumer时，会涉及到调用其他服务，feign调用时，这个api接口所在包，需要显式的配置扫描。

比如，我feign包下，有个UserServiceApi，里面是comsumer调用provider的api接口，那需要在consumer项目中配置：

@EnableFeignClients("com.java4all.feign")

### 4.涉及变量传递
参考0.1版本
#### 1.自行处理
由于tcc的每一个阶段，使用的参数是相同的，
try中如果对接口参数做了任何处理，那么在cc阶段，也需要做相同的处理。
比如，controller中，方法中拿到参数后做了处理，然后去调用的try的实现，
那么这个处理，必须也同步在confirm和cancel中。

此种方式，比较原始，建议在框架层面来处理。

#### 2.框架处理
借助CompensableContextAware，CompensableContext，
在try阶段，把处理后的值存入上下文中，cc阶段直接获取使用即可。

set value参考：https://github.com/distributed-demo/company-server/blob/0.1/src/main/java/com/java4all/service/impl/CompanyServiceImpl.java

get value参考：https://github.com/distributed-demo/company-server/blob/0.1/src/main/java/com/java4all/service/impl/CompanyServiceConfirm.java

### 5.负载粒度
ByteTCC计划对负载均衡的支持粒度，可分为两种：a、按事务进行负载均衡；b、按请求进行负载均衡。
#### 1.按事务进行负载均衡
在某个事务T内，consumer端应用app1首次向provider端应用app2（集群环境）发起请求时，
ByteTCC使用random负载均衡策略将其随机分发到一个app2实例（如inst2）；

后续app1在该事务T内再次向app2发起请求时，将始终落在inst2（即首次请求的处理实例）上。
#### 2.按请求进行负载均衡
consumer端应用app1向provider端应用app2（集群环境）发起请求时，
ByteTCC始终按业务系统指定的负载均衡策略将请求分发到一个app2实例。

### 6.0.2版本
有如下5个服务：
- eureka-server 8761
- eureka-server2 8762
- bank-server
- company-server
- user-server

eureka-server,eureka-server2为一个eureka集群，其他服务注册到eureka-server上。
##### 其他信息可参考文档：https://github.com/liuyangming/ByteTCC/wiki/User-Guide-0.5.x
