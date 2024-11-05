Celmet安装与配置

Celmet = CDN Helmet || CC Helmet

Celmet可在512M内存VPS上稳定运行，大多数场景下内存开销低于64MB。仅需1个json配置文件，利用docker可快速部署和无痛迁移。

快速开始
----

1.编辑配置文件

```text-plain
[
    {
        "port": 8081,
        "targetDomain": "b1.celmet.d9.ee",
        "ssl": true,
        "cacheExpires": 7200,
        "rateMaxURIWithIP": 10,
        "ratePeriodURIWithIP": 120,
        "rateMaxIP": 100,
        "ratePeriodIP": 60,
        "hashKey": "O0k7I9nWTnFG5DmP7DE7IRezOQKm5RdP",
        "hashArg": "h",
        "timeArg": "t",
        "authType": "aliyun-c2",
        "statusPath": "celmet"
    }
]
```

根据需要替换上述“targetDomain”、“hashKey”、“authType”等参数值，并保存至"/etc/celmet/celmet.json"

2.Docker快速部署

docker run -d --name celmet -p 8081:8081 -v /etc/celmet:/etc/celmet d9ee/celmet

生产环境建议增加: --restart always

3.问题排查

docker logs celmet

* * *

参数说明
----

port：监听端口，范围限制：1024～49151整数（celmet允许多组配置，但端口不可重复）

targetDomain：跳转域名，即CDN绑定域名，不允许空字符串

ssl：CDN是否开启SSL，为true时，跳转url前缀为https://，反之则为http://

cacheExpires：缓存过期时间，单位为秒，0或负值表示禁止缓存，但会导致用户频繁访问时浏览器重复请求静态资源，一方面会增加CDN流量消耗，另一方面容易误触发请求次数限制

rateMaxURIWithIP：请求次数限制（针对URI+IP生效），要求为正整数

ratePeriodURIWithIP：上述请求次数限制对应的时间周期（针对URI+IP生效），单位为秒，要求为正整数

rateMaxIP：请求次数限制（针对IP生效），要求为正整数

ratePeriodIP：上述请求次数限制对应的时间周期（针对IP生效），要求为正整数

hashKey：用于计算URL鉴权签名的用户自定义key，允许大小写字母、数字，字符串最小允许长度6，最大允许长度32（此参数应与CDN服务商处设置一致）

hashArg：跳转URL中签名参数名称，建议使用纯小写字母，字符串最小允许长度1，最大允许长度16（此参数应与CDN服务商处设置一致，部分鉴权方式无需此参数，可任意填写但不得为空）

timeArg：跳转URL中时间戳参数名称，建议使用纯小写字母，字符串最小允许长度1，最大允许长度16（此参数应与CDN服务商处设置一致，部分鉴权方式无需此参数，可任意填写但不得为空）

authType：CDN服务商及URL鉴权方式，将在下面的章节中详细介绍

statusPath：Celmet运行状态查看路径，当设置为”celmet“时，状态查看url为 http://ip:port/celmet 或 http(s)://domain/celmet

* * *

CDN服务商及URL鉴权方式支持情况
------------------

