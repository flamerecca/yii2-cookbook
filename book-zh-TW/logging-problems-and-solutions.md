紀錄：問題與解決方法
===============================

在Yii裡面，紀錄(Logging) 是非常有彈性的。基本上很簡單，但是要取得所有想要的功能，有時很花時間。下面是一些解決方案。
希望你可以找到需要的功能

紀錄404頁面，並且不發送email
---------------------------------------------

404 not found頁面太常出現了，似乎不大值得寄email提醒。不過，紀錄404頁面有時還是很有幫助的，我們來實做看看！


```php
'components' => [
    'log' => [
        'targets' => [
            'file' => [
                'class' => 'yii\log\FileTarget',
                'categories' => ['yii\web\HttpException:404'],
                'levels' => ['error', 'warning'],
            ],
            'email' => [
                'class' => 'yii\log\EmailTarget',
                'except' => ['yii\web\HttpException:404'],
                'levels' => ['error', 'warning'],
                'message' => ['from' => 'robot@example.com', 'to' => 'admin@example.com'],
            ],
        ],
    ],
],
```

When there's unhandled exception in the application Yii logs it additionally to displaying it
to end user or showing customized error page. Exception message is what actually to be written and
the fully qualified exception class name is the category we can use to filter messages when
configuring targets. 404 can be triggered by throwing `yii\web\NotFoundHttpException` or automatically.
In both cases exception class is the same and is inherited from `\yii\web\HttpException` which is a bit
special in regards to logging. The speciality is the fact that HTTP status code prepended by `:` is
appended to the end of the log message category. 上面的程式碼，我們用 `categories` 以在檔案紀錄內包含 404 錯誤，用 `except` 以在email 提醒中排除 404 錯誤。

即時紀錄
-----------------

Yii 預設會先累積紀錄，等到程式結束或者到達一定的數量（1000筆）時才真正寫入紀錄。有時我們可能會需要是即時的紀錄。像是當我們正在運行重要的任務，希望透過log檢查進度的時候。 這種狀況下，我們需要修改 config 檔如下：

```php
'components' => [
    'log' => [
        'flushInterval' => 1, // <-- 這裡
        'targets' => [
            'file' => [
                'class' => 'yii\log\FileTarget',
                'levels' => ['error', 'warning'],
                'exportInterval' => 1, // <-- 以及這裡
            ],
        ],
    ]
]
```
