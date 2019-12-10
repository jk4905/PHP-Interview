### 安装 Swoole
通过 PECL 安装
```shell
pecl channel-update pecl.php.net
pecl install swoole
``` 

再查找下 php.ini 位置
```shell
php -i |grep php.ini
```

在 php.ini 中添加
```text
extension=swoole.so
```

最后重启 nginx，查看是否成功加载 swoole
```shell
php -m | grep swoole
```