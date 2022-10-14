# 自定义微信转发卡片(React)
## 官方地址
### [微信网页开发/JS-SDK](https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/JS-SDK.html)

## 安装SDK
```npm install weixin-js-sdk```

## 前端

### 判断浏览器是否为微信浏览器
微信自带的转发卡片功能，只能在微信浏览器上使用，普通网页无法使用
```jsx
 const isWxBrowser = () => {
    const ua = navigator.userAgent.toLowerCase();
    return ua.match(/MicroMessenger/i) == 'micromessenger'; //true即为微信浏览器，flase即不是微信浏览器
}
```
### 注入权限验证配置
如果判断当前浏览器为微信浏览器，则注入配置
```jsx
 httpclient.getWxConfigParams(window.location.href.split('#')[0]).then((res) => {
    wx.config({
        debug: true, // 开启调试模式,调用的所有 api 的返回值会在客户端 alert 出来，若要查看传入的参数，可以在 pc 端打开，参数信息会通过 log 打出，仅在 pc 端时才会打印。
        appId: res.appId, // 必填，公众号的唯一标识
        timestamp: res.timeStamp, // 必填，生成签名的时间戳
        nonceStr: res.nonceStr, // 必填，生成签名的随机串
        signature: res.signature,// 必填，签名
        jsApiList: ['updateAppMessageShareData'],// 必填，需要使用的 JS 接口列表
    });
``` 
这里需要的appId，timestamp，nonceStr和signature我们需要向后端发起请求获取。例子里我们定义里一个getWxcConfigParams方法，传入当前的url路径作为参数，返回的结果便是我们需要的全部数据

### 使用ready及error验证接口处理
```jsx
wx.ready(function(){
  // config信息验证后会执行 ready 方法，所有接口调用都必须在 config 接口获得结果之后，config是一个客户端的异步操作，所以如果需要在页面加载时就调用相关接口，则须把相关接口放在 ready 函数中调用来确保正确执行。对于用户触发时才调用的接口，则可以直接调用，不需要放在 ready 函数中。
});

wx.error(function(res){
    // config信息验证失败会执行 error 函数，如签名过期导致验证失败，具体错误信息可以打开 config 的debug模式查看，也可以在返回的 res 参数中查看，对于 SPA 可以在这里更新签名。
});

```
### 定义转发卡片
需要在wx.ready中调用
```jsx
wx.ready(function () {//需在用户可能点击分享按钮前就先调用
    wx.updateAppMessageShareData({
        title: data.title, // 分享标题
        desc: data.name, // 分享描述
        link: data.link, // 分享链接，该链接域名或路径必须与当前页面对应的公众号 JS 安全域名一致
        imgUrl: data.imgUrl, // 分享图标
        success: function () {// 设置成功回调函数
        }
    })
});
```

## 后端
后端要提供给前端相应的微信接口参数，首先需要根据微信公众号ID和微信公众号SECRET获取微信的**access_token**，然后根据**access_toekn**获取**jsapi_ticket**，再由**jsapi_ticket**，**nonceStr**，**timeStamp**和**url**合成**encodeStr**，通过sha1签名算法生成最终的**signature**。最后再将需要返回的数据整合，传递给前端。  

注：需要在微信公众号平台中给使用的域名添加白名单
### getWechatAccessToken
```php
<?php

namespace App\Services\WechatAccessToken;

use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\Cache;


class getWechatAccessToken extends Controller
{
    function httpGet($url){  //需要到php-curl扩展
        $ch = curl_init();
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_HEADER, false);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
        $data = curl_exec($ch);
        curl_close($ch);
        return $data;
    }
    function getAccessToken(){  //获取access_token并存储在缓存中
        $cached = Cache::get('wechat_access_token');
        $cache_hit = true;
        if(!$cached){  //没有获取过的token
            $data = new \stdClass;
            $cache_hit = false;
        }else{  //有获取过的token
            $data = json_decode($cached);
            if($data->expire_time < time()){  //token过期(7200s)
                $cache_hit = false;
            }
        }
        if(!$cache_hit){  //重新获取
            $appId = env("WECHAT_APP_ID");  //微信公众号ID
            $appSecret = env("WECHAT_APP_SECRET");  //微信公众号SECRET
            $url = "https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=$appId&secret=$appSecret";
            $res = json_decode($this->httpGet($url));
            $access_token = @$res->access_token;
            if ($access_token) {
                $data->expire_time = time() + 7000;
                $data->access_token = $access_token;
                Cache::forever("wechat_access_token", json_encode($data));
            }
        }else{
            $access_token = $data->access_token;
        }
        return $access_token;
    }
    function getJsApiTicket(){  //获取jsapi_ticket并存储在缓存中
        $cached = Cache::get("jsapi_ticket");
        $cache_hit = true;
        if (!$cached) {
            $data = new \stdClass;
            $cache_hit = false;
        } else {
            $data = json_decode($cached);
            if ($data->expire_time < time())
                $cache_hit = false;
        }
        $ticket = null;
        if (!$cache_hit) {
            $accessToken = $this->getAccessToken();
            if ($accessToken) {
                $url = "https://api.weixin.qq.com/cgi-bin/ticket/getticket?type=jsapi&access_token=$accessToken";
                $res = json_decode($this->httpGet($url));
                $ticket = $res->ticket;
                if ($ticket) {
                    $data->expire_time = time() + 7000;
                    $data->jsapi_ticket = $ticket;
                    Cache::forever("jsapi_ticket", json_encode($data));
                }
            }
        } else {
            $ticket = $data->jsapi_ticket;
        }
        return $ticket;
    }
}
```
### getWxConfigParams
```php
function getWxConfigParams(getWechatAccessToken $getWechatAccessToken,Request $request){  //注入之前定义好的Service
    $nonceStr = Str::random(32);  //随机生成32位字符串
    $timestamp = strval(time());  //当前时间戳
    $ticket = $getWechatAccessToken->getJsApiTicket();  //获取jsapi_ticket    
    $encodeStr = "jsapi_ticket=$ticket&noncestr=$nonceStr&timestamp=$timestamp&url=$request->url";  //生成encodeStr，注意传入当前的url
    $signature = sha1($encodeStr);  //sha1算法生成签名
    $response['appId'] = env("WECHAT_APP_ID"); //微信公众号ID
    $response['timeStamp'] = $timestamp;
    $response['nonceStr'] = $nonceStr;
    $response['signature'] = $signature;
    return $response;
}
```