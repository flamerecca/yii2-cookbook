處理 cookies
================

用純 PHP 處理 HTTP cookies 並不難，但是 Yii 讓它用起來更方便。這篇訣竅裡面說明如何進行一般的 cookie 操作。

發送cookie
----------------

要發送一個cookie，換句話說， to create it and schedule for sending to the browser，要先建立一個新的`\yii\web\Cookie`物件，並加入 cookie 回應的集合裡面：

```php
$cookie = new Cookie([
    'name' => 'cookie_monster',
    'value' => 'Me want cookie!',
    'expire' => time() + 86400 * 365,
]);
\Yii::$app->getResponse()->getCookies()->add($cookie);
```

In the above we're passing parameters to cookie class constructor. These basically the same as used with native PHP [setcookie](http://php.net/manual/en/function.setcookie.php) function:

- `name` - cookie 名稱
- `value` - cookie 的值。 Make sure it's a string. Browsers typically aren't happy about binary data in cookies.
- `domain` - domain you're setting the cookie for.
- `expire` - unix timestamp indicating time when the cookie should be automatically deleted.
- `path` - the path on the server in which the cookie will be available on.
- `secure` - 如果設為 `true`，cookie 只會在HTTPS連線時傳送
- `httpOnly` - 如果設為 `true`, cookie will not be available via JavaScript.

讀取cookie
----------------

讀取 cookie 的程式碼如下：

```php
$value = \Yii::$app->getRequest()->getCookies()->getValue('my_cookie');
```

子網域 Cookie
----------------------

Because of security reasons, by default cookies are accessible only on the same domain from which they were set.
For example, if you have set a cookie on domain `example.com`, you cannot get it on domain `www.example.com`.
So if you're planning to use subdomains (i.e. admin.example.com, profile.example.com), you need to set `domain`
explicitly:

```php
$cookie = new Cookie([
	'name' => 'cookie_monster',
	'value' => 'Me want cookie everywhere!',
	'expire' => time() + 86400 * 365,
	'domain' => '.example.com' // <<<=== HERE
]);
\Yii::$app->getResponse()->getCookies()->add($cookie);
```

Now cookie can be read from all subdomains of `example.com`.

Cross-subdomain authentication and identity cookies
---------------------------------------------------

In case of autologin or "remember me" cookie, the same quirks as in case of subdomain cookies are applying.
But this time you need to configure user component, setting `identityCookie` array to desired cookie config.

Open you application config file and add `identityCookie` parameters to user component configuration:

```php
$config = [
    // ...
    'components' => [
        // ...
        'user' => [
            'class' => 'yii\web\User',
            'identityClass' => 'app\models\User',
            'enableAutoLogin' => true,
            'loginUrl' => '/user/login',
            'identityCookie' => [ // <---- here!
                'name' => '_identity',
                'httpOnly' => true,
                'domain' => '.example.com',
            ],
        ],
    ],
];
```

Session cookie parameters
-------------------------

???
TBD

其他資料
--------

- [API reference](http://stuff.cebe.cc/yii2docs/yii-web-cookie.html)
- [PHP 文件](http://php.net/manual/en/function.setcookie.php)
- [RFC 6265](http://www.faqs.org/rfcs/rfc6265.html)
