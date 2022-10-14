# 基于PHPSocket.io实现前后端通信(React+Laravel)

## 官方地址
### [Github/phpsocket.io](https://github.com/walkor/phpsocket.io)
### [wokerman/phpsocket.io](https://www.workerman.net/phpsocket_io)
### [socket.io](https://socket.io/)

## 安装

### 后端(server)
安装phpsocket.io，命令行中运行 

`composer require workerman/phpsocket.io`
### 前端(client)
安装socket.io-client，命令行中运行  

`npm install socket.io-client`  

注：此处安装的socket.io-client版本可能会与phpsocket.io不匹配，
phpsocket.io只适配socket.io >= v1.3.0 或 <= v2.x，请安装合适版本。

## 前后端通信连接

### 后端(server)

phpsocket.io不能通过前端网页的请求调用，只能在命令行内调用，我们需要新建start.php文件，并在命令行内输入
`php start.php start`，启动通信服务端，后续可加上`-d`参数以DAEMON模式启用。

start.php  
```
<?php
require_once '../../../vendor/autoload.php'  //你的vender/autoload.php的路径;

use Workerman\Worker;
use PHPSocketIO\SocketIO;

$io = new SocketIO(3120);  
// 创建phpsocket.io服务端，并赋予监听端口3120

$io->on('connection', function($socket){  
  echo "new connection coming\n";
});
//定义connect事件：当有用户端连接时，会在终端显示"new connection comming"

Worker::runAll();
```
### 前端(client)

前端使用socket.io-client
```jsx
import io from "socket.io-client";

const socket = io('http://127.0.0.1:3120'); //本地监听3120端口
socket.on('connect_error', (ex) => {  //连接失败，调用connect_error事件输出错误信息
    console.log(ex);
});
socket.on('connect', function (){  //连接成功，调用connect事件，输出'connect success'
    console.log('connect success');
});
```
## 使用举例(实现一对一或一对多通信)

服务端与客户端都调用on方法定义事件，调用emit方法触发事件

### 后端(server)
```php
 $socket->on('login',function ($user_id)use($socket){ 
       $socket->join($user_id);
 });
 //定义login事件，接收user_id作为参数，通过join方式将该用户分组
 
 $socket->on('inform',function ($user_id)use($io){
       $io->to($user_id)->emit('update');
 });
 //定义inform事件，接受user_id作为参数，触发客户端中，以当前user_id作为组名的小组中的update事件
```
### 前端(client)
```jsx
socket.emit('login',user.id); //前端用户成功登录时，触发服务端定义好的login事件，从而完成分组

socket.emit('inform',user_id); //在用户更新信息，或需要告知被通信人更新数据时，触发服务端inform事件

socket.on('update',function (){ //定义用户端事件update，当服务端触发此事件时，前端重新获取数据，更新页面
    getUpdateData();  //获取数据以及更新数据方法请自行定义
});
```

## 线上环境使用
支持SSL(wss或https)
### 后端(server)
SSL 要求workerman>=3.3.7 phpsocket.io>=1.1.1
```php
$context = array(
    'ssl' => array(
    'local_cert'  => '/your/path/of/server.crt',
    'local_pk'    => '/your/path/of/server.key',
    'verify_peer' => false
    )
);
//定义ssl选项，传入服务器证书等路径

$io = new SocketIO(3120,$context);
//定义接口后加上ssl参数
```

### 前端(client)
```jsx
const socket = io('https://yourwebsite.com:3120',{transports: ['websocket']});
//切换域名地址到你的网页，并设置transports模式为'websocket'
```

注：一定要在服务端启动socket.io`php start.php start -d`后，才能在前端看到连接成功的输出