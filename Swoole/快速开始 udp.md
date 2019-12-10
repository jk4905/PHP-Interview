### 快速开始 UDP
server_udp.php
```php
<?php
//创建Server对象，监听 127.0.0.1:5200端口，类型为SWOOLE_SOCK_UDP
$server = new swoole_server('127.0,0,1', '5200', SWOOLE_PROCESS, SWOOLE_SOCK_UDP);

//监听数据接收事件
$server->on('Packet', function ($server, $data, $clientInfo) {
    $server->sendto($clientInfo['address'], $clientInfo['port'], "Server " . $data);
    var_dump($clientInfo);
});
// 开始监听
$server->start();
```

udp 服务不需要连接，直接监听
使用下面命令进行测试
```shell
netcat -u 127.0.0.1 5200
```

结果：
服务器端：
```shell
vagrant@homestead:~/Code$ php server_udp.php
array(4) {
  ["server_socket"]=>
  int(3)
  ["server_port"]=>
  int(5200)
  ["address"]=>
  string(9) "127.0.0.1"
  ["port"]=>
  int(32951)
}
```
客户端：
```shell
vagrant@homestead:~/Code$ netcat -u 127.0.0.1 5200
11
Server 11
```