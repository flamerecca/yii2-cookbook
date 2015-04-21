處理 cookies
================

用純 PHP 處理 HTTP cookies 並不難，但是 Yii 讓它用起來更方便。這篇訣竅裡面說明如何進行一般的 cookie 操作。

發送cookie
----------------

要發送一個cookie，換句話說，建立一個 cookie 並傳輸至瀏覽器，要先建立一個新的`\yii\web\Cookie`物件，並加入 cookie 回應的集合裡面：

```php
$cookie = new Cookie([
    'name' => 'cookie_monster',
    'value' => 'Me want cookie!',
    'expire' => time() + 86400 * 365,
]);
\Yii::$app->getResponse()->getCookies()->add($cookie);
```

上面的 code，我們將參數傳輸至 Yii 內 cookie 類別的 constructor，這些參數基本上與 PHP [setcookie](http://php.net/manual/en/function.setcookie.php) 函數裡面的參數相同：

- `name` - cookie 名稱
- `value` - cookie 的值。請確定該值是字串。瀏覽器基本上不喜歡 cookie 內有 binary 的資料。
- `domain` - cookie 所屬的 domain
- `expire` - cookie 應該過期的 unix timestamp
- `secure` - 如果設為 `true`，cookie 只會在HTTPS連線時傳送
- `httpOnly` - 如果設為 `true`，cookie 無法用 JavaScript 操作。

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

Session cookie 參數
-------------------------

???
TBD

其他資料
--------

- [API reference](http://stuff.cebe.cc/yii2docs/yii-web-cookie.html)
- [PHP 文件](http://php.net/manual/en/function.setcookie.php)
- [RFC 6265](http://www.faqs.org/rfcs/rfc6265.html)
