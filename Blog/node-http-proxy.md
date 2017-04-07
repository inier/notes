## HTTP代理模块（HTTP Proxy）
> 这个模块可以转发请求到其他的服务器。

HTTP/1.0无法使用keepalive（后端服务器将为每个请求创建并且删除连接）。nginx为浏览器发送HTTP/1.1并为后端服务器发送HTTP/1.0，这样浏览器就可以为浏览器处理keepalive。
如下例：
```
location / { proxy_pass http://localhost:8000; proxy_set_header X-Real-IP $remote_addr; }
```
注意当使用http proxy模块（甚至FastCGI），所有的连接请求在发送到后端服务器之前nginx将缓存它们。因此，在测量从后端传送的数据时，它的进度显示可能不正确。

### 指令 proxy_buffer_size
+ 语法：proxy_buffer_size the_size
+ 默认值：proxy_buffer_size 4k/8k
+ 使用字段：http, server, location
##### 设置从被代理服务器读取的第一部分应答的缓冲区大小。通常情况下这部分应答中包含一个小的应答头。 默认情况下这个值的大小为指令proxy_buffers中指定的一个缓冲区的大小，不过可以将其设置为更小。

### proxy_buffering
+ 语法：proxy_buffering on|off
+ 默认值：proxy_buffering on
+ 使用字段：http, server, location
##### 为后端的服务器启用应答缓冲。如果启用缓冲，nginx假设被代理服务器能够非常快的传递应答，并将其放入缓冲区，可以使用 proxy_buffer_size和proxy_buffers设置相关参数。 如果响应无法全部放入内存，则将其写入硬盘。如果禁用缓冲，从后端传来的应答将立即被传送到客户端。
> nginx忽略被代理服务器的应答数目和所有应答的大小，接受proxy_buffer_size所指定的值。 对于基于长轮询的Comet应用需要关闭这个指令，否则异步的应答将被缓冲并且Comet无法正常工作。

### proxy_buffers
+ 语法：proxy_buffers the_number is_size;
+ 默认值：proxy_buffers 8 4k/8k;
+ 使用字段：http, server, location
##### 设置用于读取应答（来自被代理服务器）的缓冲区数目和大小，默认情况也为分页大小，根据操作系统的不同可能是4k或者8k。

### proxy_busy_buffers_size
+ 语法：proxy_busy_buffers_size size;
+ 默认值：proxy_busy_buffers_size ["#proxy buffer size"] * 2;
+ 使用字段：http, server, location, if
##### 未知。

### proxy_cache
+ 语法：proxy_cache zone_name;
+ 默认值：None
+ 使用字段：http, server, location
##### 设置一个缓存区域的名称，一个相同的区域可以在不同的地方使用。在0.7.48后，缓存遵循后端的Cache-Control, Expires以及其他等。缓存依赖代理的缓冲区，如果proxy_buffers设置为off，将不会生效。

### proxy_cache_path
+ 语法：proxy_cache_path path [levels=number] keys_zone=zone_name:zone_size [inactive=time] [max_size=size];
+ 默认值：None
+ 使用字段：
    - http指令指定缓存的路径和一些其他参数，缓存的数据存储在文件中。缓存的文件名和key为代理URL的MD5 码。
    - levels参数指定缓存的子目录数，例如：proxy_cache_path /data/nginx/cache levels=1:2 keys_zone=one:10m;文件名类似于： /data/nginx/cache/c/29/b7f54b2df7773722d382f4809d65029c 所有活动的key和元数据存储在共享的内存区域中，这个区域用keys_zone参数指定，如果在inactive参数指定的时间内缓存的数据没有被请求则被删除，默认inactive为10分钟。 cache manager进程控制磁盘的缓存大小，在max_size参数中定义，超过其大小后最少使用数据将被删除。 区域的大小按照缓存页面数的比例进行设置，一个页面（文件）的元数据大小按照操作系统来定，FreeBSD/i386下为64字节，FreeBSD/amd64下为128字节，当区域满了以后key将按照LRU（最近最少使用算法）进行处理。
> proxy_cache_path和proxy_temp_path应该使用在相同的文件系统上。

