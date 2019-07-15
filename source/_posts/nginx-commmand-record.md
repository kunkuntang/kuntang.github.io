---
layout: '[post]'
title: Nginx conf 常用配置
date: 2019-07-15 23:52:15
tags: 转载
---

因为不熟悉Nginx，对他的匹配规则似懂非懂，所以平时在部署自己项目的时候经常会遇到各种问题，因此在此记录一下一路踩过的坑。

### nginx.conf 与 conf.d 目录

首先Nginx里有一份基础配置nginx.conf文件，里面通常是nginx的一些默认配置信息，注意在默认配置前有一行引入自定义文件的代码
```
# Load modular configuration files from the /etc/nginx/conf.d directory.
# See http://nginx.org/en/docs/ngx_core_module.html#include
# for more information.
include /etc/nginx/conf.d/*.conf;
```
最后一行把 `conf` 目录下的所有以.conf后缀结尾的文件都引入进来，而因为Nginx配置文件的规则是前面先匹配到的先生效，所以我们可以把自己写的自定义配置文件在 `conf` 目录下，然后覆盖后面的默认配置。

### server 块

关于server块目前比较明白的有以下几个字段：

- `listen` 代表着监听的端口
  > 网页通常是监听80、443端口，还有一些服务应用应该监听对应的端口，如21、25等。
- `server_name` 代表监听的域名
  > 通常是在域名服务商通过设置相应的域名然后解析到服务器的ip，其设置的域名便是 `server_name` 的值
- `root` 匹配后指向的访问路径（拼接匹配的url部分）
  > 要注意当前nginx使用的用户是否有该对指向的路径有相应的访问权限。

### location 块

`location` 处于 `server` 块下，一个 `server` 块可以有若干个 `location` ，先匹配到的 `location` 规则先处理，并停止往后匹配。

```
server {
  ...
  location [pattern rule] {
    ...
  }
}
```

`location` 后面接对应的 url 匹配规则，可以是要匹配的字符串（字符串后面可带`/`也可以不带），也可以是正则。

通过 `location` 实现请求转发来处理前端接口跨域问题，还可以通过它来实现外部对内部服务的代理访问。一般我会用到以下几个字段：

- `root` 匹配后指向的访问路径（拼接匹配的url部分）
- `alias` 匹配后指向的访问路径（替换匹配的url部分）
- `proxy_pass` 把请求转发到指向地址
- `index` 匹配后指向的访问路径下的文件

#### root 与 alias

当 `location` 是用字符串来匹配时，配置 `root` 与 `alias` 是区别的：当 `location` 匹配到相应的 `url` `后，root` 对在匹配的规则后接上 `root` 指向的路径，而 `alisa` 会直接替换掉匹配的url部分。

举个例子：

```
location ~ ^/xingcard/ {
  root /data/www/;
}
```

当一个URI是 `/app/xingcard/index.html` 时，nginx将会返回服务器上 `/data/www/app/xingcard/index.html`的文件。root会根据完整的URI请求来映射，也就是/path/uri。

如果我们把 `root` 换成 `alias` 的话：

```
location ~ ^/app/ {
  alais /data/www/;
}
```

则nginx会将`/data/www/xingcard/index.html`返回给客户端。区别就是在于有没有替换掉 `location` 匹配的 `app`。

> 注意：
> 1. 使用alias时，目录名后面一定要加"/"。
> 2. alias可以指定任何名称。
> 3. alias在使用正则匹配时，必须捕捉要匹配的内容并在指定的内容处使用。
> 4. alias只能位于location块中。

#### proxy_pass 字段

`ngixn` 对于 `proxy_pass` 的处理分为两种，一种是只有IP和端口号，另一种是除了IP和端口号外还包含了其它路径（URI）（其中也包括单个`/`符）。

对于不含URI的 `proxy_pass` ， nginx 将会保留location中的路径部分，即在 `proxy_pass` 的值后面拼接上 `loaction` 的匹配路径。

对于含URI的 `proxy_pass` ，nginx将使用诸如alias的替换方式对URL进行替换。

例如：

```
server {
  ...
  location /api1/ {
    proxy_pass http://localhost:8080;
  }

  location /api2/ {
    proxy_pass http://localhost:8080/;
  }
}
```

访问 `ip/api1/login` 地址，转发到服务器的地址应该是 `http://localhost:8080/api1/login`
访问 `ip/api2/login` 地址，转发到服务器的地址应该是 `http://localhost:8080/login`

#### index 字段

> 转载自：[Nginx之坑：完全理解location中的index，配置网站初始页](https://blog.csdn.net/qq_32331073/article/details/81945134)

- 该指令后面可以跟多个文件，用空格隔开；
- 如果包括多个文件，Nginx会根据文件的枚举顺序来检查，直到查找的文件存在；
- 文件可以是相对路径也可以是绝对路径，绝对路径需要放在最后；
- 文件可以使用变量$来命名；

```
server {
  ...
  location /app {
    ...
    index  index.$geo.html  index.0.html  /index.html;
  }
}
```

该指令拥有默认值，index index.html ，即，如果没有给出index，默认初始页为index.html

Nginx给了三种方式来选择初始页，三种方式按照顺序来执行：

- [ngx_http_random_index_module](http://nginx.org/en/docs/http/ngx_http_random_index_module.html) 模块，从给定的目录中随机选择一个文件作为初始页，而且这个动作发生在 [ngx_http_index_module](http://nginx.org/en/docs/http/ngx_http_index_module.html) 之前，注意：这个模块默认情况下没有被安装，需要在安装时提供配置参数 -with-http_random_index_module；
- [ngx_http_index_module](http://nginx.org/en/docs/http/ngx_http_index_module.html) 模块，根据index指令规则来选择初始页；
- [ngx_http_autoindex_module](http://nginx.org/en/docs/http/ngx_http_autoindex_module.html) 模块，可以使用指定方式，根据给定目录中的文件列表自动生成初始页，这个动作发生在 
[ngx_http_index_module](http://nginx.org/en/docs/http/ngx_http_index_module.html) 之后，即只有通过index指令无法确认初始页，此时启用后的自动生成模块才会被使用。

**如果文件存在，则使用文件作为路径，发起内部重定向。直观上看上去就像再一次从客户端发起请求，Nginx再一次搜索location一样。** 既然是内部重定向，域名+端口不发生变化，所以只会在同一个server下搜索。同样，如果内部重定向发生在proxy_pass反向代理后，那么重定向只会发生在代理配置中的同一个server。

```
server {
    listen      80;
    server_name example.org www.example.org;    
    
    location / {
        root    /data/www;
        index   index.html index.php;
    }
    
    location ~ \.php$ {
        root    /data/www/test;
    }
}
```

上面的例子中，如果你使用example.org或www.example.org直接发起请求，那么首先会访问到“/”的location，结合root与index指令，会先判断/data/www/index.html是否存在，如果不，则接着查看
/data/www/index.php ，如果存在，则使用/index.php发起内部重定向，就像从客户端再一次发起请求一样，Nginx会再一次搜索location，毫无疑问匹配到第二个~ \.php$，从而访问到/data/www/test/index.php。