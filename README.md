<p align="center">
<a href="https://packagist.org/packages/code-lives/baidu" target="_blank"><img src="https://img.shields.io/packagist/v/code-lives/baidu?include_prereleases" alt="GitHub forks"></a>
<a href="https://packagist.org/packages/code-lives/baidu" target="_blank"><img src="https://img.shields.io/github/stars/code-lives/baidu?style=social" alt="GitHub forks"></a>
<a href="https://github.com/code-lives/baidu/fork" target="_blank"><img src="https://img.shields.io/github/forks/code-lives/baidu?style=social" alt="GitHub forks"></a>

</p>

|                                     第三方                                      | token | openid | 支付 | 回调 | 退款 | 订单查询 | 解密手机号 | 分账 | 模版消息 |
| :-----------------------------------------------------------------------------: | :---: | :----: | :--: | :--: | :--: | :------: | :--------: | :--: | :------: |
| [百度小程序](https://smartprogram.baidu.com/docs/develop/function/tune_up_2.0/) |   ✓   |   ✓    |  ✓   |  ✓   |  ✓   |    ✓     |     ✓      |  x   |    ✓     |

## 安装

```php
composer require code-lives/baidu 1.0.0
```

### ⚠️ 注意

> 金额单位分 100=1 元

> 返回结果 array 由开发者自行判断

# 预下单

```php
<?php

//引入命名空间
use Applet\Assemble\Baidu;

$pay= Baidu::init($config)->set("订单号","金额","描述","描述")->getParam();

```

# 百度小程序

### Config

| 参数名字        | 类型   | 必须 | 说明                                                            |
| --------------- | ------ | ---- | --------------------------------------------------------------- |
| appkey          | string | 是   | 百度小程序 appkey                                               |
| payappKey       | string | 是   | 百度小程序支付 appkey                                           |
| appSecret       | string | 是   | 百度小程序 aapSecret                                            |
| dealId          | int    | 是   | 百度小程序支付凭证                                              |
| isSkipAudit     | int    | 是   | 默认为 0； 0：不跳过开发者业务方审核；1：跳过开发者业务方审核。 |
| rsaPriKeyStr    | string | 是   | 私钥（只需要一行长串，不需要文件）                              |
| rsaPubKeyStr    | string | 是   | 百度小程序支付的平台公钥(支付回调需要)                          |
| notifyUrl       | string | 否   | 异步回调地址                                                    |
| refundNotifyUrl | string | 否   | 退款异步回调地址                                                |

### Token

```php
$data= Baidu::init($config)->getToken();
```

| 返回参数     | 类型   | 必须 | 说明                   |
| ------------ | ------ | ---- | ---------------------- |
| expires_in   | string | 是   | 凭证有效时间，单位：秒 |
| session_key  | string | 是   | session_key            |
| access_token | string | 是   | 获取到的凭证           |

### Openid

```php
$code="";
$data= Baidu::init($config)->getOpenid($code);
```

| 返回参数    | 类型   | 必须 | 说明        |
| ----------- | ------ | ---- | ----------- |
| session_key | string | 是   | session_key |
| openid      | string | 是   | 用户 openid |

### 解密手机号

```php
$data= Baidu::init($config)->decryptPhone($session_key, $iv, $ciphertext);
echo $phone['mobile'];
```

### 订单查询

| 参数名字     | 类型   | 必须 | 说明                 |
| ------------ | ------ | ---- | -------------------- |
| access_token | string | 是   | 根据上面的获取 token |
| tpOrderId    | string | 是   | 平台订单号           |

```php
$Baidu = Baidu::init($config);
$order = [
        'tpOrderId' => '',//订单号
        'access_token' => $Baidu->getToken()['access_token'],
    ];
$data = $Baidu->findOrder($order);
```

### 退款

| 参数名字         | 类型   | 必须 | 说明                                                                                               |
| ---------------- | ------ | ---- | -------------------------------------------------------------------------------------------------- |
| access_token     | string | 是   | 根据上面的获取 token                                                                               |
| bizRefundBatchId | int    | 是   | 百度平台的订单号                                                                                   |
| isSkipAudit      | int    | 是   | 默认为 0； 0：不跳过开发者业务方审核；1：跳过开发者业务方审核。                                    |
| orderId          | int    | 是   | 百度平台的订单号                                                                                   |
| refundReason     | string | 是   | 退款描述                                                                                           |
| refundType       | int    | 是   | 退款类型 1：用户发起退款；2：开发者业务方客服退款；3：开发者服务异常退款。百度小程序支付的平台公钥 |
| tpOrderId        | string | 是   | 自己平台订单号                                                                                     |
| userId           | int    | 是   | 用户 uid（不是自己平台 uid）                                                                       |

```php
$order = [
'token' => 'abcd',
'bizRefundBatchId' => 123456,//百度平台订单号
'isSkipAudit' => 1,
'orderId' => 123456,
'refundReason' => '测试退款',
'refundType' => 2,//
'tpOrderId' => '123',//自己平台订单号
'userId' => 123,
];
$data= Baidu::init($config)->applyOrderRefund($order);
```

### 模版消息

```php
$data = [
    "touser_openId" => "",
    "template_id" => "",
    "page" => "pages/index/index",
    "subscribe_id" => '百度from组件subscribe-id 一致',
    "data" => json_encode([
        'keyword1' => ['value' => "第一个参数"],
        'keyword2' => ['value' => "第二个参数"],
        'keyword3' =>  ['value' => "第三个参数"],
    ])
];
$data= Baidu::init($config)->sendMsg($data,$token);
$data=[
   "errno" => 0,
    "msg" => "success",
    "data" => array=> [
    "msg_key" => 1663314134696897
  ]
]
```

## 支付回调

```php
$pay = Baidu::init($config);
$status = $pay->notifyCheck();//验证
if($status){
    $order = $pay->getNotifyOrder();
    //$order['tpOrderId']
    //$order['orderId']
    //$order['userId']
    echo 'success';exit;
}
```