### proxy_cache_methods
+ 语法：proxy_cache_methods [GET HEAD POST];
+ 默认值：proxy_cache_methods GET HEAD;
+ 使用字段：http, server, location
##### GET/HEAD用来装饰语句，即你无法禁用GET/HEAD即使你只使用下列语句设置： proxy_cache_methods POST;

### proxy_cache_min_uses
+ 语法：proxy_cache_min_uses the_number;
+ 默认值：proxy_cache_min_uses 1;
+ 使用字段：http, server, location 多少次的查询后应答将被缓存，默认1。

### proxy_cache_valid
+ 语法：proxy_cache_valid reply_code [reply_code ...] time;
+ 默认值：None
+ 使用字段：http, server, location 为不同的应答设置不同的缓存时间，例如： proxy_cache_valid 200 302 10m; proxy_cache_valid 404 1m;为应答代码为200和302的设置缓存时间为10分钟，404代码缓存1分钟。 如果只定义时间： proxy_cache_valid 5m;那么只对代码为200, 301和302的应答进行缓存。 同样可以使用any参数任何应答。 proxy_cache_valid 200 302 10m; proxy_cache_valid 301 1h; proxy_cache_valid any 1m;

### proxy_cache_use_stale
+ 语法：proxy_cache_use_stale [error|timeout|updating|invalid_header|http_500|http_502|http_503|http_504|http_404|off] [...];
+ 默认值：proxy_cache_use_stale off;
+ 使用字段：http, server, location 未知。
> 这个指令的参数类似于proxy_next_upstream指令。

### proxy_connect_timeout
+ 语法：proxy_connect_timeout timeout_in_seconds
+ 使用字段：http, server, location 指定一个连接到代理服务器的超时时间，这个时间并不是指服务器传回页面的时间，而是proxy_read_timeout的声明。无论何时你的代理服务器都是正常运行的，但是如果服务器遇到一些状况（例如没有足够的线程去处理请求，请求将被放在一个连接池中延迟处理），那么这个声明无助于服务器去建立连接。

### proxy_headers_hash_bucket_size
+ 语法：proxy_headers_hash_bucket_size size;
+ 默认值：proxy_headers_hash_bucket_size 64;
+ 使用字段：http, server, location, if 设置哈希表中存储的每个数据大小（参考解释）。

### proxy_headers_hash_max_size
+ 语法：proxy_headers_hash_max_size size;
+ 默认值：proxy_headers_hash_max_size 512;
+ 使用字段：http, server, location, if 设置哈希表的最大值（参考解释）。

### proxy_hide_header
+ 语法：proxy_hide_header the_header
+ 使用字段：http, server, location
##### nginx不对从被代理服务器传来的"Date", "Server", "X-Pad"和"X-Accel-..."应答进行转发，这个参数允许隐藏一些其他的头部字段，但是如果上述提到的头部字段必须被转发，可以使用proxy_pass_header指令，例如：需要隐藏MS-OfficeWebserver和AspNet-Version可以使用如下配置： location / { proxy_hide_header X-AspNet-Version; proxy_hide_header MicrosoftOfficeWebServer; }当使用X-Accel-Redirect时这个指令非常有用。例如，你可能要在后端应用服务器对一个需要下载的文件设置一个返回头，其中X-Accel-Redirect字段即为这个文件，同时要有恰当的Content-Type，但是，重定向的URL将指向包含这个文件的文件服务器，而这个服务器传递了它自己的Content-Type，可能这并不是正确的，这样就忽略了后端应用服务器传递的Content-Type。为了避免这种情况你可以使用这个指令： location / { proxy_pass http://backend_servers; } location /files/ { proxy_pass http://fileserver; proxy_hide_header Content-Type;

### proxy_ignore_client_abort
+ 语法：proxy_ignore_client_abort [ on|off ]
+ 默认值：proxy_ignore_client_abort off
+ 使用字段：http, server, location 防止在客户端自己终端请求的情况下中断代理请求。

### proxy_ignore_headers
+ 语法：proxy_ignore_headers name [name ...] 默认值：none 使用字段：http, server, location 这个指令(0.7.54+) 禁止处理来自代理服务器的应答。 可以指定的字段为"X-Accel-Redirect", "X-Accel-Expires", "Expires"或"Cache-Control"。

