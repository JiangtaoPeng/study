
- 流量分发

- 流协议
	- HTTP
	- HLS
	- RTMP: Real Time Message Protocol

Nginx
- 多进程，一个master进程，多个worker进程
- 进程间通信主要靠共享内存
- epoll
- 

[https://juejin.im/post/5b4bdb54e51d45191e0cd774](https://juejin.im/post/5b4bdb54e51d45191e0cd774)
[https://www.jianshu.com/p/ad43e8188e87](https://www.jianshu.com/p/ad43e8188e87)
[https://www.jianshu.com/p/b3e1dbcd836c?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation](https://www.jianshu.com/p/b3e1dbcd836c?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation)
[https://www.jianshu.com/p/3fd67882e6f6](https://www.jianshu.com/p/3fd67882e6f6)
[https://blog.51cto.com/xpleaf/1903565](https://blog.51cto.com/xpleaf/1903565)



## linux select, poll, epoll
### 关键词
- 用户空间和内核空间
- 进程切换（耗资源）
从一个进程的运行转到另一个进程上运行，这个过程中经过下面这些变化：  
	1. 保存处理机上下文，包括程序计数器和其他寄存器。  
	2. 更新PCB信息。  
	3. 把进程的PCB移入相应的队列，如就绪、在某事件阻塞等队列。  
	4. 选择另一个进程执行，并更新其PCB。  
	5. 更新内存管理的数据结构。  
	6. 恢复处理机上下文。
- 进程的阻塞
	- 只有处于运行态的进程（获得CPU），才可能将其转为阻塞状态。当进程进入阻塞状态，是不占用CPU资源的
- 文件描述符
	- 表述指向文件的引用的抽象化概念
- 缓存I/O
	- 数据在传输过程中需要在应用程序地址空间和内核进行多次数据拷贝操作，这些数据拷贝操作所带来的 CPU 以及内存开销是非常大的
	- 一次I/O读取操作，会先把数据拷贝到内核缓存IO然后再拷贝用户空间的程序空间内存
- 同步和异步
- 阻塞和非阻塞
	- 阻塞：数据未准备好
	- 非阻塞：程序不断的主动问询数据是否准备好
- IO多路复用
	- I/O 多路复用的特点是通过一种机制一个进程能同时等待多个文件描述符，而这些文件描述符（套接字描述符）其中的任意一个进入读就绪状态，select()函数就可以返回。
	- 如果处理的连接数不是很高的话，使用select/epoll的web server不一定比使用multi-threading + blocking IO的web server性能更好，可能延迟还更大。select/epoll的优势并不是对于单个连接能处理得更快，而是在于能处理更多的连接。


## nginx/openresty/kong
- nginx是一个模块化设计的反向代理软件
- openresty是基于nginx的web开发平台
- kong是openresty的一个应用，即api gateway，具有api请求和代理的功能 


## nginx架构
- nginx运行在企业内网的最外层，处理流量是应用服务流量的数倍
- Master/Worker架构，Worker的数量是CPU核数
- WEB/EMAIL/TCP流量 -> Nginx 非阻塞事件驱动状态机
- 

### 进程结构
- 单进程结构
- 多进程结构而不是多线程结构，为了保证高可用性和高可靠线，多线程会共享地址空间，当一个线程出现问题后会导致真个进程失败
	- 进程间数据通信是通过共享内存
	- 父子进程间管理通过信号
	- Master Process，一般第三方模块不会在加入到Master process，Worker管理，缓存
	- Worker Process，事件驱动，每个worker进程从头到尾使用完整的CPU核
	- Cache  Process Cache Loader/Manager

#### Master进程
- 监控worker进程，子进程终止的时候会向父进程发送CHLD进程
- 管理Worker进程
- 接受信号
	- TERM，INT立刻停止进程     ->     nginx -s stop
	- QUIT优雅的退出进程     ->     nginx -s quit
	- HUP重载配置文件     ->     nginx -s reload
	- USR1重新打开日志文件，做日志切割     ->     nginx -s reopen
	- 热部署使用USR2，WINCH

#### Worker进程
可以发送TERM，INT，QUIT，USR1， WINCH信号，但不会直接发送，一般希望通过给MASTER进程发送来管理Worker进程

#### reload流程
- 向Master进程发送HUP信号
- Master进程检验所有配置语法是否正确（nginx -t）
- Master进程打开新的监听端口(conf可能引入的新端口)
- Master进程用新的conf启动新的Worker子进程
- Master向老的Worker进程发送QUIT信号
- 老的Worker关闭监听句柄，完成当前请求后退出（Worker shutdown timeout）

#### 热升级流程
- 用新的nignx binary文件替换旧的，配置文件的路径需要一致，替换旧的时候需要备份 cp -f
- 向master进程发送USR2信号（手动）
- master进程会修改pid文件名.oldbin
- master进程用新的conf生成新的master文件
- 向老master发送QUIT信号，关闭老master进程（手动）
- 老master进程会关闭但会保存下来，方便回滚->向老master发送HUP信号，新master发送QUIT信号

#### worker进程优雅的关闭流程
- 设置定时器 worker_shutdown_timeout
- 关闭监听句柄
- 关闭空闲连接
- 在循环中等待全部连接关闭
- 在全部连接关闭或timeout，进程退出

### 网络读写事件
- 接收到一个MSS报文就是一个网络事件
- nginx网络读事件：Accept建立连接，read读消息
- nginx网络写事件：write写消息
- nginx有一个事件收集分发消费器(epoll) -> 事件是生产者，对每种事件建立消费者
- 定时器事件，读写事件


###  nginx 事件模型
- epoll_wait, kernel接受TCP报文产生事件，放入事件队列，epoll_wait唤醒worker进程，从事件队列中获取事件进行处理，产生事件放入队列到内核返回网络事件
- 有队列的存在，因而nginx无法容忍第三方模块长时间占用CPU，会使得事件堆积 

### poll, select, epoll,equeue
- 在同样的句柄数(并发数)下，处理事件poll~=select>epoll>equeue
- 随着并发数的增加，poll/select处理时间增加，而epoll和equeu则不受影响
- poll/select会把大并发连接数全部扔给操作系统，因而扫描了大量无用的连接，而epoll只会选择活跃连接存入链表，操作事件存进红黑树（logn）

- poll和select同样存在一个缺点就是，包含大量文件描述符的数组被整体复制于用户态和内核的地址空间之间，而不论这些文件描述符是否就绪，它的开销随着文件描述符数量的增加而线性增大。

### 进程请求
- 传统的web服务一般是一个进程处理一个连接，存在大量的进程切换，依赖OS的进程调度实现并发
- nginx一个进程处理多个连接，用户态完成连接切换，尽量减少OS进程切换


### 同步和异步，阻塞与非阻塞
- 阻塞与非阻塞：操作系统或底层C库提供的方法或系统调用在运行的过程中似的进程进入sleep状态，某些条件不满足，操作系统将进程切换使用CPU
- openresty通过lua用同步代码实现非阻塞的效果
- 一般非阻塞靠的是异步，异步调用涉及到回调函数/事件监听
- js是单线程的，涉及到大量的异步编程，js实现异步编程采用的方法是回调函数和事件监听（事件发布订阅），但是当应用很复杂很庞大时，大量的回调会让调试程序变得举步维艰，成为开发者的噩梦。promise是在es6标准中的一种用于解决异步编程的解决方

### nginx模块
- 核心模块：http/core/event/mail/stream
- ngx_module_t模块有个type变量记录模块的类型
- 连接池其实就是一个数组，每个元素类型是ngx_connection_s，包含client/server的连接，每个连接按index对应读事件和写事件
- 读写事件类型 -> ngx_event_s
- ngx_pool_size: header/requests pool size

### nginx进程间的通信
- 信号： TERM，INT，QUIT，USR1， WINCH
- 内存共享
	- 锁（以前用的信号量，现在用的是自旋锁）
	- slab内存管理
		- 
	- 一般共享内存数据结构包含红黑树，链表
- openresty共享内存
	ngx_http_lua_api是openresty的核心模块，定义了一个sdk -> lua_shared_dict, 这个dict中数据用红黑树存储key:value, 用链表实现LRU淘汰业务数据
## nginx的内存管理
- 红黑树
- 链表
- slab内存管理（分slot块）

## nginx的十一个阶段的顺序处理
### postread阶段
#### realip
#### find_config
找到处理请求的location指令块
#### rewrite
- 重写url ```rewrite```
- 条件判断 ```return code, message```

### preaccess阶段
#### limit_req
- 对请求做限制
#### limit_conn
- 对连接做限制
### access阶段
- 有执行顺序的要求
- ```satisfy all | any```
- satisfy all 要求所有access模块成功才能允许放行
- satisfy any只要有一个access模块成功就会放行
#### access
- 默认编译进nginx
- 对ip做限制
- ```allow/deny address|cidr|unix```
#### auth_basic
- 默认编译进nginx
- 对用户名密码做限制
- http协议本身不加密
- ```auth_basic string|off
	auth_basic_user_file file
   ```
- auth_basic_user_file ->httpd-tools
	-`htpasswd -c file -b user pass`
#### auth_request
- 默认不编译进nginx
- 统一的用户权限验证系统，使用第三方做权限控制的auth_request模块
- 收到请求后，生成子请求，通过反向代理技术把请求传递给上有政策服务
- 如果上游服务返回2xx则继续
- ```auth_request uri | off;``` 这个uri就是上游服务
- ```auth_request_set $variable value;```获取upsrteam的变量

### precontent阶段
#### try_files
- 默认编译进 nginx
- 排序访问资源
- ``` try_files file ... uri/=code;```
- 	```sh
	location /first {
		try_files /system/maintenance.html
				$uri $uri/index.html $uri.html
				@lasturl;
	}
	location @lasturl {
		return 200 "lasturl!\n";
	}
	location /second {
		try_files $uri $uri/index.html $uri.html =404;
	}
	```
- 适用于反向代理情境下
#### mirror
- 默认编译进nginx
- 实时拷贝流量
- 处理请求时，生成子请求访问其他服务
- ``` mirror uri | off;```
- ``` mirror_request_body off | on;```
### content阶段
#### concat
- 阿里提供的模块
- 一次访问中返回多个小文件，提升多个小文件性能
- 格式 URI??file_1,file_2,file_3...?params
- concat/concat_delimiter/concat_types/concat_unique/concat_max_files
#### index
- 指定/访问时返回index文件内容
- 默认含有这个模块
#### autoindex
- 当URL以/结尾时，尝试以html/xml/json/jsonp等格式返回root/alias指向的目录的目录结构
- 在index之后执行，index返回结果后，这个autoindex就不起作用 
``` autoindex autoindex_exact_size autoindex_localtime autoindex_format```
#### static
- 默认是nginx框架里的，不可移除
- ``` alias/root path;```将url映射成文件路径，以返回静态文件内容
- root会将完整含location的url映射到文件路径汇总，alias只会将location后的路径映射到文件路径
- root默认值html，alias没有。root使用范围更广
- ```sh
	location /root {
		root html;
	}
	location /alias {
		alias html;
	}
	location ~/root/(\w+\.txt) {
		root html/fisrt/$1
	}
	location ~/alias/(\w+\.txt) {
		alias html/fisrt/$1;
	}
	```
分别访问：
/root/ -> html/root/index.html
/root/1.txt -> html/first/1.txt/root/1.txt
/alias/ -> html/index.html
/alias/1.txt -> html/first/1.txt
- 三个访问静态资源时的变量
	- **request_filename**待访问文件的完整路径，包括扩展名
	- **document_root**由URI和root/alias规则生成的文件夹(目录)路径
	- **realpath_root**将document_root中的软连接等换成真实路径
- 静态资源的content-type
	- types
	- default_type
	- ...
- 找不到静态文件的情况下的日志 log_not_found on | off
- static模块实现了root/alias功能时，发现访问目标是目录，但url末尾并没有加/时，会返回301重定向
	- absolute_redirect off | on;
	- server_name_in_direct on | off;
	- port_in_redirect on | off;

### log阶段
#### log模块
- 无法禁用，编译进nginx框架中了
- 将http相关请求的信息记录到日志
- ```log_format/access_log path format gzip flush if```
- 日志path是可以包含变量的，所以日志可以根据变量存放在不同的文件中
- 日志缓存 -> 超出buffer大小或者flush到达时间或者worker进程变化才会出发日志写入磁盘
- 日志压缩 
- ```open_log_file_cache max=N inactive min_uses valid```
	- max：缓存内的最大文件句柄数，超出后用LRU算法淘汰
	- inactive：文件访问完后在这段时间不会关闭
	- min_uses：在inactive时间内使用次数超过min_uses才会留在内存里
	- valid：在valid时间内检测内存中日志是否存在

## 过滤模块
- 处在log阶段之前，content阶段之后，改变响应
- preaccess -> access -> content -> header filter -> body filter -> log
- 过滤模块的顺序：image filter -> gzip
- copy_filter：使得sendfile零拷贝失效
- postpone_filter：处理子请求
- header_filter：生成响应头
- write_filter：生成响应
### sub模块
- 对响应中的字符串替换
- 默认未编译进nginx
- ``` sub_filter string replacement;``` sub_filter_once, sub_filter_types, sub_filter_last_modified(header加入last_modified时间);
- 忽略大小写
### addition模块
- 在响应前或者响应后增加内容，并不修改响应本身，方式是通过增加子请求完成
- 默认未编译进nginx
- ```add_before_body/add_after_body/addition_types```

## Nignx中变量的运行
- 变量是很好的解耦工具
### 运行原理
- 提供变量的模块：在请求还没有来之前已经在preconfiguration中定义了新的变量及解析出变量的方法
- 使用变量的模块
- 惰性求值：开始读取变量的时候才会求值
	- good performance
	- 变量值可能时刻变化，使用的时候为当前值
### nginx框架中提供的变量
- 例如：bytes_sent, body_bytes_sent
- HTTP请求相关的变量
	- 例如访问 localhost:9000?a=1&b=22
	- arg_参数名: arg_a=1, arg_b=22
	- query_string: a=1&b=22
	- args: a=1&b=22
	- is_args: 有参数则返回？否则返回空字符串
	- content_length: 请求中标识包体长度Content-Length头部的值
	- content_type: Content-Type头部的值
	- uri：请求的URI，不同于URL，不包括 ？后的参数
	- document_uri: 同uri完全一样
	- request_uri: 请求的URL，包含URI及完整的参数
	- scheme：协议名 http/https
	- request_method: 请求方法 GET或者POST
	- request_length：请求内容的 大小，包括请求行/头部/包体
	- remote_user：由HTTP Basic Authentication协议传入的用户名
	- request_body_file: 临时存放请求包体的文件
	- request_body: 请求中的包体，这个变量当且仅当使用反向代理，且设定用内存暂存包体时才有效
	- request: 原始url，含有方法和协议版本，例如GET /?a=1&b=22 HTTP/1.1
	- host：请求行中获取原始头部 -> Host头部替换请求行 -> 如果两者都取不到，则使用匹配上的server_name
	- http_头部名字: 返回 一个具体请求头部的值
		- 特殊： http_host,http_user_agent,http_referer,http_via,http_x_forwarded_for,http_cookie 
- TCP连接相关的变量
	- binary_remote_addr: 客户端地址的整型格式，IPv4/IPv6
	- remote_addr
	- remote_port
	- connection: 递增的连接序号
	- connection_requests：当前连接上执行过的请求数，对keep-alive有意义
	- proxy_protocol_addr：使用proxy_protocol协议则返回协议中的地址，否则返回空
	- procy_protocol_port
	- server_addr
	- server_port
	- TCP_INFO：tcp内核层参数，
	- server_protocol
- nginx处理请求过程中产生的变量
	- request_time: 单位是s, 精确到ms
	- server_name
	- https: 是否开启了TLS/SSL，则返回on/空
	- request_completion: 请求是否处理完成，返回 OK/空
	- request_id：随机生成的16字节id
	- request_filename: 待访问文件的完整路径
	- document_root：URI和root/alias规则生成的文件夹路径
	- realpath_root：将document_root中的软连接换成真实路径
	- limit_rate
- 发送HTTP响应时相关的变量
	- body_bytes_sent
	- bytes_sent
	- status
	- sent_trailer_名字：把响应结尾内容里值返回
	- sent_http_头部名字：响应中某个具体头部的值
		- 特殊: sent_http_content_type/content_length/location/last_modified/connection/keep_alive/transfer_encoding/cache_control/link
- nginx系统变量：不随请求发生变化
	- time_local
	- time_iso8601
	- nginx_version
	- pid: worker进程的pid
	- pipe：使用了管道则返回p/.
	- hostname: 服务器主机名，与hostname命令输出一致
	- msec：1970年1月1日到现在的时间，单位为s，精确到ms

## Referer模块
- referer防盗链，http请求头部会通过referer头部，将该网站当前页面的url带上，告诉服务器本次请求是由这个页面发起的
- 思路：通过referer模块，用valid_referer变量根据配置判断referer头部是否合法
- 目的：拒绝非正常的网站访问我们站点的资源
- 默认编译进nginx
- 指令
	- valid_referers none | blocked | server_names |  string ...;
		- none允许不带referer
		- block允许referer为空
		- server_names：若referer中站点域名与server_name中本机域名某个匹配，则允许该请求访问
		- 域名及url的字符串，可前缀后缀带*通配符
		- 正则表达式
	- invalid_referer变量：允许访问时变量值为空，不允许时变量值为1
	- referer_hash_bucket_size: 64
	- referer_hash_max_size: 2048

## 防盗链的一种解决方案secure_link
- 默认未编译进nginx
- 原理
	- 服务器生成加密后的安全连接url返回给客户端，客户端使用安全url访问服务端，由服务端secure_link变量判断是否验证通过
	- 哈希算法是不可逆的
	- 客户端只能拿到执行过哈希算法的url
	- 仅生成URL的服务器，验证url是否安全的nginx这两者，才保存执行哈希算法前的原始字符串
	- 原始字符串通常由以下部分有序组成
		- 资源位置：防止攻击者访问任何资源
		- 用户信息
		- 时间戳
		- 秘钥，仅服务器端拥有
- 指令
	- secure_link
	- secure_link_md5
	- secure_link_secret
- 变量
	- secure_link
		- secure_link指令下，值为""(验证不通过)/0(时间戳过期)/1(通过)
		- secure_link_secret指令下，访问格式`/prefix/md5/link`，值为""(验证不通过)/link(验证通过)

## 通过映射新变量提供更多的可能性
### Map模块
- 默认编译进nginx
- 配置在server块之外
- 指令
	- `map string $var {
	hostnames;
	default 0;
	*.map.tech.cn 1;
	map.tech.cn.* 2;
	xxxx 3;
	yyyy 4;
	}`
	- 判断string的值，并将其对应值存储在var变量中 
- 可以通过include优化可读性
- 通过volitale禁止缓存
### split_clients
- 默认编译进nginx中
- 百分比的计算
	- 对于变量的值执行Murmurhash2算法得到32位的整型哈希数值，记为hash
	- 32位整型数最大 数值2^32-1,记为max
	- percent = hash/max
	- 再判断percent落在哪个区域对应哪个数字
	- 百分比可以精确到小数点后两位

### geo模块
- 默认编译进nginx
- 根据客户端ip地址子网掩码，生成新的变量值
- 指令
	- geo [$remote_addr] $var {}
	- 优先最长匹配，匹配较大的子网掩码

### geoip模块
- 根据变量获得用户的地理位置
- 默认没有编译进nginx中
- 利用maxmind数据库
- 指令
	- geoip_country file;
	- geoip_proxy address|CIDR;
	- geoip_city file
- 变量
	- geoip_country_code: CN/US
	- geoip_country_code3: CHN/USA
	- geoip_country_name: China
	- geoip_latitude
	- geoip_city_contitent_code
	- geoip_region
	- ...

## 对客户端使用keepalive提升连接效率
- http协议中的keepalive
- 下游客户端，上游代理服务器
- 多个http请求，复用同一个连接
- 减少握手次数，减少并发连接数从而减少了内存消耗，降低tcp拥塞控制的影响
- 协议
	- Connection头部：close/keepalive
	- Keep-Alive头部：timeout=n
- 指令
	- keepalive-disable
	- keepalive-requests:一个连接最多有多少个请求，默认是100
	- keepalive-timeout:

# 反向代理和负载均衡
- 反向代理
- 负载均衡
- 缓存
## 原理
### 负载均衡
- 提高服务可用性
	- 容灾
	- 扩容
- 保证可扩展性
	- AKF扩展（可以组合使用）
		- X轴扩展/水平扩展
			- 扩容成本低
			- 服务无状体啊
			- 基于round-robin/least-connected算法分发请求
			- 无法解决数据量的问题
		- Y轴扩展
			- 从功能上拆分服务器
			- 通过url分发
			- 利用location path来区分
			- 成本高
		- Z轴扩展
			- 基于用户的信息进行扩展，比如cdn/用户名/用户ip
			- 基于hash算法
			- 分离减少数据的流量
### 反向代理
#### 反向代理
- downstream下游 -> upstream上游
- 四层反向代理，没有很多业务特性
	- udp -> udp
	- tcp -> tcp
- 七层反向代理，含有大量的业务信息
	- http(header/method...) -> memcached/scgi/fastcgi/wsgi/http/websocket
#### 缓存
- 时间缓存
	- 用户请求应用服务器，nginx在请求服务后，发给用户并缓存在nginx上，当用户在访问同样的资源，nginx可以之间返回
- 空间缓存
	- 访问后端服务器的时候，nginx预取一些信息
### 反向代理算法
#### 上游指令
	- 指定上游服务地址的upstream指令和server指令 
		- upstream name {}
		- server address [parameter]
		- 指定 一组上游服务器，地址可以是域名/IP/unix socket，可以加端口，端口默认是80
		- 参数
			- backup: 只有当主server不工作的时候才会发送流量到backup机器上
			- down: server下线，不提供服务
#### round-robin算法
- 加权轮询的方式取去访问上游服务
- 集成在nginx的upstream框架中
- 指令
	- weight
	- max_conns
	- max_fails
	- fail_timeout


## 问题
- X-Forwarded-For和realip地址proxy什么关系？
			













<!--stackedit_data:
eyJoaXN0b3J5IjpbLTkwMjQ5MDcxMCwxNzQwODExMjg3LC0xMj
c3OTEwNjIzLC04NTA3MzM5NjYsMjg4ODgwMzQ2LC04Mzg0MTc4
ODUsMTg4NTU3ODQxOSwxNjcxNDk0MjkzLDExMTY4NDUyMzYsLT
EwNjA4Mzc3MzgsLTE3MTEwMjEzMjMsMjA1MDYzNDgxNiwxNzEy
NTQzNDkyXX0=
-->