### Deployer-自动部署
Deployer 是一个使用 PHP 开发的轻量级的自动部署工具。

其原理就是通过 SSH 的方式登录服务器，然后执行定义好的 shell 脚本。

安装：
```shell
$ composer global require deployer/deployer
```

通过 dep 命令查看是否安装成功
![](media/15716487404871.jpg)

```初始化
$ mkdir -p ~/Code/deploy-laravel-shop
$ cd ~/Code/deploy-laravel-shop
$ dep init
```
![](media/15716487829565.jpg)

是否允许匿名收集信息
![](media/15716487895564.jpg)

打开 deploy.php 文件
```php
<?php
namespace Deployer;

// 引入官方的 Laravel 部署脚本
require 'recipe/laravel.php';

// 设置一个名为 application 的环境变量，值为 my_project
set('application', 'my_project');

// 设置一个名为 repository，值为初始化脚本时输入的 github 仓库地址
set('repository', 'https://github.com/leo108/laravel-shop-advanced-57.git');

// 设置环境变量，不需要关心
set('git_tty', true);

// shared_files / shared_dirs 这两个环境变量是数组格式，add 函数可以往数组里添加值
add('shared_files', []);
add('shared_dirs', []);

// Deployer 会把这个变量下的目录加上写权限
add('writable_dirs', []);

// 添加一台服务器，服务器 IP/域名 是 project.com
host('project.com')
    // 设置一个这台服务器独享的环境变量，名为 deploy_path，值为 ~/my_project
    // Deployer 会把花括号包裹起来的字符串替换成对应的环境变量
    ->set('deploy_path', '~/{{application}}');

// 定义一个名为 build 的任务
task('build', function () {
    // 这个任务的内容是执行 cd ~/my_project && build 命令
    run('cd {{release_path}} && build');
});

// 定义一个后置钩子，当 deploy:failed 任务被执行之后，Deployer 会执行 deploy:unlock 任务
after('deploy:failed', 'deploy:unlock');

// 定义一个前置钩子，在执行 deploy:symlink 任务之前先执行 artisan:migrate
before('deploy:symlink', 'artisan:migrate');
```

开始部署：
```php
<?php

namespace Deployer;

// 引入官方的 Laravel 部署脚本
require 'recipe/laravel.php';

// 设置一个名为 repository，值为初始化脚本时输入的 github 仓库地址
set('repository', 'https://github.com/jk4905/laravel-shop-advanced.git');

// shared_files / shared_dirs 这两个环境变量是数组格式，add 函数可以往数组里添加值
add('shared_files', []);
add('shared_dirs', []);
// 顺便把 composer 的 vendor 目录也加进来
add('copy_dirs', ['node_modules', 'vendor']);
// Deployer 会把这个变量下的目录加上写权限
set('writable_dirs', []);

// 添加一台服务器，服务器 IP/域名 是 project.com
host('121.40.73.167')
    // 设置一个这台服务器独享的环境变量，名为 deploy_path，值为 ~/my_project
    // Deployer 会把花括号包裹起来的字符串替换成对应的环境变量
    ->user('root') // 使用 root 账号登录
    ->identityFile('~/.ssh/laravel-shop-aliyun.pem') // 指定登录密钥文件路径
    ->become('www-data') // 以 www-data 身份执行命令
    ->set('deploy_path', '/var/www/laravel-shop-deployer');

//第二台
host('47.111.141.124')
    ->user('root')
    ->identityFile('~/.ssh/laravel-shop-aliyun.pem')
    ->become('www-data')
    ->set('deploy_path', '/var/www/laravel-shop'); // 第二台的部署目录与第一台不同

host('121.40.77.53')
    ->user('root')
    ->identityFile('~/.ssh/laravel-shop-aliyun.pem')
    ->become('www-data')
    ->set('deploy_path', '/var/www/laravel-shop'); // 第二台的部署目录与第一台不同


desc('Upload .env file');
task('env:upload', function () {
    // 将本地的 .env 文件上传到代码目录的 .env
    upload('.env', '{{release_path}}/.env');
});


// 定义一个前端编译的任务
desc('Yarn');
task('deploy:yarn', function () {
    // release_path 是 Deployer 的一个内部变量，代表当前代码目录路径
    // run() 的默认超时时间是 5 分钟，而 yarn 相关的操作又比较费时，因此我们在第二个参数传入 timeout = 600，指定这个命令的超时时间是 10 分钟
    run('cd {{release_path}} && SASS_BINARY_SITE=http://npm.taobao.org/mirrors/node-sass yarn && yarn production', ['timeout' => 600]);
});

// 定义一个 执行 es:migrate 命令的任务
desc('Execute elasticsearch migrate');
task('es:migrate', function () {
    // {{bin/php}} 是 Deployer 内置的变量，是 PHP 程序的绝对路径。
    run('{{bin/php}} {{release_path}}/artisan es:migrate');
})->once();

desc('Restart Horizon');
task('horizon:terminate', function() {
    run('{{bin/php}} {{release_path}}/artisan horizon:terminate');
});

// 定义一个后置钩子，在 deploy:shared 之后执行 env:upload 任务
after('deploy:shared', 'env:upload');
// 定义一个后置钩子，在 deploy:vendors 之后执行 deploy:yarn 任务
after('deploy:vendors', 'deploy:yarn');
// 在 deploy:vendors 之前调用 deploy:copy_dirs
before('deploy:vendors', 'deploy:copy_dirs');
// 路由
after('artisan:config:cache', 'artisan:route:cache');
// 定义一个后置钩子，在 artisan:migrate 之后执行 es:migrate 任务
after('artisan:migrate', 'es:migrate');
// 在 deploy:symlink 任务之后执行 horizon:terminate 任务
after('deploy:symlink', 'horizon:terminate');





// 定义一个后置钩子，当 deploy:failed 任务被执行之后，Deployer 会执行 deploy:unlock 任务
after('deploy:failed', 'deploy:unlock');

// 定义一个前置钩子，在执行 deploy:symlink 任务之前先执行 artisan:migrate
before('deploy:symlink', 'artisan:migrate');

```