### roxy_intercept_errors
+ 语法：proxy_intercept_errors [ on|off ]
+ 默认值：proxy_intercept_errors off
+ 使用字段：http, server, location 使nginx阻止HTTP应答代码为400或者更高的应答。 默认情况下被代理服务器的所有应答都将被传递。 如果将其设置为on则nginx会将阻止的这部分代码在一个error_page指令处理，如果在这个error_page中没有匹配的处理方法，则被代理服务器传递的错误应答会按原样传递。

### proxy_max_temp_file_size
+ 语法：proxy_max_temp_file_size size;
+ 默认值：proxy_max_temp_file_size 1G;
+ 使用字段：http, server, location, if 当代理缓冲区过大时使用一个临时文件的最大值，如果文件大于这个值，将同步传递请求而不写入磁盘进行缓存。 如果这个值设置为零，则禁止使用临时文件。

### proxy_method
+ 语法：proxy_method [method]
+ 默认值：None
+ 使用字段：http, server, location 允许代理一些其他的HTTP访问方法。 如果指定这个指令，nginx只允许一个请求中含有单个HTTP访问方法的请求（在该指令中指定），所以对于代理Subversion这样的请求并不是很有用。 示例配置： proxy_method PROPFIND;

### proxy_next_upstream 语法： proxy_next_upstream [error|timeout|invalid_header|http_500|http_502|http_503|http_504|http_404|off] 默认值：proxy_next_upstream error timeout 使用字段：http, server, location 确定在何种情况下请求将转发到下一个服务器：
·error - 在连接到一个服务器，发送一个请求，或者读取应答时发生错误。
·timeout - 在连接到服务器，转发请求或者读取应答时发生超时。
·invalid_header - 服务器返回空的或者错误的应答。
·http_500 - 服务器返回500代码。
·http_502 - 服务器返回502代码。
·http_503 - 服务器返回503代码。
·http_504 - 服务器返回504代码。
·http_404 - 服务器返回404代码。
·off - 禁止转发请求到下一台服务器。转发请求只发生在没有数据传递到客户端的过程中。

### proxy_pass
+ 语法：proxy_pass URL
+ 默认值：no
+ 使用字段：location, location中的if字段
确定需要代理的URL，端口和socket。可以使用主机名和端口，例如： proxy_pass http://localhost:8000/uri;也可以使用unix socket： proxy_pass unix:/path/to/backend.socket请求转发到服务器的URI部分（location字段的后面部分），将为proxy_pass指定的值的URI。例如你的location字段为/name/，proxy_pass为http://127.0.0.1，则请求/name/test将被转发至http://127.0.0.1/test。

 但是对于上述规则有两个例外，那时将无法确定被替换的location： location使用正则表达式。
 在使用代理的location中利用rewrite指令改变URI，使用这个配置可以更加精确的处理请求（break）:
 ```
 location /name/ { rewrite /name/([^/] +) /users?name=ũ break; proxy_pass http://127.0.0.1; }
 ```
 这些情况下URI并没有被映射传递。 此外，可能需要指明URI将使用同样的方式转发，因为它是来自客户端，而不是以处理过的形式发送。
 在其工作过程中：
   - ·两个以上的斜杠将被替换为一个："//" -- "/";
   - ·删除引用的当前目录："/./" -- "/";
   - ·删除引用的先前目录："/dir /../" -- "/"。

   如果在服务器上必须以未经过处理的形式发送URI，那么在这个指令中必须使用未指定URI的URL：
```
    location /some/path/ { proxy_pass http://127.0.0.1; }
```
在指令中使用变量是一种比较特殊的情况：被请求的URL不会使用并且你必须完全手工标记URL。 这意味着下列的配置并不能让你方便的进入某个你想要的虚拟主机目录，代理总是将它转发到相同的URL（在一个server字段的配置）：
```
   location / { proxy_pass http://127.0.0.1:8080/VirtualHostBase/https/$server_name:443/some/path/VirtualHostRoot; }
```
解决方法是使用rewrite和proxy_pass的组合：
```
 location / { rewrite ^(.*)$ /VirtualHostBase/https/$server_name:443/some/path/VirtualHostRoot/ũ break; proxy_pass http://127.0.0.1:8080; }
```
 这种情况下请求的URL将被重写， proxy_pass中的拖尾斜杠并没有实际意义。

