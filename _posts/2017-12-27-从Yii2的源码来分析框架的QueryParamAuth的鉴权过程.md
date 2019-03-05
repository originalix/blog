---
layout: post
title: 从Yii2的源码来分析框架的QueryParamAuth的鉴权过程
categories: 后端技术
date: 2017-12-27 19:26:29
keywords: PHP, Yii2

---

Yii是基于PHP语言打造的一款框架，了解PHP的同学对这款框架肯定也不会陌生。而我在最近使用yii2写App接口的时，查看官方了的RESTful Web服务文档，文档中对于授权验证的过程有这样一个介绍:

<!--more-->

```
如果你系那个支持以上3个认证方式，可以使用CompositeAuth，如下所示：

use yii\filters\auth\CompositeAuth;
use yii\filters\auth\HttpBasicAuth;
use yii\filters\auth\HttpBearerAuth;
use yii\filters\auth\QueryParamAuth;

public function behaviors()
{
    $behaviors = parent::behaviors();
    $behaviors['authenticator'] = [
        'class' => CompositeAuth::className(),
        'authMethods' => [
            HttpBasicAuth::className(),
            HttpBearerAuth::className(),
            QueryParamAuth::className(),
        ],
    ];
    return $behaviors;
}

authMethods 中每个单元应为一个认证方法名或配置数组。

findIdentityByAccessToken()方法的实现是系统定义的， 例如，一个简单的场景，当每个用户只有一个access token, 可存储access token 到user表的access_token列中， 方法可在User类中简单实现，如下所示：

use yii\db\ActiveRecord;
use yii\web\IdentityInterface;

class User extends ActiveRecord implements IdentityInterface
{
    public static function findIdentityByAccessToken($token, $type = null)
    {
        return static::findOne(['access_token' => $token]);
    }
}
```

以上是官方文档的原文，而我当时结合我的API接口，感觉最适合我使用的是第三种`QueryParamAuth`类型的验证，就是在请求的url中拼接上`AccessToken`。这也是常见的一种鉴权方式，而实现这些验证，框架又需要我们完成`findIdentityByAccessToken()`函数，所以为了不稀里糊涂的跟着文档弄完了，我决定从源码里探究一下实现鉴权的过程中究竟发生了什么。

至于如何实现这个过程，以及完成认证，我之前在网上已经看到有大量博客写这个了，所以在此我也不再赘述。

首先我们进入`QueryParamAuth`类里,里面有一个`authenticate($user, $request, $response)`函数,这个函数的代码是这样的:

```php
public function authenticate($user, $request, $response)
{
    // 获取get参数里的access-token, $this->tokenParam的具体名称，可以在类中指定
    $accessToken = $request->get($this->tokenParam);
    if (is_string($accessToken)) {
        //存在登录令牌的情况下，执行yii\web\user类里的loginByAccessToken方法
        $identity = $user->loginByAccessToken($accessToken, get_class($this));
        if ($identity !== null) {
            return $identity;
        }
    }
    //没有accessToken直接报错
    if ($accessToken !== null) {
        $this->handleFailure($response);
    }

    return null;
}
```

我把这段源码的注释直接附在代码中，在这里我们能大致知道这个函数做了哪些事情，但是其中`$user, $request, $response`这三个参数又是如何获取传递进来的呢。我们再进入`QueryParamAuth`的父类`AuthMethod`里一探究竟。

打开这个父类，我们能看到`$user, $request, $response`这三个参数是父类中定义的三个公开属性。而在父类中的`beforeAction()`中，获取到这三个方法，

```php
public function beforeAction($action)
{
    $response = $this->response ?: Yii::$app->getResponse();

    try {
        // 这里是重点，父类获取三个参数，传入我们之前的authenticate()函数中。
        $identity = $this->authenticate(
            $this->user ?: Yii::$app->getUser(),
            $this->request ?: Yii::$app->getRequest(),
            $response
        );
    } catch (UnauthorizedHttpException $e) {
        if ($this->isOptional($action)) {
            return true;
        }

        throw $e;
    }

    if ($identity !== null || $this->isOptional($action)) {
        return true;
    }

    $this->challenge($response);
    $this->handleFailure($response);

    return false;
}
```

我们可以看到，在`beforeAction()`函数中获取了参数，并且调用了`authenticate`函数,开始执行认证过程。现在我们知道了参数的由来，那么我们接着回到`QueryParamAuth`类里的`authenticate()`函数中，探究鉴权过程。

在`authenticate()`函数中我们看到有这样一行代码:

```php
$identity = $user->loginByAccessToken($accessToken, get_class($this));
```

在这里我提醒大家，这个`$user`指的是`yii\web\user`这个类，而我之前看到很多网上教程让大家去实现`loginByAccessToken()`这个函数，很多人在实现了这个函数之后问，为什么不调用这个函数，其实这个函数是没有必要实现的，如果你一定要实现这个函数，那么你就得把这里使用的`$user`替换成你自己的User类，因为在这个时候，还不会调用你在config里配置的user类，很多同学有了问题，还是要先看看源码，而不是发现这篇博客不行，就换另一个博客试试。。。

我们来一起看看`loginByAccessToken()`函数的代码:

```php
public function loginByAccessToken($token, $type = null)
{
    /* @var $class IdentityInterface */
    // 这里的identityClass才是我们在config中配置的那个User类
    $class = $this->identityClass;

    //这里开始执行我们在User类中实现的findIdentityByAccessToken()函数
    $identity = $class::findIdentityByAccessToken($token, $type);

    //这里的条件，在鉴权之前，我们得是登录状态
    if ($identity && $this->login($identity)) {
        return $identity;
    }

    // 否则 就算有这个用户，但是不是登录状态 一样不能通过验证
    return null;
}
```

我已经把整个函数的分析写在注释里了，在这个函数里会调用我们在config里配置的User类，并且去执行文档中让我们配置的`findIdentityByAccessToken()`函数，所以我们写的函数在此时才会派上用场，同时我们还得是登录状态才能通过鉴权，登录的话这里先不展开讲了，可以先用yii框架默认页面的登录就能通过。

至此，我们的登录鉴权就已经通过了，如果不通过的小伙伴，可以再消化一下上面的源码分析，相信你一定能查到问题究竟出现在哪一步的。