以下鉴权方式在适配开发完成后，未能逐一验证，如遇问题，欢迎到[github页面](https://github.com/d9ee/celmet)提交issue

### nginx-a

适用于基于nginx ngx\_http\_secure\_link\_module实现URL鉴权的CDN方案

签名计算方法和[阿里云URL鉴权](https://help.aliyun.com/zh/cdn/user-guide/configure-url-signing)方式C2类似（时间戳为十进制格式，根据[nginx secure link要求](https://nginx.org/en/docs/http/ngx_http_secure_link_module.html#secure_link_md5)md5签名增加base64转码处理，签名有效时长固定10s）

此为Celmet默认的鉴权方式，当配置参数中authType对应的CDN服务商或其鉴权方式暂未支持时，默认使用nginx-a

### nginx-b

适用于基于nginx ngx\_http\_secure\_link\_module实现URL鉴权的CDN方案

签名计算方法和[阿里云URL鉴权](https://help.aliyun.com/zh/cdn/user-guide/configure-url-signing)方式B类似（时间戳改为十进制格式，根据[nginx secure link要求](https://nginx.org/en/docs/http/ngx_http_secure_link_module.html#secure_link_md5)md5签名增加base64转码处理，签名有效时长固定10s）

### aliyun-c1

适用于[阿里云CDN URL鉴权](https://help.aliyun.com/zh/cdn/user-guide/configure-url-signing)方式C的格式1

时间戳为十进制格式

### aliyun-c2

适用于[阿里云CDN URL鉴权](https://help.aliyun.com/zh/cdn/user-guide/configure-url-signing)方式C的格式2

时间戳为十进制格式

### huaweicloud-c1

适用于[华为云CDN URL鉴权](https://support.huaweicloud.com/usermanual-cdn/cdn_01_0039.html)方式C1

暂不支持sha256

### huaweicloud-c2

适用于[华为云CDN URL鉴权](https://support.huaweicloud.com/usermanual-cdn/cdn_01_0039.html)方式C2

暂不支持sha256

### tencent-d

适用于[腾讯云CDN URL鉴权](https://cloud.tencent.cn/document/product/228/76110)方式D

时间戳为十进制格式

### wangsu-c

适用于[网宿科技CDN URL鉴权](https://www.wangsu.com/document/web/27859urlauth)方式C

时间戳为十进制格式，密钥计算参数需固定设置为“$ourkey$uri$time”

### wangsu-d

适用于[网宿科技CDN URL鉴权](https://www.wangsu.com/document/web/27859urlauth)方式D

时间戳为十进制格式，密钥计算参数需固定设置为“$ourkey$uri$time”

### volcengine-d

适用于[火山引擎CDN URL鉴权](https://www.volcengine.com/docs/6454/99849)方式D

时间戳为十进制格式，暂不支持sha256

### baidu-c

适用于[百度云CDN URL鉴权](https://cloud.baidu.com/doc/CDN/s/ujwvyeo0t)方式C

时间戳为十进制格式

### ksyun-a

适用于[金山云CDN URL鉴权](https://docs.ksyun.com/documents/6091)方式A（类型1）

* * *

手动安装docker image
----------------

1.从[github页面](https://github.com/d9ee/celmet/releases)下载Celmet最新版本docker镜像压缩文件（注意cpu架构）

2.将celmet-xxx64.tar上传到linux服务器

3.ssh连入服务器

4.执行“cd xxx”切换到上传目录

5.执行“docker load -i celmet-xxx64.tar”导入镜像

6.执行“docker images”查看image导入是否正常

7.执行“docker run -d --name celmet -p 8081:8081 -v /etc/celmet:/etc/celmet d9ee/celmet:axx64”

* * *

配置示例
----

### Celmet多节点配置示例

Celmet支持在配置文件中添加多组配置并同时工作，配置文件示例如下

```text-plain
[
    {
        "port": 8081,
        "targetDomain": "img.domain1.com",
        "ssl": true,
        "cacheExpires": 3600,
        "rateMaxURIWithIP": 30,
        "ratePeriodURIWithIP": 3600,
        "rateMaxIP": 3000,
        "ratePeriodIP": 86400,
        "hashKey": "ikBuHP94gER3wiFV",
        "hashArg": "h",
        "timeArg": "t",
        "authType": "aliyun-c2",
        "statusPath": "hideme123"
    },
    {
        "port": 8082,
        "targetDomain": "down.domain2.com",
        "ssl": false,
        "cacheExpires": 600,
        "rateMaxURIWithIP": 3,
        "ratePeriodURIWithIP": 1800,
        "rateMaxIP": 30,
        "ratePeriodIP": 86400,
        "hashKey": "EA67OBBMIlN9LhFx",
        "hashArg": "sign",
        "timeArg": "timestamp",
        "authType": "tencent-d",
        "statusPath": "hideme456"
    }
]
```

### Docker升级Celmet最新版本脚本示例

celmet-update.sh

```text-plain
docker stop celmet && \
docker rm celmet && \
docker pull d9ee/celmet && \
docker run -d --restart always --name celmet -p 8081:8081 -p 8082:8082 -v /etc/celmet:/etc/celmet d9ee/celmet
```

### Celmet nginx反向代理配置示例

```text-plain
set $xff "$remote_addr, $http_x_forwarded_for";

location / {
            proxy_pass                          http://127.0.0.1:8081;
            proxy_set_header Connection         $http_connection;
            proxy_http_version                  1.1;
            proxy_cache_bypass                  $http_upgrade;
            proxy_set_header Upgrade            $http_upgrade;
            proxy_set_header Connection         "upgrade";
            proxy_set_header Host               $proxy_host;
            proxy_set_header X-Real-IP          $remote_addr;
            proxy_set_header X-Forwarded-For    $xff;
            proxy_set_header X-Forwarded-Proto  $scheme;
            proxy_set_header X-Forwarded-Host   $host;
            proxy_set_header X-Forwarded-Port   $server_port;
            proxy_ssl_name $host;
            proxy_ssl_server_name on;
}
```

Celmet使用HTTP-Header中X-Forwarded-For的第一个字符串作为访客IP，因此需要强制将客户端真实IP添加到X-Forwarded-For开头，从而避免恶意用户伪造X-Forwarded-For绕过请求频率限制。

若在Celmet前置DCDN，需要根据DCDN服务商返回的客户端真实IP方式定制nginx配置，以阿里云为例

```text-plain
set $xff "$remote_addr, $http_ali_cdn_real_ip";
```

### nginx-a鉴权方式源服务器nginx配置示例

```text-plain
set $key "ikBuHP94gER3wiFV";

location / {
            secure_link $arg_h,$arg_t;
            secure_link_md5 "$key$uri$arg_t";
            if ($secure_link = "") {
                return 401;
            }
            if ($secure_link = "0") {
                return 410;
            }

            expires                             30d;
}
```

### nginx-b鉴权方式源服务器nginx配置示例

```text-plain
set $key "EA67OBBMIlN9LhFx";

location / {
            if ($uri ~* ^/([0-9]+)/([A-Za-z0-9\-\_]+)/(.+)$) {
                set $timestamp $1;
                set $md5hash $2;
                set $suri /$3;
            }
            secure_link $md5hash,$timestamp;
            secure_link_md5 "$key$suri$timestamp";
            if ($secure_link = "") {
                return 401;
            }
            if ($secure_link = "0") {
                return 410;
            }

            expires     30d;
            rewrite ^/(.*) /$suri break;
}
```

* * *

注意事项
----

[Celmet注意事项](https://celmet.d9.ee/docs/warning.html)