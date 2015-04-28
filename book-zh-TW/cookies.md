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

Yii 讀取 cookie 的程式碼如下：

```php
$value = \Yii::$app->getRequest()->getCookies()->getValue('my_cookie');
```

子網域 Cookie
----------------------

因為安全性的關係，預設上要操作 cookie ，只有設定該 cookie 的網頁才行。

舉例來說，如果`example.com`設置了一個 cookie，那麼`www.example.com`就讀不到這個 cookie。

如果你希望子網域可以使用該 cookie，（像是讓admin.example.com，profile.example.com可以操作該cookie），`domain`參數需要特別設置：

```php
$cookie = new Cookie([
	'name' => 'cookie_monster',
	'value' => 'Me want cookie everywhere!',
	'expire' => time() + 86400 * 365,
	'domain' => '.example.com' // <<<=== 重點!!!
]);
\Yii::$app->getResponse()->getCookies()->add($cookie);
```

現在，該cookie 可以讓所有`example.com`的子網域取得了。

跨子網域認證與解析 cookie 
---------------------------------------------------

In case of autologin or "remember me" cookie, the same quirks as in case of subdomain cookies are applying.
But this time you need to configure user component, setting `identityCookie` array to desired cookie config.

打開 Yii 的config 檔案，並在 component 設定裡面新增 `identityCookie` 參數：

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