### proxy_pass_header
+ 语法：proxy_pass_header the_name
+ 使用字段：http, server, location 这个指令允许转发一些被禁止的应答头。 如： location / { proxy_pass_header Server; proxy_pass_header X-MyHeader; }

### proxy_pass_request_body
+ 语法：proxy_pass_request_body [ on | off ] ;
+ 默认值：proxy_pass_request_body on;
+ 使用字段：http, server, location
##### 可用版本：0.1.29

### proxy_pass_request_headers
+ 语法：proxy_pass_request_headers [ on | off ] ;
+ 默认值：proxy_pass_request_headers on;
+ 使用字段：http, server, location
##### 可用版本：0.1.29

### proxy_redirect
+ 语法：proxy_redirect [ default|off|redirect replacement ]
+ 默认值：proxy_redirect default
+ 使用字段：http, server, location
##### 如果需要修改从被代理服务器传来的应答头中的"Location"和"Refresh"字段，可以用这个指令设置。
- 假设被代理服务器返回Location字段为： http://localhost:8000/two/some/uri/ 这个指令：
proxy_redirect http://localhost:8000/two/ http://frontend/one/;
将Location字段重写为http://frontend/one/some/uri/。

- 在代替的字段中可以不写服务器名：

```
proxy_redirect http://localhost:8000/two/ /;
```
这样就使用服务器的基本名称和端口，即使它来自非80端口。
 如果使用“default”参数，将根据location和proxy_pass参数的设置来决定。例如下列两个配置等效：
 + location /one/ { proxy_pass http://upstream:port/two/; proxy_redirect default; }
 + location /one/ { proxy_pass http://upstream:port/two/; proxy_redirect http://upstream:port/two/ /one/; }
 在指令中可以使用一些变量：
 + proxy_redirect http://localhost:8000/ http://$host:$server_port/;

 这个指令有时可以重复：
 + proxy_redirect default;
 + proxy_redirect http://localhost:8000/ /;
 + proxy_redirect http://www.example.com/ /;

 参数off将在这个字段中禁止所有的proxy_redirect指令：
 + proxy_redirect off;
 + proxy_redirect default;
 + proxy_redirect http://localhost:8000/ /;
 + proxy_redirect http://www.example.com/ /;
 利用这个指令可以为被代理服务器发出的相对重定向增加主机名：
 + proxy_redirect / /;

### proxy_read_timeout
+ 语法：proxy_read_timeout the_time
+ 默认值：proxy_read_timeout 60
+ 使用字段：http, server, location
##### 决定读取后端服务器应答的超市时间，它决定nginx将等待多久时间来取得一个请求的应答。超时时间是指完成了两次握手后并且状态为established的超时时间，而不是所有的应答时间。
> 相对于proxy_connect_timeout，这个时间可以扑捉到一台将你的连接放入连接池延迟处理并且没有数据传送的服务器，注意不要将此值设置太低，某些情况下代理服务器将花很长的时间来获得页面应答（如当接收一个需要很多计算的报表时），当然你可以设置多个不同的location。 如果被代理服务器在设置的时间内没有传递数据，nginx将关闭连接。 proxy_redirect_errors 不推荐使用，请使用 proxy_intercept_errors。

### proxy_send_lowat
+ 语法：proxy_send_lowat [ on | off ]
+ 默认值：proxy_send_lowat off;
+ 使用字段：http, server, location, if 设置SO_SNDLOWAT，这个指令仅用于FreeBSD。

### proxy_send_timeout
+ 语法：proxy_send_timeout time_in_seconds
+ 默认值：proxy_send_timeout 60
+ 使用字段：http, server, location
##### 设置代理服务器转发请求的超时时间，同样指完成两次握手后的时间，如果超过这个时间代理服务器没有数据转发到被代理服务器，nginx将关闭连接。
### proxy_set_body
+ 语法：proxy_set_body [ on | off ]
+ 默认值：proxy_set_body off;
+ 使用字段：http, server, location, if
##### 可用版本：0.3.10。

### proxy_set_header
+ 语法：proxy_set_header header value
+ 默认值： Host and Connection
+ 使用字段：http, server, location
##### 这个指令允许将发送到被代理服务器的请求头重新定义或者增加一些字段。这个值可以是一个文本，变量或者它们的组合。
proxy_set_header在指定的字段中没有定义时会从它的上级字段继承。

默认只有两个字段可以重新定义：
- proxy_set_header Host $proxy_host;
- proxy_set_header Connection Close;

+ 未修改的请求头“Host”可以用如下方式传送： proxy_set_header Host $http_host;
+ 但是如果这个字段在客户端的请求头中不存在，那么将没有数据转发被代理服务器。 这种情况下最好使用$Host变量，它的值等于请求头中的"Host"字段或服务器名： proxy_set_header Host $host;
+ 此外，可以将被代理的端口与服务器名称一起传递： proxy_set_header Host $host:$proxy_port;
+ 如果设置为空字符串，则不会传递头部到后端，例如下列设置将禁止后端使用gzip压缩： proxy_set_header Accept-Encoding "";

### proxy_store
+ 语法：proxy_store [on | off | path]
+ 默认值：proxy_store off
+ 使用字段：http, server, location
##### 这个指令设置哪些传来的文件将被存储，
- 参数"on"保持文件与alias或root指令指定的目录一致，
- 参数"off"将关闭存储，路径名中可以使用变量：
```
proxy_store /data/www$original_uri;
```
应答头中的"Last-Modified"字段设置了文件最后修改时间，为了文件的安全，可以使用proxy_temp_path指定一个临时文件目录。这个指令为那些不是经常使用的文件做一份本地拷贝。从而减少被代理服务器负载。
```
 location /images/{ root /data/www; error_page 404 = /fetch$uri; }
 location /fetch { internal; proxy_pass http://backend; proxy_store on; proxy_store_access user:rw group:rw all:r; proxy_temp_path /data/temp; alias /data/www; }
```
 或者通过这种方式：
```
  location /images/ { root /data/www; error_page 404 = @fetch; } location @fetch { internal; proxy_pass http://backend; proxy_store on; proxy_store_access user:rw group:rw all:r; proxy_temp_path /data/temp; root /data/www; }
```
> 注意proxy_store不是一个缓存，它更像是一个镜像。

### proxy_store_access
+ 语法：proxy_store_access users:permissions [users:permission ...]
+ 默认值：proxy_store_access user:rw
+ 使用字段：http, server, location 指定创建文件和目录的相关权限，如： proxy_store_access user:rw group:rw all:r;如果正确指定了组和所有的权限，则没有必要去指定用户的权限： proxy_store_access group:rw all:r;

### proxy_temp_file_write_size
+ 语法：proxy_temp_file_write_size size;
+ 默认值：proxy_temp_file_write_size ["#proxy buffer size"] * 2;
+ 使用字段：http, server, location, if 设置在写入proxy_temp_path时数据的大小，在预防一个工作进程在传递文件时阻塞太长。

### proxy_temp_path
+ 语法：proxy_temp_path dir-path [ level1 [ level2 [ level3 ] ;
+ 默认值：在configure时由--http-proxy-temp-path指定
+ 使用字段：http, server, location
类似于http核心模块中的client_body_temp_path指令，指定一个地址来缓冲比较大的被代理请求。

 proxy_upstream_fail_timeout 0.5.0版本后不推荐使用，请使用http负载均衡模块中server指令的fail_timeout参数。 proxy_upstream_fail_timeout 0.5.0版本后不推荐使用，请使用http负载均衡模块中server指令的max_fails参数。

·变量 该模块中包含一些内置变量，可以用于proxy_set_header指令中以创建头部。

$proxy_host 被代理服务器的主机名与端口号。
$proxy_host 被代理服务器的端口号。
$proxy_add_x_forwarded_for 包含客户端请求头中的"X-Forwarded-For"，与$remote_addr用逗号分开，如果没有"X-Forwarded-For"请求头，则$proxy_add_x_forwarded_for等于$remote_addr。