Celmet = CDN Helmet || CC Helmet

应用Celmet可有效降低cdn遭遇cc攻击时的流量损失，轻量、高性能，低部署成本，1Mb带宽即可撬动百万乃至千万级单日请求。

Celmet可在512M内存VPS上稳定运行，大多数场景下内存开销低于64MB。仅需1个json配置文件，利用docker可快速部署和无痛迁移。

[Celmet官网](https://celmet.d9.ee)

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

[Celmet介绍](./docs/guide.md)

[安装与配置](./docs/config.md)

[注意事项](./docs/warning.md)