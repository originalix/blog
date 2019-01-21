---
title: 在Yii2框架中使用Redis
date: 2018-03-28 20:56:16
tags: ["PHP", "Redis", "Yii2"]

---

想要在Yii2这个PHP框架中很好的使用redis键值存储，那么首先就要推荐[yii2-redis](https://github.com/yiisoft/yii2-redis)这个官方的[Github](https://github.com/originalix)库。这个库能够很好的帮助我们在Yii2框架中使用redis，它提供缓存，Session以及ActiveRecord模式的支持。

## 安装yii2-redis库

推荐使用composer安装yii2-redis库，在你的项目根目录执行

```php
php composer.phar require --prefer-dist yiisoft/yii2-redis:"~2.0.0"
```

或者将

```json
"yiisoft/yii2-redis": "~2.0.0"
```

加入你的`composer.json`文件里，之后运行`composer update`，墙内真的很慢，耐心等待即可。

<!--more-->

## 配置redis

要正确的使用这个扩展，你必须在你的应用程序的配置文件内，配置`Connection`类，一般来说，配置文件是`config\web.php`。

在你的组件里加入`redis`项目，如下:

```php
return [
    //....
    'components' => [
        'redis' => [
            'class' => 'yii\redis\Connection',
            'hostname' => 'localhost',
            'port' => 6379,
            'database' => 0,
        ],
    ]
];
```

如此之后，你便能正常的在yii2框架中使用redis。

## 示例

### 简单使用

我们先来看一段最简单的使用redis的代码:

```php
$redis = Yii::$app->redis;
$key = 'username';
if ($val = $redis->get($key)) {
    return ['redis' => $val];
} else {
    $redis->set($key, 'Leon');
    $redis->expire($key, 5);
}

return ['redis' => 'no data'];
```

没有一行注释，但是就是一目了然是不是。

寻找`username`这个key，如果找不到，设置键值存储，并且过期时间是5秒钟。

这就是一个完整的使用redis的例子。

### Cache

那么接下来，我们来看看怎么样将redis用在缓存上。

同样的，作为缓存使用，我们需要去配置文件里修改缓存项:

```php
'components' => [
    'cache' => [
        // 'class' => 'yii\caching\FileCache',
        'class' => 'yii\redis\Cache',
    ],
],
```

如果你没有配置过redis组件，那么还需要在cache下配置redis:

```php
'components' => [
    'cache' => [
        // 'class' => 'yii\caching\FileCache',
        'class' => 'yii\redis\Cache',
        'redis' => [
            'hostname' => 'localhost',
            'port' => 6379,
            'database' => 0,
        ],
    ],
],
```

示例代码如下，通俗易懂也就不过多解释了:

```php
$cache = Yii::$app->cache;
$key = 'username';

if ($cache->exists($key)) {
    return ['cache' => $cache->get($key)];
} else {
    $cache->set($key, 'Leon', 5);
}

return ['cache' => 'no cache'];
```

### Session

最后是redis用作session。也是要在组件中配置:

```php
'components' => [
    'session' => [
        'name' => 'advanced-frontend',
        'class' => 'yii\redis\Session'
    ],
],
```

如果没有配置过redis，同样需要配置:

```php
'components' => [
    'session' => [
        'name' => 'advanced-frontend',
        'class' => 'yii\redis\Session',
        'redis' => [
            'hostname' => 'localhost',
            'port' => 6379,
            'database' => 0,
        ],
    ],
],
```

示例代码如下：

```php
$session = Yii::$app->session;
$key = 'username';
if ($session->has($key)) {
    return ['session' => $session->get($key)];
} else {
    $session->set($key, 'Leon');
}
return ['session' => 'no session'];
```

在简单的示范下，如何将redis这个高效的工具用好，则是考验大家的能力了。加油吧！