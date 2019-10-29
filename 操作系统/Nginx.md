### Nginx 实现负载均衡的几种方式
#### 轮询 (默认)
请求按时间顺序逐一分配到不同服务器上，如果服务器挂了，自动剔除。
```shell
upstream backserver {
    server 192.168.0.14;
    server 192.168.0.15;
}
```

#### 加权轮询
指定轮询几率
```shell
upstream backserver {
    server 192.168.0.14 weight=3;
    server 192.168.0.15 weight=7;
}
```


#### ip_hash
上述方式存在一些问题，如用户第一次登录了一个服务器，第二次被定为到另一个服务器，那么登录信息将丢失。（虽然 session 可以存到 redis 中解决）
那么 ip_hash 可以很好解决这个问题，ip_hash 是将用户的 ip 地址通过 hash 分配给服务器。这样同一个用户就会指定到同一个服务器上进行访问。
```shell
upstream backserver {
    ip_hash;
    server 192.168.0.14;
    server 192.168.0.15;
}
``` 


#### 参考
[【Nginx】实现负载均衡的几种方式](https://blog.csdn.net/qq_28602957/article/details/61615876)
[Nginx负载均衡配置](https://blog.csdn.net/xyang81/article/details/51702900)
[Nginx 的反向代理](https://blog.csdn.net/qq_28602957/article/details/53231360)